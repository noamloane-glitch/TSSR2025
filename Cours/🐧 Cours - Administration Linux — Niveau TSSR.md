# 🐧 Cours : Administration Linux — Niveau TSSR

> **Objectif** : Maîtriser les commandes essentielles, la gestion des utilisateurs, des droits, des services et du réseau sous Linux pour administrer un serveur en environnement professionnel.

---

## PARTIE 1 — Les bases de Linux

---

## 1. L'arborescence Linux — à connaître absolument

Sous Linux, tout part d'une racine unique `/`. Pas de `C:\` ou `D:\` comme sous Windows.

```
/
├── bin/        → Commandes essentielles (ls, cp, mv...)
├── sbin/       → Commandes système réservées à root (fdisk, ifconfig...)
├── etc/        → Fichiers de configuration du système
├── home/       → Répertoires personnels des utilisateurs (/home/jean/)
├── root/       → Répertoire personnel de l'utilisateur root
├── var/        → Données variables (logs, bases de données, mails...)
│   └── log/    → Fichiers de logs système
├── tmp/        → Fichiers temporaires (vidé au redémarrage)
├── usr/        → Applications et bibliothèques utilisateur
├── opt/        → Logiciels tiers installés manuellement
├── dev/        → Fichiers représentant les périphériques (disques, USB...)
├── proc/       → Système de fichiers virtuel — état du noyau en temps réel
├── sys/        → Système de fichiers virtuel — paramètres du noyau
├── mnt/        → Points de montage temporaires
├── media/      → Montage automatique des médias (USB, CD...)
└── boot/       → Fichiers de démarrage (noyau, GRUB...)
```

> 💡 **À retenir** : La configuration est dans `/etc`, les logs dans `/var/log`, les données utilisateurs dans `/home`. Ce sont les trois dossiers les plus utiles au quotidien.

---

## 2. Les commandes fondamentales — navigation et fichiers

### Navigation

```bash
pwd                     # Afficher le répertoire courant
ls                      # Lister le contenu d'un dossier
ls -la                  # Liste détaillée avec fichiers cachés
cd /etc                 # Aller dans /etc
cd ..                   # Remonter d'un niveau
cd ~                    # Aller dans son répertoire personnel
cd -                    # Revenir au répertoire précédent
```

### Gestion de fichiers et dossiers

```bash
mkdir mon_dossier               # Créer un dossier
mkdir -p /opt/app/config        # Créer les dossiers parents si nécessaire
touch fichier.txt               # Créer un fichier vide
cp source.txt dest.txt          # Copier un fichier
cp -r dossier/ /opt/backup/     # Copier un dossier récursivement
mv ancien.txt nouveau.txt       # Déplacer / renommer
rm fichier.txt                  # Supprimer un fichier
rm -rf dossier/                 # Supprimer un dossier et son contenu (⚠️ irréversible)
```

> ⚠️ `rm -rf /` supprime tout le système. Ne jamais l'exécuter. Certaines distributions bloquent cette commande, d'autres non.

### Lecture de fichiers

```bash
cat fichier.txt                 # Afficher tout le fichier
less fichier.txt                # Afficher page par page (q pour quitter)
head -n 20 fichier.txt          # Afficher les 20 premières lignes
tail -n 20 fichier.txt          # Afficher les 20 dernières lignes
tail -f /var/log/syslog         # Suivre un fichier en temps réel (logs)
grep "erreur" fichier.txt       # Chercher "erreur" dans le fichier
grep -r "erreur" /var/log/      # Chercher récursivement dans un dossier
```

### Recherche

```bash
find / -name "fichier.txt"              # Chercher un fichier par nom
find /etc -name "*.conf"                # Chercher tous les .conf dans /etc
find / -type f -size +100M              # Fichiers de plus de 100 Mo
locate fichier.txt                      # Chercher dans la base de données (plus rapide)
which python3                           # Trouver où est installée une commande
```

---

## 3. Les flux et la redirection

