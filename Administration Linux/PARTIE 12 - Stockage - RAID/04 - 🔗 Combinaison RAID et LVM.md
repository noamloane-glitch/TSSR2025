

## 📋 Table des matières

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

## Introduction

La combinaison de **RAID** (Redundant Array of Independent Disks) et **LVM** (Logical Volume Manager) représente une approche puissante pour la gestion du stockage sous Linux. Cette stratégie permet de bénéficier simultanément de la **redondance et des performances** offertes par RAID, tout en conservant la **flexibilité de gestion** apportée par LVM.

> [!info] Complémentarité RAID et LVM
> 
> - **RAID** : gère la redondance des données et les performances au niveau matériel/logiciel
> - **LVM** : gère la flexibilité des volumes (redimensionnement, snapshots, migration)
> - **Ensemble** : infrastructure de stockage robuste, performante ET flexible

---

## Pourquoi combiner RAID et LVM ?

### 🎯 Principe de séparation des responsabilités

Chaque technologie excelle dans un domaine spécifique :

|Aspect|RAID|LVM|
|---|---|---|
|**Redondance des données**|✅ Excellent|❌ Aucune|
|**Performance I/O**|✅ Optimisée|➖ Neutre|
|**Flexibilité des volumes**|❌ Rigide|✅ Excellente|
|**Snapshots**|❌ Non supporté|✅ Natif|
|**Redimensionnement à chaud**|❌ Difficile|✅ Simple|
|**Migration de données**|❌ Complexe|✅ Facilitée|

### 💡 Scénario typique

Imaginez que vous gérez un serveur de base de données :

1. **Sans combinaison** : vous devez choisir entre sécurité (RAID) OU flexibilité (LVM)
2. **Avec combinaison** : vous obtenez les deux en empilant les technologies

> [!example] Exemple concret
> 
> - Niveau physique : RAID 10 sur 4 disques → haute disponibilité + bonnes performances
> - Niveau logique : LVM sur le RAID → possibilité d'agrandir `/var/lib/mysql` sans interruption
> - Résultat : données protégées ET infrastructure flexible

---

## Architecture RAID + LVM

### 📐 Empilement des couches

```
┌─────────────────────────────────────┐
│   Système de fichiers (ext4, xfs)  │  ← Couche 4 : Filesystem
├─────────────────────────────────────┤
│   Volumes logiques LVM (LV)        │  ← Couche 3 : Logical Volumes
├─────────────────────────────────────┤
│   Groupe de volumes LVM (VG)       │  ← Couche 2 : Volume Group
├─────────────────────────────────────┤
│   Volume physique LVM (PV)         │  ← Couche 1b : Physical Volume
├─────────────────────────────────────┤
│   Array RAID (/dev/md0)            │  ← Couche 1a : RAID Array
├─────────────────────────────────────┤
│   Disques physiques                │  ← Couche 0 : Hardware
│   /dev/sda /dev/sdb /dev/sdc       │
└─────────────────────────────────────┘
```

### 🔄 Flux de création

1. Assemblage des disques physiques en array RAID
2. Transformation de l'array RAID en volume physique LVM
3. Création d'un groupe de volumes LVM
4. Découpage en volumes logiques
5. Formatage et montage

> [!warning] Ordre important L'ordre de création est crucial : RAID d'abord, puis LVM. Inverser cet ordre (LVM sur disques individuels, puis RAID) entraînerait une perte totale de redondance au niveau LVM.

---

## Création de volumes physiques LVM sur un array RAID

### 🛠️ Procédure complète étape par étape

#### Étape 1 : Créer l'array RAID

```bash
# Exemple : RAID 5 avec 3 disques + 1 spare
mdadm --create /dev/md0 \
  --level=5 \
  --raid-devices=3 \
  --spare-devices=1 \
  /dev/sda /dev/sdb /dev/sdc /dev/sdd

# Surveiller la construction
watch cat /proc/mdstat
```

