# Windows Server Update Service (WSUS)
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)
**Sujet** : Windows Server Update Service (WSUS)
**Date** : Février 2026
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Définition|Définition]]
   - [[#Contexte : usage personnel vs entreprise|Contexte : usage personnel vs entreprise]]
   - [[#Historique et versions|Historique et versions]]
   - [[#Alternatives à WSUS|Alternatives à WSUS]]
2. [[#Configuration|Configuration]]
   - [[#Pré-requis serveur|Pré-requis serveur]]
   - [[#Groupes d'ordinateurs|Groupes d'ordinateurs]]
   - [[#Statuts des mises à jour|Statuts des mises à jour]]
   - [[#Configuration des clients|Configuration des clients]]
   - [[#Maintenance du serveur|Maintenance du serveur]]
3. [[#Stratégies de déploiement|Stratégies de déploiement]]
   - [[#Patch Tuesday et Exploit Wednesday|Patch Tuesday et Exploit Wednesday]]
   - [[#Planification décalée|Planification décalée]]
   - [[#Cas pratique|Cas pratique]]
4. [[#Surveillance et rapport|Surveillance et rapport]]
5. [[#Scénarios avancés|Scénarios avancés]]
6. [[#Points clés à retenir|Points clés à retenir]]
7. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> WSUS (Windows Server Update Services) est un **rôle intégré à Windows Server** permettant de centraliser et contrôler la distribution des mises à jour des produits Microsoft sur l'ensemble du parc informatique d'une entreprise. C'est une solution **gratuite** (hors coût de la licence Windows Server).

### Définition

> [!quote] Définition officielle
> WSUS est un rôle serveur dont l'objectif est de gérer la distribution des mises à jour des produits Microsoft sur les postes de travail et les serveurs d'une organisation.

**Avantages clés de WSUS :**
- Centralisation des mises à jour
- Contrôle administratif total
- Réduction de la bande passante (téléchargement unique depuis Microsoft)
- Automatisation des déploiements
- Conformité et rapports
- Sécurité améliorée
- Personnalisation des politiques

### Contexte : usage personnel vs entreprise

> [!info] At home vs Entreprise
> **Usage personnel** : Chaque machine Windows possède son propre service Windows Update, contacte directement les serveurs Microsoft et gère ses MAJ de manière autonome.
>
> **En entreprise**, ce mode autonome est problématique car :
> - Une machine ne peut pas redémarrer n'importe quand (continuité de service)
> - Les MAJ ne peuvent pas être installées à la volée (risque de bloquer les ressources)
>
> ⇒ **WSUS résout ces deux problèmes** en centralisant le contrôle.

### Historique et versions

| Année | Version | Remarque |
|-------|---------|----------|
| 2002 | SUS (Software Update Service) | Ancêtre de WSUS |
| 2005 | WSUS v2 | Remplacement de SUS |
| 2006 | WSUS 3 | — |
| 2009 | WSUS 3 SP2 | — |
| 2012 | WSUS 4 | Support Windows 10 et WS 2012 |
| 2016 | WSUS 5 | Windows Server 2016 |
| 2019 | WSUS 10 | Windows Server 2019 |
| 2021 | WSUS 10 | Windows Server 2022 |

### Alternatives à WSUS

> [!note] Autres outils de gestion des MAJ
> - **WSUS Offline Update** : outil libre permettant de télécharger les MAJ hors-ligne
> - **WAPT** : solution propriétaire française de gestion des MAJ et déploiement logiciel

---

## Configuration

### Pré-requis serveur

> [!important] Ressources recommandées pour WSUS
> WSUS est un serveur **gourmand en ressources**. Configuration idéale :
> - **2 CPU**
> - **Minimum 16 Go de RAM**
> - **2 volumes** : 128 Go (système) + 256 Go (stockage des MAJ)
>
> 💡 Le rôle **AD-DS n'est pas obligatoire** pour faire fonctionner WSUS.

### Groupes d'ordinateurs

> [!info] Organisation en groupes
> WSUS permet de créer des groupes d'ordinateurs pour classifier et/ou cloisonner le réseau.

**Deux modes de création de groupes :**
- En autonomie, directement dans la base de données WSUS
- En liaison avec l'Active Directory (AD)

**Types de groupes possibles :**
- Groupes indépendants (ex : "Clients", "Serveurs")
- Groupes reproduisant l'arborescence des OU de l'AD

### Statuts des mises à jour

**Statuts d'approbation (côté serveur) :**

| Statut | Description |
|--------|-------------|
| **Non-approuvée** (Unapproved) | MAJ disponible mais pas encore validée |
| **Approuvée** (Approved) | MAJ validée pour être installée sur les clients |
| **Refusée** (Declined) | MAJ non-validée (peut s'appliquer à une MAJ précédemment approuvée) |

**Statuts d'installation (retour client) :**

| Statut | Description |
|--------|-------------|
| **Échec ou en besoin** (Failed or Needed) | MAJ approuvée en erreur OU MAJ non-approuvée mais nécessaire |
| **Installée/Non-applicable** | MAJ installée OU non applicable (produit absent sur le client) |
| **Échec** (Failed) | MAJ non-installée suite à une erreur |
| **En besoin** (Needed) | MAJ applicable sur le client mais non encore installée |

### Configuration des clients

> [!tip] 3 méthodes de configuration des clients WSUS

**Méthode 1 : Stratégie locale (sans AD)** — pour les machines hors domaine
1. Ouvrir `gpedit.msc`
2. Naviguer dans `Configuration ordinateur → Modèles d'administration → Composants Windows → Windows Update`
3. Activer l'option *"Spécifier l'emplacement intranet du service de mise à jour Microsoft"*
4. Entrer l'URL du serveur : `http://wsus:8530`

**Méthode 2 : Registre (sans AD)** — pour les machines hors domaine
```
Clé : HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate
```
3 valeurs à créer :

| Valeur | Type | Contenu |
|--------|------|---------|
| `WUServer` | String (REG_SZ) | URL du serveur WSUS |
| `WUStatusServer` | String (REG_SZ) | URL du serveur WSUS |
| `UseWUServer` | DWord (REG_DWORD) | `1` |

**Méthode 3 : GPO (avec AD)** — pour les machines en domaine
1. Ouvrir la console de gestion des GPO
2. Naviguer dans `Computer Configuration → Administrative Templates → Windows Components → Windows Update`
3. Activer *"Specify Intranet Microsoft update service location"*
4. Entrer l'URL du serveur : `http://wsus:8530`

> [!note] Paramètres GPO supplémentaires
> En plus de l'emplacement du serveur, on peut configurer via GPO :
> - L'interdiction de redémarrage automatique des clients
> - L'interdiction de redémarrage dans une plage horaire donnée
> - La possibilité de retarder un redémarrage

### Maintenance du serveur

> [!tip] Bonnes pratiques de maintenance WSUS
> - Utiliser le **"Server Cleanup Wizard"** pour nettoyer les vieilles MAJ
> - Vérifier périodiquement (hebdomadairement) que les MAJ sont bien téléchargées
> - Programmer la **synchronisation 2 fois par jour** avec les serveurs Microsoft

---

## Stratégies de déploiement

### Patch Tuesday et Exploit Wednesday

> [!important] Patch Tuesday
> Le **Patch Tuesday** (ou Update Tuesday) désigne la publication par Microsoft des derniers correctifs **le 2ème mardi de chaque mois**.
> - Terme non-officiel, venu avec Windows 98
> - À l'origine limité aux MAJ de sécurité, il couvre maintenant tous les produits Microsoft

> [!warning] Exploit Wednesday — Le danger qui suit le Patch Tuesday
> Juste après le Patch Tuesday, des malwares et exploits sont créés. En faisant du **reverse engineering** sur les rapports de failles publiés, des attaquants créent rapidement des malwares et les diffusent **avant que les correctifs ne soient installés** sur les parcs.
>
> ⇒ Il est donc critique d'appliquer les MAJ de sécurité **rapidement** après le Patch Tuesday !

### Planification décalée

> [!example] Stratégie de déploiement par vagues
> La bonne pratique consiste à déployer les MAJ en **lots successifs** pour limiter les risques :

| S2 (Patch Tuesday) | S3 | S4 | S1 (mois suivant) |
|--------------------|----|----|-------------------|
| Service-TEST1 | Service-TEST2 | Service1 | Service2 |

- **Semaine 2** : Déploiement sur le groupe de test 1
- **Semaine 3** : Déploiement sur le groupe de test 2 (si S2 OK)
- **Semaine 4** : Déploiement sur le lot de production 1
- **Semaine 1** (mois suivant) : Déploiement sur le lot de production 2

### Cas pratique

> [!example] Exemple concret — Entreprise de 1000 machines, 2 sites
> **Composition du parc :**
> - 950 machines clientes (Windows 11, 10, 7)
> - 45 serveurs (Windows Server 2022, 2019, 2016)
> - 5 DC (Windows Server 2022)
>
> **Gestion par OS :**
> - Clients : gestion hebdomadaire
> - Serveurs : gestion hebdomadaire ou mensuelle
> - DC : gestion mensuelle ou planifiée
>
> **Exemple de services :** Direction, DSI, Marketing, RH, Production, Finance, Juridique, Ventes
>
> **Structure des groupes WSUS :**
> - Services non-sensibles (DSI, Marketing…) → 4 groupes : `DSI-TEST1`, `DSI-TEST2`, `DSI1`, `DSI2`
> - Services sensibles (Direction…) → 2 groupes : `DIRECTION1`, `DIRECTION2`

---

## Surveillance et rapport

> [!info] Dashboard et reporting
> WSUS propose une console de suivi avec :
> - Un **dashboard** avec widgets pour visualiser l'état global des MAJ
> - Des statuts par machine et par groupe
> - Un module de **génération de rapports** pour le suivi de conformité

---

## Scénarios avancés

### Intégration avec SCCM

> [!note] WSUS + SCCM (System Center Configuration Manager)
> Dans une architecture WSUS + SCCM :
> - WSUS devient un **serveur "en amont"** (upstream) du serveur SCCM
> - Les MAJ sont **gérées depuis SCCM** (interface unifiée)
>
> **Avantages supplémentaires avec SCCM :**
> - MAJ des applications tierces (pas seulement Microsoft)
> - MAJ de Mac (via addons)
> - Surveillance de l'intégrité et des performances du système

### Gestion via PowerShell

> [!tip] Automatisation avec PowerShell
> Le module `UpdateServices` est installé nativement sur un serveur WSUS. Il permet :
> - Gestion des clients : ajout/suppression de groupe
> - Gestion des MAJ : approbation, refus, etc.
>
> ⇒ Possibilité d'automatiser et scripter l'administration complète de WSUS

### Infrastructure multi-serveurs (grands réseaux)

> [!info] Topologie Upstream / Downstream
> Pour les très grands réseaux ou les entreprises multi-sites, la bonne pratique est de déployer des **serveurs WSUS locaux** (Downstream Servers) qui se synchronisent depuis le serveur WSUS central.
>
> ```
> [Serveurs Microsoft] → [WSUS Central (Upstream)] → [WSUS Local Site A (Downstream)]
>                                                   → [WSUS Local Site B (Downstream)]
> ```

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Architecture et intégration
- WSUS est un **rôle serveur** Windows, gratuit (hors licence WS)
- Peut fonctionner **avec ou sans AD**
- Les groupes d'ordinateurs peuvent être liés à l'AD ou gérés indépendamment

### Configuration des clients
- Configuration possible via **GPO** (avec AD), **gpedit.msc** ou **registre** (sans AD)
- URL du serveur WSUS : `http://[nom_serveur]:8530`
- Clé registre : `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate`

### Stratégie de déploiement
- Toujours utiliser une **planification décalée** par vagues (TEST1 → TEST2 → PROD)
- Appliquer les MAJ de sécurité rapidement après le **Patch Tuesday** (2ème mardi du mois)
- Attention à l'**Exploit Wednesday** : fenêtre de vulnérabilité critique

### Scénarios avancés
- Intégration possible avec **SCCM** pour gérer les MAJ tierces
- Module **PowerShell UpdateServices** pour l'automatisation
- Architecture **Upstream/Downstream** pour les grands réseaux multi-sites

> [!warning] Pièges à éviter
> - Ne pas laisser les MAJ en statut "Non-approuvé" trop longtemps (risque sécurité)
> - Ne pas négliger la maintenance régulière du serveur WSUS (Server Cleanup Wizard)
> - Ne pas oublier de configurer la synchronisation automatique 2x/jour

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|-----------|
| **WSUS** | Windows Server Update Services — rôle Windows Server pour la gestion centralisée des MAJ Microsoft |
| **MAJ** | Mise à jour — correctif ou amélioration logicielle (aussi appelé patch ou update) |
| **Patch Tuesday** | Publication mensuelle des correctifs Microsoft, le 2ème mardi de chaque mois |
| **Exploit Wednesday** | Période suivant le Patch Tuesday où des malwares exploitent les failles publiées |
| **Upstream Server** | Serveur WSUS central qui se synchronise avec Microsoft |
| **Downstream Server** | Serveur WSUS local qui se synchronise avec l'upstream |
| **GPO** | Group Policy Object — objet de stratégie de groupe pour configurer les clients |
| **SCCM** | System Center Configuration Manager — outil Microsoft de gestion avancée du parc |
| **UseWUServer** | Valeur registre (DWord=1) qui force le client à utiliser le serveur WSUS |
| **Server Cleanup Wizard** | Assistant WSUS de nettoyage des anciennes MAJ et métadonnées |
| **WAPT** | Solution propriétaire alternative à WSUS pour la gestion des MAJ |
| **AD-DS** | Active Directory Domain Services — rôle annuaire Windows (non obligatoire pour WSUS) |
