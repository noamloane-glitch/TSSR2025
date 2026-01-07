# Gestion du Processeur et de la Mémoire - Systèmes d'Exploitation
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Gestion des processeurs et de la mémoire  
**Date** : Novembre 2024  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. Introduction
2. Notion de processus
3. Gestion de la mémoire
4. L'approche GNU/Linux
5. Gestion avec PowerShell
6. Démarrage et services
7. Points clés à retenir
8. Glossaire technique

---

## Introduction

> [!abstract] Vue d'ensemble
> Ce cours couvre la gestion des processeurs et de la mémoire. Tu découvriras les mécanismes d'ordonnancement, de mémoire virtuelle, ainsi que les outils de gestion sous GNU/Linux et Windows.

### Compétences RNCP visées

- Surveiller l'utilisation des ressources système
- Gérer les processus et services
- Diagnostiquer les problèmes de performance
- Configurer le démarrage système
- Automatiser des tâches

---

## Notion de processus

### Programme vs Processus

> [!quote] Programme
> Séquence d'instructions machines (code binaire) stockée en mémoire. Élément statique.

> [!quote] Processus  
> Programme en cours d'exécution avec son propre espace mémoire, ses données et son état.

### Système à temps partagé

> [!important] Multitâche préemptif
> Système simulant l'exécution simultanée de plusieurs programmes en les faisant s'exécuter tour à tour rapidement.

**Objectifs :**
- Réactivité pour l'utilisateur
- Efficacité d'utilisation du CPU
- Protection entre processus

### Mode processeur

> [!info] Modes d'exécution
> - **Mode noyau** : accès complet, instructions privilégiées
> - **Mode utilisateur** : restrictions, appels système obligatoires

**Interruptions :**
- Horloge : base du multitâche
- I/O : fin d'opération
- Exceptions : erreurs

---

## Gestion de la mémoire

### Mémoire virtuelle

> [!important] Principe
> Chaque processus a l'illusion d'une mémoire complète. La MMU traduit adresses virtuelles ↔ physiques.

**Pagination :**
- Mémoire divisée en pages (4 Ko)
- Table des pages par processus
- Droits d'accès par page

### Swap

> [!info] Mémoire de débordement
> Pages peu utilisées déportées sur disque pour libérer la RAM.

**Avantages / Inconvénients :**
- ✅ Plus de mémoire que RAM
- ❌ Très lent (disque vs RAM)
- ❌ Thrashing si excessif

### PCB et ordonnancement

> [!quote] PCB - Process Control Block
> Structure de données contenant : PID, PPID, état, priorité, registres, pointeurs mémoire, métriques.

**Ordonnanceur :**
- FIFO, Round Robin, Priorités
- CFS (Linux) : ordonnanceur équitable

### Threads

> [!info] Processus légers
> Plusieurs fils d'exécution dans un processus, partageant la mémoire. Création légère, communication facile.

---

## L'approche GNU/Linux

### Métadonnées processus

| Métadonnée | Description |
|------------|-------------|
| **PID** | Identifiant unique |
| **PPID** | Processus parent |
| **UID/GID** | Propriétaire/Groupe |
| **TTY** | Terminal associé |
| **État** | R/S/Z/T |

**Accessible via `/proc/<PID>/`**

### Arborescence

> [!important] Init (PID=1)
> Tous les processus descendent de **init/systemd**. Relation parent-enfant par fork().

### Signaux Linux

| Signal | Numéro | Action | Description |
|--------|--------|--------|-------------|
| **SIGINT** | 2 | Terminer | Ctrl+C |
| **SIGTERM** | 15 | Terminer | Arrêt propre |
| **SIGKILL** | 9 | Terminer | Destruction immédiate |
| **SIGSTOP** | 19 | Suspendre | Pause |
| **SIGCONT** | 18 | Reprendre | Reprise |

### Commandes essentielles

```bash
# Visualisation
ps aux                  # Liste processus
ps aux --forest         # Vue arborescence
pstree                  # Arbre visuel
top / htop              # Surveillance interactive

# Contrôle
kill -15 <PID>          # SIGTERM
kill -9 <PID>           # SIGKILL
killall firefox         # Par nom
pkill -u wilder         # Par utilisateur

# Gestion fg/bg
<cmd> &                 # Lancer en arrière-plan
Ctrl+Z                  # Suspendre
bg                      # Reprendre en arrière-plan
fg                      # Ramener en premier plan

# Détachement
nohup ./script.sh &     # Survit à la déconnexion
```

### Planification cron

**Format crontab :**
```
* * * * * commande
│ │ │ │ │
│ │ │ │ └─ Jour semaine (0-7)
│ │ │ └─── Mois (1-12)
│ │ └───── Jour mois (1-31)
│ └─────── Heure (0-23)
└───────── Minute (0-59)
```

