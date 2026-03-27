## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)
**Sujet** : Déploiement automatisé de Windows
**Date** : Février 2025
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Pourquoi ne pas dupliquer un poste ?|Pourquoi ne pas dupliquer un poste ?]]
   - [[#Qu'est-ce qu'un déploiement automatisé ?|Qu'est-ce qu'un déploiement automatisé ?]]
2. [[#Les outils de déploiement|Les outils de déploiement]]
   - [[#WDS (Windows Deployment Services)|WDS (Windows Deployment Services)]]
   - [[#MDT (Microsoft Deployment Toolkit)|MDT (Microsoft Deployment Toolkit)]]
   - [[#WADK (Windows Assessment and Deployment Kit)|WADK (Windows Assessment and Deployment Kit)]]
   - [[#SCCM (System Center Configuration Manager)|SCCM (System Center Configuration Manager)]]
3. [[#Le déploiement Windows|Le déploiement Windows]]
   - [[#Déploiement standard|Déploiement standard]]
   - [[#Le master (image de référence)|Le master (image de référence)]]
   - [[#Sysprep|Sysprep]]
   - [[#WinPE (Windows Preinstallation Environment)|WinPE (Windows Preinstallation Environment)]]
   - [[#PXE (Preboot Execution Environment)|PXE (Preboot Execution Environment)]]
   - [[#WIM (Windows Imaging Format)|WIM (Windows Imaging Format)]]
   - [[#DISM - Gestion des images WIM|DISM - Gestion des images WIM]]
4. [[#Schéma de synthèse global|Schéma de synthèse global]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le déploiement automatisé de Windows est une compétence essentielle pour tout technicien systèmes et réseaux. Il permet d'installer rapidement et de manière standardisée des systèmes d'exploitation sur un parc informatique, réduisant ainsi le temps de configuration et les erreurs humaines.

### Pourquoi ne pas dupliquer un poste ?

> [!warning] Piège à éviter - La duplication simple
> Dupliquer directement un poste Windows (par clonage de disque) sans préparation crée de graves problèmes :
> - **SID identiques** : Chaque machine Windows possède un identifiant de sécurité unique (SID). Des SID identiques causent des conflits sur le réseau et dans Active Directory
> - **Pilotes inadaptés** : Les pilotes tiers peuvent ne pas correspondre au matériel cible
> - **Traces d'utilisation** : Historiques, profils utilisateurs, configurations spécifiques restent présents
> - **Licence Windows** : La clé d'activation reste celle du poste source, violant les termes de licence

### Qu'est-ce qu'un déploiement automatisé ?

> [!quote] Définition
> Le déploiement automatisé est un processus qui permet d'installer un système d'exploitation Windows de manière standardisée et reproductible sur plusieurs machines, en utilisant des images préparées et des outils spécialisés.

**Sysprep** est l'outil central qui effectue la **dépersonnalisation / nettoyage machine** :
- Suppression du SID unique
- Nettoyage des pilotes tiers
- Réinitialisation de la clé de licence
- Suppression des traces d'utilisation
- Repassage en **OOBE** (Out Of Box Experience) - l'écran de première configuration Windows

---

## Les outils de déploiement

### WDS (Windows Deployment Services)

> [!quote] Définition officielle
> **WDS** (Windows Deployment Services) est une technologie de Microsoft permettant d'installer un système d'exploitation Windows via le réseau.

#### Caractéristiques principales

| Aspect | Description |
|--------|-------------|
| **Successeur** | RIS (Remote Installation Service) |
| **OS supportés** | Windows Vista à 10 / Server 2008 à 2016 |
| **Disponibilité** | Rôle serveur depuis Windows Server 2008 SP2 |
| **Type** | Service de déploiement à distance |

#### Utilités de WDS

> [!info] Fonctions principales
> - **Déploiement d'images WIM** via le réseau
> - **Boot PXE** : Fourniture d'images de démarrage par TFTP
> - **Automatisation** : Utilisation de fichiers de réponse XML

#### Configuration requise

> [!important] Prérequis serveur
> **Sur un serveur Windows :**
> - WDS est un **rôle à ajouter** sur un serveur Windows
> - Une **partition dédiée** doit être allouée à ce rôle
> - Le rôle **AD DS n'est pas obligatoire** (peut fonctionner en standalone)
> - Un **serveur DHCP doit être disponible** sur le réseau

---

### MDT (Microsoft Deployment Toolkit)

> [!quote] Définition
> **MDT** (Microsoft Deployment Toolkit) est une solution gratuite de déploiement de Microsoft fonctionnant à l'aide de séquences de tâches. Doit être utilisé avec **WADK** pour fonctionner.

#### Deux modes d'utilisation

| Mode | Utilisation | Niveau d'automatisation | Intervention humaine |
|------|-------------|------------------------|---------------------|
| **LiteTouch** | MDT seul | Partielle | Requise (sélections manuelles) |
| **ZeroTouch** | MDT + SCCM | Complète | Aucune |

> [!warning] Limitation importante
> MDT n'installe **pas de service TFTP** ⇒ pas de démarrage PXE possible seul.
> 
> **Solution** : Utiliser MDT et WDS ensemble pour combiner leurs avantages.

#### Fonctionnalités de MDT

> [!info] Automatisations possibles
> MDT est **beaucoup plus personnalisable que WDS** :
> 
> **Configuration système :**
> - Nom de l'ordinateur
> - Jonction au domaine Active Directory
> - Installation de pilotes matériels
> 
> **Applications et scripts :**
> - Installation d'applications sélectionnées ou en mode transparent
> - Exécution de scripts (VBS, CMD, PowerShell)
> - Configuration via séquences de tâches
> 
> **Sécurité et migration :**
> - Sauvegarde / Restauration des profils utilisateurs
> - Activation de BitLocker (chiffrement de disque)

#### Utilité principale

> [!tip] Cas d'usage
> MDT permet :
> - **Automatisation de la fabrication d'images** (création de masters personnalisés)
> - **Automatisation de l'installation d'images** (déploiement sur le parc)

---

### WADK (Windows Assessment and Deployment Kit)

> [!quote] Définition
> **WADK** (Windows Assessment and Deployment Kit) est une suite d'outils conçue par Microsoft pour le déploiement d'OS. Anciennement appelé **WAIK** (Windows Automated Installation Kit).

#### Composants principaux du WADK

| Outil | Nom complet | Fonction |
|-------|-------------|----------|
| **USMT** | User State Migration Tool | Migration de données utilisateur |
| **ACT** | Application Compatibility Toolkit | Évaluation de compatibilité des applications post-migration |
| **DISM** | Deployment Image Servicing and Management | Gestion d'images d'OS (pilotes, packages, etc.) |
| **WinPE** | Windows Preinstallation Environment | Environnement de préinstallation Windows pour déploiement |
| **VAMT** | Volume Activation Management Tool | Activation de logiciels (clés MAK ou KMS) |
| **Sysprep** | System Preparation Tool | Initialisation d'OS pour capture d'image |

> [!note] Types de clés d'activation
> - **MAK** : Clés d'Activation Multiple (Multiple Activation Key) - nombre limité d'activations
> - **KMS** : Service de Gestion de Clés (Key Management Service) - activation centralisée réseau

---

### SCCM (System Center Configuration Manager)

> [!quote] Définition
> **SCCM** (System Center Configuration Manager) est un logiciel de gestion de système de la suite System Center conçu par Microsoft, plutôt utilisé pour de **grands parcs informatiques**. Licence propriétaire.

#### Composition

> [!important] Intégration complète
> SCCM **contient WDS et MDT** et fournit un déploiement d'OS complet par images WIM.

#### Fonctionnalités étendues

> [!info] Au-delà du déploiement
> **Gestion complète du parc :**
> - Prise de main à distance
> - Gestion de correctifs (patch management)
> - Automatisation de tâches
> - Télédistribution d'applications
> - Inventaire matériel et logiciel
> - Gestion de la conformité
> - Administration des politiques de sécurité

#### Architecture SCCM

> [!example] Architecture hiérarchique
> 
> **CAS - Site Central d'Administration :**
> - Gère les mises à jour des sites primaires
> - Administration globale (reporting)
> - → Supporte jusqu'à **25 sites primaires**
> 
> **Site primaire :**
> - Prend en charge les sites secondaires enfants (→ 250 max)
> - Peut être Point de Distribution (DP)
> - → Gère jusqu'à **100 000 clients**
> 
> **Site secondaire :**
> - Géré par serveur de site primaire
> - Possède sa propre base de données SQL
> - Déploie les clients SCCM
> - Peut être Point de Distribution (DP)
> - → Gère jusqu'à **5 000 clients**
> 
> **DP - Point de Distribution :**
> - Envoie le contenu aux clients
> - → Distribue vers **4 000 clients**

```
┌─────────────────────────────────────┐
│ CAS (Central Administration Site)   │
│ • Reporting global                  │
│ • MAJ sites primaires               │
│ → 25 sites primaires max            │
└────────────┬────────────────────────┘
             │
    ┌────────┴────────┐
    │                 │
┌───▼─────────┐  ┌───▼─────────┐
│ Site        │  │ Site        │
│ Primaire 1  │  │ Primaire 2  │
│ 100k clients│  │ 100k clients│
└──┬──────┬───┘  └─────────────┘
   │      │
   │      └──────┐
   ▼             ▼
┌──────────┐  ┌──────────┐
│ Site     │  │ DP       │
│Secondaire│  │ 4k       │
│ 5k       │  │ clients  │
│ clients  │  │          │
└────┬─────┘  └──────────┘
     │
     ▼
  Clients
```

> [!tip] Cas d'usage SCCM
> SCCM est adapté pour :
> - Grandes entreprises (plusieurs centaines/milliers de postes)
> - Besoin de gestion centralisée complète
> - Budget disponible (licence payante)
> - Équipe IT dédiée à l'administration

---

## Le déploiement Windows

### Déploiement standard

> [!info] Processus classique sans automatisation
> **Étapes d'un déploiement Windows standard :**
> 1. **Démarrage** du poste sur un support bootable (clé USB, DVD)
> 2. **Chargement de WinPE** (environnement de préinstallation)
> 3. **Assistant d'installation de Windows** (sélections manuelles)
> 4. **Installation de Windows** sur l'ordinateur

> [!warning] Limites du déploiement standard
> - Intervention manuelle requise sur chaque poste
> - Temps de déploiement long pour un parc
> - Risque d'erreurs humaines
> - Configuration non standardisée

---

### Le master (image de référence)

> [!quote] Définition
> Le **master** est une image disque de référence capturée à partir d'un ordinateur de référence configuré selon les besoins de l'entreprise.

#### Les 3 types d'images master

> [!important] Stratégies d'images

| Type | Contenu | Avantages | Inconvénients | Cas d'usage |
|------|---------|-----------|---------------|-------------|
| **Thin Image** | OS uniquement | • Légère<br>• Flexible<br>• Facile à maintenir | • Logiciels déployés après<br>• Temps total plus long | Environnements avec GPO/MDT pour installation logiciels |
| **Thick Image** | OS + tous les logiciels | • Déploiement rapide<br>• Prêt à l'emploi | • Lourde<br>• Refaire l'image à chaque MAJ | Postes standardisés identiques |
| **Hybrid Image** | OS + logiciels de base | • Équilibre<br>• Base fonctionnelle | • Compromis entre taille et MAJ | **Recommandée** : OS + antivirus, navigateur, bureautique |

> [!tip] Choix du type d'image
> **En tant que TSSR, privilégiez l'Hybrid Image pour :**
> - Avoir une base fonctionnelle immédiate
> - Maintenir une flexibilité pour les logiciels métiers
> - Réduire le temps de déploiement tout en facilitant les mises à jour

> [!example] Contenu typique d'une Hybrid Image
> **OS :** Windows 10/11 Professionnel
> **Logiciels de base :**
> - Antivirus d'entreprise (ex: Windows Defender ATP, Symantec)
> - Navigateur web (Edge, Chrome, Firefox)
> - Suite bureautique (Microsoft Office, LibreOffice)
> - Outils de communication (Teams, Zoom)
> - Lecteur PDF (Adobe Reader, Foxit)
> 
> **Logiciels métiers :** Déployés après via GPO, SCCM, ou scripts

---

### Sysprep

> [!quote] Définition
> **Sysprep** (System Preparation Tool) est l'outil qui permet de faire le "rescellement" d'une image Windows afin d'obtenir un master déployable.

#### Commande Sysprep

```cmd
C:\Windows\System32\sysprep\sysprep.exe /oobe /generalize /shutdown
```

#### Paramètres expliqués

| Paramètre | Signification | Effet |
|-----------|---------------|-------|
| `/oobe` | Out-Of-Box Experience | Lance la séquence de personnalisation au prochain démarrage |
| `/generalize` | Généralisation | Permet le déploiement sur différents types de matériel<br>• Supprime le SID unique<br>• Nettoie les pilotes spécifiques<br>• Réinitialise les informations système |
| `/shutdown` | Arrêt | Éteint l'ordinateur après le sysprep |

> [!warning] RÈGLE ABSOLUE
> **Après un sysprep, il ne faut JAMAIS rallumer le PC manuellement !**
> 
> Si le PC redémarre, il va :
> - Générer un nouveau SID
> - Reconfigurer le système
> - **Rendre l'image inutilisable** pour le déploiement
> 
> **Action après sysprep :**
> 1. Le PC s'éteint automatiquement
> 2. Démarrer sur WinPE (clé USB, réseau PXE)
> 3. Capturer l'image avec DISM ou MDT
> 4. Ne JAMAIS démarrer sur le disque dur

> [!tip] Bonnes pratiques Sysprep
> **Avant de lancer Sysprep :**
> 1. Installer tous les logiciels souhaités
> 2. Configurer les paramètres système de base
> 3. Vérifier que tout fonctionne correctement
> 4. Créer un point de restauration (au cas où)
> 5. Exécuter Sysprep
> 6. Capturer immédiatement l'image

---

### WinPE (Windows Preinstallation Environment)

> [!quote] Définition
> **WinPE** (Windows Preinstallation Environment) est un noyau système léger d'un OS Windows, utilisé pour l'installation d'un Windows "complet".

#### Caractéristiques de WinPE

| Aspect | Description |
|--------|-------------|
| **Existence** | Depuis Windows XP |
| **Exécution** | À partir du réseau ou d'un support amovible (USB, CD) |
| **Objectif** | Installation d'un Windows complet |
| **Remplacement** | Créé pour remplacer MS-DOS avec beaucoup plus de fonctionnalités |
| **Disponibilité** | Fourni avec MDT et WDS |

#### Commandes essentielles WinPE - DiskPart

> [!important] Utilitaire de ligne de commande pour la gestion des disques

```cmd
diskpart                    # Lancer l'utilitaire

# --- Affichage ---
list disk                   # Affiche la liste des disques physiques
list volume                 # Affiche tous les volumes (partitions)
list partition              # Affiche les partitions du disque sélectionné

# --- Sélection ---
select disk 0               # Sélectionne le disque 0
select volume 1             # Sélectionne le volume 1
select partition 1          # Sélectionne la partition 1

# --- Nettoyage ---
clean                       # Détruit TOUTES les partitions du disque sélectionné

# --- Création ---
create partition primary size=100000    # Partition primaire de 100 Go
create partition extended               # Partition étendue
create volume simple size=50000        # Volume simple de 50 Go

# --- Formatage ---
format fs=ntfs quick label="Windows"    # Formatage rapide NTFS
format fs=fat32 quick                   # Formatage FAT32

# --- Activation ---
active                      # Active la partition (bootable)

# --- Attribution ---
assign letter=C             # Attribue la lettre C:
remove letter=D             # Retire la lettre D:

# --- Exemple complet de préparation disque ---
diskpart
select disk 0
clean
create partition primary size=500
format fs=fat32 quick label="System"
active
assign letter=S
create partition primary
format fs=ntfs quick label="Windows"
assign letter=C
exit
```

> [!warning] Danger de la commande clean
> `clean` détruit **irrémédiablement** toutes les partitions et données du disque sélectionné.
> Toujours vérifier avec `list disk` quel disque est sélectionné avant d'exécuter `clean`.

> [!example] Cas d'usage typique WinPE
> **Scénario : Installation Windows sur une nouvelle machine**
> 
> 1. **Boot** sur WinPE (via PXE ou clé USB)
> 2. **DiskPart** : Préparer les partitions
> 3. **DISM** : Appliquer l'image WIM sur C:
> 4. **BcdBoot** : Configurer le boot
> 5. **Redémarrage** : L'ordinateur démarre sur Windows installé

---

### PXE (Preboot Execution Environment)

> [!quote] Définition
> **PXE** (Preboot Execution Environment) permet de démarrer un ordinateur à partir du réseau en récupérant une image d'OS. L'OS peut être utilisé directement ou installé.

> [!info] PXE n'est pas lié à Windows
> PXE est un standard industriel. Le projet **Syslinux** (PXELinux) permet de faire du boot réseau pour installer des OS Linux.

#### Les 3 étapes du boot PXE

> [!important] Processus de boot réseau
> 
> 1. **Demande d'adresse IP et du fichier d'amorçage** auprès d'un serveur DHCP/BOOTP
> 2. **Téléchargement du fichier à amorcer** depuis un serveur TFTP
> 3. **Exécution du fichier d'amorce** (généralement WinPE)

#### Détails du protocole Boot PXE

> [!example] Échanges réseau détaillés

```
┌─────────┐         ┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│ Client  │         │ Serveur DHCP │         │ Serveur TFTP │         │ Serveur HTTP │
└────┬────┘         └──────┬───────┘         └──────┬───────┘         └──────┬───────┘
     │                     │                        │                        │
     │ 1. DHCP-DISCOVER    │                        │                        │
     │  (UDP 67)           │                        │                        │
     │  Demande @IP +      │                        │                        │
     │  amorce BootP       │                        │                        │
     │────────────────────>│                        │                        │
     │                     │                        │                        │
     │ 2. DHCP-OFFER       │                        │                        │
     │  (UDP 68)           │                        │                        │
     │  @IP + option PXE   │                        │                        │
     │  (60, 66, 67)       │                        │                        │
     │<────────────────────│                        │                        │
     │                     │                        │                        │
     │ 3. DHCP-REQUEST     │                        │                        │
     │  (UDP)              │                        │                        │
     │  Confirmation @IP   │                        │                        │
     │────────────────────>│                        │                        │
     │                     │                        │                        │
     │ 4. DHCP-ACK         │                        │                        │
     │  (UDP)              │                        │                        │
     │  Acquittement +     │                        │                        │
     │  début bail IP      │                        │                        │
     │<────────────────────│                        │                        │
     │                     │                        │                        │
     │ 5. NBP (Network BootStrap Protocol) (UDP)    │                        │
     │  Demande chargement fichier d'amorce         │                        │
     │─────────────────────────────────────────────>│                        │
     │                     │                        │                        │
     │ 6. Envoi du fichier d'amorce (TFTP)          │                        │
     │  (boot.wim - WinPE)                          │                        │
     │<─────────────────────────────────────────────│                        │
     │                     │                        │                        │
     │ 7. Requête de l'installateur                                          │
     │  (HTTP/SMB)                                                            │
     │───────────────────────────────────────────────────────────────────────>│
     │                     │                        │                        │
     │ 8. Délivrance de l'installateur                                       │
     │  (install.wim)                                                         │
     │<───────────────────────────────────────────────────────────────────────│
```

#### Options DHCP pour PXE

> [!note] Options de retour DHCP essentielles

| Option | Nom | Description |
|--------|-----|-------------|
| **60** | Class Identifier | Localisation par le client ("PXEClient") |
| **66** | TFTP Server Name | Nom ou adresse IP du serveur TFTP |
| **67** | Bootfile Name | Nom du fichier d'amorce à télécharger (ex: `boot\x64\wdsnbp.com`) |

> [!tip] Configuration DHCP pour PXE
> **Sur un serveur DHCP Windows :**
> 
> 1. Ouvrir console DHCP
> 2. Clic droit sur **IPv4** → **Propriétés**
> 3. Onglet **Avancé** → Cocher "Serveur PXE"
> 4. Ou configurer manuellement :
>    - Option 66 : `192.168.1.10` (IP du serveur WDS/TFTP)
>    - Option 67 : `boot\x64\wdsnbp.com` (pour architecture 64 bits)

---

### WIM (Windows Imaging Format)

> [!quote] Définition
> **WIM** (Windows Imaging Format) est un type de fichier image disque dur développé par Microsoft. Extensions : `.wim` ou `.swm` (split WIM).

#### Caractéristiques du format WIM

| Aspect | Description |
|--------|-------------|
| **Existence** | Depuis Windows XP SP2 |
| **Orientation** | Fichiers et métadonnées (vs secteurs pour .ghost, .cue) |
| **Indépendance** | Format indépendant du matériel |
| **Multi-images** | 1 fichier WIM peut contenir **plusieurs images** |
| **Compression** | Support de plusieurs niveaux de compression |

#### Utilisation des images WIM

> [!info] Compatibilité et manipulation
> 
> **Utilisable avec :**
> - WADK (DISM, ImageX)
> - MDT
> - WinPE
> 
> **Opérations possibles :**
> - Montage dans un dossier (exploration/modification)
> - Rendu bootable (application sur partition)

#### Fichiers WIM par défaut Windows

> [!example] Localisation dans l'ISO Windows
> Les fichiers suivants sont dans le dossier **`sources`** de l'ISO d'installation Windows :
> 
> - **`install.wim`** : Contient les éditions Windows (Home, Pro, Enterprise...)
> - **`boot.wim`** : Contient le WinPE utilisé pour l'installation

---

### DISM - Gestion des images WIM

> [!quote] Définition
> **DISM** (Deployment Image Servicing and Management) est l'outil de gestion d'images Microsoft qui remplace **ImageX**. Disponible dans WADK.

#### Outils de gestion WIM

| Outil | Statut | Description |
|-------|--------|-------------|
| **ImageX** | Ancien | Disponible sur WADK (anciennes versions) |
| **DISM** | Actuel | Remplaçant d'ImageX avec plus de fonctionnalités |

#### Actions disponibles sur les WIM

> [!important] Opérations DISM sur fichiers WIM

| Action | Description |
|--------|-------------|
| **Mount** | Montage de l'image dans un dossier |
| **Unmount** | Démontage de l'image (avec ou sans sauvegarde) |
| **Capture** | Copie la structure de données vers un nouveau WIM |
| **Append** | Ajoute une image dans un WIM existant |
| **Export** | Extrait une image d'un WIM vers un nouveau WIM |
| **Delete** | Suppression d'une image dans un WIM multi-images |
| **Info** | Affichage des métadonnées XML de l'image |
| **Apply** | Application d'une image WIM sur une partition |

#### Commandes DISM essentielles

> [!example] Lignes de commande DISM

**1. Lister les images dans un WIM :**
```cmd
DISM /Get-WimInfo /WimFile:C:\Sources\install.wim
```

**2. Montage d'une image WIM :**
```cmd
DISM /Mount-Wim /WimFile:C:\Sources\install.wim /Index:1 /MountDir:C:\Mount
```
- `/WimFile:` : Chemin du fichier WIM source
- `/Index:` : Numéro de l'image à monter (voir avec `/Get-WimInfo`)
- `/MountDir:` : Dossier de destination (doit exister et être vide)

**3. Enregistrer les modifications (commit) :**
```cmd
DISM /Unmount-Wim /MountDir:C:\Mount /Commit
```

**4. Annuler les modifications (discard) :**
```cmd
DISM /Unmount-Wim /MountDir:C:\Mount /Discard
```

**5. Capturer une image :**
```cmd
DISM /Capture-Image /ImageFile:C:\Images\custom.wim /CaptureDir:C:\ /Name:"Windows 10 Pro Personnalisé"
```

**6. Appliquer une image sur une partition :**
```cmd
DISM /Apply-Image /ImageFile:C:\Images\install.wim /Index:1 /ApplyDir:D:\
```

**7. Ajouter une image à un WIM existant :**
```cmd
DISM /Append-Image /ImageFile:C:\Images\multi.wim /CaptureDir:C:\ /Name:"Config Marketing"
```

**8. Supprimer une image d'un WIM :**
```cmd
DISM /Delete-Image /ImageFile:C:\Images\install.wim /Index:2
```

**9. Exporter une image :**
```cmd
DISM /Export-Image /SourceImageFile:C:\Sources\install.wim /SourceIndex:4 /DestinationImageFile:C:\Images\enterprise.wim
```

> [!tip] Modification d'une image WIM
> **Workflow typique pour personnaliser une image :**
> 
> 1. **Monter** l'image
> ```cmd
> DISM /Mount-Wim /WimFile:install.wim /Index:1 /MountDir:C:\Mount
> ```
> 
> 2. **Modifier** le contenu (ajouter pilotes, applications, etc.)
> ```cmd
> # Ajouter un pilote
> DISM /Image:C:\Mount /Add-Driver /Driver:C:\Drivers\network.inf
> 
> # Ajouter un package
> DISM /Image:C:\Mount /Add-Package /PackagePath:C:\Packages\update.cab
> ```
> 
> 3. **Démonter avec commit**
> ```cmd
> DISM /Unmount-Wim /MountDir:C:\Mount /Commit
> ```

> [!warning] Pièges DISM courants
> - **Dossier de montage non vide** : Le `/MountDir` doit être un dossier vide
> - **Oublier /Commit** : Utiliser `/Discard` annule toutes les modifications
> - **Montage bloqué** : Si DISM plante, nettoyer avec `DISM /Cleanup-Wim`
> - **Index incorrect** : Toujours vérifier l'index avec `/Get-WimInfo` avant

#### Structure interne d'un fichier WIM

> [!note] Anatomie d'un fichier WIM

```
┌─────────────────────────────────────┐
│         En-tête WIM                 │  ← Informations générales
├─────────────────────────────────────┤
│     Ressources de fichiers          │
│  ┌─────────────────────────────┐   │
│  │ Blocs de données compressés │   │  ← Données réelles des fichiers
│  └─────────────────────────────┘   │
├─────────────────────────────────────┤
│   Métadonnées des ressources        │
│  ┌─────────────────────────────┐   │
│  │ Descripteurs de structures  │   │  ← Arborescence dossiers,
│  │ (dossiers, sécurité, ...)   │   │    permissions, attributs
│  └─────────────────────────────┘   │
├─────────────────────────────────────┤
│      Table d'adresses               │  ← Index des ressources
├─────────────────────────────────────┤
│        Données XML                  │  ← Métadonnées de l'image
│  ┌─────────────────────────────┐   │    (nom, description, version)
│  │ Caractéristiques et infos   │   │
│  │ sur l'image                 │   │
│  └─────────────────────────────┘   │
├─────────────────────────────────────┤
│     Table d'intégrité               │  ← Checksums pour vérification
└─────────────────────────────────────┘
```

> [!info] Avantages du format WIM
> - **Déduplication** : Fichiers identiques stockés une seule fois
> - **Compression** : Économie d'espace disque
> - **Multi-images** : Plusieurs éditions Windows dans un seul fichier
> - **Non destructif** : Montage sans modifier l'image source
> - **Indépendant du matériel** : Déployable sur différentes configurations

---

## Schéma de synthèse global

> [!abstract] Vue d'ensemble du processus de déploiement automatisé

```
┌─────────────────────────────────────────────────────────────────┐
│                    WADK (Windows Assessment                     │
│                 and Deployment Kit)                             │
│  ┌──────┬───────┬────────┬────────┬─────────┬─────────┐        │
│  │ USMT │  ACT  │  DISM  │ WinPE  │  VAMT   │ Sysprep │        │
│  └──────┴───────┴────────┴────────┴─────────┴─────────┘        │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ Fournit les outils
                 ▼
┌────────────────────────────────────────────────────────────────┐
│                  WDS (Windows Deployment Services)             │
│  • WinPE via PXE (TFTP)                                        │
│  • Fichiers de réponse XML                                     │
│  • Déploiement images WIM                                      │
└────────────┬───────────────────────────────────────────────────┘
             │
             │ Peut être combiné avec
             ▼
┌────────────────────────────────────────────────────────────────┐
│          MDT (Microsoft Deployment Toolkit)                    │
│  • WinPE personnalisé                                          │
│  • Séquences de tâches                                         │
│  • Fichiers OS, Drivers, Applications                          │
│                                                                 │
│  ┌─────────────────┐              ┌──────────────────┐        │
│  │   LiteTouch     │              │   ZeroTouch      │        │
│  │ (MDT seul)      │              │ (MDT + SCCM)     │        │
│  │ Interventions   │              │ Aucune           │        │
│  │ humaines        │              │ intervention     │        │
│  └─────────────────┘              └──────────────────┘        │
└────────────┬───────────────────────────────────────────────────┘
             │
             │ Intégré dans
             ▼
┌────────────────────────────────────────────────────────────────┐
│   SCCM (System Center Configuration Manager)                  │
│  • Contient WDS + MDT                                          │
│  • Gestion complète du parc                                    │
│  • Architecture hiérarchique (CAS → Sites → DP)                │
│  • ZeroTouch automatique                                       │
└────────────────────────────────────────────────────────────────┘

                        ▼
              ┌──────────────────┐
              │  Image WIM       │
              │  (DISM)          │
              │                  │
              │  • install.wim   │
              │  • boot.wim      │
              └────────┬─────────┘
                       │
                       │ Déployée via
                       ▼
              ┌──────────────────┐
              │  Client final    │
              │  • Boot PXE      │
              │  • WinPE         │
              │  • Installation  │
              │  • OOBE          │
              └──────────────────┘
```

> [!success] Workflow de déploiement complet
> 
> **Préparation de l'image :**
> 1. Installer Windows sur un poste de référence
> 2. Installer logiciels de base (Hybrid Image)
> 3. Configurer le système
> 4. Exécuter **Sysprep** `/oobe /generalize /shutdown`
> 5. Capturer l'image avec **DISM** (fichier .wim)
> 
> **Configuration de l'infrastructure :**
> 6. Installer **WDS** sur un serveur Windows
> 7. Configurer **DHCP** avec options PXE (66, 67)
> 8. Installer **MDT** et **WADK** (optionnel, pour plus de fonctionnalités)
> 9. Importer l'image WIM dans WDS/MDT
> 10. Créer séquences de tâches (MDT) ou fichiers de réponse (WDS)
> 
> **Déploiement sur les clients :**
> 11. Démarrer le poste client en **PXE** (F12 au boot)
> 12. Le poste récupère **WinPE** via TFTP
> 13. WinPE lance l'installation (automatique ou interactive)
> 14. L'image WIM est appliquée sur le disque
> 15. Le poste redémarre et passe en **OOBE**
> 16. Configuration finale (nom, domaine, utilisateur)

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **Sysprep** est **obligatoire** avant toute capture d'image pour supprimer le SID et dépersonnaliser le système
- Un **master** peut être **Thin** (OS seul), **Thick** (OS+tout), ou **Hybrid** (OS+base) - privilégier Hybrid
- **WinPE** est l'environnement léger utilisé pour installer Windows (remplace MS-DOS)
- **PXE** permet le boot réseau via DHCP + TFTP pour démarrer WinPE sans support physique

### Outils de déploiement

- **WDS** : Service Windows de base pour déploiement réseau (PXE + WIM)
- **MDT** : Outil gratuit de personnalisation avancée (séquences de tâches)
  - **LiteTouch** = MDT seul (interventions manuelles)
  - **ZeroTouch** = MDT + SCCM (automatique)
- **WADK** : Suite d'outils (DISM, WinPE, Sysprep, USMT, etc.)
- **SCCM** : Solution complète payante pour grands parcs (contient WDS+MDT)

### Format et gestion d'images

- **WIM** : Format d'image Windows (multi-images, compressé, indépendant du matériel)
- **DISM** : Outil pour manipuler les WIM (mount, capture, apply, append, export)
- **install.wim** et **boot.wim** sont dans le dossier `sources` de l'ISO Windows

### Boot réseau PXE

- **Étapes PXE** : DHCP (IP + options 66/67) → TFTP (fichier boot) → Exécution WinPE
- **Options DHCP** :
  - 60 : Identification client PXE
  - 66 : Adresse serveur TFTP
  - 67 : Nom fichier d'amorce

### Commandes critiques

```cmd
# Sysprep (sur le master)
C:\Windows\System32\sysprep\sysprep.exe /oobe /generalize /shutdown

# DISM - Lister images
DISM /Get-WimInfo /WimFile:install.wim

# DISM - Monter image
DISM /Mount-Wim /WimFile:install.wim /Index:1 /MountDir:C:\Mount

# DISM - Démonter avec sauvegarde
DISM /Unmount-Wim /MountDir:C:\Mount /Commit

# DISM - Capturer image
DISM /Capture-Image /ImageFile:custom.wim /CaptureDir:C:\ /Name:"Mon Image"

# DISM - Appliquer image
DISM /Apply-Image /ImageFile:install.wim /Index:1 /ApplyDir:D:\
```

### Architecture et dimensionnement

**SCCM :**
- CAS → 25 sites primaires
- Site primaire → 100k clients, 250 sites secondaires
- Site secondaire → 5k clients
- DP → 4k clients

### Pièges à éviter

- ❌ Ne **jamais** démarrer un PC après Sysprep (génère nouveau SID)
- ❌ Ne **jamais** dupliquer directement un poste sans Sysprep (SID identiques)
- ❌ Ne **jamais** utiliser `diskpart clean` sans vérifier le disque sélectionné
- ❌ Ne **jamais** oublier `/Commit` lors du démontage d'image DISM (perte modifications)
- ⚠️ MDT seul ne fait **pas** de PXE (besoin de WDS en complément)

### Cas d'usage par outil

| Besoin | Outil recommandé |
|--------|------------------|
| Petit parc (<50 postes), déploiement simple | **WDS** seul |
| Parc moyen (<500 postes), personnalisation | **WDS + MDT** (LiteTouch) |
| Grand parc (>500 postes), automatisation complète | **SCCM** (ZeroTouch) |
| Gestion complète parc + sécurité + inventaire | **SCCM** |

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **ACT** | Application Compatibility Toolkit - Outil d'évaluation de compatibilité des applications après migration d'OS |
| **Append** | Action DISM d'ajout d'une image dans un WIM existant (vs Capture qui crée un nouveau WIM) |
| **BootP** | Bootstrap Protocol - Protocole permettant à un ordinateur d'obtenir son adresse IP au démarrage |
| **CAS** | Central Administration Site - Site central d'administration SCCM gérant jusqu'à 25 sites primaires |
| **Commit** | Action de sauvegarde des modifications lors du démontage d'une image DISM |
| **DHCP** | Dynamic Host Configuration Protocol - Protocole d'attribution automatique d'adresses IP |
| **DISM** | Deployment Image Servicing and Management - Outil de gestion d'images Windows (remplace ImageX) |
| **DiskPart** | Utilitaire en ligne de commande pour la gestion des disques et partitions |
| **DP** | Distribution Point - Point de distribution SCCM distribuant le contenu vers ~4000 clients |
| **Hybrid Image** | Image master contenant l'OS + logiciels de base (antivirus, bureautique, navigateur) |
| **ImageX** | Ancien outil de gestion d'images WIM, remplacé par DISM |
| **KMS** | Key Management Service - Service de gestion de clés pour activation centralisée Windows |
| **LiteTouch** | Mode MDT avec interventions humaines (sélections manuelles) |
| **MAK** | Multiple Activation Key - Clé d'activation à usage multiple (nombre limité) |
| **Master** | Image disque de référence capturée depuis un poste configuré et sysprep |
| **MDT** | Microsoft Deployment Toolkit - Solution gratuite de déploiement par séquences de tâches |
| **Mount** | Action de montage d'une image WIM dans un dossier pour consultation/modification |
| **OOBE** | Out-Of-Box Experience - Écran de première configuration Windows (langue, compte, réseau) |
| **PXE** | Preboot Execution Environment - Standard permettant le boot réseau via DHCP/TFTP |
| **RIS** | Remote Installation Service - Ancien service de déploiement Windows, remplacé par WDS |
| **SCCM** | System Center Configuration Manager - Solution complète de gestion de parc (payante) |
| **SID** | Security Identifier - Identifiant unique de sécurité d'une machine/utilisateur Windows |
| **Site primaire** | Site SCCM gérant jusqu'à 100k clients et 250 sites secondaires enfants |
| **Site secondaire** | Site SCCM gérant jusqu'à 5k clients, possède sa propre base SQL |
| **Sysprep** | System Preparation Tool - Outil de dépersonnalisation d'un système avant capture d'image |
| **TFTP** | Trivial File Transfer Protocol - Protocole simple de transfert de fichiers (utilisé par PXE) |
| **Thick Image** | Image master contenant l'OS + tous les logiciels (lourde, peu flexible) |
| **Thin Image** | Image master contenant uniquement l'OS (légère, logiciels déployés après) |
| **USMT** | User State Migration Tool - Outil de migration de données utilisateur entre postes |
| **VAMT** | Volume Activation Management Tool - Outil de gestion d'activation en volume (MAK/KMS) |
| **WADK** | Windows Assessment and Deployment Kit - Suite d'outils Microsoft pour déploiement (ex-WAIK) |
| **WAIK** | Windows Automated Installation Kit - Ancien nom du WADK |
| **WDS** | Windows Deployment Services - Rôle serveur Windows pour déploiement réseau (PXE + WIM) |
| **WIM** | Windows Imaging Format - Format d'image disque Microsoft (.wim, multi-images, compressé) |
| **WinPE** | Windows Preinstallation Environment - Noyau Windows léger pour installation/maintenance |
| **ZeroTouch** | Mode MDT+SCCM sans intervention humaine (entièrement automatisé) |

---

> [!success] Conclusion
> En tant que **TSSR**, la maîtrise du déploiement automatisé Windows est une compétence différenciante. Vous devez savoir :
> 
> ✅ Différencier **WDS**, **MDT**, et **SCCM** et choisir selon le contexte
> ✅ Préparer une **image master** avec **Sysprep** correctement
> ✅ Manipuler des fichiers **WIM** avec **DISM**
> ✅ Comprendre le processus **PXE** et configurer **DHCP/TFTP**
> ✅ Créer des séquences de tâches **MDT** pour personnaliser les déploiements
> 
> Cette compétence est **directement valorisable** lors de votre titre RNCP et en entreprise pour gérer efficacement le déploiement de postes à grande échelle.

---

**Document créé pour la préparation du titre TSSR - Février 2025** 📚✨