> [!tip] Choix du niveau RAID
> 
> - **RAID 1** : pour données critiques (mirroring)
> - **RAID 5** : bon compromis stockage/redondance
> - **RAID 6** : tolérance à 2 pannes
> - **RAID 10** : performances maximales + redondance

#### Étape 2 : Initialiser le volume physique LVM

```bash
# Créer un PV directement sur l'array RAID
pvcreate /dev/md0

# Vérifier la création
pvdisplay /dev/md0
pvs  # Vue synthétique
```

**Sortie attendue :**

```
  "/dev/md0" is a new physical volume of "2.73 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/md0
  VG Name               
  PV Size               2.73 TiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
```

#### Étape 3 : Créer un groupe de volumes

```bash
# Créer un VG nommé "vg_data" sur le PV
vgcreate vg_data /dev/md0

# Vérifier
vgdisplay vg_data
vgs
```

> [!info] Naming convention Préfixez vos VG par leur fonction : `vg_data`, `vg_backup`, `vg_database` pour faciliter la gestion.

#### Étape 4 : Créer des volumes logiques

```bash
# Créer un LV de 500 Go pour PostgreSQL
lvcreate -L 500G -n lv_postgresql vg_data

# Créer un LV utilisant 50% de l'espace restant
lvcreate -l 50%FREE -n lv_backup vg_data

# Vérifier
lvdisplay /dev/vg_data/lv_postgresql
lvs
```

#### Étape 5 : Formater et monter

```bash
# Formater en XFS (recommandé pour bases de données)
mkfs.xfs /dev/vg_data/lv_postgresql

# Créer le point de montage
mkdir -p /var/lib/postgresql

# Monter
mount /dev/vg_data/lv_postgresql /var/lib/postgresql

# Rendre permanent dans /etc/fstab
echo "/dev/vg_data/lv_postgresql /var/lib/postgresql xfs defaults 0 2" >> /etc/fstab
```

### 📊 Vérification complète de la pile

```bash
# Vérifier chaque couche
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE

# Exemple de sortie :
# NAME                    SIZE TYPE  MOUNTPOINT      FSTYPE
# sda                     1.8T disk                  
# └─md0                   2.7T raid5                 LVM2_member
#   └─vg_data-lv_postgresql 500G lvm   /var/lib/postgresql xfs
```

---

## Avantages de la combinaison RAID + LVM

### ✅ Avantages hérités du RAID

#### 1. **Redondance des données**

```bash
# Simulation de panne d'un disque
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb

# Les données restent accessibles !
ls /var/lib/postgresql  # Fonctionne toujours

# Remplacement à chaud
mdadm /dev/md0 --add /dev/sde
# Reconstruction automatique
```

> [!info] Protection transparente LVM ne voit qu'un seul device (`/dev/md0`). La panne d'un disque est gérée par RAID de façon invisible pour LVM.

#### 2. **Performances optimisées**

- **RAID 0/10** : lectures et écritures parallélisées
- **RAID 5/6** : lectures distribuées sur plusieurs disques
- LVM bénéficie de ces performances sans configuration supplémentaire

### ✅ Avantages hérités de LVM

#### 1. **Redimensionnement flexible**

```bash
# Agrandir un volume logique à chaud
lvextend -L +200G /dev/vg_data/lv_postgresql
xfs_growfs /var/lib/postgresql  # Étendre le filesystem

# Réduire (nécessite démontage pour ext4, impossible pour xfs)
umount /mnt/olddata
e2fsck -f /dev/vg_data/lv_olddata
resize2fs /dev/vg_data/lv_olddata 300G
lvreduce -L 300G /dev/vg_data/lv_olddata
```

#### 2. **Snapshots pour sauvegardes cohérentes**

```bash
# Créer un snapshot avant maintenance
lvcreate -L 50G -s -n snap_postgresql_backup /dev/vg_data/lv_postgresql

# Monter le snapshot en lecture seule
mkdir /mnt/snap_backup
mount -o ro /dev/vg_data/snap_postgresql_backup /mnt/snap_backup

# Sauvegarder depuis le snapshot (données cohérentes)
rsync -avz /mnt/snap_backup/ /backup/postgresql/

# Supprimer le snapshot
umount /mnt/snap_backup
lvremove /dev/vg_data/snap_postgresql_backup
```