```bash
commande > fichier.txt          # Rediriger la sortie vers un fichier (écrase)
commande >> fichier.txt         # Rediriger en ajoutant à la fin
commande 2> erreurs.txt         # Rediriger les erreurs
commande 2>&1                   # Rediriger les erreurs vers la sortie standard
commande1 | commande2           # Pipe : sortie de cmd1 = entrée de cmd2
```

**Exemples pratiques** :

```bash
ls -la /etc | grep ".conf"              # Lister uniquement les .conf
cat /var/log/syslog | grep "error"      # Filtrer les erreurs dans les logs
ps aux | grep apache                    # Chercher le processus apache
df -h | grep "/$"                       # Voir l'occupation de la partition racine
```

---

## PARTIE 2 — Utilisateurs et droits

---

## 4. Gestion des utilisateurs

### Créer, modifier, supprimer

```bash
useradd jean                            # Créer l'utilisateur jean
useradd -m -s /bin/bash jean            # Avec répertoire home et shell bash
useradd -G sudo,www-data jean           # Avec groupes supplémentaires
passwd jean                             # Définir/changer le mot de passe de jean
usermod -aG sudo jean                   # Ajouter jean au groupe sudo (sans le retirer des autres)
usermod -s /bin/bash jean               # Changer le shell de jean
userdel jean                            # Supprimer jean (garde le home)
userdel -r jean                         # Supprimer jean ET son répertoire home
```

### Informations sur les utilisateurs

```bash
id jean                                 # Voir l'UID, GID et groupes de jean
whoami                                  # Afficher l'utilisateur courant
who                                     # Voir les utilisateurs connectés
w                                       # Voir les utilisateurs connectés + ce qu'ils font
last                                    # Historique des connexions
cat /etc/passwd                         # Liste de tous les utilisateurs
cat /etc/shadow                         # Mots de passe hashés (root uniquement)
cat /etc/group                          # Liste des groupes
```

### Structure de /etc/passwd

```
jean:x:1001:1001:Jean Dupont:/home/jean:/bin/bash
 │   │  │    │       │            │          └── Shell
 │   │  │    │       │            └── Répertoire home
 │   │  │    │       └── Commentaire (nom complet)
 │   │  │    └── GID (groupe principal)
 │   │  └── UID (identifiant utilisateur)
 │   └── x = mot de passe dans /etc/shadow
 └── Nom d'utilisateur
```

---

## 5. Les droits Linux — lecture indispensable

### Comprendre les permissions

```bash
ls -la /home/jean/
# drwxr-xr-x  2 jean jean 4096 jan 15 10:00 Documents
# │││││││││
# │││└└└└└└── Droits des autres (others)
# │││   └└└── Droits du groupe
# │└└└──────── Droits du propriétaire (user)
# └──────────── Type : d=dossier, -=fichier, l=lien
```

### Les trois droits

|Droit|Lettre|Valeur numérique|Sur fichier|Sur dossier|
|---|---|---|---|---|
|Read|r|4|Lire le fichier|Lister le contenu|
|Write|w|2|Modifier le fichier|Créer/supprimer des fichiers|
|Execute|x|1|Exécuter le fichier|Traverser le dossier|

### Calculer les droits en octal

```
rwxr-xr-x
│││ │││ │││
│││ │││ └└└── Others : r-x = 4+0+1 = 5
│││ └└└────── Group  : r-x = 4+0+1 = 5
└└└────────── User   : rwx = 4+2+1 = 7
→ chmod 755
```

**Tableau de référence** :

|Octal|Lettres|Signification|
|---|---|---|
|777|rwxrwxrwx|Tout le monde peut tout faire (dangereux)|
|755|rwxr-xr-x|Propriétaire tout, autres lecture+exécution|
|644|rw-r--r--|Propriétaire lecture+écriture, autres lecture|
|600|rw-------|Propriétaire uniquement (clés SSH, fichiers sensibles)|
|700|rwx------|Propriétaire uniquement avec exécution|

### Changer les droits et le propriétaire

