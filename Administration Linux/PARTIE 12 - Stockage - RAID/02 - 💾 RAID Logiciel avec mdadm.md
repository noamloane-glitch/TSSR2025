
---

## 📑 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## 🔧 1. Installation de mdadm

### Qu'est-ce que mdadm ?

**mdadm** (Multiple Device Administrator) est l'outil de référence pour gérer le RAID logiciel sous Linux. Il permet de créer, assembler, gérer et surveiller les arrays RAID sans nécessiter de carte RAID matérielle.

### Installation selon la distribution

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install mdadm

# Red Hat/CentOS/Fedora
sudo dnf install mdadm
# ou pour les anciennes versions
sudo yum install mdadm

# Arch Linux
sudo pacman -S mdadm
```

### Vérification de l'installation

```bash
# Vérifier la version installée
mdadm --version

# Vérifier que le module kernel est chargé
lsmod | grep raid
```

> [!info] Module kernel Le support RAID est généralement compilé dans le noyau Linux moderne. Les modules `raid0`, `raid1`, `raid456`, `raid10` sont chargés automatiquement selon les besoins.

---

## 🏗️ 2. Création d'un array RAID

### Syntaxe de base

```bash
mdadm --create /dev/md<N> \
      --level=<type_raid> \
      --raid-devices=<nombre> \
      /dev/sd<X> /dev/sd<Y> ...
```

### Paramètres principaux

|Paramètre|Description|Exemple|
|---|---|---|
|`--create`|Crée un nouvel array|`/dev/md0`|
|`--level`|Type de RAID|`0`, `1`, `5`, `6`, `10`|
|`--raid-devices`|Nombre de disques actifs|`2`, `3`, `4`...|
|`--spare-devices`|Nombre de disques de secours|`1`, `2`...|
|`--chunk`|Taille des chunks (RAID 0/5/6/10)|`64`, `128`, `512` (en Ko)|
|`--metadata`|Version des métadonnées|`1.2` (par défaut)|
|`--name`|Nom de l'array|`backup`, `data`|

### Exemples pratiques

#### RAID 1 (miroir) avec 2 disques

```bash
# Création d'un RAID 1 basique
sudo mdadm --create /dev/md0 \
           --level=1 \
           --raid-devices=2 \
           /dev/sdb /dev/sdc

# Avec un disque de spare
sudo mdadm --create /dev/md0 \
           --level=1 \
           --raid-devices=2 \
           --spare-devices=1 \
           /dev/sdb /dev/sdc /dev/sdd
```

#### RAID 5 avec 4 disques

```bash
# RAID 5 avec chunk size de 128 Ko
sudo mdadm --create /dev/md1 \
           --level=5 \
           --raid-devices=4 \
           --chunk=128 \
           /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

#### RAID 10 avec 4 disques

```bash
# RAID 10 (1+0)
sudo mdadm --create /dev/md2 \
           --level=10 \
           --raid-devices=4 \
           /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

> [!warning] Confirmation requise Lors de la création, mdadm demande confirmation car l'opération va effacer les données existantes sur les disques. Tapez `y` pour confirmer.

### Processus de synchronisation initiale

Après la création, l'array entre en phase de **resync** (synchronisation) :

```bash
# Observer la progression de la synchronisation
cat /proc/mdstat

# Exemple de sortie :
# md0 : active raid1 sdc[1] sdb[0]
#       104320 blocks super 1.2 [2/2] [UU]
#       [=====>...............]  resync = 28.2% (29440/104320)
```

> [!tip] Performance durant le resync La synchronisation initiale peut prendre du temps. Vous pouvez utiliser l'array pendant ce processus, mais les performances seront réduites.

### Options avancées

```bash
# Forcer la création sans confirmation
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 \
           /dev/sdb /dev/sdc --force

# Créer avec un nom personnalisé
sudo mdadm --create /dev/md/backup \
           --level=1 \
           --raid-devices=2 \
           --name=backup \
           /dev/sdb /dev/sdc

# Créer en mode assume-clean (pas de resync initial - DANGEREUX)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 \
           /dev/sdb /dev/sdc --assume-clean