**Exemples :**
```bash
# Éditer crontab
crontab -e

# Tous les jours à 3h30
30 3 * * * /chemin/backup.sh

# Toutes les 5 minutes
*/5 * * * * /chemin/check.sh

# Lundis à 8h
0 8 * * 1 /chemin/rapport.sh
```

**Commande at :**
```bash
# Tâche ponctuelle
echo "/chemin/script.sh" | at 14:30
echo "backup.sh" | at now + 2 hours

# Liste et suppression
atq                     # Liste
atrm 7                  # Supprimer tâche 7
```

### Gestion swap Linux

```bash
# Visualisation
free -h                 # Mémoire et swap
swapon --show           # Swaps actifs

# Gestion
sudo swapon -a          # Activer tous
sudo swapoff -a         # Désactiver tous
sudo mkswap /dev/sda2   # Initialiser partition

# Créer fichier swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Permanent dans /etc/fstab
/swapfile  none  swap  sw  0  0
```

---

## Gestion avec PowerShell

### Métadonnées Windows

| Métadonnée | Équivalent Linux |
|------------|------------------|
| **Id** | PID |
| **ProcessName** | CMD |
| **ParentId** | PPID |
| **UserName** | UID |
| **Handles** | File descriptors |

### Cmdlets principaux

```powershell
# Visualisation
Get-Process                                    # Tous les processus
Get-Process -Name firefox                      # Par nom
Get-Process -Id 1234                           # Par ID
Get-Process -IncludeUserName                   # Avec utilisateur

# Trier par ressources
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# Contrôle
Stop-Process -Id 1234                          # Arrêter
Stop-Process -Name firefox -Force              # Forcer
Start-Process notepad.exe                      # Lancer
Wait-Process -Name notepad                     # Attendre fin

# Exécution
Invoke-Command -ScriptBlock { Get-Process }    # Local
Invoke-Command -ComputerName SRV01 -ScriptBlock { Get-Process }  # Distant

# Informations système
Get-ComputerInfo
Get-CimInstance -ClassName Win32_OperatingSystem
Get-CimInstance -ClassName Win32_Processor
```

---

## Démarrage et services

### Boot MBR/UEFI

**MBR (Legacy) :**
```
BIOS → MBR → Bootloader → Noyau
```
- Limite 2 To
- 4 partitions primaires max

**UEFI :**
```
UEFI → ESP (FAT32) → Bootloader → Noyau
```
- Support > 2 To (GPT)
- Secure Boot : bootloaders signés

### Bootloaders

**GRUB (Linux) :**
- Multi-boot universel
- Configuration : `/etc/default/grub`
- Régénération : `sudo update-grub`

**Bootmgr (Windows) :**
- Lit BCD (Boot Configuration Data)
- Lance winload.exe
- SCM démarre les services

### Services Linux - Systemd

```bash
# État service
systemctl status nginx

# Contrôle
sudo systemctl start nginx       # Démarrer
sudo systemctl stop nginx        # Arrêter
sudo systemctl restart nginx     # Redémarrer
sudo systemctl reload nginx      # Recharger config

# Démarrage auto
sudo systemctl enable nginx      # Activer
sudo systemctl disable nginx     # Désactiver
sudo systemctl enable --now nginx  # Activer + démarrer

# Vérification
systemctl is-enabled nginx
systemctl is-active nginx

# Liste
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl --failed               # Services échoués

# Logs
journalctl -u nginx              # Logs service
journalctl -u nginx -f           # Temps réel
journalctl -b                    # Depuis boot
journalctl --since today
journalctl -p err                # Erreurs uniquement

# Analyse boot
systemd-analyze
systemd-analyze blame            # Par temps
```

### Services Windows

```powershell
# Liste
Get-Service
Get-Service | Where-Object {$_.Status -eq "Running"}

# Contrôle
Start-Service -Name "W32Time"
Stop-Service -Name "W32Time"
Restart-Service -Name "W32Time"
Suspend-Service -Name "W32Time"        # Pause
Resume-Service -Name "W32Time"         # Reprise

# Configuration
Set-Service -Name "W32Time" -StartupType Automatic
Set-Service -Name "W32Time" -StartupType Manual
Set-Service -Name "W32Time" -StartupType Disabled

# Dépendances
Get-Service -Name "W32Time" -DependentServices
Get-Service -Name "W32Time" -RequiredServices
```

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **Programme** : code statique sur disque
- **Processus** : programme en exécution avec ressources
- **Multitâche préemptif** : partage du CPU par commutations rapides
- **Ordonnanceur** : décide quel processus exécuter

### Mémoire

- **Mémoire virtuelle** : illusion mémoire complète par processus
- **MMU** : traduction adresses virtuelles ↔ physiques
- **Pagination** : division en pages de 4 Ko
- **Swap** : extension RAM sur disque (lent)
- **Thrashing** : swap excessif → performances catastrophiques

