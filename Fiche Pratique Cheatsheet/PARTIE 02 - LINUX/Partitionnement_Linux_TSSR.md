# Partitionnement Linux

> Découper un disque en partitions logiques, les formater et les monter.  
> Outils principaux : `fdisk` (CLI), `cfdisk` (semi-graphique), `parted` (GPT/MBR).

---

## Concepts clés

### Nomenclature des disques

| Type | Format | Exemple |
|------|--------|---------|
| SATA / USB | `/dev/sdX` | `/dev/sda`, `/dev/sdb` |
| NVMe | `/dev/nvmeXnYpZ` | `/dev/nvme0n1p1` |
| IDE (ancien) | `/dev/hdX` | `/dev/hda` |

> ⚠️ Les noms `/dev/sdX` peuvent **changer au redémarrage** → toujours utiliser l'UUID dans `/etc/fstab`

### Systèmes de partitionnement

| Système | Partitions max | Taille max | Usage |
|---------|---------------|------------|-------|
| **MBR** | 4 primaires (ou 3+1 étendue + logiques) | 2 Tio | Ancien, BIOS |
| **GPT** | 128 | 8 Zio | ✅ Moderne, UEFI |

### Types de systèmes de fichiers

| FS | Usage | Commande |
|----|-------|---------|
| `ext4` | ✅ Linux standard | `mkfs.ext4` |
| `xfs` | Hautes performances | `mkfs.xfs` |
| `swap` | Mémoire virtuelle | `mkswap` |
| `vfat` | FAT32, compatible tout OS | `mkfs.vfat` |
| `ntfs` | Windows | `mkfs.ntfs` |

---

## Procédure avec `fdisk` (MBR — CLI interactif)

> Outil classique, **MBR uniquement** (par défaut), interactif.

### Commandes internes fdisk

| Touche | Action |
|--------|--------|
| `m` | Aide |
| `p` | Afficher les partitions |
| `n` | Nouvelle partition |
| `d` | Supprimer une partition |
| `t` | Changer le type |
| `w` | ✅ Écrire et quitter |
| `q` | Quitter sans sauvegarder |

### Procédure complète

```bash
# 1. Identifier les disques disponibles
sudo fdisk -l
lsblk

# 2. Ouvrir fdisk sur le disque cible
sudo fdisk /dev/sdb

# --- Dans fdisk ---
# 3. Créer une nouvelle partition
n          # nouvelle partition
p          # primaire (ou e pour étendue)
1          # numéro de partition
           # Entrée = début par défaut
+6G        # taille (ex: +6G, +2G, ou Entrée = reste du disque)

# 4. Changer le type si besoin (ex: swap)
t          # changer type
2          # numéro de partition
82         # code hex : 82 = swap, 83 = Linux, 8e = LVM

# 5. Vérifier
p          # afficher le résultat

# 6. Écrire et quitter
w
```

---

## Procédure avec `cfdisk` (MBR/GPT — semi-graphique)

> Interface ncurses, plus intuitive. Navigation au clavier.

```bash
sudo cfdisk /dev/sdb
```

Navigation : flèches ← → pour les actions (`New`, `Delete`, `Type`, `Write`, `Quit`)

> ✅ Recommandé pour les débutants — même résultat que `fdisk`

---

## Procédure avec `parted` (MBR/GPT — scriptable)

> Outil puissant, supporte **GPT et MBR**, utilisable en script.

```bash
# Mode interactif
sudo parted /dev/sdb

# --- Dans parted ---
print                          # afficher les partitions
mklabel gpt                    # créer table GPT (ou msdos pour MBR)
mkpart primary ext4 0% 6GB     # partition 1 : 6 Go ext4
mkpart primary ext4 6GB 8GB    # partition 2 : 2 Go ext4
mkpart primary linux-swap 8GB 100%  # partition 3 : swap reste
quit

# Mode non-interactif (scriptable)
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary ext4 0% 6GB
sudo parted -s /dev/sdb mkpart primary ext4 6GB 8GB
sudo parted -s /dev/sdb mkpart primary linux-swap 8GB 100%
```

---

## Formatage des partitions

```bash
# Formater en ext4 (avec label)
sudo mkfs.ext4 -L "DATA" /dev/sdb1
sudo mkfs.ext4 -L "PERSO" /dev/sdb2

# Initialiser le swap
sudo mkswap -L "SWAP" /dev/sdb3

# Activer le swap
sudo swapon /dev/sdb3

# Désactiver un swap existant si nécessaire
sudo swapoff /dev/sdaX

# Vérifier les UUID après formatage
sudo blkid /dev/sdb1
lsblk -f
```

---

## Montage manuel (temporaire)

```bash
# Créer les points de montage
sudo mkdir -p /mnt/data
sudo mkdir -p /home/wilder/Documents/personnel

# Monter manuellement
sudo mount /dev/sdb1 /mnt/data
sudo mount /dev/sdb2 /home/wilder/Documents/personnel

# Vérifier
df -h
mount | grep sdb
```

---

## Montage automatique — `/etc/fstab`

```bash
# Récupérer les UUID
sudo blkid

# Éditer fstab
sudo nano /etc/fstab
```

### Structure d'une ligne fstab

```
UUID=xxxx-xxxx   /mnt/data   ext4   defaults   0   2
```

| Champ | Valeur | Signification |
|-------|--------|---------------|
| Device | `UUID=...` | ✅ Toujours UUID (stable) |
| Point de montage | `/mnt/data` | Répertoire existant |
| Type FS | `ext4`, `swap` | Système de fichiers |
| Options | `defaults` | Options standard |
| Dump | `0` | Pas de sauvegarde automatique |
| Pass | `0`=non, `1`=racine, `2`=autres | Ordre vérification fsck |

### Exemple fstab complet

```bash
# Partition DATA
UUID=a1b2c3d4-...   /mnt/data                       ext4   defaults   0   2

# Partition PERSO
UUID=e5f6a7b8-...   /home/wilder/Documents/personnel ext4   defaults   0   2

# Partition SWAP
UUID=c9d0e1f2-...   none                             swap   sw         0   0
```

### Tester et appliquer

```bash
# ⚠️ Toujours tester avant de redémarrer
sudo mount -a           # monter tout ce qui est dans fstab
df -h                   # vérifier les montages
sudo swapon --show      # vérifier le swap
```

---

## Commandes de diagnostic

| Commande | Rôle |
|----------|------|
| `lsblk` | Lister disques et partitions |
| `lsblk -f` | Avec FS, labels et UUID |
| `sudo fdisk -l` | Détails complets des disques |
| `sudo blkid` | UUID et types de toutes les partitions |
| `df -h` | Espace disque des partitions montées |
| `sudo swapon --show` | État du swap actif |
| `free -h` | Mémoire RAM + swap |
| `sudo fsck /dev/sdb1` | Vérifier/réparer un FS |

---

## ⚠️ À retenir absolument

- Toujours `lsblk` / `fdisk -l` **avant** toute opération — vérifier le bon disque
- `fdisk` = MBR par défaut | `parted` = MBR + GPT
- `cfdisk` = même chose que `fdisk` mais plus visuel
- Type swap dans fdisk = code `82`, dans parted = `linux-swap`
- UUID dans `/etc/fstab` — jamais `/dev/sdX` (instable)
- `sudo mount -a` pour tester fstab **avant de redémarrer**
- Une erreur dans `/etc/fstab` peut **empêcher le démarrage** du système
- Pass fstab : `1` pour `/` (racine), `2` pour les autres, `0` pour swap
