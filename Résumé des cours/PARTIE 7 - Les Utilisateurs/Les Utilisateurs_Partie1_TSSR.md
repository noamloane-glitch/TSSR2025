# Les utilisateurs - Partie 1
## Découverte et définitions

---

**Formation TSSR - Titre RNCP**  
**Date de révision :** Novembre 2025  
**Contexte :** Préparation titre Technicien Supérieur Systèmes et Réseaux

---

## 📋 Sommaire

1. [Définition](#définition)
   - [Qu'est-ce qu'un utilisateur ?](#un-utilisateur)
   - [Les groupes](#les-groupes)
   - [Les identifiants uniques](#les-identifiants-uniques-uid)
   - [Le périmètre](#le-périmètre)
   - [Les droits d'accès](#les-droits-daccès)
2. [Droits d'accès sur Linux](#les-droits-daccès-sur-linux)
   - [Types de droits Linux](#les-types-de-droits-daccès-linux)
   - [Affichage des droits](#laffichage-des-droits-daccès)
   - [ACL (Access Control Lists)](#les-droits-avancés-acl)
   - [Commandes Linux](#quelques-commandes)
3. [Droits d'accès sur Windows](#les-droits-daccès-sur-windows)
   - [Types de droits Windows](#les-types-de-droits-daccès-windows)
   - [L'héritage](#lhéritage)
   - [Affichage PowerShell](#laffichage-des-droits-daccès-1)
   - [Commandes PowerShell](#quelques-commandes-1)
4. [Répertoire personnel](#répertoire-personnel)
   - [Linux : /home](#sur-linux)
   - [Windows : C:\Users](#sur-windows)
5. [Points clés à retenir](#points-clés-à-retenir)
6. [Glossaire technique](#glossaire-technique)

---

## 🔍 Questions fondamentales

Avant de commencer, réfléchissons à ces questions :

- ❓ Au sein d'un OS, qu'est-ce qu'un **utilisateur** ?
- ❓ Au sein d'un OS, qu'est-ce qu'un **droit d'accès** ?
- ❓ Point de vue CNIL, informatique ?

---

## 👤 Définition

### Un utilisateur

#### L'approche encyclopédique

> **Définition :** En informatique, le terme **utilisateur** (anciennement un opérateur ou un informaticien) est employé pour désigner une personne qui utilise un système informatisé (ordinateur ou robot) mais qui n'est pas nécessairement informaticien (par opposition au programmeur par exemple).

*Source : Wikipedia*

---

### Un utilisateur - Dans la vraie vie !

**Un utilisateur peut être :**

1. 👨‍💻 **Association à un être humain**
   - Personne physique qui utilise des programmes ou des systèmes
   - Exemple : `jdupont`, `marie.martin`

2. 🤖 **Association à des services**
   - Compte système pour des applications
   - Exemple : `www-data` sur serveur web, `mysql`, `nginx`

3. 👑 **Association à des rôles**
   - Comptes avec privilèges spéciaux
   - Exemple : `root` (Linux), `administrateur` (Windows)

---

### Les groupes

#### Des utilisateurs (mais pas que)

**Les groupes sont des contenants :**

| Système | Contenu | Particularité |
|---------|---------|---------------|
| **Linux** | Contiennent des **utilisateurs** | Un groupe ne peut contenir que des utilisateurs |
| **Windows** | Contiennent des **utilisateurs** OU d'**autres groupes** | Imbrication possible (groupes dans groupes) |

> 💡 **Utilité des groupes :** Simplifier la gestion des permissions en attribuant des droits à un groupe plutôt qu'à chaque utilisateur individuellement.

---

### Les identifiants uniques (UID)

#### Mais qui êtes-vous ?

**Définition :**

> Un **identifiant unique** est un **numéro attribué par l'OS** à chaque utilisateur du système. Ce numéro est utilisé pour :
> - **Identifier** l'utilisateur (ou le groupe) auprès du système
> - **Déterminer** les ressources système auxquelles l'utilisateur (ou le groupe) peut accéder

---

### Les identifiants uniques (Linux)

#### Avec la console Linux

#### 📊 Types d'identifiants Linux

| Type | Acronyme | Signification | Utilisation |
|------|----------|---------------|-------------|
| **UID** | User IDentifier | Identifiant unique d'**utilisateur** | Identifie chaque utilisateur |
| **GID** | Group IDentifier | Identifiant unique de **groupe** | Identifie chaque groupe |

---

#### Exemple avec la commande `id`

```bash
wilder@Ubuntu:~$ id wilder
uid=1000(wilder) gid=1000(wilder) groupes=1000(wilder),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),122(lpadmin),134(lxd),135(sambashare)
```

**Décomposition :**
- `uid=1000(wilder)` : UID 1000, nom `wilder`
- `gid=1000(wilder)` : GID principal 1000, nom `wilder`
- `groupes=...` : Liste de tous les groupes dont l'utilisateur est membre

---

#### Exemple avec `/etc/group`

```bash
wilder@Ubuntu:~$ cat /etc/group | grep lpadmin
lpadmin:x:122:wilder
```

**Format du fichier `/etc/group` :**
```
nom_groupe:mot_de_passe:GID:membres
```

**Décomposition :**
- `lpadmin` : Nom du groupe
- `x` : Mot de passe (stocké ailleurs)
- `122` : GID du groupe
- `wilder` : Membres du groupe

---

### Les identifiants uniques (Windows)

#### Avec la console Windows

#### 📊 Type d'identifiant Windows

| Type | Acronyme | Signification | Utilisation |
|------|----------|---------------|-------------|
| **SID** | Security IDentifier | Identifiant de sécurité | Identifie utilisateurs ET groupes |

> 💡 **Particularité Windows :** Le même type d'identifiant (SID) est utilisé pour les utilisateurs et les groupes.

---

#### SID utilisateur

```powershell
PS C:\> Get-WmiObject win32_useraccount | Where-Object {$_.Name -eq "wilder"} | Select-Object Name,SID

Name      SID                                          
----      ---                                          
wilder    S-1-5-21-2973921518-4066818644-3297592939-500
```

---

#### SID groupe

```powershell
PS C:\> Get-WmiObject win32_group | Where-Object {$_.Name -eq "GrpTest"} | Select-Object Name,SID

Name       SID         
----       ---         
GrpTest    S-1-5-32-544
```

---

#### 🔍 Structure d'un SID

```
S-1-5-21-2973921518-4066818644-3297592939-500
│ │ │  │                                     │
│ │ │  │                                     └─ RID (Relative ID)
│ │ │  └─────────────────────────────────────── Identifiant du domaine
│ │ └──────────────────────────────────────────── Autorité d'identification
│ └─────────────────────────────────────────────── Version
└───────────────────────────────────────────────── Préfixe (S = SID)
```

---

### Le périmètre

#### Que peut-il faire ?

**Périmètre fonctionnel :**

> **Définition :** Représente **toutes les fonctionnalités disponibles**, ou **toutes les applications** d'une solution logicielle.

**Détermine :**
- Les **applications** réunies dans le système d'information
- Les **fonctionnalités** mises en place
- Ce à quoi l'utilisateur peut **accéder**

#### 🎯 Exemples de périmètres

| Utilisateur | Périmètre |
|-------------|-----------|
| **Administrateur** | Accès complet au système, configuration, gestion des utilisateurs |
| **Utilisateur standard** | Applications de bureautique, navigation web, impression |
| **Service web** | Accès aux fichiers web, base de données, pas d'accès au système |

---

### Les droits d'accès

#### La définition encyclopédique

> **Définition :** Les **droits d'accès** sont des **métadonnées particulières** qui décrivent les droits en **lecture**, **écriture** et **exécution** selon l'**utilisateur**, le **groupe**, ou les **autres**.

*Source : Wikipedia*

---

### Les types de droits d'accès

#### Définition dans un SI

**Les 6 types de droits fondamentaux :**

| Droit | Symbole | Description |
|-------|---------|-------------|
| **Non-accès** | - | Refus/Interdiction d'accès |
| **Lecture** | **R** (Read) | Lire le contenu |
| **Écriture** | **W** (Write) | Créer, modifier le contenu |
| **Modification** | M | Modifier les propriétés |
| **Suppression** | D | Supprimer le fichier/dossier |
| **Exécution** | **X** (eXecute) | Exécuter un fichier/traverser un dossier |

> 💡 **Note :** Les droits exacts dépendent du système d'exploitation (Linux, Windows, etc.)

---

## 🐧 Les droits d'accès sur Linux

### Les types de droits d'accès (Linux)

#### Qui a le droit ?

**Tout fichier ou dossier se voit attribuer des droits pour 3 catégories :**

#### 📊 Les 3 catégories de propriété

| Catégorie | Symbole | Description | Application |
|-----------|---------|-------------|-------------|
| **Utilisateur propriétaire** | `u` (user) | UID propriétaire du fichier | S'applique pour l'utilisateur ayant le **même UID** |
| **Groupe propriétaire** | `g` (group) | GID propriétaire du fichier | S'applique pour un utilisateur **appartenant à ce groupe** |
| **Autres** | `o` (other) | Tous les autres | S'applique à ceux **ne rentrant pas** dans les 2 premières catégories |

---

### L'affichage des droits d'accès

#### Comment je le vois ?

**Sous la forme d'une chaîne de 10 caractères :**

```
-rwxrw-r--
│└┬┘└┬┘└┬┘
│ │  │  └─── Droits pour les "autres" (other)
│ │  └────── Droits pour le groupe (group)
│ └───────── Droits pour le propriétaire (user)
└─────────── Type de fichier
```

#### 🔍 Détail de la structure

- **1 caractère** pour le **type de fichier**
- **3 × 3 groupes** pour définir les droits des 3 identités
- On utilise le tiret (`-`) si un droit n'existe pas

---

### Dans un terminal avec `ls -l`

#### Comment je le vois ? (2)

```bash
wilder@Ubuntu:~$ ls -l file
-rw-rw-r-- 1 wilder wilder 0 sept. 23 15:45 file
```

**Décomposition complète :**

```
-rw-rw-r-- 1 wilder wilder 0 sept. 23 15:45 file
│└─┬─┘└─┬─┘└─┬─┘│   │      │    │      │         │
│  │    │    │   │   │      │    │      │         └─ Nom du fichier
│  │    │    │   │   │      │    │      └─────────── Date de modification
│  │    │    │   │   │      │    └────────────────── Taille en octets
│  │    │    │   │   │      └─────────────────────── Groupe propriétaire
│  │    │    │   │   └────────────────────────────── Utilisateur propriétaire
│  │    │    │   └────────────────────────────────── Nombre de liens
│  │    │    └────────────────────────────────────── Droits autres (r--)
│  │    └─────────────────────────────────────────── Droits groupe (rw-)
│  └──────────────────────────────────────────────── Droits utilisateur (rw-)
└─────────────────────────────────────────────────── Type (-=fichier, d=dossier)
```

---

#### 📋 Premier caractère : Type de fichier

| Caractère | Type |
|-----------|------|
| `-` | Fichier ordinaire |
| `d` | Répertoire (directory) |
| `l` | Lien symbolique (link) |
| `c` | Périphérique caractère |
| `b` | Périphérique bloc |
| `s` | Socket |
| `p` | Pipe (tube) |

---

#### 📋 Droits : r, w, x

| Droit | Symbole | Fichier | Répertoire |
|-------|---------|---------|------------|
| **Lecture** | `r` | Lire le contenu | Lister le contenu |
| **Écriture** | `w` | Modifier le contenu | Créer/supprimer des fichiers |
| **Exécution** | `x` | Exécuter le fichier | Traverser (accéder) au répertoire |
| **Aucun** | `-` | Pas de permission | Pas de permission |

---

### Exemples de droits Linux

#### 📝 Exemples commentés

```bash
# Fichier avec droits complets pour le propriétaire
-rwx------ 1 user group 1234 jan 01 file.sh
# user: rwx (lecture, écriture, exécution)
# group: --- (aucun droit)
# other: --- (aucun droit)

# Répertoire accessible à tous
drwxr-xr-x 2 user group 4096 jan 01 docs/
# user: rwx (total)
# group: r-x (lecture + execution)
# other: r-x (lecture + execution)

# Fichier modifiable par le groupe
-rw-rw-r-- 1 user group 5678 jan 01 data.txt
# user: rw- (lecture + écriture)
# group: rw- (lecture + écriture)
# other: r-- (lecture seule)
```

---

### Notation octale des droits

#### 🔢 Conversion binaire → octal

**Chaque groupe de 3 droits = 1 chiffre octal**
**r** = 4 
**w** = 2
**x** = 1
**-** = 0

| Binaire | Octal | Droits | Description          |
| ------- | ----- | ------ | -------------------- |
| `000`   | **0** | `---`  | Aucun droit          |
| `001`   | **1** | `--x`  | Exécution seule      |
| `010`   | **2** | `-w-`  | Écriture seule       |
| `011`   | **3** | `-wx`  | Écriture + exécution |
| `100`   | **4** | `r--`  | Lecture seule        |
| `101`   | **5** | `r-x`  | Lecture + exécution  |
| `110`   | **6** | `rw-`  | Lecture + écriture   |
| `111`   | **7** | `rwx`  | Tous les droits      |

---

#### Exemples de notation octale

```bash
# -rwxr-xr-x
# user=rwx(7), group=r-x(5), other=r-x(5)
chmod 755 fichier

# -rw-r--r--
# user=rw-(6), group=r--(4), other=r--(4)
chmod 644 fichier

# -rwx------
# user=rwx(7), group=---(0), other=---(0)
chmod 700 fichier

# -rw-rw-r--
# user=rw-(6), group=rw-(6), other=r--(4)
chmod 664 fichier
```

---

### Les droits avancés ACL

#### Pour aller plus loin

**ACL = Access Control Lists (Listes de Contrôle d'Accès)**

#### 🎯 Caractéristiques

- ✅ Amènent une **gestion des droits plus fine**
- ✅ Se **superposent** aux droits classiques, **ne les remplacent pas**
- ✅ Utiles pour des droits aux utilisateurs qui :
  - N'ont pas de droits classiques
  - Ne sont pas dans le groupe ayant des droits

#### 💡 Cas d'usage

**Exemple :** Donner des droits spécifiques à un utilisateur sans le mettre dans le groupe propriétaire du fichier.

---

### rwx pour wilder1 sur test.txt

#### Cas concret d'ACL

**Avant l'ajout d'ACL :**

```bash
wilder@Ubuntu:~$ getfacl file
# file: file
# owner: wilder
# group: wilder
user::rw-
group::rw-
other::r--
```

---

**Ajout d'un droit ACL pour wilder1 :**

```bash
wilder@Ubuntu:~$ setfacl -m u:wilder1:rwx file
```

**Après l'ajout d'ACL :**

```bash
wilder@Ubuntu:~$ getfacl file
# file: file
# owner: wilder
# group: wilder
user::rw-
user:wilder1:rwx
group::rw-
mask::rwx
other::r--
```

> 🔍 **Observation :** L'utilisateur `wilder1` a maintenant des droits `rwx` spécifiques sur ce fichier, sans être le propriétaire ni dans le groupe.

---

### Quelques commandes

#### Pour le terminal

| Commande | Fonction |
|----------|----------|
| `ls -l` | Afficher les droits des fichiers d'un répertoire |
| `chown` | Changement du propriétaire |
| `chgrp` | Changement de groupe |
| `chmod` | Changement des droits d'accès |

#### 📖 Pour aller plus loin

**Documentation Ubuntu :** https://doc.ubuntu-fr.org/permissions

---

#### Exemples de commandes

```bash
# Afficher les droits
ls -l fichier.txt

# Changer le propriétaire
sudo chown utilisateur fichier.txt

# Changer le groupe
sudo chgrp groupe fichier.txt

# Changer propriétaire ET groupe
sudo chown utilisateur:groupe fichier.txt

# Changer les droits (symbolique)
chmod u+x fichier.sh          # Ajouter exécution pour user
chmod g-w fichier.txt         # Retirer écriture pour group
chmod o=r fichier.txt         # Définir lecture seule pour other
chmod a+r fichier.txt         # Ajouter lecture pour all

# Changer les droits (octal)
chmod 755 script.sh           # rwxr-xr-x
chmod 644 document.txt        # rw-r--r--
chmod 700 privé.txt           # rwx------

# Récursif (pour dossiers)
chmod -R 755 /mon/dossier     # Applique à tous les fichiers/sous-dossiers
```

---

### Quelques commandes sur les ACL

#### Pour le terminal

| Commande | Fonction |
|----------|----------|
| `setfacl` | Ajout/suppression de droits ACL |
| `getfacl` | Affichage des droits ACL |

#### 📖 Pour aller plus loin

**Documentation ACL :** https://linux.goffinet.org/administration/securite-locale/access-control-lists-acls-linux/

---

#### Exemples de commandes ACL

```bash
# Afficher les ACL d'un fichier
getfacl fichier.txt

# Ajouter un droit ACL pour un utilisateur
setfacl -m u:username:rwx fichier.txt

# Ajouter un droit ACL pour un groupe
setfacl -m g:groupname:rw fichier.txt

# Retirer un droit ACL
setfacl -x u:username fichier.txt

# Retirer tous les droits ACL
setfacl -b fichier.txt

# Copier les ACL d'un fichier vers un autre
getfacl fichier1.txt | setfacl --set-file=- fichier2.txt

# Appliquer récursivement
setfacl -R -m u:username:rwx /mon/dossier

# Définir des ACL par défaut (pour les nouveaux fichiers)
setfacl -d -m u:username:rwx /mon/dossier
```

---

## 🪟 Les droits d'accès sur Windows

### Les types de droits d'accès (Windows)

#### Les types de droits

**Il y en a 5 principaux :**

| Droit | Description | Permet |
|-------|-------------|--------|
| **Contrôle total** | Tous les droits | Tout faire sur le fichier/dossier |
| **Modification** | Lecture + écriture + suppression | Modifier et supprimer |
| **Lecture et exécution** | Lire + exécuter | Voir et lancer |
| **Écriture** | **W** (Write) | Créer de nouveaux fichiers/sous-dossiers |
| **Lecture** | **R** (Read) | Voir le contenu |

---

**Avec 2 possibilités pour chaque droit :**

| État | Anglais | Effet |
|------|---------|-------|
| **Autoriser** | Allow | Le droit est **accordé** ✅ |
| **Refuser** | Deny | Le droit est **explicitement refusé** ❌ |

> ⚠️ **Important :** Un **Deny** (Refuser) l'emporte **toujours** sur un **Allow** (Autoriser)

---

### Détail des droits Windows

#### 🔍 Décomposition des permissions

| Permission | Lecture | Écriture | Exécution | Suppression | Modifier permissions | Prendre possession |
|------------|---------|----------|-----------|-------------|---------------------|-------------------|
| **Contrôle total** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Modification** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Lecture et exécution** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| **Écriture** | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Lecture** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

### Utilisateurs et groupes

#### Qui a le droit ?

**Les droits de tout fichier ou dossier s'adressent aux 2 types suivants :**

1. **Les utilisateurs**
   - Comptes individuels
   - Exemple : `wilder`, `marie.dupont`

2. **Les groupes d'utilisateurs**
   - Conteneurs d'utilisateurs (et de groupes sous Windows)
   - Exemple : `Administrateurs`, `Utilisateurs`, `GrpCompta`

> 💡 **Avantage des groupes :** Gérer les permissions pour plusieurs utilisateurs en une seule fois

---

### L'héritage

#### On ne parle pas d'argent là !

> **Définition :** Par défaut, un **sous-dossier hérite des droits d'accès de son dossier parent**. L'utilisateur qui a reçu l'autorisation de modifier les droits peut **remplacer les droits de ses groupes** en attribuant des droits différents à un sous-dossier.

*Source : IBM*

---

#### 🌳 Schéma de l'héritage

```
C:\Projets
├─ Droits : Groupe "Dev" = Modification
│
├─ Projet1\
│  └─ Hérite : Groupe "Dev" = Modification ✅
│
├─ Projet2\
│  └─ Hérite : Groupe "Dev" = Modification ✅
│
└─ ProjetSecret\
   └─ Héritage désactivé ❌
      Droits custom : Groupe "Admin" = Contrôle total
```

---

#### ⚙️ Gestion de l'héritage

**Actions possibles :**
- ✅ **Activer l'héritage** : Le dossier reçoit les droits du parent
- ❌ **Désactiver l'héritage** : 2 options :
  - **Convertir** : Copier les droits hérités en droits explicites
  - **Supprimer** : Retirer tous les droits hérités

> ⚠️ **Attention :** Désactiver l'héritage peut rendre le dossier inaccessible si aucun droit explicite n'est défini !

---

### L'affichage des droits d'accès

#### Comment je le vois en console ?

```powershell
PS C:\> Get-Acl -Path c:\temp\file.txt | Format-List

Path    : Microsoft.PowerShell.Core\FileSystem::C:\temp\file.txt
Owner   : PCLab\wilder
Group   : PCLab\Aucun
Access  : BUILTIN\Administrateurs Allow  FullControl
          AUTORITE NT\Système Allow  FullControl
          BUILTIN\Utilisateurs Allow  ReadAndExecute, Synchronize
          AUTORITE NT\Utilisateurs authentifiés Allow  Modify, Synchronize
Audit   : 
Sddl    : O:S-1-5-21-2676666238-4226417603-221635342-1000G:S-1-5-21-...
```

---

**Décomposition :**

| Élément | Description | Valeur exemple |
|---------|-------------|----------------|
| **Path** | Chemin complet du fichier | `C:\temp\file.txt` |
| **Owner** | Propriétaire | `PCLab\wilder` |
| **Group** | Groupe principal | `PCLab\Aucun` |
| **Access** | Liste des droits | Voir ci-dessous |
| **Audit** | Configuration d'audit | (vide par défaut) |
| **Sddl** | Security Descriptor Definition Language | Format technique |

---

**Access - Format :**
```
[Utilisateur/Groupe] [Allow/Deny] [Permissions]
```

**Exemples :**
```
BUILTIN\Administrateurs Allow FullControl
→ Le groupe Administrateurs a le contrôle total

BUILTIN\Utilisateurs Allow ReadAndExecute, Synchronize
→ Le groupe Utilisateurs peut lire et exécuter
```

---

### L'affichage des droits d'accès

#### La méthode GUI

**Accès via l'interface graphique :**

1. **Clic droit** sur le fichier/dossier
2. **Propriétés**
3. Onglet **Sécurité**

**Affichage :**
- Liste des utilisateurs/groupes
- Permissions pour chaque utilisateur/groupe
- Coches : ☑ (Autoriser) ou ☑ (Refuser)
- Boutons : **Modifier**, **Avancé**

---

### Control total pour wilder1 sur file.txt

#### Cas concret PowerShell

```powershell
# Récupérer l'ACL actuelle
PS C:\> $acl = Get-Acl C:\temp\file.txt

# Créer une nouvelle règle d'accès
PS C:\> $AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("wilder1","FullControl","Allow")

# Appliquer la règle à l'ACL
PS C:\> $Acl.SetAccessRule($AccessRule)

# Sauvegarder l'ACL modifiée
PS C:\> $Acl | Set-Acl C:\temp\file.txt
```

---

**Vérification :**

```powershell
PS C:\> Get-Acl C:\temp\file.txt | Format-List

Path    : Microsoft.PowerShell.Core\FileSystem::C:\temp\file.txt
Owner   : PCLab\wilder
Group   : PCLab\wilder
Access  : PCLab\wilder1 Allow FullControl
          BUILTIN\Administrateurs Allow FullControl
          AUTORITE NT\Système Allow FullControl
          BUILTIN\Utilisateurs Allow ReadAndExecute, Synchronize
          AUTORITE NT\Utilisateurs authentifiés Allow Modify, Synchronize
```

> 🔍 **Observation :** L'utilisateur `wilder1` a maintenant le contrôle total sur le fichier

---

### Quelques commandes

#### En PowerShell

| Commande | Fonction |
|----------|----------|
| `Get-Acl` | Afficher les droits utilisateurs sur fichier ou répertoire |
| `Set-Acl` | Modifier les droits sur fichier ou répertoire |
| `icacls` | Commande CMD pour gérer les ACL (alternative) |

#### 📖 Pour aller plus loin

**Documentation :** https://petri.com/how-to-use-powershell-to-manage-folder-permissions/

---

#### Exemples PowerShell

```powershell
# Afficher les droits
Get-Acl C:\dossier | Format-List

# Copier les droits d'un fichier vers un autre
$acl = Get-Acl C:\source.txt
Set-Acl C:\destination.txt $acl

# Ajouter un droit (Lecture) pour un utilisateur
$acl = Get-Acl C:\fichier.txt
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "utilisateur",
    "Read",
    "Allow"
)
$acl.SetAccessRule($rule)
Set-Acl C:\fichier.txt $acl

# Retirer un utilisateur des droits
$acl = Get-Acl C:\fichier.txt
$acl.Access | Where-Object {$_.IdentityReference -eq "utilisateur"} | ForEach-Object {
    $acl.RemoveAccessRule($_)
}
Set-Acl C:\fichier.txt $acl
```

---

#### Exemples avec icacls (CMD)

```cmd
REM Afficher les droits
icacls C:\dossier

REM Ajouter contrôle total pour un utilisateur
icacls C:\fichier.txt /grant utilisateur:F

REM Ajouter lecture seule
icacls C:\fichier.txt /grant utilisateur:R

REM Retirer un utilisateur
icacls C:\fichier.txt /remove utilisateur

REM Copier les droits
icacls C:\source /save droits.txt
icacls C:\destination /restore droits.txt

REM Réinitialiser les droits (hériter du parent)
icacls C:\dossier /reset

REM Appliquer récursivement
icacls C:\dossier /grant utilisateur:F /T
```

**Permissions icacls :**
- `F` = Full control (Contrôle total)
- `M` = Modify (Modification)
- `RX` = Read and execute (Lecture et exécution)
- `R` = Read (Lecture)
- `W` = Write (Écriture)

---

## 🏠 Répertoire personnel

### Définition

#### /home/ - C:\Users\

**Le répertoire personnel :**

- ✅ Stocke les **dossiers utilisateurs**
- ✅ Est créé **automatiquement** pour chaque nouvel utilisateur
- ✅ L'utilisateur peut y stocker ses **données personnelles**
- ✅ Différent du **répertoire individuel** (point de vue juridique)

| Système | Emplacement |
|---------|-------------|
| **Linux** | `/home/` |
| **Windows** | `C:\Users\` |

---

### Sur Linux

#### Chez le pingouin 🐧

**Caractéristiques :**

- Le répertoire de chaque utilisateur est sous **`/home`**
- **Exemple :** Le répertoire personnel de l'utilisateur `wilder` est `/home/wilder`
- **Exception :** `root` a son répertoire sous **`/root`** (pas dans `/home`)
- **Raccourci shell :** `~` (tilde)
- **Paramètre par défaut de `cd`** (sans argument)

---

#### 📝 Exemples

```bash
# Aller dans son répertoire personnel
cd
# ou
cd ~
# ou
cd /home/wilder

# Variable d'environnement
echo $HOME
# Affiche : /home/wilder

# Aller dans le home d'un autre utilisateur (si droits)
cd ~autre_utilisateur
# Équivalent à : cd /home/autre_utilisateur
```

---

### Contenu de /home/xxx

#### Le contenu

```bash
wilder@Ubuntu:~$ ls -l /home/wilder
total 36
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Bureau
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Documents
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Images
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Modèles
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Musique
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Public
drwx------  3 wilder wilder 4096 juin  26 02:09 snap
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Téléchargements
drwxr-xr-x  2 wilder wilder 4096 juin  26 02:09 Vidéos
```

---

#### 📂 Dossiers standards Linux

| Dossier | Utilisation |
|---------|-------------|
| `Bureau` | Fichiers du bureau (Desktop) |
| `Documents` | Documents personnels |
| `Images` | Photos et images |
| `Modèles` | Modèles de documents |
| `Musique` | Fichiers audio |
| `Public` | Fichiers partagés avec d'autres utilisateurs |
| `Téléchargements` | Fichiers téléchargés |
| `Vidéos` | Fichiers vidéo |
| `.config` | Configuration des applications (caché) |
| `.local` | Données locales des applications (caché) |
| `.bashrc` | Configuration du shell bash (caché) |

> 💡 **Note :** Les fichiers/dossiers commençant par `.` sont **cachés** (utiliser `ls -a` pour les voir)

---

### Sur Windows

#### À la fenêtre 🪟

**Caractéristiques :**

- Le répertoire de chaque utilisateur (même l'administrateur) est sous **`C:\Users\`**
- **Exemple :** Le répertoire personnel de l'utilisateur `wilder1` est `C:\Users\wilder1`
- **Variable d'environnement :** `%USERPROFILE%`

---

#### 📝 Exemples

```powershell
# Aller dans son répertoire personnel
cd $env:USERPROFILE
# ou
cd ~
# ou
cd C:\Users\wilder

# Afficher le chemin
echo $env:USERPROFILE
# Affiche : C:\Users\wilder
```

```cmd
REM En CMD classique
cd %USERPROFILE%
echo %USERPROFILE%
```

---

### Contenu de C:\Users\xxx

#### Le contenu

```powershell
PS C:\> Get-ChildItem -Path C:\Users\wilder

Mode          LastWriteTime         Length Name                                                                                                          
----          -------------         ------ ----                                                                                                          
d-r---        04/07/2022    16:43          Contacts                                                                                                      
d-r---        30/08/2022    15:03          Desktop                                                                                                       
d-r---        10/08/2022    01:56          Documents                                                                                                     
d-r---        08/08/2022    17:11          Downloads                                                                                                     
d-r---        04/07/2022    16:43          Favorites                                                                                                     
d-r---        04/07/2022    16:43          Links                                                                                                         
d-r---        04/07/2022    16:43          Music                                                                                                         
dar--l        26/09/2022    19:04          OneDrive                                                                                                      
d-r---        04/07/2022    16:44          Pictures                                                                                                      
d-r---        04/07/2022    16:43          Saved Games                                                                                                   
d-r---        05/07/2022    11:50          Searches                                                                                  
d-r---        11/07/2022    14:46          Videos
```

---

#### 📂 Dossiers standards Windows

| Dossier | Nom français | Utilisation |
|---------|--------------|-------------|
| `Desktop` | Bureau | Fichiers du bureau |
| `Documents` | Documents | Documents personnels |
| `Downloads` | Téléchargements | Fichiers téléchargés |
| `Pictures` | Images | Photos et images |
| `Music` | Musique | Fichiers audio |
| `Videos` | Vidéos | Fichiers vidéo |
| `Favorites` | Favoris | Favoris du navigateur |
| `Contacts` | Contacts | Carnet d'adresses |
| `Searches` | Recherches | Recherches enregistrées |
| `Links` | Liens | Liens rapides |
| `Saved Games` | Jeux sauvegardés | Sauvegardes de jeux |
| `OneDrive` | OneDrive | Synchronisation cloud Microsoft |
| `AppData` | - | Données d'applications (caché) |

---

## 🎯 Points clés à retenir

### ✅ Définitions fondamentales

- [ ] **Utilisateur** : Personne, service ou rôle utilisant le système
- [ ] **Groupe** : Contenant d'utilisateurs (et de groupes sous Windows)
- [ ] **UID/GID** (Linux) : Identifiants uniques numériques
- [ ] **SID** (Windows) : Security Identifier (pour utilisateurs ET groupes)
- [ ] **Droits d'accès** : Métadonnées définissant R, W, X par utilisateur/groupe

### ✅ Linux - Droits d'accès

- [ ] **3 catégories** : User (u), Group (g), Other (o)
- [ ] **3 types de droits** : Read (r), Write (w), eXecute (x)
- [ ] **Format d'affichage** : 10 caractères (`-rwxrw-r--`)
- [ ] **Premier caractère** : Type de fichier (`-` = fichier, `d` = dossier)
- [ ] **Notation octale** : 755 = `rwxr-xr-x`
- [ ] **Commandes** : `ls -l`, `chmod`, `chown`, `chgrp`
- [ ] **ACL** : Droits avancés avec `setfacl` et `getfacl`

### ✅ Windows - Droits d'accès

- [ ] **5 types de droits** : Contrôle total, Modification, Lecture et exécution, Écriture, Lecture
- [ ] **2 états** : Allow (Autoriser), Deny (Refuser)
- [ ] **Deny l'emporte sur Allow**
- [ ] **Héritage** : Les sous-dossiers héritent du parent par défaut
- [ ] **Commandes PowerShell** : `Get-Acl`, `Set-Acl`
- [ ] **Commande CMD** : `icacls`

### ✅ Répertoires personnels

**Linux :**
- [ ] Emplacement : `/home/utilisateur`
- [ ] Exception : `/root` pour root
- [ ] Raccourci : `~`
- [ ] Variable : `$HOME`

**Windows :**
- [ ] Emplacement : `C:\Users\utilisateur`
- [ ] Raccourci PowerShell : `~`
- [ ] Variable : `%USERPROFILE%` (CMD) ou `$env:USERPROFILE` (PowerShell)

---

## 📊 Tableaux récapitulatifs

### Comparaison Linux vs Windows

| Aspect | Linux | Windows |
|--------|-------|---------|
| **Identifiant utilisateur** | UID (numérique) | SID (alphanumérique) |
| **Identifiant groupe** | GID (numérique) | SID (alphanumérique) |
| **Catégories de droits** | User, Group, Other | Utilisateurs, Groupes |
| **Types de droits** | r, w, x | 5 niveaux |
| **Héritage** | Non (par défaut) | Oui (par défaut) |
| **Droits avancés** | ACL (setfacl) | ACL (intégré) |
| **Répertoire personnel** | `/home/user` | `C:\Users\user` |
| **Commande droits** | `chmod`, `ls -l` | `icacls`, `Get-Acl` |

---

### Droits Linux - Tableau de conversion

| Octal | Binaire | Symbolique | Description |
|-------|---------|------------|-------------|
| **0** | 000 | `---` | Aucun droit |
| **1** | 001 | `--x` | Exécution |
| **2** | 010 | `-w-` | Écriture |
| **3** | 011 | `-wx` | Écriture + Exécution |
| **4** | 100 | `r--` | Lecture |
| **5** | 101 | `r-x` | Lecture + Exécution |
| **6** | 110 | `rw-` | Lecture + Écriture |
| **7** | 111 | `rwx` | Tous les droits |

---

### Permissions courantes Linux

| Octal | Symbolique | Usage typique |
|-------|------------|---------------|
| **644** | `-rw-r--r--` | Fichier texte standard |
| **755** | `-rwxr-xr-x` | Script exécutable |
| **700** | `-rwx------` | Fichier privé |
| **666** | `-rw-rw-rw-` | Fichier modifiable par tous |
| **777** | `-rwxrwxrwx` | Tous les droits (à éviter !) |
| **600** | `-rw-------` | Fichier privé lecture/écriture |
| **750** | `-rwxr-x---` | Exécutable pour user et group |

---

### Droits Windows - Permissions

| Permission | Lire | Écrire | Exécuter | Supprimer | Modifier ACL | Propriété |
|------------|------|--------|----------|-----------|--------------|-----------|
| **Lecture** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Écriture** | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Lecture et exécution** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| **Modification** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Contrôle total** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 📖 Glossaire technique

| Terme | Définition |
|-------|------------|
| **Utilisateur** | Entité (personne, service, rôle) utilisant un système informatique |
| **Groupe** | Conteneur regroupant des utilisateurs pour simplifier la gestion |
| **UID** | User IDentifier - Identifiant unique d'utilisateur (Linux) |
| **GID** | Group IDentifier - Identifiant unique de groupe (Linux) |
| **SID** | Security IDentifier - Identifiant de sécurité (Windows) |
| **Droits d'accès** | Métadonnées définissant les permissions R/W/X |
| **ACL** | Access Control List - Liste de contrôle d'accès (droits avancés) |
| **Périmètre fonctionnel** | Ensemble des fonctionnalités accessibles à un utilisateur |
| **Propriétaire** | Utilisateur possédant un fichier/dossier |
| **r (read)** | Droit de lecture |
| **w (write)** | Droit d'écriture/modification |
| **x (execute)** | Droit d'exécution |
| **Héritage** | Transmission des droits du parent aux sous-éléments |
| **Allow** | Autoriser (Windows) |
| **Deny** | Refuser (Windows) - prioritaire sur Allow |
| **Répertoire personnel** | Dossier dédié à un utilisateur (`/home/user` ou `C:\Users\user`) |
| **~** (tilde) | Raccourci vers le répertoire personnel |
| **chmod** | Commande Linux pour modifier les droits |
| **chown** | Commande Linux pour changer le propriétaire |
| **chgrp** | Commande Linux pour changer le groupe |
| **setfacl** | Commande Linux pour définir des ACL |
| **getfacl** | Commande Linux pour afficher les ACL |
| **Get-Acl** | Cmdlet PowerShell pour lire les ACL |
| **Set-Acl** | Cmdlet PowerShell pour modifier les ACL |
| **icacls** | Commande CMD pour gérer les ACL Windows |

---

## 📝 Checklist de révision

### Avant l'examen TSSR, je dois savoir :

#### Concepts généraux
- [ ] Définir un utilisateur (3 associations)
- [ ] Expliquer ce qu'est un groupe
- [ ] Différence entre UID/GID (Linux) et SID (Windows)
- [ ] Définir le périmètre fonctionnel
- [ ] Lister les 6 types de droits d'accès

#### Linux - Droits
- [ ] Expliquer les 3 catégories (user, group, other)
- [ ] Lire et interpréter `-rwxrw-r--`
- [ ] Connaître la signification du 1er caractère (`-`, `d`, `l`)
- [ ] Convertir entre notation symbolique et octale
- [ ] Utiliser `chmod` (symbolique et octal)
- [ ] Utiliser `chown` et `chgrp`
- [ ] Comprendre l'utilité des ACL
- [ ] Utiliser `setfacl` et `getfacl`

#### Windows - Droits
- [ ] Lister les 5 types de permissions
- [ ] Différence entre Allow et Deny
- [ ] Comprendre que Deny > Allow
- [ ] Expliquer l'héritage
- [ ] Lire la sortie de `Get-Acl`
- [ ] Créer une règle d'accès avec PowerShell
- [ ] Utiliser `icacls` en ligne de commande

#### Répertoires personnels
- [ ] Emplacement Linux : `/home/user`
- [ ] Exception Linux : `/root` pour root
- [ ] Raccourci : `~` et variable `$HOME`
- [ ] Emplacement Windows : `C:\Users\user`
- [ ] Variable Windows : `%USERPROFILE%` ou `$env:USERPROFILE`
- [ ] Connaître les dossiers standards (Documents, Desktop, etc.)

---

**🎓 Bon courage pour ta préparation au titre RNCP TSSR !**

---

*Document de révision créé pour la formation TSSR - Novembre 2025*  
*Les utilisateurs - Partie 1 : Découverte et définitions*