```bash
chmod 755 script.sh                     # Changer les droits en octal
chmod +x script.sh                      # Ajouter le droit d'exécution à tous
chmod u+x,g-w script.sh                 # Ajouter x au user, retirer w au groupe
chmod -R 644 /var/www/html/             # Appliquer récursivement

chown jean fichier.txt                  # Changer le propriétaire
chown jean:web fichier.txt              # Changer propriétaire ET groupe
chown -R jean:jean /home/jean/          # Récursivement
```

### Les droits spéciaux

|Droit|Description|Usage|
|---|---|---|
|**SUID** (4xxx)|Le fichier s'exécute avec les droits du propriétaire|`passwd` — s'exécute en root même lancé par jean|
|**SGID** (2xxx)|Fichiers créés dans le dossier héritent du groupe|Dossiers partagés entre équipes|
|**Sticky bit** (1xxx)|Seul le propriétaire peut supprimer son fichier|`/tmp` — chacun peut créer mais pas supprimer les fichiers des autres|

```bash
chmod u+s script.sh                     # Ajouter SUID
chmod g+s dossier_partage/              # Ajouter SGID
chmod +t /tmp                           # Ajouter sticky bit
```

---

## PARTIE 3 — Gestion des services

---

## 6. Systemd — le gestionnaire de services

La quasi-totalité des distributions Linux modernes utilisent **systemd** pour gérer les services.

### Commandes essentielles

```bash
systemctl start nginx                   # Démarrer nginx
systemctl stop nginx                    # Arrêter nginx
systemctl restart nginx                 # Redémarrer nginx
systemctl reload nginx                  # Recharger la config sans couper le service
systemctl status nginx                  # Voir l'état du service
systemctl enable nginx                  # Activer au démarrage
systemctl disable nginx                 # Désactiver au démarrage
systemctl is-active nginx               # Vérifier si le service est actif
systemctl is-enabled nginx              # Vérifier si le service démarre au boot

systemctl list-units --type=service     # Lister tous les services
systemctl list-units --failed           # Lister les services en échec
```

### Voir les logs d'un service

```bash
journalctl -u nginx                     # Logs du service nginx
journalctl -u nginx -f                  # Suivre en temps réel
journalctl -u nginx --since "1 hour ago"  # Logs de la dernière heure
journalctl -u nginx -n 50               # 50 dernières lignes
journalctl -p err                       # Uniquement les erreurs
```

---

## 7. Les processus

```bash
ps aux                                  # Lister tous les processus
ps aux | grep apache                    # Chercher le processus apache
top                                     # Moniteur de processus interactif
htop                                    # Version améliorée de top (si installé)
kill 1234                               # Envoyer SIGTERM (arrêt propre) au PID 1234
kill -9 1234                            # Envoyer SIGKILL (arrêt forcé) au PID 1234
killall nginx                           # Tuer tous les processus nommés nginx
pgrep nginx                             # Trouver le PID de nginx
pkill nginx                             # Tuer par nom
nice -n 10 commande                     # Lancer une commande avec une priorité basse
```

---

## PARTIE 4 — Réseau sous Linux

---

## 8. Configuration réseau

### Voir la configuration actuelle

```bash
ip a                                    # Voir les interfaces et adresses IP
ip link show                            # Voir l'état des interfaces
ip route show                           # Voir la table de routage
ip route show default                   # Voir la passerelle par défaut
ss -tuln                                # Voir les ports en écoute
ss -tunap                               # Voir les connexions avec PID
cat /etc/resolv.conf                    # Voir les serveurs DNS configurés
```

### Configurer une IP temporaire (non persistante)

```bash
ip addr add 192.168.1.50/24 dev eth0    # Ajouter une IP
ip addr del 192.168.1.50/24 dev eth0    # Supprimer une IP
ip link set eth0 up                     # Activer l'interface
ip link set eth0 down                   # Désactiver l'interface
ip route add default via 192.168.1.1    # Ajouter une route par défaut
```

