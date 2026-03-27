

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

## 🔌 Ajout et retrait de disques

### Ajout d'un disque à un array existant

L'ajout d'un disque permet d'augmenter la redondance ou la capacité d'un array RAID. Cette opération varie selon le niveau RAID utilisé.

> [!info] Contexte d'utilisation
> 
> - Augmenter la redondance d'un RAID (ajouter un spare)
> - Étendre la capacité d'un RAID 5/6
> - Remplacer un disque défaillant après l'avoir retiré

#### Syntaxe pour ajouter un disque

```bash
# Ajouter un disque à un array
mdadm --add /dev/md0 /dev/sdd1

# Ajouter plusieurs disques simultanément
mdadm --add /dev/md0 /dev/sdd1 /dev/sde1

# Vérifier l'état après l'ajout
mdadm --detail /dev/md0
```

> [!example] Exemple pratique : Ajout d'un spare
> 
> ```bash
> # Préparer le nouveau disque
> parted /dev/sdd mklabel gpt
> parted /dev/sdd mkpart primary 0% 100%
> 
> # Ajouter le disque comme spare
> mdadm --add /dev/md0 /dev/sdd1
> 
> # Vérifier son statut
> cat /proc/mdstat
> # Le disque apparaît comme (S) = spare
> ```

#### Augmenter la taille d'un array (grow)

Pour certains niveaux RAID (1, 5, 6, 10), il est possible d'ajouter des disques pour augmenter la capacité.

```bash
# Étape 1 : Ajouter le nouveau disque
mdadm --add /dev/md0 /dev/sdd1

# Étape 2 : Augmenter le nombre de disques actifs
mdadm --grow /dev/md0 --raid-devices=4

# Étape 3 : Suivre la reconstruction
watch -n 1 cat /proc/mdstat

# Étape 4 : Étendre le système de fichiers (après reconstruction)
resize2fs /dev/md0  # Pour ext4
xfs_growfs /mnt/raid  # Pour XFS
```

> [!warning] Attention lors du grow
> 
> - La reconstruction peut prendre plusieurs heures selon la taille
> - Les performances sont dégradées pendant l'opération
> - Assurez-vous d'avoir une sauvegarde avant de modifier la structure
> - N'oubliez pas d'étendre le système de fichiers après

### Retrait d'un disque d'un array

Le retrait d'un disque nécessite généralement de le marquer comme défaillant avant de pouvoir le retirer physiquement.

```bash
# Étape 1 : Marquer le disque comme défaillant
mdadm --fail /dev/md0 /dev/sdc1

# Étape 2 : Retirer le disque de l'array
mdadm --remove /dev/md0 /dev/sdc1

# Vérifier que le disque a été retiré
mdadm --detail /dev/md0
```

> [!tip] Astuce : Retrait d'un spare Un disque spare peut être retiré directement sans le marquer comme défaillant :
> 
> ```bash
> mdadm --remove /dev/md0 /dev/sdd1
> ```

---

## ⚠️ Marquage d'un disque comme défaillant

### Comprendre le marquage manuel

Le marquage manuel d'un disque comme défaillant (`--fail`) simule une panne. C'est utile pour tester la redondance ou pour retirer un disque suspect avant qu'il ne tombe vraiment en panne.

> [!info] Quand marquer un disque comme défaillant ?
> 
> - Pour tester la procédure de failover
> - Lorsque vous observez des erreurs SMART
> - Avant de retirer physiquement un disque
> - Pour forcer l'utilisation d'un spare

### Syntaxe du marquage

```bash
# Marquer un disque comme défaillant
mdadm --fail /dev/md0 /dev/sdc1

# Syntaxe alternative (forme courte)
mdadm -f /dev/md0 /dev/sdc1

# Vérifier l'état après marquage
mdadm --detail /dev/md0
cat /proc/mdstat
```

