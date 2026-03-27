# Active Directory - partie 1

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Service d'annuaire - Active Directory (partie 1)

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Qu'est-ce qu'Active Directory ?|Qu'est-ce qu'Active Directory ?]]
   - [[#Historique et évolution|Historique et évolution]]
   - [[#Le protocole LDAP|Le protocole LDAP]]
   - [[#Annuaire LDAP vs Base de données relationnelle|Annuaire LDAP vs Base de données relationnelle]]
   - [[#Les différents rôles AD|Les différents rôles AD]]
2. [[#Arborescence AD|Arborescence AD]]
   - [[#Objets AD|Objets AD]]
   - [[#Unités d'Organisation (OU)|Unités d'Organisation (OU)]]
   - [[#Domaine|Domaine]]
   - [[#Workgroup vs Domaine|Workgroup vs Domaine]]
   - [[#Arbre|Arbre]]
   - [[#Forêt|Forêt]]
3. [[#Composants AD|Composants AD]]
   - [[#Contrôleur de domaine (DC)|Contrôleur de domaine (DC)]]
   - [[#RODC (Read Only Domain Controller)|RODC (Read Only Domain Controller)]]
   - [[#Catalogue Global|Catalogue Global]]
4. [[#Points clés à retenir|Points clés à retenir]]
5. [[#Glossaire technique|Glossaire technique]]
6. [[#Liens et ressources|Liens et ressources]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Active Directory (AD) est la mise en œuvre par Microsoft des services d'annuaire LDAP pour les systèmes d'exploitation Windows. C'est un élément central de l'infrastructure réseau d'entreprise qui permet la gestion centralisée des ressources, des utilisateurs et des politiques de sécurité.

### Pourquoi étudier Active Directory ?

En tant que **TSSR**, tu dois maîtriser Active Directory car :
- C'est le système d'annuaire le plus déployé en entreprise (25% du marché des outils de gestion d'identité)
- Il constitue le cœur de l'administration système Windows
- Il permet la gestion centralisée de milliers d'utilisateurs et de ressources
- C'est une compétence indispensable pour le titre RNCP TSSR

---

## Qu'est-ce qu'Active Directory ?

> [!quote] Définition officielle Microsoft
> Un répertoire est une structure hiérarchique qui stocke des informations sur les objets sur le réseau. Un service d'annuaire, tel que Active Directory Domain Services (AD DS), fournit les méthodes permettant de stocker les données d'annuaire et de mettre ces données à la disposition des utilisateurs et administrateurs du réseau. Par exemple, les services de domaine Active Directory stockent des informations sur les comptes d'utilisateurs, comme les noms, les mots de passe, les numéros de téléphone et permettent aux utilisateurs autorisés du même réseau d'accéder à ces informations.

> [!quote] Définition Wikipédia
> Active Directory (AD) est la mise en œuvre par Microsoft des services d'annuaire LDAP (Lightweight Directory Access Protocol, qui est une norme pour les systèmes d'annuaire) pour les systèmes d'exploitation Windows.

> [!info] Définition technique simplifiée
> AD DS est un système qui :
> - Intègre un **stockage** (une base de données hiérarchique)
> - Fournit des **services** pour mettre en relation les utilisateurs et les ressources réseau
> - Contient des **objets** (utilisateurs, ordinateurs, services, etc.)
> - Est administrable en **GUI** ou en **CLI** (PowerShell)
> - Utilise **LDAP** pour accéder à l'annuaire

### Objectifs principaux d'Active Directory

Active Directory a deux objectifs fondamentaux :

1. **Fournir des services centralisés** :
   - Identification des utilisateurs
   - Authentification sécurisée
   - Gestion de politiques dans un réseau multi-OS

2. **Répertorier les éléments du réseau** :
   - Comptes utilisateurs
   - Serveurs et postes de travail
   - Faciliter leur gestion centralisée

> [!tip] Active Directory = Chef d'orchestre
> AD est au cœur de l'infrastructure réseau Windows. Il orchestre l'ensemble des interactions entre utilisateurs, ordinateurs et ressources du réseau.

---

## Historique et évolution

> [!note] Chronologie d'Active Directory
> - **1996** : Nommé initialement **NTDS** (NT Directory Services)
> - **1999** : Première utilisation majeure avec **Windows 2000 Server**
> - Évolution depuis la base de comptes de domaine **SAM** avec l'utilisation du protocole **LDAP**
> - Évolution du stockage AD de **Jet** à **ESENT**

### Avant Active Directory : la base SAM

> [!info] Base SAM (Security Account Manager)
> Avant AD, Windows utilisait la base SAM pour gérer les comptes d'utilisateurs et de groupes sur des ordinateurs locaux ou dans un domaine.
> 
> **Caractéristiques de SAM** :
> - Efficace pour les petits réseaux
> - Limitée en capacités de gestion
> - Peu évolutive pour les grands réseaux
> - Pas de structure hiérarchique

La base SAM était suffisante pour les petits environnements, mais l'évolution vers des réseaux d'entreprise plus complexes a nécessité une solution plus puissante et évolutive : **Active Directory**.

---

## Le protocole LDAP

> [!important] LDAP : Le protocole à la base de l'AD
> **LDAP** (Lightweight Directory Access Protocol) est un protocole standardisé et ouvert pour accéder et gérer les services d'annuaire. Active Directory utilise LDAP pour toutes ses opérations d'annuaire.

### Fonctionnement de LDAP

LDAP permet de :
- **Rechercher** des données dans l'annuaire AD de manière structurée
- **Manipuler** les objets et leurs attributs
- **Interroger** l'annuaire selon des critères précis
- **Modifier** les informations de l'annuaire

### Avantages de LDAP

> [!success] Points forts de LDAP
> - **Grande compatibilité** avec de nombreux services et applications
> - **Méthode standard** d'interrogation et de modification de l'annuaire
> - **Intégration possible** avec d'autres systèmes d'annuaire :
>   - Systèmes GNU/Linux
>   - OpenLDAP
>   - Solutions SSO (Single Sign-On)
>   - Applications tierces supportant LDAP

---

## Annuaire LDAP vs Base de données relationnelle

> [!note] Comparaison des modèles de données
> "Un annuaire LDAP est comme l'annuaire téléphonique"
> 
> Un annuaire LDAP est une **base de données hiérarchique**, ce qui le différencie d'un **SGBD relationnel** classique.

### SGBD relationnel (base de données classique)

> [!example] Exemple : Bibliothèque avec base relationnelle
> 
> **Structure en tables liées** :
> 
> | Table | Colonnes principales |
> |-------|---------------------|
> | **Emprunteurs** | ID_emprunteur, Nom, Prénom, Adresse, Téléphone, Email |
> | **Livres** | ID_livres, Titre, Volume, ISBN, ID_éditeur, Année_parution |
> | **Exemplaires** | ID_exemplaires, ID_livre, ID_emprunteur, Date_emprunt, Date_retour |
> | **Auteurs** | ID_auteur, Nom, Prénom, Date_naissance, Date_décès |
> | **Livre-Auteurs** | ID_livre, ID_auteur, Ordre |

**Exemple de table relationnelle "voiture"** :

| clé | marque | couleur | plaque |
|-----|--------|---------|---------|
| 1 | Renault | bleu | 1233 DC 81 |
| 2 | BMW | rouge | 1213 DC 95 |
| 3 | Audi | orange | 2342 AC 66 |
| 4 | Mercedes | argent | 1234 CD 88 |

**Éléments d'une table relationnelle** :
- **Nom de relation** : le nom de la table (ex: "voiture")
- **Colonnes** : attributs ou champs
- **Tuple** : ligne de données
- **Valeurs** : contenu des cellules

### Base de données hiérarchique (LDAP)

> [!example] Exemple : Classification animale hiérarchique
> 
> ```
> animal
> ├── mammifère
> │   ├── ongulé
> │   └── carnivore
> └── oiseau
>     ├── aigle
>     └── moineau
> ```

> [!example] Exemple : Structure LDAP typique
> 
> ```
> dc=mylab,dc=fr
> ├── ou=users,dc=mylab,dc=fr
> │   ├── uid=user1,ou=users,dc=mylab,dc=fr
> │   └── uid=user2,ou=users,dc=mylab,dc=fr
> └── ou=computers,dc=mylab,dc=fr
>     ├── uid=cptr1,ou=computers,dc=mylab,dc=fr
>     └── uid=cptr2,ou=computers,dc=mylab,dc=fr
> ```

> [!important] Différence clé
> **SGBD relationnel** : Les données sont organisées en **tables avec relations** (clés primaires/étrangères)
> 
> **Annuaire LDAP** : Les données sont organisées en **arborescence hiérarchique** (structure parent-enfant)

---

## Les différents rôles AD

Microsoft propose plusieurs rôles Active Directory pour différents besoins :

### AD DS (Active Directory Domain Service)

> [!important] Le rôle principal
> **AD DS** est le rôle fondamental d'Active Directory.
> 
> **Fonctionnalités** :
> - Mise en œuvre d'un **domaine** et d'un **annuaire** Active Directory
> - Gestion des **utilisateurs**, **ordinateurs**, **groupes**
> - Gestion de l'**ouverture de session**
> - **Contrôle d'accès** aux ressources
> - Application des **stratégies de groupe** (GPO)

### Les autres rôles AD

> [!info] AD CS (Active Directory Certificate Service)
> **Gestion des certificats** :
> - Création et gestion des **clés cryptographiques**
> - Émission et révocation de **certificats numériques**
> - Infrastructure PKI (Public Key Infrastructure)

> [!info] AD FS (Active Directory Federation Services)
> Disponible depuis **Windows Server 2008**
> 
> **Single Sign-On (SSO)** :
> - Gestion d'un **portail SSO** pour les applications
> - Authentification fédérée entre organisations
> - Accès simplifié aux applications cloud et web

> [!info] AD RMS (Active Directory Rights Management Services)
> Disponible depuis **Windows Server 2008 R2**
> 
> **Gestion des droits numériques** :
> - Gestion des **autorisations fines** sur les fichiers
> - Protection des documents sensibles
> - Fonctionne uniquement avec les **applications compatibles** (comme Microsoft Office)

> [!info] AD LDS (Active Directory Lightweight Directory Services)
> **Service d'annuaire allégé** :
> - Annuaire LDAP **sans domaine**
> - Pas de contrôle d'accès Windows intégré
> - Utilisé pour des applications spécifiques nécessitant un annuaire LDAP

---

## Quelques chiffres clés

> [!note] Statistiques Active Directory
> - **25%** du marché dans la catégorie des outils de gestion d'identité et d'accès
> - **Plus de 2 milliards** : Nombre maximum d'objets pouvant être créés
> - **999** : Nombre maximum de GPO applicables à un objet
> - **1015** : Nombre maximum d'appartenances de groupe pour un objet

---

## Arborescence AD

> [!abstract] Structure logique
> L'arborescence AD représente une **structure logique indépendante du site physique**. C'est une organisation virtuelle des ressources réseau qui ne dépend pas de la topologie physique du réseau.

---

## Objets AD

> [!quote] Définition
> Les **objets AD** sont les éléments de base de la base de données Active Directory. Ils représentent les ressources physiques, logiques et les services au sein d'un environnement réseau.

> [!important] Tout est objet dans AD !
> Dans Active Directory, chaque élément (utilisateur, ordinateur, groupe, imprimante, etc.) est représenté par un **objet** avec ses propres attributs.

### Types d'objets courants

Les objets AD peuvent être :
- Des **utilisateurs** (comptes de personnes)
- Des **ordinateurs** (stations de travail, serveurs)
- Des **groupes** (collections d'utilisateurs ou d'ordinateurs)
- Des **imprimantes** (périphériques partagés)
- Des **dossiers partagés** (ressources réseau)
- Des **unités d'organisation** (conteneurs)

### Attributs des objets

Chaque objet possède :
- Un **Distinguished Name (DN)** : chemin complet dans l'arborescence
- Un **Name** : nom de l'objet
- Un **ObjectClass** : type d'objet (user, computer, group, etc.)
- Un **ObjectGUID** : identifiant unique global

> [!example] Exemple PowerShell - Recherche d'objets
> ```powershell
> PS C:\Lab> Get-ADObject -Filter {Name -like "*server*"}
> 
> DistinguishedName                               Name      ObjectClass   ObjectGUID
> -----------------                               ----      -----------   ----------
> CN=Server,CN=System,DC=lab,DC=lan              Server    samServer     a15302e...
> CN=WIN1,OU=Domain Controllers,DC=lab,DC=lan    WIN1      computer      357d4a5...
> ```

---

## Unités d'Organisation (OU)

> [!quote] Définition
> L'**Unité d'Organisation**, ou **OU** (Organizational Unit), est le niveau le plus bas de la structure hiérarchique d'Active Directory.
> 
> Les OUs sont des "boîtes" dans lesquelles les objets tels que les utilisateurs, les groupes et les ordinateurs sont organisés.

### Fonctionnalités des OU

> [!important] À quoi servent les OU ?
> Les Unités d'Organisation permettent :
> 
> 1. **Gestion administrative fine** :
>    - Délégation de droits d'administration
>    - Contrôle granulaire des permissions
> 
> 2. **Organisation logique** :
>    - Structuration des objets de l'annuaire
>    - Reflet de l'organisation de l'entreprise
> 
> 3. **Application de stratégies de groupe** :
>    - Liaison de **GPO** (Group Policy Objects)
>    - Contrôle de l'environnement utilisateurs et ordinateurs
> 
> 4. **Délégation de pouvoir** :
>    - Attribution de droits spécifiques par OU
>    - Gestion décentralisée selon l'organisation

### Structure typique avec OU

> [!example] Exemple d'organisation en OU
> ```
> Domaine entreprise
> ├── OU Utilisateurs
> │   ├── OU Nantes
> │   ├── OU Bordeaux
> │   ├── OU Lyon
> │   └── OU Paris
> ├── OU Ordinateurs
> │   ├── OU PC bureau
> │   ├── OU PC portable
> │   └── OU Serveurs
> └── OU Groupes
> ```

> [!example] Exemple PowerShell - Informations sur une OU
> ```powershell
> PS C:\Lab> Get-ADOrganizationalUnit -Filter {Name -like "*serveurs*"}
> 
> City                     : 
> Country                  : 
> DistinguishedName        : OU=Serveurs,OU=Bordeaux,OU=Ordinateurs,DC=lab,DC=lan
> LinkedGroupPolicyObjects : {}
> ManagedBy                : 
> Name                     : Serveurs
> ObjectClass              : organizationalUnit
> ObjectGUID               : 073ef3cc-76b6-4d30-a241-0e45aef90183
> PostalCode               : 
> State                    : 
> StreetAddress            :
> ```

> [!tip] Bonne pratique
> Organise tes OU selon la **structure de ton entreprise** (par service, par localisation géographique, ou par type de ressource) pour faciliter l'administration et l'application des GPO.

---

## Domaine

> [!quote] Définition
> Un **domaine AD** est une unité administrative et de sécurité dans un environnement Active Directory. Il représente un groupe de ressources réseau et d'utilisateurs qui sont gérés comme une seule entité.

### Fonctionnalités du domaine AD

> [!important] Contrôle Centralisé
> **Gestion centralisée** de :
> - Politiques de sécurité
> - Comptes d'utilisateurs
> - Comptes d'ordinateurs
> - Ressources réseau
> 
> → Permet une **administration simplifiée** pour l'authentification et l'autorisation.

> [!important] Périmètre de Sécurité
> À l'intérieur du domaine :
> - Les **politiques** sont appliquées de manière cohérente
> - Les **contrôles d'accès** sont uniformes
> - Toutes les ressources sont soumises aux **mêmes règles de sécurité**

> [!important] Partage de Ressources
> Les utilisateurs d'un même domaine peuvent :
> - Partager des **fichiers**
> - Accéder aux **imprimantes** réseau
> - Utiliser des **applications** partagées
> - Avec des **contrôles d'accès** gérés centralement

> [!important] Authentification et Autorisation
> Active Directory gère :
> - L'**authentification** des utilisateurs et ordinateurs
> - Le **contrôle d'accès** aux ressources réseau
> - Basé sur les **politiques de sécurité** définies

> [!important] Réplication des Données
> - Les informations du domaine sont **répliquées** entre les contrôleurs de domaine
> - Assure la **cohérence** des données
> - Garantit la **disponibilité** de l'annuaire dans tout le domaine

> [!important] Structure Hiérarchique
> Un domaine peut faire partie d'une structure plus large :
> - Appelée **forêt AD**
> - Une forêt = collection de plusieurs domaines
> - Relations d'approbation entre domaines

### Structure hiérarchique d'un domaine

> [!example] Exemple de structure en cascade
> ```
> Domaine entreprise
> ├── OU Utilisateurs
> │   ├── OU Nantes
> │   ├── OU Bordeaux
> │   ├── OU Lyon
> │   └── OU Paris
> ├── OU Ordinateurs
> │   ├── OU PC bureau
> │   ├── OU PC portable
> │   └── OU Serveurs
> └── OU Groupes
> ```

> [!example] Exemple PowerShell - Informations sur le domaine
> ```powershell
> PS C:\Lab> Get-ADDomain | Select-Object DomainControllersContainer,DomainMode,DomainSID,Name | Format-List
> 
> DomainControllersContainer : OU=Domain Controllers,DC=lab,DC=lan
> DomainMode                 : Windows2016Domain
> DomainSID                  : S-1-5-21-3649124935-1597064440-2657112874
> Name                       : lab
> ```

---

## Workgroup vs Domaine

> [!important] Deux modes de gestion différents
> Windows propose deux modes d'organisation des ordinateurs en réseau : **Workgroup** (groupe de travail) et **Domaine**. Chaque mode a ses caractéristiques et ses cas d'usage.

### Comparaison détaillée

| Critère | Workgroup | Domaine |
|---------|-----------|---------|
| **Réseau** | Tous les ordinateurs doivent être sur le **même réseau local** | Les ordinateurs peuvent être sur des **réseaux différents** |
| **Rôle des ordinateurs** | Tous les ordinateurs ont le **même rôle** standard | Un ou plusieurs ordinateurs sont des **serveurs (DC)** |
| **Connexion** | On peut se connecter **uniquement où un compte local** a été créé | On peut se connecter **n'importe où** avec un compte de domaine |
| **Limite** | Limite fonctionnelle à **quelques dizaines de machines** | **Pas de limite** au nombre de machines |
| **Centralisation** | **Aucune centralisation** (administration locale) | **Centralisation** avec les contrôleurs de domaine |

> [!tip] Quand utiliser un Workgroup ?
> - Petits réseaux domestiques ou SOHO (Small Office/Home Office)
> - Moins de 10-20 ordinateurs
> - Pas de besoin de gestion centralisée
> - Budget limité (pas besoin de serveur)

> [!tip] Quand utiliser un Domaine ?
> - Réseaux d'entreprise de toute taille
> - Besoin de gestion centralisée
> - Nécessité de politiques de sécurité uniformes
> - Mobilité des utilisateurs (connexion depuis n'importe quel poste)
> - Plus de 20 ordinateurs à gérer

---

## Arbre

> [!quote] Définition
> Un **arbre** est une arborescence de domaines qui partagent un schéma et une configuration communs, formant un espace de noms contigu.

### Caractéristiques d'un arbre

> [!info] Structure d'un arbre AD
> - Constitué de **plusieurs domaines** liés entre eux
> - Partage un **schéma commun** et une **configuration commune**
> - Forme un **espace de noms contigu** (continuité DNS)
> - Les domaines sont liés par des **relations d'approbation** (trust)

### Deux visions possibles

Un arbre peut être visualisé de deux façons :
1. **Vue par les relations d'approbation** entre les domaines
2. **Vue par l'espace de noms** de l'arborescence de domaine

> [!example] Exemple de structure en arbre
> ```
>                 Domaine A (racine)
>                      │
>         ┌────────────┼────────────┐
>         │                         │
>    Domaine B                 Domaine C
>    ├── OU                    ├── OU
>    └── OU                    └── OU
> ```

---

## Forêt

> [!quote] Définition
> Une **forêt** est une structure hiérarchique de plusieurs domaines indépendants. C'est le conteneur de sécurité et de réplication le plus élevé dans Active Directory.

### Caractéristiques d'une forêt AD

> [!important] Structure d'une forêt
> Dans une forêt Active Directory :
> 
> - **Tous les arbres** partagent un **schéma d'annuaire commun**
> - **Tous les domaines** :
>   - Partagent un **Catalogue Global** commun
>   - Fonctionnent de façon **indépendante**
>   - Ont des **relations possibles** entre eux (approbations)

### Représentation d'une forêt

> [!example] Exemple de forêt avec plusieurs arbres
> ```
> Forêt
> ├── Arbre 1
> │   ├── Domaine (racine)
> │   │   ├── OU
> │   │   └── OU
> │   └── Sous-domaine
> └── Arbre 2
>     └── Domaine (racine)
>         ├── OU
>         └── OU
> ```

### Avantages d'une forêt

> [!success] Pourquoi utiliser une forêt ?
> Les forêts offrent :
> - **Simplification de l'administration**
> - **Flexibilité** organisationnelle
> - **Authentification inter-domaines** automatique
> - **Partage de ressources** entre domaines

> [!example] Exemple concret d'utilisation
> ```
> masociete.fr (forêt racine)
> ├── paris.masociete.fr
> ├── lyon.masociete.fr
> └── mafilliale.com (arbre séparé)
>     ├── nantes.mafilliale.com
>     │   └── it.nantes.mafilliale.com
>     └── berlin.mafilliale.com
> ```
> 
> **Cas d'usage** :
> Un utilisateur du domaine `lyon.masociete.fr` pourra :
> - Accéder à des ressources situées dans le domaine `it.nantes.mafilliale.com`
> - Se connecter sur une machine du domaine `berlin.mafilliale.com`
> - (Avec les autorisations appropriées)

> [!example] Exemple PowerShell - Informations sur la forêt
> ```powershell
> PS C:\Lab> Get-ADForest
> 
> ApplicationPartitions  : {DC=ForestDnsZones,DC=lab,DC=lan, 
>                           DC=DomainDnsZones,DC=lab,DC=lan}
> CrossForestReferences  : {}
> DomainNamingMaster     : AD1.lab.lan
> Domains                : {lab.lan}
> ForestMode             : Windows2016Forest
> GlobalCatalogs         : {AD1.lab.lan}
> Name                   : lab.lan
> PartitionsContainer    : CN=Partitions,CN=Configuration,DC=lab,DC=lan
> RootDomain             : lab.lan
> SchemaMaster           : AD1.lab.lan
> Sites                  : {Default-First-Site-Name}
> SPNSuffixes            : {}
> UPNSuffixes            : {}
> ```

---

## Composants AD

> [!abstract] Les composants essentiels
> Active Directory repose sur plusieurs composants techniques clés qui assurent son fonctionnement et sa disponibilité.

---

## Contrôleur de domaine (DC)

> [!quote] Définition
> Un **contrôleur de domaine**, ou **DC** (Domain Controller), est un serveur crucial pour un domaine Active Directory.

### Caractéristiques du DC

> [!important] Rôle du contrôleur de domaine
> - **Tous les domaines** ont au moins un DC
> - Un DC est **indispensable** au bon fonctionnement du domaine
> - Si le DC est **éteint ou corrompu** → Le domaine devient **inutilisable**
> 
> **Le DC est un serveur maître** qui :
> - Héberge la **base de données Active Directory**
> - Gère l'**authentification** des utilisateurs
> - Applique les **stratégies de groupe** (GPO)
> - Réplique les données avec les autres DC

> [!warning] Point critique
> Le contrôleur de domaine est un **SPOF** (Single Point of Failure) potentiel. C'est pourquoi on déploie généralement **plusieurs DC** pour assurer la **redondance** et la **haute disponibilité**.

> [!example] Exemple PowerShell - Informations sur un DC
> ```powershell
> PS C:\Lab> Get-ADDomainController
> 
> ComputerObjectDN           : CN=AD1,OU=Domain Controllers,DC=lab,DC=lan
> DefaultPartition           : DC=lab,DC=lan
> Domain                     : lab.lan
> Enabled                    : True
> Forest                     : lab.lan
> HostName                   : AD1.lab.lan
> IPv4Address                : 10.10.1.2
> IsGlobalCatalog            : True
> IsReadOnly                 : False
> LdapPort                   : 389
> Name                       : AD1
> OperatingSystem            : Windows Server 2022 Standard Evaluation
> OperatingSystemVersion     : 10.0 (20348)
> OperationMasterRoles       : {SchemaMaster, DomainNamingMaster, 
>                               PDCEmulator, RIDMaster...}
> ServerObjectDN             : CN=AD1,CN=Servers,CN=Default-First-Site-Name,
>                              CN=Sites,CN=Configuration,DC=lab,DC=lan
> Site                       : Default-First-Site-Name
> SslPort                    : 636
> ```

---

## RODC (Read Only Domain Controller)

> [!quote] Définition
> Un **RODC** (Read Only Domain Controller) est un contrôleur de domaine avec des droits en **lecture seule**.

### Caractéristiques du RODC

> [!info] Spécificités du RODC
> - Serveur ayant le rôle de **DC en lecture seule**
> - Possède uniquement des **droits de lecture** sur l'annuaire
> - **Aucune modification** ne peut être effectuée directement
> - Souvent utilisé pour les **sites distants** ou **peu sécurisés**

### Pourquoi utiliser un RODC ?

> [!success] Avantages du RODC
> 
> **1. Équilibrage de la charge** :
> - Répartition des requêtes d'authentification
> - Amélioration des performances sur les sites distants
> 
> **2. Continuité de service** :
> - Le serveur AD principal transmet les **modifications** au RODC du site distant
> - Transmission également des modifications du **DNS**
> - Les utilisateurs du site distant disposent d'un service de résolution de nom **local**
> - Maintien de l'accès à Internet même en cas de **coupure de liaison** avec le DC principal
> 
> **3. Sécurité renforcée** :
> - En cas de vol ou compromission du serveur distant
> - Pas de mots de passe en cache (sauf configuration spécifique)
> - Impossibilité de modifier l'AD depuis le RODC

> [!tip] Cas d'usage typique
> - **Agences distantes** avec connexion WAN limitée
> - **Sites peu sécurisés** physiquement
> - Lieux où l'**expertise IT locale** est limitée
> - Besoin d'**authentification locale** sans risque de compromission de l'AD principal

---

## Catalogue Global

> [!quote] Définition
> Le **catalogue global AD** est une version spéciale d'un contrôleur de domaine qui contient des informations étendues sur l'ensemble de la forêt AD, en plus des données de son propre domaine.

### Fonctionnalités du catalogue global

> [!important] Contrôleur de Domaine Étendu
> Le serveur catalogue global est un **DC avec des capacités étendues** :
> - Contient sa **propre base de données** de domaine (complète)
> - Plus des **informations partielles** sur tous les autres domaines de la forêt
> - Permet de **localiser des objets** dans toute la forêt
> - Les autres DC s'appuient sur lui pour les recherches inter-domaines

> [!important] Répliqua Partiel pour tous les Domaines
> **Fonctionnement de la réplication** :
> - Possède un **répliqua partiel** de tous les attributs de tous les domaines de la forêt
> - Dispose d'**informations globales** sur la forêt entière
> - Contient les attributs les plus **fréquemment recherchés** (pas tous les attributs)
> - Optimise les **recherches inter-domaines**

### Qui est Catalogue Global ?

> [!note] Configuration du Catalogue Global
> - Le **premier DC créé** au sein d'une forêt est **automatiquement catalogue global**
> - À l'installation d'un AD dans une nouvelle forêt → le DC sera catalogue global
> - Il est possible de configurer **d'autres DC** en tant que serveur de catalogue global
> - Permet de **réguler le trafic** et d'améliorer les performances

> [!tip] Bonnes pratiques
> - Configurer **au moins 2 catalogues globaux** par site (redondance)
> - Placer un catalogue global dans **chaque site géographique** important
> - Éviter de désactiver le catalogue global sur le **seul DC d'un domaine**

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **Active Directory** est la mise en œuvre Microsoft des services d'annuaire **LDAP**
- AD utilise une **base de données hiérarchique** (différente d'une BDD relationnelle)
- Le protocole **LDAP** est standardisé et permet l'interopérabilité avec d'autres systèmes
- AD a remplacé la base **SAM** pour offrir une **évolutivité** et des fonctionnalités avancées

### Structure logique de l'arborescence AD

- **Objet AD** : Élément de base représentant une ressource (utilisateur, ordinateur, groupe, etc.)
- **OU (Unité d'Organisation)** : Conteneur permettant l'organisation logique et l'application de GPO
- **Domaine** : Unité administrative et de sécurité, périmètre de gestion centralisée
- **Arbre** : Ensemble de domaines partageant un schéma commun et un espace de noms contigu
- **Forêt** : Structure hiérarchique de plusieurs arbres avec schéma et catalogue global communs

### Différences Workgroup vs Domaine

| Critère | Workgroup | Domaine |
|---------|-----------|---------|
| **Centralisation** | Aucune | Totale (via DC) |
| **Nombre de machines** | Quelques dizaines | Illimité |
| **Connexion** | Locale uniquement | Depuis n'importe quel poste |
| **Réseau** | Même réseau local | Réseaux différents possibles |

### Composants techniques

- **DC (Domain Controller)** : Serveur hébergeant la BDD AD, gère authentification et réplication
- **RODC (Read Only DC)** : DC en lecture seule pour sites distants ou peu sécurisés
- **Catalogue Global** : DC étendu contenant des infos partielles sur toute la forêt

### Les 5 rôles Active Directory

1. **AD DS** : Rôle principal - gestion domaine, utilisateurs, GPO
2. **AD CS** : Gestion des certificats et PKI
3. **AD FS** : Single Sign-On (SSO) pour applications
4. **AD RMS** : Gestion des droits numériques sur fichiers
5. **AD LDS** : Annuaire LDAP léger sans domaine

### Chiffres importants à retenir

- **Plus de 2 milliards** d'objets maximum dans AD
- **999** GPO maximum par objet
- **1015** appartenances de groupe maximum par objet
- **25%** du marché de gestion d'identité et d'accès

### Commandes PowerShell essentielles

```powershell
# Objets AD
Get-ADObject -Filter {Name -like "*pattern*"}

# Unité d'Organisation
Get-ADOrganizationalUnit -Filter {Name -like "*pattern*"}

# Domaine
Get-ADDomain

# Forêt
Get-ADForest

# Contrôleur de domaine
Get-ADDomainController
```

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Active Directory (AD)** | Mise en œuvre Microsoft des services d'annuaire LDAP pour les systèmes Windows, permettant la gestion centralisée des ressources réseau |
| **AD DS** | Active Directory Domain Services - Rôle principal permettant la gestion d'un domaine et d'un annuaire |
| **Annuaire LDAP** | Base de données hiérarchique structurée pour stocker et organiser des informations sur les ressources réseau |
| **Arbre** | Ensemble de domaines partageant un schéma commun et formant un espace de noms contigu |
| **Attribut** | Caractéristique d'un objet AD (nom, prénom, adresse email, etc.) |
| **Base SAM** | Security Account Manager - Ancienne base de comptes Windows, remplacée par AD pour les environnements domaine |
| **Catalogue Global** | Contrôleur de domaine contenant des informations partielles sur tous les objets de tous les domaines de la forêt |
| **DC (Domain Controller)** | Contrôleur de domaine - Serveur hébergeant la base de données Active Directory et gérant l'authentification |
| **Distinguished Name (DN)** | Chemin complet unique d'un objet dans l'arborescence LDAP (ex: CN=User,OU=Users,DC=lab,DC=lan) |
| **Domaine** | Unité administrative et de sécurité regroupant des ressources réseau gérées centralement |
| **ESENT** | Extensible Storage Engine - Moteur de base de données utilisé par AD (remplace Jet) |
| **Forêt** | Structure hiérarchique de plus haut niveau contenant un ou plusieurs arbres de domaines |
| **GPO (Group Policy Object)** | Stratégie de groupe - Ensemble de règles appliquées aux utilisateurs et ordinateurs |
| **LDAP** | Lightweight Directory Access Protocol - Protocole standardisé pour accéder aux services d'annuaire |
| **NTDS** | NT Directory Services - Nom initial d'Active Directory en 1996 |
| **Objet AD** | Élément de base de la BDD Active Directory représentant une ressource (utilisateur, ordinateur, groupe, etc.) |
| **ObjectGUID** | Identifiant unique global d'un objet AD, immuable et universel |
| **OU (Organizational Unit)** | Unité d'Organisation - Conteneur permettant l'organisation logique des objets et l'application de GPO |
| **RODC** | Read Only Domain Controller - Contrôleur de domaine en lecture seule pour sites distants |
| **Schéma AD** | Définition de tous les types d'objets et attributs possibles dans l'annuaire |
| **SGBD** | Système de Gestion de Base de Données - Base de données relationnelle classique (différent de LDAP) |
| **SSO** | Single Sign-On - Authentification unique permettant l'accès à plusieurs applications |
| **Workgroup** | Groupe de travail - Mode de réseau pair-à-pair sans contrôleur de domaine ni gestion centralisée |

---

## Conclusion

> [!success] Ce que tu dois maîtriser
> Après l'étude de cette partie 1 sur Active Directory, tu dois être capable de :
> 
> ✅ **Définir** ce qu'est un annuaire LDAP et une base de données hiérarchique
> 
> ✅ **Connaître** les éléments de la structure logique de l'arborescence AD :
>    - Objet
>    - OU (Unité d'Organisation)
>    - Domaine
>    - Arbre
>    - Forêt
> 
> ✅ **Expliquer** les différences entre un domaine et un workgroup
> 
> ✅ **Identifier** les composants techniques :
>    - DC (Domain Controller)
>    - RODC (Read Only Domain Controller)
>    - Catalogue Global
> 
> ✅ **Comprendre** les 5 rôles Active Directory (DS, CS, FS, RMS, LDS)

---

## Liens et ressources

> [!info] Aucun lien externe n'était présent dans le document PowerPoint source
> Ce document de révision a été créé exclusivement à partir du contenu des slides du cours.

---

**Document créé le** : Janvier 2026  
**Format** : Markdown pour Obsidian  
**Version** : 1.0  
**Auteur** : Support de révision TSSR
