# RAID sous Linux - Cheatsheet TSSR
> CCP 8 — Sauvegardes & restaurations / CCP 3 — Exploiter des serveurs Linux

## Concepts clés

| Concept | Valeur / Description |
|--------|-------------------|
| RAID 0 | Striping — performance, aucune redondance |
| RAID 1 | Mirroring — redondance totale, 50% espace utile |
| RAID 5 | Striping + parité distribuée — min. 3 disques, 1 disque de perte toléré |
| RAID 6 | Striping + double parité — min. 4 disques, 2 disques de perte tolérés |
| RAID 10 | RAID 1+0 — min. 4 disques, performance + redondance |
| mdadm | Outil Linux de gestion du RAID logiciel |
| `/dev/mdX` | Nom du périphérique RAID (ex : `/dev/md0`) |
| Chunk size | Taille du bloc de striping (défaut : 512 Ko) |

## Commandes principales

| Action | Commande |
|--------|----------|
| Créer un RAID 1 | `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc` |
| Créer un RAID 5 | `mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd` |
| Afficher l'état du RAID | `mdadm --detail /dev/md0` |
| État rapide de tous les RAID | `cat /proc/mdstat` |
| Arrêter un RAID | `mdadm --stop /dev/md0` |
| Supprimer un RAID | `mdadm --zero-superblock /dev/sdb` |
| Ajouter un disque de spare | `mdadm --add /dev/md0 /dev/sde` |
| Retirer un disque | `mdadm --remove /dev/md0 /dev/sdb` |
| Marquer un disque défaillant | `mdadm --fail /dev/md0 /dev/sdb` |
| Lancer une reconstruction | `mdadm --assemble /dev/md0 /dev/sdb /dev/sdc` |
| Sauvegarder la config | `mdadm --detail --scan >> /etc/mdadm/mdadm.conf` |
| Formater le RAID | `mkfs.ext4 /dev/md0` |
| Monter le RAID | `mount /dev/md0 /mnt/raid` |

## Fichiers importants

| Fichier | Contenu |
|---------|---------|
| `/etc/mdadm/mdadm.conf` | Configuration des arrays RAID |
| `/proc/mdstat` | État en temps réel des RAID actifs |
| `/etc/fstab` | Montage automatique au démarrage |

## Points de vigilance

| Risque / Erreur | Explication |
|-----------------|-------------|
| RAID ≠ sauvegarde | Un RAID ne protège pas contre suppression ou corruption logique |
| Rebuild lent | La reconstruction consomme des I/O — prévoir une fenêtre de maintenance |
| Oublier `mdadm.conf` | Sans sauvegarde de config, le RAID ne se réassemble pas au boot |
| RAID 0 = zéro tolérance | Perte d'un disque = perte totale des données |
| Surveiller `/proc/mdstat` | Intégrer dans la supervision (Nagios, Zabbix…) |
| Taille utile RAID 5 | `(N-1) × taille_disque` |
| Taille utile RAID 6 | `(N-2) × taille_disque` |