> [!example] Exemple de marquage et observation
> 
> ```bash
> # État avant
> mdadm --detail /dev/md0 | grep -A 4 "Number"
> #    Number   Major   Minor   RaidDevice State
> #       0       8       16        0      active sync   /dev/sdb1
> #       1       8       32        1      active sync   /dev/sdc1
> 
> # Marquer sdc1 comme défaillant
> mdadm --fail /dev/md0 /dev/sdc1
> 
> # État après
> mdadm --detail /dev/md0 | grep -A 5 "Number"
> #    Number   Major   Minor   RaidDevice State
> #       0       8       16        0      active sync   /dev/sdb1
> #       2       8       48        1      spare rebuilding   /dev/sdd1
> #
> #       1       8       32        -      faulty   /dev/sdc1
> ```

### Conséquences du marquage

|Action automatique|Description|
|---|---|
|**Activation du spare**|Si un spare est disponible, il remplace immédiatement le disque défaillant|
|**Reconstruction**|Le RAID entame une reconstruction sur le spare|
|**Logs système**|Des messages sont enregistrés dans `/var/log/syslog` ou `journalctl`|
|**État dégradé**|L'array passe en mode dégradé jusqu'à reconstruction complète|

> [!warning] Précautions
> 
> - Un array RAID 0 devient inutilisable si un disque est marqué défaillant
> - Un RAID 5 avec un disque défaillant peut perdre toutes les données si un second disque tombe en panne pendant la reconstruction
> - Vérifiez toujours qu'un spare est disponible avant de marquer un disque

---

## 🔄 Remplacement d'un disque défaillant

Le remplacement d'un disque défaillant est une opération courante en production. Elle doit être effectuée méthodiquement pour garantir l'intégrité des données.

### Procédure complète de remplacement

#### Étape 1 : Identifier le disque défaillant

```bash
# Vérifier l'état de l'array
mdadm --detail /dev/md0

# Consulter les logs du kernel
dmesg | grep -i "raid\|md"

# Vérifier /proc/mdstat
cat /proc/mdstat
```

> [!example] Sortie typique d'un disque défaillant
> 
> ```
> /dev/md0:
>    Number   Major   Minor   RaidDevice State
>       0       8       16        0      active sync   /dev/sdb1
>       -       0        0        1      removed
> 
>       2       8       32        -      faulty   /dev/sdc1
> ```

#### Étape 2 : Retirer le disque défaillant

```bash
# Si le disque n'est pas déjà marqué comme faulty, le marquer
mdadm --fail /dev/md0 /dev/sdc1

# Retirer le disque de l'array
mdadm --remove /dev/md0 /dev/sdc1

# Vérifier le retrait
mdadm --detail /dev/md0 | grep "/dev/sdc1"
# Ne devrait rien retourner
```

#### Étape 3 : Préparer le nouveau disque

```bash
# Identifier le nouveau disque (exemple : /dev/sde)
lsblk

# Copier la table de partitions du disque sain (si même taille)
sfdisk -d /dev/sdb | sfdisk /dev/sde

# Alternative : créer manuellement la partition
parted /dev/sde mklabel gpt
parted /dev/sde mkpart primary 0% 100%
parted /dev/sde set 1 raid on
```

> [!tip] Copie de table de partitions La méthode `sfdisk -d` est rapide et évite les erreurs de taille de partition.

#### Étape 4 : Ajouter le nouveau disque à l'array

```bash
# Ajouter le disque
mdadm --add /dev/md0 /dev/sde1

# Vérifier que la reconstruction démarre
cat /proc/mdstat
# md0 : active raid1 sde1[2] sdb1[0]
#       10475520 blocks super 1.2 [2/1] [U_]
#       [>....................]  recovery =  0.8% (88192/10475520)
```

#### Étape 5 : Surveiller la reconstruction

```bash
# Surveiller en temps réel
watch -n 2 cat /proc/mdstat

# Vérifier les détails
mdadm --detail /dev/md0

# Estimer le temps restant
cat /proc/mdstat | grep finish
```

> [!info] Durée de reconstruction La durée dépend de :
> 
> - La taille du disque (plusieurs heures pour des TB)
> - La charge système (I/O)
> - Les limites de vitesse configurées dans `/proc/sys/dev/raid/speed_limit_*`

### Remplacement à chaud (hot swap)

Sur du matériel supportant le hot swap, vous pouvez retirer physiquement le disque sans éteindre le serveur.

