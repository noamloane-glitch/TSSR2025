# LVM sous Linux - Cheatsheet TSSR
> CCP 3 — Exploiter des serveurs Linux / CCP 8 — Sauvegardes & restaurations

## Concepts clés

| Concept | Description |
|--------|-------------|
| PV (Physical Volume) | Disque ou partition préparé pour LVM (`/dev/sdb`, `/dev/sdb1`) |
| VG (Volume Group) | Groupe de PV — pool de stockage global |
| LV (Logical Volume) | Volume logique créé depuis un VG — utilisé comme partition |
| PE (Physical Extent) | Unité d'allocation de base dans un VG (défaut : 4 Mo) |
| Thin provisioning | Allocation à la demande — espace réservé virtuellement |
| Snapshot | Copie instantanée d'un LV — utile pour sauvegardes à chaud |

## Commandes — Physical Volumes

| Action | Commande |
|--------|----------|
| Initialiser un disque en PV | `pvcreate /dev/sdb` |
| Lister les PV | `pvs` ou `pvdisplay` |
| Supprimer un PV | `pvremove /dev/sdb` |

## Commandes — Volume Groups

| Action | Commande |
|--------|----------|
| Créer un VG | `vgcreate vg_data /dev/sdb` |
| Ajouter un PV à un VG | `vgextend vg_data /dev/sdc` |
| Retirer un PV d'un VG | `vgreduce vg_data /dev/sdc` |
| Lister les VG | `vgs` ou `vgdisplay` |
| Supprimer un VG | `vgremove vg_data` |

## Commandes — Logical Volumes

| Action | Commande |
|--------|----------|
| Créer un LV de 10 Go | `lvcreate -L 10G -n lv_home vg_data` |
| Créer un LV en % du VG | `lvcreate -l 100%FREE -n lv_data vg_data` |
| Étendre un LV (+5 Go) | `lvextend -L +5G /dev/vg_data/lv_home` |
| Étendre + redimensionner FS | `lvextend -r -L +5G /dev/vg_data/lv_home` |
| Réduire un LV (ext4) | `resize2fs` puis `lvreduce -L -2G /dev/vg_data/lv_home` |
| Créer un snapshot | `lvcreate -s -L 2G -n snap_home /dev/vg_data/lv_home` |
| Supprimer un LV | `lvremove /dev/vg_data/lv_home` |
| Lister les LV | `lvs` ou `lvdisplay` |
| Formater un LV | `mkfs.ext4 /dev/vg_data/lv_home` |
| Monter un LV | `mount /dev/vg_data/lv_home /mnt/home` |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| Réduction = risque de perte | Toujours démonter et vérifier le FS avant de réduire |
| `lvextend` sans `-r` | Le FS n'est pas redimensionné automatiquement sans l'option `-r` |
| Snapshot = espace limité | Si le snapshot est plein, il devient invalide |
| `/etc/fstab` | Utiliser l'UUID ou le chemin `/dev/vg/lv` pour le montage persistant |
| xfs ne se réduit pas | XFS ne supporte que l'extension, jamais la réduction |
| Sauvegarder avant modification | Toujours sauvegarder avant `lvreduce` ou `vgreduce` |
