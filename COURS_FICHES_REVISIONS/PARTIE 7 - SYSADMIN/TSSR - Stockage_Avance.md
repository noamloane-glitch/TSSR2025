## ⚡ L'essentiel en 5 minutes - Stockage Avancé (RAID / LVM)

### 📌 C'est quoi en 2 lignes ?
Technologies de virtualisation du stockage permettant d'agréger plusieurs disques physiques pour améliorer la **capacité**, les **performances** et la **fiabilité** du stockage, tout en offrant une gestion flexible des partitions.

---

### 💡 Concepts clés à retenir :

* **RAID** : Redundant Array of Independent Disks - Agrégation de disques pour performance/fiabilité/capacité
* **LVM** : Logical Volume Manager - Couche d'abstraction entre disques physiques et partitions logiques
* **Striping** : Répartition des données sur plusieurs disques (performance)
* **Mirroring** : Duplication des données sur plusieurs disques (fiabilité)
* **Physical Extent (PE)** : Unité d'allocation dans les Physical Volumes (LVM)
* **Logical Extent (LE)** : Unité d'allocation dans les Logical Volumes (LVM)
* **Snapshot (COW)** : Copie instantanée à l'écriture d'un volume logique
* **Hot-plug** : Remplacement de disque à chaud sans arrêt du système

---

### 💻 Commandes essentielles :

```bash
# 🐧 RAID logiciel Linux (mdadm)
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1  # Créer RAID 5
mdadm --detail /dev/md0                    # Afficher infos RAID
mdadm --manage /dev/md0 --add /dev/sdd1    # Ajouter disque spare
cat /proc/mdstat                           # État des RAID
```

```bash
# 🐧 LVM - Physical Volumes
pvcreate /dev/sdb1                         # Créer PV
pvdisplay                                  # Afficher PV
pvs                                        # Liste courte des PV
```

```bash
# 🐧 LVM - Volume Groups
vgcreate vg_data /dev/sdb1 /dev/sdc1       # Créer VG avec 2 PV
vgextend vg_data /dev/sdd1                 # Ajouter PV au VG
vgdisplay                                  # Afficher VG
vgs                                        # Liste courte des VG
```

```bash
# 🐧 LVM - Logical Volumes
lvcreate -L 10G -n lv_app vg_data          # Créer LV de 10 Go
lvcreate -l 100%FREE -n lv_data vg_data    # Utiliser tout l'espace libre
lvextend -L +5G /dev/vg_data/lv_app        # Agrandir de 5 Go
lvreduce -L -2G /dev/vg_data/lv_app        # Réduire de 2 Go (DANGEREUX)
lvdisplay                                  # Afficher LV
lvs                                        # Liste courte des LV
```

```bash
# 🐧 LVM - Snapshots
lvcreate -L 1G -s -n snap_app /dev/vg_data/lv_app    # Créer snapshot
lvconvert --merge /dev/vg_data/snap_app              # Restaurer snapshot
```

```bash
# 🐧 Redimensionnement avec système de fichiers
resize2fs /dev/vg_data/lv_app              # ext4 : adapter FS après lvextend
xfs_growfs /mnt/data                       # XFS : agrandir FS (monté)
```

---

### 📐 Calculs RAID :

* **RAID 0 (Striping)** : Capacité totale = n × capacité du plus petit disque | Tolérance = 0 panne
* **RAID 1 (Mirroring)** : Capacité totale = capacité du plus petit disque | Tolérance = n-1 pannes
* **RAID 5** : Capacité totale = (n-1) × capacité du plus petit disque | Tolérance = 1 panne
* **RAID 6** : Capacité totale = (n-2) × capacité du plus petit disque | Tolérance = 2 pannes
* **RAID 10** : Capacité totale = (n/2) × capacité du plus petit disque | Tolérance = 1 panne par miroir

**Exemple RAID 5 :**
```
Données : 3 disques de 1 To en RAID 5
Calcul : (3-1) × 1 To = 2 To
Résultat : 2 To utilisables, 1 To de parité
```

**Exemple LVM :**
```
Données : VG de 100 Go, créer LV de 30 Go
Commande : lvcreate -L 30G -n lv_web vg_data
Résultat : LV de 30 Go, reste 70 Go disponibles dans VG
```

---

### ⚠️ Pièges à éviter :

* ❌ **RAID ≠ Sauvegarde** : Le RAID protège contre les pannes matérielles, pas contre les erreurs humaines ou corruptions
* ❌ **Ne jamais réduire un LV sans réduire le FS d'abord** : Risque de perte de données totale
* ❌ **XFS ne supporte PAS la réduction** : Impossible de réduire un LV avec XFS (uniquement extension)
* ❌ **RAID 0 n'offre AUCUNE fiabilité** : Une panne = perte totale des données
* ❌ **Mélanger disques de tailles différentes dans RAID** : Capacité limitée au plus petit disque (gaspillage)
* ❌ **Oublier de sauvegarder avant manipulation LVM** : Les opérations de redimensionnement sont risquées
* ❌ **Ne pas vérifier l'état du RAID régulièrement** : Un disque peut être défaillant sans que vous le sachiez