> [!tip] Snapshots et cohérence Les snapshots LVM permettent de sauvegarder une base de données en cours d'exécution avec une cohérence temporelle, sans arrêt de service.

#### 3. **Migration de données simplifiée**

```bash
# Ajouter un nouveau disque RAID au VG existant
mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdf /dev/sdg
pvcreate /dev/md1
vgextend vg_data /dev/md1

# Migrer un LV vers le nouveau RAID
pvmove -n lv_postgresql /dev/md0 /dev/md1

# Retirer l'ancien RAID
vgreduce vg_data /dev/md0
```

### 🎯 Avantages combinés uniques

|Scénario|RAID seul|LVM seul|RAID + LVM|
|---|---|---|---|
|Panne disque|✅ Données protégées|❌ Perte totale|✅ Données protégées|
|Agrandir partition|❌ Très difficile|✅ Simple|✅ Simple|
|Snapshot pour backup|❌ Impossible|✅ Possible|✅ Possible|
|Performances I/O|✅ Optimisées|➖ Standard|✅ Optimisées|
|Migration hardware|❌ Complexe|⚠️ Risqué|✅ Sécurisé + flexible|

---

## Cas d'usage pratiques

### 🗄️ Cas 1 : Serveur de base de données

**Besoin :**

- Haute disponibilité (pas de perte de données)
- Performances élevées (IOPS)
- Capacité évolutive (croissance imprévisible)

**Solution :**

```bash
# 1. RAID 10 sur 4 disques SSD (performances + redondance)
mdadm --create /dev/md0 --level=10 --raid-devices=4 \
  /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1

# 2. Stack LVM
pvcreate /dev/md0
vgcreate vg_database /dev/md0
lvcreate -L 800G -n lv_mysql vg_database

# 3. Optimisations filesystem
mkfs.xfs -d su=64k,sw=2 /dev/vg_database/lv_mysql  # Alignement RAID
mount -o noatime,nodiratime /dev/vg_database/lv_mysql /var/lib/mysql
```

> [!example] Avantages obtenus
> 
> - RAID 10 : tolérance à 2 pannes + excellentes performances lecture/écriture
> - LVM : ajout facile de 200 Go quand la base croît
> - Snapshots : sauvegardes cohérentes sans arrêt MySQL

### 📦 Cas 2 : Serveur de fichiers / NAS

**Besoin :**

- Grande capacité de stockage
- Protection contre pannes disques
- Gestion flexible de quotas par service

**Solution :**

```bash
# 1. RAID 6 sur 8 disques (tolérance 2 pannes, bon ratio espace)
mdadm --create /dev/md0 --level=6 --raid-devices=8 \
  /dev/sd{a,b,c,d,e,f,g,h}

# 2. LVM avec plusieurs volumes logiques
pvcreate /dev/md0
vgcreate vg_storage /dev/md0
lvcreate -L 2T -n lv_documents vg_storage
lvcreate -L 5T -n lv_media vg_storage
lvcreate -L 1T -n lv_backups vg_storage

# 3. Filesystems adaptés
mkfs.ext4 /dev/vg_storage/lv_documents
mkfs.xfs /dev/vg_storage/lv_media
mkfs.ext4 /dev/vg_storage/lv_backups
```

> [!tip] Gestion par service Chaque service (documents, média, backups) a son propre LV, permettant des snapshots, quotas et redimensionnements indépendants.

### 🖥️ Cas 3 : Infrastructure virtualisée

**Besoin :**

- Stockage des images VM
- Snapshots rapides pour tests
- Ajout dynamique de capacité

**Solution :**