### Configuration persistante — Netplan (Ubuntu 20.04+)

```yaml
# /etc/netplan/00-config.yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.10
          - 8.8.8.8
```

```bash
netplan apply                           # Appliquer la configuration Netplan
```

### Outils de diagnostic réseau

```bash
ping 8.8.8.8                            # Test de connectivité IP
ping -c 4 google.com                    # 4 pings vers google.com
traceroute 8.8.8.8                      # Tracer le chemin des paquets
tracepath 8.8.8.8                       # Alternative à traceroute
nslookup google.com                     # Résolution DNS
dig google.com                          # Résolution DNS détaillée
dig google.com MX                       # Enregistrement MX
curl -v https://google.com              # Test HTTP complet
wget https://example.com/fichier.zip    # Télécharger un fichier
netstat -tuln                           # Ports en écoute (ancienne méthode)
```

---

## 9. Le pare-feu sous Linux — UFW et iptables

### UFW (Uncomplicated Firewall) — Ubuntu/Debian

```bash
ufw enable                              # Activer le pare-feu
ufw disable                             # Désactiver le pare-feu
ufw status verbose                      # Voir l'état et les règles
ufw allow 22                            # Autoriser SSH
ufw allow 80/tcp                        # Autoriser HTTP
ufw allow 443/tcp                       # Autoriser HTTPS
ufw deny 23                             # Bloquer Telnet
ufw allow from 192.168.1.0/24           # Autoriser tout le sous-réseau local
ufw allow from 192.168.1.10 to any port 22  # SSH depuis une IP précise seulement
ufw delete allow 80/tcp                 # Supprimer une règle
ufw reset                               # Remettre à zéro
```

### iptables — bases

```bash
iptables -L -n -v                       # Lister toutes les règles
iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # Autoriser SSH entrant
iptables -A INPUT -p tcp --dport 80 -j ACCEPT      # Autoriser HTTP
iptables -A INPUT -j DROP              # Bloquer tout le reste (en dernier)
iptables -D INPUT -p tcp --dport 80 -j ACCEPT      # Supprimer une règle
iptables-save > /etc/iptables/rules.v4  # Sauvegarder les règles
iptables-restore < /etc/iptables/rules.v4  # Restaurer les règles
```

---

## PARTIE 5 — Stockage et système de fichiers

---

## 10. Gestion des disques et partitions

```bash
lsblk                                   # Lister les disques et partitions
fdisk -l                                # Détail des disques (root requis)
df -h                                   # Espace utilisé par partition
du -sh /var/log/                        # Taille du dossier /var/log
du -sh /* | sort -rh | head -10         # Les 10 plus gros dossiers à la racine

mount /dev/sdb1 /mnt/data              # Monter une partition
umount /mnt/data                        # Démonter une partition
```

### /etc/fstab — montage automatique au démarrage

```bash
# /etc/fstab
# Device          Point de montage  Système  Options  dump  pass
/dev/sda1         /                 ext4     defaults  0     1
/dev/sdb1         /data             ext4     defaults  0     2
UUID=xxxx-xxxx    /backup           ext4     defaults  0     2
```

```bash
mount -a                                # Monter tout ce qui est dans fstab
```

---

## PARTIE 6 — Gestion des paquets

---

## 11. APT — Debian / Ubuntu

```bash
apt update                              # Mettre à jour la liste des paquets
apt upgrade                             # Mettre à jour les paquets installés
apt full-upgrade                        # Mise à jour complète avec résolution de dépendances
apt install nginx                       # Installer nginx
apt install -y nginx                    # Installer sans confirmation
apt remove nginx                        # Désinstaller (garde la config)
apt purge nginx                         # Désinstaller + supprimer la config
apt autoremove                          # Supprimer les paquets inutiles
apt search nginx                        # Chercher un paquet
apt show nginx                          # Informations sur un paquet
dpkg -l | grep nginx                    # Vérifier si un paquet est installé
```

### YUM / DNF — CentOS / RHEL / Fedora

