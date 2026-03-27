
## 📋 Table des matières

1. [Qu'est-ce qu'Active Directory](#quest-ce-quactive-directory)
2. [Forêt, arborescence, domaine](#forêt-arborescence-domaine)
3. [Contrôleurs de domaine (DC)](#contrôleurs-de-domaine-dc)
4. [Avantages d'Active Directory](#avantages-dactive-directory)
5. [LDAP et Kerberos](#ldap-et-kerberos)

---

## Qu'est-ce qu'Active Directory

### 📖 Définition

**Active Directory (AD)** est un service d'annuaire développé par Microsoft pour les réseaux Windows. Il s'agit d'une base de données centralisée qui stocke et organise les informations sur les ressources réseau (utilisateurs, ordinateurs, imprimantes, etc.) et permet leur gestion unifiée.

> [!info] Service d'annuaire
> Un service d'annuaire est comparable à un annuaire téléphonique pour un réseau informatique. Il répertorie tous les objets (utilisateurs, machines, ressources) et leurs attributs, permettant de les localiser et de les gérer efficacement.

### 🎯 Rôle principal

Active Directory fournit :
- **Authentification centralisée** : un utilisateur se connecte une seule fois pour accéder à toutes les ressources autorisées
- **Autorisation** : définit qui peut accéder à quoi
- **Gestion centralisée** : administration des utilisateurs, groupes, ordinateurs depuis un point unique
- **Réplication** : synchronisation automatique des données entre plusieurs serveurs

### 🔑 Composants fondamentaux

| Composant | Description |
|-----------|-------------|
| **Objets** | Éléments individuels (utilisateur, ordinateur, imprimante) |
| **Attributs** | Propriétés des objets (nom, email, département) |
| **Schéma** | Définit les types d'objets et leurs attributs possibles |
| **Catalogue global** | Index de tous les objets de la forêt pour les recherches rapides |

> [!example] Exemple concret
> Lorsqu'un employé Jean Dupont rejoint l'entreprise :
> - Un objet "utilisateur" est créé dans AD
> - Ses attributs incluent : nom, prénom, email, département, numéro de téléphone
> - Il reçoit un compte unique pour accéder aux ressources de l'entreprise
> - L'administrateur définit ses droits d'accès depuis la console AD

---

## Forêt, arborescence, domaine

### 🌲 Structure hiérarchique

Active Directory utilise une structure hiérarchique à trois niveaux pour organiser les ressources :

```
Forêt (Forest)
    └── Arborescence (Tree)
            └── Domaine (Domain)
                    └── Unité d'organisation (OU)
```

### 🍃 Domaine (Domain)

**Définition** : Un domaine est la limite administrative de base dans Active Directory. C'est un groupe d'objets (utilisateurs, ordinateurs, groupes) qui partagent la même base de données d'annuaire.

**Caractéristiques** :
- Possède un nom DNS unique (ex: `entreprise.com`)
- Contient sa propre base de données AD
- Applique ses propres stratégies de sécurité (GPO)
- Limite de réplication : les modifications sont répliquées entre tous les DC du domaine

> [!tip] Quand créer un domaine ?
> Un domaine est généralement créé pour :
> - Représenter une entité géographique (Paris, Londres)
> - Représenter une division organisationnelle (Marketing, IT)
> - Isoler des stratégies de sécurité différentes

**Exemple de nommage** :
```
entreprise.com           → Domaine racine
paris.entreprise.com     → Sous-domaine
marketing.entreprise.com → Sous-domaine
```

### 🌳 Arborescence (Tree)

**Définition** : Une arborescence est une hiérarchie de domaines qui partagent un espace de noms DNS contigu (continu).

**Caractéristiques** :
- Les domaines enfants héritent du nom de domaine parent
- Relations d'approbation bidirectionnelles et transitives automatiques
- Partage du même schéma et configuration
- Catalogue global commun

**Structure d'arborescence** :
```
entreprise.com (racine)
    ├── paris.entreprise.com
    │       └── dev.paris.entreprise.com
    └── london.entreprise.com
```

> [!info] Espace de noms contigu
> Les domaines forment un arbre avec une racine commune. `dev.paris.entreprise.com` appartient à l'arborescence car il continue le nom `entreprise.com`.

### 🌲 Forêt (Forest)

**Définition** : Une forêt est l'ensemble de toutes les arborescences Active Directory qui partagent le même schéma, la même configuration et le même catalogue global. C'est la limite de sécurité ultime dans AD.

**Caractéristiques** :
- Peut contenir une ou plusieurs arborescences
- Premier domaine créé = domaine racine de la forêt
- Schéma unique partagé par tous les domaines
- Relations d'approbation automatiques entre tous les domaines
- Catalogue global unique

**Exemple de forêt multi-arborescences** :
```
Forêt
    ├── Arborescence 1: entreprise.com
    │       ├── paris.entreprise.com
    │       └── london.entreprise.com
    └── Arborescence 2: filiale.fr
            └── lyon.filiale.fr
```

> [!warning] Limite de sécurité
> La forêt est la véritable limite de sécurité dans AD, pas le domaine. Un administrateur de domaine dans la forêt peut potentiellement obtenir des privilèges dans d'autres domaines de la même forêt.

### 📊 Comparaison des niveaux

| Niveau | Portée | Nom DNS | Base de données | Schéma |
|--------|--------|---------|-----------------|--------|
| **Domaine** | Limite administrative de base | Unique | Propre | Partagé (forêt) |
| **Arborescence** | Hiérarchie de domaines | Contigu | Multiple | Partagé (forêt) |
| **Forêt** | Limite de sécurité ultime | Multiple | Multiple | Unique |

### 🔗 Relations d'approbation (Trust)

Les relations d'approbation permettent aux utilisateurs d'un domaine d'accéder aux ressources d'un autre domaine.

**Dans une forêt** :
- Approbations **automatiques** et **bidirectionnelles**
- **Transitives** : si A approuve B et B approuve C, alors A approuve C

**Types d'approbation** :
- **Parente-enfant** : entre un domaine parent et son sous-domaine
- **Arbre-racine** : entre la racine de la forêt et une nouvelle arborescence
- **Raccourci** : créée manuellement pour optimiser l'authentification
- **Externe** : entre domaines de forêts différentes (non transitive)
- **Forêt** : entre deux forêts entières

> [!example] Exemple de transitivité
> ```
> entreprise.com approuve paris.entreprise.com
> paris.entreprise.com approuve dev.paris.entreprise.com
> → entreprise.com approuve automatiquement dev.paris.entreprise.com
> ```

---

## Contrôleurs de domaine (DC)

### 🖥️ Définition

Un **Contrôleur de Domaine (Domain Controller - DC)** est un serveur Windows Server qui exécute les services Active Directory Domain Services (AD DS). Il héberge et gère la base de données Active Directory pour son domaine.

### 🎯 Rôles et responsabilités

Un DC assure les fonctions suivantes :

1. **Authentification** :
   - Valide les identifiants des utilisateurs lors de la connexion
   - Délivre des tickets Kerberos pour l'accès aux ressources

2. **Stockage de l'annuaire** :
   - Héberge une copie complète de la base de données AD du domaine
   - Stocke tous les objets et leurs attributs

3. **Réplication** :
   - Synchronise les modifications avec les autres DC du domaine
   - Assure la cohérence des données à travers le réseau

4. **Application des stratégies** :
   - Distribue les stratégies de groupe (GPO) aux ordinateurs et utilisateurs
   - Applique les configurations de sécurité

### 🔄 Modèle multi-maître

Active Directory utilise un modèle de **réplication multi-maître** (multi-master replication) :

- **Tous les DC sont égaux** : modifications possibles sur n'importe quel DC
- **Réplication automatique** : les changements se propagent à tous les DC
- **Haute disponibilité** : si un DC tombe, les autres continuent à fonctionner
- **Tolérance aux pannes** : pas de point unique de défaillance

> [!info] Avantage du multi-maître
> Contrairement aux systèmes maître-esclave traditionnels, il n'y a pas de DC "principal". Chaque DC peut accepter des modifications, ce qui améliore les performances et la disponibilité.

### 👑 Rôles FSMO (Flexible Single Master Operations)

Bien que AD soit multi-maître, certaines opérations critiques nécessitent un seul DC responsable pour éviter les conflits. Ces rôles sont appelés **FSMO** :

#### Rôles au niveau de la forêt (1 par forêt)

| Rôle | Nom complet | Responsabilité |
|------|-------------|----------------|
| **Schema Master** | Maître du schéma | Gère les modifications du schéma AD (ajout de classes ou attributs) |
| **Domain Naming Master** | Maître de nommage des domaines | Autorise l'ajout/suppression de domaines dans la forêt |

#### Rôles au niveau du domaine (1 par domaine)

| Rôle | Nom complet | Responsabilité |
|------|-------------|----------------|
| **RID Master** | Maître RID | Distribue les pools de RID (Relative Identifier) pour créer des SID uniques |
| **PDC Emulator** | Émulateur PDC | Synchronisation de l'heure, verrouillages de comptes, compatibilité NT4 |
| **Infrastructure Master** | Maître d'infrastructure | Maintient les références entre domaines (groupes multi-domaines) |

> [!warning] Perte d'un rôle FSMO
> Si un DC hébergeant un rôle FSMO tombe, les opérations liées à ce rôle ne fonctionneront plus. Il faut alors transférer ou saisir (seize) le rôle vers un autre DC.

**Commandes PowerShell pour identifier les rôles FSMO** :
```powershell
# Voir tous les rôles FSMO de la forêt
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster

# Voir les rôles FSMO du domaine
Get-ADDomain | Select-Object RIDMaster, PDCEmulator, InfrastructureMaster

# Vue complète
netdom query fsmo
```

### 🔐 Contrôleur de domaine en lecture seule (RODC)

Un **Read-Only Domain Controller (RODC)** est un type spécial de DC introduit dans Windows Server 2008.

**Caractéristiques** :
- Contient une copie **en lecture seule** de la base de données AD
- Ne réplique **jamais** vers d'autres DC (réplication unidirectionnelle)
- Cache sélectif des mots de passe
- Aucun rôle FSMO

**Cas d'usage** :
- Sites distants avec sécurité physique limitée
- Bureaux avec peu de personnel IT
- Environnements où le vol de serveur est un risque

> [!tip] Sécurité RODC
> Si un RODC est compromis, seuls les mots de passe mis en cache sont en danger, pas l'ensemble de l'annuaire. Les autres DC restent sécurisés.

### 📍 Sites Active Directory

Un **site AD** représente un emplacement physique avec une bonne connectivité réseau (généralement un LAN).

**Avantages des sites** :
- Optimisation de la réplication (utilise les liens les plus rapides)
- Authentification locale (les utilisateurs se connectent au DC le plus proche)
- Optimisation de la recherche de services (impression, DFS)

**Configuration** :
```
Site Paris
    ├── Sous-réseau : 192.168.1.0/24
    └── DC : DC-PARIS-01, DC-PARIS-02

Site Londres
    ├── Sous-réseau : 192.168.10.0/24
    └── DC : DC-LONDON-01
```

> [!example] Scénario d'utilisation
> Une entreprise avec un siège à Paris et une filiale à Londres :
> - 2 sites AD configurés (Paris, Londres)
> - Un DC dans chaque site
> - La réplication entre sites utilise un planning (ex: toutes les heures)
> - Les employés de Londres s'authentifient sur DC-LONDON-01 (plus rapide)

### 🔄 Réplication inter-DC

La réplication assure que tous les DC ont les mêmes informations.

**Types de réplication** :

1. **Intra-site** (dans le même site) :
   - Rapide et fréquente (notifications de changement)
   - Non compressée
   - Utilise le protocole RPC

2. **Inter-site** (entre sites différents) :
   - Programmée selon un planning
   - Compressée (économise la bande passante)
   - Peut utiliser RPC ou SMTP (pour les domaines différents)

**Conflits de réplication** :
Si deux modifications contradictoires sont faites simultanément sur deux DC différents, AD résout le conflit avec :
- **USN (Update Sequence Number)** : numéro de version le plus élevé gagne
- **Timestamp** : modification la plus récente gagne
- En cas d'égalité parfaite : le GUID du DC fait foi

> [!warning] Latence de réplication
> Les changements ne sont pas instantanés sur tous les DC. Il peut y avoir un délai (secondes en intra-site, minutes/heures en inter-site selon la configuration).

---

## Avantages d'Active Directory

### 🎯 Centralisation de l'administration

**Gestion unifiée** :
- Administration de milliers d'utilisateurs depuis une console unique
- Déploiement de logiciels à grande échelle
- Application de configurations standardisées (GPO)
- Réinitialisation de mots de passe centralisée

> [!example] Sans AD vs Avec AD
> **Sans AD** : Pour changer le mot de passe d'un utilisateur sur 50 machines, un administrateur doit se connecter à chaque machine individuellement.
> 
> **Avec AD** : L'administrateur change le mot de passe une fois dans AD, et le changement s'applique automatiquement partout.

### 🔐 Sécurité renforcée

**Authentification unique (SSO - Single Sign-On)** :
- Un utilisateur s'identifie une fois
- Accès transparent à toutes les ressources autorisées
- Réduit les risques liés aux mots de passe multiples

**Contrôle d'accès granulaire** :
- Permissions définies au niveau utilisateur, groupe, OU
- Héritage et délégation de droits
- Audit des accès et modifications

**Stratégies de sécurité centralisées** :
- Complexité des mots de passe
- Durée de validité des sessions
- Verrouillage automatique après tentatives échouées
- Chiffrement et signature des communications

| Fonctionnalité | Avantage sécurité |
|----------------|-------------------|
| **Kerberos** | Authentification forte avec tickets chiffrés |
| **LDAPS** | Communication chiffrée vers l'annuaire |
| **Groupes de sécurité** | Gestion simplifiée des droits d'accès |
| **GPO de sécurité** | Application uniforme des politiques |

### 📈 Scalabilité

Active Directory peut gérer :
- **Millions d'objets** dans un seul domaine
- **Centaines de sites** géographiquement distribués
- **Milliers de DC** en réplication
- **Plusieurs forêts** interconnectées

**Croissance flexible** :
- Ajout de DC sans interruption de service
- Création de nouveaux domaines selon les besoins
- Extension de la forêt avec de nouvelles arborescences

> [!tip] Performance
> La scalabilité d'AD est prouvée : des organisations comme la NASA ou de grandes multinationales gèrent des millions d'objets sans problème de performance.

### 🔄 Haute disponibilité

**Tolérance aux pannes** :
- Modèle multi-maître : aucun point unique de défaillance
- Si un DC tombe, les autres prennent le relais automatiquement
- Réplication continue assure la cohérence des données

**Maintenance sans interruption** :
- Mise à jour d'un DC sans affecter le service
- Ajout/retrait de DC à chaud
- Migration de rôles FSMO sans downtime

**Plan de reprise d'activité (DRP)** :
- Sauvegarde de l'état système (System State)
- Restauration autoritaire possible
- Récupération de la forêt entière possible

### 🌐 Interopérabilité

**Standards ouverts** :
- Basé sur **LDAP** (Lightweight Directory Access Protocol)
- Utilise **DNS** pour la résolution de noms
- Authentification **Kerberos** (standard IETF)
- Compatible avec les systèmes non-Windows

**Intégrations possibles** :
- Linux/Unix peuvent s'authentifier via AD (Samba, Winbind, SSSD)
- Applications tierces supportent LDAP
- Services cloud (Microsoft 365, Azure) s'intègrent nativement
- Synchronisation avec d'autres annuaires possible

### 📋 Stratégies de groupe (GPO)

Les **Group Policy Objects** permettent de :
- Déployer des configurations automatiquement
- Installer des logiciels à distance
- Configurer des paramètres de sécurité
- Mapper des lecteurs réseau
- Définir des fonds d'écran, des économiseurs d'écran

**Portée d'application** :
```
Forêt → Domaine → OU → Utilisateurs/Ordinateurs
```

> [!example] Cas d'usage GPO
> - Bloquer l'accès au Panneau de configuration pour les utilisateurs standards
> - Installer automatiquement Microsoft Office sur tous les PC
> - Configurer le pare-feu Windows uniformément
> - Rediriger les dossiers Documents vers un serveur de fichiers

### 🔍 Recherche et localisation de ressources

**Catalogue global** :
- Recherche rapide dans toute la forêt
- Localisation d'utilisateurs, groupes, imprimantes
- Index optimisé pour les requêtes LDAP

**Services de localisation** :
- Trouver le DC le plus proche (basé sur les sites AD)
- Localiser les serveurs de fichiers DFS
- Découvrir les imprimantes réseau par emplacement

### 👥 Gestion des identités et des accès (IAM)

Active Directory est le cœur de l'IAM dans les environnements Microsoft :

**Gestion du cycle de vie des utilisateurs** :
1. **Création** : nouvel employé → compte AD
2. **Modification** : changement de poste → mise à jour des groupes
3. **Désactivation** : départ temporaire → compte désactivé
4. **Suppression** : départ définitif → compte supprimé

**Délégation de contrôle** :
- Permettre au service RH de créer des comptes
- Permettre au help desk de réinitialiser des mots de passe
- Sans donner les droits d'administrateur complets

> [!info] Principe du moindre privilège
> AD permet d'appliquer le principe de sécurité du moindre privilège : donner uniquement les droits nécessaires pour accomplir une tâche, rien de plus.

---

## LDAP et Kerberos

Active Directory s'appuie sur deux protocoles essentiels : **LDAP** pour l'accès à l'annuaire et **Kerberos** pour l'authentification.

### 📖 LDAP (Lightweight Directory Access Protocol)

#### Qu'est-ce que LDAP ?

**LDAP** est un protocole standard pour accéder et maintenir des services d'annuaire distribués sur un réseau IP.

**Caractéristiques** :
- Protocole **client-serveur**
- Fonctionne sur TCP/IP
- Port par défaut : **389** (LDAP), **636** (LDAPS - sécurisé)
- Basé sur le standard X.500 (simplifié)
- Format des données : structure arborescente

#### Structure des données LDAP

Les données LDAP sont organisées en **arbre hiérarchique** appelé **DIT (Directory Information Tree)**.

**Composants d'une entrée LDAP** :

| Composant | Signification | Exemple |
|-----------|---------------|---------|
| **DC** | Domain Component | dc=entreprise, dc=com |
| **OU** | Organizational Unit | ou=Utilisateurs |
| **CN** | Common Name | cn=Jean Dupont |
| **DN** | Distinguished Name | cn=Jean Dupont,ou=Utilisateurs,dc=entreprise,dc=com |

**Exemple de DN (Distinguished Name)** :
```
CN=Jean Dupont,OU=Marketing,OU=Utilisateurs,DC=entreprise,DC=com
│         │            │              │                │
│         │            │              │                └─ Domaine racine
│         │            │              └─ Sous-domaine
│         │            └─ OU parent
│         └─ OU de l'utilisateur
└─ Nom de l'utilisateur
```

> [!info] DN unique
> Le DN est l'identifiant unique d'un objet dans l'annuaire LDAP. Deux objets ne peuvent pas avoir le même DN.

#### Opérations LDAP

LDAP définit plusieurs opérations pour interagir avec l'annuaire :

| Opération | Description | Utilisation |
|-----------|-------------|-------------|
| **Bind** | Connexion et authentification | S'identifier auprès du serveur |
| **Search** | Recherche d'entrées | Trouver des utilisateurs, groupes |
| **Add** | Ajout d'une entrée | Créer un nouvel utilisateur |
| **Modify** | Modification d'attributs | Changer un numéro de téléphone |
| **Delete** | Suppression d'une entrée | Supprimer un compte |
| **Compare** | Comparer une valeur | Vérifier un mot de passe |
| **Unbind** | Déconnexion | Fermer la session |

**Exemple de requête LDAP (recherche)** :
```ldap
Base DN: dc=entreprise,dc=com
Filter: (&(objectClass=user)(department=IT))
Scope: subtree
Attributes: cn, mail, telephoneNumber
```

Cette requête cherche tous les utilisateurs du département IT et retourne leur nom, email et téléphone.

#### Filtres de recherche LDAP

Les filtres LDAP utilisent une syntaxe spécifique :

| Opérateur | Signification | Exemple |
|-----------|---------------|---------|
| `=` | Égal | `(cn=Jean Dupont)` |
| `>=` | Supérieur ou égal | `(employeeNumber>=1000)` |
| `<=` | Inférieur ou égal | `(employeeNumber<=5000)` |
| `~=` | Approximativement égal | `(cn~=Jon)` |
| `*` | Joker (wildcard) | `(cn=Jean*)` |
| `&` | ET logique | `(&(condition1)(condition2))` |
| `\|` | OU logique | `(\|(dept=IT)(dept=Dev))` |
| `!` | NON logique | `(!(objectClass=computer))` |

> [!example] Exemples de filtres courants
> ```ldap
> # Tous les utilisateurs actifs
> (&(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
> 
> # Utilisateurs dont le nom commence par "A"
> (&(objectClass=user)(cn=A*))
> 
> # Membres du groupe Admins ou Support
> (|(memberOf=CN=Admins,OU=Groups,DC=entreprise,DC=com)
>    (memberOf=CN=Support,OU=Groups,DC=entreprise,DC=com))
> ```

#### LDAP vs LDAPS

**LDAP** (port 389) :
- Communication en **clair**
- Vulnérable aux attaques man-in-the-middle
- Mots de passe visibles sur le réseau

**LDAPS** (port 636) :
- Communication **chiffrée** via SSL/TLS
- Authentification du serveur via certificat
- Protection des données sensibles

> [!warning] Sécurité
> Il est fortement recommandé d'utiliser LDAPS pour toutes les communications avec Active Directory, surtout si le réseau n'est pas entièrement sécurisé.

#### Outils LDAP

**Requêtes LDAP en ligne de commande** :
```bash
# ldapsearch (Linux)
ldapsearch -H ldap://dc01.entreprise.com -b "dc=entreprise,dc=com" \
  -D "cn=admin,dc=entreprise,dc=com" -w password \
  "(objectClass=user)" cn mail

# dsquery (Windows)
dsquery user -name "Jean*"

# PowerShell
Get-ADUser -Filter {Name -like "Jean*"} -Properties mail, telephoneNumber
```

### 🔐 Kerberos

#### Qu'est-ce que Kerberos ?

**Kerberos** est un protocole d'authentification réseau développé au MIT. Il permet une authentification sécurisée dans un environnement non sécurisé sans transmettre de mots de passe sur le réseau.

**Principes fondamentaux** :
- Authentification basée sur des **tickets**
- Utilise la **cryptographie symétrique** (clés partagées)
- Fait confiance à un **tiers de confiance** : le KDC (Key Distribution Center)
- Protège contre les attaques par rejeu (replay attacks)
- Fournit l'authentification mutuelle (client et serveur)

**Composants Kerberos dans AD** :

| Composant | Rôle | Localisation |
|-----------|------|--------------|
| **KDC** | Key Distribution Center | Contrôleur de domaine |
| **AS** | Authentication Service | Partie du KDC |
| **TGS** | Ticket Granting Service | Partie du KDC |
| **Client** | Utilisateur ou service | Poste de travail |
| **Service** | Ressource demandée | Serveur de fichiers, etc. |

#### Processus d'authentification Kerberos

Le processus d'authentification Kerberos se déroule en plusieurs étapes :

**1. Demande de TGT (Ticket Granting Ticket)** :
```
Client → AS (Authentication Service)
    Envoie : Nom d'utilisateur, timestamp chiffré
    
AS → Client
    Envoie : TGT chiffré avec la clé du TGS
            Session Key chiffrée avec la clé de l'utilisateur
```

**2. Demande de Service Ticket** :
```
Client → TGS (Ticket Granting Service)
    Envoie : TGT, nom du service demandé, authenticator
    
TGS → Client
    Envoie : Service Ticket chiffré avec la clé du service
            Session Key chiffrée pour la communication avec le service
```

**3. Accès au service** :
```
Client → Service
    Envoie : Service Ticket, authenticator
    
Service → Client
    Envoie : Confirmation d'authentification (optionnel)
```

> [!info] TGT = Pass universel
> Le TGT fonctionne comme un "pass universel" : une fois obtenu, il peut être utilisé pour demander l'accès à n'importe quel service sans renvoyer le mot de passe.

**Schéma simplifié** :
```
1. Connexion utilisateur
   ↓
2. Obtention TGT (valide 10h par défaut)
   ↓
3. Accès à un partage réseau
   ↓
4. Utilisation du TGT pour obtenir un Service Ticket
   ↓
5. Accès au partage avec le Service Ticket
   ↓
6. Accès à une imprimante (réutilise le TGT)
   ↓
7. Nouveau Service Ticket pour l'imprimante
   ↓
8. Impression réussie
```

#### Chiffrement Kerberos

Kerberos utilise la cryptographie symétrique avec différentes clés :

| Clé | Dérivée de | Usage |
|-----|------------|-------|
| **Clé utilisateur** | Hash du mot de passe | Chiffrer la session key AS |
| **Clé TGS** | Secret du KDC | Chiffrer le TGT |
| **Clé service** | Hash du mot de passe du service | Chiffrer le Service Ticket |
| **Session Key** | Générée aléatoirement | Communication temporaire |

**Algorithmes de chiffrement supportés** :
- AES256-SHA1
- AES128-SHA1
- RC4-HMAC (ancienne méthode, déconseillée)
- DES (obsolète, désactivé par défaut)

> [!warning] RC4 déprécié
> Microsoft recommande de désactiver RC4 et d'utiliser uniquement AES pour une sécurité optimale. RC4 est vulnérable aux attaques modernes.

#### Durée de vie des tickets

Les tickets Kerberos ont une durée de validité limitée pour des raisons de sécurité :

| Ticket | Durée par défaut | Renouvellement maximum |
|--------|------------------|------------------------|
| **TGT** | 10 heures | 7 jours |
| **Service Ticket** | 10 heures | - |

**Pourquoi une durée limitée ?** :
- Limiter la fenêtre d'exploitation si un ticket est volé
- Forcer la réauthentification régulière
- Permettre la révocation des droits (expire naturellement)

Le TGT peut être **renouvelé automatiquement** sans redemander le mot de passe, tant qu'on reste dans la période de renouvellement (7 jours par défaut).

#### Avantages de Kerberos

✅ **Sécurité renforcée** :
- Mots de passe jamais transmis sur le réseau
- Chiffrement fort de toutes les communications
- Protection contre les attaques par rejeu (timestamps)
- Authentification mutuelle (client ↔ serveur)

✅ **Performance** :
- TGT réutilisable pour plusieurs services
- Pas de vérification centralisée à chaque accès
- Cache local des tickets

✅ **Interopérabilité** :
- Standard ouvert (RFC 4120)
- Supporté par Windows, Linux, macOS
- Compatible avec de nombreuses applications

✅ **Single Sign-On (SSO)** :
- Une authentification pour toutes les ressources
- Expérience utilisateur transparente
- Réduction des demandes de mot de passe

#### Kerberos vs NTLM

Active Directory supporte aussi NTLM (NT LAN Manager), l'ancien protocole d'authentification de Windows. Voici une comparaison :

| Critère | Kerberos | NTLM |
|---------|----------|------|
| **Sécurité** | Forte (AES, aucun mot de passe sur le réseau) | Faible (MD4, vulnérable) |
| **Authentification mutuelle** | Oui | Non |
| **Performance** | Meilleure (tickets réutilisables) | Moins bonne |
| **SSO** | Oui | Limité |
| **Délégation** | Oui (contrainte et non contrainte) | Non |
| **Interopérabilité** | Standard ouvert | Propriétaire Microsoft |
| **Recommandation** | **Préféré** | Héritage uniquement |

> [!tip] Désactiver NTLM
> Pour des raisons de sécurité, Microsoft recommande de désactiver NTLM et d'utiliser exclusivement Kerberos dans les environnements modernes.

#### SPN (Service Principal Name)

Pour que Kerberos fonctionne, chaque service doit avoir un **SPN** enregistré dans Active Directory.

**Format d'un SPN** :
```
ServiceClass/Host:Port/ServiceName
```

**Exemples** :
```
HTTP/webserver.entreprise.com
MSSQLSvc/sqlserver.entreprise.com:1433
HOST/fileserver.entreprise.com
```

**Commandes pour gérer les SPN** :
```powershell
# Lister les SPN d'un compte
setspn -L compte_service

# Ajouter un SPN
setspn -A HTTP/webapp.entreprise.com compte_service

# Supprimer un SPN
setspn -D HTTP/webapp.entreprise.com compte_service

# Vérifier les SPN dupliqués
setspn -X
```

> [!warning] SPN dupliqués
> Un SPN ne peut être enregistré qu'une seule fois dans AD. Des SPN dupliqués causent des échecs d'authentification Kerberos.

#### Délégation Kerberos

La **délégation** permet à un service d'agir au nom d'un utilisateur pour accéder à d'autres services.

**Types de délégation** :

1. **Non contrainte (Unconstrained)** :
   - Le service peut accéder à **n'importe quel** service au nom de l'utilisateur
   - ⚠️ Risque de sécurité élevé
   - Déconseillé

2. **Contrainte (Constrained)** :
   - Le service peut accéder uniquement aux services **spécifiés**
   - Plus sécurisé
   - Recommandé

3. **Contrainte basée sur les ressources (Resource-Based)** :
   - Contrôle depuis la ressource cible
   - Plus flexible pour les environnements multi-forêts
   - Introduction dans Windows Server 2012

> [!example] Cas d'usage : Délégation
> Un serveur web doit accéder à une base de données SQL Server au nom de l'utilisateur connecté :
> 1. L'utilisateur s'authentifie auprès du serveur web
> 2. Le serveur web reçoit un ticket avec délégation
> 3. Le serveur web utilise ce ticket pour s'authentifier auprès de SQL Server
> 4. SQL Server voit l'identité de l'utilisateur final, pas celle du serveur web

---

## 🎯 Récapitulatif

Active Directory est bien plus qu'un simple annuaire d'utilisateurs. C'est un système complet de gestion des identités et des accès qui :

✅ **Organise** les ressources réseau dans une structure hiérarchique (forêt → arborescence → domaine)

✅ **Centralise** l'administration via les contrôleurs de domaine avec réplication multi-maître

✅ **Sécurise** l'authentification et l'autorisation grâce à Kerberos et LDAP

✅ **Scale** de quelques utilisateurs à des millions d'objets sans dégradation

✅ **Assure** la haute disponibilité avec tolérance aux pannes et réplication automatique

✅ **Facilite** la gestion via les stratégies de groupe (GPO) et la délégation de contrôle

**Les deux protocoles fondamentaux** :
- **LDAP** : accès et interrogation de l'annuaire
- **Kerberos** : authentification sécurisée sans transmission de mots de passe

> [!tip] Prochaines étapes
> Cette introduction pose les bases conceptuelles d'Active Directory. Les parties suivantes approfondiront :
> - L'installation et la configuration d'AD DS
> - La gestion des objets (utilisateurs, groupes, ordinateurs)
> - Les stratégies de groupe (GPO)
> - La sécurisation et l'audit
> - La maintenance et la résolution de problèmes

---

*Cours créé pour Obsidian - Active Directory - Partie 5*