### Signaux Linux

| Signal | Usage |
|--------|-------|
| **SIGTERM (15)** | Arrêt propre (essayer en premier) |
| **SIGKILL (9)** | Destruction immédiate (dernier recours) |
| **SIGINT (2)** | Ctrl+C |

### Commandes Linux essentielles

```bash
ps aux --forest         # Liste processus
htop                    # Surveillance interactive
kill -15 <PID>          # Arrêt propre
kill -9 <PID>           # Force
nohup cmd &             # Détachement session
crontab -e              # Planification
free -h                 # Mémoire/swap
systemctl status svc    # État service
```

### PowerShell essentiels

```powershell
Get-Process             # Liste processus
Stop-Process -Id <ID>   # Arrêt
Get-Service             # Liste services
Start-Service -Name svc # Démarrer service
```

### Démarrage

- **UEFI** > BIOS (moderne, > 2 To, Secure Boot)
- **GRUB** (Linux) : multi-boot
- **Bootmgr** (Windows) : BCD → winload.exe → SCM
- **Systemd** (Linux) : init moderne, parallélisation
- **SCM** (Windows) : gestionnaire de services

### Pièges à éviter

1. ❌ `kill -9` systématique → toujours essayer `-15` d'abord
2. ❌ Oublier `sudo` pour opérations système
3. ❌ Swap > 50% régulier → ajouter RAM
4. ❌ Chemins relatifs dans crontab → toujours absolus
5. ❌ Service `start` sans `enable` → pas de démarrage auto

### Bonnes pratiques

1. ✅ Monitorer régulièrement CPU/RAM (`htop`, Task Manager)
2. ✅ Surveiller le swap (usage élevé = besoin RAM)
3. ✅ Consulter logs système (`journalctl`, Event Viewer)
4. ✅ Tester scripts avant planification cron
5. ✅ Documenter services personnalisés
6. ✅ SIGTERM avant SIGKILL
7. ✅ Utiliser `nohup` pour tâches longues SSH
8. ✅ Désactiver services inutiles
9. ✅ Sauvegarder config bootloader avant modification
10. ✅ Maintenir média de récupération

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Processus** | Programme en exécution avec état et ressources |
| **Thread** | Fil d'exécution léger au sein d'un processus |
| **Multitâche préemptif** | Exécution apparemment simultanée de programmes |
| **Ordonnanceur** | Décide quel processus exécuter et quand |
| **PCB** | Structure de données OS contenant métadonnées processus |
| **PID** | Identifiant unique du processus |
| **PPID** | PID du processus parent |
| **Mémoire virtuelle** | Abstraction donnant illusion mémoire complète |
| **MMU** | Traduit adresses virtuelles en physiques |
| **Pagination** | Division mémoire en pages fixes (4 Ko) |
| **Table des pages** | Correspondance virtuel/physique avec droits |
| **Swap** | Extension RAM sur disque |
| **Défaut de page** | Exception lors d'accès page non en RAM |
| **Thrashing** | Swap excessif paralysant le système |
| **Signal** | Notification inter-processus |
| **SIGTERM** | Demande arrêt propre |
| **SIGKILL** | Destruction immédiate |
| **Zombie** | Processus terminé non récupéré par parent |
| **Orphelin** | Processus dont parent est mort |
| **Démon** | Processus arrière-plan lancé au boot |
| **Init/Systemd** | Premier processus (PID=1) |
| **Cron** | Planificateur tâches récurrentes |
| **nohup** | Détache processus de session |
| **UEFI** | Firmware moderne remplaçant BIOS |
| **ESP** | Partition FAT32 contenant bootloaders UEFI |
| **Secure Boot** | N'autorise que bootloaders signés |
| **GRUB** | Bootloader principal Linux |
| **Bootmgr** | Bootloader Windows |
| **Systemd** | Système démarrage Linux moderne |
| **Systemctl** | Gestion services systemd |
| **SCM** | Service Control Manager Windows |
| **CFS** | Completely Fair Scheduler Linux |
| **Nice value** | Priorité processus (-20 à +19) |
| **Fork** | Création processus enfant clone |
| **WMI** | Windows Management Instrumentation |
| **Cmdlet** | Commande PowerShell (Verbe-Nom) |

---

> [!success] Document de révision complet
> Ce document couvre l'intégralité du cours sur la **gestion des processeurs et de la mémoire**. Tu as maintenant tous les éléments pour :
> - Comprendre le multitâche et l'ordonnancement
> - Maîtriser la mémoire virtuelle et le swap
> - Gérer les processus Linux et Windows
> - Planifier des tâches (cron, at)
> - Administrer les services (systemd, SCM)
> - Comprendre le boot système
> - Diagnostiquer les problèmes de performance
> 
> **Bon courage pour ton titre RNCP TSSR !** 🎓✨

---

**Fin du document de révision**
