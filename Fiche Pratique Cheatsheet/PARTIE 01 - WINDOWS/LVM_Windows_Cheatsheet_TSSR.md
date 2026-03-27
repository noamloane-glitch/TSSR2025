# LVM sous Windows - Cheatsheet TSSR
> CCP 2 — Exploiter des serveurs Windows / CCP 8 — Sauvegardes & restaurations

## Concepts clés

| Concept | Description |
|--------|-------------|
| LVM | Logical Volume Manager — n'existe pas nativement sous Windows |
| Équivalent Windows | Disques dynamiques + volumes logiques (ou Storage Spaces) |
| Volume fractionné | Regroupe plusieurs disques en un seul volume logique (≈ LVM PV+VG+LV) |
| Storage Spaces | Technologie moderne équivalente au LVM (Windows 8+ / Server 2012+) |
| Pool de stockage | Regroupe des disques physiques (≈ Volume Group LVM) |
| Espace de stockage | Volume logique créé depuis un pool (≈ Logical Volume LVM) |
| Thin provisioning | Allocation à la demande — disponible dans Storage Spaces |
| Extend on-the-fly | Extension possible sans démonter le volume |

## Correspondance LVM Linux ↔ Windows

| Concept LVM Linux | Équivalent Windows |
|-------------------|--------------------|
| Physical Volume (PV) | Disque physique dans un pool |
| Volume Group (VG) | Pool de stockage (Storage Spaces) |
| Logical Volume (LV) | Espace de stockage virtuel |
| `lvcreate` | `New-VirtualDisk` |
| `lvextend` | `Resize-VirtualDisk` |
| `vgcreate` | `New-StoragePool` |
| `pvs / vgs / lvs` | `Get-PhysicalDisk`, `Get-StoragePool`, `Get-VirtualDisk` |

## Commandes PowerShell

| Action | Commande |
|--------|----------|
| Lister les disques physiques | `Get-PhysicalDisk` |
| Créer un pool de stockage | `New-StoragePool -FriendlyName "Pool1" -StorageSubSystemFriendlyName (Get-StorageSubSystem).FriendlyName -PhysicalDisks (Get-PhysicalDisk -CanPool $true)` |
| Créer un volume simple | `New-VirtualDisk -StoragePoolFriendlyName "Pool1" -FriendlyName "Vol1" -Size 50GB -ResiliencySettingName Simple` |
| Étendre un volume virtuel | `Resize-VirtualDisk -FriendlyName "Vol1" -Size 100GB` |
| Initialiser et formater | `Initialize-Disk`, puis `New-Partition`, puis `Format-Volume` |
| Voir les pools | `Get-StoragePool` |
| Voir les volumes virtuels | `Get-VirtualDisk` |
| Supprimer un volume | `Remove-VirtualDisk -FriendlyName "Vol1"` |
| Extension via diskpart | `extend size=X` (dans diskpart, pour volumes dynamiques) |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| LVM n'existe pas sous Windows | Le terme LVM est Linux — utiliser Storage Spaces ou disques dynamiques |
| Disques dynamiques = legacy | Microsoft recommande Storage Spaces depuis Server 2012 |
| Extension non réversible | Agrandir un volume est facile, réduire est risqué ou impossible |
| Thin provisioning | Risque de surconsommation si les VMs écrivent plus que prévu |
| Sauvegarde avant extension | Toujours sauvegarder avant modification de la structure de stockage |
| Compatibilité | Les pools Storage Spaces ne sont pas portables entre serveurs facilement |