```bash
# 1. Marquer et retirer logiquement
mdadm --fail /dev/md0 /dev/sdc1
mdadm --remove /dev/md0 /dev/sdc1

# 2. Retirer physiquement le disque
echo 1 > /sys/block/sdc/device/delete

# 3. Insérer le nouveau disque physiquement

# 4. Rescanner les bus SCSI
echo "- - -" > /sys/class/scsi_host/host0/scan

# 5. Vérifier la détection du nouveau disque
lsblk

# 6. Ajouter le disque à l'array
mdadm --add /dev/md0 /dev/sdc1
```

> [!warning] Hot swap : précautions
> 
> - Vérifiez que votre matériel supporte le hot swap
> - Utilisez les LEDs de façade pour identifier physiquement le bon disque
> - Notez le numéro de série avant retrait : `smartctl -i /dev/sdc`

---

## 🔨 Reconstruction d'un array

### Comprendre le processus de reconstruction (resync)

La reconstruction est le processus par lequel le RAID reconstruit les données sur un disque neuf ou après une panne. Pendant cette phase, l'array fonctionne en mode dégradé.

> [!info] Types de reconstruction
> 
> - **Recovery** : Reconstruction après remplacement d'un disque défaillant
> - **Resync** : Vérification et synchronisation périodique de tous les disques
> - **Check** : Vérification de la cohérence sans modification (lecture seule)
> - **Repair** : Correction des incohérences détectées

### Suivre la progression de la reconstruction

```bash
# Méthode 1 : /proc/mdstat (la plus simple)
cat /proc/mdstat
# md0 : active raid5 sdd1[3] sdc1[2] sdb1[1] sda1[0]
#       1953260544 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/3] [UUU_]
#       [==>..................]  recovery = 12.6% (82416768/651086848)
#       finish=127.3min speed=74432K/sec

# Méthode 2 : mdadm --detail
mdadm --detail /dev/md0 | grep -A 5 "State"

# Méthode 3 : Surveillance continue
watch -n 1 'cat /proc/mdstat | grep -A 2 md0'
```

### Informations importantes dans /proc/mdstat

|Information|Signification|
|---|---|
|`[UUU_]`|3 disques actifs (U), 1 absent (_)|
|`[=====>...........]`|Barre de progression visuelle|
|`recovery = 12.6%`|Pourcentage de progression|
|`finish=127.3min`|Temps estimé restant|
|`speed=74432K/sec`|Vitesse de reconstruction actuelle|

### Contrôler la vitesse de reconstruction

La vitesse de reconstruction peut être ajustée pour équilibrer entre performance et temps de reconstruction.

```bash
# Voir les limites actuelles (en Ko/sec)
cat /proc/sys/dev/raid/speed_limit_min
cat /proc/sys/dev/raid/speed_limit_max

# Augmenter la vitesse maximale (100 Mo/sec)
echo 102400 > /proc/sys/dev/raid/speed_limit_max

# Augmenter la vitesse minimale (50 Mo/sec)
echo 51200 > /proc/sys/dev/raid/speed_limit_min

# Rendre permanent (dans /etc/sysctl.conf)
echo "dev.raid.speed_limit_min = 51200" >> /etc/sysctl.conf
echo "dev.raid.speed_limit_max = 102400" >> /etc/sysctl.conf
```

> [!tip] Optimisation de la vitesse
> 
> - **Production** : Limites basses pour ne pas impacter les performances
> - **Maintenance** : Limites hautes pour reconstruire rapidement
> - Par défaut : min=1000 Ko/s, max=200000 Ko/s

### Lancer manuellement une vérification

```bash
# Lancer une vérification (check)
echo check > /sys/block/md0/md/sync_action

# Lancer une réparation (repair)
echo repair > /sys/block/md0/md/sync_action

# Voir l'action en cours
cat /sys/block/md0/md/sync_action

# Voir les erreurs détectées
cat /sys/block/md0/md/mismatch_cnt
```

> [!example] Vérification mensuelle automatique
> 
> ```bash
> # Créer un script de vérification
> cat << 'EOF' > /usr/local/bin/raid-check.sh
> #!/bin/bash
> for array in /dev/md*; do
>     echo check > /sys/block/$(basename $array)/md/sync_action
> done
> EOF
> 
> chmod +x /usr/local/bin/raid-check.sh
> 
> # Ajouter à cron (1er dimanche du mois à 2h)
> echo "0 2 1-7 * 0 root /usr/local/bin/raid-check.sh" >> /etc/crontab
> ```

