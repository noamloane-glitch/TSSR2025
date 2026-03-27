## 📋 Table des matières

- [Introduction](#introduction)
- [Le problème de la volatilité](#le-problème-de-la-volatilité)
- [iptables-save et iptables-restore](#iptables-save-et-iptables-restore)
  - [iptables-save : Exporter les règles](#iptables-save--exporter-les-règles)
  - [iptables-restore : Importer les règles](#iptables-restore--importer-les-règles)
  - [Format des fichiers de règles](#format-des-fichiers-de-règles)
- [Fichiers de configuration système](#fichiers-de-configuration-système)
  - [Emplacements standards](#emplacements-standards)
  - [Structure des répertoires](#structure-des-répertoires)
  - [Gestion des permissions](#gestion-des-permissions)
- [Services et automatisation](#services-et-automatisation)
  - [iptables-persistent (Debian/Ubuntu)](#iptables-persistent-debianubuntu)
  - [Services systemd personnalisés](#services-systemd-personnalisés)
  - [Scripts de démarrage](#scripts-de-démarrage)
  - [Automatisation avec cron](#automatisation-avec-cron)
- [Différences selon les distributions](#différences-selon-les-distributions)
  - [Debian/Ubuntu](#debianubuntu)
  - [Red Hat/CentOS/Rocky](#red-hatcentosrocky)
  - [Arch Linux](#arch-linux)
  - [Tableau comparatif](#tableau-comparatif)
- [Stratégies de sauvegarde](#stratégies-de-sauvegarde)
- [Bonnes pratiques](#bonnes-pratiques)

---

## 🎯 Introduction

La **persistance des règles iptables** est un aspect crucial de la gestion d'un pare-feu Linux. Par défaut, toutes les règles iptables sont **volatiles** : elles résident uniquement en mémoire et disparaissent au redémarrage du système.

### Pourquoi la persistance est-elle importante ?

- **Continuité de service** : Le pare-feu doit se rétablir automatiquement après un redémarrage
- **Cohérence de la sécurité** : Les règles de sécurité doivent rester actives en permanence
- **Gestion opérationnelle** : Éviter la configuration manuelle après chaque reboot
- **Automatisation** : Permettre le déploiement reproductible de configurations

> [!warning] Risque majeur
> Un système sans persistance des règles iptables peut redémarrer avec un pare-feu vide ou une policy par défaut dangereuse, exposant le système à des menaces.

---

## ⚠️ Le problème de la volatilité

### Comportement par défaut

```bash
# Configuration d'un pare-feu
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Vérification
sudo iptables -L -n
# Les règles sont actives

# Redémarrage du système
sudo reboot

# Après le redémarrage
sudo iptables -L -n
# ⚠️ Toutes les règles ont disparu !
# Les policies sont revenues à ACCEPT par défaut
```

### Conséquences d'une non-persistance

| Scénario | Sans persistance | Avec persistance |
|----------|------------------|------------------|
| **Redémarrage planifié** | Pare-feu désactivé | Pare-feu restauré automatiquement |
| **Panne électrique** | Système exposé au démarrage | Protection immédiate |
| **Mise à jour noyau** | Reconfiguration manuelle nécessaire | Configuration automatique |
| **Déploiement** | Script de post-installation requis | Mécanisme natif |

> [!info] État par défaut
> Après un redémarrage sans persistance :
> - Policies : `INPUT ACCEPT`, `FORWARD ACCEPT`, `OUTPUT ACCEPT`
> - Règles : Aucune (chaînes vides)
> - Tables : Réinitialisées (filter, nat, mangle, raw)

---

## 💿 iptables-save et iptables-restore

### iptables-save : Exporter les règles

La commande `iptables-save` génère une représentation textuelle de toutes les règles actives dans un format compatible avec `iptables-restore`.

#### Syntaxe de base

```bash
# Afficher toutes les règles (toutes les tables)
sudo iptables-save

# Sauvegarder dans un fichier
sudo iptables-save > /etc/iptables/rules.v4

# Sauvegarder une table spécifique
sudo iptables-save -t filter > /etc/iptables/filter-rules.v4
sudo iptables-save -t nat > /etc/iptables/nat-rules.v4
```

#### Options disponibles

```bash
# -c, --counters : Inclure les compteurs de paquets/octets
sudo iptables-save -c > rules-with-counters.v4

# -t, --table : Exporter une table spécifique
sudo iptables-save -t mangle > mangle-rules.v4

# Sans option : Exporte toutes les tables avec compteurs à zéro
sudo iptables-save > all-rules.v4
```

> [!tip] Astuce
> L'option `-c` est utile pour conserver les statistiques, mais peut compliquer la comparaison de fichiers. Pour les sauvegardes de configuration, omettez généralement cette option.

#### Exemples pratiques

```bash
# Sauvegarde horodatée
sudo iptables-save > /root/iptables-backup-$(date +%Y%m%d-%H%M%S).rules

# Sauvegarde quotidienne dans un répertoire dédié
BACKUP_DIR="/var/backups/iptables"
sudo mkdir -p "$BACKUP_DIR"
sudo iptables-save > "$BACKUP_DIR/rules-$(date +%Y%m%d).v4"

# Sauvegarde avant modification (avec confirmation de contenu)
echo "=== Sauvegarde des règles actuelles ==="
sudo iptables-save | tee /tmp/iptables-before-changes.rules
echo "=== Sauvegarde terminée ==="
```

### iptables-restore : Importer les règles

La commande `iptables-restore` charge des règles depuis un fichier au format `iptables-save`.

#### Syntaxe de base

```bash
# Restaurer depuis un fichier (avec flush implicite)
sudo iptables-restore < /etc/iptables/rules.v4

# Restaurer sans vider les règles existantes
sudo iptables-restore -n < /etc/iptables/rules.v4

# Tester la syntaxe sans appliquer
sudo iptables-restore -t < /etc/iptables/rules.v4

# Mode verbeux
sudo iptables-restore -v < /etc/iptables/rules.v4
```

#### Options importantes

| Option | Description | Utilisation |
|--------|-------------|-------------|
| **-n, --noflush** | Ne pas vider les chaînes existantes | Ajouter des règles sans écraser |
| **-t, --test** | Tester la syntaxe sans appliquer | Validation avant déploiement |
| **-v, --verbose** | Mode verbeux | Débogage et suivi |
| **-c, --counters** | Restaurer les compteurs | Conservation des statistiques |
| **-w, --wait** | Attendre le verrou xtables | Éviter les conflits |

> [!warning] Comportement par défaut
> Sans l'option `-n`, `iptables-restore` **vide toutes les chaînes** avant de charger les nouvelles règles. Cela peut couper les connexions actives !

#### Comparaison flush vs noflush

```bash
# Scénario 1 : Remplacement complet (défaut)
sudo iptables-restore < new-rules.v4
# 1. Vide toutes les chaînes
# 2. Charge les nouvelles règles
# Résultat : Configuration complètement remplacée

# Scénario 2 : Ajout de règles (noflush)
sudo iptables-restore -n < additional-rules.v4
# 1. Conserve les règles existantes
# 2. Ajoute les nouvelles règles
# Résultat : Fusion des configurations
```

> [!info] Cas d'usage
> - **Flush (défaut)** : Déploiement initial, remplacement total de configuration
> - **Noflush (-n)** : Ajout de règles temporaires, tests sans perturbation

#### Restauration sécurisée avec validation

```bash
#!/bin/bash
# safe-restore.sh

RULES_FILE="$1"

# Vérifier l'existence du fichier
if [ ! -f "$RULES_FILE" ]; then
    echo "❌ Fichier $RULES_FILE introuvable"
    exit 1
fi

# Tester la syntaxe
echo "🔍 Validation de la syntaxe..."
if ! sudo iptables-restore -t < "$RULES_FILE"; then
    echo "❌ Erreur de syntaxe détectée"
    exit 1
fi

# Sauvegarder la config actuelle
echo "💾 Sauvegarde de la configuration actuelle..."
sudo iptables-save > /tmp/iptables-backup-pre-restore.rules

# Appliquer les règles
echo "⚙️ Application des nouvelles règles..."
if sudo iptables-restore < "$RULES_FILE"; then
    echo "✅ Règles appliquées avec succès"
    
    # Demander confirmation
    read -t 30 -p "Confirmer (y/N) ? " confirm
    if [[ ! "$confirm" =~ ^[Yy]$ ]]; then
        echo "⏪ Rollback..."
        sudo iptables-restore < /tmp/iptables-backup-pre-restore.rules
        echo "✅ Configuration précédente restaurée"
        exit 1
    fi
else
    echo "❌ Erreur lors de l'application"
    exit 1
fi

echo "✅ Configuration finale confirmée"
```

### Format des fichiers de règles

#### Structure générale

```bash
# Exemple de fichier généré par iptables-save
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
COMMIT

*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o eth0 -j MASQUERADE
COMMIT

*mangle
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
COMMIT
```

#### Explication ligne par ligne

```bash
# Début de déclaration d'une table
*filter
# Syntaxe : *<nom_table>

# Définition des policies des chaînes built-in
:INPUT DROP [0:0]
# Syntaxe : :<CHAINE> <POLICY> [paquets:octets]
# [0:0] = compteurs (0 paquets, 0 octets)

# Ajout d'une règle
-A INPUT -i lo -j ACCEPT
# Syntaxe identique à iptables -A

# Fin de table et validation atomique
COMMIT
# Applique toutes les règles de la table en une seule transaction
```

> [!info] Transaction atomique
> Le mot-clé `COMMIT` est crucial : il garantit que toutes les règles d'une table sont appliquées en une seule opération atomique. Si une erreur survient, aucune règle de cette table n'est appliquée.

#### Format avec compteurs

```bash
# Génération avec compteurs
sudo iptables-save -c > rules-with-counters.v4

# Résultat
*filter
:INPUT DROP [523:45234]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [18392:1849234]
[1234:98765] -A INPUT -i lo -j ACCEPT
[8765:456789] -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
[42:3360] -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
COMMIT
```

#### Édition manuelle des fichiers

```bash
# Les fichiers peuvent être édités manuellement
sudo nano /etc/iptables/rules.v4
```

> [!warning] Précautions lors de l'édition manuelle
> - Respecter rigoureusement la syntaxe
> - Toujours terminer chaque table par `COMMIT`
> - Utiliser `iptables-restore -t` pour valider avant application
> - Conserver une sauvegarde de la version fonctionnelle

#### Commentaires dans les fichiers

```bash
# Les lignes commençant par # sont des commentaires
# Ils sont ignorés par iptables-restore

*filter
# === Politique par défaut ===
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]

# === Règles de base ===
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# === Services autorisés ===
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT  # SSH
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT  # HTTP
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT # HTTPS

COMMIT
```

---

## 📁 Fichiers de configuration système

### Emplacements standards

Chaque distribution Linux adopte des conventions différentes pour stocker les règles iptables.

#### Debian/Ubuntu

```bash
# Emplacement principal (avec iptables-persistent)
/etc/iptables/rules.v4              # Règles IPv4
/etc/iptables/rules.v6              # Règles IPv6

# Ancien emplacement (Debian < 8)
/etc/iptables/rules                 # Règles génériques
```

#### Red Hat/CentOS/Rocky/Fedora

```bash
# Services iptables (ancien système)
/etc/sysconfig/iptables             # Règles IPv4
/etc/sysconfig/ip6tables            # Règles IPv6

# Avec firewalld (système moderne)
/etc/firewalld/                     # Configuration firewalld
# Note : firewalld utilise son propre format XML
```

#### Arch Linux

```bash
# Emplacements recommandés
/etc/iptables/iptables.rules        # Règles IPv4
/etc/iptables/ip6tables.rules       # Règles IPv6
```

> [!tip] Convention personnelle
> Si vous gérez manuellement la persistance, créez votre propre convention :
> ```bash
> /opt/firewall/
> ├── rules.v4
> ├── rules.v6
> └── backups/
>     ├── rules-20250212.v4
>     └── rules-20250211.v4
> ```

### Structure des répertoires

#### Arborescence recommandée

```bash
/etc/iptables/
├── rules.v4                    # Configuration IPv4 active
├── rules.v6                    # Configuration IPv6 active
├── rules.d/                    # Règles modulaires (optionnel)
│   ├── 00-policies.rules
│   ├── 10-basic.rules
│   ├── 20-services.rules
│   └── 99-logging.rules
└── backups/                    # Sauvegardes automatiques
    ├── rules.v4.20250212
    ├── rules.v4.20250211
    └── rules.v4.20250210
```

#### Approche modulaire

```bash
# Créer un système modulaire
sudo mkdir -p /etc/iptables/rules.d

# Fichier 00-policies.rules
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
COMMIT

# Fichier 10-basic.rules
*filter
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
COMMIT

# Fichier 20-services.rules
*filter
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
COMMIT

# Script de compilation
cat /etc/iptables/rules.d/*.rules > /etc/iptables/rules.v4
```

> [!warning] Limites de l'approche modulaire
> L'approche modulaire nécessite un script personnalisé pour fusionner les fichiers, car `iptables-restore` attend un format atomique avec un seul `COMMIT` par table.

### Gestion des permissions

#### Permissions recommandées

```bash
# Fichiers de règles : lecture seule root
sudo chmod 600 /etc/iptables/rules.v4
sudo chown root:root /etc/iptables/rules.v4

# Répertoire : accessible uniquement par root
sudo chmod 700 /etc/iptables
sudo chown root:root /etc/iptables

# Vérification
ls -la /etc/iptables/
# drwx------  3 root root 4096 Feb 12 10:00 .
# -rw-------  1 root root 1234 Feb 12 10:00 rules.v4
```

> [!info] Justification des permissions strictes
> Les règles iptables contiennent des informations sensibles sur l'architecture réseau et les services exposés. Un accès non autorisé pourrait révéler des vulnérabilités potentielles.

#### Script de validation des permissions

```bash
#!/bin/bash
# check-iptables-permissions.sh

RULES_FILE="/etc/iptables/rules.v4"

# Vérifier les permissions
PERMS=$(stat -c "%a" "$RULES_FILE")
OWNER=$(stat -c "%U" "$RULES_FILE")

if [ "$PERMS" != "600" ]; then
    echo "⚠️ Permissions incorrectes : $PERMS (attendu: 600)"
    echo "Correction : sudo chmod 600 $RULES_FILE"
fi

if [ "$OWNER" != "root" ]; then
    echo "⚠️ Propriétaire incorrect : $OWNER (attendu: root)"
    echo "Correction : sudo chown root:root $RULES_FILE"
fi
```

---

## ⚙️ Services et automatisation

### iptables-persistent (Debian/Ubuntu)

Le paquet `iptables-persistent` fournit une solution clé en main pour la persistance des règles sur les systèmes Debian et Ubuntu.

#### Installation

```bash
# Mise à jour et installation
sudo apt update
sudo apt install iptables-persistent

# Pendant l'installation, deux questions sont posées :
# - Sauvegarder les règles IPv4 actuelles ? (Oui/Non)
# - Sauvegarder les règles IPv6 actuelles ? (Oui/Non)
```

> [!tip] Installation non-interactive
> Pour automatiser l'installation sans questions :
> ```bash
> echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
> echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections
> sudo apt install -y iptables-persistent
> ```

#### Commandes principales

```bash
# Sauvegarder les règles actuelles
sudo netfilter-persistent save
# Écrit dans /etc/iptables/rules.v4 et rules.v6

# Recharger les règles depuis les fichiers
sudo netfilter-persistent reload

# Vider toutes les règles
sudo netfilter-persistent flush

# Démarrer le service (chargement des règles)
sudo systemctl start netfilter-persistent

# Activer le chargement automatique au démarrage
sudo systemctl enable netfilter-persistent

# Vérifier le statut
sudo systemctl status netfilter-persistent
```

#### Fonctionnement

```bash
# Le service utilise des scripts dans :
/usr/share/netfilter-persistent/plugins.d/
├── 15-ip4tables
└── 25-ip6tables

# Ces scripts :
# 1. Chargent les règles au démarrage depuis /etc/iptables/
# 2. Fournissent les commandes save/reload/flush
```

#### Workflow typique

```bash
# 1. Configurer les règles en live
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 2. Tester que tout fonctionne
curl https://exemple.com

# 3. Sauvegarder la configuration
sudo netfilter-persistent save

# 4. La configuration survivra au redémarrage
sudo reboot

# Après redémarrage, les règles sont automatiquement rechargées
```

> [!info] Avantages de iptables-persistent
> - ✅ Installation simple et rapide
> - ✅ Intégration native avec systemd
> - ✅ Commandes claires et intuitives
> - ✅ Support IPv4 et IPv6
> - ✅ Maintenance active par Debian/Ubuntu

### Services systemd personnalisés

Pour les distributions sans `iptables-persistent` ou pour un contrôle plus fin, créez un service systemd personnalisé.

#### Création d'un service de restauration

```bash
# 1. Créer le fichier de service
sudo nano /etc/systemd/system/iptables-restore.service
```

```ini
[Unit]
Description=Restore iptables firewall rules
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /etc/iptables/rules.v4
ExecStart=/sbin/ip6tables-restore /etc/iptables/rules.v6
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

```bash
# 2. Recharger systemd
sudo systemctl daemon-reload

# 3. Activer le service
sudo systemctl enable iptables-restore.service

# 4. Démarrer manuellement (pour tester)
sudo systemctl start iptables-restore.service

# 5. Vérifier le statut
sudo systemctl status iptables-restore.service
```

#### Service avec sauvegarde automatique

```bash
# Créer un service de sauvegarde
sudo nano /etc/systemd/system/iptables-save.service
```

```ini
[Unit]
Description=Save iptables firewall rules
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-save -f /etc/iptables/rules.v4
ExecStart=/sbin/ip6tables-save -f /etc/iptables/rules.v6

[Install]
WantedBy=multi-user.target
```

```bash
# Timer pour sauvegarde quotidienne
sudo nano /etc/systemd/system/iptables-save.timer
```

```ini
[Unit]
Description=Save iptables rules daily

[Timer]
OnCalendar=daily
OnBootSec=5min
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Activer le timer
sudo systemctl enable iptables-save.timer
sudo systemctl start iptables-save.timer

# Vérifier les timers actifs
sudo systemctl list-timers --all | grep iptables
```

> [!tip] Ordre d'exécution
> Le paramètre `Before=network-pre.target` garantit que le pare-feu est actif **avant** que les interfaces réseau ne soient pleinement configurées, offrant une protection dès le démarrage.

### Scripts de démarrage

#### Script init.d traditionnel (systèmes anciens)

```bash
# Créer le script
sudo nano /etc/init.d/iptables-custom
```

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          iptables-custom
# Required-Start:    $local_fs $network
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Restore iptables rules
# Description:       Load iptables rules from /etc/iptables/rules.v4
### END INIT INFO

RULES_FILE="/etc/iptables/rules.v4"

case "$1" in
    start)
        echo "Loading iptables rules..."
        if [ -f "$RULES_FILE" ]; then
            iptables-restore < "$RULES_FILE"
            echo "iptables rules loaded."
        else
            echo "Rules file not found: $RULES_FILE"
            exit 1
        fi
        ;;
    stop)
        echo "Flushing iptables rules..."
        iptables -F
        iptables -X
        iptables -P INPUT ACCEPT
        iptables -P FORWARD ACCEPT
        iptables -P OUTPUT ACCEPT
        echo "iptables rules flushed."
        ;;
    restart|reload)
        $0 stop
        $0 start
        ;;
    save)
        echo "Saving iptables rules..."
        iptables-save > "$RULES_FILE"
        echo "iptables rules saved."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|reload|save}"
        exit 1
        ;;
esac

exit 0
```

```bash
# Rendre exécutable
sudo chmod +x /etc/init.d/iptables-custom

# Activer au démarrage (système SysV)
sudo update-rc.d iptables-custom defaults

# Ou sur Red Hat/CentOS
sudo chkconfig iptables-custom on
```

#### Script personnalisé avec rollback automatique

```bash
sudo nano /usr/local/bin/apply-iptables-rules.sh
```

```bash
#!/bin/bash
# apply-iptables-rules.sh - Restauration sécurisée avec rollback

RULES_FILE="/etc/iptables/rules.v4"
BACKUP_FILE="/tmp/iptables-backup-before-apply.rules"
TIMEOUT=60

# Fonction de nettoyage
cleanup() {
    rm -f "$BACKUP_FILE"
}

trap cleanup EXIT

# Vérifier l'existence du fichier
if [ ! -f "$RULES_FILE" ]; then
    echo "❌ Fichier $RULES_FILE introuvable"
    exit 1
fi

# Tester la syntaxe
if ! iptables-restore -t < "$RULES_FILE"; then
    echo "❌ Erreur de syntaxe dans $RULES_FILE"
    exit 1
fi

# Sauvegarder l'état actuel
iptables-save > "$BACKUP_FILE"

# Appliquer les nouvelles règles
iptables-restore < "$RULES_FILE"

echo "✅ Règles appliquées. Test de connectivité..."

# Rollback automatique si pas de confirmation
(
    sleep $TIMEOUT
    if [ -f "$BACKUP_FILE" ]; then
        echo "⏪ Timeout atteint - Rollback automatique"
        iptables-restore < "$BACKUP_FILE"
    fi
) &
ROLLBACK_PID=$!

# Demander confirmation
read -t $((TIMEOUT - 5)) -p "Confirmer les règles ? (y/N) : " confirm

if [[ "$confirm" =~ ^[Yy]$ ]]; then
    kill $ROLLBACK_PID 2>/dev/null
    echo "✅ Configuration confirmée"
else
    echo "⏪ Rollback manuel"
    iptables-restore < "$BACKUP_FILE"
fi
```

```bash
# Rendre exécutable
sudo chmod +x /usr/local/bin/apply-iptables-rules.sh
```

### Automatisation avec cron

#### Sauvegarde quotidienne

```bash
# Éditer la crontab root
sudo crontab -e
```

```bash
# Sauvegarder iptables tous les jours à 2h du matin
0 2 * * * /sbin/iptables-save > /etc/iptables/rules.v4

# Sauvegarde avec rotation (garder 7 jours)
0 2 * * * /sbin/iptables-save > /var/backups/iptables/rules-$(date +\%Y\%m\%d).v4 && find /var/backups/iptables -name "rules-*.v4" -mtime +7 -delete
```

#### Script de sauvegarde avancé

```bash
sudo nano /usr/local/bin/backup-iptables.sh
```

```bash
#!/bin/bash
# backup-iptables.sh

BACKUP_DIR="/var/backups/iptables"
DATE=$(date +%Y%m%d-%H%M%S)
RETENTION_DAYS=30

# Créer le répertoire
mkdir -p "$BACKUP_DIR"

# Sauvegarder IPv4
iptables-save > "$BACKUP_DIR/rules.v4.$DATE"

# Sauvegarder IPv6
ip6tables-save > "$BACKUP_DIR/rules.v6.$DATE"

# Créer un lien symbolique vers la dernière sauvegarde
ln -sf "$BACKUP_DIR/rules.v4.$DATE" "$BACKUP_DIR/rules.v4.latest"
ln -sf "$BACKUP_DIR/rules.v6.$DATE" "$BACKUP_DIR/rules.v6.latest"

# Nettoyer les anciennes sauvegardes
find "$BACKUP_DIR" -name "rules.*.v*.*" -mtime +$RETENTION_DAYS -delete

# Compter les sauvegardes
COUNT=$(find "$BACKUP_DIR" -name "rules.v4.*" ! -name "*.latest" | wc -l)
echo "✅ Sauvegarde effectuée. Total : $COUNT sauvegardes conservées."
```

```bash
# Rendre exécutable
sudo chmod +x /usr/local/bin/backup-iptables.sh

# Ajouter à cron
sudo crontab -e
```

```bash
# Exécuter la sauvegarde tous les jours à 3h
0 3 * * * /usr/local/bin/backup-iptables.sh >> /var/log/iptables-backup.log 2>&1
```

#### Vérification de cohérence hebdomadaire

```bash
sudo crontab -e
```

```bash
# Vérifier la cohérence des règles chaque lundi à 6h
0 6 * * 1 /usr/local/bin/check-iptables-health.sh
```

```bash
# Script de vérification
sudo nano /usr/local/bin/check-iptables-health.sh
```

```bash
#!/bin/bash
# check-iptables-health.sh

RULES_FILE="/etc/iptables/rules.v4"
ADMIN_EMAIL="admin@example.com"

# Vérifier que le fichier existe
if [ ! -f "$RULES_FILE" ]; then
    echo "⚠️ Fichier de règles manquant" | mail -s "Alerte iptables" $ADMIN_EMAIL
    exit 1
fi

# Vérifier la syntaxe
if ! iptables-restore -t < "$RULES_FILE"; then
    echo "⚠️ Erreur de syntaxe dans les règles" | mail -s "Alerte iptables" $ADMIN_EMAIL
    exit 1
fi

# Compter les règles actives
ACTIVE_RULES=$(iptables -L -n | grep -c "^ACCEPT\|^DROP\|^REJECT")

if [ $ACTIVE_RULES -lt 5 ]; then
    echo "⚠️ Nombre de règles anormalement bas : $ACTIVE_RULES" | mail -s "Alerte iptables" $ADMIN_EMAIL
fi

echo "✅ Vérification OK - $ACTIVE_RULES règles actives"
```

---

## 🐧 Différences selon les distributions

### Debian/Ubuntu

#### Méthode recommandée : iptables-persistent

```bash
# Installation
sudo apt install iptables-persistent

# Fichiers
/etc/iptables/rules.v4
/etc/iptables/rules.v6

# Service
sudo systemctl status netfilter-persistent

# Commandes
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

#### Alternative : Service systemd manuel

```bash
# Si iptables-persistent n'est pas souhaité
sudo systemctl enable iptables-restore.service
```

> [!info] Particularité Debian
> Debian utilise le framework `netfilter-persistent` qui permet d'ajouter des plugins pour d'autres outils de filtrage (nftables, ebtables).

### Red Hat/CentOS/Rocky

#### Méthode traditionnelle (iptables service)

```bash
# Installation du service iptables
sudo yum install iptables-services   # CentOS 7
sudo dnf install iptables-services   # CentOS 8+, Rocky

# Désactiver firewalld (si présent)
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl mask firewalld

# Activer iptables
sudo systemctl enable iptables
sudo systemctl start iptables

# Sauvegarder les règles
sudo service iptables save
# ou
sudo iptables-save > /etc/sysconfig/iptables

# Restaurer les règles
sudo systemctl restart iptables
```

#### Fichiers de configuration

```bash
# Règles IPv4
/etc/sysconfig/iptables

# Règles IPv6
/etc/sysconfig/ip6tables

# Configuration du service
/etc/sysconfig/iptables-config
```

#### Exemple de /etc/sysconfig/iptables

```bash
# Generated by iptables-save
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

> [!warning] Firewalld vs iptables
> Les versions récentes de RHEL/CentOS utilisent `firewalld` par défaut. Si vous préférez iptables classique, désactivez impérativement firewalld pour éviter les conflits.

### Arch Linux

#### Méthode recommandée : Service systemd

```bash
# Installation (iptables est déjà installé)
sudo pacman -S iptables

# Créer le répertoire
sudo mkdir -p /etc/iptables

# Sauvegarder les règles
sudo iptables-save > /etc/iptables/iptables.rules
sudo ip6tables-save > /etc/iptables/ip6tables.rules

# Activer les services
sudo systemctl enable iptables.service
sudo systemctl enable ip6tables.service

# Démarrer
sudo systemctl start iptables.service
sudo systemctl start ip6tables.service
```

#### Fichiers

```bash
# Règles IPv4
/etc/iptables/iptables.rules

# Règles IPv6
/etc/iptables/ip6tables.rules
```

> [!tip] Arch Linux
> Arch fournit des services systemd natifs pour iptables (`iptables.service` et `ip6tables.service`) qui utilisent ces emplacements standards.

### Tableau comparatif

| Distribution | Paquet | Fichier IPv4 | Fichier IPv6 | Service | Commande save |
|--------------|--------|--------------|--------------|---------|---------------|
| **Debian/Ubuntu** | iptables-persistent | /etc/iptables/rules.v4 | /etc/iptables/rules.v6 | netfilter-persistent | `netfilter-persistent save` |
| **RHEL/CentOS/Rocky** | iptables-services | /etc/sysconfig/iptables | /etc/sysconfig/ip6tables | iptables | `service iptables save` |
| **Arch Linux** | iptables | /etc/iptables/iptables.rules | /etc/iptables/ip6tables.rules | iptables | `iptables-save > ...` |
| **Gentoo** | iptables | /var/lib/iptables/rules-save | /var/lib/ip6tables/rules-save | iptables | `/etc/init.d/iptables save` |

---

## 💾 Stratégies de sauvegarde

### Sauvegarde avant modification

```bash
#!/bin/bash
# pre-change-backup.sh

BACKUP_DIR="/var/backups/iptables/pre-change"
mkdir -p "$BACKUP_DIR"

TIMESTAMP=$(date +%Y%m%d-%H%M%S)
iptables-save > "$BACKUP_DIR/rules-$TIMESTAMP.v4"

echo "✅ Sauvegarde pré-modification : $BACKUP_DIR/rules-$TIMESTAMP.v4"
```

### Rotation automatique

```bash
#!/bin/bash
# rotate-iptables-backups.sh

BACKUP_DIR="/var/backups/iptables"
RETENTION=30  # jours

# Supprimer les sauvegardes de plus de 30 jours
find "$BACKUP_DIR" -name "rules-*.v4" -mtime +$RETENTION -delete

# Afficher le nombre de sauvegardes conservées
COUNT=$(find "$BACKUP_DIR" -name "rules-*.v4" | wc -l)
echo "Sauvegardes conservées : $COUNT"
```

### Versioning avec Git

```bash
# Initialiser un dépôt Git
sudo mkdir -p /etc/iptables
cd /etc/iptables
sudo git init
sudo git config user.name "System"
sudo git config user.email "root@localhost"

# Premier commit
sudo iptables-save > rules.v4
sudo git add rules.v4
sudo git commit -m "Configuration initiale"

# Après chaque modification
sudo netfilter-persistent save
cd /etc/iptables
sudo git add rules.v4
sudo git commit -m "Ajout règle HTTPS"

# Voir l'historique
sudo git log --oneline

# Restaurer une version précédente
sudo git checkout <commit-hash> rules.v4
sudo iptables-restore < rules.v4
```

> [!tip] Avantage du versioning Git
> - Historique complet des modifications
> - Possibilité de revenir à n'importe quelle version
> - Annotations avec messages de commit
> - Diff entre versions

---

## ✅ Bonnes pratiques

### 1. Toujours tester avant de rendre persistant

```bash
# 1. Appliquer les règles en mémoire
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# 2. Tester le fonctionnement
curl http://localhost:8080

# 3. Seulement après validation, sauvegarder
sudo netfilter-persistent save
```

### 2. Conserver plusieurs générations de sauvegarde

```bash
# Script de sauvegarde incrémentale
BACKUP_DIR="/var/backups/iptables"
DATE=$(date +%Y%m%d-%H%M%S)

iptables-save > "$BACKUP_DIR/rules-$DATE.v4"
ln -sf "$BACKUP_DIR/rules-$DATE.v4" "$BACKUP_DIR/rules-latest.v4"

# Garder les 10 dernières sauvegardes
ls -t "$BACKUP_DIR"/rules-*.v4 | tail -n +11 | xargs rm -f
```

### 3. Documenter les modifications

```bash
# Ajouter un en-tête documenté
cat << 'EOF' > /etc/iptables/rules.v4
# ==================================
# Configuration iptables
# Date: $(date)
# Auteur: Franck
# Modification: Ajout règle HTTPS
# ==================================
EOF

iptables-save >> /etc/iptables/rules.v4
```

### 4. Utiliser un système de rollback automatique

```bash
# Appliquer avec fenêtre de confirmation
sudo /usr/local/bin/apply-iptables-rules.sh
# Si pas de confirmation dans 60s → rollback automatique
```

### 5. Vérifier après chaque redémarrage

```bash
# Ajouter une vérification post-boot
sudo nano /etc/rc.local
```

```bash
#!/bin/bash
# Vérifier que les règles sont chargées
RULE_COUNT=$(iptables -L -n | grep -c "ACCEPT\|DROP\|REJECT")

if [ $RULE_COUNT -lt 5 ]; then
    echo "⚠️ Nombre de règles anormalement bas" | mail -s "Alerte iptables" admin@example.com
fi
```

### 6. Séparer IPv4 et IPv6

```bash
# Toujours gérer séparément
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6

# Restaurer séparément
sudo iptables-restore < /etc/iptables/rules.v4
sudo ip6tables-restore < /etc/iptables/rules.v6
```

### 7. Permissions strictes

```bash
# Fichiers lisibles uniquement par root
sudo chmod 600 /etc/iptables/rules.*
sudo chown root:root /etc/iptables/rules.*
```

### 8. Audit régulier

```bash
#!/bin/bash
# audit-iptables.sh

echo "=== Audit iptables $(date) ==="

# Vérifier l'existence des fichiers
for file in /etc/iptables/rules.v4 /etc/iptables/rules.v6; do
    if [ -f "$file" ]; then
        echo "✅ $file existe"
    else
        echo "❌ $file manquant"
    fi
done

# Vérifier les services
if systemctl is-enabled netfilter-persistent &>/dev/null; then
    echo "✅ Service netfilter-persistent activé"
else
    echo "❌ Service netfilter-persistent non activé"
fi

# Compter les règles actives
RULES=$(iptables -L -n | grep -c "ACCEPT\|DROP\|REJECT")
echo "📊 Règles actives : $RULES"
```

### 9. Intégration CI/CD

```yaml
# Exemple GitLab CI pour validation
validate_iptables:
  stage: test
  script:
    - iptables-restore -t < rules.v4
    - echo "✅ Syntaxe valide"
  only:
    - merge_requests
```

### 10. Plan de reprise après sinistre

```bash
# Documenter la procédure de restauration complète
# /root/RESTORE-IPTABLES.md

## Procédure de restauration d'urgence

1. Se connecter en mode rescue
2. Restaurer depuis la dernière sauvegarde
   sudo iptables-restore < /var/backups/iptables/rules-latest.v4
3. Vérifier le fonctionnement
   sudo iptables -L -n -v
4. Rendre persistant
   sudo netfilter-persistent save
```

---

> [!info] Résumé
> **Commandes essentielles :**
> - `iptables-save > fichier` : Exporter les règles
> - `iptables-restore < fichier` : Importer les règles
> - `netfilter-persistent save` : Sauvegarder (Debian/Ubuntu)
> - `service iptables save` : Sauvegarder (RHEL/CentOS)
> - `systemctl enable netfilter-persistent` : Activer la persistance au boot

> [!warning] Rappel critique
> Sans mécanisme de persistance, votre pare-feu sera **désactivé à chaque redémarrage** ! Configurez toujours la persistance lors du déploiement initial.
