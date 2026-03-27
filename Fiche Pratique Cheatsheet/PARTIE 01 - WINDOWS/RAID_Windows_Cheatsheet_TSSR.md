# RAID sous Windows - Cheatsheet TSSR
> CCP 8 — Sauvegardes & restaurations / CCP 2 — Exploiter des serveurs Windows

## Concepts clés

| Concept | Valeur / Description |
|--------|-------------------|
| RAID logiciel | Géré par Windows (Gestionnaire de disques ou Storage Spaces) |
| RAID matériel | Géré par le contrôleur RAID (carte dédiée, indépendant de l'OS) |
| Disque de base | Mode par défaut — partitions simples, pas de RAID |
| Disque dynamique | Requis pour RAID logiciel Windows (sauf Storage Spaces) |
| Storage Spaces | Technologie RAID moderne Windows Server 2012+ / Windows 8+ |
| Pool de stockage | Groupe de disques physiques dans Storage Spaces |
| Espace de stockage | Volume virtuel créé depuis un pool |

## Types de volumes (disques dynamiques)

| Type | Équivalent RAID | Description |
|------|----------------|-------------|
| Volume fractionné | — | Données réparties sur 2+ disques sans redondance |
| Volume agrégé par bandes | RAID 0 | Striping, performance, aucune tolérance de panne |
| Volume en miroir | RAID 1 | Duplication sur 2 disques |
| RAID-5 | RAID 5 | Striping + parité, min. 3 disques (Server uniquement) |

## Commandes et outils

| Action | Commande / Outil |
|--------|-----------------|
| Gestionnaire de disques (GUI) | `diskmgmt.msc` |
| Gestion via CLI | `diskpart` |
| Lister les disques | `list disk` (dans diskpart) |
| Convertir en disque dynamique | `convert dynamic` (dans diskpart) |
| Créer un miroir (PowerShell) | `New-StoragePool`, `New-VirtualDisk` |
| Voir les Storage Spaces | `Get-StoragePool` |
| Créer un pool | `New-StoragePool -FriendlyName "Pool1" -StorageSubSystemFriendlyName ...` |
| Créer un volume miroir | `New-VirtualDisk -StoragePoolFriendlyName "Pool1" -ResiliencySettingName Mirror` |
| Réparer un volume | `Repair-VirtualDisk -FriendlyName "NomVolume"` |
| État des disques virtuels | `Get-VirtualDisk` |

## Storage Spaces — Niveaux de résilience

| Résilience | Équivalent | Disques min. | Tolérance |
|------------|-----------|-------------|-----------|
| Simple | RAID 0 | 1 | Aucune |
| Mirror (2 voies) | RAID 1 | 2 | 1 disque |
| Mirror (3 voies) | — | 5 | 2 disques |
| Parity | RAID 5 | 3 | 1 disque |
| Dual Parity | RAID 6 | 7 | 2 disques |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| RAID ≠ sauvegarde | Ne protège pas contre corruption ou suppression accidentelle |
| Disques dynamiques | Non supportés en dual-boot ou sur disques amovibles |
| RAID 5 logiciel Windows | Disponible uniquement sur Windows Server, pas sur les éditions Desktop |
| Rebuild consommateur | Reconstruction sollicite fortement les I/O — prévoir hors production |
| Storage Spaces vs RAID classique | Préférer Storage Spaces sur Windows Server 2012+ |
| Surveiller l'état | Vérifier régulièrement dans `diskmgmt.msc` ou via `Get-VirtualDisk` |
| Perte de disque dynamique | La conversion disque dynamique → de base nécessite suppression des volumes |