### Interrompre une reconstruction

```bash
# Mettre en pause la reconstruction
echo idle > /sys/block/md0/md/sync_action

# Reprendre la reconstruction
echo check > /sys/block/md0/md/sync_action  # Pour check
echo repair > /sys/block/md0/md/sync_action  # Pour repair
```

> [!warning] Attention à l'interruption
> 
> - Ne jamais éteindre le serveur pendant une reconstruction
> - L'interruption d'une reconstruction laisse l'array en état dégradé
> - Si nécessaire, utilisez `idle` pour mettre en pause proprement

---

## 👁️ Surveillance avec mdadm --monitor

### Principe de la surveillance

La surveillance active permet de détecter rapidement les problèmes et de recevoir des alertes en cas de défaillance. `mdadm` peut fonctionner en mode daemon pour surveiller continuellement les arrays.

> [!info] Pourquoi surveiller ?
> 
> - Détection précoce des pannes
> - Notification automatique par email
> - Anticipation des problèmes (disques lents, erreurs)
> - Automatisation de certaines réponses

### Lancer mdadm en mode surveillance

```bash
# Mode surveillance simple (premier plan)
mdadm --monitor /dev/md0

# Mode surveillance avec options
mdadm --monitor --scan --daemonise --mail=admin@example.com --delay=1800

# Explication des options :
# --scan        : Surveiller tous les arrays dans /proc/mdstat
# --daemonise   : Lancer en arrière-plan (daemon)
# --mail        : Adresse email pour les alertes
# --delay=1800  : Vérifier toutes les 1800 secondes (30 minutes)
```

### Configuration dans /etc/mdadm/mdadm.conf

La configuration permanente se fait dans le fichier de configuration.

```bash
# Éditer le fichier de configuration
nano /etc/mdadm/mdadm.conf

# Ajouter ou modifier les lignes suivantes :
MAILADDR admin@example.com
MAILFROM raid-monitor@server.local
```

> [!example] Configuration complète de mdadm.conf
> 
> ```bash
> # Arrays à surveiller (généré avec --detail --scan)
> ARRAY /dev/md0 metadata=1.2 name=server:0 UUID=12345678:abcdefab:12345678:abcdefab
> ARRAY /dev/md1 metadata=1.2 name=server:1 UUID=87654321:bafedcba:87654321:bafedcba
> 
> # Configuration des alertes
> MAILADDR admin@example.com
> MAILFROM raid-alert@server.local
> 
> # Programme personnalisé à exécuter en cas d'événement
> PROGRAM /usr/local/bin/raid-alert.sh
> ```

### Service systemd pour la surveillance

Sur les systèmes modernes, mdadm utilise systemd pour la surveillance.

```bash
# Vérifier le statut du service
systemctl status mdmonitor

# Activer le service au démarrage
systemctl enable mdmonitor

# Démarrer le service
systemctl start mdmonitor

# Voir les logs de surveillance
journalctl -u mdmonitor -f
```

> [!tip] Fichier de service personnalisé
> 
> ```bash
> # Voir le fichier de service
> systemctl cat mdmonitor
> 
> # Personnaliser si nécessaire
> systemctl edit mdmonitor
> # Ajouter : Environment="MDADM_MONITOR_ARGS=--delay=900"
> ```

### Options avancées de surveillance

```bash
# Test de l'envoi d'email
mdadm --monitor --scan --test --oneshot

# Surveillance avec seuil personnalisé
mdadm --monitor /dev/md0 --delay=600 --threshold=2

# Script personnalisé pour les événements
mdadm --monitor --scan --program=/usr/local/bin/custom-alert.sh
```

### Événements surveillés

|Événement|Description|Action typique|
|---|---|---|
|**DeviceDisappeared**|Un disque a disparu|Email + investigation|
|**RebuildStarted**|Début de reconstruction|Email informatif|
|**RebuildFinished**|Fin de reconstruction|Email confirmation|
|**Fail**|Disque marqué défaillant|Email urgent|
|**DegradedArray**|Array en mode dégradé|Email urgent + action|
|**SpareActive**|Un spare est activé|Email informatif|

