## 📋 Table des matières

- [Introduction](#introduction)
- [Lister les règles](#lister-les-règles)
  - [Option -L (--list)](#option--l---list)
  - [Option -S (--list-rules)](#option--s---list-rules)
  - [Comparaison -L vs -S](#comparaison--l-vs--s)
- [Ajouter des règles](#ajouter-des-règles)
  - [Option -A (--append)](#option--a---append)
  - [Option -I (--insert)](#option--i---insert)
  - [Différences entre -A et -I](#différences-entre--a-et--i)
- [Supprimer des règles](#supprimer-des-règles)
  - [Option -D (--delete)](#option--d---delete)
  - [Option -F (--flush)](#option--f---flush)
  - [Suppression sécurisée](#suppression-sécurisée)
- [Sauvegarde et restauration](#sauvegarde-et-restauration)
  - [iptables-save](#iptables-save)
  - [iptables-restore](#iptables-restore)
  - [Persistance des règles](#persistance-des-règles)
- [Bonnes pratiques](#bonnes-pratiques)

---

## 🎯 Introduction

Les commandes iptables permettent de gérer dynamiquement les règles du pare-feu Netfilter. La maîtrise de ces commandes est essentielle pour :
- Diagnostiquer les problèmes de filtrage
- Modifier les règles sans interruption de service
- Sauvegarder et restaurer les configurations
- Automatiser la gestion du pare-feu

> [!warning] Attention
> Toutes les commandes iptables nécessitent les privilèges root (sudo). Une erreur peut couper l'accès réseau !

---

## 📊 Lister les règles

### Option -L (--list)

La commande `-L` affiche les règles dans un format lisible et détaillé.

#### Syntaxe de base

```bash
# Lister toutes les règles de toutes les tables
sudo iptables -L

# Lister les règles d'une chaîne spécifique
sudo iptables -L <CHAINE>

# Exemple : lister uniquement INPUT
sudo iptables -L INPUT
```

#### Options complémentaires

```bash
# -v (--verbose) : affiche des informations détaillées
sudo iptables -L -v
# Affiche : paquets, octets, interfaces, options TOS, etc.

# -n (--numeric) : affiche les adresses IP et ports en numérique
sudo iptables -L -n
# Évite la résolution DNS (plus rapide)

# --line-numbers : affiche les numéros de ligne
sudo iptables -L --line-numbers
# Utile pour supprimer une règle précise

# -t <table> : spécifie la table
sudo iptables -t nat -L
# Par défaut : table filter
```

#### Combinaisons courantes

```bash
# Affichage complet et optimisé
sudo iptables -L -n -v --line-numbers

# Affichage pour une chaîne spécifique
sudo iptables -L INPUT -n -v --line-numbers

# Vérifier la table NAT
sudo iptables -t nat -L -n -v
```

> [!example] Exemple de sortie
> ```
> Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
> num   pkts bytes target     prot opt in     out     source               destination         
> 1        0     0 ACCEPT     tcp  --  *      *       192.168.1.0/24       0.0.0.0/0            tcp dpt:22
> 2      150 12000 DROP       all  --  *      *       10.0.0.0/8           0.0.0.0/0           
> ```

### Option -S (--list-rules)

La commande `-S` affiche les règles au format de commande iptables (format machine-readable).

#### Syntaxe de base

```bash
# Lister toutes les règles au format commande
sudo iptables -S

# Lister les règles d'une chaîne spécifique
sudo iptables -S INPUT

# Avec une table spécifique
sudo iptables -t nat -S
```

#### Caractéristiques

```bash
# Format de sortie : commandes iptables directement réutilisables
sudo iptables -S
# Résultat exemple :
# -P INPUT ACCEPT
# -P FORWARD DROP
# -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
# -A INPUT -s 10.0.0.0/8 -j DROP
```

> [!tip] Astuce
> L'option `-S` est idéale pour scripter ou copier-coller des règles, car elle retourne des commandes prêtes à l'emploi.

### Comparaison -L vs -S

| Critère | -L (list) | -S (list-rules) |
|---------|-----------|-----------------|
| **Format** | Tableau lisible | Commandes iptables |
| **Lisibilité humaine** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Réutilisabilité** | ❌ | ✅ |
| **Statistiques** | ✅ (avec -v) | ❌ |
| **Numéros de ligne** | ✅ (avec --line-numbers) | ❌ |
| **Utilisation en script** | ❌ | ✅ |
| **Copier-coller** | ❌ | ✅ |

> [!info] Quand utiliser quoi ?
> - **-L** : pour diagnostiquer, visualiser, comprendre les règles actives
> - **-S** : pour sauvegarder, scripter, dupliquer des configurations

---

## ➕ Ajouter des règles

### Option -A (--append)

L'option `-A` ajoute une règle **à la fin** d'une chaîne existante.

#### Syntaxe générale

```bash
sudo iptables -A <CHAINE> [options] -j <CIBLE>
```

#### Exemples pratiques

```bash
# Autoriser SSH depuis un réseau spécifique
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT

# Bloquer une adresse IP
sudo iptables -A INPUT -s 203.0.113.50 -j DROP

# Autoriser le trafic établi
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Bloquer tout trafic ICMP (ping)
sudo iptables -A INPUT -p icmp -j DROP

# Limiter les connexions HTTP (anti-DDoS basique)
sudo iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 20 -j REJECT
```

> [!warning] Ordre des règles
> Avec `-A`, la règle est ajoutée à la fin. Si une règle précédente correspond déjà, la nouvelle ne sera jamais évaluée !

#### Exemple d'impact de l'ordre

```bash
# ❌ MAUVAIS : cette règle ne servira à rien
sudo iptables -A INPUT -j DROP           # Bloque tout en premier
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # Ne sera jamais atteinte !

# ✅ BON : ordre logique
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # Règle spécifique d'abord
sudo iptables -A INPUT -j DROP                       # Règle générale à la fin
```

### Option -I (--insert)

L'option `-I` insère une règle **à une position spécifique** (par défaut en première position).

#### Syntaxe générale

```bash
# Insérer en première position
sudo iptables -I <CHAINE> [options] -j <CIBLE>

# Insérer à une position précise
sudo iptables -I <CHAINE> <NUMERO> [options] -j <CIBLE>
```

#### Exemples pratiques

```bash
# Insérer en première position (priorité maximale)
sudo iptables -I INPUT -s 203.0.113.100 -j DROP

# Insérer à la 3ème position
sudo iptables -I INPUT 3 -p tcp --dport 443 -j ACCEPT

# Insérer une règle de log avant une règle de drop
sudo iptables -I INPUT 1 -s 10.0.0.0/8 -j LOG --log-prefix "Blocked: "
sudo iptables -I INPUT 2 -s 10.0.0.0/8 -j DROP
```

> [!tip] Astuce
> Utilisez `-I` pour ajouter rapidement une règle d'urgence (blocage d'IP malveillante) sans modifier toute la configuration.

#### Cas d'usage typique

```bash
# Situation : pare-feu actif, attaque en cours
# Besoin : bloquer immédiatement une IP

# Solution : insertion en tête de chaîne
sudo iptables -I INPUT 1 -s 198.51.100.42 -j DROP

# Vérification
sudo iptables -L INPUT -n --line-numbers
# La règle apparaît en position 1
```

### Différences entre -A et -I

| Critère | -A (append) | -I (insert) |
|---------|-------------|-------------|
| **Position** | Fin de chaîne | Début ou position spécifique |
| **Priorité** | Faible | Élevée |
| **Usage** | Construction progressive | Ajout urgent ou correction |
| **Risque** | Règle ignorée si ordre mal pensé | Peut casser la logique existante |
| **Vitesse d'exécution** | Plus rapide | Légèrement plus lente |

> [!info] Recommandation
> - **-A** : pour construire méthodiquement un pare-feu
> - **-I** : pour les interventions d'urgence ou les règles prioritaires

---

## ❌ Supprimer des règles

### Option -D (--delete)

L'option `-D` supprime une règle spécifique d'une chaîne.

#### Méthode 1 : Suppression par spécification exacte

```bash
# Syntaxe : reproduire exactement la règle à supprimer
sudo iptables -D <CHAINE> [spécification complète]
```

```bash
# Exemple : supprimer une règle précise
# D'abord, lister pour voir la règle
sudo iptables -L INPUT -n --line-numbers

# Supprimer en reproduisant la règle exactement
sudo iptables -D INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
```

> [!warning] Piège courant
> La spécification doit être **exactement identique** à la règle existante (même ordre des options, mêmes valeurs).

#### Méthode 2 : Suppression par numéro de ligne (recommandée)

```bash
# Syntaxe : indiquer le numéro de ligne
sudo iptables -D <CHAINE> <NUMERO>
```

```bash
# 1. Lister avec numéros de ligne
sudo iptables -L INPUT -n --line-numbers
# Chain INPUT (policy ACCEPT)
# num  target     prot opt source               destination
# 1    ACCEPT     tcp  --  192.168.1.0/24       0.0.0.0/0            tcp dpt:22
# 2    DROP       all  --  10.0.0.0/8           0.0.0.0/0
# 3    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80

# 2. Supprimer la règle numéro 2
sudo iptables -D INPUT 2

# 3. Vérifier
sudo iptables -L INPUT -n --line-numbers
```

> [!tip] Bonne pratique
> Toujours utiliser `--line-numbers` et supprimer par numéro : c'est plus sûr et plus rapide.

#### Exemples de suppressions courantes

```bash
# Supprimer une règle de blocage d'IP
sudo iptables -D INPUT -s 203.0.113.50 -j DROP

# Supprimer une règle NAT (table nat)
sudo iptables -t nat -D POSTROUTING 1

# Supprimer toutes les occurrences d'une règle (boucle)
while sudo iptables -D INPUT -p tcp --dport 8080 -j ACCEPT 2>/dev/null; do :; done
```

### Option -F (--flush)

L'option `-F` **vide complètement** une ou toutes les chaînes.

#### Syntaxe

```bash
# Vider toutes les chaînes de la table filter
sudo iptables -F

# Vider une chaîne spécifique
sudo iptables -F <CHAINE>

# Vider toutes les chaînes d'une table spécifique
sudo iptables -t nat -F
```

#### Exemples pratiques

```bash
# Vider la chaîne INPUT
sudo iptables -F INPUT

# Vider toutes les chaînes (filter)
sudo iptables -F

# Vider la table NAT complète
sudo iptables -t nat -F

# Vider toutes les tables
sudo iptables -t filter -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -t raw -F
```

> [!warning] Danger critique
> **`iptables -F` ne supprime PAS les policies !** Si votre policy INPUT est DROP, vous perdez l'accès après un flush !

#### Flush sécurisé

```bash
# ❌ DANGEREUX en SSH distant
sudo iptables -F  # Si policy INPUT=DROP, connexion coupée !

# ✅ SÉCURISÉ : réinitialisation complète
sudo iptables -P INPUT ACCEPT    # D'abord, policy permissive
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -F                  # Ensuite, flush
sudo iptables -X                  # Supprimer chaînes custom
```

### Suppression sécurisée

#### Script de suppression sûre

```bash
#!/bin/bash
# reset-iptables-safe.sh

echo "🛡️ Réinitialisation sécurisée d'iptables"

# 1. Sauvegarder la config actuelle
echo "📦 Sauvegarde de la configuration actuelle..."
sudo iptables-save > /tmp/iptables-backup-$(date +%Y%m%d-%H%M%S).rules

# 2. Mettre toutes les policies en ACCEPT
echo "✅ Mise en place de policies permissives..."
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# 3. Flush toutes les tables
echo "🧹 Nettoyage des règles..."
sudo iptables -t filter -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -t raw -F

# 4. Supprimer les chaînes personnalisées
echo "🗑️ Suppression des chaînes personnalisées..."
sudo iptables -t filter -X
sudo iptables -t nat -X
sudo iptables -t mangle -X
sudo iptables -t raw -X

# 5. Réinitialiser les compteurs
echo "0️⃣ Réinitialisation des compteurs..."
sudo iptables -Z

echo "✅ Réinitialisation terminée - Pare-feu vide avec policies ACCEPT"
```

> [!tip] Conseil professionnel
> Toujours tester les suppressions importantes dans un environnement de test ou utiliser un script avec rollback automatique.

---

## 💾 Sauvegarde et restauration

### iptables-save

La commande `iptables-save` exporte toutes les règles actives dans un format réutilisable.

#### Syntaxe et utilisation

```bash
# Afficher les règles au format save
sudo iptables-save

# Sauvegarder dans un fichier
sudo iptables-save > /etc/iptables/rules.v4

# Sauvegarder avec horodatage
sudo iptables-save > /root/iptables-backup-$(date +%Y%m%d-%H%M%S).rules

# Sauvegarder une table spécifique
sudo iptables-save -t nat > /etc/iptables/nat-rules.v4
```

#### Format de sortie

```bash
# Exemple de sortie iptables-save
*filter
:INPUT ACCEPT [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -j DROP
COMMIT

*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o eth0 -j MASQUERADE
COMMIT
```

> [!info] Structure du fichier
> - Commence par `*<table>` (filter, nat, mangle, raw)
> - Définit les policies avec `:CHAINE POLICY`
> - Liste les règles avec `-A`
> - Se termine par `COMMIT`

#### Sauvegarde complète automatisée

```bash
#!/bin/bash
# backup-iptables.sh

BACKUP_DIR="/root/iptables-backups"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="$BACKUP_DIR/iptables-$TIMESTAMP.rules"

# Créer le répertoire si nécessaire
mkdir -p "$BACKUP_DIR"

# Sauvegarder
sudo iptables-save > "$BACKUP_FILE"

# Vérifier
if [ $? -eq 0 ]; then
    echo "✅ Sauvegarde réussie : $BACKUP_FILE"
    # Garder seulement les 10 dernières sauvegardes
    ls -t "$BACKUP_DIR"/iptables-*.rules | tail -n +11 | xargs rm -f
else
    echo "❌ Erreur de sauvegarde"
    exit 1
fi
```

### iptables-restore

La commande `iptables-restore` charge des règles depuis un fichier.

#### Syntaxe et options

```bash
# Restaurer depuis un fichier
sudo iptables-restore < /etc/iptables/rules.v4

# Restaurer sans vider les règles existantes (-n, --noflush)
sudo iptables-restore -n < /etc/iptables/rules.v4

# Tester sans appliquer (-t, --test)
sudo iptables-restore -t < /etc/iptables/rules.v4

# Restaurer avec verbosité
sudo iptables-restore -v < /etc/iptables/rules.v4
```

#### Différence entre restauration avec/sans flush

```bash
# AVEC flush (par défaut) : remplace complètement les règles
sudo iptables-restore < rules.v4
# Équivalent à : flush + chargement des nouvelles règles

# SANS flush (-n) : ajoute aux règles existantes
sudo iptables-restore -n < rules.v4
# Les anciennes règles restent, les nouvelles s'ajoutent
```

> [!warning] Attention au flush
> Par défaut, `iptables-restore` vide les règles existantes. Utilisez `-n` si vous voulez conserver les règles actuelles.

#### Script de restauration sécurisée

```bash
#!/bin/bash
# restore-iptables-safe.sh

RULES_FILE="/etc/iptables/rules.v4"
TIMEOUT=60  # Secondes avant rollback

# Vérifier que le fichier existe
if [ ! -f "$RULES_FILE" ]; then
    echo "❌ Fichier $RULES_FILE introuvable"
    exit 1
fi

# Tester la syntaxe
echo "🔍 Test de syntaxe..."
sudo iptables-restore -t < "$RULES_FILE"
if [ $? -ne 0 ]; then
    echo "❌ Erreur de syntaxe dans le fichier"
    exit 1
fi

# Sauvegarder la config actuelle
echo "💾 Sauvegarde de la configuration actuelle..."
sudo iptables-save > /tmp/iptables-backup-before-restore.rules

# Appliquer les nouvelles règles
echo "⚙️ Application des nouvelles règles..."
sudo iptables-restore < "$RULES_FILE"

# Demander confirmation
echo "✅ Règles appliquées. Vérifiez que tout fonctionne."
echo "⏰ Confirmation requise dans ${TIMEOUT}s (sinon rollback automatique)"
read -t $TIMEOUT -p "Confirmer ? (y/N) : " confirm

if [[ "$confirm" =~ ^[Yy]$ ]]; then
    echo "✅ Configuration confirmée et conservée"
    rm -f /tmp/iptables-backup-before-restore.rules
else
    echo "⏪ Rollback vers la configuration précédente..."
    sudo iptables-restore < /tmp/iptables-backup-before-restore.rules
    echo "✅ Configuration précédente restaurée"
fi
```

### Persistance des règles

Par défaut, les règles iptables sont **volatiles** : elles disparaissent au redémarrage.

#### Sur Debian/Ubuntu avec iptables-persistent

```bash
# Installer le paquet
sudo apt update
sudo apt install iptables-persistent

# Sauvegarder les règles actuelles
sudo netfilter-persistent save

# Emplacement des fichiers
# IPv4 : /etc/iptables/rules.v4
# IPv6 : /etc/iptables/rules.v6

# Recharger les règles sauvegardées
sudo netfilter-persistent reload

# Au démarrage, les règles sont automatiquement chargées
```

#### Méthode manuelle avec systemd

```bash
# 1. Créer un service systemd
sudo nano /etc/systemd/system/iptables-restore.service
```

```ini
[Unit]
Description=Restore iptables rules
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /etc/iptables/rules.v4
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

```bash
# 2. Activer le service
sudo systemctl enable iptables-restore.service

# 3. Sauvegarder les règles actuelles
sudo iptables-save > /etc/iptables/rules.v4

# 4. Tester
sudo systemctl start iptables-restore.service
sudo systemctl status iptables-restore.service
```

#### Méthode avec cron (sauvegarde régulière)

```bash
# Ajouter une tâche cron pour sauvegarde automatique
sudo crontab -e
```

```bash
# Sauvegarder iptables tous les jours à 2h du matin
0 2 * * * /sbin/iptables-save > /etc/iptables/rules.v4

# Sauvegarder avec rotation
0 2 * * * /sbin/iptables-save > /root/iptables-backup-$(date +\%Y\%m\%d).rules && find /root -name "iptables-backup-*.rules" -mtime +7 -delete
```

> [!tip] Astuce pro
> Combinez `iptables-persistent` pour le chargement au boot et une sauvegarde cron régulière pour conserver l'historique.

---

## ✅ Bonnes pratiques

### 1. Toujours sauvegarder avant modification

```bash
# Avant toute intervention majeure
sudo iptables-save > /tmp/iptables-backup-$(date +%Y%m%d-%H%M%S).rules
```

### 2. Utiliser des scripts avec rollback

```bash
#!/bin/bash
# apply-rules-safe.sh

# Sauvegarde
iptables-save > /tmp/backup.rules

# Application
iptables-restore < nouvelles-rules.v4

# Attendre confirmation (60s)
read -t 60 -p "OK? (y/N) " confirm
[[ "$confirm" != "y" ]] && iptables-restore < /tmp/backup.rules
```

### 3. Commenter les règles complexes

```bash
# Utiliser -m comment pour documenter
sudo iptables -A INPUT -s 10.0.0.0/8 -j DROP -m comment --comment "Blocage réseau interne de test"

# Affichage
sudo iptables -L -n -v
# Affiche le commentaire dans la sortie
```

### 4. Tester en environnement isolé

```bash
# Utiliser une VM ou un conteneur pour tester
# Ne jamais tester en production directement
```

### 5. Documenter les changements

```bash
# Tenir un journal des modifications
echo "$(date) - Ajout règle SSH depuis 192.168.1.0/24" >> /var/log/iptables-changes.log
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
```

### 6. Utiliser des chaînes personnalisées

```bash
# Créer des chaînes pour organiser les règles
sudo iptables -N LOGGING
sudo iptables -A INPUT -j LOGGING
sudo iptables -A LOGGING -j LOG --log-prefix "IPTables: "
sudo iptables -A LOGGING -j DROP

# Plus lisible et maintenable
```

### 7. Ordre optimal des règles

```bash
# 1. Autoriser loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# 2. Autoriser trafic établi
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 3. Règles spécifiques (services)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 4. Logging (optionnel)
sudo iptables -A INPUT -j LOG --log-prefix "INPUT-DROP: "

# 5. Policy par défaut
sudo iptables -A INPUT -j DROP
```

### 8. Vérifications systématiques

```bash
# Après chaque modification
sudo iptables -L -n -v --line-numbers

# Vérifier les compteurs
sudo iptables -L -v -n | grep -E "(Chain|pkts)"

# Tester la connectivité
ping -c 3 8.8.8.8
curl -I https://www.google.com
```

### 9. Protection contre les erreurs SSH

```bash
# Méthode 1 : Ajouter une règle temporaire permissive
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
# Faire les modifs
# Supprimer la règle temporaire
sudo iptables -D INPUT 1

# Méthode 2 : Utiliser at pour un rollback automatique
echo "iptables-restore < /tmp/backup.rules" | sudo at now + 5 minutes
# Si connexion perdue, restauration auto dans 5 min
```

### 10. Monitoring et alertes

```bash
# Script de surveillance des règles critiques
#!/bin/bash
# check-iptables.sh

RULES_COUNT=$(sudo iptables -L INPUT -n | grep -c "ACCEPT.*tcp dpt:22")

if [ $RULES_COUNT -eq 0 ]; then
    echo "⚠️ ALERTE : Aucune règle SSH détectée !"
    # Envoyer alerte
    mail -s "Alerte iptables" admin@example.com <<< "Règle SSH manquante"
fi
```

---

> [!info] Résumé des commandes essentielles
> - **Lister** : `iptables -L -n -v --line-numbers`
> - **Lister (format save)** : `iptables -S`
> - **Ajouter** : `iptables -A <CHAINE> [règle]`
> - **Insérer** : `iptables -I <CHAINE> [position] [règle]`
> - **Supprimer** : `iptables -D <CHAINE> <numero>`
> - **Vider** : `iptables -F` (+ mettre policies ACCEPT avant !)
> - **Sauvegarder** : `iptables-save > fichier.rules`
> - **Restaurer** : `iptables-restore < fichier.rules`

> [!warning] Rappel sécurité
> Ne jamais exécuter `iptables -F` sur un serveur distant sans avoir d'abord mis les policies en ACCEPT. Vous risquez de perdre définitivement l'accès !
