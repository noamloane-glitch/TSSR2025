# Partitionnement Windows

> Windows gère les disques avec des **lettres de lecteur** (C:, D:, E:...).  
> Montage automatique au démarrage — pas de `/etc/fstab` équivalent.  
> 3 outils : `diskpart` (CLI), `diskmgmt.msc` (GUI), PowerShell.

---

## Concepts clés Windows

| Concept | Windows | Linux équivalent |
|---------|---------|-----------------|
| Identification partition | Lettre `C:`, `D:`... | `/dev/sda1`, UUID |
| Montage | Automatique | Manuel + `/etc/fstab` |
| Partitionnement CLI | `diskpart` | `fdisk` / `parted` |
| Partitionnement GUI | `diskmgmt.msc` | `gparted` |
| Formatage CLI | `format` / `diskpart` | `mkfs.ext4` |
| FS par défaut | NTFS | ext4 |

### Systèmes de fichiers Windows

| FS | Usage | Taille max partition |
|----|-------|---------------------|
| **NTFS** | ✅ Standard Windows (journalisé, sécurité) | 16 Tio |
| **FAT32** | Compatibilité USB / multi-OS | 2 Tio (fichier max 4 Go) |
| **exFAT** | Clés USB modernes, fichiers > 4 Go | 128 Pio |
| **ReFS** | Serveurs, tolérance aux pannes | Très grande |

---

## Procédure avec `diskpart` (CLI — administrateur requis)

> Outil interactif en ligne de commande. Doit être lancé **en tant qu'administrateur**.

### Lancer diskpart

```cmd
diskpart
```

### Commandes de navigation

| Commande | Rôle |
|----------|------|
| `list disk` | Lister tous les disques physiques |
| `list volume` | Lister tous les volumes (partitions montées) |
| `list partition` | Lister les partitions du disque sélectionné |
| `select disk 1` | Sélectionner le disque 1 |
| `select partition 2` | Sélectionner la partition 2 |
| `select volume 3` | Sélectionner le volume 3 |
| `detail disk` | Détails du disque sélectionné |
| `detail partition` | Détails de la partition sélectionnée |

### Procédure complète : nouveau disque

```cmd
diskpart

# 1. Identifier le disque cible
list disk
select disk 1               # ⚠️ vérifier le bon numéro !

# 2. Initialiser le disque (nouveau disque vierge)
clean                       # ⚠️ DÉTRUIT TOUTES LES DONNÉES
convert gpt                 # table GPT (ou mbr pour MBR)

# 3. Créer les partitions
create partition primary size=10240    # 10 Go (en Mo)
create partition primary size=5120     # 5 Go
create partition primary               # reste du disque

# 4. Formater chaque partition
select partition 1
format fs=ntfs quick label="DATA"

select partition 2
format fs=ntfs quick label="PERSO"

select partition 3
format fs=fat32 quick label="SHARED"

# 5. Assigner des lettres de lecteur
select partition 1
assign letter=D

select partition 2
assign letter=E

select partition 3
assign letter=F

# 6. Vérifier
list volume

# 7. Quitter
exit
```

### Autres commandes utiles

| Commande | Rôle |
|----------|------|
| `shrink desired=5120` | Réduire le volume de 5 Go |
| `extend size=2048` | Étendre le volume de 2 Go |
| `delete partition` | ⚠️ Supprimer la partition sélectionnée |
| `remove letter=D` | Retirer la lettre de lecteur |
| `assign mount=C:\mount\data` | Monter dans un dossier (sans lettre) |
| `active` | Marquer la partition comme bootable (MBR) |
| `convert dynamic` | Convertir en disque dynamique |
| `convert basic` | Convertir en disque de base |

---

## Procédure avec `diskmgmt.msc` (GUI)

### Lancer l'outil

```
Win + R → diskmgmt.msc
# ou
Clic droit "Ce PC" → Gérer → Gestion des disques
# ou
Clic droit menu Démarrer → Gestion des disques
```

### Actions disponibles (clic droit)

| Zone de clic droit | Actions disponibles |
|-------------------|---------------------|
| **Espace non alloué** | Nouveau volume simple → assistant guidé |
| **Partition existante** | Formater / Supprimer / Réduire / Étendre |
| **Partition existante** | Modifier la lettre de lecteur |
| **Disque (zone grise gauche)** | Initialiser le disque / Convertir MBR↔GPT |

### Procédure : créer une partition (GUI)

1. Clic droit sur **espace non alloué** → `Nouveau volume simple`
2. Assistant → définir la **taille** (Mo)
3. Attribuer une **lettre de lecteur** ou un dossier
4. Choisir le **système de fichiers** (NTFS recommandé)
5. Définir le **label** (nom du volume)
6. Cocher **Formatage rapide**
7. Terminer

### Procédure : réduire un volume (GUI)

1. Clic droit sur la partition → `Réduire le volume`
2. Indiquer la taille à réduire (en Mo)
3. → Espace non alloué créé à droite

---

## Procédure avec PowerShell (administrateur requis)

```powershell
# Lister les disques
Get-Disk

# Lister les partitions d'un disque
Get-Partition -DiskNumber 1

# Lister les volumes
Get-Volume

# Initialiser un nouveau disque
Initialize-Disk -Number 1 -PartitionStyle GPT

# Créer une partition (taille en octets)
New-Partition -DiskNumber 1 -Size 10GB -DriveLetter D

# Formater la partition
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "DATA" -Confirm:$false

# Tout en une seule chaîne
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -Size 10GB -DriveLetter D
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "DATA" -Confirm:$false

# Supprimer une partition
Remove-Partition -DiskNumber 1 -PartitionNumber 2 -Confirm:$false

# Redimensionner un volume
Resize-Partition -DiskNumber 1 -PartitionNumber 2 -Size 20GB
```

---

## Comparatif des 3 méthodes

| Critère | `diskpart` | `diskmgmt.msc` | PowerShell |
|---------|-----------|----------------|------------|
| Interface | CLI interactif | GUI | CLI scriptable |
| Scriptable | ⚠️ Partiel | ❌ Non | ✅ Oui |
| WinPE / déploiement | ✅ Oui | ❌ Non | ✅ Oui |
| Débutant | ⚠️ Moyen | ✅ Facile | ⚠️ Moyen |
| Précision | ✅ Haute | ⚠️ Moyenne | ✅ Haute |

---

## Commandes de diagnostic

| Commande | Rôle |
|----------|------|
| `diskpart` + `list disk` | Voir tous les disques |
| `diskpart` + `list volume` | Voir tous les volumes montés |
| `Get-Disk` | Infos disques (PowerShell) |
| `Get-Volume` | Infos volumes (PowerShell) |
| `Get-Partition -DiskNumber 1` | Partitions d'un disque |
| `chkdsk D: /f` | Vérifier/réparer un volume |
| `chkdsk D: /r` | Vérifier + récupérer secteurs défectueux |

---

## ⚠️ À retenir absolument

- `diskpart` et PowerShell nécessitent d'être lancés **en tant qu'administrateur**
- `clean` dans diskpart = **destruction immédiate** de toutes les données — vérifier `list disk` avant !
- Windows monte automatiquement les disques — **pas de fstab** à configurer
- NTFS = FS par défaut pour les partitions Windows
- FAT32 = limite fichier à **4 Go** → utiliser exFAT pour les gros fichiers sur USB
- `convert gpt` / `convert mbr` dans diskpart = **perte des données** du disque
- Tailles dans `diskpart` = en **Mo** (`size=10240` = 10 Go)
- Tailles dans PowerShell = en **octets** (`-Size 10GB` = notation automatique)