```bash
# 1. RAID 5 sur 4 disques (compromis espace/performance)
mdadm --create /dev/md0 --level=5 --raid-devices=4 \
  /dev/sda /dev/sdb /dev/sdc /dev/sdd

# 2. Thin provisioning LVM pour VM
pvcreate /dev/md0
vgcreate vg_vms /dev/md0
lvcreate -L 2T --thinpool thin_vms vg_vms

# 3. Créer des volumes fins pour chaque VM
lvcreate -V 100G --thin -n vm_web01 vg_vms/thin_vms
lvcreate -V 200G --thin -n vm_db01 vg_vms/thin_vms
```

> [!info] Thin provisioning Allocation d'espace à la demande : vous pouvez allouer 500 Go à vos VMs alors que vous n'avez que 300 Go physiquement. L'espace réel n'est consommé qu'au fur et à mesure de l'utilisation.

### 💾 Cas 4 : Serveur de sauvegarde

**Besoin :**

- Énorme capacité
- Redondance critique
- Snapshots pour versioning

**Solution :**

```bash
# 1. RAID 6 sur 12 disques de grande capacité
mdadm --create /dev/md0 --level=6 --raid-devices=12 \
  /dev/sd{a,b,c,d,e,f,g,h,i,j,k,l}

# 2. LVM avec snapshots automatisés
pvcreate /dev/md0
vgcreate vg_backup /dev/md0
lvcreate -L 20T -n lv_backups vg_backup

# 3. Script de snapshot quotidien
cat > /usr/local/bin/daily_snapshot.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d)
lvcreate -L 500G -s -n snap_backup_$DATE /dev/vg_backup/lv_backups
# Supprimer les snapshots de plus de 7 jours
lvs --noheadings -o lv_name vg_backup | grep "snap_backup" | \
  while read snap; do
    AGE=$(echo $snap | sed 's/snap_backup_//')
    if [[ $AGE < $(date -d '7 days ago' +%Y%m%d) ]]; then
      lvremove -f /dev/vg_backup/$snap
    fi
  done
EOF
chmod +x /usr/local/bin/daily_snapshot.sh
```

---

## Bonnes pratiques

### 🎯 Conception de l'architecture

> [!tip] Règle du bon ordonnancement **Toujours** créer le RAID avant LVM. Le RAID gère le matériel, LVM gère la logique.

#### 1. **Alignement des tailles**

```bash
# Vérifier l'alignement du RAID
mdadm --detail /dev/md0 | grep "Chunk Size"
# Output: Chunk Size : 512K

# Créer le filesystem avec alignement correspondant
mkfs.xfs -d su=512k,sw=3 /dev/vg_data/lv_app
#        su = chunk size du RAID
#        sw = nombre de data disks (RAID5 avec 4 disques = 3 data + 1 parity)
```

> [!info] Pourquoi l'alignement ? Un mauvais alignement peut dégrader les performances jusqu'à 30-40%. L'alignement garantit que les opérations I/O LVM correspondent aux boundaries du RAID.

#### 2. **Réservation d'espace dans le VG**

```bash
# Ne jamais allouer 100% du VG
vgdisplay vg_data | grep "VG Size"
# VG Size: 2.73 TiB

# Allouer maximum 90-95%
lvcreate -L 2.4T -n lv_data vg_data  # Garde ~300G libres

# Espace libre pour :
# - Snapshots temporaires
# - Extension d'urgence
# - Opérations de maintenance (pvmove)
```

#### 3. **Monitoring proactif**

```bash
# Script de surveillance combinée RAID + LVM
cat > /usr/local/bin/storage_health.sh << 'EOF'
#!/bin/bash
echo "=== RAID Status ==="
cat /proc/mdstat
mdadm --detail /dev/md0 | grep -E "State|Failed|Spare"

echo -e "\n=== LVM Status ==="
vgs
lvs -a -o +devices

echo -e "\n=== Disk Usage ==="
df -h | grep vg_
EOF

# Automatiser avec cron (quotidien)
echo "0 8 * * * /usr/local/bin/storage_health.sh | mail -s 'Storage Report' admin@example.com" | crontab -
```

### 🔒 Sécurité et fiabilité

#### 1. **Sauvegarder les métadonnées**