```bash
dnf update                              # Mettre à jour
dnf install httpd                       # Installer Apache
dnf remove httpd                        # Désinstaller
dnf search httpd                        # Chercher un paquet
rpm -qa | grep httpd                    # Lister les paquets installés contenant "httpd"
```

---

## PARTIE 7 — Scripting Bash

---

## 12. Les bases du scripting Bash

### Structure d'un script

```bash
#!/bin/bash
# Commentaire
# Ce script fait...

echo "Début du script"

# Variable
NOM="Jean"
echo "Bonjour $NOM"

# Variable spéciales
echo "Nom du script : $0"
echo "Premier argument : $1"
echo "Nombre d'arguments : $#"
```

### Conditions

```bash
#!/bin/bash

# if/else
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "nginx est installé"
else
    echo "nginx n'est pas installé"
fi

# Comparaisons numériques
if [ $1 -gt 10 ]; then
    echo "$1 est supérieur à 10"
fi

# Comparaisons de chaînes
if [ "$NOM" = "Jean" ]; then
    echo "Bonjour Jean"
fi
```

**Opérateurs de test** :

|Test fichier|Signification|
|---|---|
|`-f fichier`|Le fichier existe et est un fichier régulier|
|`-d dossier`|Le dossier existe|
|`-e chemin`|Le chemin existe (fichier ou dossier)|
|`-r fichier`|Le fichier est lisible|
|`-x fichier`|Le fichier est exécutable|

|Test numérique|Signification|
|---|---|
|`-eq`|Égal|
|`-ne`|Différent|
|`-gt`|Supérieur|
|`-lt`|Inférieur|
|`-ge`|Supérieur ou égal|

### Boucles

```bash
# Boucle for
for i in 1 2 3 4 5; do
    echo "Itération $i"
done

# Boucle for sur des fichiers
for fichier in /etc/*.conf; do
    echo "Config trouvée : $fichier"
done

# Boucle while
COMPTEUR=0
while [ $COMPTEUR -lt 5 ]; do
    echo "Compteur : $COMPTEUR"
    COMPTEUR=$((COMPTEUR + 1))
done
```

### Exemple de script utile — sauvegarde

```bash
#!/bin/bash
# Script de sauvegarde simple

DATE=$(date +%Y%m%d_%H%M%S)
SOURCE="/var/www/html"
DESTINATION="/backup/web_$DATE.tar.gz"

echo "Début de la sauvegarde : $DATE"

if [ -d "$SOURCE" ]; then
    tar -czf "$DESTINATION" "$SOURCE"
    echo "Sauvegarde créée : $DESTINATION"
else
    echo "Erreur : le dossier source $SOURCE n'existe pas"
    exit 1
fi

echo "Sauvegarde terminée."
```

---

## PARTIE 8 — Logs et supervision

---

## 13. Les logs Linux

|Fichier|Contenu|
|---|---|
|`/var/log/syslog`|Logs système généraux (Debian/Ubuntu)|
|`/var/log/messages`|Logs système généraux (CentOS/RHEL)|
|`/var/log/auth.log`|Authentifications, sudo, SSH|
|`/var/log/kern.log`|Logs du noyau|
|`/var/log/nginx/access.log`|Accès HTTP nginx|
|`/var/log/nginx/error.log`|Erreurs nginx|
|`/var/log/apache2/access.log`|Accès HTTP Apache|
|`/var/log/dpkg.log`|Installations de paquets (Debian)|

```bash
tail -f /var/log/syslog                 # Suivre les logs en temps réel
grep "Failed password" /var/log/auth.log  # Chercher les échecs SSH
grep "error" /var/log/nginx/error.log   # Chercher les erreurs nginx
journalctl -xe                          # Logs systemd avec erreurs récentes
journalctl --since "2024-01-15 08:00"   # Logs depuis une date précise
```

---

## PARTIE 9 — Scénarios de dépannage

---

### 🔴 Scénario 1 — Un service ne démarre pas

**Situation** :

