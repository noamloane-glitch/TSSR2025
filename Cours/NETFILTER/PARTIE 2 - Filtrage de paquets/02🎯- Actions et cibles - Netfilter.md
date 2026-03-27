
## 📚 Table des matières

- [Introduction aux actions](#introduction-aux-actions)
- [Actions de base](#actions-de-base)
  - [ACCEPT - Autoriser](#accept---autoriser)
  - [DROP - Rejeter silencieusement](#drop---rejeter-silencieusement)
  - [REJECT - Rejeter avec notification](#reject---rejeter-avec-notification)
- [Comparaison ACCEPT, DROP, REJECT](#comparaison-accept-drop-reject)
- [LOG - Journalisation](#log---journalisation)
  - [Syntaxe et options](#syntaxe-et-options)
  - [Niveaux de log](#niveaux-de-log)
  - [Limitation du logging](#limitation-du-logging)
  - [Analyse des logs](#analyse-des-logs)
- [Suivi des paquets](#suivi-des-paquets)
  - [ULOG - Logging avancé](#ulog---logging-avancé)
  - [NFLOG - Logging moderne](#nflog---logging-moderne)
  - [TRACE - Débogage](#trace---débogage)
- [Cibles personnalisées](#cibles-personnalisées)
  - [Créer une chaîne personnalisée](#créer-une-chaîne-personnalisée)
  - [Utiliser une chaîne personnalisée](#utiliser-une-chaîne-personnalisée)
  - [RETURN - Retour à la chaîne parente](#return---retour-à-la-chaîne-parente)
  - [Cas d'usage avancés](#cas-dusage-avancés)
- [Politique par défaut](#politique-par-défaut)
  - [Philosophies de filtrage](#philosophies-de-filtrage)
  - [Définir les politiques](#définir-les-politiques)
  - [Politiques recommandées](#politiques-recommandées)
- [Actions spéciales](#actions-spéciales)
- [Pièges courants](#pièges-courants)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction aux actions

Dans Netfilter/iptables, une **action** (ou **cible**, en anglais *target*) définit **ce qui arrive à un paquet** lorsqu'il correspond à une règle.

> [!info] Concept fondamental
> Une règle = **Critères de correspondance** + **Action**
> ```bash
> iptables -A INPUT -p tcp --dport 80 -j ACCEPT
> #                 ↑                    ↑
> #              critères             action
> ```

L'option `-j` (jump) spécifie l'action à exécuter. Le terme "jump" vient du fait que le traitement du paquet "saute" vers cette cible.

---

## Actions de base

### ACCEPT - Autoriser

`ACCEPT` **autorise le paquet** à continuer son chemin dans le système.

```bash
# Autoriser le trafic HTTP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Autoriser tout le trafic depuis une IP de confiance
iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Autoriser le ping (ICMP)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
```

**Comportement :**
- Le paquet est **immédiatement accepté**
- Il **ne passe plus par les règles suivantes** de la même chaîne
- Il continue vers sa destination

> [!tip] Optimisation
> Placez les règles ACCEPT les plus fréquentes en haut de la chaîne pour améliorer les performances.

---

### DROP - Rejeter silencieusement

`DROP` **rejette le paquet silencieusement**, sans aucune notification à l'émetteur.

```bash
# Bloquer le trafic Telnet
iptables -A INPUT -p tcp --dport 23 -j DROP

# Bloquer une IP malveillante
iptables -A INPUT -s 198.51.100.50 -j DROP

# Bloquer tout le trafic non autorisé (en fin de chaîne)
iptables -A INPUT -j DROP
```

**Comportement :**
- Le paquet est **supprimé** immédiatement
- **Aucune réponse** n'est envoyée à l'émetteur
- L'émetteur attend jusqu'au **timeout** de connexion

> [!warning] Impact sur l'émetteur
> Avec DROP, l'application émettrice doit attendre le timeout complet (peut être long : 30s-2min pour TCP). Cela peut donner l'impression que le système est lent ou planté.

**Quand utiliser DROP :**
- ✅ Pour bloquer du trafic malveillant (scans de ports, attaques)
- ✅ Quand vous voulez rester "furtif" (ne pas révéler qu'un firewall existe)
- ✅ Pour décourager les scanners automatiques
- ❌ Pas pour du trafic légitime (mauvaise expérience utilisateur)

---

### REJECT - Rejeter avec notification

`REJECT` **rejette le paquet** mais **envoie une notification** d'erreur à l'émetteur.

```bash
# Rejeter avec message ICMP par défaut
iptables -A INPUT -p tcp --dport 23 -j REJECT

# Rejeter avec un type de message spécifique
iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with icmp-port-unreachable

# Rejeter TCP avec un RST
iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset

# Rejeter avec un message ICMP personnalisé
iptables -A INPUT -p udp -j REJECT --reject-with icmp-host-unreachable
```

**Types de messages REJECT :**

| Option | Protocole | Message envoyé | Usage |
|--------|-----------|----------------|-------|
| `icmp-net-unreachable` | Tous | Réseau inaccessible | Simuler un problème réseau |
| `icmp-host-unreachable` | Tous | Hôte inaccessible | Simuler un hôte éteint |
| `icmp-port-unreachable` | Tous | Port inaccessible (défaut) | Service non disponible |
| `icmp-proto-unreachable` | Tous | Protocole non supporté | Rare |
| `icmp-net-prohibited` | Tous | Réseau interdit (admin) | Filtrage administratif |
| `icmp-host-prohibited` | Tous | Hôte interdit (admin) | Filtrage administratif |
| `icmp-admin-prohibited` | Tous | Communication interdite | Politique de sécurité |
| `tcp-reset` | TCP uniquement | Paquet RST TCP | Fermeture propre TCP |

**Comportement :**
- Le paquet est **supprimé**
- Une **réponse d'erreur** est envoyée à l'émetteur
- L'émetteur reçoit un **échec immédiat** (pas de timeout)

> [!example] Différence pratique
> ```bash
> # DROP : telnet attend 60 secondes avant "Connection timed out"
> iptables -A INPUT -p tcp --dport 23 -j DROP
> 
> # REJECT : telnet échoue immédiatement avec "Connection refused"
> iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset
> ```

**Quand utiliser REJECT :**
- ✅ Pour du trafic légitime mais non autorisé
- ✅ Quand vous voulez un feedback rapide (debugging, développement)
- ✅ Pour être "poli" avec les utilisateurs légitimes
- ❌ Pas pour du trafic d'attaque (éviter de révéler des informations)

---

## Comparaison ACCEPT, DROP, REJECT

| Critère | ACCEPT | DROP | REJECT |
|---------|--------|------|--------|
| **Paquet** | Autorisé | Supprimé | Supprimé |
| **Réponse** | Aucune (normal) | Aucune | Message d'erreur |
| **Délai émetteur** | Immédiat | Timeout complet | Immédiat |
| **Furtivité** | N/A | Maximale | Faible |
| **Debug friendly** | N/A | Non | Oui |
| **Usage typique** | Trafic autorisé | Trafic malveillant | Trafic refusé |
| **Bande passante** | N/A | Minimale | Légèrement plus élevée |

> [!tip] Règle générale
> ```bash
> # Trafic légitime → ACCEPT
> iptables -A INPUT -p tcp --dport 443 -j ACCEPT
> 
> # Attaques, scans → DROP
> iptables -A INPUT -p tcp --dport 1:1023 -j DROP
> 
> # Refus poli → REJECT
> iptables -A INPUT -p tcp --dport 8080 -j REJECT --reject-with tcp-reset
> ```

---

## LOG - Journalisation

L'action `LOG` **enregistre les informations du paquet** dans les logs système sans bloquer le paquet.

> [!warning] LOG n'est PAS terminal
> Contrairement à ACCEPT/DROP/REJECT, LOG **ne termine pas** le traitement du paquet. Le paquet continue vers les règles suivantes.

### Syntaxe et options

```bash
# LOG simple
iptables -A INPUT -j LOG

# LOG avec préfixe personnalisé
iptables -A INPUT -j LOG --log-prefix "FIREWALL-INPUT: "

# LOG avec niveau de gravité
iptables -A INPUT -j LOG --log-level 4 --log-prefix "FW: "

# LOG avec options complètes
iptables -A INPUT -j LOG \
  --log-prefix "DROP-INPUT: " \
  --log-level warning \
  --log-tcp-options \
  --log-ip-options

# Pattern typique : LOG puis DROP
iptables -A INPUT -p tcp --dport 23 -j LOG --log-prefix "TELNET-BLOCK: "
iptables -A INPUT -p tcp --dport 23 -j DROP
```

**Options de LOG :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--log-prefix` | Préfixe personnalisé (max 29 caractères) | `"SSH-DENIED: "` |
| `--log-level` | Niveau de sévérité (0-7 ou nom) | `4` ou `warning` |
| `--log-tcp-sequence` | Inclut les numéros de séquence TCP | - |
| `--log-tcp-options` | Inclut les options TCP | - |
| `--log-ip-options` | Inclut les options IP | - |
| `--log-uid` | Inclut l'UID du processus émetteur (local) | - |

> [!tip] Préfixe lisible
> Utilisez des préfixes cohérents et descriptifs pour faciliter l'analyse :
> ```bash
> --log-prefix "FW-INPUT-DROP: "
> --log-prefix "FW-FORWARD-BLOCK: "
> --log-prefix "SSH-BRUTE: "
> ```

---

### Niveaux de log

Les niveaux de log correspondent aux niveaux syslog standard :

| Niveau | Nom | Valeur | Signification | Usage |
|--------|-----|--------|---------------|-------|
| 0 | `emerg` | Urgence | Système inutilisable | Jamais pour firewall |
| 1 | `alert` | Alerte | Action immédiate requise | Attaques critiques |
| 2 | `crit` | Critique | Conditions critiques | Violations graves |
| 3 | `err` | Erreur | Erreurs | Tentatives suspectes |
| 4 | `warning` | Avertissement | Avertissements | **Défaut, usage général** |
| 5 | `notice` | Notice | Normal mais significatif | Événements notables |
| 6 | `info` | Information | Informatif | Debugging détaillé |
| 7 | `debug` | Debug | Messages de debug | Debugging verbose |

```bash
# Niveau par défaut (warning/4)
iptables -A INPUT -j LOG --log-prefix "FW: "

# Niveau critique pour attaques
iptables -A INPUT -p tcp --syn --dport 22 -m recent --name SSH --update --seconds 60 --hitcount 5 \
  -j LOG --log-level crit --log-prefix "SSH-BRUTE-FORCE: "

# Niveau debug pour troubleshooting
iptables -A FORWARD -j LOG --log-level debug --log-prefix "FW-DEBUG: "
```

> [!info] Configuration syslog
> Les logs apparaissent dans `/var/log/syslog` ou `/var/log/messages` selon la distribution. Vous pouvez configurer rsyslog pour les rediriger :
> ```bash
> # /etc/rsyslog.d/10-iptables.conf
> :msg,contains,"FW-" /var/log/firewall.log
> & stop
> ```

---

### Limitation du logging

> [!warning] Danger : Saturation des logs
> Sans limitation, le logging peut :
> - **Remplir le disque** rapidement
> - **Ralentir le système** (I/O intensif)
> - Faciliter les **attaques DoS** (flooding de logs)

**Solution : Module `limit`**

```bash
# Limiter à 5 logs par minute
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "INPUT: "

# Avec burst (pic initial)
iptables -A INPUT -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "INPUT: "

# Exemple complet : LOG limité puis DROP
iptables -A INPUT -p tcp --dport 23 \
  -m limit --limit 3/min --limit-burst 5 \
  -j LOG --log-prefix "TELNET-BLOCK: " --log-level notice
iptables -A INPUT -p tcp --dport 23 -j DROP
```

**Paramètres de `limit` :**

| Paramètre | Description | Valeurs possibles | Défaut |
|-----------|-------------|-------------------|--------|
| `--limit` | Taux moyen | `n/second`, `n/minute`, `n/hour`, `n/day` | `3/hour` |
| `--limit-burst` | Pic initial autorisé | Nombre entier | `5` |

**Fonctionnement :**
1. Permet un **burst initial** (ex: 10 paquets)
2. Puis **recharge** au taux spécifié (ex: 5/minute)
3. Les paquets au-delà sont ignorés (ne matchent pas la règle)

```bash
# Burst = 10, recharge = 5/min
# T=0s  : 10 paquets loggés (burst)
# T=12s : 1 paquet loggé (rechargé)
# T=24s : 1 paquet loggé (rechargé)
# ...
```

---

### Analyse des logs

**Format typique d'un log iptables :**

```
Feb 12 14:32:15 serveur kernel: [12345.678] FW-INPUT-DROP: IN=eth0 OUT= MAC=00:11:22:33:44:55:66:77:88:99:aa:bb:08:00 SRC=203.0.113.50 DST=192.168.1.100 LEN=60 TOS=0x00 PREC=0x00 TTL=54 ID=12345 DF PROTO=TCP SPT=54321 DPT=22 WINDOW=5840 RES=0x00 SYN URGP=0
```

**Champs importants :**

| Champ | Signification | Exemple |
|-------|---------------|---------|
| `FW-INPUT-DROP:` | Préfixe personnalisé | Votre identifiant |
| `IN=eth0` | Interface entrante | `eth0`, `wlan0` |
| `OUT=eth1` | Interface sortante | (vide pour INPUT) |
| `SRC=` | IP source | `203.0.113.50` |
| `DST=` | IP destination | `192.168.1.100` |
| `PROTO=TCP` | Protocole | `TCP`, `UDP`, `ICMP` |
| `SPT=54321` | Port source | Port éphémère client |
| `DPT=22` | Port destination | Service visé |
| `SYN` | Flags TCP | `SYN`, `ACK`, `FIN`, etc. |

**Commandes d'analyse utiles :**

```bash
# Suivre les logs en temps réel
tail -f /var/log/syslog | grep "FW-"

# Compter les tentatives par IP source
grep "SSH-BRUTE" /var/log/firewall.log | awk '{print $10}' | sort | uniq -c | sort -rn

# Top 10 des IP bloquées
grep "DROP" /var/log/firewall.log | grep -oP 'SRC=\K[0-9.]+' | sort | uniq -c | sort -rn | head -10

# Analyser les ports ciblés
grep "DROP" /var/log/firewall.log | grep -oP 'DPT=\K[0-9]+' | sort | uniq -c | sort -rn

# Logs des dernières 24h
journalctl --since "24 hours ago" | grep "iptables"
```

> [!tip] Outils d'analyse
> Pour une analyse avancée, utilisez :
> - **fail2ban** : bannissement automatique d'IPs
> - **logwatch** : rapports quotidiens
> - **goaccess** : analyse visuelle (si logs HTTP)
> - **awstats** : statistiques détaillées

---

## Suivi des paquets

### ULOG - Logging avancé

`ULOG` (Userspace Logging) permet d'envoyer les paquets vers un daemon userspace pour un traitement personnalisé.

> [!warning] Obsolète
> ULOG est **déprécié** depuis le kernel 3.17. Utilisez NFLOG à la place.

```bash
# ULOG basique (legacy)
iptables -A INPUT -j ULOG --ulog-nlgroup 1 --ulog-prefix "FW: "
```

---

### NFLOG - Logging moderne

`NFLOG` (Netfilter Logging) est le remplaçant moderne d'ULOG.

```bash
# NFLOG vers groupe 1
iptables -A INPUT -p tcp --dport 80 -j NFLOG --nflog-group 1

# NFLOG avec préfixe
iptables -A INPUT -j NFLOG --nflog-group 2 --nflog-prefix "HTTP: "

# NFLOG avec limitation de taille
iptables -A INPUT -j NFLOG --nflog-group 1 --nflog-range 256

# NFLOG avec seuil de paquets
iptables -A INPUT -j NFLOG --nflog-group 1 --nflog-threshold 10
```

**Options NFLOG :**

| Option | Description | Défaut |
|--------|-------------|--------|
| `--nflog-group` | Numéro de groupe (0-65535) | 0 |
| `--nflog-prefix` | Préfixe (max 64 caractères) | - |
| `--nflog-range` | Nombre d'octets à capturer | Paquet entier |
| `--nflog-threshold` | Envoyer par lots de N paquets | 1 |

**Utilisation avec ulogd2 :**

```bash
# Installer ulogd2
apt-get install ulogd2

# Configuration dans /etc/ulogd.conf
[nflog1]
group=1
netlink_resync_timeout=60

# Règle iptables
iptables -A INPUT -j NFLOG --nflog-group 1 --nflog-prefix "TRACKED: "

# Démarrer ulogd2
systemctl start ulogd2
```

> [!tip] Avantages de NFLOG
> - Capture complète des paquets (pcap possible)
> - Traitement userspace personnalisé
> - Moins de charge kernel que LOG
> - Export vers bases de données, SIEM, etc.

---

### TRACE - Débogage

`TRACE` permet de **tracer le parcours complet d'un paquet** à travers toutes les tables et chaînes.

> [!warning] Debugging uniquement
> TRACE génère énormément de logs. **Jamais en production !**

```bash
# Activer le module
modprobe nf_log_ipv4

# Tracer un paquet spécifique (table raw)
iptables -t raw -A PREROUTING -p tcp --dport 80 -s 192.168.1.100 -j TRACE

# Activer le logging de trace
sysctl net.netfilter.nf_log.2=nf_log_ipv4

# Voir les traces
tail -f /var/log/syslog | grep TRACE
```

**Exemple de sortie TRACE :**

```
TRACE: raw:PREROUTING:policy:2 IN=eth0 SRC=192.168.1.100 DST=192.168.1.1
TRACE: mangle:PREROUTING:policy:1 IN=eth0 SRC=192.168.1.100 DST=192.168.1.1
TRACE: nat:PREROUTING:policy:1 IN=eth0 SRC=192.168.1.100 DST=192.168.1.1
TRACE: filter:INPUT:rule:3 IN=eth0 SRC=192.168.1.100 DST=192.168.1.1
```

> [!tip] Cas d'usage
> Utilisez TRACE quand :
> - Une règle ne fonctionne pas comme prévu
> - Vous devez comprendre le chemin exact d'un paquet
> - Vous débuggez des interactions complexes entre tables

---

## Cibles personnalisées

Les **chaînes personnalisées** (custom chains) permettent d'organiser et de réutiliser des ensembles de règles.

> [!info] Pourquoi créer des chaînes ?
> - **Organisation** : grouper des règles par fonction
> - **Réutilisation** : éviter la duplication
> - **Performance** : sauter des groupes de règles non applicables
> - **Lisibilité** : code plus clair et maintenable

### Créer une chaîne personnalisée

```bash
# Créer une chaîne vide
iptables -N LOG_AND_DROP

# Créer une chaîne pour SSH
iptables -N SSH_RULES

# Créer une chaîne pour filtrage web
iptables -N WEB_FILTER
```

**Commandes de gestion :**

```bash
# Lister une chaîne
iptables -L LOG_AND_DROP -n -v

# Vider une chaîne
iptables -F LOG_AND_DROP

# Supprimer une chaîne (doit être vide et non référencée)
iptables -X LOG_AND_DROP

# Renommer une chaîne
iptables -E OLD_NAME NEW_NAME
```

> [!warning] Suppression impossible
> Vous ne pouvez pas supprimer une chaîne :
> - Qui contient des règles → Videz-la d'abord avec `-F`
> - Qui est référencée par une autre règle → Supprimez les références

---

### Utiliser une chaîne personnalisée

```bash
# Créer la chaîne
iptables -N LOG_AND_DROP

# Ajouter des règles dans la chaîne
iptables -A LOG_AND_DROP -j LOG --log-prefix "DROPPED: " --log-level notice
iptables -A LOG_AND_DROP -j DROP

# Utiliser la chaîne depuis INPUT
iptables -A INPUT -p tcp --dport 23 -j LOG_AND_DROP
iptables -A INPUT -s 198.51.100.0/24 -j LOG_AND_DROP
```

**Exemple : Filtrage SSH avec fail2ban-like :**

```bash
# Créer la chaîne SSH
iptables -N SSH_FILTER

# Règles dans SSH_FILTER
iptables -A SSH_FILTER -m recent --name SSH --update --seconds 60 --hitcount 4 \
  -j LOG --log-prefix "SSH-BRUTE: "
iptables -A SSH_FILTER -m recent --name SSH --update --seconds 60 --hitcount 4 -j DROP
iptables -A SSH_FILTER -m recent --name SSH --set
iptables -A SSH_FILTER -j ACCEPT

# Rediriger le trafic SSH vers la chaîne
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j SSH_FILTER
```

**Exemple : Chaîne de logging centralisée :**

```bash
# Chaîne de log
iptables -N LOGGING

# Configuration du logging
iptables -A LOGGING -m limit --limit 5/min -j LOG --log-prefix "FW-DROP: " --log-level warning
iptables -A LOGGING -j DROP

# Utilisation
iptables -A INPUT -p tcp --dport 135:139 -j LOGGING  # Bloquer NetBIOS
iptables -A INPUT -p udp --dport 135:139 -j LOGGING
iptables -A INPUT -p tcp --dport 445 -j LOGGING      # Bloquer SMB
```

---

### RETURN - Retour à la chaîne parente

`RETURN` **termine le traitement dans la chaîne courante** et retourne à la chaîne qui a appelé celle-ci.

```bash
# Créer une chaîne
iptables -N TEST_CHAIN

# Règle avec RETURN
iptables -A TEST_CHAIN -s 192.168.1.100 -j RETURN  # Sortir si IP = .100
iptables -A TEST_CHAIN -j LOG --log-prefix "NOT-.100: "
iptables -A TEST_CHAIN -j DROP

# Appeler la chaîne
iptables -A INPUT -p tcp --dport 80 -j TEST_CHAIN
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

**Comportement :**
- Dans une chaîne personnalisée : retourne à la chaîne appelante
- Dans une chaîne built-in (INPUT, etc.) : applique la politique par défaut

> [!example] Utilisation pratique
> ```bash
> # Chaîne de whitelist
> iptables -N WHITELIST
> iptables -A WHITELIST -s 192.168.1.0/24 -j RETURN    # IPs locales OK
> iptables -A WHITELIST -s 10.0.0.100 -j RETURN        # IP admin OK
> iptables -A WHITELIST -j LOG --log-prefix "NOT-WHITELISTED: "
> iptables -A WHITELIST -j DROP
> 
> # Protéger un service
> iptables -A INPUT -p tcp --dport 3306 -j WHITELIST   # MySQL
> ```

---

### Cas d'usage avancés

**1. Organisation par protocole :**

```bash
# Chaînes par protocole
iptables -N TCP_RULES
iptables -N UDP_RULES
iptables -N ICMP_RULES

# Dispatcher
iptables -A INPUT -p tcp -j TCP_RULES
iptables -A INPUT -p udp -j UDP_RULES
iptables -A INPUT -p icmp -j ICMP_RULES

# Règles TCP
iptables -A TCP_RULES -p tcp --dport 22 -j ACCEPT
iptables -A TCP_RULES -p tcp --dport 80 -j ACCEPT
iptables -A TCP_RULES -p tcp --dport 443 -j ACCEPT

# Règles UDP
iptables -A UDP_RULES -p udp --dport 53 -j ACCEPT
iptables -A UDP_RULES -p udp --dport 123 -j ACCEPT
```

**2. Gestion d'attaques par service :**

```bash
# Protection web
iptables -N WEB_PROTECTION
iptables -A WEB_PROTECTION -m connlimit --connlimit-above 20 --connlimit-mask 32 -j REJECT
iptables -A WEB_PROTECTION -m recent --name WEB --update --seconds 10 --hitcount 30 -j DROP
iptables -A WEB_PROTECTION -m recent --name WEB --set
iptables -A WEB_PROTECTION -j ACCEPT

# Appliquer
iptables -A INPUT -p tcp --dport 80 -j WEB_PROTECTION
iptables -A INPUT -p tcp --dport 443 -j WEB_PROTECTION
```

**3. Zones de confiance :**

```bash
# Chaînes par zone
iptables -N ZONE_TRUSTED    # 192.168.1.0/24
iptables -N ZONE_DMZ        # 10.0.1.0/24
iptables -N ZONE_GUEST      # 172.16.0.0/24

# Règles TRUSTED : tout autorisé
iptables -A ZONE_TRUSTED -j ACCEPT

# Règles DMZ : services publics uniquement
iptables -A ZONE_DMZ -p tcp --dport 80 -j ACCEPT
iptables -A ZONE_DMZ -p tcp --dport 443 -j ACCEPT
iptables -A ZONE_DMZ -j DROP

# Règles GUEST : web uniquement
iptables -A ZONE_GUEST -p tcp --dport 80 -j ACCEPT
iptables -A ZONE_GUEST -p tcp --dport 443 -j ACCEPT
iptables -A ZONE_GUEST -p udp --dport 53 -j ACCEPT
iptables -A ZONE_GUEST -j DROP

# Router selon la source
iptables -A INPUT -s 192.168.1.0/24 -j ZONE_TRUSTED
iptables -A INPUT -s 10.0.1.0/24 -j ZONE_DMZ
iptables -A INPUT -s 172.16.0.0/24 -j ZONE_GUEST
```

---

## Politique par défaut

La **politique par défaut** (default policy) définit **l'action appliquée aux paquets qui ne matchent aucune règle**.

### Philosophies de filtrage

**1. Approche whitelist (restrictive) :**

```bash
# Bloquer par défaut, autoriser explicitement
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP  # Ou ACCEPT si moins strict

# Puis autoriser ce qui est nécessaire
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

✅ **Avantages :**
- Sécurité maximale par défaut
- Approche "fail-secure"
- Recommandé pour serveurs exposés

❌ **Inconvénients :**
- Peut casser des services si mal configuré
- Demande plus de travail initial

**2. Approche blacklist (permissive) :**

```bash
# Autoriser par défaut, bloquer explicitement
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# Puis bloquer ce qui est indésirable
iptables -A INPUT -p tcp --dport 23 -j DROP
iptables -A INPUT -s 198.51.100.0/24 -j DROP
```

✅ **Avantages :**
- Plus simple à mettre en place
- Moins de risque de se bloquer

❌ **Inconvénients :**
- Moins sécurisé
- Approche "fail-open"
- Non recommandé pour production

> [!warning] Recommandation de sécurité
> **Utilisez toujours l'approche whitelist** pour les systèmes en production. C'est le principe du "moindre privilège".

---

### Définir les politiques

```bash
# Syntaxe
iptables -P CHAÎNE ACTION

# Exemples
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

> [!warning] Attention : risque de blocage
> Si vous êtes connecté en SSH et que vous faites :
> ```bash
> iptables -P INPUT DROP  # Sans règle SSH → vous êtes bloqué !
> ```
> 
> **Ordre sécurisé :**
> ```bash
> # 1. D'abord autoriser SSH
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
> 
> # 2. PUIS changer la politique
> iptables -P INPUT DROP
> ```

**Script de sécurité avec auto-rollback :**

```bash
#!/bin/bash
# Sauvegarde actuelle
iptables-save > /tmp/iptables.backup

# Appliquer nouvelles règles
iptables -F
iptables -P INPUT ACCEPT  # Temporaire
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP

# Test avec timeout
echo "Règles appliquées. Test SSH dans 60 secondes..."
echo "Appuyez sur CTRL+C pour valider, sinon rollback automatique."
sleep 60

# Rollback
echo "Rollback..."
iptables-restore < /tmp/iptables.backup
```

---

### Politiques recommandées

**Serveur web public :**

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback et connexions établies
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Services publics
iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # SSH (limitez la source si possible)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # HTTPS

# Ping limité
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
```

**Routeur/Firewall :**

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP  # Contrôle strict du forwarding
iptables -P OUTPUT ACCEPT

# Management local uniquement
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i eth0 -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT

# Forwarding autorisé
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.1.0/24 -j ACCEPT
```

**Poste de travail :**

```bash
# Plus permissif
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Connexions entrantes uniquement si établies
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Bloquer les scans
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

---

## Actions spéciales

**QUEUE - File d'attente userspace :**

```bash
# Envoyer vers userspace (obsolète, voir NFQUEUE)
iptables -A INPUT -j QUEUE
```

**NFQUEUE - File d'attente moderne :**

```bash
# Envoyer vers une application userspace
iptables -A INPUT -p tcp --dport 80 -j NFQUEUE --queue-num 0

# Avec plage de queues (load balancing)
iptables -A INPUT -j NFQUEUE --queue-balance 0:3
```

**MARK - Marquage de paquets :**

```bash
# Marquer un paquet (pour routage avancé)
iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-mark 1

# Utiliser la marque
iptables -A FORWARD -m mark --mark 1 -j ACCEPT
```

> [!info] Usage avancé
> Ces actions sont utilisées pour :
> - IDS/IPS userspace (Snort, Suricata avec NFQUEUE)
> - QoS et routage avancé (MARK)
> - Analyse approfondie des paquets

---

## Pièges courants

### 1. LOG sans limitation

```bash
# ❌ DANGEREUX : peut saturer le disque
iptables -A INPUT -j LOG
iptables -A INPUT -j DROP

# ✅ CORRECT : logging limité
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "FW: "
iptables -A INPUT -j DROP
```

### 2. Oublier que LOG n'est pas terminal

```bash
# ❌ MAUVAIS : le paquet est loggé ET accepté (règle suivante)
iptables -A INPUT -p tcp --dport 23 -j LOG
iptables -A INPUT -j ACCEPT

# ✅ CORRECT : LOG puis DROP explicite
iptables -A INPUT -p tcp --dport 23 -j LOG --log-prefix "TELNET: "
iptables -A INPUT -p tcp --dport 23 -j DROP
```

### 3. Chaîne personnalisée sans RETURN/DROP final

```bash
# ❌ RISQUE : si aucune règle ne match, retourne à INPUT
iptables -N MY_CHAIN
iptables -A MY_CHAIN -s 192.168.1.100 -j ACCEPT
# Si source != .100, retourne à INPUT et continue

# ✅ CORRECT : action par défaut explicite
iptables -N MY_CHAIN
iptables -A MY_CHAIN -s 192.168.1.100 -j ACCEPT
iptables -A MY_CHAIN -j DROP  # Bloquer le reste
```

### 4. Confondre politique et règles

```bash
# ❌ La politique s'applique SI aucune règle ne match
iptables -P INPUT ACCEPT
iptables -A INPUT -j DROP  # Bloque TOUT (la politique n'est jamais atteinte)

# ✅ Cohérence règles/politique
iptables -P INPUT DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# Politique DROP appliquée au reste
```

### 5. DROP vs REJECT pour debugging

```bash
# ❌ Difficile à debugger : timeout long
iptables -A INPUT -p tcp --dport 8080 -j DROP

# ✅ Feedback immédiat pendant le debug
iptables -A INPUT -p tcp --dport 8080 -j REJECT --reject-with tcp-reset
```

### 6. Supprimer une chaîne référencée

```bash
# ❌ ERREUR
iptables -N MY_CHAIN
iptables -A INPUT -j MY_CHAIN
iptables -X MY_CHAIN  # FAIL: chain is in use

# ✅ CORRECT
iptables -D INPUT -j MY_CHAIN  # Supprimer la référence
iptables -F MY_CHAIN           # Vider la chaîne
iptables -X MY_CHAIN           # Supprimer la chaîne
```

---

## Bonnes pratiques

### 1. Toujours limiter le logging

```bash
# Pattern standard
iptables -A INPUT -m limit --limit 5/min --limit-burst 10 \
  -j LOG --log-prefix "FW-INPUT-DROP: " --log-level notice
iptables -A INPUT -j DROP
```

### 2. Utiliser des préfixes de log cohérents

```bash
# Convention de nommage
iptables -A INPUT -j LOG --log-prefix "FW-INPUT-DROP: "
iptables -A FORWARD -j LOG --log-prefix "FW-FORWARD-DROP: "
iptables -A OUTPUT -j LOG --log-prefix "FW-OUTPUT-DROP: "

# Ou par service
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH-ATTEMPT: "
iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "HTTP-ACCESS: "
```

### 3. Organiser avec des chaînes personnalisées

```bash
# Créer une structure claire
iptables -N ACCEPT_LOG
iptables -A ACCEPT_LOG -j LOG --log-prefix "ACCEPTED: " --log-level info
iptables -A ACCEPT_LOG -j ACCEPT

iptables -N DROP_LOG
iptables -A DROP_LOG -m limit --limit 5/min -j LOG --log-prefix "DROPPED: "
iptables -A DROP_LOG -j DROP

# Utiliser
iptables -A INPUT -p tcp --dport 22 -j ACCEPT_LOG
iptables -A INPUT -j DROP_LOG
```

### 4. Documenter avec des commentaires

```bash
# Module comment
iptables -A INPUT -p tcp --dport 22 -j ACCEPT \
  -m comment --comment "SSH access from anywhere"

iptables -A INPUT -s 10.0.0.0/8 -j DROP \
  -m comment --comment "Block private RFC1918 from WAN"
```

### 5. Politique whitelist systématique

```bash
# Toujours bloquer par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT  # Ou DROP si besoin de contrôle strict
```

### 6. Centraliser le logging dans un fichier dédié

```bash
# /etc/rsyslog.d/10-iptables.conf
:msg,contains,"FW-" /var/log/firewall.log
& stop

# Rotation des logs
# /etc/logrotate.d/firewall
/var/log/firewall.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root adm
    postrotate
        /usr/lib/rsyslog/rsyslog-rotate
    endscript
}
```

### 7. Tester les règles avant application

```bash
#!/bin/bash
# Script de test avec rollback automatique
BACKUP=/tmp/iptables.backup.$(date +%s)

# Sauvegarder
iptables-save > $BACKUP

# Appliquer
./mes-nouvelles-regles.sh

# Test
echo "Nouvelles règles actives. Test pendant 60s..."
read -t 60 -p "Appuyez sur ENTRÉE pour valider (timeout = rollback): " || {
    echo "Timeout - Rollback..."
    iptables-restore < $BACKUP
    exit 1
}

echo "Règles validées !"
```

### 8. Utiliser REJECT pour le développement, DROP pour la production

```bash
# Développement : feedback rapide
iptables -A INPUT -j REJECT --reject-with icmp-port-unreachable

# Production : furtivité
iptables -A INPUT -j DROP
```

---

> [!tip] 💡 Points clés à retenir
> 1. **ACCEPT** = autoriser, **DROP** = bloquer silencieusement, **REJECT** = bloquer avec notification
> 2. **LOG n'est PAS terminal** : le paquet continue après le log
> 3. **Toujours limiter le logging** avec `-m limit`
> 4. Les **chaînes personnalisées** organisent et réutilisent les règles
> 5. **Politique par défaut DROP** = approche sécurisée (whitelist)
> 6. **RETURN** retourne à la chaîne parente
> 7. **Préfixes de log cohérents** facilitent l'analyse
> 8. **Documenter avec `-m comment`** pour la maintenabilité