---

### ✅ Bonnes pratiques :

* ✅ **Utiliser des disques identiques dans RAID** : Même modèle, même capacité, même performance
* ✅ **Toujours tester la reconstruction RAID** : Ajouter un disque spare pour reconstruction automatique
* ✅ **Partitionner les disques avec label LVM** : Évite que les outils considèrent le disque comme vide
* ✅ **Laisser 10-15% d'espace libre dans VG** : Facilite les opérations et permet les snapshots
* ✅ **Faire un snapshot avant manipulation** : Permet un rollback rapide en cas d'erreur
* ✅ **Surveiller /proc/mdstat régulièrement** : Détection précoce des pannes RAID
* ✅ **Documenter la structure LVM** : Noter VG, PV, LV et leurs usages
* ✅ **Cloisonner les espaces critiques** : Séparer /var/log, /tmp, /home sur des LV distincts

---

### 📚 Vocabulaire technique :

| Terme | Définition courte |
|-------|------------------|
| **JBOD** | Just a Bunch Of Disks - Concaténation simple sans redondance |
| **Striping** | Répartition des données par bandes sur plusieurs disques |
| **Parity** | Information de parité (XOR) permettant reconstruction en cas de panne |
| **PV (Physical Volume)** | Disque ou partition physique intégré dans LVM |
| **VG (Volume Group)** | Groupe de PV formant un pool de stockage |
| **LV (Logical Volume)** | Partition logique découpée dans un VG |
| **PE/LE** | Physical/Logical Extent - Blocs d'allocation (4 Mo par défaut) |
| **COW (Copy-On-Write)** | Mécanisme de copie différée utilisé par les snapshots |
| **Hot-plug** | Capacité à remplacer composant sans arrêt du système |
| **Array** | Grappe/cluster RAID formé de plusieurs disques |
| **Spare disk** | Disque de réserve pour reconstruction automatique |
| **NAS** | Network Attached Storage - Serveur de fichiers en réseau |
| **SAN** | Storage Area Network - Réseau dédié au stockage bloc |
| **iSCSI** | Internet SCSI - Protocole SCSI sur IP pour SAN |

---

### 🎯 À retenir ABSOLUMENT :

1. 💡 **Théorique** : RAID améliore capacité/performance/fiabilité mais NE REMPLACE PAS les sauvegardes
2. 💻 **Pratique** : Workflow LVM → `pvcreate` → `vgcreate` → `lvcreate` → `mkfs` → `mount`
3. ⚠️ **Piège** : Réduire un LV AVANT de réduire le système de fichiers = PERTE DE DONNÉES GARANTIE

---

### 🔧 Workflow complet LVM :

```bash
# 1. Préparer le disque
fdisk /dev/sdb                    # Créer partition type "8e" (LVM)

# 2. Créer PV
pvcreate /dev/sdb1

# 3. Créer VG
vgcreate vg_data /dev/sdb1

# 4. Créer LV
lvcreate -L 20G -n lv_app vg_data

# 5. Créer système de fichiers
mkfs.ext4 /dev/vg_data/lv_app

# 6. Monter
mkdir /mnt/app
mount /dev/vg_data/lv_app /mnt/app

# 7. Rendre permanent
echo "/dev/vg_data/lv_app /mnt/app ext4 defaults 0 0" >> /etc/fstab
```

---

### 📊 Comparaison RAID :

| Niveau | Disques min | Capacité | Performance | Fiabilité | Usage |
|--------|-------------|----------|-------------|-----------|-------|
| **RAID 0** | 2 | n × disque | ⭐⭐⭐⭐⭐ | ❌ Aucune | Performance pure (temporaire) |
| **RAID 1** | 2 | 1 × disque | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Données critiques |
| **RAID 5** | 3 | (n-1) × disque | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Compromis idéal serveurs |
| **RAID 6** | 4 | (n-2) × disque | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Très haute disponibilité |
| **RAID 10** | 4 | n/2 × disque | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Performance + fiabilité |

---

### 🔥 Scénarios de pannes :

**RAID 5 avec 4 disques (3 To utilisables) :**
- 1 disque tombe : ✅ Système fonctionne, reconstruction possible
- 2 disques tombent : ❌ PERTE TOTALE DES DONNÉES

**LVM - Snapshot avant mise à jour :**
```bash
# Avant MAJ risquée
lvcreate -L 5G -s -n snap_avant_maj /dev/vg_data/lv_app
# Si problème → lvconvert --merge pour rollback
# Si OK → lvremove /dev/vg_data/snap_avant_maj
```

---

### 💾 Différences NAS vs SAN :

| Critère | NAS | SAN |
|---------|-----|-----|
| **Niveau** | Fichiers (haut niveau) | Blocs (bas niveau) |
| **Protocoles** | NFS, SMB/CIFS, FTP | iSCSI, Fibre Channel, FCoE |
| **Réseau** | Ethernet standard | Réseau dédié spécialisé |
| **Complexité** | Simple | Complexe |
| **Coût** | Économique | Élevé |
| **Usage** | PME, partage fichiers | Datacenters, virtualisation |
