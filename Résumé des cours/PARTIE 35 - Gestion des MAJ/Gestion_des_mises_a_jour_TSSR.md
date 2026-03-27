# La Gestion des Mises à Jour
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)
**Sujet** : Gestion des mises à jour — activité de sécurité obligatoire
**Date** : Février 2026
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Définition|Définition]]
   - [[#Pourquoi faire des mises à jour ?|Pourquoi faire des mises à jour ?]]
   - [[#Les risques de l'inaction|Les risques de l'inaction]]
2. [[#Types de mises à jour|Types de mises à jour]]
   - [[#Classification par catégorie|Classification par catégorie]]
   - [[#Classification par gravité|Classification par gravité]]
3. [[#Stratégies|Stratégies]]
   - [[#Déploiement immédiat|Déploiement immédiat]]
   - [[#Déploiement testé|Déploiement testé]]
   - [[#Patch Management|Patch Management]]
4. [[#Outils|Outils]]
5. [[#Études de cas|Études de cas]]
   - [[#Milieu industriel|Milieu industriel]]
   - [[#Plan de préproduction|Plan de préproduction]]
   - [[#Bonnes pratiques|Bonnes pratiques]]
6. [[#Points clés à retenir|Points clés à retenir]]
7. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> La gestion des mises à jour est une **activité de sécurité obligatoire** pour tout TSSR. Elle touche à la fois la sécurité, les performances, la conformité réglementaire et la réputation de l'entreprise. Ne pas faire ses MAJ, c'est laisser une porte ouverte aux attaquants.

### Définition

> [!quote] Définition officielle
> En informatique, une **mise à jour** (*update* en anglais) est une modification apportée à un logiciel ou un OS pour améliorer ses performances, corriger des erreurs, éliminer des vulnérabilités de sécurité ou ajouter de nouvelles fonctionnalités.
>
> **Synonymes :** MAJ, correctif, patch

**Deux étapes distinctes :**
- **Publication** : mise à disposition de la MAJ par l'éditeur (ex : Microsoft via Patch Tuesday)
- **Installation** : application effective du correctif sur le système cible

### Pourquoi faire des mises à jour ?

> [!important] Les 4 objectifs des MAJ

| Objectif | Exemples concrets |
|----------|------------------|
| **Sécurité** | MAJ antivirus (logiciel + base de signatures), correctifs de failles |
| **Performance** | MAJ de pilote graphique ou réseau |
| **Nouvelles fonctionnalités** | MAJ OS → modification du type de domaine AD |
| **Correction de bugs** | MAJ de version logicielle corrigeant des dysfonctionnements |

> [!warning] Failles de sécurité majeures liées aux MAJ non appliquées
> - **Heartbleed** (2012) : faille SSL/TLS massive
> - **WannaCry** (2017) : ransomware exploitant EternalBlue
> - **Spectre** (2018) : faille matérielle des processeurs
> - **Log4Shell** (2021) : faille critique dans la librairie Java Log4j

### Les risques de l'inaction

> [!warning] L'exemple WannaCry — Un effet papillon critique
> - **2012** : Faille EternalBlue découverte par la NSA (accès administrateur à distance)
> - **14/03/2017** : Microsoft publie un correctif (MAJ disponible)
> - De nombreuses entreprises **n'appliquent pas** cette MAJ
> - **12/05/2017** : WannaCry (exploitant EternalBlue) est lancé
>   - But : bloquer un ordinateur et demander une rançon (ransomware)
>   - **Jusqu'à 300 000 ordinateurs impactés dans plus de 150 pays**
>
> ⇒ **Morale** : appliquer les MAJ de sécurité **rapidement** après leur publication !

**Conséquences d'une absence de MAJ :**

| Domaine | Conséquence |
|---------|------------|
| **Sécurité** | Vulnérabilités zero-day exploitables, RCE (Remote Code Execution) |
| **Performance** | Détérioration des performances, incompatibilités, défaillances système |
| **Conformité** | Non-respect des exigences légales et réglementaires (CNIL, etc.) |
| **Réputation** | Impact sur l'image de l'entreprise en cas d'incident |

> [!example] Types de vulnérabilités courantes
> - Vulnérabilités de **configuration**
> - Vulnérabilités **logicielles** (bugs, failles de code)
> - Vulnérabilités **matérielles** (ex : Spectre, Meltdown)
> - Vulnérabilités **utilisateur** (phishing, ingénierie sociale)
> - Vulnérabilités **réseau** et **applicatives**

---

## Types de mises à jour

### Classification par catégorie

> [!info] 2 grandes catégories de MAJ

**MAJ de fonctionnalités et correctifs** *(optionnelle)*
- Correctifs : petites modifications pour résoudre des bugs
- Fonctionnalités : ajouts ou améliorations significatives

**MAJ de sécurité** *(prioritaire)*
- Modifications corrigeant une vulnérabilité ou renforçant la sécurité
- Protègent contre : virus, malwares, attaques réseau, exploitations de failles
- **Doivent être appliquées rapidement** pour minimiser la fenêtre de vulnérabilité

### Classification par gravité

> [!important] Les 3 niveaux de gravité — À connaître pour le RNCP

| Niveau | Priorité | Objectif | Exemples |
|--------|---------|----------|----------|
| **Mineure** | Optionnelle | Corrections de bugs, ajustements visuels, optimisation | MAJ d'interface, optimisation de code |
| **Majeure** | Prioritaire | Nouvelles fonctionnalités, modifications substantielles | Nouveau module, refonte d'architecture |
| **Critique** | **Obligatoire** | Correction de failles de sécurité | Patch zero-day, MAJ d'urgence anti-attaque |

> [!tip] Mnémotechnique — Les niveaux de gravité
> **M**ineure → **M**odification légère
> **M**ajeure → **M**utation importante
> **C**ritique → **C**orrectif d'urgence
> *(M, M, C — "Mets-Mets Correctif")*

---

## Stratégies

### Déploiement immédiat

> [!info] Déploiement immédiat — "Tout, maintenant !"
> Les MAJ sont déployées sur tous les systèmes **dès qu'elles sont disponibles**.

| | Détail |
|-|--------|
| **Avantages** | Rapidité (correction immédiate des failles), Simplicité (moins de planification) |
| **Inconvénients** | Risque de bugs introduits par les MAJ, Moins de contrôle pour les admins |
| **Pour qui ?** | Usage individuel/personnel, PME, TPME |

### Déploiement testé

> [!info] Déploiement testé — "Il est urgent de réfléchir"
> Les MAJ sont d'abord testées sur un panel avant déploiement global.

| | Détail |
|-|--------|
| **Avantages** | Fiabilité (problèmes identifiés avant la prod), Planification maîtrisée |
| **Inconvénients** | Délai (fenêtre de vulnérabilité plus longue), Ressources supplémentaires nécessaires |
| **Pour qui ?** | Grandes entreprises avec un service SI et du personnel dédié |

> [!example] Comparatif des deux approches

```
Déploiement IMMÉDIAT :
MAJ disponible → Installation directe → ⚠️ Risque de régression

Déploiement TESTÉ :
MAJ disponible → Tests (groupe pilote) → Validation → Déploiement progressif → ✅ Maîtrisé
```

### Patch Management

> [!important] Patch Management — La gestion professionnelle des correctifs
> Le **Patch Management** (Gestion des Correctifs) est une approche structurée pour gérer le déploiement des MAJ.

**Contexte justifiant le Patch Management :**
- Les MAJ peuvent être publiées régulièrement (mensuellement) ou de façon irrégulière
- Il peut exister des MAJ mineures ET majeures pour une même cible (OS, logiciel…)
- Le CoT (Cost of Ownership) est élevé sans outil dédié

**Stratégie de Patch Management — Les 3 piliers :**

| Pilier | Description |
|--------|-------------|
| **Outil dédié** | Serveur central de gestion des MAJ (WSUS, Ivanti…) |
| **Publication automatique** | Synchronisation et distribution automatisées |
| **Délais d'installation** | Fenêtres de maintenance planifiées |

---

## Outils

> [!info] Les principaux outils de Patch Management

| Outil | OS cible | Spécificité |
|-------|---------|-------------|
| **WSUS** | Windows uniquement | Rôle Windows Server gratuit (hors licence), gère les produits Microsoft |
| **Ivanti Patch Management** | Windows + multi-OS | Solution propriétaire, gère aussi les produits non-Microsoft |
| **APT** | Linux (Debian/Ubuntu) | Gestionnaire de paquets, gère MAJ et installations |

> [!note] WSUS en détail
> WSUS (Windows Server Update Services) permet :
> - Centralisation des mises à jour
> - Contrôle administratif
> - Économie de bande passante
> - Rapports et tableaux de bord
> - Automatisation du déploiement
> - Liaison avec l'Active Directory

---

## Études de cas

### Milieu industriel

> [!example] Problématique industrielle — Un cas réel complexe

**Contexte :**
Windows 11, dès lors qu'il est connecté à internet sans serveur de MAJ, ne permet de retarder les MAJ que de **2 à 3 semaines maximum**.

**Problème :**
Certaines MAJ Windows peuvent gêner ou empêcher le démarrage de logiciels métier industriels (supervision, GMAO…).

**Scénario problématique :**
```
MAJ posant problème → Désinstallation → Déconnexion Internet
→ Reconnexion Internet → La MAJ se réinstalle automatiquement ← ♻️ Boucle infinie !
```

**Solution 1 : Isolation totale** *(non recommandée)*
- Isoler totalement le poste d'Internet → La MAJ ne s'applique plus
- ⚠️ Mais : plus de MAJ antivirus, risques via clé USB, disque dur externe, ERP…
- ⇒ Projets informatiques compromis

**Solution 2 : Gestion maîtrisée** *(recommandée)*
- Stratégie de MAJ régulière et maîtrisée du parc Windows
- Intégration des MAJ des logiciels métier dans un plan de maintenance
- Serveur dédié pour déployer ou bloquer les MAJ
- **⇒ Mise en place d'un PCA/PRA obligatoire**

### Plan de préproduction

> [!info] Déploiement en préproduction — La méthode professionnelle

**Caractéristiques d'un bon plan de préproduction :**

| Étape | Description |
|-------|------------|
| **Groupement** | Création de groupes de machines pour différer les MAJ |
| **Gestion centralisée** | Outil tiers (WSUS, Ivanti…) pour piloter les déploiements |
| **Validation manuelle** | Chaque publication est approuvée par un gestionnaire |
| **Évaluation des risques** | Feedback utilisateurs + reporting console à chaque étape |

**Processus type :**
```
Patch Tuesday (S2) → TEST1 (S2) → TEST2 (S3) → PROD1 (S4) → PROD2 (S1)
                         ↓            ↓            ↓
                    Validation   Validation   Validation
```

### Bonnes pratiques

> [!tip] Les bonnes pratiques de gestion des MAJ en entreprise

1. **Ne pas tarder** pour tester ou déployer les MAJ (surtout les critiques)
2. Télécharger les MAJ **uniquement depuis les sources officielles**
3. **Définir clairement les cibles** : machines, OS, applications concernées
4. **Planifier** la publication et l'installation (fenêtres de maintenance)
5. Mettre en place un **PCA/PRA** pour les environnements critiques
6. Assurer une **communication utilisateurs** avant les redémarrages planifiés

> [!warning] Erreurs à ne jamais commettre
> - Appliquer des MAJ critiques en production sans passer par un environnement de test
> - Négliger les MAJ antivirus (base de signatures = aussi important que les MAJ OS)
> - Laisser des machines sans MAJ de sécurité trop longtemps (fenêtre d'exploitation)
> - Télécharger des MAJ depuis des sources non-officielles

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définitions essentielles
- **MAJ** = modification d'un logiciel/OS pour corriger, améliorer ou sécuriser
- **Publication** ≠ **Installation** — deux étapes bien distinctes
- **Patch Management** = approche structurée de gestion des MAJ en entreprise

### Types de MAJ (par gravité)
- **Critique** → Obligatoire et immédiate (failles de sécurité)
- **Majeure** → Prioritaire (nouvelles fonctionnalités majeures)
- **Mineure** → Optionnelle (corrections mineures, UI)

### Stratégies de déploiement
- **Immédiat** : simple mais risqué — pour PME/particuliers
- **Testé** : fiable mais plus lent — pour grandes entreprises
- **Patch Management** : approche structurée avec outil dédié, groupes, validation

### Outils
- **Windows** : WSUS (gratuit, Microsoft uniquement) ou Ivanti (payant, multi-éditeur)
- **Linux** : APT (Debian/Ubuntu), YUM/DNF (Red Hat/Fedora)

### L'exemple WannaCry
- Faille publiée → Correctif disponible → Non-appliqué → Ransomware → 300 000 victimes
- **Leçon** : appliquer les MAJ de sécurité **rapidement** !

> [!warning] Points d'attention critiques
> - Les MAJ **critiques** ne se discutent pas : elles s'installent en priorité
> - Toujours tester avant de déployer en production (déploiement testé)
> - Le Patch Management nécessite des ressources dédiées et un outil centralisé
> - En milieu industriel : prévoir un PCA/PRA avant toute stratégie de MAJ

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|-----------|
| **MAJ / Update** | Mise à jour — modification d'un logiciel pour corriger, améliorer ou sécuriser |
| **Patch** | Synonyme de correctif, souvent utilisé pour les MAJ de sécurité |
| **Patch Management** | Gestion structurée et centralisée du déploiement des correctifs |
| **Zero-day** | Vulnérabilité exploitée avant qu'un correctif ne soit disponible |
| **RCE** | Remote Code Execution — exécution de code à distance via une faille |
| **EternalBlue** | Faille Windows (SMB) exploitée par WannaCry en 2017 |
| **WannaCry** | Ransomware de 2017 exploitant EternalBlue sur les parcs non mis à jour |
| **Ransomware** | Logiciel malveillant chiffrant les données et demandant une rançon |
| **Patch Tuesday** | Publication mensuelle des correctifs Microsoft (2ème mardi du mois) |
| **Déploiement immédiat** | Stratégie d'application des MAJ dès leur disponibilité |
| **Déploiement testé** | Stratégie de test des MAJ sur un panel avant déploiement global |
| **PCA** | Plan de Continuité d'Activité — garantit la continuité des services en cas d'incident |
| **PRA** | Plan de Reprise d'Activité — permet de reprendre l'activité après un sinistre |
| **CoT** | Cost of Ownership — coût total de possession/d'exploitation |
| **WSUS** | Windows Server Update Services — outil Microsoft de gestion centralisée des MAJ |
| **Ivanti** | Solution propriétaire de Patch Management multi-éditeur |
| **APT** | Advanced Package Tool — gestionnaire de paquets Linux (Debian/Ubuntu) |
| **GMAO** | Gestion de Maintenance Assistée par Ordinateur — logiciel métier industriel |
| **CNIL** | Commission Nationale de l'Informatique et des Libertés — autorité française de protection des données |