- Après une mise à jour, nginx ne démarre plus
- `systemctl start nginx` → échoue sans message clair

**Questions à se poser** :

1. Quel est le message d'erreur exact dans les logs ?
2. Y a-t-il une erreur de syntaxe dans la configuration ?
3. Un autre processus occupe-t-il le port ?
4. Les droits sur les fichiers de config sont-ils corrects ?

**Marche à suivre** :

```bash
systemctl status nginx                  # 1. Message d'erreur systemd
journalctl -u nginx -n 50               # 2. Logs détaillés du service
nginx -t                                # 3. Tester la syntaxe de la configuration
ss -tuln | grep :80                     # 4. Le port 80 est-il déjà utilisé ?
ps aux | grep nginx                     # 5. Un processus nginx zombie tourne-t-il ?
```

> 💡 **Réflexe** : `nginx -t` (ou `apache2 -t`) avant tout redémarrage. Ça vérifie la config et évite de couper un service fonctionnel avec une config cassée.

---

### 🔴 Scénario 2 — Disque plein sur le serveur

**Situation** :

- Des alertes remontent : disque à 100%
- Les applications commencent à dysfonctionner
- Les logs ne s'écrivent plus

**Questions à se poser** :

1. Quelle partition est pleine ?
2. Quel dossier consomme le plus d'espace ?
3. S'agit-il de logs, de fichiers temporaires, ou de données applicatives ?

**Marche à suivre** :

```bash
df -h                                   # 1. Voir quelle partition est pleine
du -sh /* 2>/dev/null | sort -rh | head -10  # 2. Trouver les plus gros dossiers
du -sh /var/log/* | sort -rh | head -10 # 3. Logs — souvent le coupable
find /var/log -name "*.log" -size +100M # 4. Trouver les gros fichiers de log

# Solutions rapides
> /var/log/syslog                       # Vider un fichier de log (sans le supprimer)
find /tmp -mtime +7 -delete             # Supprimer les fichiers de +7 jours dans /tmp
journalctl --vacuum-size=500M           # Limiter les logs systemd à 500 Mo
```

> 💡 **Bonne pratique** : Configurer **logrotate** pour que les logs soient automatiquement archivés et nettoyés. La configuration est dans `/etc/logrotate.d/`.

---

### 🔴 Scénario 3 — Impossible de se connecter en SSH

**Situation** :

- Un admin tente de se connecter en SSH à un serveur
- La connexion expire (timeout) ou est refusée
- Le serveur est pourtant pingable

**Questions à se poser** :

1. Le service SSH est-il démarré sur le serveur ?
2. Le port SSH (22 ou autre) est-il ouvert dans le firewall ?
3. L'IP source est-elle autorisée (fail2ban, règles UFW) ?
4. L'utilisateur est-il autorisé à se connecter en SSH ?

**Marche à suivre** :

```bash
# Sur le serveur (si accès console disponible)
systemctl status ssh                    # SSH est-il démarré ?
ss -tuln | grep :22                     # Le port 22 écoute-t-il ?
ufw status                              # Le firewall bloque-t-il ?
cat /var/log/auth.log | grep ssh        # Voir les tentatives de connexion

# Vérifier fail2ban
fail2ban-client status sshd             # L'IP est-elle bannie ?
fail2ban-client unban 192.168.1.50      # Débloquer une IP

# Vérifier la config SSH
cat /etc/ssh/sshd_config | grep -E "PermitRootLogin|PasswordAuthentication|Port"
```

> 💡 **Cause classique** : fail2ban a banni l'IP après plusieurs tentatives de connexion échouées. Vérifier toujours fail2ban avant de paniquer.

---

### 🔴 Scénario 4 — Un utilisateur ne peut pas exécuter une commande

**Situation** :

- L'utilisateur `jean` tente d'exécuter `systemctl restart nginx`
- Message : "Permission denied" ou "jean is not in the sudoers file"
- L'admin veut donner à jean uniquement le droit de redémarrer nginx, pas les droits root complets

**Questions à se poser** :