```

> [!warning] Option --assume-clean N'utilisez `--assume-clean` que si vous êtes absolument certain que les disques sont vierges ou contiennent des données identiques. Sinon, vous risquez la corruption de données.

---

## 👁️ 3. Surveillance de l'état

### Commande mdadm --detail

La commande `--detail` fournit des informations complètes sur un array :

```bash
# Afficher les détails d'un array
sudo mdadm --detail /dev/md0
```

#### Exemple de sortie détaillée

```bash
/dev/md0:
           Version : 1.2
     Creation Time : Mon Jan 20 10:30:45 2026
        Raid Level : raid1
        Array Size : 104320 (101.89 MiB 106.82 MB)
     Used Dev Size : 104320 (101.89 MiB 106.82 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Mon Jan 20 10:35:12 2026
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : server:0
              UUID : a1b2c3d4:e5f6g7h8:i9j0k1l2:m3n4o5p6
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
```

> [!info] Champs importants
> 
> - **State** : `clean` (sain), `degraded` (dégradé), `recovering` (en récupération)
> - **Active Devices** : nombre de disques actifs
> - **Failed Devices** : nombre de disques défaillants
> - **UUID** : identifiant unique de l'array

### Fichier /proc/mdstat

Le fichier `/proc/mdstat` offre une vue synthétique de tous les arrays RAID :

```bash
# Visualisation simple
cat /proc/mdstat

# Surveillance en temps réel
watch -n 1 cat /proc/mdstat
```

#### Exemple de sortie

```bash
Personalities : [raid1] [raid6] [raid5] [raid4] 
md0 : active raid1 sdc[1] sdb[0]
      104320 blocks super 1.2 [2/2] [UU]
      
md1 : active raid5 sdf[3] sde[2] sdd[1] sdb[0]
      314572800 blocks super 1.2 level 5, 128k chunk, algorithm 2 [4/4] [UUUU]
      
unused devices: <none>
```

> [!example] Interprétation des symboles
> 
> - `[UU]` : tous les disques sont actifs
> - `[U_]` : un disque est manquant ou défaillant
> - `[___U]` : trois disques sont défaillants (array en panne)

### Surveillance continue avec --monitor

```bash
# Mode daemon pour surveiller les arrays
sudo mdadm --monitor --scan --daemonise

# Surveiller un array spécifique et envoyer des emails
sudo mdadm --monitor /dev/md0 --mail=admin@example.com --delay=1800

# Mode test sans fork
sudo mdadm --monitor --scan --test
```

|Option|Description|
|---|---|
|`--scan`|Surveille tous les arrays trouvés|
|`--daemonise`|Tourne en arrière-plan|
|`--mail`|Email de destination pour les alertes|
|`--delay`|Délai en secondes entre les vérifications|
|`--test`|Mode test (affiche sans envoyer d'emails)|

### Commandes de diagnostic

```bash
# Examiner les métadonnées d'un disque
sudo mdadm --examine /dev/sdb

# Vérifier tous les disques membres
sudo mdadm --examine --scan

# État bref de tous les arrays
sudo mdadm --detail --scan

# Statistiques détaillées
sudo mdadm --detail --scan --verbose
```

> [!tip] Automatisation de la surveillance Configurez le monitoring en daemon pour recevoir des alertes automatiques en cas de problème. Cela sera configuré via le fichier de configuration.

---

## ⚙️ 4. Configuration persistante

### Fichier /etc/mdadm/mdadm.conf

Le fichier de configuration `mdadm.conf` permet de :

- Identifier automatiquement les arrays au démarrage
- Configurer les alertes email
- Définir les politiques de création
- Stocker les préférences globales

### Localisation du fichier

|Distribution|Chemin|
|---|---|
|Debian/Ubuntu|`/etc/mdadm/mdadm.conf`|
|Red Hat/CentOS/Fedora|`/etc/mdadm.conf`|
|Arch Linux|`/etc/mdadm.conf`|

### Génération automatique de la configuration

```bash
# Générer la configuration pour tous les arrays actifs
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

# Ou avec la syntaxe complète
sudo mdadm --detail --scan --verbose | sudo tee /etc/mdadm/mdadm.conf
```

> [!warning] Écraser ou ajouter ?
> 
> - `>` écrase le fichier existant
> - `>>` ou `tee -a` ajoute au fichier existant
> 
> Vérifiez le contenu avant d'écraser !

### Structure du fichier mdadm.conf

```bash
# /etc/mdadm/mdadm.conf

# Déclaration des arrays avec UUID (recommandé)
ARRAY /dev/md0 UUID=a1b2c3d4:e5f6g7h8:i9j0k1l2:m3n4o5p6
ARRAY /dev/md1 UUID=b2c3d4e5:f6g7h8i9:j0k1l2m3:n4o5p6q7

# Alternative : déclaration par nom
ARRAY /dev/md/backup name=backup

# Alternative : déclaration par devices (moins recommandé)
ARRAY /dev/md2 devices=/dev/sdb,/dev/sdc,/dev/sdd

# Configuration des emails d'alerte
MAILADDR admin@example.com

# Programme d'envoi d'email (optionnel)
PROGRAM /usr/sbin/handle-mdadm-events

# Périphériques à créer automatiquement
CREATE owner=root group=disk mode=0660 auto=yes

# Homehost (nom de la machine)
HOMEHOST <system>

# Politique de métadonnées par défaut
METADATA 1.2
```

### Directives principales

#### ARRAY

Définit un array RAID :

```bash
# Par UUID (RECOMMANDÉ - le plus fiable)
ARRAY /dev/md0 UUID=a1b2c3d4:e5f6g7h8:i9j0k1l2:m3n4o5p6

# Par nom
ARRAY /dev/md/backup name=backup

# Par devices (déconseillé - peut changer si disques réorganisés)
ARRAY /dev/md1 devices=/dev/sdb,/dev/sdc

# Avec metadata spécifique
ARRAY /dev/md2 UUID=... metadata=1.2
```

> [!tip] Pourquoi utiliser l'UUID ? L'UUID est unique et ne change pas même si vous déplacez les disques ou changez leur ordre. C'est la méthode la plus fiable.

#### MAILADDR

Configure les alertes email :

```bash
# Email simple
MAILADDR admin@example.com

# Plusieurs destinataires (séparés par des virgules)
MAILADDR admin@example.com,sysadmin@example.com

# Désactiver les emails
MAILADDR root
```

#### CREATE

Définit les permissions par défaut pour les nouveaux arrays :

```bash
CREATE owner=root group=disk mode=0660 auto=yes metadata=1.2
```

|Paramètre|Description|Valeur par défaut|
|---|---|---|
|`owner`|Propriétaire|`root`|
|`group`|Groupe|`disk`|
|`mode`|Permissions|`0660`|
|`auto`|Détection auto|`yes`|
|`metadata`|Version metadata|`1.2`|

#### HOMEHOST

Identifie la machine hôte :

```bash
# Utiliser le nom de la machine
HOMEHOST <system>

# Nom spécifique
HOMEHOST server01

# Désactiver la vérification du hostname
HOMEHOST <ignore>
```

### Exemple de configuration complète

```bash
# /etc/mdadm/mdadm.conf - Configuration complète

# Arrays RAID définis par UUID
ARRAY /dev/md0 UUID=a1b2c3d4:e5f6g7h8:i9j0k1l2:m3n4o5p6
ARRAY /dev/md1 UUID=b2c3d4e5:f6g7h8i9:j0k1l2m3:n4o5p6q7
ARRAY /dev/md/backup name=backup UUID=c3d4e5f6:g7h8i9j0:k1l2m3n4:o5p6q7r8

# Configuration des alertes
MAILADDR admin@example.com

# Paramètres de création par défaut
CREATE owner=root group=disk mode=0660 auto=yes metadata=1.2

# Identification de l'hôte
HOMEHOST <system>

# Niveau de détail des logs
VERBOSE yes

# Metadata par défaut
METADATA 1.2
```

### Validation de la configuration

```bash
# Vérifier la syntaxe
sudo mdadm --detail --scan --config=/etc/mdadm/mdadm.conf

# Tester l'assemblage avec la configuration
sudo mdadm --assemble --scan --config=/etc/mdadm/mdadm.conf --test

# Afficher la configuration actuelle
sudo cat /etc/mdadm/mdadm.conf
```

> [!warning] Mise à jour obligatoire Après avoir créé un nouvel array, n'oubliez **jamais** de mettre à jour `mdadm.conf`, sinon l'array ne sera pas assemblé automatiquement au démarrage !

### Bonnes pratiques

```bash
# 1. Toujours faire une sauvegarde avant modification
sudo cp /etc/mdadm/mdadm.conf /etc/mdadm/mdadm.conf.backup

# 2. Regénérer proprement la configuration
sudo mdadm --detail --scan | sudo tee /etc/mdadm/mdadm.conf.new
sudo mv /etc/mdadm/mdadm.conf.new /etc/mdadm/mdadm.conf

# 3. Ajouter les paramètres supplémentaires manuellement
echo "MAILADDR admin@example.com" | sudo tee -a /etc/mdadm/mdadm.conf
```

---

## 🚀 5. Assemblage automatique au démarrage

### Principe de l'assemblage automatique

L'assemblage automatique garantit que vos arrays RAID sont disponibles dès le démarrage du système. Cela implique :

1. La configuration de mdadm.conf (vue précédemment)
2. La mise à jour de l'initramfs
3. L'activation du service systemd

### Mise à jour de l'initramfs

L'**initramfs** (initial RAM filesystem) est l'environnement de démarrage temporaire qui monte les systèmes de fichiers avant le système complet. Il doit contenir la configuration mdadm.

#### Debian/Ubuntu

```bash
# Mettre à jour l'initramfs
sudo update-initramfs -u

# Mettre à jour pour un kernel spécifique
sudo update-initramfs -u -k 5.15.0-56-generic

# Vérifier que mdadm est inclus
lsinitramfs /boot/initrd.img-$(uname -r) | grep mdadm
```

#### Red Hat/CentOS/Fedora

```bash
# Régénérer l'initramfs
sudo dracut --force

# Pour un kernel spécifique
sudo dracut --force /boot/initramfs-$(uname -r).img $(uname -r)

# Vérifier le contenu
lsinitrd /boot/initramfs-$(uname -r).img | grep mdadm
```

#### Arch Linux

```bash
# Régénérer avec mkinitcpio
sudo mkinitcpio -P

# Pour un preset spécifique
sudo mkinitcpio -p linux

# Vérifier la configuration
cat /etc/mkinitcpio.conf | grep mdadm
```

> [!info] Hook mdadm_udev Sous Arch Linux, assurez-vous que le hook `mdadm_udev` est présent dans `/etc/mkinitcpio.conf` :
> 
> ```bash
> HOOKS=(base udev autodetect modconf block mdadm_udev filesystems keyboard fsck)
> ```

### Configuration du service systemd

#### Vérifier et activer le service mdmonitor

```bash
# Vérifier le statut du service
sudo systemctl status mdmonitor

# Activer au démarrage
sudo systemctl enable mdmonitor

# Démarrer immédiatement
sudo systemctl start mdmonitor

# Vérifier qu'il est bien activé
sudo systemctl is-enabled mdmonitor
```

#### Commandes systemd pour mdadm

```bash
# Recharger la configuration systemd
sudo systemctl daemon-reload

# Redémarrer le monitoring
sudo systemctl restart mdmonitor

# Afficher les logs du service
sudo journalctl -u mdmonitor -f

# Afficher les dernières erreurs
sudo journalctl -u mdmonitor -p err
```

### Tests d'assemblage

#### Test à froid (après redémarrage)

```bash
# Vérifier que les arrays sont assemblés
cat /proc/mdstat

# Vérifier les détails
sudo mdadm --detail /dev/md0

# Vérifier tous les arrays configurés
sudo mdadm --detail --scan
```

#### Test manuel d'assemblage

```bash
# Stopper un array (ATTENTION : démonte d'abord !)
sudo umount /mnt/raid
sudo mdadm --stop /dev/md0

# Assembler manuellement avec la config
sudo mdadm --assemble --scan

# Assembler un array spécifique
sudo mdadm --assemble /dev/md0

# Assembler avec un fichier de config personnalisé
sudo mdadm --assemble --scan --config=/etc/mdadm/mdadm.conf.test
```

### Processus complet de configuration

Voici le workflow complet pour garantir l'assemblage au démarrage :

```bash
# 1. Créer l'array RAID
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# 2. Attendre la fin du resync (optionnel mais recommandé)
watch cat /proc/mdstat

# 3. Ajouter l'array à la configuration
echo "ARRAY /dev/md0 UUID=$(sudo mdadm --detail /dev/md0 | grep UUID | awk '{print $3}')" \
  | sudo tee -a /etc/mdadm/mdadm.conf

# 4. Mettre à jour l'initramfs
sudo update-initramfs -u    # Debian/Ubuntu
# OU
sudo dracut --force         # RHEL/CentOS/Fedora
# OU
sudo mkinitcpio -P          # Arch Linux

# 5. Activer le service de monitoring
sudo systemctl enable --now mdmonitor

# 6. Vérifier la configuration
sudo mdadm --detail --scan
```

### Dépannage de l'assemblage automatique

#### L'array n'est pas assemblé au démarrage

```bash
# 1. Vérifier la configuration
sudo cat /etc/mdadm/mdadm.conf

# 2. Vérifier que l'UUID correspond
sudo mdadm --detail /dev/md0 | grep UUID
grep UUID /etc/mdadm/mdadm.conf

# 3. Vérifier les logs de démarrage
sudo journalctl -b | grep mdadm

# 4. Examiner les métadonnées des disques
sudo mdadm --examine /dev/sdb /dev/sdc
```

#### L'array est assemblé en mode degraded

```bash
# Identifier le disque manquant
sudo mdadm --detail /dev/md0

# Vérifier les logs
sudo journalctl -u mdmonitor | tail -50

# Examiner /proc/mdstat
cat /proc/mdstat
```

> [!warning] Array non trouvé au boot Si l'array n'est pas trouvé, vérifiez que :
> 
> - L'UUID dans mdadm.conf est correct
> - L'initramfs a été mis à jour après modification de mdadm.conf
> - Les disques sont bien détectés par le BIOS/UEFI
> - Les modules kernel RAID sont chargés

### Configuration avancée : root sur RAID

Si votre système root (/) est sur RAID, des étapes supplémentaires sont nécessaires :

```bash
# 1. S'assurer que mdadm est dans l'initramfs
# (normalement automatique)

# 2. Configuration pour Debian/Ubuntu
# Éditer /etc/initramfs-tools/conf.d/mdadm
echo "BOOT_DEGRADED=true" | sudo tee /etc/initramfs-tools/conf.d/mdadm

# 3. Mettre à jour GRUB si nécessaire
sudo update-grub

# 4. Mettre à jour l'initramfs
sudo update-initramfs -u -k all
```

> [!tip] Démarrage en mode dégradé `BOOT_DEGRADED=true` permet au système de démarrer même si l'array RAID est en mode dégradé (un disque manquant). Utile pour la maintenance, mais à surveiller !

---

## 💿 6. Formatage et montage d'un array RAID

### Formatage de l'array

Une fois l'array créé et synchronisé, il faut le formater avec un système de fichiers.

#### Systèmes de fichiers recommandés

|Système|Usage recommandé|Avantages|
|---|---|---|
|`ext4`|Usage général, serveurs|Stable, journalisé, performant|
|`xfs`|Gros fichiers, haute performance|Excellent pour grandes partitions, parallélisation|
|`btrfs`|Snapshots, compression|Fonctionnalités avancées, snapshots natifs|
|`f2fs`|SSD RAID|Optimisé pour flash, bon pour SSD RAID|

#### Formatage en ext4

```bash
# Formatage basique
sudo mkfs.ext4 /dev/md0

# Avec label
sudo mkfs.ext4 -L "RAID_BACKUP" /dev/md0

# Avec options optimisées
sudo mkfs.ext4 -L "RAID_DATA" \
               -m 1 \
               -O extent,flex_bg \
               -E stride=32,stripe-width=64 \
               /dev/md0
```

> [!info] Options ext4 pour RAID
> 
> - `-m 1` : réserve 1% pour root (au lieu de 5%)
> - `stride` : taille du chunk en blocs de 4K (chunk 128K = 32 blocs)
> - `stripe-width` : stride × (nombre de disques de données)
>     - RAID 1 : pas utilisé
>     - RAID 5 (4 disques) : stride × 3 = 32 × 3 = 96
>     - RAID 10 (4 disques) : stride × 2 = 32 × 2 = 64

#### Formatage en XFS

```bash
# Formatage basique
sudo mkfs.xfs /dev/md0

# Avec label et optimisations RAID
sudo mkfs.xfs -L "RAID_DATA" \
              -d su=128k,sw=3 \
              /dev/md0
```

> [!info] Options XFS pour RAID
> 
> - `su` (stripe unit) : taille du chunk (128k pour chunk de 128 Ko)
> - `sw` (stripe width) : nombre de disques de données
>     - RAID 5 (4 disques) : 3 (4 - 1 parité)
>     - RAID 6 (6 disques) : 4 (6 - 2 parités)

#### Formatage en Btrfs

```bash
# Formatage basique
sudo mkfs.btrfs /dev/md0

# Avec label
sudo mkfs.btrfs -L "RAID_DATA" /dev/md0

# Pour RAID 1, Btrfs peut gérer sa propre redondance
# (mais généralement on laisse mdadm gérer le RAID)
sudo mkfs.btrfs -L "DATA" -m raid1 -d raid1 /dev/sdb /dev/sdc
```

> [!warning] Btrfs et mdadm Si vous utilisez mdadm pour le RAID, formatez l'array md en mode simple :
> 
> ```bash
> sudo mkfs.btrfs -m single -d single /dev/md0
> ```
> 
> Sinon, Btrfs va dupliquer la redondance déjà fournie par mdadm.

### Création du point de montage

```bash
# Créer le répertoire de montage
sudo mkdir -p /mnt/raid

# Ou pour un usage spécifique
sudo mkdir -p /mnt/backup
sudo mkdir -p /srv/data
```

### Montage manuel

```bash
# Montage simple
sudo mount /dev/md0 /mnt/raid

# Avec options spécifiques
sudo mount -o noatime,nodiratime /dev/md0 /mnt/raid

# Vérifier le montage
df -h /mnt/raid
mount | grep md0
```

> [!tip] Option noatime `noatime` désactive la mise à jour des timestamps d'accès, améliorant les performances. Utile pour la plupart des usages.

### Montage automatique avec /etc/fstab

Pour monter automatiquement l'array au démarrage, ajoutez une entrée dans `/etc/fstab`.

#### Par chemin de périphérique (déconseillé)

```bash
/dev/md0    /mnt/raid    ext4    defaults,noatime    0    2
```

#### Par UUID (RECOMMANDÉ)

```bash
# 1. Obtenir l'UUID du système de fichiers (pas du RAID !)
sudo blkid /dev/md0

# Sortie exemple :
# /dev/md0: UUID="a8f2c9d1-4e5b-6f7a-8c9d-0e1f2a3b4c5d" TYPE="ext4"

# 2. Ajouter à /etc/fstab
UUID=a8f2c9d1-4e5b-6f7a-8c9d-0e1f2a3b4c5d    /mnt/raid    ext4    defaults,noatime    0    2
```

#### Par LABEL

```bash
# 1. Créer un label (si pas déjà fait)
sudo e2label /dev/md0 "RAID_DATA"

# 2. Ajouter à /etc/fstab
LABEL=RAID_DATA    /mnt/raid    ext4    defaults,noatime    0    2
```

### Structure complète d'une ligne fstab

```
<périphérique>  <point_montage>  <type>  <options>           <dump>  <pass>
UUID=xxx...     /mnt/raid        ext4    defaults,noatime    0       2
```

|Champ|Description|Valeurs courantes|
|---|---|---|
|périphérique|UUID, LABEL, ou /dev/mdX|UUID recommandé|
|point_montage|Où monter|`/mnt/raid`, `/srv/data`|
|type|Système de fichiers|`ext4`, `xfs`, `btrfs`|
|options|Options de montage|`defaults`, `noatime`, `nodiratime`|
|dump|Sauvegarde avec dump|`0` (désactivé)|
|pass|Ordre de vérification fsck|`0` (pas de vérif), `2` (après root)|

### Options de montage courantes

```bash
# Options recommandées pour ext4 sur RAID
defaults,noatime,nodiratime,errors=remount-ro

# Options pour XFS sur RAID
defaults,noatime,nodiratime,logbufs=8,logbsize=256k

# Options pour Btrfs
defaults,noatime,compress=zstd,space_cache=v2
```

|Option|Description|Usage|
|---|---|---|
|`defaults`|Options par défaut (rw, suid, dev, exec, auto, nouser, async)|Toujours|
|`noatime`|Ne pas mettre à jour le timestamp d'accès|Performance|
|`nodiratime`|Ne pas mettre à jour le timestamp d'accès des répertoires|Performance|
|`errors=remount-ro`|Remonter en lecture seule en cas d'erreur|Sécurité ext4|
|`nofail`|Continue le boot même si le montage échoue|Arrays optionnels|
|`auto`|Monte automatiquement avec `mount -a`|Par défaut|
|`noauto`|Ne monte pas automatiquement|Montage manuel|

### Exemple complet : création et montage

Workflow complet de A à Z :

```bash
# 1. Créer l'array RAID 1
sudo mdadm --create /dev/md0 \
           --level=1 \
           --raid-devices=2 \
           /dev/sdb /dev/sdc

# 2. Attendre la fin du resync (optionnel)
watch -n 5 cat /proc/mdstat

# 3. Formater en ext4 avec optimisations
sudo mkfs.ext4 -L "BACKUP" -m 1 /dev/md0

# 4. Créer le point de montage
sudo mkdir -p /mnt/backup

# 5. Obtenir l'UUID du filesystem
UUID=$(sudo blkid -s UUID -o value /dev/md0)

# 6. Ajouter à /etc/fstab
echo "UUID=$UUID  /mnt/backup  ext4  defaults,noatime  0  2" | sudo tee -a /etc/fstab

# 7. Tester le montage
sudo mount -a

# 8. Vérifier
df -h /mnt/backup
mount | grep md0

# 9. Ajouter l'array à mdadm.conf
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

# 10. Mettre à jour l'initramfs
sudo update-initramfs -u  # Debian/Ubuntu

# 11. Activer le monitoring
sudo systemctl enable --now mdmonitor
```

### Test du montage automatique

```bash
# Vérifier la syntaxe de fstab
sudo mount -av

# Tester avec findmnt
findmnt --verify

# Simuler un redémarrage du montage
sudo umount /mnt/raid
sudo mount -a
df -h /mnt/raid
```

> [!warning] Erreurs fstab critiques Une erreur dans `/etc/fstab` peut empêcher le système de démarrer ! Testez toujours avec `mount -a` avant de redémarrer.

### Permissions et propriété

```bash
# Définir le propriétaire après montage
sudo chown -R user:user /mnt/raid

# Permissions pour usage partagé
sudo chmod 775 /mnt/raid
sudo chown root:users /mnt/raid

# Créer une structure de répertoires
sudo mkdir -p /mnt/raid/{data,backup,archive}
sudo chown user:user /mnt/raid/data
```

### Vérification post-configuration

```bash
# Vérifier l'état de l'array
sudo mdadm --detail /dev/md0

# Vérifier le montage
mount | grep md0
df -h /mnt/raid

# Vérifier la configuration persistante
cat /etc/mdadm/mdadm.conf | grep md0
cat /etc/fstab | grep raid

# Vérifier le service de monitoring
sudo systemctl status mdmonitor

# Test d'écriture
sudo touch /mnt/raid/test_file
ls -l /mnt/raid/
sudo rm /mnt/raid/test_file
```

---

## 🎯 Pièges courants et bonnes pratiques

### ⚠️ Pièges à éviter

> [!warning] Erreur 1 : Créer un RAID sans backup Ne créez **jamais** un array RAID sur des disques contenant des données importantes sans avoir fait un backup. La commande `--create` efface tout !

> [!warning] Erreur 2 : Oublier de mettre à jour mdadm.conf Après chaque création d'array, mettez à jour `/etc/mdadm/mdadm.conf` et l'initramfs, sinon l'array ne sera pas assemblé au boot.

> [!warning] Erreur 3 : Utiliser /dev/sdX dans fstab Les noms de périphériques (`/dev/sdb`) peuvent changer au redémarrage. Utilisez toujours UUID ou LABEL dans `/etc/fstab`.

> [!warning] Erreur 4 : Ignorer les alertes de monitoring Configurez toujours `MAILADDR` dans `mdadm.conf` et vérifiez régulièrement les emails d'alerte.

> [!warning] Erreur 5 : Ne pas tester le resync Après avoir créé un array, laissez le resync initial se terminer avant de mettre en production, surtout pour RAID 5/6.

### ✅ Bonnes pratiques

> [!tip] Pratique 1 : Utiliser des UUID partout
> 
> - UUID dans `mdadm.conf` pour identifier les arrays
> - UUID dans `fstab` pour identifier les filesystems
> - Plus fiable que les noms de périphériques

> [!tip] Pratique 2 : Documenter vos arrays
> 
> ```bash
> # Créer un fichier de documentation
> sudo mdadm --detail /dev/md0 | sudo tee /root/raid-md0-info.txt
> echo "Créé le $(date)" | sudo tee -a /root/raid-md0-info.txt
> echo "Disques: /dev/sdb, /dev/sdc" | sudo tee -a /root/raid-md0-info.txt
> ```

> [!tip] Pratique 3 : Surveillance proactive
> 
> ```bash
> # Script de vérification quotidien (à ajouter dans cron)
> #!/bin/bash
> mdadm --detail --scan | mail -s "RAID Status $(hostname)" admin@example.com
> ```

> [!tip] Pratique 4 : Tester la récupération Testez régulièrement la procédure de remplacement d'un disque défaillant en environnement de test.

> [!tip] Pratique 5 : Optimiser les paramètres
> 
> ```bash
> # Ajuster la vitesse de resync (en Ko/s)
> echo 50000 | sudo tee /proc/sys/dev/raid/speed_limit_min
> echo 200000 | sudo tee /proc/sys/dev/raid/speed_limit_max
> 
> # Rendre permanent dans /etc/sysctl.conf
> dev.raid.speed_limit_min = 50000
> dev.raid.speed_limit_max = 200000
> ```

### 🔍 Astuces avancées

> [!tip] Astuce 1 : Nommer les arrays de façon explicite
> 
> ```bash
> sudo mdadm --create /dev/md/backup \
>            --name=backup \
>            --level=1 \
>            --raid-devices=2 \
>            /dev/sdb /dev/sdc
> ```
> 
> Le nom apparaîtra dans `/dev/md/` et sera plus lisible.

> [!tip] Astuce 2 : Vérifier la santé périodiquement
> 
> ```bash
> # Lancer une vérification manuelle (sans corriger)
> echo check | sudo tee /sys/block/md0/md/sync_action
> 
> # Suivre la progression
> cat /sys/block/md0/md/sync_action
> cat /proc/mdstat
> 
> # Voir les erreurs détectées (devrait être 0)
> cat /sys/block/md0/md/mismatch_cnt
> ```

> [!tip] Astuce 3 : Automatiser les vérifications mensuelles
> 
> ```bash
> # Ajouter dans /etc/cron.monthly/raid-check
> #!/bin/bash
> echo check > /sys/block/md0/md/sync_action
> ```

> [!tip] Astuce 4 : Monitorer les performances
> 
> ```bash
> # Statistiques I/O en temps réel
> iostat -x 2 /dev/md0
> 
> # Avec iotop pour voir les processus
> sudo iotop -o
> ```

> [!tip] Astuce 5 : Bitmap pour accélérer le resync
> 
> ```bash
> # Ajouter un bitmap interne (accélère le resync après un crash)
> sudo mdadm --grow /dev/md0 --bitmap=internal
> 
> # Vérifier
> sudo mdadm --detail /dev/md0 | grep Bitmap
> 
> # Retirer le bitmap
> sudo mdadm --grow /dev/md0 --bitmap=none
> ```

### 📊 Tableau récapitulatif des commandes essentielles

|Action|Commande|
|---|---|
|Créer un array|`mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc`|
|Voir l'état|`cat /proc/mdstat`|
|Détails d'un array|`mdadm --detail /dev/md0`|
|Examiner un disque|`mdadm --examine /dev/sdb`|
|Mettre à jour la config|`mdadm --detail --scan >> /etc/mdadm/mdadm.conf`|
|Mettre à jour initramfs|`update-initramfs -u` (Debian) / `dracut --force` (RHEL)|
|Activer monitoring|`systemctl enable --now mdmonitor`|
|Formater|`mkfs.ext4 -L "LABEL" /dev/md0`|
|Obtenir UUID|`blkid /dev/md0`|
|Tester fstab|`mount -av`|
|Stopper un array|`mdadm --stop /dev/md0`|
|Assembler|`mdadm --assemble --scan`|

---

## 📝 Résumé

Vous savez maintenant :

✅ **Installer mdadm** sur différentes distributions Linux ✅ **Créer des arrays RAID** avec la commande `mdadm --create` et comprendre les différents paramètres ✅ **Surveiller l'état** des arrays avec `mdadm --detail`, `/proc/mdstat` et le monitoring en daemon ✅ **Configurer la persistance** via `/etc/mdadm/mdadm.conf` avec les directives ARRAY, MAILADDR, CREATE, etc. ✅ **Assurer l'assemblage automatique** au démarrage en mettant à jour l'initramfs et en activant le service systemd ✅ **Formater et monter** les arrays avec les systèmes de fichiers appropriés et configurer le montage automatique via `/etc/fstab`

Le RAID logiciel avec mdadm offre une solution flexible, performante et gratuite pour la redondance des données. En suivant les bonnes pratiques de configuration, de surveillance et de maintenance, vous assurez la disponibilité et l'intégrité de vos données.

> [!success] Point clé La configuration persistante (mdadm.conf + initramfs + fstab) est **cruciale** pour garantir que vos arrays RAID seront disponibles après chaque redémarrage. Ne négligez jamais cette étape !

---

_Cours rédigé pour Obsidian - Administration Linux - RAID logiciel avec mdadm_