> [!example] Script d'alerte personnalisé
> 
> ```bash
> #!/bin/bash
> # /usr/local/bin/raid-alert.sh
> 
> EVENT=$1
> DEVICE=$2
> 
> case $EVENT in
>     DeviceDisappeared|Fail|DegradedArray)
>         # Alerte critique
>         logger -p user.crit "RAID CRITICAL: $EVENT on $DEVICE"
>         # Envoi SMS via API (exemple)
>         curl -X POST "https://api.sms.com/send" \
>              -d "message=RAID Alert: $EVENT on $DEVICE"
>         ;;
>     RebuildFinished)
>         # Notification normale
>         logger -p user.info "RAID INFO: $EVENT on $DEVICE"
>         ;;
> esac
> ```

### Vérification manuelle de l'état

```bash
# État détaillé de tous les arrays
mdadm --detail --scan

# Vérifier uniquement les arrays en problème
grep -H . /sys/block/md*/md/degraded
# Retourne 1 si dégradé, 0 si OK

# Statistiques de santé
cat /sys/block/md0/md/dev-sda1/errors
```

> [!warning] Monitoring en production
> 
> - Configurez toujours des alertes email
> - Testez régulièrement la réception des alertes
> - Documentez la procédure de réponse aux alertes
> - Intégrez RAID dans votre système de monitoring global (Nagios, Zabbix, etc.)

---

## 🛑 Arrêt et suppression d'un array

### Différence entre arrêt et suppression

|Action|Commande|Effet|Réversible ?|
|---|---|---|---|
|**Arrêt**|`--stop`|Désactive l'array temporairement|Oui (avec `--assemble`)|
|**Suppression**|`--remove`|Retire un disque de l'array|Dépend du contexte|
|**Destruction**|`--zero-superblock`|Efface les métadonnées RAID|Non (perte définitive)|

### Arrêt d'un array (--stop)

L'arrêt désactive un array mais conserve toutes les métadonnées. Utilisé pour la maintenance ou avant extinction du serveur.

```bash
# Démonter le système de fichiers
umount /mnt/raid

# Vérifier qu'aucun processus n'utilise l'array
lsof | grep /dev/md0
fuser -m /dev/md0

# Arrêter l'array
mdadm --stop /dev/md0

# Vérifier l'arrêt
cat /proc/mdstat
# md0 ne devrait plus apparaître
```

> [!warning] Prérequis avant arrêt
> 
> - Le système de fichiers **doit** être démonté
> - Aucun processus ne doit utiliser les périphériques
> - L'array ne doit pas être en cours de reconstruction

### Réactiver un array arrêté

```bash
# Réassembler l'array
mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1 /dev/sdd1

# Alternative : scan automatique
mdadm --assemble --scan

# Remonter le système de fichiers
mount /dev/md0 /mnt/raid

# Vérifier l'état
mdadm --detail /dev/md0
```

> [!tip] Assemblage automatique au démarrage Les arrays configurés dans `/etc/mdadm/mdadm.conf` sont automatiquement réassemblés au boot.

### Suppression complète d'un array

La suppression complète implique l'arrêt ET l'effacement des métadonnées RAID sur tous les disques.

```bash
# Étape 1 : Sauvegarder les données importantes !
# (Si nécessaire)

# Étape 2 : Démonter
umount /mnt/raid

# Étape 3 : Arrêter l'array
mdadm --stop /dev/md0

# Étape 4 : Effacer les superblocks de chaque disque
mdadm --zero-superblock /dev/sdb1
mdadm --zero-superblock /dev/sdc1
mdadm --zero-superblock /dev/sdd1

# Étape 5 : Supprimer la configuration de /etc/mdadm/mdadm.conf
sed -i '/md0/d' /etc/mdadm/mdadm.conf

# Étape 6 : Supprimer de /etc/fstab
sed -i '/\/dev\/md0/d' /etc/fstab

# Étape 7 : Mettre à jour initramfs
update-initramfs -u
```