1. Jean est-il dans le groupe sudo ?
2. Faut-il lui donner tous les droits sudo ou uniquement sur nginx ?
3. Comment configurer sudo de façon granulaire ?

**Marche à suivre** :

```bash
# Option 1 : Ajouter jean au groupe sudo (droits root complets — attention !)
usermod -aG sudo jean

# Option 2 : Droits sudo granulaires via sudoers
visudo                                  # Toujours éditer sudoers avec visudo

# Dans le fichier sudoers, ajouter :
jean ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
jean ALL=(ALL) NOPASSWD: /bin/systemctl stop nginx
jean ALL=(ALL) NOPASSWD: /bin/systemctl start nginx
```

> ⚠️ **Toujours utiliser `visudo`** pour éditer `/etc/sudoers`. Cette commande vérifie la syntaxe avant d'enregistrer. Une erreur dans sudoers peut bloquer tous les accès sudo.

---

### 🔴 Scénario 5 — Le serveur est très lent

**Situation** :

- Les utilisateurs signalent des lenteurs sur une application hébergée sur le serveur
- Le serveur est accessible mais répond très lentement
- Aucune alerte réseau

**Questions à se poser** :

1. Le CPU est-il saturé ?
2. La RAM est-elle épuisée (swap utilisé) ?
3. Le disque est-il en I/O wait élevé ?
4. Un processus particulier consomme-t-il toutes les ressources ?

**Marche à suivre** :

```bash
top                                     # Vue globale CPU, RAM, processus
# Dans top : trier par CPU → touche P | trier par RAM → touche M

htop                                    # Version colorée de top (si installé)

free -h                                 # Voir la RAM et le swap utilisés
vmstat 1 5                              # Statistiques système toutes les secondes

iostat -x 1 5                           # I/O disque (si sysstat installé)
iotop                                   # Voir quel processus utilise le disque

# Si un processus consomme tout
ps aux --sort=-%cpu | head -10          # Top 10 processus par CPU
ps aux --sort=-%mem | head -10          # Top 10 processus par RAM
kill -9 <PID>                           # Tuer le processus si nécessaire
```

> 💡 **Swap utilisé = danger** : quand la RAM est pleine, Linux utilise le swap (disque). C'est des dizaines de fois plus lent. Si le swap est à 100% → le serveur va planter ou devenir inutilisable.

---

### 🔴 Scénario 6 — Un fichier de configuration a été modifié par erreur

**Situation** :

- Un admin a modifié `/etc/nginx/nginx.conf`
- nginx ne démarre plus
- Il ne se souvient plus de ce qu'il a modifié

**Questions à se poser** :

1. Y a-t-il une sauvegarde du fichier ?
2. Peut-on voir les dernières modifications avec les logs ?
3. Le paquet nginx fournit-il un fichier de configuration par défaut ?

**Marche à suivre** :

```bash
nginx -t                                # 1. Identifier l'erreur de config

# 2. Vérifier s'il y a une sauvegarde
ls /etc/nginx/*.bak
ls /etc/nginx/*.orig

# 3. Voir les dernières modifications du fichier
stat /etc/nginx/nginx.conf              # Date de modification
git log /etc/nginx/                     # Si le dossier est géré avec git

# 4. Restaurer la config par défaut du paquet
apt install --reinstall nginx           # Réinstaller nginx (restaure les configs par défaut)
dpkg -l nginx                           # Vérifier la version installée

# 5. Voir ce qu'il y a dans le fichier de config original du paquet
dpkg -L nginx | grep nginx.conf         # Trouver le fichier du paquet
```

> 💡 **Bonne pratique** : Toujours faire une copie avant de modifier un fichier de config : `cp nginx.conf nginx.conf.bak`. Et idéalement, gérer `/etc` avec **git** (etckeeper) pour avoir un historique de toutes les modifications.

---

## 14. Exercices d'entraînement

---

**Exercice 1** — Quels sont les droits du fichier suivant et qui peut faire quoi ?

