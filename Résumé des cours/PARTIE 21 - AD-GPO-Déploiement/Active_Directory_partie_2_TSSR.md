# Active Directory - partie 2

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Service d'annuaire - Active Directory (partie 2)

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Protocoles réseaux associés|Protocoles réseaux associés]]
   - [[#DNS|DNS]]
   - [[#SNTP (Simple Network Time Protocol)|SNTP]]
   - [[#LDAP|LDAP]]
   - [[#LDIF|LDIF]]
   - [[#Kerberos|Kerberos]]
   - [[#X509|X509]]
   - [[#NTFS|NTFS]]
2. [[#Fonctionnalités avancées|Fonctionnalités avancées]]
   - [[#Niveau fonctionnel|Niveau fonctionnel]]
   - [[#Schéma AD|Schéma AD]]
   - [[#Réplication|Réplication]]
   - [[#Les 5 rôles FSMO|Les 5 rôles FSMO]]
3. [[#Les objets AD|Les objets AD]]
   - [[#Définition des objets|Définition des objets]]
   - [[#Les attributs|Les attributs]]
   - [[#Les classes d'objets|Les classes d'objets]]
   - [[#Les identifiants uniques|Les identifiants uniques]]
4. [[#Bonnes pratiques d'administration|Bonnes pratiques d'administration]]
   - [[#Gestion des identités et accès|Gestion des identités et accès]]
   - [[#Microsoft Tiering Model|Microsoft Tiering Model]]
   - [[#JIT & JEA|JIT & JEA]]
   - [[#AGDLP|AGDLP]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]
7. [[#Liens et ressources|Liens et ressources]]

---

## Protocoles réseaux associés

> [!abstract] Vue d'ensemble
> Active Directory s'appuie sur un ensemble de protocoles réseaux interconnectés pour fonctionner correctement. Ces protocoles assurent la résolution de noms, la synchronisation du temps, l'authentification, la sécurité et le partage de ressources.

> [!info] Protocoles principaux liés à AD
> Active Directory utilise plusieurs protocoles qui travaillent ensemble :
> - **DNS** : Résolution de noms et services
> - **SNTP** : Synchronisation temporelle
> - **LDAP/LDIF** : Accès et modification de l'annuaire
> - **Kerberos** : Authentification sécurisée
> - **X509** : Certificats numériques
> - **NTFS** : Permissions et contrôle d'accès
> - **SMB/CIFS** : Partage de fichiers

---

## DNS

> [!important] Service obligatoire pour Active Directory
> Le **DNS** (Domain Name System) est un service **absolument nécessaire** pour l'utilisation d'Active Directory. Sans DNS fonctionnel, l'AD ne peut pas opérer correctement.

### Utilisation du DNS dans AD

> [!info] Double rôle du DNS
> Le DNS dans Active Directory est utilisé pour :
> 
> 1. **Résolution des noms** :
>    - Conversion des noms d'hôtes en adresses IP
>    - Exemple : `serveur1.lab.lan` → `10.10.1.5`
> 
> 2. **Résolution des services** (SRV records) :
>    - Localisation des contrôleurs de domaine
>    - Recherche des services LDAP, Kerberos, etc.
>    - Exemple : `_ldap._tcp.lab.lan`

> [!warning] Attention aux problèmes DNS !
> Les problèmes de DNS peuvent gravement affecter le fonctionnement d'Active Directory :
> - Impossibilité d'authentification
> - Échecs de réplication entre DC
> - Services AD indisponibles
> - Clients incapables de localiser les contrôleurs de domaine
> 
> **⚠️ Un DNS défaillant = Un AD défaillant**

> [!tip] Bonne pratique
> - Configurer au moins **2 serveurs DNS** pour la redondance
> - Utiliser les DC comme serveurs DNS
> - Vérifier régulièrement les enregistrements SRV dans les zones DNS

---

## SNTP (Simple Network Time Protocol)

> [!quote] Définition
> **SNTP** (Simple Network Time Protocol) est le protocole utilisé pour la synchronisation des horloges des systèmes dans un réseau Active Directory.

### Fonctionnement

> [!info] Synchronisation temporelle
> - Stocke l'heure en **UTC** (Temps Universel Coordonné)
> - Affiche l'heure locale en tenant compte du **fuseau horaire**
> - L'**émulateur PDC** fait office de source de temps pour le domaine
> - Les autres DC se synchronisent sur l'émulateur PDC
> - Les clients se synchronisent sur les DC

### Importance cruciale pour AD

> [!important] SNTP est impératif pour Kerberos
> Une bonne synchronisation temporelle est **absolument nécessaire** pour le protocole d'authentification **Kerberos** utilisé par Windows.

### Problèmes causés par une mauvaise synchronisation

> [!warning] Conséquences d'un désynchronisation temporelle
> Une mauvaise synchronisation du temps entraîne de nombreux problèmes :
> 
> - **Problèmes d'authentification Kerberos** :
>   - Tickets Kerberos refusés (écart de temps > 5 minutes par défaut)
>   - Impossibilité de se connecter au domaine
> 
> - **Problèmes de réplication** :
>   - Échecs de réplication entre contrôleurs de domaine
>   - Incohérences dans la base de données AD
> 
> - **Échecs de connexion aux ressources** :
>   - Accès refusé aux partages réseau
>   - Services indisponibles
> 
> - **Échecs de communication sécurisée** :
>   - Problèmes avec les certificats (validité temporelle)
>   - Échecs SSL/TLS
> 
> - **Problèmes de journalisation et d'audit** :
>   - Horodatage incorrect des événements
>   - Traçabilité compromise

> [!tip] Tolérance Kerberos
> Par défaut, Kerberos tolère un décalage de **5 minutes** maximum entre le client et le serveur. Au-delà, l'authentification échoue.

---

## LDAP

> [!quote] Le protocole fondamental
> **LDAP** (Lightweight Directory Access Protocol) est la **colonne vertébrale** d'Active Directory et constitue le **standard des services d'annuaire**.

### Opérations LDAP de base

> [!info] Commandes LDAP essentielles
> LDAP permet 5 opérations de base sur l'annuaire :
> 
> | Opération | Description |
> |-----------|-------------|
> | **bind** | Connexion et authentification à l'annuaire |
> | **search** | Recherche d'objets selon des critères |
> | **add** | Ajout d'un nouvel objet dans l'annuaire |
> | **delete** | Suppression d'un objet de l'annuaire |
> | **modify** | Modification des attributs d'un objet existant |

### Ports LDAP

> [!note] Ports de communication
> - **Port 389** : LDAP standard (non chiffré)
> - **Port 636** : LDAPS (LDAP sur SSL/TLS - chiffré)

> [!tip] Toujours privilégier LDAPS
> Pour sécuriser les communications avec l'annuaire, utilise **LDAPS** (port 636) plutôt que LDAP non chiffré.

---

## LDIF

> [!quote] Définition
> Les fichiers **LDIF** (LDAP Data Interchange Format) sont des **fichiers textes** permettant d'interagir avec Active Directory de manière structurée.

### Utilité du format LDIF

> [!info] Fonctionnalités LDIF
> Le format LDIF permet :
> - **Importations** de données dans AD
> - **Exportations** de données depuis AD
> - **Modifications** en masse de l'annuaire
> - **Chargement** d'AD à partir d'une base de données externe
> - **Synchronisation** avec des systèmes tiers

> [!example] Cas d'usage typique
> **Gestion des comptes utilisateurs à partir de la BDD RH** :
> 
> 1. Export de la base de données RH au format CSV
> 2. Conversion en format LDIF
> 3. Import automatique dans Active Directory
> 4. Création ou mise à jour des comptes utilisateurs
> 
> → Automatisation de la création/modification des comptes employés

> [!example] Exemple de fichier LDIF
> ```ldif
> dn: CN=John Doe,OU=Users,DC=lab,DC=lan
> changetype: add
> objectClass: user
> cn: John Doe
> sAMAccountName: jdoe
> givenName: John
> sn: Doe
> userPrincipalName: jdoe@lab.lan
> mail: john.doe@lab.lan
> ```

---

## Kerberos

> [!quote] Définition
> **Kerberos** est le protocole d'authentification central par défaut dans Windows depuis **Windows 2000**. Il remplace les anciens protocoles LM/NTLM utilisés jusqu'à Windows NT4.

### Caractéristiques de Kerberos

> [!important] Sécurité renforcée
> Kerberos offre plusieurs avantages en termes de sécurité :
> 
> - **Compatible Kerberos V5** (standard ouvert)
> - **Authentification mutuelle** :
>   - Le client authentifie le serveur
>   - Le serveur authentifie le client
>   - Protection contre les attaques "man-in-the-middle"
> 
> - **Chiffrement fort** des échanges
> - **Tickets à durée de vie limitée** (10 heures par défaut)

### Intégration avec DNS

> [!info] Localisation du serveur Kerberos
> L'adresse du **serveur Kerberos** utilisé pour l'ouverture de session est extraite du **DNS** via les enregistrements SRV.
> 
> Exemple d'enregistrement :
> ```
> _kerberos._tcp.lab.lan
> ```

### Processus d'authentification Kerberos (simplifié)

> [!note] Étapes d'authentification
> 1. Le client demande un **TGT** (Ticket Granting Ticket) au KDC
> 2. Le KDC vérifie l'identité et retourne le TGT chiffré
> 3. Le client présente son TGT pour obtenir un ticket de service
> 4. Le client utilise ce ticket pour accéder à la ressource
> 5. Le serveur valide le ticket et accorde l'accès

> [!warning] Dépendance temporelle
> Kerberos dépend fortement de la **synchronisation temporelle** (SNTP). Un décalage de plus de 5 minutes entraîne l'échec de l'authentification.

---

## X509

> [!quote] Définition
> **X.509** est une norme internationale définissant le format des certificats numériques utilisés pour l'authentification et le chiffrement.

### Services de certificats Windows

> [!info] AD CS (Active Directory Certificate Services)
> Windows Server propose les **services de certificats** compatibles X.509 via le rôle **AD CS**.

### Avantages des certificats X.509

> [!success] Renforcement de la sécurité
> Les certificats X.509 offrent :
> 
> - **Authentification** : Vérification d'identité
> - **Intégrité** : Garantie de non-modification des données
> - **Confidentialité** : Chiffrement des communications
> - **Non-répudiation** : Preuve de l'origine des actions

### Utilisation dans les services Windows

> [!example] Services utilisant les certificats
> Les certificats X.509 sont utilisables par de nombreux services :
> 
> - **Authentification par carte à puce** :
>   - Connexion physique sécurisée
>   - Double facteur d'authentification
> 
> - **EFS (Encrypting File System)** :
>   - Chiffrement de fichiers sur NTFS
>   - Protection des données sensibles
> 
> - **IPsec** :
>   - Chiffrement des données sur le réseau
>   - VPN sécurisés
>   - Communications inter-sites
> 
> - **HTTPS/SSL/TLS** :
>   - Sécurisation des sites web
>   - Protection des échanges applicatifs

---

## NTFS

> [!quote] Système de fichiers et sécurité
> **NTFS** (New Technology File System) est le système de fichiers Windows qui intègre la **gestion des droits** et des **contrôles d'accès** en collaboration avec Active Directory.

### Fonctionnalités liées à AD

> [!info] Gestion des permissions
> NTFS permet :
> - **Gestion des droits** d'accès aux fichiers et dossiers
> - **Contrôles d'accès** basés sur les identités AD
> - **Gestion des permissions par groupes AD** :
>   - Utilisation des groupes de sécurité
>   - Application de la méthode AGDLP
> - **Héritage des permissions** dans l'arborescence
> - **Audit des accès** aux ressources

### Types de permissions NTFS

> [!note] Permissions de base
> | Permission | Description |
> |------------|-------------|
> | **Lecture (R)** | Lire le contenu des fichiers/dossiers |
> | **Écriture (W)** | Créer et modifier des fichiers |
> | **Modification (M)** | Lecture + Écriture + Suppression |
> | **Contrôle total (F)** | Toutes les permissions + modification des ACL |

> [!tip] Bonne pratique avec les groupes AD
> Ne **jamais** attribuer les permissions NTFS directement aux utilisateurs. Utilise toujours des **groupes de sécurité AD** et applique la méthode **AGDLP**.

---

## Fonctionnalités avancées

> [!abstract] Fonctionnalités de gestion avancées
> Active Directory propose plusieurs fonctionnalités avancées qui permettent une gestion fine et évolutive de l'infrastructure : niveau fonctionnel, schéma, réplication et rôles FSMO.

---

## Niveau fonctionnel

> [!quote] Définition
> Le **niveau fonctionnel** (Functional Level) définit les fonctionnalités Active Directory disponibles dans un domaine ou une forêt. Il dépend de la version de Windows Server utilisée.

### Principe du niveau fonctionnel

> [!important] Règle de base
> - À la création d'un domaine, le niveau fonctionnel correspond à la **version de l'OS serveur** depuis lequel on crée le domaine
> - Un domaine créé sous Server 2016 aura au maximum un niveau fonctionnel Server 2016
> - Le **niveau fonctionnel de la forêt** = niveau fonctionnel **minimum** des domaines qui la composent

### Évolution du niveau fonctionnel

> [!example] Exemple d'évolution
> **Situation initiale** :
> - 5 DC sous Windows Server 2012 R2
> - → Niveau fonctionnel = **Windows Server 2012 R2**
> 
> **Après ajout de nouveaux DC** :
> - 3 DC sous Windows Server 2012 R2
> - 2 DC sous Windows Server 2019
> - → Niveau fonctionnel = **Windows Server 2012 R2** (OS le plus ancien)

> [!info] Pourquoi faire une montée de niveau fonctionnel ?
> Une montée de niveau fonctionnel permet de :
> - **Avoir les dernières fonctionnalités** AD disponibles
> - **Prendre en charge les derniers OS** (client et serveur)
> - **Améliorer la sécurité** avec de nouvelles fonctionnalités
> - **Optimiser les performances** de l'annuaire

> [!warning] Opération irréversible !
> ⚠️ **Impossible de faire machine arrière** sur une montée de niveau fonctionnel !
> 
> Avant de monter le niveau fonctionnel :
> 1. Vérifier que **tous les DC** sont à jour
> 2. S'assurer qu'aucun retour en arrière ne sera nécessaire
> 3. Effectuer une **sauvegarde complète** de l'AD
> 4. Tester dans un environnement de laboratoire

> [!tip] Planification de montée de niveau
> **Processus recommandé** :
> 1. Mettre à jour tous les DC vers la même version Windows Server
> 2. Attendre quelques semaines pour vérifier la stabilité
> 3. Monter le niveau fonctionnel du domaine
> 4. Monter le niveau fonctionnel de la forêt

---

## Schéma AD

> [!quote] Définition
> Le **schéma** Active Directory est la définition de tous les types d'objets et de tous les attributs qui peuvent exister dans l'annuaire. C'est l'équivalent de la structure des tables et champs dans une base de données.

### Rôle du schéma

> [!info] Structure de l'annuaire
> Le schéma définit :
> - Les **classes d'objets** possibles (user, computer, group, etc.)
> - Les **attributs** de chaque classe (name, email, phone, etc.)
> - Les **relations** entre les classes
> - Les **règles de validation** des données

> [!example] Exemple pour un utilisateur
> Pour un objet de type **utilisateur**, les attributs peuvent être :
> 
> **Identifiants** :
> - SID (Security Identifier)
> - GUID (Globally Unique Identifier)
> - sAMAccountName
> - userPrincipalName
> 
> **Classe** :
> - user
> 
> **Informations système** :
> - lastLogon
> - lastLogonTimestamp
> - pwdLastSet
> - accountExpires
> 
> **Informations personnelles** :
> - givenName (prénom)
> - sn (surname - nom)
> - displayName
> - mail
> - telephoneNumber

### Attributs liés au schéma

> [!note] Relations entre objets
> Les **attributs liés au schéma** permettent d'établir des liens entre objets :
> - Appartenance aux groupes (`memberOf`)
> - Gestion des relations hiérarchiques
> - Liens entre ressources et utilisateurs

> [!warning] Modification du schéma
> Modifier le schéma AD est une opération **critique** :
> - Nécessite le rôle **Schema Master** (FSMO)
> - Impact sur **toute la forêt**
> - Opération **irréversible** dans la plupart des cas
> - Nécessite une **planification rigoureuse**

---

## Réplication

> [!quote] Définition
> La **réplication** est le processus par lequel Active Directory synchronise les données entre tous les contrôleurs de domaine pour assurer la **redondance** et la **cohérence** des informations.

### Objectif de la réplication

> [!important] Redondance et cohérence
> La réplication amène la **redondance de données** entre DC. Ces données doivent être **identiques** sur les différents contrôleurs de domaine pour garantir un service continu et fiable.

### Données répliquées

> [!info] Éléments synchronisés
> La réplication gère la synchronisation de :
> - La **base d'annuaire AD** (NTDS.dit)
> - Les **GPO** (Group Policy Objects) via SYSVOL
> - Les **scripts** de connexion et GPO
> - Les zones **DNS** intégrées à AD

### Processus de réplication

> [!example] Étapes du processus de réplication
> Suite à une modification d'objet sur **DC1** :
> 
> 1. **DC1** demande au **KCC** s'il y a d'autres DC dans la topologie
> 2. Le **KCC** indique qu'il existe un **DC2**
> 3. **DC1** demande au **DNS** où se trouve le **DC2**
> 4. Le **DNS** envoie les informations de localisation de **DC2**
> 5. **DC1** indique au **DC2** qu'il a des modifications à répliquer
> 6. **DC2** demande à **DC1** quelles sont ces modifications
> 7. **DC1** envoie les modifications à **DC2**
> 8. **DC2** met à jour sa base de données

> [!note] Schéma du processus de réplication
> ```
> DC1                          DNS                          DC2
>  │                            │                            │
>  │──1. Autres DC ?──>         │                            │
>  │<─2. Oui, DC2──────         │                            │
>  │──3. Où est DC2 ?────────>  │                            │
>  │<─4. Info DC2──────────────  │                            │
>  │──5. J'ai des modifs─────────────────────────────────>   │
>  │<─6. Transmets-les──────────────────────────────────────  │
>  │──7. Les voici──────────────────────────────────────>     │
>  │                            │                  8. MAJ BDD │
> ```

---

## Le KCC (Knowledge Consistency Checker)

> [!quote] Définition
> Le **KCC** (Knowledge Consistency Checker) est un processus présent sur tous les contrôleurs de domaine qui génère et maintient automatiquement la **topologie de réplication**.

### Rôle du KCC

> [!info] Gestionnaire de topologie
> Le KCC s'occupe de :
> - **Générer** automatiquement une topologie de réplication optimale
> - **Maintenir** cette topologie à jour
> - **Régénérer** une nouvelle topologie en cas de changement :
>   - Ajout d'un nouveau DC
>   - Suppression d'un DC
>   - Déplacement d'un DC vers un autre site

### Fonctionnement

> [!note] Topologie automatique
> - Le KCC fonctionne en **intra-site** (au sein d'un site AD)
> - Le KCC fonctionne en **inter-site** (entre sites AD différents)
> - Création automatique d'une **topologie en anneau** (ring topology) pour la redondance
> - Chaque DC a au moins **2 partenaires de réplication** (entrante et sortante)

> [!example] Exemple de topologie générée par KCC
> ```
>       DC1
>      /   \
>    DC5   DC2
>     |     |
>    DC4   DC3
>      \   /
>       ---
> ```

---

## Intervalle de réplication

> [!info] Fréquence de synchronisation
> L'intervalle de réplication définit la fréquence à laquelle les contrôleurs de domaine se synchronisent :
> 
> **En intra-site** (au sein d'un même site) :
> - Intervalle par défaut : **5 minutes**
> - Exception : Actions liées à la sécurité → Réplication **immédiate**
> 
> **En inter-site** (entre sites différents) :
> - Intervalle par défaut : **180 minutes** (3 heures)
> - Minimum configurable : **15 minutes**

### Optimisation

> [!tip] Équilibre latence vs bande passante
> - Un **petit intervalle** réduit la **latence** (données plus récentes)
> - Mais augmente la **quantité de trafic réseau**
> 
> **Recommandations** :
> - Pour les **partitions d'annuaire de domaine** : privilégier une **faible latence**
> - Pour les **liaisons WAN limitées** : augmenter l'intervalle pour réduire le trafic
> - Activer la **compression de réplication** sur les liens inter-sites lents

> [!warning] Réplication immédiate
> Certaines modifications déclenchent une **réplication immédiate** (sans attendre l'intervalle) :
> - Modification de mot de passe utilisateur
> - Verrouillage de compte
> - Modification de stratégie de sécurité critique

---

## Les 5 rôles FSMO

> [!quote] Définition
> **FSMO** signifie **Flexible Single Master Operation**. Ce sont des rôles spécifiques attribués à certains contrôleurs de domaine pour gérer des opérations qui nécessitent un maître unique (pas de multi-maître possible).

> [!important] Pourquoi des rôles FSMO ?
> Active Directory utilise normalement un modèle **multi-maître** (tous les DC peuvent écrire). Cependant, certaines opérations critiques nécessitent un **maître unique** pour éviter les conflits.

### Rôles au niveau de la forêt (2 rôles)

> [!important] Schema Master - Maître de schéma
> **Quantité** : 1 seul par forêt (obligatoire)
> 
> **Rôle** :
> - Gère les **mises à jour du schéma** Active Directory
> - Seul DC autorisé à modifier le schéma
> - Modifications propagées à tous les DC de la forêt
> 
> **Importance** : Critique pour l'évolution de l'infrastructure AD

> [!important] Domain Naming Master - Maître d'attribution de noms de domaine
> **Quantité** : 1 seul par forêt
> 
> **Rôle** :
> - Gère les **noms de domaines** dans la forêt
> - Peut **ajouter** de nouveaux domaines
> - Peut **supprimer** des domaines existants
> - Garantit l'unicité des noms de domaines
> 
> **Importance** : Nécessaire lors de modifications structurelles de la forêt

### Rôles au niveau du domaine (3 rôles)

> [!important] RID Master - Maître RID
> **Quantité** : 1 seul par domaine
> 
> **Rôle** :
> - Gère l'**attribution des RID** (Relative IDentifier)
> - Distribue des **pools de RID** aux autres DC (blocs de 500 par défaut)
> - Compose avec le Domain SID pour créer les **SID uniques**
> 
> **Importance** : Sans lui, impossible de créer de nouveaux objets après épuisement du pool local

> [!important] Infrastructure Master - Maître d'infrastructure
> **Quantité** : 1 seul par domaine
> 
> **Rôle** :
> - Gère les **relations entre objets** de domaines différents
> - Met à jour les **références croisées** entre domaines
> - Synchronise les modifications de noms entre domaines
> 
> **Importance** : Critique dans les environnements multi-domaines

> [!important] PDC Emulator - Émulateur PDC
> **Quantité** : 1 seul par domaine (primordial !)
> 
> **Rôle** :
> - Gère la **synchronisation du temps** pour tout le domaine
> - Traite les **modifications de mots de passe** en priorité
> - Gère le **processus de verrouillage de comptes**
> - Compatible avec les anciens clients NT4
> - Point de contact pour les GPO
> 
> **Importance** : Le plus sollicité des rôles FSMO, critique pour l'authentification

### Récapitulatif des rôles FSMO

| Rôle | Niveau | Quantité | Importance | Fonction principale |
|------|--------|----------|-----------|---------------------|
| **Schema Master** | Forêt | 1 | Critique | Modification du schéma |
| **Domain Naming Master** | Forêt | 1 | Importante | Gestion des domaines |
| **RID Master** | Domaine | 1 par domaine | Critique | Attribution des SID |
| **Infrastructure Master** | Domaine | 1 par domaine | Moyenne | Relations inter-domaines |
| **PDC Emulator** | Domaine | 1 par domaine | Très critique | Temps, mots de passe, GPO |

---

## Bonnes pratiques FSMO

> [!success] Répartition des rôles FSMO
> 
> **À éviter** :
> - ❌ Ne pas avoir qu'un **seul DC avec tous les rôles FSMO**
> - ❌ Par défaut, le premier DC d'une nouvelle forêt cumule les **5 rôles**
> 
> **Recommandations** :
> - ✅ Dès que possible, avoir **au moins 2 DC**
> - ✅ Séparer les rôles par **zone** :
>   - **DC1** : Rôles forêt (Schema Master + Domain Naming Master)
>   - **DC2** : Rôles domaine (RID + Infrastructure + PDC Emulator)
> - ✅ **Idéalement** : 5 DC avec un rôle FSMO installé sur chacun
> - ✅ Configurer des **DC de secours** pour prendre le relais en cas de panne

> [!tip] Placement des rôles
> **PDC Emulator** :
> - Sur un serveur **performant** (le plus sollicité)
> - Sur le **site principal** de l'entreprise
> 
> **Infrastructure Master** :
> - Ne **jamais** le placer sur un serveur Catalogue Global (sauf si tous les DC sont GC)

> [!example] Commandes PowerShell - Identifier les rôles FSMO
> ```powershell
> # Voir tous les rôles FSMO du domaine
> Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
> 
> # Voir tous les rôles FSMO de la forêt
> Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
> 
> # Voir tous les rôles en une seule commande
> netdom query fsmo
> ```

---

## Ne pas confondre !

> [!warning] Attention à la terminologie !
> 
> Ne pas confondre trois types de "rôles" différents dans Windows Server :
> 
> **1. Rôles de serveur Windows Server** :
> - DHCP, DNS, AD DS, AD FS, Hyper-V, IIS, etc.
> - S'installent via "Gestionnaire de serveur"
> 
> **2. Rôles Active Directory** :
> - AD DS, AD CS, AD FS, AD RMS, AD LDS
> - Ce sont en fait des **rôles serveur** spécialisés !
> 
> **3. Rôles FSMO** :
> - Schema Master, Domain Naming Master, RID Master, Infrastructure Master, PDC Emulator
> - Ce sont des **rôles de DC** dans le cadre d'**AD DS**

---

## Les objets AD

> [!abstract] Vue d'ensemble
> Les objets Active Directory sont les éléments de base qui composent l'annuaire. Comprendre leur structure, leurs attributs et leurs identifiants est essentiel pour maîtriser AD.

---

## Définition des objets

> [!quote] Qu'est-ce qu'un objet AD ?
> Chaque objet Active Directory :
> - Est une **instance d'une classe** définie dans le schéma
> - Possède tous les **attributs de sa classe**
> - Représente une ressource, un utilisateur ou un service

> [!example] Exemple : objet utilisateur
> Un objet **utilisateur** existe en tant qu'**instance de la classe "user"** et possède tous les attributs de cette classe (nom, prénom, email, téléphone, etc.).

### Types d'objets

> [!info] Les 3 catégories d'objets AD
> Les objets Active Directory se classent en **3 types** :
> 
> **1. Les ressources** :
> - Postes de travail
> - Serveurs
> - Imprimantes
> - Dossiers partagés
> - Calendriers partagés
> 
> **2. Les utilisateurs** :
> - Comptes individuels
> - Groupes de sécurité
> - Groupes de distribution
> 
> **3. Les services** :
> - Services de courrier électronique
> - Services d'applications
> - Points de connexion de service (SCP)

---

## Les attributs

> [!quote] Définition
> Les **attributs** sont les caractéristiques et les informations qu'un objet peut contenir. Ils définissent les éléments d'information qu'une classe (et donc une instance de cette classe) peut posséder.

### Caractéristiques des attributs

> [!info] Propriétés des attributs
> - Chaque **classe d'objet** a son propre jeu d'attributs
> - Les attributs sont **définis par le schéma AD**
> - Certains attributs sont **obligatoires**, d'autres **optionnels**
> - Les attributs peuvent être **mono-valués** ou **multi-valués**

> [!example] Exemple PowerShell - Lister tous les attributs d'un utilisateur
> ```powershell
> PS C:\Lab> Get-ADUser -Filter * -Properties * | Where-Object {$_.Name -eq "User1"}
> 
> AccountExpirationDate                : 
> accountExpires                       : 9223372036854775807
> AccountLockoutTime                   : 
> AccountNotDelegated                  : False
> AllowReversiblePasswordEncryption    : False
> AuthenticationPolicy                 : {}
> AuthenticationPolicySilo             : {}
> BadLogonCount                        : 0
> badPasswordTime                      : 0
> badPwdCount                          : 0
> CannotChangePassword                 : False
> CanonicalName                        : lab.lan/LabUtilisateurs/User1
> Certificates                         : {}
> City                                 : 
> CN                                   : User1
> codePage                             : 0
> Company                              : MyCompany
> (... et des dizaines d'autres attributs)
> ```

---

## Les classes d'objets

> [!info] Attribut "ObjectClass"
> L'attribut **ObjectClass** définit le **type d'un objet**. Il détermine si l'objet est un conteneur ou un objet final (leaf object).

### Conteneurs

> [!note] Types de conteneurs
> Les conteneurs peuvent contenir d'autres objets :
> 
> - **Container** : Conteneur natif AD
> - **Organizational Unit (OU)** : Unité d'Organisation
> - **Group** : Groupe (peut contenir des utilisateurs ou d'autres groupes)

### Leaf Objects (objets finaux)

> [!note] Types d'objets finaux
> Les objets finaux ne peuvent pas contenir d'autres objets :
> 
> - **User** : Utilisateur (compte de personne)
> - **Computer** : Ordinateur (client ou serveur)
> - **Printer** : Imprimante réseau
> - **Contact** : Contact externe (sans compte Windows)

---

## Les groupes

> [!important] Caractéristiques des groupes
> Les groupes Active Directory ont deux dimensions : **l'étendue** et **le type**.

### Étendue du groupe

> [!info] Les 3 étendues possibles
> 
> **1. Domaine local (Domain Local)** :
> - Portée limitée au **domaine local**
> - Peut contenir des membres de **tous les domaines approuvés**
> - Utilisé pour attribuer des **permissions sur les ressources locales**
> 
> **2. Globale (Global)** :
> - Portée sur **tous les domaines approuvés** de la forêt
> - Peut contenir uniquement des membres du **domaine local**
> - Utilisé pour regrouper des **utilisateurs par métier/fonction**
> 
> **3. Universelle (Universal)** :
> - Portée **maximale** (ensemble de la forêt)
> - Peut contenir des objets de **tous les domaines** de la forêt
> - Répliqué dans le **Catalogue Global**
> - À utiliser avec parcimonie (impact sur la réplication)

### Type de groupe

> [!info] Les 2 types de groupes
> 
> **1. Sécurité (Security)** :
> - Utilisé pour les **autorisations** et **permissions**
> - Peut être utilisé comme liste de distribution
> - Type par défaut et le plus courant
> 
> **2. Distribution (Distribution)** :
> - Utilisé uniquement pour les **listes de distribution** email
> - Ne peut **pas** être utilisé pour les permissions
> - Pas de SID attribué

> [!tip] Méthode AGDLP
> Pour une gestion optimale des permissions, utilise la méthode **AGDLP** (voir section dédiée plus loin).

---

## Les Unités d'Organisation (OU)

> [!quote] Rappel sur les OU
> Les **OU** (Organizational Units) sont des objets de classe **conteneur** utilisés pour hiérarchiser Active Directory.

### Caractéristiques des OU

> [!info] Propriétés des OU
> - Une OU peut contenir d'**autres classes d'objets** :
>   - Utilisateurs
>   - Ordinateurs
>   - Groupes
>   - Autres OU (imbrication)
> - Permet l'application de **GPO** (stratégies de groupe)
> - Permet la **délégation d'administration**
> - Structure l'AD de manière **logique**

---

## Les identifiants uniques

> [!abstract] Vue d'ensemble
> Chaque objet Active Directory possède plusieurs identifiants uniques qui permettent de le référencer de manière non ambiguë.

### GUID (Globally Unique Identifier)

> [!quote] Définition du GUID
> Le **GUID** (Globally Unique Identifier) est un identifiant unique global attribué à chaque objet AD à sa création.

> [!info] Caractéristiques du GUID
> - **Unique** au sein d'une forêt (et même universellement)
> - Attribué à la **création** de l'objet
> - **Ne change jamais** (même si l'objet est déplacé ou renommé)
> - Codé sur **128 bits**
> - Attribut AD : `ObjectGUID`
> 
> **Composition** :
> - 122 bits : Nombres aléatoires
> - 6 bits : Nombres fixes (version et variante)

> [!example] Exemple de GUID
> ```
> 3F2504E0-4F89-11D3-9A0C-0305E82C3301
> ```

### SID (Security Identifier)

> [!quote] Définition du SID
> Le **SID** (Security Identifier) est un identifiant de sécurité utilisé pour le contrôle d'accès.

> [!info] Caractéristiques du SID
> - **Unique** au sein d'un domaine
> - Attribué à la **création** de l'objet
> - **Peut changer** (ex: migration vers un autre domaine)
> - Longueur maximale : **256 caractères**
> - Attribut AD : `ObjectSID`
> 
> **Composition** :
> - Security principal domain SID (identifiant du domaine)
> - RID (Relative ID - numéro séquentiel unique)

> [!example] Exemple de SID
> ```
> S-1-5-21-156063872-1535639461-3779917529-1134
> │ │ │  └────────────────┬────────────────┘ │
> │ │ │                   │                   │
> │ │ │           Domain SID              RID (1134)
> │ │ │
> │ │ └─ NT Authority (5)
> │ └─── Revision (1)
> └───── Prefix "S"
> ```

### DN (Distinguished Name)

> [!quote] Définition du DN
> Le **DN** (Distinguished Name) est le chemin LDAP complet d'un objet dans l'annuaire Active Directory.

> [!info] Caractéristiques du DN
> - **Unique** au sein d'une forêt
> - Correspond au **chemin LDAP** dans l'annuaire
> - La longueur **dépend de l'emplacement** de l'objet dans l'AD
> - Attribut AD : `DistinguishedName`
> 
> **Composition** :
> - Le **nom du domaine** (DC components)
> - Le(s) **conteneur(s)** où se trouve l'objet (OU components)
> - Le **nom de l'objet** (CN - Common Name)

> [!example] Exemple de DN
> ```
> cn=wilder,ou=RS,ou=utilisateurs,ou=Paris,ou=France,dc=masociete,dc=fr
> │         │     │               │       │         └──────┬──────┘
> │         │     │               │       │          Domain Components
> │         │     │               │       └─ OU: France
> │         │     │               └───────── OU: Paris
> │         │     └─────────────────────────── OU: utilisateurs
> │         └───────────────────────────────────── OU: RS
> └─────────────────────────────────────────────── CN: wilder
> ```

### Récapitulatif des identifiants

| Identifiant | Portée | Immuable | Usage principal |
|-------------|--------|----------|-----------------|
| **GUID** | Forêt (universel) | ✅ Oui | Référence technique permanente |
| **SID** | Domaine | ❌ Non | Contrôle d'accès et permissions |
| **DN** | Forêt | ❌ Non | Chemin LDAP et localisation |

> [!example] Exemple PowerShell - Afficher les identifiants
> ```powershell
> PS C:\Lab> $Var1 = Get-ADUser -Filter * -Properties * | Where-Object {$_.Name -eq "User1"}
> 
> PS C:\Lab> $Var1 | Select Name,DistinguishedName,ObjectGUID,ObjectSID | Format-List
> 
> Name              : User1
> DistinguishedName : CN=User1,OU=LabUtilisateurs,DC=lab,DC=lan
> ObjectGUID        : 2c188179-b4cd-4f44-80cc-98e0720b8bfc
> ObjectSID         : S-1-5-21-11617303-4238263364-3208815124-1103
> ```

---

## Bonnes pratiques d'administration

> [!abstract] Sécurisation de l'infrastructure AD
> L'administration d'Active Directory nécessite la mise en place de bonnes pratiques de sécurité rigoureuses pour protéger l'infrastructure contre les menaces internes et externes.

---

## Gestion des identités et accès

### Principe de moindre privilège (JEA)

> [!important] Just Enough Administration
> **Principe fondamental** : Limiter les droits d'accès au **strict nécessaire**.
> 
> - Attribuer **uniquement les droits nécessaires** pour exécuter des tâches précises
> - Éviter les comptes avec des privilèges excessifs
> - Réviser régulièrement les permissions accordées
> - Appliquer le principe du "need-to-know"

### Comptes d'administration séparés

> [!important] Règle des Tiers (Tiering Model)
> **Bonne pratique essentielle** : Avoir des **comptes distincts** pour les tâches administratives et les tâches quotidiennes.
> 
> **Exemple** :
> - `jdupont` : Compte utilisateur standard pour le travail quotidien
> - `jdupont-admin` : Compte administrateur pour les tâches d'administration
> - `jdupont-da` : Compte Domain Admin (uniquement si nécessaire)

> [!warning] Jamais de travail quotidien avec un compte admin !
> Ne **jamais** utiliser un compte à privilèges élevés pour :
> - Consulter ses emails
> - Naviguer sur Internet
> - Ouvrir des documents
> - Effectuer des tâches bureautiques

### LAPS (Local Administrator Password Solution)

> [!info] Gestion des mots de passe administrateur local
> **LAPS** permet de :
> - **Changer automatiquement** le mot de passe du compte administrateur local des clients
> - **Stocker** ces mots de passe dans Active Directory
> - **Attribuer des permissions** pour consulter ces mots de passe
> - **Auditer** les accès aux mots de passe
> 
> → Évite d'avoir le **même mot de passe** administrateur local sur tous les postes

---

## Politique et contrôles d'accès

### Politiques de mot de passe renforcées

> [!important] Sécurité des mots de passe
> Établir des règles strictes pour la création de mots de passe robustes :
> 
> - **Complexité** : Majuscules, minuscules, chiffres, caractères spéciaux
> - **Longueur minimale** : Au moins 12-14 caractères (idéalement 16+)
> - **Délais d'expiration** : Renouvellement périodique (ex: 90 jours)
> - **Historique** : Empêcher la réutilisation des anciens mots de passe
> - **Verrouillage de compte** : Après X tentatives infructueuses

> [!tip] Phrase de passe vs mot de passe
> Privilégier les **phrases de passe** longues plutôt que des mots de passe courts complexes :
> - ❌ `P@ssw0rd!` (11 caractères)
> - ✅ `J'aime-les-chats-noirs-2024!` (29 caractères)

### Méthode AGDLP

> [!important] Séparation identité et ressources
> **Principe** : Les droits d'accès ne sont **jamais directement sur les utilisateurs**.
> 
> - Les droits d'accès sont attribués aux **groupes**
> - Les ressources ne connaissent que les **groupes de permissions**
> - Les utilisateurs sont membres de **groupes métiers**
> - Les groupes métiers appartiennent aux **groupes de permissions**

> [!note] AGDLP sera détaillé dans une section dédiée plus loin

---

## Opérations sur le réseau

### Protéger les contrôleurs de domaine

> [!warning] Sécurité physique et réseau des DC
> Les DC doivent être **isolés et protégés** :
> 
> - Les placer dans un **réseau dédié et sécurisé** :
>   - VLAN spécifique pour les DC
>   - DMZ sécurisée si exposition nécessaire
>   - Règles de firewall strictes
> 
> - **Restreindre l'accès physique** :
>   - Salle serveur sécurisée
>   - Accès contrôlé et tracé
>   - Vidéosurveillance

### Mises à jour régulières

> [!important] Maintenance cyclique
> **Application régulière des mises à jour de sécurité** :
> 
> - Patches de sécurité Windows Server
> - Mises à jour cumulatives mensuelles
> - Mises à jour d'urgence (zero-day)
> 
> **Faire évoluer les systèmes** :
> - Montée de version des OS serveur
> - Mise à jour des appliances (firewall, etc.)
> - Remplacement du matériel obsolète

> [!tip] Processus de patch management
> 1. Tester les mises à jour en environnement de test
> 2. Déployer d'abord sur un DC secondaire
> 3. Vérifier le bon fonctionnement
> 4. Déployer sur les autres DC
> 5. Documenter les changements

### Sécuriser les communications LDAP

> [!important] LDAPS - LDAP sur SSL/TLS
> **Protéger les transmissions** avec LDAPS (LDAP Sécurisé) :
> 
> - Utiliser le **port 636** (LDAPS) au lieu du port 389 (LDAP)
> - Déployer des **certificats SSL/TLS** sur les DC
> - **Désactiver LDAP non chiffré** si possible
> - Forcer l'utilisation de **Channel Binding** et **LDAP Signing**

> [!warning] LDAP non chiffré = risque
> Sur un LDAP non chiffré (port 389) :
> - Les **identifiants** circulent en clair
> - Les **requêtes** peuvent être interceptées
> - Risque de **man-in-the-middle**

---

## Surveillance

### Audits réguliers et surveillance des logs

> [!important] Détection des activités anormales
> 
> **Effectuer des audits réguliers** :
> - **Audits internes** : Revue des configurations et permissions
> - **Audits externes** : Tests d'intrusion et assessments
> - **Audits de conformité** : RGPD, ISO 27001, etc.
> 
> **Examiner les logs** pour détecter les activités suspectes :
> - Connexions inhabituelles (heures, lieux)
> - Échecs d'authentification répétés
> - Modifications d'objets sensibles
> - Ajouts de comptes ou de groupes
> - Élévations de privilèges

> [!tip] Logs critiques à surveiller
> - **Event ID 4720** : Création de compte utilisateur
> - **Event ID 4728/4732** : Ajout de membre à un groupe privilégié
> - **Event ID 4625** : Échec d'authentification
> - **Event ID 4768/4769** : Tickets Kerberos TGT/TGS
> - **Event ID 4776** : Authentification NTLM

### Stratégie de sauvegarde et de récupération

> [!important] Règle 3-2-1
> Préparer des plans de **sauvegarde et de restauration** pour les urgences :
> 
> **Règle 3-2-1** :
> - **3** copies des données
> - Sur **2** supports différents
> - **1** copie hors site (offsite)
> 
> **Éléments à sauvegarder** :
> - Base de données AD (NTDS.dit)
> - SYSVOL (GPO et scripts)
> - État du système (System State)
> - Configuration DNS
> - Documentation de l'infrastructure

> [!warning] Tester les restaurations !
> Une sauvegarde non testée est une sauvegarde inutile. Effectuer régulièrement des **tests de restauration** en environnement de laboratoire.

---

## Microsoft Tiering Model

> [!quote] Définition
> Le **Microsoft Tiering Model** (Modèle en Tiers) est un modèle de sécurité qui sépare les ressources et les administrateurs en niveaux pour **limiter les risques de propagation d'attaques** dans l'environnement Active Directory.

### Principe du Tiering Model

> [!important] Séparation en niveaux hermétiques
> Le modèle sépare les composants de l'infrastructure en fonction de leur **niveau d'importance** et rend ces différentes couches **hermétiques** les unes des autres.
> 
> **Objectif** : Empêcher qu'une compromission à un niveau inférieur puisse se propager aux niveaux supérieurs.

---

## Tier 0 - Niveau le plus critique

> [!important] Tier 0 : Contrôle de l'AD
> **Composants** :
> - **Contrôleurs de domaine** (DC)
> - **Administrateurs d'entreprise** (Enterprise Admins)
> - Autres actifs avec **contrôle direct** sur l'ensemble de l'environnement AD :
>   - Serveurs AD
>   - PKI (Infrastructure à clés publiques)
>   - ADFS (Federation Services)
>   - Serveurs de gestion (SCCM, etc.)
> 
> **Importance** : C'est la **couche la plus importante** et sensible.

### Règles d'accès Tier 0

> [!warning] Restrictions strictes
> Un administrateur **Tier 0** :
> - Peut gérer **uniquement** des composants de la couche **Tier 0**
> - Peut **RDP uniquement** sur des serveurs intégrés à ce niveau
> - Ne doit **JAMAIS** se connecter à des serveurs d'une couche inférieure
> - Ne doit **JAMAIS** utiliser son compte pour des tâches courantes
> 
> **Pourquoi ?** Une connexion sur une machine compromise de niveau inférieur pourrait permettre le vol des identifiants Tier 0 et compromettre tout l'AD !

---

## Tier 1 - Serveurs et applications

> [!info] Tier 1 : Gestion des serveurs applicatifs
> **Composants** :
> - **Serveurs d'applications**
> - **Middlewares**
> - **Serveurs de gestion** :
>   - SCCM (Configuration Manager)
>   - WSUS (Windows Server Update Services)
>   - SCOM (Operations Manager)
>   - Serveurs de bases de données
> - **Administrateurs** qui gèrent ces services

### Règles d'accès Tier 1

> [!warning] Restrictions Tier 1
> Un administrateur **Tier 1** :
> - Gère les **serveurs applicatifs** et **middlewares** de l'entreprise
> - Ne doit **PAS** être utilisé pour se connecter à une couche **supérieure** (Tier 0)
> - Ne doit **PAS** être utilisé pour se connecter à une couche **inférieure** (Tier 2)
> 
> **Pourquoi ?** Éviter la compromission en cascade entre les tiers.

---

## Tier 2 - Postes de travail

> [!info] Tier 2 : Environnement utilisateur
> **Composants** :
> - **Postes de travail** des utilisateurs finaux
> - **Postes des administrateurs** (pour leur travail quotidien)
> - **Périphériques mobiles** (smartphones, tablettes)
> - **Équipements BYOD** (Bring Your Own Device)

### Niveau de risque

> [!warning] Couche la plus "à risque"
> Le **Tier 2** est la couche la plus exposée aux menaces :
> 
> **Vecteurs d'attaque** :
> - **Erreurs humaines** (clic sur lien malveillant, etc.)
> - **Intrusions** (phishing, ransomware, malware)
> - **Périphériques mobiles** compromis
> - **Clés USB** infectées
> - **Sites web malveillants**

### Règles d'accès Tier 2

> [!warning] Restrictions Tier 2
> Un administrateur **Tier 2** :
> - Gère les **postes de travail** des utilisateurs
> - Ce compte ne doit **JAMAIS** être utilisé pour accéder aux couches supérieures (Tier 0 ou Tier 1)
> 
> **Pourquoi ?** Un poste utilisateur compromis ne doit pas pouvoir compromettre les serveurs ou l'infrastructure AD.

---

## Schéma du Tiering Model

> [!example] Vue d'ensemble du modèle en tiers
> ```
> ┌─────────────────────────────────────────────────┐
> │               TIER 0 (Critique)                 │
> │  DC, Enterprise Admins, PKI, ADFS              │
> │  ⚠️  Accès UNIQUEMENT depuis Tier 0            │
> └─────────────────────────────────────────────────┘
>                      ↑ Isolation
>                      │ (Pas de connexion descendante)
> ┌─────────────────────────────────────────────────┐
> │               TIER 1 (Important)                │
> │  Serveurs applicatifs, SCCM, WSUS, BDD         │
> │  ⚠️  Isolation vers Tier 0 et Tier 2           │
> └─────────────────────────────────────────────────┘
>                      ↑ Isolation
>                      │ (Pas de connexion descendante)
> ┌─────────────────────────────────────────────────┐
> │               TIER 2 (À risque)                 │
> │  Postes utilisateurs, mobiles, BYOD            │
> │  ⚠️  Zone la plus exposée aux attaques         │
> └─────────────────────────────────────────────────┘
> ```

> [!tip] Mise en œuvre progressive
> Le Tiering Model peut sembler complexe à mettre en place. Commence par :
> 1. Identifier les assets **Tier 0** (DC et serveurs critiques)
> 2. Créer des **comptes séparés** pour les admins Tier 0
> 3. Implémenter des **restrictions d'accès** sur les DC
> 4. Étendre progressivement aux autres tiers

---

## JIT & JEA

> [!abstract] Concepts de sécurité avancée
> **JIT** (Just-In-Time) et **JEA** (Just Enough Administration) sont deux concepts complémentaires qui renforcent considérablement la sécurité de l'administration Active Directory.

---

## JIT (Just-In-Time)

> [!quote] Définition JIT
> **Just-In-Time** permet aux administrateurs d'obtenir les privilèges nécessaires pour une tâche spécifique **pendant une période limitée**. À l'expiration, les droits élevés sont **révoqués automatiquement**.

### Problème adressé par JIT

> [!warning] Le problème des privilèges permanents
> En général, dans les entreprises, les comptes à privilèges élevés **gardent ces privilèges tout au long de leur vie**.
> 
> **⇒ En cas de compromission de compte, ces privilèges constituent un trou de sécurité majeur !**

### Solution JIT

> [!success] Élévation temporaire des privilèges
> **Principe** : Avoir les privilèges nécessaires **à un instant donné** pour une **période donnée**.
> 
> **Fonctionnement** :
> 1. L'administrateur **demande** une élévation de privilèges
> 2. La demande nécessite une **approbation** par une ou plusieurs personnes
> 3. L'administrateur doit fournir les **détails de l'intervention**
> 4. Les privilèges sont accordés pour une **durée limitée** (ex: 4 heures)
> 5. À l'expiration, les privilèges sont **automatiquement révoqués**

> [!example] Exemple de workflow JIT
> ```
> 1. Admin demande : "Besoin d'être Domain Admin pour ajouter un DC"
> 2. Manager approuve : "OK pour 4 heures"
> 3. Admin reçoit les privilèges Domain Admin
> 4. Admin effectue sa tâche
> 5. Après 4 heures : Privilèges automatiquement retirés
> ```

---

## JEA (Just Enough Administration)

> [!quote] Définition JEA
> **Just Enough Administration** limite les privilèges des administrateurs aux **seuls droits nécessaires** pour effectuer une tâche spécifique, réduisant ainsi les risques associés à l'utilisation de comptes à privilèges élevés.

### Principe de JEA

> [!important] Principe de moindre privilège granulaire
> Au lieu de donner des droits d'administration complets, JEA permet de créer des **rôles sur mesure** avec uniquement les permissions requises.
> 
> **Exemple** : Un administrateur qui doit uniquement redémarrer un service n'a pas besoin d'être **Administrateur local complet**.

> [!example] Exemple pratique
> **Sans JEA** :
> - Admin a besoin de redémarrer le service IIS
> - On lui donne les droits "Administrateur local" sur le serveur
> - → Il peut maintenant tout faire sur le serveur !
> 
> **Avec JEA** :
> - Admin a besoin de redémarrer le service IIS
> - On lui crée un rôle JEA avec uniquement :
>   - `Restart-Service -Name W3SVC`
> - → Il peut uniquement redémarrer IIS, rien d'autre

### Technologies JEA

> [!info] Implémentation technique
> JEA s'appuie sur :
> - **PowerShell Remoting**
> - **Session Configurations**
> - **Role Capability Files** (.psrc)
> - **Constrainted Language Mode**

---

## Fusion JIT & JEA

> [!success] Combinaison puissante
> La combinaison de **JIT** et **JEA** offre une sécurité optimale :
> 
> - **JIT** : Limitation dans le **temps**
> - **JEA** : Limitation dans les **actions possibles**
> 
> **Résultat** : L'administrateur a :
> - Les privilèges **uniquement quand il en a besoin**
> - **Uniquement les permissions** nécessaires pour sa tâche
> - Pour une **durée limitée**
> - Avec un **audit complet** de ses actions

> [!example] Exemple combiné JIT + JEA
> ```
> 1. Admin demande : "Redémarrer le service SQL sur PROD-SQL01"
> 2. Manager approuve : "OK pour 1 heure"
> 3. Admin obtient un rôle JEA temporaire avec UNIQUEMENT :
>    - Restart-Service -Name MSSQLSERVER sur PROD-SQL01
> 4. Admin redémarre le service
> 5. Après 1 heure : Rôle JEA automatiquement révoqué
> 6. Toutes les actions sont auditées
> ```

---

## AGDLP

> [!quote] Définition de l'acronyme
> **AGDLP** est une méthode d'organisation des permissions gérées dans AD DS.
> 
> - **A** → **Account** (Comptes utilisateurs)
> - **G** → **Global groups** (Groupes métiers)
> - **DL** → **Domain Local Groups** (Groupes de droits)
> - **P** → **Permissions** (Permissions sur les ressources)

### Principe fondamental

> [!important] Ne JAMAIS donner des droits directement aux utilisateurs !
> AGDLP évite la **TRÈS mauvaise pratique** de donner des droits d'accès directement à un utilisateur.
> 
> **Principe** :
> 1. Un **utilisateur** appartient à un **groupe métier** (Global)
> 2. Ce **groupe métier** appartient à un ou plusieurs **groupes de droits** (Domain Local)
> 3. Ces **groupes de droits** ont des **permissions** sur des ressources

### Pourquoi utiliser AGDLP ?

> [!success] Avantages de la méthode AGDLP
> - **Simplicité de gestion** : Modifier un groupe au lieu de centaines d'utilisateurs
> - **Évolutivité** : Facile d'ajouter/retirer des utilisateurs
> - **Séparation des rôles** : Groupes métiers vs groupes de permissions
> - **Traçabilité** : Compréhension claire de "qui a accès à quoi"
> - **Réutilisabilité** : Un groupe métier peut avoir accès à plusieurs ressources

---

## Exemple 1 : Comptabilité

> [!example] Scénario
> **Bob** et **Alice** sont 2 comptables nouvellement embauchés.
> 
> Ils doivent avoir accès aux dossiers suivants :
> - Dossier **"Compta"** en **lecture et écriture**
> - Dossier **"Paye"** en **lecture seule**
> 
> ⚠️ **On ne donne PAS directement accès aux 2 dossiers aux 2 utilisateurs !**

### Solution avec AGDLP

> [!success] Mise en œuvre AGDLP
> 
> **Étape 1** : Créer le groupe métier
> - Créer le groupe **Global** : `Grp_Compta`
> - Ajouter **Bob** et **Alice** dans `Grp_Compta`
> 
> **Étape 2** : Créer les groupes de droits
> - Créer le groupe **Domain Local** : `Grp_Compta_RW`
>   - Attribuer les permissions **NTFS RW** sur le dossier partagé `\\serveur\Compta`
> - Créer le groupe **Domain Local** : `Grp_Paye_R`
>   - Attribuer les permissions **NTFS R** sur le dossier partagé `\\serveur\Paye`
> 
> **Étape 3** : Lier les groupes
> - Ajouter `Grp_Compta` comme **membre** de :
>   - `Grp_Compta_RW`
>   - `Grp_Paye_R`
> 
> **⇒ Résultat** : Bob et Alice ont accès aux 2 dossiers avec les bons droits.

> [!note] Schéma AGDLP - Exemple 1
> ```
> Utilisateurs (A)    Groupes métiers (G)    Groupes droits (DL)       Permissions (P)
> 
> Bob          ──┐
>               ├──> Grp_Compta ──┬──> Grp_Compta_RW ──> \\serveur\Compta (RW)
> Alice        ──┘                │
>                                 └──> Grp_Paye_R ──────> \\serveur\Paye (R)
> ```

---

## Exemple 2 : Accès transversal

> [!example] Scénario
> **Alice** doit maintenant avoir accès à un dossier **"Finance"** en **lecture seule**, mais **pas Bob** !
> 
> ⚠️ **On ne donne PAS directement accès à ce dossier à Alice !**

### Solution avec AGDLP

> [!success] Mise en œuvre AGDLP avec groupe transversal
> 
> **Étape 1** : Créer un groupe métier transversal
> - Créer le groupe **Global** : `Grp_Compta_Finance`
> - Ajouter **Alice** dans `Grp_Compta_Finance`
> 
> **Étape 2** : Créer le groupe de droits
> - Créer le groupe **Domain Local** : `Grp_Finance_R`
>   - Attribuer les permissions **NTFS R** sur le dossier partagé `\\serveur\Finance`
> 
> **Étape 3** : Lier les groupes
> - Ajouter `Grp_Compta_Finance` comme **membre** de `Grp_Finance_R`
> 
> **⇒ Résultat** : Alice a accès au dossier Finance avec les bons droits, mais pas Bob.

> [!note] Schéma AGDLP - Exemple 2
> ```
> Utilisateurs (A)    Groupes métiers (G)           Groupes droits (DL)    Permissions (P)
> 
> Alice ──┬──> Grp_Compta ──┬──> Grp_Compta_RW ──> \\serveur\Compta (RW)
>         │                 └──> Grp_Paye_R ──────> \\serveur\Paye (R)
>         │
>         └──> Grp_Compta_Finance ──> Grp_Finance_R ──> \\serveur\Finance (R)
> 
> Bob ────────> Grp_Compta ──┬──> Grp_Compta_RW ──> \\serveur\Compta (RW)
>                             └──> Grp_Paye_R ──────> \\serveur\Paye (R)
> ```

---

## Récapitulatif AGDLP

> [!success] Méthodologie AGDLP
> 
> **Règles d'or** :
> 1. ✅ Les **utilisateurs** (A) sont membres de **groupes métiers** (G)
> 2. ✅ Les **groupes métiers** (G) sont membres de **groupes de droits** (DL)
> 3. ✅ Les **groupes de droits** (DL) ont des **permissions** (P) sur les ressources
> 4. ❌ **JAMAIS** de permissions directes sur un utilisateur
> 5. ❌ **JAMAIS** de permissions directes sur un groupe métier

> [!tip] Nomenclature recommandée
> Pour faciliter la gestion, adopte une **nomenclature claire** :
> 
> **Groupes métiers (Global)** :
> - Préfixe : `Grp_`
> - Exemple : `Grp_Compta`, `Grp_RH`, `Grp_IT`
> 
> **Groupes de droits (Domain Local)** :
> - Préfixe : `Grp_` + Ressource + Niveau d'accès
> - Exemple : `Grp_Compta_RW`, `Grp_Paye_R`, `Grp_Finance_RW`

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Protocoles réseaux associés

- **DNS** : Service **obligatoire** pour AD, résolution de noms ET services (SRV)
- **SNTP** : Synchronisation temporelle **impérative** pour Kerberos (tolérance 5 min)
- **LDAP** : Colonne vertébrale d'AD, 5 opérations de base (bind, search, add, delete, modify)
- **LDIF** : Format texte pour import/export/modification en masse de l'annuaire
- **Kerberos** : Protocole d'authentification par défaut depuis Windows 2000, authentification mutuelle
- **X.509** : Certificats numériques pour authentification, chiffrement et non-répudiation
- **NTFS** : Gestion des permissions en synergie avec les groupes AD

### Fonctionnalités avancées

**Niveau fonctionnel** :
- Détermine les fonctionnalités AD disponibles
- Correspond à la version de l'OS serveur le plus ancien
- ⚠️ Montée de niveau **irréversible**

**Schéma AD** :
- Définition de tous les types d'objets et attributs possibles
- Modification = opération critique sur toute la forêt

**Réplication** :
- Synchronise les données entre DC (annuaire, GPO, scripts, DNS)
- Processus en 8 étapes avec KCC, DNS et DC
- Intervalles : 5 min (intra-site), 180 min (inter-site)

**KCC (Knowledge Consistency Checker)** :
- Génère automatiquement la topologie de réplication
- Présent sur tous les DC
- Régénère la topologie en cas de changement

### Les 5 rôles FSMO

| Rôle | Niveau | Fonction | Criticité |
|------|--------|----------|-----------|
| **Schema Master** | Forêt | Modification du schéma | Critique |
| **Domain Naming Master** | Forêt | Gestion des domaines | Importante |
| **RID Master** | Domaine | Attribution des SID | Critique |
| **Infrastructure Master** | Domaine | Relations inter-domaines | Moyenne |
| **PDC Emulator** | Domaine | Temps, passwords, GPO | Très critique |

> [!warning] Ne pas confondre
> - Rôles de serveur Windows (DHCP, DNS, IIS...)
> - Rôles Active Directory (AD DS, AD CS, AD FS...)
> - Rôles FSMO (Schema Master, RID Master, PDC Emulator...)

### Les objets AD

**Types d'objets** :
- Ressources (ordinateurs, imprimantes, dossiers partagés)
- Utilisateurs (comptes individuels et groupes)
- Services (courrier, applications)

**Attributs** :
- Caractéristiques d'un objet définies par le schéma
- Chaque classe a son propre jeu d'attributs
- `ObjectClass` : Définit le type (container, user, computer, group, etc.)

**Groupes** :
- **Étendue** : Domain Local, Global, Universal
- **Type** : Security (permissions) ou Distribution (email)

**Identifiants uniques** :

| ID | Portée | Immuable | Usage |
|----|--------|----------|-------|
| **GUID** | Forêt | ✅ Oui | Référence permanente |
| **SID** | Domaine | ❌ Non | Contrôle d'accès |
| **DN** | Forêt | ❌ Non | Chemin LDAP |

### Bonnes pratiques d'administration

**Gestion des identités** :
- **JEA** : Principe de moindre privilège
- **Comptes séparés** : Tiers Model (compte standard + compte admin)
- **LAPS** : Gestion des mots de passe admin locaux

**Sécurité réseau** :
- Protéger les DC (VLAN dédié, DMZ)
- MAJ régulières (patch management)
- Sécuriser LDAP avec **LDAPS** (port 636)

**Surveillance** :
- Audits réguliers (internes et externes)
- Surveillance des logs (Event IDs 4720, 4728, 4625...)
- Sauvegarde avec **règle 3-2-1**

**Microsoft Tiering Model** :
- **Tier 0** : DC, Enterprise Admins (plus critique)
- **Tier 1** : Serveurs applicatifs, SCCM, WSUS
- **Tier 2** : Postes utilisateurs (plus à risque)
- Isolation stricte entre les tiers

**JIT & JEA** :
- **JIT** : Privilèges temporaires (limitation dans le temps)
- **JEA** : Privilèges minimaux (limitation des actions)
- Combinaison = sécurité optimale

**AGDLP** :
- **A**ccount → **G**lobal group → **D**omain **L**ocal group → **P**ermissions
- ❌ Jamais de permissions directes sur utilisateurs
- ✅ Toujours passer par des groupes

### Commandes PowerShell essentielles

```powershell
# Utilisateurs avec tous les attributs
Get-ADUser -Filter * -Properties *

# Groupes et leur étendue
Get-ADGroup -Filter * | Select Name, GroupScope, GroupCategory

# Rôles FSMO
Get-ADDomain | Select PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select SchemaMaster, DomainNamingMaster

# Contrôleurs de domaine
Get-ADDomainController
```

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **AGDLP** | Méthode d'organisation des permissions : Account → Global group → Domain Local group → Permissions |
| **Attribut** | Caractéristique d'un objet AD définie par le schéma (nom, email, téléphone, etc.) |
| **DN (Distinguished Name)** | Chemin LDAP complet d'un objet dans l'arborescence AD |
| **DNS** | Domain Name System - Service obligatoire pour AD assurant la résolution de noms et services |
| **FSMO** | Flexible Single Master Operation - 5 rôles spécifiques attribués à certains DC |
| **GUID** | Globally Unique Identifier - Identifiant unique universel sur 128 bits, immuable |
| **JEA** | Just Enough Administration - Limitation des privilèges au strict nécessaire |
| **JIT** | Just-In-Time - Élévation temporaire des privilèges pour une durée limitée |
| **KCC** | Knowledge Consistency Checker - Service générant automatiquement la topologie de réplication |
| **Kerberos** | Protocole d'authentification par défaut Windows (depuis 2000), avec authentification mutuelle |
| **LAPS** | Local Administrator Password Solution - Gestion automatique des mots de passe admin locaux |
| **LDAP** | Lightweight Directory Access Protocol - Protocole standard d'accès aux annuaires |
| **LDAPS** | LDAP Sécurisé sur SSL/TLS (port 636) |
| **LDIF** | LDAP Data Interchange Format - Format texte pour import/export/modification de l'annuaire |
| **Niveau fonctionnel** | Version AD déterminant les fonctionnalités disponibles, basée sur l'OS le plus ancien |
| **NTFS** | New Technology File System - Système de fichiers avec gestion intégrée des permissions AD |
| **PDC Emulator** | Rôle FSMO gérant le temps, les mots de passe et le verrouillage de comptes |
| **Réplication** | Processus de synchronisation des données entre contrôleurs de domaine |
| **RID** | Relative Identifier - Composant du SID assurant l'unicité dans un domaine |
| **Schéma AD** | Définition de tous les types d'objets et attributs possibles dans l'annuaire |
| **SID** | Security Identifier - Identifiant de sécurité unique dans un domaine |
| **SNTP** | Simple Network Time Protocol - Protocole de synchronisation temporelle |
| **Tiering Model** | Modèle de sécurité Microsoft séparant l'infrastructure en 3 niveaux hermétiques |
| **X.509** | Norme définissant le format des certificats numériques |

---

## Conclusion

> [!success] Ce que tu dois maîtriser
> Après l'étude de cette partie 2 sur Active Directory, tu dois être capable de :
> 
> ✅ **Comprendre les protocoles réseaux** associés à AD :
>    - DNS, SNTP, LDAP, Kerberos, X.509, NTFS
> 
> ✅ **Maîtriser les fonctionnalités avancées** :
>    - Niveau fonctionnel et ses implications
>    - Schéma AD et son rôle
>    - Processus de réplication et KCC
>    - Les 5 rôles FSMO et leur importance
> 
> ✅ **Connaître les objets AD en profondeur** :
>    - Classes d'objets et attributs
>    - Types de groupes (étendue et type)
>    - Identifiants uniques (GUID, SID, DN)
> 
> ✅ **Appliquer les bonnes pratiques d'administration** :
>    - Gestion des identités (JEA, comptes séparés, LAPS)
>    - Sécurité réseau (protection DC, MAJ, LDAPS)
>    - Microsoft Tiering Model (Tier 0/1/2)
>    - JIT & JEA pour la sécurité avancée
>    - Méthode AGDLP pour les permissions
> 
> ✅ **Distinguer** :
>    - Rôles serveur vs Rôles AD vs Rôles FSMO
>    - Groupes métiers vs Groupes de droits

---

## Liens et ressources

> [!info] Aucun lien externe n'était présent dans le document PowerPoint source
> Ce document de révision a été créé exclusivement à partir du contenu des slides du cours.

---

**Document créé le** : Janvier 2026  
**Format** : Markdown pour Obsidian  
**Version** : 1.0  
**Auteur** : Support de révision TSSR