```bash
# Sauvegarder la configuration RAID
mdadm --detail --scan > /etc/mdadm/mdadm.conf

# Sauvegarder la configuration LVM
vgcfgbackup vg_data
# Stocké dans /etc/lvm/backup/vg_data

# Restauration en cas de corruption :
vgcfgrestore -f /etc/lvm/backup/vg_data vg_data
```

#### 2. **Tests de restauration**

```bash
# Simuler une panne régulièrement (environnement de test)
mdadm /dev/md0 --fail /dev/sdb
# Vérifier que tout fonctionne
# Remplacer et vérifier la reconstruction
mdadm /dev/md0 --add /dev/sde
watch cat /proc/mdstat
```

### 📈 Performance

#### 1. **Monitoring des I/O**

```bash
# Identifier les goulots d'étranglement
iostat -x 2 10  # Rafraîchissement toutes les 2s, 10 fois

# Surveiller spécifiquement le RAID
iostat -x /dev/md0 2

# Métriques importantes :
# - %util : taux d'utilisation (>80% = saturation)
# - await : latence moyenne (>20ms = problème)
# - r/s, w/s : IOPS lecture/écriture
```

#### 2. **Optimisation read-ahead**

```bash
# Augmenter le read-ahead pour workloads séquentiels
blockdev --setra 8192 /dev/md0  # 4 Mo de read-ahead

# Vérifier
blockdev --getra /dev/md0
```

---

## Pièges courants à éviter

### ⚠️ Piège 1 : LVM sur disques individuels + RAID logiciel

**❌ Mauvaise approche :**

```bash
# ERREUR : LVM d'abord, RAID ensuite
pvcreate /dev/sda /dev/sdb /dev/sdc
vgcreate vg_data /dev/sda /dev/sdb /dev/sdc
# Puis tentative de RAID → IMPOSSIBLE ou perte de redondance
```

**✅ Bonne approche :**

```bash
# RAID d'abord
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
# LVM ensuite
pvcreate /dev/md0
vgcreate vg_data /dev/md0
```

> [!warning] Conséquence Si vous créez LVM sur disques individuels, vous perdez toute redondance. Une panne de disque = perte du VG entier.

### ⚠️ Piège 2 : Oublier de synchroniser avant maintenance

```bash
# ❌ Maintenance sans vérification
mdadm /dev/md0 --fail /dev/sdb  # Panne simulée
mdadm /dev/md0 --remove /dev/sdb
# ... attendre la reconstruction ? NON !

# ✅ Vérifier l'état avant toute opération
cat /proc/mdstat
# Attendre que la reconstruction soit à 100%
# [====>................]  recovery = 25.3% (...)
# Seulement ensuite, procéder à la maintenance
```

### ⚠️ Piège 3 : Snapshots trop petits

```bash
# ❌ Snapshot insuffisant
lvcreate -L 10G -s -n snap_db /dev/vg_data/lv_database
# Si plus de 10 Go de modifications, le snapshot devient invalide !

# ✅ Dimensionner selon l'activité
# Règle : 20-30% de la taille du LV pour snapshot temporaire
lvcreate -L 200G -s -n snap_db /dev/vg_data/lv_database  # LV de 1To
```

> [!warning] Snapshot invalidé Un snapshot plein devient automatiquement invalide et inutilisable. Surveillez avec `lvs -a` la colonne "Data%".

### ⚠️ Piège 4 : Négliger l'espace libre

```bash
# ❌ Allouer 100% du VG
vgs vg_data  # VG Size: 2 TiB
lvcreate -L 2T -n lv_full vg_data
# Plus d'espace pour snapshots, extensions, ou pvmove !

# ✅ Garder 10-15% libre
lvcreate -L 1.7T -n lv_data vg_data  # Garde 300 Go libres
```

### ⚠️ Piège 5 : RAID hardware et LVM incompatibles

> [!warning] RAID hardware Si vous utilisez un contrôleur RAID hardware, il présente un seul disque virtuel (`/dev/sda`) au système. Vous pouvez directement créer LVM dessus sans mdadm. Ne tentez pas de créer un RAID logiciel (mdadm) par-dessus un RAID hardware.