```
-rwxr-x--- 1 www-data devs 4096 jan 15 script.sh
```

<details> <summary>👁️ Voir la réponse</summary>

- **Propriétaire** (`www-data`) : `rwx` → peut lire, écrire et exécuter
- **Groupe** (`devs`) : `r-x` → peut lire et exécuter, mais pas modifier
- **Autres** : `---` → aucun droit, pas d'accès

En octal : **750**

</details>

---

**Exercice 2** — Tu veux que tous les fichiers créés dans `/var/www/projet/` appartiennent automatiquement au groupe `webteam`, peu importe quel utilisateur les crée. Quelle commande utilises-tu ?

<details> <summary>👁️ Voir la réponse</summary>

```bash
chown :webteam /var/www/projet/         # Changer le groupe du dossier
chmod g+s /var/www/projet/              # Activer le SGID sur le dossier
```

Le **SGID** sur un dossier fait que tous les fichiers créés à l'intérieur héritent du groupe du dossier (`webteam`), quel que soit l'utilisateur qui les crée.

</details>

---

**Exercice 3** — Un serveur a ces caractéristiques :

- `df -h` → `/` à 95% d'utilisation
- `free -h` → swap à 80% utilisé
- `top` → un processus PHP consomme 90% du CPU

Dans quel ordre gères-tu les problèmes et pourquoi ?

<details> <summary>👁️ Voir la réponse</summary>

**Ordre de priorité** :

1. **CPU à 90% (PHP)** → Problème immédiat qui ralentit tout. Identifier le processus et le tuer ou le limiter. `kill <PID>` ou redémarrer le service PHP.
    
2. **Disque à 95%** → Urgent. Si le disque atteint 100%, plus rien ne peut écrire (logs, sessions, uploads). Trouver et supprimer les gros fichiers. `du -sh /* | sort -rh`
    
3. **Swap à 80%** → Conséquence des deux premiers. En libérant le CPU (le process PHP consomme probablement aussi de la RAM), le swap devrait se libérer naturellement.
    

</details>

---

## 15. Aide-mémoire rapide

```
NAVIGATION
pwd → où suis-je ?         ls -la → lister avec détails
cd /etc → aller dans /etc  cd ~ → aller dans son home

FICHIERS
cp / mv / rm               find / -name "*.conf"
cat / less / tail -f       grep "texte" fichier

UTILISATEURS
useradd -m -s /bin/bash jean      passwd jean
usermod -aG sudo jean             userdel -r jean
id jean                           who / last

DROITS (mémoriser les octaux)
777 → tout le monde tout    755 → owner tout, others r-x
644 → owner rw, others r    600 → owner uniquement
chmod 755 fichier           chown jean:groupe fichier

SERVICES (systemctl)
start / stop / restart / status / enable / disable
journalctl -u nginx -f → logs en temps réel

RÉSEAU
ip a                        → interfaces et IPs
ip route show               → table de routage
ss -tuln                    → ports en écoute
ping / traceroute / dig     → diagnostic

DISQUE
df -h                       → espace par partition
du -sh /dossier/            → taille d'un dossier
lsblk                       → disques et partitions

PAQUETS (Debian/Ubuntu)
apt update && apt upgrade   → mettre à jour
apt install <paquet>        → installer
apt remove / purge          → désinstaller

LOGS
/var/log/syslog             → logs système
/var/log/auth.log           → authentifications et SSH
journalctl -u <service>     → logs d'un service systemd
tail -f /var/log/syslog     → suivre en temps réel
```

---

> ✅ **À retenir** : L'administration Linux repose sur 4 réflexes :
> 
> 1. **Lire les logs** avant toute chose — ils te disent toujours ce qui se passe
> 2. **Vérifier les droits** — 80% des "Permission denied" viennent d'un `chmod` ou `chown` oublié
> 3. **Tester la config** avant de redémarrer un service — `nginx -t`, `apache2 -t`
> 4. **Faire des sauvegardes** avant toute modification — `cp fichier fichier.bak`