> [!example] Script complet de suppression
> 
> ```bash
> #!/bin/bash
> # Suppression sécurisée d'un array RAID
> 
> ARRAY=$1
> 
> if [ -z "$ARRAY" ]; then
>     echo "Usage: $0 /dev/mdX"
>     exit 1
> fi
> 
> # Récupérer les disques membres
> DISKS=$(mdadm --detail $ARRAY | grep active | awk '{print $7}')
> 
> echo "Array: $ARRAY"
> echo "Disques: $DISKS"
> read -p "Confirmer la suppression ? (yes/no): " CONFIRM
> 
> if [ "$CONFIRM" != "yes" ]; then
>     echo "Annulé."
>     exit 0
> fi
> 
> # Démonter
> umount $ARRAY 2>/dev/null
> 
> # Arrêter
> mdadm --stop $ARRAY
> 
> # Effacer les superblocks
> for disk in $DISKS; do
>     echo "Effacement de $disk..."
>     mdadm --zero-superblock $disk
> done
> 
> echo "Array supprimé avec succès."
> ```

### Suppression d'un disque spécifique (--remove)

```bash
# Marquer le disque comme défaillant d'abord
mdadm --fail /dev/md0 /dev/sdc1

# Retirer le disque de l'array
mdadm --remove /dev/md0 /dev/sdc1

# Le disque peut maintenant être réutilisé ailleurs
mdadm --zero-superblock /dev/sdc1
```

### Cas particulier : Array qui refuse de s'arrêter

```bash
# Identifier ce qui bloque
lsof /dev/md0
fuser -mv /dev/md0

# Forcer l'arrêt des processus (avec précaution)
fuser -km /dev/md0

# Réessayer l'arrêt
mdadm --stop /dev/md0

# En dernier recours (NON RECOMMANDÉ en production)
mdadm --stop /dev/md0 --force
```

> [!warning] Force stop : dangers
> 
> - Risque de corruption de données
> - À utiliser uniquement si l'array est déjà corrompu
> - Préférez toujours identifier et résoudre la cause du blocage

### Nettoyage après suppression

```bash
# Supprimer les fichiers de périphérique (si nécessaire)
rm -f /dev/md0

# Vérifier qu'il ne reste pas de métadonnées
mdadm --examine /dev/sdb1
# Devrait retourner une erreur

# Réinitialiser complètement les disques (optionnel)
wipefs -a /dev/sdb1
wipefs -a /dev/sdc1
wipefs -a /dev/sdd1
```

> [!tip] Réutilisation des disques Après `--zero-superblock` et `wipefs`, les disques sont prêts à être :
> 
> - Utilisés dans un nouveau RAID
> - Formatés pour un usage normal
> - Intégrés à un LVM

### Vérification finale

```bash
# S'assurer qu'aucun array n'est actif
cat /proc/mdstat
# Devrait montrer "Personalities" sans arrays actifs

# Vérifier mdadm.conf
cat /etc/mdadm/mdadm.conf | grep -v "^#"

# Vérifier fstab
grep "/dev/md" /etc/fstab
# Ne devrait rien retourner
```

---

## 📋 Récapitulatif des commandes essentielles

|Action|Commande|
|---|---|
|Ajouter un disque|`mdadm --add /dev/md0 /dev/sdc1`|
|Marquer défaillant|`mdadm --fail /dev/md0 /dev/sdc1`|
|Retirer un disque|`mdadm --remove /dev/md0 /dev/sdc1`|
|Surveiller|`mdadm --monitor --scan --daemonise`|
|Vérifier état|`cat /proc/mdstat`|
|Détails array|`mdadm --detail /dev/md0`|
|Arrêter array|`mdadm --stop /dev/md0`|
|Effacer superblock|`mdadm --zero-superblock /dev/sdc1`|
|Vitesse rebuild|`echo 100000 > /proc/sys/dev/raid/speed_limit_max`|

> [!info] Bonne pratique finale
> 
> - Testez toujours vos procédures de remplacement en environnement de test
> - Documentez les configurations spécifiques de vos arrays
> - Planifiez des vérifications périodiques automatiques
> - Maintenez des spares disponibles pour un remplacement rapide
> - Surveillez les logs système régulièrement