```bash
# Avec RAID hardware
lsblk
# sda  (volume RAID présenté par le contrôleur)

# Directement LVM
pvcreate /dev/sda
vgcreate vg_data /dev/sda
```

---

## Astuces avancées

### 💡 Astuce 1 : Cache SSD pour accélérer un RAID HDD

```bash
# RAID 5 sur disques HDD (capacité)
mdadm --create /dev/md0 --level=5 --raid-devices=4 \
  /dev/sda /dev/sdb /dev/sdc /dev/sdd

# LVM avec cache SSD
pvcreate /dev/md0
pvcreate /dev/nvme0n1  # SSD NVMe comme cache
vgcreate vg_data /dev/md0 /dev/nvme0n1

# Créer un LV avec cache
lvcreate -L 2T -n lv_data vg_data /dev/md0
lvcreate -L 100G -n lv_data_cache vg_data /dev/nvme0n1
lvconvert --type cache --cachepool lv_data_cache vg_data/lv_data
```

> [!tip] Performances hybrides Le cache SSD accélère les lectures fréquentes (hot data) tout en conservant la capacité du RAID HDD. Amélioration typique : 2-5x sur les lectures.

### 💡 Astuce 2 : Migration RAID à chaud

```bash
# Passer de RAID 5 à RAID 6 sans interruption
mdadm --grow /dev/md0 --level=6 --raid-devices=4 --backup-file=/root/raid_backup

# Surveiller la conversion
watch cat /proc/mdstat

# LVM fonctionne normalement pendant toute l'opération
```

### 💡 Astuce 3 : Snapshots en cascade pour tests

```bash
# Production
lvcreate -L 1T -n lv_prod vg_data

# Snapshot pour développement
lvcreate -L 100G -s -n snap_dev /dev/vg_data/lv_prod

# Snapshot du snapshot pour tests destructifs
lvcreate -L 50G -s -n snap_test /dev/vg_data/snap_dev

# Chaque environnement est isolé
mount /dev/vg_data/lv_prod /prod
mount /dev/vg_data/snap_dev /dev
mount /dev/vg_data/snap_test /test
```

### 💡 Astuce 4 : Monitoring automatisé avec mdadm

```bash
# Configurer les alertes email
echo "MAILADDR admin@example.com" >> /etc/mdadm/mdadm.conf

# Activer le daemon de monitoring
systemctl enable mdmonitor
systemctl start mdmonitor

# mdadm enverra automatiquement un email en cas de :
# - Disque défaillant
# - Reconstruction démarrée/terminée
# - Degraded array
```

### 💡 Astuce 5 : Automatisation de l'extension

```bash
# Script pour étendre automatiquement si utilisation > 85%
cat > /usr/local/bin/auto_extend_lv.sh << 'EOF'
#!/bin/bash
THRESHOLD=85
LV="/dev/vg_data/lv_application"
MOUNT="/app"

USAGE=$(df -h $MOUNT | awk 'NR==2 {print $5}' | sed 's/%//')

if [ $USAGE -gt $THRESHOLD ]; then
    echo "Usage at ${USAGE}%, extending LV by 100G"
    lvextend -L +100G $LV
    xfs_growfs $MOUNT  # ou resize2fs pour ext4
    echo "Extension completed" | mail -s "LV Auto-extended" admin@example.com
fi
EOF

# Cron quotidien
echo "0 2 * * * /usr/local/bin/auto_extend_lv.sh" | crontab -
```

---

## 🎓 Synthèse

La combinaison RAID + LVM offre une infrastructure de stockage professionnelle qui allie :

- ✅ **Sécurité** : redondance des données via RAID
- ✅ **Performance** : optimisations I/O du RAID
- ✅ **Flexibilité** : redimensionnement et snapshots LVM
- ✅ **Évolutivité** : ajout de capacité sans interruption
- ✅ **Maintenabilité** : opérations de maintenance simplifiées

**Règle d'or** : RAID gère le matériel, LVM gère la logique. Cette séparation des responsabilités crée une solution robuste et pérenne.

---