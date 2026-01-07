# Les utilisateurs - Partie 2
## Gestion des utilisateurs

---

**Formation TSSR - Titre RNCP**  
**Date de révision :** Novembre 2025  
**Contexte :** Préparation titre Technicien Supérieur Systèmes et Réseaux

---

## 📋 Sommaire

1. [La sécurité](#la-sécurité)
   - [IAM (Identity and Access Management)](#gestion-des-identités-et-des-accès-iam)
   - [Identification](#identification)
   - [Authentification](#authentification)
2. [Gestion des utilisateurs GNU/Linux](#gestion-des-utilisateurs-gnulinux)
   - [Le fichier /etc/passwd](#liste-des-utilisateurs)
   - [Le fichier /etc/shadow](#base-des-mots-de-passe)
   - [Le fichier /etc/group](#base-des-groupes)
   - [Commandes Linux](#administration-des-utilisateurs)
3. [Gestion des utilisateurs Windows](#gestion-des-utilisateurs-windows)
   - [Get-LocalUser](#liste-des-utilisateurs-1)
   - [La base SAM](#la-base-sam)
   - [Get-LocalGroup](#liste-des-groupes)
   - [Commandes PowerShell](#administration-des-utilisateurs-1)
4. [Points clés à retenir](#points-clés-à-retenir)
5. [Glossaire technique](#glossaire-technique)

---

## 🔍 Questions fondamentales

Avant de commencer, réfléchissons à ces questions :

- ❓ Que pouvez-vous dire sur la **sécurité des utilisateurs** sur les systèmes **Windows** ?
- ❓ Et sur **Linux** ?

---

## 🔐 La sécurité

### Gestion des identités et des accès (IAM)

#### Comment faire ?

> **Définition :** Pratique qui consiste à **s'assurer que les personnes et les entités ayant une identité numérique** ont le **bon niveau d'accès** aux ressources de l'entreprise (réseaux et BDD).

**Les rôles d'utilisateur et les privilèges d'accès sont définis et gérés par un système IAM.**

---

#### 🎯 Composantes de l'IAM

| Composante | Description |
|------------|-------------|
| **Identités** | Utilisateurs, services, applications |
| **Authentification** | Vérification de l'identité |
| **Autorisation** | Définition des accès autorisés |
| **Audit** | Traçabilité des actions |
| **Gouvernance** | Gestion des politiques de sécurité |

---

#### 💡 Objectifs de l'IAM

1. ✅ **Sécurité** : Contrôler qui accède à quoi
2. ✅ **Conformité** : Respecter les règlementations (RGPD, etc.)
3. ✅ **Productivité** : Simplifier l'accès aux ressources
4. ✅ **Audit** : Tracer les accès et actions
5. ✅ **Gestion centralisée** : Administrer depuis un point unique

---

### Identification

#### Qui suis-je ?

> **Définition :** Étape **indispensable** où l'on doit **enregistrer l'identité de l'utilisateur**. Avant de pouvoir se connecter à son compte, il doit entrer un **identifiant (login)**.

**Cette information est :**
- ✅ Un renseignement attribué **à titre individuel**
- ✅ **Unique** dans le système

---

#### 📝 Exemples d'identifiants

| Type | Exemple | Particularité |
|------|---------|---------------|
| **Nom d'utilisateur** | `jdupont`, `marie.martin` | Le plus courant |
| **Email** | `user@example.com` | Utilisé pour les services web |
| **Numéro** | `1000`, `UID:1001` | Identifiant système |
| **Badge** | `A12345` | Identification physique |

> ⚠️ **Important :** L'identification seule **ne prouve pas** l'identité, elle la **déclare** seulement.

---

### Authentification

#### Vous pouvez rentrer !

> **Définition :** **Cumule l'identification et l'authentification** afin d'accéder à un service. Consiste à **vérifier qu'une tentative de connexion est légitime**. L'autorisation est accordée **après une authentification réussie**.

---

#### 🔑 Processus complet

```
┌─────────────────┐
│ IDENTIFICATION  │  "Je suis jdupont"
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│AUTHENTIFICATION │  "Voici mon mot de passe"
└────────┬────────┘
         │
         ▼ (Vérification)
         │
    ┌────┴────┐
    │  OK ?   │
    └────┬────┘
         │
    ┌────┴─────────┐
    │              │
 ✅ Oui         ❌ Non
    │              │
    ▼              ▼
  ACCÈS        REFUS
  AUTORISÉ     D'ACCÈS
```

---

#### 🎯 Facteurs d'authentification

| Facteur | Description | Exemple |
|---------|-------------|---------|
| **Ce que je sais** | Connaissance | Mot de passe, PIN, question secrète |
| **Ce que je possède** | Possession | Carte à puce, token, smartphone |
| **Ce que je suis** | Biométrie | Empreinte digitale, reconnaissance faciale |
| **Où je suis** | Localisation | Adresse IP, géolocalisation |

> 💡 **MFA (Multi-Factor Authentication)** : Combiner 2 facteurs ou plus pour plus de sécurité

---

## 🐧 Gestion des utilisateurs GNU/Linux

### Liste des utilisateurs

#### La liste des utilisateurs locaux

**Fichier : `/etc/passwd`**

**Caractéristiques :**
- ✅ Fichier **texte**
- ✅ **1 ligne par utilisateur**
- ✅ **7 colonnes** séparées par `:`

---

#### 📊 Structure du fichier /etc/passwd

| Colonne | Nom | Description |
|---------|-----|-------------|
| **1** | **Login** | Nom de connexion |
| **2** | **Password** | Validation du mot de passe (`x` ou `*`) |
| **3** | **UID** | Identifiant utilisateur (numérique) |
| **4** | **GID** | Identifiant de groupe principal (numérique) |
| **5** | **GECOS** | Commentaire, description, nom complet |
| **6** | **Home** | Répertoire personnel (home directory) |
| **7** | **Shell** | Shell de lancement |

> 📌 **Convention :** `root` a toujours **UID = 0**

---

### Exemple de fichier passwd

#### cat /etc/passwd

```bash
wilder@Ubuntu:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
wilder:x:1000:1000:Some Heroic Wilder,,,:/home/wilder:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:126:65534::/run/sshd:/usr/sbin/nologin
```

---

#### 🔍 Analyse ligne par ligne

**Ligne root :**
```
root:x:0:0:root:/root:/bin/bash
 │   │ │ │  │    │      └─ Shell : /bin/bash
 │   │ │ │  │    └────────── Home : /root
 │   │ │ │  └─────────────── GECOS : root
 │   │ │ └────────────────── GID : 0
 │   │ └──────────────────── UID : 0
 │   └─────────────────────── Password : x (dans /etc/shadow)
 └─────────────────────────── Login : root
```

**Ligne wilder :**
```
wilder:x:1000:1000:Some Heroic Wilder,,,:/home/wilder:/bin/bash
  │    │  │    │          │                │             └─ Shell interactif
  │    │  │    │          │                └───────────── Home personnel
  │    │  │    │          └────────────────────────────── Description
  │    │  │    └───────────────────────────────────────── Groupe principal
  │    │  └────────────────────────────────────────────── UID utilisateur
  │    └───────────────────────────────────────────────── Mot de passe chiffré ailleurs
  └────────────────────────────────────────────────────── Nom de connexion
```

---

#### 🤖 Comptes système vs utilisateurs

| Type | UID | Shell | Exemple |
|------|-----|-------|---------|
| **root** | 0 | `/bin/bash` | Administrateur |
| **Système** | 1-999 | `/usr/sbin/nologin` | `daemon`, `www-data`, `sshd` |
| **Utilisateur** | 1000+ | `/bin/bash` | `wilder` |

> 💡 **Shell `/usr/sbin/nologin`** : Empêche la connexion interactive (pour les services)

---

### Administration des utilisateurs

#### Quelques commandes utiles

| Commande | Fonction | Exemple |
|----------|----------|---------|
| `passwd` | Modifier un mot de passe | `passwd wilder` |
| `adduser` | Ajout d'utilisateurs | `adduser newuser` |
| `deluser` | Suppression d'utilisateurs | `deluser olduser` |
| `usermod` | Modifier un utilisateur | `usermod -aG sudo wilder` |
| `chfn` | Modifier la description (GECOS) | `chfn wilder` |
| `chsh` | Modifier le shell par défaut | `chsh -s /bin/zsh wilder` |
| `chage` | Modifier durée de validité | `chage -M 90 wilder` |
| `newusers` | Création d'utilisateurs par lot | `newusers users.txt` |
| `pwck` | Vérification du format des fichiers | `pwck` |

---

#### 📝 Exemples de commandes

```bash
# Créer un utilisateur
sudo adduser john
# Crée automatiquement : home, groupe, demande mot de passe

# Modifier le shell
sudo chsh -s /bin/zsh john

# Ajouter à un groupe
sudo usermod -aG sudo john

# Modifier le GECOS (nom complet)
sudo chfn -f "John Doe" john

# Changer le mot de passe
sudo passwd john

# Verrouiller un compte
sudo passwd -l john

# Déverrouiller un compte
sudo passwd -u john

# Supprimer un utilisateur (garder le home)
sudo deluser john

# Supprimer un utilisateur ET son home
sudo deluser --remove-home john

# Expirer le mot de passe (force le changement)
sudo passwd -e john
```

---

### Base des mots de passe

#### Les empreintes de mots de passe

**Fichier : `/etc/shadow`**

**Caractéristiques :**
- ✅ Fichier **texte**
- ✅ **1 ligne par utilisateur**
- ✅ **9 colonnes** séparées par `:`
- ✅ **Lisible uniquement par root**

---

#### 📊 Structure du fichier /etc/shadow

| Colonne | Nom | Description |
|---------|-----|-------------|
| **1** | **Login** | Nom de connexion |
| **2** | **Password** | Mot de passe chiffré (+ sel) - `!` ou `*` = connexion impossible |
| **3** | **Lastchange** | Date de dernière modification (jours depuis 01/01/1970) |
| **4** | **Minimum** | Nombre de jours minimum entre changements |
| **5** | **Maximum** | Nombre de jours maximum de validité |
| **6** | **Warn** | Nombre de jours d'avertissement avant expiration |
| **7** | **Inactive** | Nombre de jours de tolérance après expiration |
| **8** | **Expire** | Date de fin de validité du compte |
| **9** | **Reserved** | Champ sans utilisation actuelle |

---

### Exemple de fichier shadow

#### cat /etc/shadow

```bash
wilder@Ubuntu:~$ sudo cat /etc/shadow
root:!:19081:0:99999:7:::
daemon:*:18912:0:99999:7:::
bin:*:18912:0:99999:7:::
man:*:18912:0:99999:7:::
lp:*:18912:0:99999:7:::
nobody:*:18912:0:99999:7:::
systemd-timesync:*:18912:0:99999:7:::
messagebus:*:18912:0:99999:7:::
syslog:*:18912:0:99999:7:::
wilder:$6$Je34t19kkZ2ZGs9f$PleinDeCaracteresCryptiques:19081:0:99999:7:::
sshd:*:19090:0:99999:7:::
```

---

#### 🔍 Analyse du mot de passe

**Format du champ password :**
```
$id$salt$hash

$6$Je34t19kkZ2ZGs9f$PleinDeCaracteresCryptiques
│   │               │
│   │               └─ Hash du mot de passe
│   └───────────────── Sel (salt) - aléatoire
└───────────────────── ID de l'algorithme
```

**Algorithmes de hachage :**

| ID | Algorithme | Sécurité |
|----|------------|----------|
| `$1$` | MD5 | ❌ Obsolète |
| `$2a$` | Blowfish | ⚠️ Acceptable |
| `$5$` | SHA-256 | ✅ Bon |
| `$6$` | SHA-512 | ✅✅ Recommandé |
| `$y$` | yescrypt | ✅✅✅ Moderne |

---

#### 🔐 Valeurs spéciales du champ password

| Valeur | Signification |
|--------|---------------|
| `!` ou `*` | **Compte verrouillé** - Connexion impossible |
| `!!` | **Pas de mot de passe défini** - Connexion impossible |
| `(vide)` | **Pas de mot de passe** - Connexion sans mot de passe (dangereux !) |

---

#### 📅 Gestion de l'expiration

```bash
# Ligne exemple :
wilder:$6$...:19081:0:99999:7:::
           │    │   │    │   │
           │    │   │    │   └─ Inactive : non défini
           │    │   │    └───── Warn : 7 jours
           │    │   └────────── Maximum : 99999 jours
           │    └────────────── Minimum : 0 jour
           └─────────────────── Lastchange : jour 19081 (≈ 2022)
```

**Commandes d'expiration :**
```bash
# Afficher les infos d'expiration
sudo chage -l wilder

# Forcer le changement au prochain login
sudo chage -d 0 wilder

# Définir expiration du mot de passe (90 jours)
sudo chage -M 90 wilder

# Définir avertissement (7 jours avant expiration)
sudo chage -W 7 wilder

# Définir expiration du compte (date)
sudo chage -E 2025-12-31 wilder
```

---

### Base des groupes

#### La liste des groupes

**Fichier : `/etc/group`**

**Caractéristiques :**
- ✅ Fichier **texte**
- ✅ **1 ligne par groupe**
- ✅ **4 colonnes** séparées par `:`

---

#### 📊 Structure du fichier /etc/group

| Colonne | Nom | Description |
|---------|-----|-------------|
| **1** | **Groupname** | Nom de groupe |
| **2** | **Password** | Mot de passe (x, *) - rarement utilisé |
| **3** | **GID** | Identifiant de groupe (numérique) |
| **4** | **Members** | Liste des membres du groupe (séparés par `,`) |

> 📌 **Convention :** `root` a toujours **GID = 0**

---

### Exemple de fichier group

#### cat /etc/group

```bash
wilder@Ubuntu:~$ cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
tape:x:26:
sudo:x:27:wilder
users:x:100:
nogroup:x:65534:
crontab:x:105:
nopasswdlogin:x:124:
wilder:x:1000:
sambashare:x:135:wilder
```

---

#### 🔍 Analyse ligne par ligne

**Ligne sudo :**
```
sudo:x:27:wilder
 │   │ │   └─ Membres : wilder
 │   │ └───── GID : 27
 │   └─────── Password : x (non utilisé)
 └─────────── Groupname : sudo
```

**Ligne sambashare :**
```
sambashare:x:135:wilder
    │      │  │    └─ Membres : wilder
    │      │  └────── GID : 135
    │      └───────── Password : x
    └──────────────── Groupname : sambashare
```

---

#### 🎯 Groupes système importants

| Groupe | GID | Description |
|--------|-----|-------------|
| `root` | 0 | Super-utilisateur |
| `sudo` | 27 | Utilisateurs pouvant utiliser sudo |
| `adm` | 4 | Accès aux logs système |
| `cdrom` | 24 | Accès aux lecteurs CD |
| `plugdev` | 46 | Montage de périphériques |
| `lpadmin` | 122 | Administration des imprimantes |

---

### Administration des groupes

#### D'autres commandes utiles

| Commande | Fonction | Exemple |
|----------|----------|---------|
| `newgrp` | Prendre un nouveau groupe | `newgrp developers` |
| `groupadd` | Ajout d'un groupe | `groupadd devops` |
| `groupdel` | Suppression d'un groupe | `groupdel oldgroup` |
| `groupmod` | Modifier un groupe | `groupmod -n newname oldname` |
| `grpck` | Vérification du format | `grpck` |
| `gpasswd` | Gérer les mots de passe de groupe | `gpasswd -a user group` |

---

#### 📝 Exemples de commandes

```bash
# Créer un groupe
sudo groupadd developers

# Ajouter un utilisateur à un groupe
sudo usermod -aG developers john
# ou
sudo gpasswd -a john developers

# Retirer un utilisateur d'un groupe
sudo gpasswd -d john developers

# Changer le GID d'un groupe
sudo groupmod -g 2000 developers

# Renommer un groupe
sudo groupmod -n devops developers

# Supprimer un groupe
sudo groupdel oldgroup

# Afficher les groupes d'un utilisateur
groups john
# ou
id john

# Changer de groupe principal (temporaire)
newgrp developers
```

---

### Interagir avec les utilisateurs

#### Encore plus de commandes !!!

| Commande | Fonction | Exemple |
|----------|----------|---------|
| `id` | Affiche ses UID/GID et groupes | `id` ou `id wilder` |
| `whoami` | Alias de `id -un` (nom utilisateur) | `whoami` |
| `who` | Affiche les utilisateurs connectés | `who` |
| `w` | Affiche qui fait quoi | `w` |
| `last` | Historique des connexions | `last` |
| `su` | Changer d'utilisateur | `su - john` |
| `sudo` | Lancer une commande avec un autre UID | `sudo apt update` |
| `exit` | Quitter une session | `exit` |
| `logout` | Quitter une session login | `logout` |

---

#### 📝 Exemples pratiques

```bash
# Afficher mes informations
id
# Résultat : uid=1000(wilder) gid=1000(wilder) groupes=1000(wilder),27(sudo)

# Afficher mon nom
whoami
# Résultat : wilder

# Voir qui est connecté
who
# Résultat :
# wilder   tty7         2025-11-19 09:30 (:0)
# john     pts/0        2025-11-19 10:15 (192.168.1.10)

# Voir qui fait quoi
w
# Affiche : utilisateurs + processus + charge système

# Historique des connexions
last
# Affiche les dernières connexions

# Changer d'utilisateur (avec environnement)
su - john
# Demande le mot de passe de john

# Devenir root
sudo -i
# ou
sudo su -

# Exécuter une commande en tant qu'autre utilisateur
sudo -u john whoami
# Résultat : john

# Lancer un shell en tant que root
sudo -s
```

---

## 🪟 Gestion des utilisateurs Windows

### Liste des utilisateurs

#### Comment les avoir ?

**Les utilisateurs qui ont eu un compte activé au moins 1 fois ont leur dossier de profil dans `C:\Users`**

**Méthode basique : `Get-LocalUser`**

**Format de sortie :**
- ✅ **1 ligne par utilisateur**
- ✅ **3 colonnes** :
  - Nom
  - Activation (Enabled)
  - Description

---

### Exemple de liste utilisateurs

#### Liste simple

```powershell
PS C:\> Get-LocalUser

Name                  Enabled  Description                                                                                               
----                  -------  -----------                                                                                               
Administrateur        False    Compte d'utilisateur d'administration                                                                     
wilder                True     Compte utilisateur de test                                                                                         
DefaultAccount        False    Compte utilisateur géré par le système.                                                                   
Invité                False    Compte d'utilisateur invité                                                                               
WDAGUtilityAccount    False    Compte d'utilisateur géré et utilisé par le …
```

---

#### 🔍 Comptes Windows standards

| Compte | Activé | Fonction |
|--------|--------|----------|
| **Administrateur** | Non (par défaut) | Compte admin local |
| **Invité** | Non | Accès temporaire limité |
| **DefaultAccount** | Non | Compte système (Windows 10+) |
| **WDAGUtilityAccount** | Non | Windows Defender Application Guard |

> 💡 **Bonne pratique :** Le compte Administrateur est désactivé par défaut pour des raisons de sécurité

---

### Liste des utilisateurs

#### La méthode détaillée

**Méthode WMI : `Get-WmiObject Win32_UserAccount`**

```powershell
Get-WmiObject Win32_UserAccount -Filter "LocalAccount='True'" | Format-Table -AutoSize
```

**Format de sortie :**
- ✅ **1 ligne par utilisateur**
- ✅ **6 colonnes** :
  - Type de compte
  - Légende (emplacement)
  - Domaine
  - SID
  - Nom complet
  - Nom

---

### Exemple de liste utilisateurs

#### La liste détaillée

```powershell
PS C:\> Get-WmiObject Win32_UserAccount -Filter "LocalAccount='True'" | Format-Table -AutoSize

AccountType  Caption                    Domain  SID                                                   FullName        Name              
-----------  -------                    ------  ---                                                   --------        ----              
512          PCLab\Administrateur       PCLab   S-1-5-21-3909285403-2394092363-769350273-500                          Administrateur    
512          PCLab\wilder               PCLab   S-1-5-21-3909285403-2394092363-769350273-1001         wilder wilder   wilder             
512          PCLab\DefaultAccount       PCLab   S-1-5-21-3909285403-2394092363-769350273-503                          DefaultAccount    
512          PCLab\Invité               PCLab   S-1-5-21-3909285403-2394092363-769350273-501                          Invité
```

---

#### 📊 AccountType

| Valeur | Type de compte |
|--------|----------------|
| **256** | Compte temporaire |
| **512** | Compte normal |
| **2048** | Compte d'approbation |
| **4096** | Compte ordinateur |

---

### La base SAM

#### La méthode détaillée

> **Définition :** Le **Gestionnaire de comptes de sécurité (SAM)** est une **base de données** qui est présente sur les ordinateurs exécutant un OS Windows.

**Fonction :**
- ✅ Stocke les **comptes d'utilisateur**
- ✅ Stocke les **descripteurs de sécurité** pour les utilisateurs sur l'ordinateur local

**Emplacement :**
```
%SystemRoot%\system32\Config\SAM
```

**En général :**
```
C:\Windows\System32\Config\SAM
```

---

#### 🔐 Sécurité de la base SAM

**Protection :**
- ✅ **Fichier verrouillé** pendant que Windows fonctionne
- ✅ **Accessible uniquement** par le système (SYSTEM)
- ✅ **Chiffré** avec une clé dérivée du système
- ✅ **Sauvegardé** dans les points de restauration

**Vulnérabilités :**
- ⚠️ Peut être copié en mode offline (boot sur autre OS)
- ⚠️ Peut être attaqué avec des outils spécialisés
- ⚠️ Hachages extractibles pour attaque par dictionnaire

> 🛡️ **Protection :** Utiliser des mots de passe forts + BitLocker pour chiffrer le disque

---

### Les mots de passe en GUI

#### La méthode graphique

**Méthode 1 : Via le menu Windows**
- Aller dans le **Gestionnaire d'identification**
- Panneau de configuration > Comptes d'utilisateurs > Gestionnaire d'identification

**Méthode 2 : En ligne de commande**
```cmd
rundll32.exe keymgr.dll,KRShowKeyMgr
```

**Affichage :**
- Informations d'identification Windows
- Informations d'identification web
- Informations d'identification génériques

---

### Liste des groupes

#### La liste des groupes

**Méthode basique : `Get-LocalGroup`**

**Format de sortie :**
- ✅ **1 ligne par groupe**
- ✅ **2 colonnes** :
  - Nom
  - Description

---

### Exemple de listing de groupes

#### La manière rapide

```powershell
PS C:\> Get-LocalGroup

Name                             Description                                                                                                                          
----                             -----------                                                                                                                          
Administrateurs                  Les membres du groupe Administrateurs dispo…                         
Administrateurs Hyper-V          Les membres de ce groupe disposent d'un acc…                          
Duplicateurs                     Prend en charge la réplication des fichiers dans…                                                                          
IIS_IUSRS                        Groupe intégré utilisé par les services Internet (IIS).                                                                              
Invités                          Les membres du groupe Invités disposent par déf…
```

---

### Liste des groupes

#### La manière détaillée

**Méthode WMI : `Get-WmiObject Win32_group`**

**Format de sortie :**
- ✅ **1 ligne par groupe**
- ✅ **4 colonnes** :
  - Légende (emplacement)
  - Domaine
  - Nom
  - SID

---

### Exemple de listing de groupes

#### La liste détaillée

```powershell
PS C:\> Get-WmiObject win32_group

Caption                                           Domain  Name                                         SID         
-------                                           ------  ----                                         ---         
PCLab\Administrateurs                             PCLab   Administrateurs                              S-1-5-32-544
PCLab\Administrateurs Hyper-V                     PCLab   Administrateurs Hyper-V                      S-1-5-32-578
PCLab\Duplicateurs                                PCLab   Duplicateurs                                 S-1-5-32-552
PCLab\IIS_IUSRS                                   PCLab   IIS_IUSRS                                    S-1-5-32-568
PCLab\Invités                                     PCLab   Invités                                      S-1-5-32-546
PCLab\Opérateurs de chiffrement                   PCLab   Opérateurs de chiffrement                    S-1-5-32-569
PCLab\Opérateurs de configuration réseau          PCLab   Opérateurs de configuration réseau           S-1-5-32-556
PCLab\Opérateurs de sauvegarde                    PCLab   Opérateurs de sauvegarde                     S-1-5-32-551
PCLab\Propriétaires d'appareils                   PCLab   Propriétaires d'appareils                    S-1-5-32-583
PCLab\System Managed Accounts Group               PCLab   System Managed Accounts Group                S-1-5-32-581
PCLab\Utilisateurs                                PCLab   Utilisateurs                                 S-1-5-32-545
PCLab\Utilisateurs avec pouvoir                   PCLab   Utilisateurs avec pouvoir                    S-1-5-32-547
PCLab\Utilisateurs de gestion à distance          PCLab   Utilisateurs de gestion à distance           S-1-5-32-580
PCLab\Utilisateurs du Bureau à distance           PCLab   Utilisateurs du Bureau à distance            S-1-5-32-555
```

---

#### 🎯 Groupes Windows importants

| Groupe | SID | Description |
|--------|-----|-------------|
| **Administrateurs** | S-1-5-32-544 | Contrôle total du système |
| **Utilisateurs** | S-1-5-32-545 | Utilisateurs standards |
| **Invités** | S-1-5-32-546 | Accès limité temporaire |
| **Utilisateurs avec pouvoir** | S-1-5-32-547 | Privilèges limités d'admin |
| **Opérateurs de sauvegarde** | S-1-5-32-551 | Sauvegarde et restauration |
| **Utilisateurs du Bureau à distance** | S-1-5-32-555 | Connexion RDP |
| **Opérateurs de configuration réseau** | S-1-5-32-556 | Configuration réseau |

---

### Administration des utilisateurs

#### Quelques commandes utiles

| Commande | Fonction |
|----------|----------|
| `Disable-LocalUser` | Désactiver un compte utilisateur |
| `Enable-LocalUser` | Activer un compte utilisateur |
| `Get-LocalUser` | Lister les comptes utilisateurs locaux |
| `New-LocalUser` | Créer un nouveau compte utilisateur local |
| `Remove-LocalUser` | Supprimer un compte utilisateur local |
| `Rename-LocalUser` | Renommer un compte utilisateur |
| `Set-LocalUser` | Modifier un compte utilisateur |

---

#### 📝 Exemples - Gestion utilisateurs

```powershell
# Créer un utilisateur
New-LocalUser -Name "john" -Password (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -FullName "John Doe" -Description "Utilisateur de test"

# Activer un utilisateur
Enable-LocalUser -Name "john"

# Désactiver un utilisateur
Disable-LocalUser -Name "john"

# Modifier la description
Set-LocalUser -Name "john" -Description "Développeur"

# Changer le mot de passe
$password = Read-Host -AsSecureString "Nouveau mot de passe"
Set-LocalUser -Name "john" -Password $password

# Forcer le changement de mot de passe au prochain login
Set-LocalUser -Name "john" -PasswordNeverExpires $false

# Renommer un utilisateur
Rename-LocalUser -Name "john" -NewName "john.doe"

# Supprimer un utilisateur
Remove-LocalUser -Name "john"

# Afficher les détails d'un utilisateur
Get-LocalUser -Name "john" | Format-List *
```

---

### Administration des utilisateurs

#### D'autres commandes utiles

| Commande | Fonction |
|----------|----------|
| `Get-LocalGroup` | Lister les groupes de sécurité locaux |
| `New-LocalGroup` | Créer un nouveau groupe de sécurité local |
| `Remove-LocalGroup` | Supprimer un groupe de sécurité |
| `Rename-LocalGroup` | Renommer un groupe |
| `Set-LocalGroup` | Modifier un groupe |
| `Add-LocalGroupMember` | Ajouter un membre dans un groupe local |
| `Get-LocalGroupMember` | Récupérer les membres d'un groupe local |
| `Remove-LocalGroupMember` | Retirer un membre d'un groupe |

---

#### 📝 Exemples - Gestion groupes

```powershell
# Créer un groupe
New-LocalGroup -Name "Developers" -Description "Équipe de développement"

# Ajouter un utilisateur à un groupe
Add-LocalGroupMember -Group "Developers" -Member "john"

# Ajouter plusieurs utilisateurs
Add-LocalGroupMember -Group "Developers" -Member "john", "marie", "paul"

# Voir les membres d'un groupe
Get-LocalGroupMember -Group "Developers"

# Retirer un membre
Remove-LocalGroupMember -Group "Developers" -Member "john"

# Modifier la description
Set-LocalGroup -Name "Developers" -Description "Équipe de développement logiciel"

# Renommer un groupe
Rename-LocalGroup -Name "Developers" -NewName "DevTeam"

# Supprimer un groupe
Remove-LocalGroup -Name "DevTeam"

# Lister tous les groupes dont un utilisateur est membre
Get-LocalGroup | Where-Object { 
    (Get-LocalGroupMember -Group $_.Name).Name -contains "PCLab\john" 
}
```

---

## 🎯 Points clés à retenir

### ✅ Concepts de sécurité

- [ ] **IAM** : Gestion des identités et des accès
- [ ] **Identification** : Déclaration de l'identité (login)
- [ ] **Authentification** : Vérification de l'identité (mot de passe)
- [ ] **Autorisation** : Définition des accès autorisés
- [ ] **MFA** : Authentification multi-facteurs (ce que je sais + possède + suis)

### ✅ Linux - Fichiers utilisateurs

- [ ] `/etc/passwd` : Liste des utilisateurs (7 colonnes)
  - Format : `login:x:UID:GID:GECOS:home:shell`
  - Convention : root = UID 0
- [ ] `/etc/shadow` : Mots de passe chiffrés (9 colonnes)
  - Lisible uniquement par root
  - Format : `login:password:lastchange:min:max:warn:inactive:expire:`
  - `!` ou `*` = compte verrouillé
- [ ] `/etc/group` : Liste des groupes (4 colonnes)
  - Format : `groupname:x:GID:members`

### ✅ Linux - Commandes utilisateurs

- [ ] `adduser` / `useradd` : Créer un utilisateur
- [ ] `deluser` / `userdel` : Supprimer un utilisateur
- [ ] `usermod` : Modifier un utilisateur
- [ ] `passwd` : Changer le mot de passe
- [ ] `chage` : Gérer l'expiration du mot de passe
- [ ] `id` : Afficher UID/GID/groupes
- [ ] `whoami` : Afficher le nom d'utilisateur
- [ ] `who` / `w` : Voir qui est connecté
- [ ] `su` : Changer d'utilisateur
- [ ] `sudo` : Exécuter en tant qu'un autre utilisateur

### ✅ Linux - Commandes groupes

- [ ] `groupadd` : Créer un groupe
- [ ] `groupdel` : Supprimer un groupe
- [ ] `groupmod` : Modifier un groupe
- [ ] `gpasswd -a` : Ajouter un utilisateur à un groupe
- [ ] `gpasswd -d` : Retirer un utilisateur d'un groupe
- [ ] `usermod -aG` : Ajouter aux groupes secondaires
- [ ] `groups` : Afficher les groupes d'un utilisateur

### ✅ Windows - Commandes utilisateurs

- [ ] `Get-LocalUser` : Lister les utilisateurs
- [ ] `New-LocalUser` : Créer un utilisateur
- [ ] `Remove-LocalUser` : Supprimer un utilisateur
- [ ] `Enable-LocalUser` / `Disable-LocalUser` : Activer/Désactiver
- [ ] `Set-LocalUser` : Modifier un utilisateur
- [ ] `Rename-LocalUser` : Renommer un utilisateur
- [ ] `Get-WmiObject Win32_UserAccount` : Infos détaillées

### ✅ Windows - Commandes groupes

- [ ] `Get-LocalGroup` : Lister les groupes
- [ ] `New-LocalGroup` : Créer un groupe
- [ ] `Remove-LocalGroup` : Supprimer un groupe
- [ ] `Add-LocalGroupMember` : Ajouter un membre
- [ ] `Get-LocalGroupMember` : Lister les membres
- [ ] `Remove-LocalGroupMember` : Retirer un membre

### ✅ Windows - SAM

- [ ] Base de données locale des utilisateurs
- [ ] Emplacement : `C:\Windows\System32\Config\SAM`
- [ ] Verrouillée pendant le fonctionnement de Windows
- [ ] Contient les hachages des mots de passe
- [ ] Accès GUI : `rundll32.exe keymgr.dll,KRShowKeyMgr`

---

## 📊 Tableaux récapitulatifs

### Comparaison Linux vs Windows

| Aspect | Linux | Windows |
|--------|-------|---------|
| **Fichier utilisateurs** | `/etc/passwd` | Base SAM |
| **Mots de passe** | `/etc/shadow` | Base SAM (chiffrée) |
| **Groupes** | `/etc/group` | Base SAM |
| **Créer utilisateur** | `adduser` | `New-LocalUser` |
| **Supprimer utilisateur** | `deluser` | `Remove-LocalUser` |
| **Créer groupe** | `groupadd` | `New-LocalGroup` |
| **Ajouter au groupe** | `usermod -aG` | `Add-LocalGroupMember` |
| **Lister utilisateurs** | `cat /etc/passwd` | `Get-LocalUser` |
| **Changer mot de passe** | `passwd` | `Set-LocalUser -Password` |

---

### Colonnes /etc/passwd

| N° | Nom | Exemple | Description |
|----|-----|---------|-------------|
| 1 | Login | `wilder` | Nom de connexion |
| 2 | Password | `x` | Référence à /etc/shadow |
| 3 | UID | `1000` | User ID |
| 4 | GID | `1000` | Group ID principal |
| 5 | GECOS | `Some Heroic Wilder` | Description |
| 6 | Home | `/home/wilder` | Répertoire personnel |
| 7 | Shell | `/bin/bash` | Shell de connexion |

---

### Colonnes /etc/shadow

| N° | Nom | Exemple | Description |
|----|-----|---------|-------------|
| 1 | Login | `wilder` | Nom de connexion |
| 2 | Password | `$6$...` | Hash du mot de passe |
| 3 | Lastchange | `19081` | Jours depuis 01/01/1970 |
| 4 | Minimum | `0` | Jours minimum entre changements |
| 5 | Maximum | `99999` | Jours maximum de validité |
| 6 | Warn | `7` | Jours d'avertissement |
| 7 | Inactive | (vide) | Jours de tolérance |
| 8 | Expire | (vide) | Date d'expiration du compte |
| 9 | Reserved | (vide) | Réservé |

---

### Algorithmes de hachage

| ID | Algorithme | Bits | Sécurité | Recommandation |
|----|------------|------|----------|----------------|
| `$1$` | MD5 | 128 | ❌ Faible | Obsolète |
| `$2a$` | Blowfish | 448 | ⚠️ Acceptable | Éviter |
| `$5$` | SHA-256 | 256 | ✅ Bon | Acceptable |
| `$6$` | SHA-512 | 512 | ✅✅ Très bon | **Recommandé** |
| `$y$` | yescrypt | Variable | ✅✅✅ Excellent | Moderne |

---

## 📖 Glossaire technique

| Terme | Définition |
|-------|------------|
| **IAM** | Identity and Access Management - Gestion des identités et des accès |
| **Identification** | Déclaration de son identité (login) |
| **Authentification** | Vérification de l'identité (mot de passe) |
| **Autorisation** | Permissions accordées après authentification |
| **MFA** | Multi-Factor Authentication - Authentification multi-facteurs |
| **UID** | User IDentifier - Identifiant utilisateur Linux |
| **GID** | Group IDentifier - Identifiant de groupe Linux |
| **SID** | Security IDentifier - Identifiant de sécurité Windows |
| **GECOS** | Champ commentaire dans /etc/passwd (nom complet, bureau, téléphone, etc.) |
| **Shell** | Interpréteur de commandes (/bin/bash, /bin/zsh, etc.) |
| **Hash** | Empreinte cryptographique d'un mot de passe |
| **Salt** | Valeur aléatoire ajoutée au mot de passe avant hachage |
| **SAM** | Security Account Manager - Base de données utilisateurs Windows |
| **/etc/passwd** | Fichier contenant la liste des utilisateurs Linux |
| **/etc/shadow** | Fichier contenant les mots de passe chiffrés Linux |
| **/etc/group** | Fichier contenant la liste des groupes Linux |
| **adduser** | Commande pour créer un utilisateur (Linux) |
| **usermod** | Commande pour modifier un utilisateur (Linux) |
| **passwd** | Commande pour changer le mot de passe (Linux) |
| **chage** | Commande pour gérer l'expiration des mots de passe (Linux) |
| **sudo** | Super User DO - Exécuter en tant que root ou autre utilisateur |
| **Get-LocalUser** | Cmdlet PowerShell pour lister les utilisateurs |
| **New-LocalUser** | Cmdlet PowerShell pour créer un utilisateur |
| **Add-LocalGroupMember** | Cmdlet PowerShell pour ajouter un membre à un groupe |
| **WMI** | Windows Management Instrumentation - Framework d'administration Windows |

---

## 📝 Checklist de révision

### Avant l'examen TSSR, je dois savoir :

#### Concepts de sécurité
- [ ] Définir IAM (Identity and Access Management)
- [ ] Différence entre identification et authentification
- [ ] Expliquer les facteurs d'authentification
- [ ] Comprendre le principe du MFA

#### Linux - Fichiers système
- [ ] Connaître la structure de `/etc/passwd`
- [ ] Connaître la structure de `/etc/shadow`
- [ ] Connaître la structure de `/etc/group`
- [ ] Savoir que root a UID=0 et GID=0
- [ ] Comprendre GECOS (champ description)
- [ ] Identifier un compte système (/usr/sbin/nologin)

#### Linux - Gestion utilisateurs
- [ ] Créer un utilisateur : `adduser`
- [ ] Supprimer un utilisateur : `deluser`
- [ ] Modifier un utilisateur : `usermod`
- [ ] Changer le mot de passe : `passwd`
- [ ] Gérer l'expiration : `chage`
- [ ] Modifier le shell : `chsh`
- [ ] Verrouiller/déverrouiller : `passwd -l/-u`

#### Linux - Gestion groupes
- [ ] Créer un groupe : `groupadd`
- [ ] Supprimer un groupe : `groupdel`
- [ ] Ajouter au groupe : `usermod -aG` ou `gpasswd -a`
- [ ] Retirer du groupe : `gpasswd -d`
- [ ] Lister les groupes : `groups`

#### Linux - Commandes d'information
- [ ] Afficher UID/GID : `id`
- [ ] Afficher le nom : `whoami`
- [ ] Voir qui est connecté : `who`, `w`
- [ ] Historique connexions : `last`
- [ ] Changer d'utilisateur : `su -`
- [ ] Exécuter en sudo : `sudo`

#### Windows - Commandes PowerShell
- [ ] Lister utilisateurs : `Get-LocalUser`
- [ ] Créer utilisateur : `New-LocalUser`
- [ ] Supprimer utilisateur : `Remove-LocalUser`
- [ ] Activer/Désactiver : `Enable/Disable-LocalUser`
- [ ] Lister groupes : `Get-LocalGroup`
- [ ] Créer groupe : `New-LocalGroup`
- [ ] Ajouter au groupe : `Add-LocalGroupMember`
- [ ] Lister membres : `Get-LocalGroupMember`

#### Windows - SAM
- [ ] Comprendre ce qu'est la base SAM
- [ ] Connaître l'emplacement : `C:\Windows\System32\Config\SAM`
- [ ] Savoir qu'elle est verrouillée pendant le fonctionnement
- [ ] Accès au gestionnaire : `rundll32.exe keymgr.dll,KRShowKeyMgr`

#### Sécurité des mots de passe
- [ ] Comprendre le hachage (hash)
- [ ] Comprendre le salt
- [ ] Connaître les algorithmes (MD5, SHA-256, SHA-512)
- [ ] Identifier `!` ou `*` = compte verrouillé
- [ ] Gérer l'expiration des mots de passe

---

**🎓 Bon courage pour ta préparation au titre RNCP TSSR !**

---

*Document de révision créé pour la formation TSSR - Novembre 2025*  
*Les utilisateurs - Partie 2 : Gestion des utilisateurs*
