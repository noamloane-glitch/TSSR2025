# Suivi des incidents

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Suivi des incidents - Helpdesk et gestion d'incidents  
**Date** : Janvier 2026  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#ITIL - Information Technology Infrastructure Library|ITIL - Information Technology Infrastructure Library]]
   - [[#Qu'est-ce qu'un incident ?|Qu'est-ce qu'un incident ?]]
   - [[#Qu'est-ce qu'un problème ?|Qu'est-ce qu'un problème ?]]
   - [[#Incident vs problème|Incident vs problème]]
   - [[#Conséquences et importance|Conséquences et importance]]
2. [[#Organisation d'une équipe support|Organisation d'une équipe support]]
   - [[#Le Système d'Information|Le Système d'Information]]
   - [[#Le Support SI|Le Support SI]]
   - [[#Personnels et compétences|Personnels et compétences]]
   - [[#Les niveaux de gestion|Les niveaux de gestion]]
3. [[#Gestion des incidents|Gestion des incidents]]
   - [[#Procédure de gestion - Les 8 étapes|Procédure de gestion - Les 8 étapes]]
   - [[#SLA - Service Level Agreement|SLA - Service Level Agreement]]
   - [[#Schéma complet du processus|Schéma complet du processus]]
4. [[#Démarche de diagnostic|Démarche de diagnostic]]
   - [[#Importance du diagnostic|Importance du diagnostic]]
   - [[#Les étapes du diagnostic|Les étapes du diagnostic]]
   - [[#Exemples de mauvaises déterminations|Exemples de mauvaises déterminations]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> La gestion des incidents informatiques est un processus essentiel pour tout service support. Elle permet de maintenir la qualité de service, de minimiser les interruptions et d'assurer la satisfaction des utilisateurs. Ce document présente les concepts fondamentaux du helpdesk, la méthodologie ITIL, et les processus de gestion d'incidents selon les meilleures pratiques professionnelles.

### Pourquoi étudier le suivi des incidents ?

En tant que **TSSR**, tu dois :
- Comprendre le rôle et l'organisation d'un service support
- Maîtriser le processus de gestion d'incidents selon ITIL
- Savoir différencier incident, problème et demande de service
- Appliquer une démarche de diagnostic méthodique
- Connaître les niveaux de support et d'escalade

---

## ITIL - Information Technology Infrastructure Library

> [!quote] Définition
> **ITIL** (Information Technology Infrastructure Library) est un cadre de travail (méthodologie) qui fournit des meilleures pratiques pour la gestion des services informatiques (**ITSM** - IT Service Management).

### Objectif d'ITIL

> [!info] Approche ITIL
> ITIL propose une approche détaillée pour :
> - La **conception** des services informatiques
> - La **livraison** des services
> - La **maintenance** des services
> - L'**alignement** des services IT sur les besoins de l'entreprise

### Pourquoi utiliser ITIL ?

> [!success] Avantages d'ITIL pour l'ITSM
> - **Méthodologie structurée** pour gérer incidents et problèmes
> - **Minimisation des impacts** sur l'entreprise
> - **Cadre spécifique** pour l'ITSM (contrairement à Agile/Scrum)

> [!note] ITIL vs Agile/Scrum
> **Différences fondamentales** :
> - **Agile et Scrum** : Axées sur le développement de produits et la gestion de projets
> - **ITIL** : Spécifiquement conçu pour l'ITSM (processus de gestion d'incidents et de problèmes)
> 
> ITIL est donc plus adapté au contexte support et maintenance que les méthodologies agiles.

---

## Qu'est-ce qu'un incident ?

> [!quote] Définition officielle
> Un **incident informatique** est un événement imprévu qui perturbe ou diminue le fonctionnement d'un Système d'Information ⇒ diminution de la **QoS** (Qualité de Service).

> [!important] Caractéristique clé
> Un incident **ne fait pas partie du fonctionnement normal** d'un service.

### Exemples d'incidents

> [!example] Cas concrets d'incidents
> 1. **Incident d'authentification** : Un utilisateur ne peut pas se connecter au SI interne
> 2. **Incident réseau** : Le réseau de l'entreprise est soudainement devenu très lent
> 3. **Incident d'accès** : L'accès à une application spécifique est bloqué pour un groupe d'utilisateurs

---

## Qu'est-ce qu'un problème ?

> [!quote] Définition ITIL
> Selon ITIL, un **problème** est la **cause inconnue** d'un ou plusieurs incidents.

### Relation incident-problème

> [!info] Principe fondamental
> Un problème explique **pourquoi** un incident se produit. C'est la racine du dysfonctionnement.

### Exemples de problèmes

> [!example] Causes des incidents précédents
> Pour les incidents listés plus haut, voici leurs problèmes associés :
> 
> 1. **Mauvaise configuration du service d'authentification** → empêche certains utilisateurs de se connecter au SI
> 2. **Mauvaise configuration d'un routeur** → Goulot d'étranglement sur le réseau
> 3. **Droits d'accès mal configurés** → Blocage de l'accès à une application pour un groupe d'utilisateurs

---

## Incident vs problème

> [!important] Différence essentielle
> **Il ne faut JAMAIS confondre incident et problème.**
> 
> - ⇒ Un **incident** est l'**effet** d'un problème
> - ⇒ Un **problème** est la **cause** d'un ou plusieurs incidents

### Gestion différenciée

> [!note] Approches distinctes
> Le processus de gestion des incidents **diffère** de la gestion des problèmes :
> 
> **Gestion des problèmes** :
> - Met un accent plus important sur la **prévention**
> - Un problème peut être dans un **état latent** (n'a pas encore causé d'incident)
> - Recherche de la **cause racine**
> 
> **Gestion des incidents** :
> - Focus sur la **restauration rapide du service**
> - Résolution **immédiate** (même avec solution de contournement)
> - Traite les **symptômes** en priorité

> [!tip] Mnémotechnique
> - **I**ncident = **I**mpact (ce qui est visible, l'effet)
> - **P**roblème = **P**ourquoi (la cause cachée)

---

## Conséquences et importance

### Les conséquences d'un incident

> [!warning] Effets d'un incident non géré
> Conséquences plus ou moins graves :
> - **Temps d'arrêt** (downtime)
> - **Perte de données** (data loss)
> - **Perturbation des activités** métier
> 
> ⇒ Il est **crucial** de suivre et de gérer efficacement les incidents informatiques afin de minimiser leur impact sur l'entreprise.

> [!info] Notions clés
> - **Criticité** : Gravité de l'incident
> - **Priorité** : Ordre de traitement basé sur criticité et impact

### Importance de la gestion d'incidents

> [!success] Bénéfices d'une gestion efficace
> Pour une entreprise, une bonne gestion d'incidents apporte :
> 
> **Bénéfices directs** :
> - **Réduction des temps d'arrêt** → minimisation des perturbations
> - **Économies de coûts** probables
> - **Protection des données** (éviter les fuites)
> 
> **Bénéfices indirects** :
> - **Satisfaction des clients** en hausse (résolution rapide et efficace)
> - **Amélioration de la réputation** de l'entreprise
> - **Confiance** accrue dans le SI

---

## Organisation d'une équipe support

### Le service en charge

> [!info] Équipe Support SI
> En entreprise, dès qu'un service informatique atteint une taille conséquente, des collaborateurs sont dédiés au support.
> 
> Ce service peut être :
> - **Distinct** du service Infrastructure réseaux
> - **Fusionné** avec le service Infrastructure

---

## Le Système d'Information

> [!quote] Définition du SI
> Le **Système d'Information** met l'information (un ensemble de données) au service de l'entreprise.

### Rôle du SI

> [!info] Interactions du SI
> Le SI est perpétuellement en interaction avec :
> - Le **système de décision** (ou de pilotage)
> - Le **système opérationnel** (production des objectifs fixés)

### Structure du SI

> [!note] Composantes du SI
> Le SI est structuré autour de trois composantes :
> - **Organisationnelles** : Processus, procédures
> - **Humaines** : Utilisateurs, gestionnaires
> - **Technologiques** : Matériel, logiciels, réseaux
> 
> Ces composantes permettent de **recueillir, stocker, traiter et diffuser** l'information entre l'ensemble des acteurs pour pérenniser l'action de l'entreprise.

### Schéma du SI

```
┌─────────────────────────┐
│  Système de Décision    │
│                         │
└───────────┬─────────────┘
            │ Décisions
            ↓
┌─────────────────────────┐
│ Système d'Informations  │←──── Informations de régulation
│                         │
└───────────┬─────────────┘
            │ Ordres précis
            ↓
┌─────────────────────────┐
│  Système Opérationnel   │
│                         │
└─────────────────────────┘

Flux d'informations : Comptes-rendus (↑) + Ordres (↓)
Flux de biens et services : Opérations métier
```

---

## Le Support SI

> [!quote] Définition
> Le **Support SI** (ou équipe support) fait partie du système opérationnel et dépend du SI, tout comme le service Réseaux Infrastructure ou l'Informatique décisionnelle.

### Caractéristiques du Support SI

> [!info] Nature du Support SI
> - Peut être **interne** à l'entreprise ou **externalisé**
> - C'est avant tout un **centre de service** composé d'informaticiens qualifiés
> - Centre névralgique de la relation utilisateurs-SI

### Terminologie

> [!note] Différentes appellations
> Les termes désignant ce support sont multiples :
> - **Équipe support** ou **Support SI**
> - **Hot line**
> - **Helpdesk**
> - **Centre de services** (Service Desk)
> 
> Ces termes sont souvent utilisés de manière interchangeable, même si "Service Desk" est le terme ITIL officiel.

---

## Personnels et compétences

### Les personnels du Support SI

> [!info] Profils du support
> Ce sont des **techniciens et/ou ingénieurs** qui sont à l'écoute des utilisateurs afin de répondre à :
> - **Déclarations d'incidents**
> - **Demandes d'utilisation** d'outils informatiques (messagerie, internet, accès à distance...)

> [!note] Encadrement
> Le support peut être géré par :
> - Un **responsable de support**
> - Un **superviseur**
> - Un **team-leader**
> 
> Ces responsables interviennent sur des problématiques d'**organisation** ET dans la **réponse technique**.

### Pourquoi organiser un support SI ?

> [!success] Objectifs du support
> - **Faciliter** le quotidien d'une entreprise dans la manipulation et l'exploitation de son parc informatique
> - **Assister** les utilisateurs (périmètre technique, formation)
> - **Gérer** le maintien en condition opérationnelle (**MCO**)
> - Avoir un **catalogue de services** clair
> - Mettre en place des **process qualité** (ITIL, ISO)

### Compétences requises

> [!important] Compétences clés du support SI
> 
> **1. Communication et relation utilisateur**
> - Être en mesure de **comprendre les problèmes** exprimés
> - Excellente **écoute active**
> - **Pédagogie** pour expliquer les solutions
> 
> **2. Diagnostic et résolution**
> - Résolution **rapide** de problèmes techniques
> - Utilisation des **outils et procédures** appropriés
> 
> **3. Connaissances techniques**
> - Connaissances **approfondies** des systèmes informatiques
> - Maîtrise des logiciels, matériels, OS, outils de gestion de réseau
> 
> **4. Gestion**
> - **Gestion de projet** et **gestion du temps**
> - Priorisation des tâches
> 
> **5. Adaptabilité**
> - Forte capacité d'**adaptation**
> - **Apprentissage en continu** pour rester à jour sur les nouvelles technologies

> [!tip] Pour ton CV
> Ces compétences sont à mettre en avant lors de tes recherches d'emploi TSSR !

---

## Les niveaux de gestion

> [!info] Organisation en niveaux
> Le support est classiquement organisé en **4 niveaux** selon la complexité et l'expertise requise.

### Détail des niveaux

| Niveau | Nom | Rôle | Activités |
|--------|-----|------|-----------|
| **Niveau 0** | Accueil | Enregistrement, Catégorisation | Création du ticket, tri des demandes |
| **Niveau 1** | Support initial | Niveau 0 + Priorisation, Résolution par procédures | Application de solutions connues, FAQ |
| **Niveau 2** | Support avancé | Analyse, Résolution par analyse, Suivi | Investigation technique, diagnostic |
| **Niveau 3** | Expertise | Analyse approfondie, Résolution experte, Suivi | Problèmes complexes, R&D, éditeurs |

> [!note] Escalade
> Si un niveau ne peut pas résoudre un incident, il **escalade** au niveau supérieur.
> 
> **Principe** : Résoudre au niveau le plus bas possible pour optimiser les ressources.

> [!example] Exemple de répartition
> - **Niveau 0/1** : Hotline téléphonique, chat support, réinitialisation de mots de passe
> - **Niveau 2** : Techniciens sur site, configuration avancée, dépannage matériel
> - **Niveau 3** : Ingénieurs systèmes/réseaux, problèmes architecturaux, bugs logiciels

---

## Gestion des incidents

## Procédure de gestion - Les 8 étapes

> [!abstract] Vue d'ensemble du processus
> La gestion d'incidents ITIL suit un processus structuré en **8 étapes** permettant un traitement systématique et efficace.

### Tableau récapitulatif

| Ordre | Nom du processus |
|-------|------------------|
| **1** | Identification / Détection |
| **2** | Notification |
| **3** | Enregistrement |
| **4** | Catégorisation et priorisation |
| **5** | Diagnostic et investigation |
| **6** | Suivi (ou escalade) |
| **7** | Résolution (et documentation) |
| **8** | Clôture |

---

### (1) Identification / Détection

> [!quote] Définition
> C'est la **détermination d'un incident**, soit la séparation de ce qui est un fonctionnement inhabituel de l'état normal.

#### Moyens de détection

> [!info] Comment détecter ?
> L'identification peut se faire par :
> - **Personnes** : Utilisateurs, techniciens
> - **Logiciels d'identification** : Outils de supervision, alertes automatiques

> [!example] Exemples de détection automatique
> - Une **panne sur une baie de disque** (alerte RAID)
> - L'**arrêt de la climatisation** en salle serveur (capteur température)
> - Un **excès de connexions** sur le port d'un matériel (monitoring réseau)

#### Importance de l'identification

> [!important] Pourquoi identifier ?
> Cette identification est **obligatoire** car elle permet de séparer les incidents des **autres types de demandes** envoyées au support SI :
> 
> **Demandes de service** :
> - Création d'un compte utilisateur
> - Oubli d'un mot de passe
> - Installation d'un nouveau matériel
> 
> **Demandes d'information** :
> - Demande d'information
> - Conseil
> - Documentation

> [!warning] Piège courant
> Ne pas confondre une **demande de service** (changement planifié) avec un **incident** (événement imprévu). Une demande de mot de passe oublié n'est PAS un incident !

---

### (2) Notification

> [!quote] Définition
> C'est le fait de **signaler un incident** au service concerné, en général le Support SI.

#### Canaux de notification

> [!info] Moyens de signalement
> La notification peut se faire par :
> - **Téléphone** (hotline)
> - **Mail** (adresse dédiée support)
> - **SMS** (alertes automatiques)
> - **Logiciel de gestion d'incidents** (ticketing)
> - **Utilisateur en face à face** (bureau support)

#### Informations à fournir

> [!note] Contenu de la notification
> La notification doit s'accompagner des **informations nécessaires** pour la résolution :
> - **État actuel** avec détails pertinents
> - **Type d'incident**
> - **Utilisateur(s) affecté(s)**
> - **Heure de début**
> - **Impact estimé**

> [!tip] Bonne pratique
> Former les utilisateurs à fournir des informations complètes dès la notification accélère considérablement la résolution.

---

### (3) Enregistrement

> [!quote] Définition
> Avec les informations notifiées, la **création du ticket d'incident** va s'effectuer.

#### Format du ticket

> [!info] Support d'enregistrement
> Ce ticket prend souvent la forme d'un **formulaire à remplir** :
> - **Formulaire logiciel** : Fenêtre d'une application dédiée
> - **Formulaire web** : Page web de ticketing
> 
> L'enregistrement des données s'effectue dans une **base de données** centralisée.

#### Informations typiques d'un ticket

> [!note] Contenu du ticket
> Un ticket d'incident contient généralement :
> - **Numéro unique** (ID du ticket)
> - **Date/heure** de création
> - **Utilisateur demandeur** (nom, service, contact)
> - **Description** du problème
> - **Catégorie** (réseau, poste de travail, application...)
> - **Priorité** (à déterminer)
> - **Statut** (nouveau, en cours, résolu, fermé)
> - **Assigné à** (technicien responsable)
> - **Historique** des actions

> [!example] Exemple de numérotation
> - Ticket #2025-00142
> - INC-20250113-001
> - REQ-2025-W02-015

---

### (4) Catégorisation et priorisation

> [!quote] Définition
> La **catégorisation** est le processus de classement des incidents selon différents critères pour optimiser leur traitement.

#### Critères de catégorisation

> [!info] Classification des incidents
> 
> **Par gravité** :
> - Impact sur le fonctionnement
> 
> **Par urgence** :
> - Délai acceptable de résolution
> 
> **Par type** :
> - Téléphonie
> - Messagerie
> - Réseaux
> - Applications métier
> - Poste de travail
> - Serveurs

#### Niveaux de gravité

> [!note] Échelle de gravité

| Gravité | Description | Exemple |
|---------|-------------|---------|
| **Faible** | L'utilisateur peut continuer à travailler, problème peu gênant | Souris sans fil déchargée |
| **Normal** | L'utilisateur peut continuer à travailler mais c'est gênant, à résoudre dans la journée | Imprimante en panne, contournement possible |
| **Urgent** | L'utilisateur est bloqué partiellement | Accès à une application métier impossible |
| **Critique** | L'utilisateur ne peut plus travailler du tout | Poste de travail HS, panne réseau |

> [!important] Par défaut
> En cas de doute, le niveau de gravité **Normal** est attribué par défaut.

#### Niveaux d'impact

> [!info] Périmètre de l'incident
> 
> | Impact | Périmètre affecté |
> |--------|------------------|
> | **Utilisateur** | Une seule personne |
> | **Service** | Un groupe/département |
> | **Site** | Un bâtiment/site géographique |
> | **Entreprise** | Toute l'organisation |

#### Matrice de priorisation

> [!important] Détermination de la priorité
> La priorité combine **Gravité × Impact**

| | | **Niveau d'impact** | | | |
|---|---|---|---|---|---|
| | | **Utilisateur** | **Service** | **Site** | **Entreprise** |
| **Niveau de gravité** | **Faible** | 😃 Mineur | 😃 Mineur | 😕 Majeur | 😕 Majeur |
| | **Normal** | 😃 Mineur | 😕 Majeur | 😕 Majeur | 😕 Majeur |
| | **Urgent** | 😕 Majeur | 😕 Majeur | ☠️ Critique | ☠️ Critique |
| | **Critique** | 😕 Majeur | ☠️ Critique | ☠️ Critique | ☠️ Critique |

#### Niveaux de priorité

> [!success] Classification finale

**Priorité Mineur** 😃
- Incident à faible impact, peu gênant
- Peut être résolu dans un délai plus long
- **Procédure de gestion classique**

**Priorité Majeur** 😕
- Incident gênant, n'empêche pas complètement l'utilisation
- Nécessite une intervention rapide
- **Procédure de gestion classique**

**Priorité Critique** ☠️
- Incident bloquant, empêche complètement l'utilisation d'un service
- Intervention **immédiate** requise
- **Procédure de gestion de crise**

> [!warning] Gestion de crise
> Les incidents **critiques** déclenchent une procédure spéciale avec :
> - Mobilisation d'équipes supplémentaires
> - Communication vers la direction
> - Plan de continuité d'activité (PCA)
> - Astreintes éventuelles

---

### (5) Diagnostic et investigation

> [!quote] Définition
> À partir des informations de la notification, on va analyser la situation pour arriver à une résolution.

> [!info] Objectif
> Il s'agit de :
> - **Comprendre pourquoi** l'incident s'est produit
> - **Déterminer les actions** à entreprendre pour le résoudre

> [!note] Voir section détaillée
> Le diagnostic fait l'objet d'une section complète : [[#Démarche de diagnostic]]

---

### (6) Suivi (ou escalade)

> [!quote] Définition
> Le **suivi** concerne les incidents avec de longues résolutions ou nécessitant une escalade.

#### Objectifs du suivi

> [!info] Pourquoi suivre ?
> Le suivi permet de :
> - **Relancer** si nécessaire un intervenant
> - **Solliciter** un prestataire externe
> - **Contacter** à nouveau l'utilisateur
> - **Escalader** vers un niveau supérieur si besoin
> - **Tenir informées** les parties prenantes

#### Types de suivi

> [!note] Modalités de suivi
> 
> **Suivi temporel** :
> - Vérification régulière de l'avancement
> - Respect des délais SLA
> 
> **Escalade hiérarchique** :
> - Niveau 1 → Niveau 2 → Niveau 3
> - Information du management si nécessaire
> 
> **Escalade fonctionnelle** :
> - Vers un autre service (réseau, sécurité...)
> - Vers un éditeur/constructeur

> [!tip] Bonne pratique
> Planifier des points réguliers sur les incidents majeurs/critiques (ex: 2x par jour).

---

### (7) Résolution

> [!quote] Définition
> C'est le processus de **traitement et de correction** d'un incident qui va permettre de revenir à un état d'utilisation normal.

> [!info] Actions
> On va :
> 1. **Mettre en œuvre** les actions nécessaires pour corriger le problème
> 2. **Vérifier** que l'incident est effectivement résolu
> 3. **Tester** le retour à la normale
> 4. **Valider** avec l'utilisateur

#### La documentation

> [!important] Capitalisation du savoir
> Il s'agit de **consigner les informations** sur l'incident dans un registre ou une base de données afin de pouvoir en tenir compte dans l'avenir.

> [!note] Supports de documentation
> - **Wiki** (ex : Redmine, Confluence)
> - **FAQ** (Foire Aux Questions)
> - **Base de connaissances** (Knowledge Base)
> - **Gestion documentaire** (SharePoint, Nextcloud)

> [!success] Avantages de la documentation
> - **Réutilisation** : Résolutions plus rapides pour incidents similaires
> - **Formation** : Aide à former les nouveaux arrivants
> - **Amélioration continue** : Identification des incidents récurrents
> - **Autonomie** : Les utilisateurs peuvent trouver des solutions eux-mêmes

#### La communication

> [!info] Retour vers les utilisateurs
> Cette partie, **quelquefois négligée**, est pourtant importante.

> [!note] Objectifs de la communication
> Il s'agit d'**informer les personnes concernées** par l'incident :
> - De la **situation**
> - De la manière dont elle a été **résolue**
> - Des éventuelles **actions préventives**

> [!success] Bénéfices
> Cette communication permet de :
> - **Valider** avec les utilisateurs que tout fonctionne
> - Aller **au-delà** des données techniques du support
> - Améliorer la **satisfaction** utilisateur
> - **Rassurer** sur la prise en charge

> [!tip] Bonne pratique
> Toujours contacter l'utilisateur pour confirmer que le problème est résolu avant de clôturer le ticket.

---

### (8) Clôture

> [!quote] Définition
> Après que le service a été restauré, l'incident est **fermé**, avec une confirmation que l'incident est résolu et que les utilisateurs sont satisfaits.

#### Critères de clôture

> [!info] Conditions de fermeture
> Un ticket peut être clôturé si :
> - Le **service est restauré**
> - L'**utilisateur confirme** la résolution
> - Les **tests** sont concluants
> - La **documentation** est à jour
> - Le **délai de validation** est écoulé (ex: 3 jours sans retour)

> [!warning] Attention
> Ne jamais clôturer un ticket sans validation utilisateur sauf politique explicite de clôture automatique.

#### Informations de clôture

> [!note] Éléments à renseigner
> - **Date/heure de clôture**
> - **Solution appliquée**
> - **Temps passé** (pour statistiques)
> - **Niveau de satisfaction** (si enquête)
> - **Code de fermeture** (résolu, contournement, non reproductible...)

---

## SLA - Service Level Agreement

> [!quote] Définition
> Le **SLA** (Service Level Agreement) est le **contrat** qui définit les niveaux de service attendus et les modalités de fourniture de ce service.

### Critères d'un SLA

> [!info] Éléments définis dans un SLA
> 
> **1. Temps de réponse**
> - Précision du **délai de première réponse** selon criticité
> - Exemple : 15 min pour critique, 2h pour majeur, 8h pour mineur
> 
> **2. Disponibilité**
> - Taux de **disponibilité** des services
> - Exemple : 99,9% pour serveurs critiques, 99% pour réseau
> 
> **3. Qualité**
> - **Taux de résolution** des incidents au premier contact
> - **Taux de satisfaction** des utilisateurs
> - **Respect des délais** de résolution

### Exemple de SLA

> [!example] Tableau SLA typique

| Priorité | Temps de réponse | Temps de résolution | Disponibilité |
|----------|------------------|---------------------|---------------|
| **Critique** | 15 minutes | 4 heures | Intervention 24/7 |
| **Majeur** | 2 heures | 24 heures | Horaires ouvrés |
| **Mineur** | 8 heures | 5 jours ouvrés | Horaires ouvrés |

> [!important] Engagement contractuel
> Le SLA est un **engagement contractuel**. Son non-respect peut entraîner :
> - Des pénalités financières
> - Une dégradation de la réputation
> - La rupture du contrat

---

## Schéma complet du processus

> [!abstract] Vue d'ensemble ITIL
> Voici le schéma complet du processus de gestion d'incidents selon ITIL :

```
┌─────────────────────────────────────────────────────────────┐
│                    Centre de services                        │
│  • Outils de supervision système et réseau                   │
│  • Remontée d'alerte automatique                             │
└────────────────────────┬────────────────────────────────────┘
                         ↓
                  ┌──────────────┐
                  │  Détection   │
                  └──────┬───────┘
                         ↓
                  ┌──────────────┐
                  │Enregistrement│
                  └──────┬───────┘
                         ↓
                  ┌──────────────┐
                  │Classification│←─────── Informations de
                  └──────┬───────┘         configuration (CMDB)
                         ↓
         ┌───────────────────────────────┐
         │  Recherche et diagnostic      │
         └───────────────┬───────────────┘
                         ↓
         ┌───────────────────────────────┐
         │      Support initial (N1)     │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ↓                               ↓
┌─────────────────┐           ┌─────────────────┐
│  Solution ?     │   Non →   │Support avancé   │
│  connue ?       │           │  (N2 et N3)     │
└────────┬────────┘           └────────┬────────┘
         │ Oui                         │
         └──────────┬──────────────────┘
                    ↓
        ┌───────────────────────┐
        │ Résolution et         │←─── Sol. contournement
        │ restauration service  │←─── Gestion des problèmes
        └───────────┬───────────┘ ──→ Demande de changement
                    │
                    ↓ MAJ infos incident
        ┌───────────────────────┐
        │      Clôture          │
        └───────────┬───────────┘
                    │
                    ↓ Compte-rendu
        ┌───────────────────────┐
        │  Centre de service    │──→ Communication
        │  → Utilisateur        │    vers utilisateur
        └───────────────────────┘
                    │
                    ↓
        ┌───────────────────────┐
        │  Rapport de gestion   │
        │  + Mise à jour CMDB   │
        └───────────────────────┘
```

> [!note] CMDB
> **CMDB** (Configuration Management Database) : Base de données contenant les informations de configuration du SI (inventaire, relations, historique).

---

## Démarche de diagnostic

## Importance du diagnostic

> [!quote] Définition
> La **démarche de diagnostic** est importante car elle permet de comprendre **pourquoi** un incident s'est produit et de déterminer les **actions à entreprendre** pour le résoudre.

### Bénéfices d'un bon diagnostic

> [!success] Avantages
> Un diagnostic rigoureux amène :
> 
> **Qualité** :
> - **Assurance** de la bonne identification d'incident
> - **Efficacité** des actions de résolution
> - **Amélioration** de la qualité de service (QoS)
> 
> **Économie** :
> - **Économies de temps** et de coûts (ROI positif)
> - Évite les interventions inutiles
> 
> **Prévention** :
> - **Prévention** des futurs incidents
> - Identification des problèmes sous-jacents
> 
> **Satisfaction** :
> - **Amélioration globale** de la satisfaction utilisateur
> - Confiance accrue dans le support

---

## Les étapes du diagnostic

> [!info] Processus détaillé
> La démarche de diagnostic suit **7 étapes** méthodiques :

1. **Recueil des informations**
2. **Analyse des informations**
3. **Test des hypothèses**
4. **Détermination de la cause de l'incident**
5. **Planification de la résolution**
6. Résolution de l'incident (déjà abordé)
7. Suivi de l'incident (déjà abordé)

---

### (1) Recueil des informations

> [!important] Étape cruciale
> C'est la **première étape** qui va permettre d'avoir une bonne résolution d'incidents.

#### Sources d'information

> [!info] D'où viennent les infos ?
> Ces informations vont être recueillies par :
> 
> - **Entretiens** auprès des utilisateurs
> - **Journaux d'événements** (logs système, application, sécurité)
> - **Retours de supervision** (monitoring, alertes)
> - **Outils de diagnostic** spécialisés

> [!example] Outils de diagnostic courants
> - **SolarWinds** : Monitoring réseau et système
> - **PRTG** : Supervision infrastructure
> - **Wireshark** : Analyse de trafic réseau
> - **Event Viewer** : Journaux Windows
> - **syslog** : Logs Linux/Unix
> - **Nagios / Zabbix** : Supervision open source

#### Questions à poser à l'utilisateur

> [!tip] Les 5W + H (méthode journalistique)
> - **What** (Quoi) : Que se passe-t-il exactement ?
> - **When** (Quand) : Depuis quand ? À quel moment ?
> - **Where** (Où) : Sur quel équipement/application ?
> - **Who** (Qui) : Qui est affecté ? Combien d'utilisateurs ?
> - **Why** (Pourquoi) : Qu'essayiez-vous de faire ?
> - **How** (Comment) : Comment l'incident se manifeste-t-il ?

---

### (2) Analyse des informations

> [!quote] Définition
> Cette étape consiste à **examiner les informations recueillies** et à **déterminer les causes possibles** de l'incident.

#### Méthodes d'analyse

> [!info] Approches possibles
> L'analyse se fait de manière **manuelle** ou à l'aide d'**outils de diagnostic**.

> [!note] Exemples de méthodes d'analyse

**1. Analyse de la chaîne de valeur**
- Décomposer le système en différentes parties
- Analyser chacune d'elles pour déterminer la cause
- **Exemple** : Décomposition du modèle OSI (couche par couche)

**2. Analyse de la cause première (Root Cause Analysis)**
- Remonter aux causes **profondes** de l'incident
- Utilisation de grilles d'analyse structurées
- **Exemple** : Diagramme d'Ishikawa (diagramme en arêtes de poisson)

> [!example] Diagramme d'Ishikawa - Les 5M
> 
> Catégories de causes à analyser :
> - **Matériel** (Machine) : Équipements défaillants
> - **Main d'œuvre** : Erreur humaine, formation
> - **Méthode** : Procédures inadaptées
> - **Matière** : Données corrompues, incompatibles
> - **Milieu** : Environnement (température, réseau...)

#### Modèle OSI pour l'analyse réseau

> [!tip] Diagnostic réseau par couches

| Couche | Nom | Vérifications |
|--------|-----|---------------|
| **7** | Application | Problème applicatif, configuration |
| **6** | Présentation | Encodage, format de données |
| **5** | Session | Sessions TCP, connexions |
| **4** | Transport | Ports, TCP/UDP |
| **3** | Réseau | Routage, IP, ping |
| **2** | Liaison | Switch, VLAN, MAC |
| **1** | Physique | Câbles, connecteurs, LED |

---

### (3) Test des hypothèses

> [!quote] Définition
> Cela consiste à **vérifier les hypothèses** émises lors de l'analyse en effectuant des **tests ciblés**.

> [!info] Objectif
> Les résultats vont **affiner ou valider** les hypothèses émises.

#### Types de tests

> [!note] Exemples de tests possibles

**Test de régression**
- Permet de vérifier que les modifications n'ont **pas causé de problème ailleurs**
- Teste les fonctionnalités connexes

**Test de charge**
- Permet de vérifier la **performance** du système
- Soumission à une forte utilisation simulée
- Détecte les goulots d'étranglement

**Test de sécurité**
- Permet de vérifier la **sécurité** du système
- Détecte les failles de sécurité
- Audit de configuration

> [!example] Exemples concrets
> - **Test ping** : Vérifier la connectivité réseau
> - **Test telnet/nc** : Vérifier l'ouverture d'un port
> - **Test de débit** : iperf, speedtest
> - **Test authentification** : Tentative de connexion
> - **Consultation logs** : Recherche d'erreurs spécifiques

---

### (4) Détermination de la cause

> [!important] Étape finale du diagnostic
> Étape cruciale pour déterminer les **actions à entreprendre** pour la résolution et pouvoir **prévenir les futurs incidents**.

> [!info] Méthode
> Elle consiste à utiliser :
> - Les **résultats des tests**
> - Les **informations recueillies**
> 
> Pour déterminer la **cause exacte** de l'incident.

#### Importance de la précision

> [!warning] Danger d'une mauvaise détermination
> **Importance de la précision** dans la détermination de la cause !
> 
> Si la détermination de la cause est **erronée**, cela peut entraîner :
> - Des **actions inappropriées**
> - La **récurrence** de l'incident
> - Des **coûts supplémentaires** inutiles
> - Une **perte de temps** considérable
> - Une **insatisfaction** utilisateur

---

## Exemples de mauvaises déterminations

> [!warning] Cas d'école à éviter
> Voici plusieurs exemples réels de diagnostics erronés et leurs conséquences.

### Exemple 1 : Problème de connexion réseau

> [!example] Diagnostic initial erroné

**Situation** :
- Un utilisateur signale un **problème de connexion au réseau**

**Cause déterminée** (FAUSSE) :
- Problème de carte réseau

**Action entreprise** :
- Carte réseau **remplacée**

**Résultat** :
- ❌ Le problème **persiste**

---

**Analyse correcte** (après 2ème investigation) :
- Le problème était causé par une **mauvaise configuration du routeur**
- Et NON par la carte réseau de l'ordinateur

**Conséquences** :
- ⚠️ Le remplacement de la carte réseau n'a **pas résolu le problème**
- ⚠️ A entraîné des **coûts supplémentaires inutiles**
- ⚠️ **Perte de temps** pour l'utilisateur et le support

> [!tip] Leçon
> Toujours tester la connectivité au niveau du switch/routeur avant de suspecter le matériel client.

---

### Exemple 2 : Serveur défaillant

> [!example] Diagnostic initial erroné

**Situation** :
- Un **serveur est défaillant**

**Cause déterminée** (FAUSSE) :
- Problème de disque dur

**Action entreprise** :
- Disque dur **remplacé**

**Résultat** :
- ❌ Le problème **persiste**

---

**Analyse correcte** (après 2ème investigation) :
- Le problème était causé par une **mauvaise configuration de la carte mère**
- Et NON par le disque dur

**Conséquences** :
- ⚠️ Le remplacement du disque dur n'a **pas résolu le problème**
- ⚠️ A entraîné des **coûts supplémentaires inutiles**
- ⚠️ **Downtime prolongé** du serveur

> [!tip] Leçon
> Vérifier les logs BIOS/UEFI et les indicateurs LED du serveur avant de remplacer des composants.

---

### Exemple 3 : Base de données lente

> [!example] Diagnostic initial erroné

**Situation** :
- Le fonctionnement d'une **BDD est lent** pour les utilisateurs

**Cause déterminée** (FAUSSE) :
- Pas assez de RAM sur le serveur

**Action entreprise** :
- **Ajout de RAM**

**Résultat** :
- ❌ Le problème **persiste**

---

**Analyse correcte** (après 2ème investigation) :
- Le problème était causé par une **mauvaise configuration de l'allocation de mémoire pour la BDD**
- L'ajout de RAM n'a donc **pas résolu le problème**

**Conséquences** :
- ⚠️ A entraîné des **coûts supplémentaires inutiles**
- ⚠️ **Insatisfaction** des utilisateurs toujours présente

> [!tip] Leçon
> Analyser les métriques de performance (CPU, RAM, I/O disque, requêtes lentes) avant d'ajouter du matériel. Souvent, l'optimisation de configuration suffit.

---

### (5) Planification de la résolution

> [!quote] Définition
> Une fois la cause de l'incident déterminée, il faut **planifier les actions de résolution** avec les **ressources nécessaires** (temps, personnel, matériel).

#### Approche méthodique

> [!info] Planification structurée
> Cela doit être effectué de manière **méthodique** pour s'assurer :
> - De l'**efficacité** des actions
> - Du **minimum d'impact** utilisateur

#### Éléments à prendre en compte

> [!note] Critères de planification
> 
> **1. Priorité de l'incident**
> - Critique : Intervention immédiate
> - Majeur : Dans les heures qui suivent
> - Mineur : Planification normale
> 
> **2. Disponibilité des ressources**
> - Personnel qualifié disponible
> - Matériel/licences nécessaires
> - Fenêtres de maintenance
> 
> **3. Impact sur les utilisateurs**
> - Minimiser les perturbations
> - Communiquer les interruptions planifiées
> - Prévoir des solutions de contournement

> [!example] Exemple de planification
> **Incident** : Serveur de messagerie défaillant (Priorité Critique)
> 
> **Plan de résolution** :
> 1. **Immédiat** : Basculement sur serveur de secours (15 min)
> 2. **J+0** : Diagnostic du serveur principal (2h)
> 3. **J+0** : Remplacement composant défaillant (1h)
> 4. **J+0** : Tests et validation (30 min)
> 5. **J+1** : Retour sur serveur principal lors de la fenêtre de maintenance (2h du matin)
> 6. **J+1** : Communication et vérification utilisateurs

> [!tip] Communication
> Toujours **communiquer le plan** aux utilisateurs et à la direction, surtout pour les incidents critiques/majeurs.

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **ITIL** : Méthodologie de gestion des services IT (ITSM), spécialement adaptée au support
- **Incident** : Événement imprévu perturbant le SI = **EFFET** (ce qui est visible)
- **Problème** : Cause inconnue d'un ou plusieurs incidents = **CAUSE** (ce qui est caché)
- Un problème peut être **latent** (pas encore d'incident visible)

### Organisation du support

- **Support SI** (Helpdesk/Service Desk) : Centre de services composé de techniciens qualifiés
- Peut être **interne** ou **externalisé**
- **4 niveaux** : N0 (enregistrement), N1 (résolution par procédures), N2 (analyse), N3 (expertise)
- **Compétences clés** : Communication, diagnostic, connaissances techniques, gestion, adaptabilité

### Processus de gestion des incidents - Les 8 étapes

1. **Identification/Détection** : Séparer incident des demandes de service
2. **Notification** : Signaler l'incident (téléphone, mail, ticketing)
3. **Enregistrement** : Créer le ticket dans la BDD
4. **Catégorisation et priorisation** : Classifier selon gravité × impact
5. **Diagnostic** : Comprendre pourquoi et comment résoudre
6. **Suivi** : Relances, escalade si nécessaire
7. **Résolution** : Corriger + **documenter** + **communiquer**
8. **Clôture** : Fermer après validation utilisateur

### Priorisation des incidents

**Matrice Gravité × Impact** :
- **Priorité Mineur** 😃 : Gestion classique, délai souple
- **Priorité Majeur** 😕 : Gestion classique, intervention rapide
- **Priorité Critique** ☠️ : Gestion de crise, intervention immédiate

**Niveaux de gravité** :
- Faible : Peu gênant
- Normal : Gênant, contournement possible
- Urgent : Utilisateur bloqué partiellement
- Critique : Utilisateur ne peut plus travailler

**Niveaux d'impact** :
- Utilisateur < Service < Site < Entreprise

### SLA (Service Level Agreement)

- **Contrat** définissant les niveaux de service attendus
- Critères : **Temps de réponse**, **Disponibilité**, **Qualité**
- Engagement contractuel avec possibles pénalités
- Exemple : Incident critique = réponse sous 15 min, résolution sous 4h

### Démarche de diagnostic - 5 étapes principales

1. **Recueil d'informations** : Logs, utilisateurs, outils de supervision
2. **Analyse** : Chaîne de valeur (modèle OSI), cause première (Ishikawa)
3. **Test des hypothèses** : Tests ciblés pour valider/infirmer
4. **Détermination de la cause** : Identifier la **vraie cause** (précision critique !)
5. **Planification** : Organiser la résolution (ressources, impact, priorité)

### Importance de la documentation

- **Base de connaissances** : Wiki, FAQ, procédures
- **Capitalisation** : Résolutions futures plus rapides
- **Formation** : Aide nouveaux arrivants
- **Amélioration continue** : Identifier incidents récurrents

### Communication et clôture

- **Toujours** informer l'utilisateur de la résolution
- Valider que le problème est bien résolu
- Ne **jamais** clôturer sans confirmation utilisateur (sauf politique explicite)

### Pièges à éviter

⚠️ Ne PAS confondre incident et demande de service
⚠️ Ne PAS négliger la communication utilisateur
⚠️ Ne PAS clôturer sans validation
⚠️ **ATTENTION au diagnostic erroné** : Coûts inutiles + incident récurrent
⚠️ Toujours vérifier les hypothèses par des tests avant intervention

### Outils essentiels TSSR

- **Ticketing** : Gestion des incidents (Redmine, ServiceNow, GLPI)
- **Supervision** : SolarWinds, PRTG, Nagios, Zabbix
- **Diagnostic réseau** : Wireshark, ping, traceroute, nmap
- **Logs** : Event Viewer (Windows), syslog (Linux)
- **Documentation** : Wiki, Confluence, base de connaissances

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **ITIL** | Information Technology Infrastructure Library - Cadre de bonnes pratiques pour la gestion des services IT |
| **ITSM** | IT Service Management - Gestion des services informatiques |
| **Incident** | Événement imprévu perturbant ou diminuant le fonctionnement du SI |
| **Problème** | Cause inconnue d'un ou plusieurs incidents |
| **QoS** | Quality of Service - Qualité de Service |
| **Helpdesk** | Centre de support technique (synonyme : Service Desk, Hot line) |
| **Support SI** | Équipe en charge de l'assistance utilisateurs et de la résolution d'incidents |
| **Ticket** | Enregistrement formalisé d'un incident dans un système de gestion |
| **Escalade** | Transmission d'un incident à un niveau supérieur de compétence |
| **MCO** | Maintien en Condition Opérationnelle - Activités assurant le fonctionnement continu |
| **SLA** | Service Level Agreement - Contrat définissant les niveaux de service |
| **Gravité** | Niveau d'impact d'un incident sur le fonctionnement (Faible, Normal, Urgent, Critique) |
| **Impact** | Périmètre affecté par l'incident (Utilisateur, Service, Site, Entreprise) |
| **Priorité** | Classification combinant gravité et impact (Mineur, Majeur, Critique) |
| **CMDB** | Configuration Management Database - Base de données des configurations du SI |
| **N0, N1, N2, N3** | Niveaux de support (0=enregistrement, 1=initial, 2=avancé, 3=expert) |
| **Root Cause Analysis** | Analyse de la cause première - Méthode pour identifier la vraie cause d'un problème |
| **Diagramme d'Ishikawa** | Diagramme en arêtes de poisson pour analyser les causes (5M) |
| **5M** | Machine, Main d'œuvre, Méthode, Matière, Milieu - Catégories d'analyse des causes |
| **Modèle OSI** | Modèle en 7 couches pour l'analyse des problèmes réseau |
| **Base de connaissances** | Référentiel documentant les incidents et leurs résolutions |
| **FAQ** | Foire Aux Questions - Réponses aux questions fréquentes |
| **Solution de contournement** | Mesure temporaire permettant de rétablir le service en attendant la résolution définitive |
| **Gestion de crise** | Procédure d'urgence pour incidents critiques |
| **PCA** | Plan de Continuité d'Activité - Procédures en cas d'incident majeur |
| **Downtime** | Temps d'arrêt d'un service |
| **ROI** | Return On Investment - Retour sur investissement |
| **Astreinte** | Disponibilité en dehors des heures ouvrées |
| **Fenêtre de maintenance** | Période planifiée pour interventions techniques |
| **Monitoring** | Surveillance continue de l'infrastructure IT |
| **Logs** | Journaux d'événements système |
| **Ticketing** | Système de gestion des tickets d'incidents |

---

> [!success] Document de révision complet
> Ce document couvre l'ensemble des concepts essentiels sur le suivi des incidents et la gestion d'un helpdesk pour la préparation du titre RNCP TSSR. La méthodologie ITIL présentée ici est universellement reconnue dans le domaine du support informatique.

> [!tip] Pour aller plus loin
> - Étudier la méthodologie ITIL 4 (dernière version)
> - Pratiquer avec un outil de ticketing (GLPI, osTicket)
> - Créer des fiches de procédures pour incidents courants
> - S'entraîner au diagnostic avec des scénarios d'incidents
> - Maîtriser les outils de supervision (Nagios, PRTG)

---

**Bon courage pour tes révisions ! 📚✨**
