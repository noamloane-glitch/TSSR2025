## 📚 Table des matières

- [Introduction](#introduction)
- [Correspondance d'états (state/conntrack)](#correspondance-détats-stateconntrack)
  - [Le suivi de connexion](#le-suivi-de-connexion)
  - [Module state (legacy)](#module-state-legacy)
  - [Module conntrack (moderne)](#module-conntrack-moderne)
  - [États des connexions](#états-des-connexions)
  - [Exemples pratiques](#exemples-pratiques)
  - [Optimisation des performances](#optimisation-des-performances)
- [Correspondance par interface](#correspondance-par-interface)
  - [Options -i et -o](#options--i-et--o)
  - [Interfaces physiques vs virtuelles](#interfaces-physiques-vs-virtuelles)
  - [Wildcards et patterns](#wildcards-et-patterns)
  - [Cas d'usage courants](#cas-dusage-courants)
- [Correspondance par plage d'adresses](#correspondance-par-plage-dadresses)
  - [Module iprange](#module-iprange)
  - [Plages de ports](#plages-de-ports)
  - [Comparaison avec CIDR](#comparaison-avec-cidr)
- [Modules de correspondance courants](#modules-de-correspondance-courants)
  - [Module multiport](#module-multiport)
  - [Module limit](#module-limit)
  - [Module recent](#module-recent)
  - [Module connlimit](#module-connlimit)
  - [Module owner](#module-owner)
  - [Module mac](#module-mac)
  - [Module string](#module-string)
  - [Module time](#module-time)
  - [Module comment](#module-comment)
  - [Module tcp (options TCP)](#module-tcp-options-tcp)
  - [Module icmp](#module-icmp)
- [Combinaison de modules](#combinaison-de-modules)
- [Pièges courants](#pièges-courants)
- [Bonnes pratiques](#bonnes-pratiques)

---

## Introduction

Les règles avancées de filtrage permettent de créer des **politiques de sécurité sophistiquées** en utilisant des critères de correspondance plus complexes que les simples IP/port/protocole.

> [!info] Pourquoi les règles avancées ?
> - **Sécurité renforcée** : filtrage contextuel basé sur l'état
> - **Performance** : autoriser les connexions établies sans re-vérifier
> - **Flexibilité** : critères multiples et conditions complexes
> - **Protection** : limitation de débit, détection d'abus

---

## Correspondance d'états (state/conntrack)

### Le suivi de connexion

Le **connection tracking** (conntrack) est le mécanisme du noyau Linux qui **suit l'état des connexions** réseau.

> [!info] Concept clé
> Au lieu de traiter chaque paquet indépendamment, conntrack comprend qu'un paquet fait partie d'une **conversation bidirectionnelle**.

```
Client → Serveur : SYN
Serveur → Client : SYN-ACK
Client → Serveur : ACK
↓
Connexion ESTABLISHED
```

**Avantages :**
- ✅ Performances : pas besoin de règles pour les paquets de retour
- ✅ Sécurité : empêche les paquets "orphelins" non sollicités
- ✅ Simplicité : une règle pour autoriser la connexion entière

---

### Module state (legacy)

Le module `state` est l'ancienne interface pour le suivi de connexion.

> [!warning] Déprécié
> Le module `state` est **obsolète** depuis iptables 1.4.20. Utilisez `conntrack` à la place.

```bash
# Syntaxe legacy (éviter)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

---

### Module conntrack (moderne)

Le module `conntrack` est l'interface **moderne et recommandée** pour le suivi de connexion.

```bash
# Syntaxe moderne
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Avec options supplémentaires
iptables -A INPUT -m conntrack --ctstate NEW --ctproto tcp -j ACCEPT
```

**Options principales :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--ctstate` | État(s) de connexion | `NEW,ESTABLISHED,RELATED` |
| `--ctproto` | Protocole de la connexion | `tcp`, `udp`, `icmp` |
| `--ctorigsrc` | IP source originale | `192.168.1.100` |
| `--ctorigdst` | IP destination originale | `10.0.0.1` |
| `--ctreplsrc` | IP source de la réponse | `10.0.0.1` |
| `--ctrepldst` | IP destination de la réponse | `192.168.1.100` |
| `--ctdir` | Direction du paquet | `ORIGINAL`, `REPLY` |
| `--ctstatus` | Statut de la connexion | `EXPECTED`, `SEEN_REPLY` |

---

### États des connexions

**États principaux :**

| État | Description | Quand ? | Exemple |
|------|-------------|---------|---------|
| `NEW` | Nouveau paquet initiant une connexion | Premier paquet (SYN TCP) | Requête HTTP initiale |
| `ESTABLISHED` | Paquet d'une connexion établie | Après handshake | Données HTTP |
| `RELATED` | Nouvelle connexion liée à une existante | FTP data, ICMP errors | FTP transfert de fichier |
| `INVALID` | Paquet ne correspondant à aucune connexion | Paquet malformé/suspect | Scan, paquet corrompu |
| `UNTRACKED` | Paquet non suivi (table raw) | Marqué NOTRACK | Trafic haute performance |

**Diagramme des états TCP :**

```
NEW (SYN) → ESTABLISHED (SYN-ACK, ACK, DATA) → FIN/RST → Fermée
                    ↓
              RELATED (par ex. ICMP error)
```

---

### Exemples pratiques

**1. Configuration de base (pattern universel) :**

```bash
# Autoriser les connexions établies et reliées
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Autoriser nouvelles connexions SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# Politique par défaut
iptables -P INPUT DROP
```

> [!tip] Performance
> Placez **toujours** la règle ESTABLISHED,RELATED en **première position** :
> - 90%+ du trafic est ESTABLISHED
> - Évite le traitement de toutes les autres règles
> - Gain de performance massif

**2. Bloquer les paquets invalides :**

```bash
# Bloquer les paquets suspects en premier
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Puis autoriser le trafic légitime
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m conntrack --ctstate NEW -p tcp --dport 80 -j ACCEPT
```

**3. FTP avec RELATED (mode passif) :**

```bash
# Charger le module FTP helper
modprobe nf_conntrack_ftp

# Autoriser connexion de contrôle FTP
iptables -A INPUT -p tcp --dport 21 -m conntrack --ctstate NEW -j ACCEPT

# Autoriser connexions de données (RELATED)
iptables -A INPUT -m conntrack --ctstate RELATED -j ACCEPT

# Connexions établies
iptables -A INPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

> [!info] Helper modules
> Certains protocoles (FTP, SIP, H.323) nécessitent des modules helpers :
> ```bash
> # Lister les helpers chargés
> cat /proc/net/nf_conntrack_expect
> 
> # Charger un helper
> modprobe nf_conntrack_ftp
> modprobe nf_conntrack_sip
> ```

**4. Autoriser ping entrant, bloquer sortant :**

```bash
# Autoriser ping ENTRANT (echo-request)
iptables -A INPUT -p icmp --icmp-type echo-request -m conntrack --ctstate NEW -j ACCEPT

# Autoriser réponses (echo-reply)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT

# Bloquer ping SORTANT (si politique OUTPUT = DROP)
iptables -A OUTPUT -p icmp --icmp-type echo-request -j DROP
iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

**5. Suivi par direction (ORIGINAL vs REPLY) :**

```bash
# Autoriser seulement si direction = ORIGINAL
iptables -A FORWARD -m conntrack --ctdir ORIGINAL --ctstate NEW -j ACCEPT

# Logger les paquets REPLY
iptables -A FORWARD -m conntrack --ctdir REPLY -j LOG --log-prefix "REPLY: "
```

**6. Filtrage basé sur l'IP originale (post-NAT) :**

```bash
# Autoriser connexions depuis un réseau spécifique (IP d'origine, pas NATée)
iptables -A INPUT -m conntrack --ctorigsrc 192.168.1.0/24 -j ACCEPT

# Bloquer destination originale
iptables -A FORWARD -m conntrack --ctorigdst 10.0.0.100 -j DROP
```

---

### Optimisation des performances

**1. Ordre optimal des règles :**

```bash
#!/bin/bash
# Règles optimisées pour performance

# 1. Loopback (trafic local intensif)
iptables -A INPUT -i lo -j ACCEPT

# 2. INVALID en premier (protection)
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# 3. ESTABLISHED,RELATED (90%+ du trafic)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# 4. Nouvelles connexions autorisées
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT

# 5. Politique par défaut
iptables -P INPUT DROP
```

**2. Table de connexions :**

```bash
# Voir la table de suivi
conntrack -L

# Statistiques
conntrack -S

# Nombre de connexions actives
cat /proc/sys/net/netfilter/nf_conntrack_count

# Maximum autorisé
cat /proc/sys/net/netfilter/nf_conntrack_max

# Augmenter la limite (serveur haute charge)
echo 131072 > /proc/sys/net/netfilter/nf_conntrack_max

# Réduire les timeouts (libérer plus vite)
echo 600 > /proc/sys/net/netfilter/nf_conntrack_tcp_timeout_established
```

**3. Désactiver le tracking pour haute performance :**

```bash
# Table raw : désactiver le tracking pour certains flux
iptables -t raw -A PREROUTING -p tcp --dport 80 -j NOTRACK
iptables -t raw -A OUTPUT -p tcp --sport 80 -j NOTRACK

# Puis accepter sans tracking
iptables -A INPUT -m conntrack --ctstate UNTRACKED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate UNTRACKED -j ACCEPT
```

> [!warning] NOTRACK et sécurité
> Désactiver le tracking **réduit la sécurité** :
> - Pas de protection contre les paquets orphelins
> - Pas de détection de connexions suspectes
> - À utiliser uniquement pour du trafic de confiance haute performance

---

## Correspondance par interface

### Options -i et -o

Les options `-i` (input interface) et `-o` (output interface) filtrent selon l'**interface réseau**.

```bash
# -i : interface ENTRANTE (INPUT, FORWARD, PREROUTING)
iptables -A INPUT -i eth0 -j ACCEPT

# -o : interface SORTANTE (OUTPUT, FORWARD, POSTROUTING)
iptables -A OUTPUT -o eth1 -j ACCEPT

# FORWARD utilise les deux
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

**Disponibilité par chaîne :**

| Chaîne | -i (entrée) | -o (sortie) |
|--------|-------------|-------------|
| INPUT | ✅ | ❌ |
| OUTPUT | ❌ | ✅ |
| FORWARD | ✅ | ✅ |
| PREROUTING | ✅ | ❌ |
| POSTROUTING | ❌ | ✅ |

> [!warning] Erreur courante
> ```bash
> # ❌ ERREUR : -o n'existe pas dans INPUT
> iptables -A INPUT -o eth0 -j ACCEPT  # FAIL
> 
> # ✅ CORRECT
> iptables -A INPUT -i eth0 -j ACCEPT
> ```

---

### Interfaces physiques vs virtuelles

**Types d'interfaces :**

```bash
# Physiques
eth0, eth1        # Ethernet
wlan0, wlan1      # WiFi
enp0s3, enp0s8    # Nommage systemd

# Virtuelles
lo                # Loopback
tun0, tun1        # VPN (OpenVPN, WireGuard)
br0, br1          # Bridge
veth0, veth1      # Virtual Ethernet (Docker, LXC)
docker0           # Docker bridge
ppp0              # PPP (dial-up, PPPoE)
```

**Exemples pratiques :**

```bash
# Loopback : toujours autoriser
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Interface externe (WAN)
iptables -A INPUT -i eth0 -m conntrack --ctstate NEW -j DROP  # Bloquer nouvelles entrantes

# Interface interne (LAN)
iptables -A INPUT -i eth1 -j ACCEPT  # Confiance totale

# VPN : autoriser uniquement via tunnel
iptables -A INPUT -i tun0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT ! -i tun0 -p tcp --dport 22 -j DROP

# Docker : autoriser uniquement containers
iptables -A INPUT -i docker0 -j ACCEPT
```

---

### Wildcards et patterns

Les wildcards `+` permettent de **matcher plusieurs interfaces** avec un pattern.

```bash
# Syntaxe : interface+
# Match toutes les interfaces commençant par "interface"

# Toutes les interfaces Ethernet
iptables -A INPUT -i eth+ -j ACCEPT  # eth0, eth1, eth2, etc.

# Toutes les interfaces WiFi
iptables -A INPUT -i wlan+ -j ACCEPT  # wlan0, wlan1, etc.

# Tous les tunnels VPN
iptables -A INPUT -i tun+ -j ACCEPT  # tun0, tun1, tun2, etc.

# Tous les bridges
iptables -A FORWARD -i br+ -o br+ -j ACCEPT  # br0, br1, etc.

# Toutes interfaces sauf loopback
iptables -A INPUT ! -i lo -p tcp --dport 22 -j ACCEPT
```

> [!tip] Négation avec !
> ```bash
> # Autoriser TOUT sauf une interface
> iptables -A INPUT ! -i eth0 -j ACCEPT  # Toutes interfaces sauf eth0
> 
> # Bloquer une interface spécifique
> iptables -A INPUT -i eth0 -j DROP
> iptables -A INPUT -j ACCEPT  # Reste autorisé
> ```

---

### Cas d'usage courants

**1. Firewall multi-interfaces (routeur) :**

```bash
# eth0 = WAN (Internet)
# eth1 = LAN (192.168.1.0/24)
# eth2 = DMZ (10.0.1.0/24)

# Politique par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Loopback
iptables -A INPUT -i lo -j ACCEPT

# Connexions établies
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# INPUT : autoriser SSH depuis LAN uniquement
iptables -A INPUT -i eth1 -p tcp --dport 22 -j ACCEPT

# FORWARD : LAN → Internet
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# FORWARD : DMZ → Internet (HTTP/HTTPS uniquement)
iptables -A FORWARD -i eth2 -o eth0 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -p tcp --dport 443 -j ACCEPT

# FORWARD : Internet → DMZ (port 80 vers serveur web)
iptables -A FORWARD -i eth0 -o eth2 -p tcp --dport 80 -d 10.0.1.10 -j ACCEPT

# FORWARD : bloquer DMZ → LAN
iptables -A FORWARD -i eth2 -o eth1 -j DROP
```

**2. Protection d'interface externe :**

```bash
# eth0 = Internet (non fiable)

# Bloquer scans de ports
iptables -A INPUT -i eth0 -p tcp --tcp-flags ALL NONE -j DROP  # NULL scan
iptables -A INPUT -i eth0 -p tcp --tcp-flags ALL ALL -j DROP   # XMAS scan

# Bloquer IPs privées (spoofing)
iptables -A INPUT -i eth0 -s 10.0.0.0/8 -j DROP
iptables -A INPUT -i eth0 -s 172.16.0.0/12 -j DROP
iptables -A INPUT -i eth0 -s 192.168.0.0/16 -j DROP

# Limiter connexions nouvelles
iptables -A INPUT -i eth0 -m conntrack --ctstate NEW -m limit --limit 10/s -j ACCEPT
iptables -A INPUT -i eth0 -m conntrack --ctstate NEW -j DROP
```

**3. Isolation de VMs/containers :**

```bash
# Autoriser containers entre eux
iptables -A FORWARD -i docker0 -o docker0 -j ACCEPT

# Autoriser containers vers Internet
iptables -A FORWARD -i docker0 ! -o docker0 -j ACCEPT

# Bloquer Internet → containers (sauf ESTABLISHED)
iptables -A FORWARD ! -i docker0 -o docker0 -m conntrack --ctstate NEW -j DROP
```

**4. Hotspot WiFi public :**

```bash
# wlan0 = WiFi public
# eth0 = Internet

# Bloquer inter-clients WiFi
iptables -A FORWARD -i wlan0 -o wlan0 -j DROP

# Autoriser WiFi → Internet uniquement HTTP/HTTPS
iptables -A FORWARD -i wlan0 -o eth0 -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -p tcp --dport 443 -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -p udp --dport 53 -j ACCEPT  # DNS

# Bloquer le reste
iptables -A FORWARD -i wlan0 -j DROP
```

---

## Correspondance par plage d'adresses

### Module iprange

Le module `iprange` permet de spécifier des **plages d'IPs non-CIDR** (intervalles arbitraires).

```bash
# Syntaxe
iptables -A INPUT -m iprange --src-range IP_DÉBUT-IP_FIN -j ACTION
iptables -A INPUT -m iprange --dst-range IP_DÉBUT-IP_FIN -j ACTION

# Exemples
# Autoriser une plage source
iptables -A INPUT -m iprange --src-range 192.168.1.10-192.168.1.50 -j ACCEPT

# Bloquer une plage destination
iptables -A OUTPUT -m iprange --dst-range 10.0.0.100-10.0.0.200 -j DROP

# Combinaison source ET destination
iptables -A FORWARD -m iprange \
  --src-range 192.168.1.0-192.168.1.255 \
  --dst-range 10.0.1.0-10.0.1.255 \
  -j ACCEPT

# Négation
iptables -A INPUT -m iprange ! --src-range 192.168.1.1-192.168.1.254 -j DROP
```

**Options :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--src-range` | Plage d'IPs source | `192.168.1.10-192.168.1.50` |
| `--dst-range` | Plage d'IPs destination | `10.0.0.1-10.0.0.100` |

> [!example] Cas d'usage
> ```bash
> # Plage DHCP : .100 à .200
> iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 \
>   -p udp --dport 67:68 -j ACCEPT
> 
> # Bloquer une plage d'IPs malveillantes
> iptables -A INPUT -m iprange --src-range 203.0.113.0-203.0.113.255 -j DROP
> ```

---

### Plages de ports

Les plages de ports se spécifient avec `:` (pas besoin de module spécial).

```bash
# Plage de ports destination
iptables -A INPUT -p tcp --dport 8000:9000 -j ACCEPT

# Plage de ports source
iptables -A INPUT -p tcp --sport 1024:65535 -j ACCEPT

# Combinaison
iptables -A FORWARD -p tcp --sport 80 --dport 1024:65535 -j ACCEPT

# Depuis un port spécifique jusqu'à la fin
iptables -A INPUT -p tcp --dport 10000: -j ACCEPT  # 10000 à 65535

# Depuis le début jusqu'à un port spécifique
iptables -A INPUT -p tcp --dport :1023 -j DROP  # 0 à 1023 (privilégiés)
```

> [!tip] Ports éphémères
> Les ports source client sont généralement dans la plage 32768-65535 (ou 1024-65535).
> ```bash
> # Autoriser réponses des serveurs web
> iptables -A INPUT -p tcp --sport 80 --dport 1024: -j ACCEPT
> ```

---

### Comparaison avec CIDR

**Quand utiliser iprange vs CIDR :**

| Critère | CIDR | iprange |
|---------|------|---------|
| **Alignement réseau** | Doit être aligné (masque) | Arbitraire |
| **Syntaxe** | Plus simple | Plus verbeux |
| **Performance** | Légèrement plus rapide | Acceptable |
| **Flexibilité** | Limitée | Totale |

```bash
# CIDR : réseau aligné
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT  # .0 à .255

# iprange : plage arbitraire
iptables -A INPUT -m iprange --src-range 192.168.1.50-192.168.1.150 -j ACCEPT

# iprange : multiple réseaux CIDR
# Au lieu de 2 règles CIDR, 1 règle iprange
iptables -A INPUT -m iprange --src-range 192.168.0.0-192.168.3.255 -j ACCEPT
# Équivalent à : 192.168.0.0/22 (mais plus lisible dans certains cas)
```

> [!tip] Préférer CIDR quand possible
> Si votre plage correspond à un réseau CIDR, utilisez `-s` / `-d` :
> - Plus simple
> - Légèrement plus performant
> - Plus standard

---

## Modules de correspondance courants

### Module multiport

Le module `multiport` permet de spécifier **plusieurs ports** dans une seule règle.

```bash
# Sans multiport : 3 règles
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# Avec multiport : 1 règle
iptables -A INPUT -p tcp -m multiport --dports 80,443,8080 -j ACCEPT

# Ports source
iptables -A INPUT -p tcp -m multiport --sports 80,443 -j ACCEPT

# Source ET destination
iptables -A INPUT -p tcp -m multiport --sports 1024:65535 --dports 80,443 -j ACCEPT

# Jusqu'à 15 ports
iptables -A INPUT -p tcp -m multiport --dports 22,25,53,80,110,143,443,465,587,993,995 -j ACCEPT

# Combinaison ports et plages
iptables -A INPUT -p tcp -m multiport --dports 22,80,443,8000:9000 -j ACCEPT
```

**Options :**

| Option | Description | Limite |
|--------|-------------|--------|
| `--sports` | Ports source multiples | Max 15 |
| `--dports` | Ports destination multiples | Max 15 |
| `--ports` | Ports source OU destination | Max 15 |

> [!warning] Limite de 15 ports
> Si vous avez plus de 15 ports, créez plusieurs règles multiport ou une chaîne personnalisée.

> [!tip] Optimisation
> Multiport réduit le nombre de règles → meilleure performance et lisibilité.

---

### Module limit

Le module `limit` implémente un **limiteur de débit** (token bucket algorithm).

```bash
# Syntaxe de base
iptables -A INPUT -m limit --limit TAUX -j ACTION

# Limiter à 5 paquets par minute
iptables -A INPUT -p icmp -m limit --limit 5/min -j ACCEPT
iptables -A INPUT -p icmp -j DROP  # Le reste est bloqué

# Avec burst (pic initial)
iptables -A INPUT -p tcp --dport 80 -m limit --limit 10/sec --limit-burst 20 -j ACCEPT

# Différentes unités de temps
iptables -A INPUT -m limit --limit 100/second -j ACCEPT
iptables -A INPUT -m limit --limit 1000/minute -j ACCEPT
iptables -A INPUT -m limit --limit 10000/hour -j ACCEPT
iptables -A INPUT -m limit --limit 50000/day -j ACCEPT
```

**Options :**

| Option | Description | Défaut | Unités |
|--------|-------------|--------|--------|
| `--limit` | Taux moyen | 3/hour | `/second`, `/minute`, `/hour`, `/day` |
| `--limit-burst` | Burst initial | 5 | Nombre entier |

**Fonctionnement du token bucket :**

```
Bucket : [🪙🪙🪙🪙🪙]  (burst = 5, rechargé à 1/sec)

T=0s  : 5 paquets arrivent → ✅ acceptés (bucket vide)
T=0s  : 6ème paquet → ❌ rejeté (pas de token)
T=1s  : 1 token rechargé
T=1s  : Paquet arrive → ✅ accepté (1 token utilisé)
```

**Exemples pratiques :**

```bash
# 1. Anti-bruteforce SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m limit --limit 3/min --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# 2. Limiter le logging
iptables -A INPUT -m limit --limit 5/min --limit-burst 10 \
  -j LOG --log-prefix "FW-DROP: "
iptables -A INPUT -j DROP

# 3. Limiter les pings (anti-flood)
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/sec --limit-burst 5 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# 4. Limiter nouvelles connexions HTTP
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW \
  -m limit --limit 50/sec --limit-burst 100 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j DROP
```

> [!warning] Limit vs Connlimit
> - **limit** : limite le DÉBIT (paquets/temps)
> - **connlimit** : limite le NOMBRE de connexions simultanées (voir plus bas)

---

### Module recent

Le module `recent` permet de **tracker les IPs récentes** et créer des listes dynamiques.

> [!info] Cas d'usage
> - Détection de bruteforce
> - Limitation par IP
> - Blacklisting temporaire
> - Port knocking

```bash
# Syntaxe
iptables -A INPUT -m recent --ACTION --name LISTE [OPTIONS] -j CIBLE

# Actions
--set       # Ajouter l'IP à la liste
--update    # Mettre à jour le timestamp si IP existe
--rcheck    # Vérifier si IP existe (sans update)
--remove    # Retirer l'IP de la liste

# Options
--seconds N    # Fenêtre de temps
--hitcount N   # Nombre d'occurrences
--name NOM     # Nom de la liste
```

**Exemple 1 : Anti-bruteforce SSH**

```bash
# Créer liste et compter tentatives
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set --name SSH

# Si 4+ tentatives en 60 secondes → DROP
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# Sinon ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

**Exemple 2 : Port knocking simple**

```bash
# Séquence : 7000 → 8000 → 9000 → SSH ouvert

# Étape 1 : knock sur 7000
iptables -A INPUT -p tcp --dport 7000 -m recent --name KNOCK1 --set -j DROP

# Étape 2 : knock sur 8000 (si 7000 dans les 5s)
iptables -A INPUT -p tcp --dport 8000 -m recent --name KNOCK1 --rcheck --seconds 5 \
  -m recent --name KNOCK2 --set -j DROP

# Étape 3 : knock sur 9000 (si 8000 dans les 5s)
iptables -A INPUT -p tcp --dport 9000 -m recent --name KNOCK2 --rcheck --seconds 5 \
  -m recent --name KNOCK3 --set -j DROP

# SSH accessible pendant 60s après séquence complète
iptables -A INPUT -p tcp --dport 22 -m recent --name KNOCK3 --rcheck --seconds 60 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

**Exemple 3 : Whitelist temporaire**

```bash
# Port spécial pour whitelist (ex: 8888)
iptables -A INPUT -p tcp --dport 8888 -m recent --name WHITELIST --set -j DROP

# Autoriser HTTP/SSH si dans whitelist (30 min)
iptables -A INPUT -m recent --name WHITELIST --rcheck --seconds 1800 -j ACCEPT

# Sinon politique normale
iptables -A INPUT -p tcp --dport 22 -j DROP
```

**Exemple 4 : Limitation par IP**

```bash
# Max 20 connexions HTTP par minute par IP
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -m recent --set --name HTTP

iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 20 --name HTTP \
  -j LOG --log-prefix "HTTP-FLOOD: "

iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 20 --name HTTP -j DROP
```

**Gestion des listes :**

```bash
# Voir la liste
cat /proc/net/xt_recent/SSH

# Vider une liste
echo / > /proc/net/xt_recent/SSH

# Ajouter manuellement une IP
echo +192.168.1.100 > /proc/net/xt_recent/SSH

# Retirer une IP
echo -192.168.1.100 > /proc/net/xt_recent/SSH
```

> [!tip] Ajuster la taille maximale
> ```bash
> # Par défaut : 100 IPs par liste
> # Augmenter à 1000
> echo 1000 > /sys/module/xt_recent/parameters/ip_list_tot
> ```

---

### Module connlimit

Le module `connlimit` limite le **nombre de connexions simultanées** par IP.

```bash
# Syntaxe
iptables -A INPUT -m connlimit --connlimit-above N [--connlimit-mask M] -j ACTION

# Limiter à 10 connexions SSH par IP
iptables -A INPUT -p tcp --dport 22 -m connlimit --connlimit-above 10 --connlimit-mask 32 -j REJECT

# Limiter connexions HTTP
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 50 --connlimit-mask 32 -j REJECT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Limiter par réseau /24 (au lieu de par IP)
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 100 --connlimit-mask 24 -j REJECT
```

**Options :**

| Option | Description | Défaut |
|--------|-------------|--------|
| `--connlimit-above N` | Rejeter si > N connexions | - |
| `--connlimit-upto N` | Accepter si ≤ N connexions | - |
| `--connlimit-mask M` | Masque de réseau (0-32 pour IPv4) | 32 (par IP) |
| `--connlimit-saddr` | Compter par source (défaut) | - |
| `--connlimit-daddr` | Compter par destination | - |

**Exemples avancés :**

```bash
# 1. Limite différente selon le protocole
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 50 -j REJECT
iptables -A INPUT -p tcp --dport 443 -m connlimit --connlimit-above 100 -j REJECT

# 2. Limite par réseau /24
# Max 200 connexions simultanées depuis un /24
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 200 --connlimit-mask 24 -j REJECT

# 3. Whitelist avant limitation
iptables -A INPUT -s 192.168.1.100 -j ACCEPT  # IP admin illimitée
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 20 -j REJECT

# 4. Combinaison avec recent (double protection)
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 5 -j DROP
iptables -A INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j REJECT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

> [!warning] Limite vs Connlimit
> - **limit** : débit de nouveaux paquets (paquets/sec)
> - **connlimit** : connexions simultanées actives
> 
> Utiliser les DEUX pour une protection optimale !

---

### Module owner

Le module `owner` filtre selon le **propriétaire du processus** (local uniquement).

> [!warning] Chaîne OUTPUT uniquement
> Le module owner ne fonctionne que sur la chaîne OUTPUT (et POSTROUTING), car seuls les paquets locaux ont un propriétaire.

```bash
# Par UID (user ID)
iptables -A OUTPUT -m owner --uid-owner 1000 -j ACCEPT

# Par GID (group ID)
iptables -A OUTPUT -m owner --gid-owner 1000 -j ACCEPT

# Par nom d'utilisateur (résolu en UID)
iptables -A OUTPUT -m owner --uid-owner www-data -j ACCEPT

# Par processus (PID)
iptables -A OUTPUT -m owner --pid-owner 1234 -j ACCEPT

# Par socket (avancé)
iptables -A OUTPUT -m owner --socket-exists -j ACCEPT
```

**Options :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--uid-owner` | User ID ou nom | `1000` ou `john` |
| `--gid-owner` | Group ID ou nom | `1000` ou `www-data` |
| `--pid-owner` | Process ID | `1234` |
| `--sid-owner` | Session ID | `5678` |
| `--socket-exists` | Vérifie si socket existe | - |

**Exemples pratiques :**

```bash
# 1. Restreindre accès Internet par utilisateur
# Seul 'john' peut accéder au web
iptables -A OUTPUT -m owner --uid-owner john -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner john -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j DROP
iptables -A OUTPUT -p tcp --dport 443 -j DROP

# 2. Forcer un utilisateur à passer par proxy
# 'alice' ne peut pas accéder directement au web
iptables -A OUTPUT -m owner --uid-owner alice -p tcp --dport 80 -j DROP
iptables -A OUTPUT -m owner --uid-owner alice -p tcp --dport 443 -j DROP

# 3. Isoler un service
# www-data (Apache) ne peut accéder qu'à la BDD locale
iptables -A OUTPUT -m owner --uid-owner www-data -d 127.0.0.1 -p tcp --dport 3306 -j ACCEPT
iptables -A OUTPUT -m owner --uid-owner www-data -j DROP

# 4. Bloquer root sur Internet (sécurité)
iptables -A OUTPUT -m owner --uid-owner 0 -p tcp --dport 80 -j DROP
iptables -A OUTPUT -m owner --uid-owner 0 -p tcp --dport 443 -j DROP

# 5. Autoriser uniquement certains groupes
iptables -A OUTPUT -m owner --gid-owner internet-users -j ACCEPT
iptables -A OUTPUT -j DROP
```

> [!tip] Cas d'usage
> - Contrôle parental / filtrage utilisateur
> - Isolation de services (containers, jails)
> - Sécurité : empêcher root de surfer
> - Forcer utilisation d'un proxy
> - Audit : tracker trafic par utilisateur

---

### Module mac

Le module `mac` filtre selon l'**adresse MAC source**.

```bash
# Syntaxe
iptables -A INPUT -m mac --mac-source ADRESSE_MAC -j ACTION

# Autoriser une MAC spécifique
iptables -A INPUT -m mac --mac-source 00:11:22:33:44:55 -j ACCEPT

# Bloquer une MAC
iptables -A INPUT -m mac --mac-source AA:BB:CC:DD:EE:FF -j DROP

# Whitelist MAC pour DHCP
iptables -A INPUT -p udp --dport 67:68 -m mac --mac-source 00:11:22:33:44:55 -j ACCEPT
iptables -A INPUT -p udp --dport 67:68 -j DROP

# Plusieurs MACs (chaîne personnalisée)
iptables -N MAC_WHITELIST
iptables -A MAC_WHITELIST -m mac --mac-source 00:11:22:33:44:55 -j RETURN
iptables -A MAC_WHITELIST -m mac --mac-source 00:AA:BB:CC:DD:EE -j RETURN
iptables -A MAC_WHITELIST -j DROP

iptables -A INPUT -i eth1 -j MAC_WHITELIST
```

> [!warning] Limitations
> - Fonctionne uniquement sur le **réseau local** (même broadcast domain)
> - Les MACs **ne traversent pas les routeurs**
> - Facilement **usurpable** (MAC spoofing)
> - Utilisez en **complément** d'autres mesures, pas seul

**Cas d'usage :**

```bash
# 1. Réseau WiFi : whitelist de devices
iptables -A INPUT -i wlan0 ! -m mac --mac-source 00:11:22:33:44:55 -j DROP

# 2. Serveur DHCP : autoriser uniquement MACs connues
iptables -A INPUT -p udp --dport 67 -m mac --mac-source 00:AA:BB:CC:DD:EE -j ACCEPT
iptables -A INPUT -p udp --dport 67 -j DROP

# 3. Audit : logger accès par MAC
iptables -A INPUT -m mac --mac-source 00:11:22:33:44:55 \
  -j LOG --log-prefix "DEVICE-1: "
```

---

### Module string

Le module `string` recherche des **chaînes de caractères** dans le payload des paquets.

> [!warning] Performance
> L'inspection de contenu est **coûteuse en CPU**. Utilisez avec modération et combinez avec d'autres critères.

```bash
# Syntaxe
iptables -A CHAIN -m string --string "TEXTE" --algo ALGO -j ACTION

# Algorithmes disponibles
--algo bm    # Boyer-Moore (plus rapide)
--algo kmp   # Knuth-Morris-Pratt

# Bloquer requêtes contenant "admin"
iptables -A INPUT -p tcp --dport 80 -m string --string "admin" --algo bm -j DROP

# Bloquer User-Agent spécifique
iptables -A INPUT -p tcp --dport 80 -m string --string "User-Agent: BadBot" --algo bm -j DROP

# Bloquer téléchargement .exe
iptables -A FORWARD -p tcp --dport 80 -m string --string ".exe" --algo bm -j DROP

# Hexadécimal (pour binaire)
iptables -A INPUT -m string --hex-string "|00 50 56|" --algo bm -j DROP
```

**Options :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--string` | Texte à chercher (ASCII) | `"admin"` |
| `--hex-string` | Hex à chercher | `"|FF FE 00|"` |
| `--algo` | Algorithme (bm ou kmp) | `bm` |
| `--from` | Position début (octets) | `0` |
| `--to` | Position fin (octets) | `500` |
| `--icase` | Insensible à la casse | - |

**Exemples avancés :**

```bash
# 1. Bloquer SQL injection basique
iptables -A INPUT -p tcp --dport 80 -m string --string "SELECT * FROM" --algo bm -j DROP
iptables -A INPUT -p tcp --dport 80 -m string --string "UNION SELECT" --algo bm -j DROP

# 2. Bloquer malware connu (signature)
iptables -A FORWARD -m string --hex-string "|4D 5A 90 00|" --algo bm -j DROP  # MZ header

# 3. Filtrage de contenu web
iptables -A FORWARD -p tcp --dport 80 -m string --string "porn" --algo bm --icase -j DROP
iptables -A FORWARD -p tcp --dport 80 -m string --string "gambling" --algo bm --icase -j DROP

# 4. Protection anti-scraping (bloquer certains User-Agents)
iptables -A INPUT -p tcp --dport 80 -m string --string "Python-urllib" --algo bm -j DROP
iptables -A INPUT -p tcp --dport 80 -m string --string "Scrapy" --algo bm -j DROP

# 5. Limiter à certaines commandes SMTP
iptables -A INPUT -p tcp --dport 25 -m string --string "RCPT TO" --algo bm -j ACCEPT
iptables -A INPUT -p tcp --dport 25 -m string --string "DATA" --algo bm -j ACCEPT
```

> [!warning] Limitations
> - Ne fonctionne **pas sur HTTPS** (chiffré)
> - Peut être **contourné** (encodage, fragmentation)
> - **Impact performance** significatif
> - Préférer un **proxy applicatif** (Squid, HAProxy) pour filtrage avancé

---

### Module time

Le module `time` filtre selon la **date et l'heure**.

```bash
# Syntaxe
iptables -A INPUT -m time [OPTIONS] -j ACTION

# Bloquer accès la nuit (22h-6h)
iptables -A INPUT -p tcp --dport 80 -m time --timestart 22:00 --timestop 06:00 -j DROP

# Autoriser uniquement en semaine
iptables -A INPUT -p tcp --dport 22 -m time --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Bloquer le weekend
iptables -A FORWARD -m time --weekdays Sat,Sun -j DROP

# Autoriser entre deux dates
iptables -A INPUT -m time --datestart 2024-01-01 --datestop 2024-12-31 -j ACCEPT
```

**Options :**

| Option | Description | Format | Exemple |
|--------|-------------|--------|---------|
| `--timestart` | Heure début | HH:MM[:SS] | `09:00` |
| `--timestop` | Heure fin | HH:MM[:SS] | `17:30` |
| `--weekdays` | Jours de la semaine | Mon-Sun | `Mon,Wed,Fri` |
| `--monthdays` | Jours du mois | 1-31 | `1,15,30` |
| `--datestart` | Date début | YYYY[-MM[-DD[THH[:MM[:SS]]]]] | `2024-01-01` |
| `--datestop` | Date fin | YYYY[-MM[-DD[THH[:MM[:SS]]]]] | `2024-12-31` |
| `--kerneltz` | Utiliser timezone kernel | - | - |
| `--utc` | Utiliser UTC | - | - |

**Exemples pratiques :**

```bash
# 1. Heures de bureau (9h-18h, lundi-vendredi)
iptables -A FORWARD -m time --timestart 09:00 --timestop 18:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A FORWARD -j DROP

# 2. Maintenance mensuelle (1er de chaque mois, 2h-4h)
iptables -A INPUT -p tcp --dport 22 \
  -m time --monthdays 1 --timestart 02:00 --timestop 04:00 -j ACCEPT

# 3. Contrôle parental (bloquer après 21h)
iptables -A FORWARD -s 192.168.1.100 \
  -m time --timestart 21:00 --timestop 23:59 -j DROP
iptables -A FORWARD -s 192.168.1.100 \
  -m time --timestart 00:00 --timestop 07:00 -j DROP

# 4. Restriction temporaire (campagne)
iptables -A INPUT -p tcp --dport 8080 \
  -m time --datestart 2024-12-01 --datestop 2024-12-31 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP

# 5. Backup nocturne uniquement
iptables -A OUTPUT -d 10.0.0.10 -p tcp --dport 873 \
  -m time --timestart 02:00 --timestop 05:00 -j ACCEPT
iptables -A OUTPUT -d 10.0.0.10 -p tcp --dport 873 -j DROP
```

> [!tip] Timezone
> Par défaut, `time` utilise UTC. Utilisez `--kerneltz` pour la timezone système :
> ```bash
> iptables -A INPUT -m time --timestart 09:00 --timestop 17:00 --kerneltz -j ACCEPT
> ```

---

### Module comment

Le module `comment` permet d'ajouter des **commentaires** aux règles.

```bash
# Syntaxe
iptables -A INPUT [...] -m comment --comment "TEXTE" -j ACTION

# Limite : 256 caractères

# Exemples
iptables -A INPUT -p tcp --dport 22 -j ACCEPT \
  -m comment --comment "SSH access for administration"

iptables -A INPUT -s 10.0.0.0/8 -j DROP \
  -m comment --comment "Block RFC1918 from WAN - security policy 2024"

iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT \
  -m comment --comment "LAN to WAN traffic allowed"

# Utile pour documenter des règles complexes
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 50 -j REJECT \
  -m comment --comment "Limit HTTP connections to 50 per IP - anti-DoS"
```

**Voir les commentaires :**

```bash
# Lister avec commentaires
iptables -L -n -v --line-numbers

# Sauvegarder avec commentaires
iptables-save > /etc/iptables/rules.v4  # Les commentaires sont préservés
```

> [!tip] Bonnes pratiques
> Utilisez des commentaires pour :
> - Expliquer **pourquoi** une règle existe
> - Référencer un **ticket/incident**
> - Indiquer la **date** de création
> - Marquer les règles **temporaires**
> 
> ```bash
> iptables -A INPUT -s 203.0.113.50 -j DROP \
>   -m comment --comment "TEMP: Block attacker - ticket #12345 - 2024-02-12"
> ```

---

### Module tcp (options TCP)

Le module `tcp` (implicite avec `-p tcp`) offre des **options avancées** pour TCP.

```bash
# Options TCP
--tcp-flags     # Tester flags TCP
--syn           # Raccourci pour SYN (nouvelle connexion)
--tcp-option    # Tester options TCP

# Drapeaux TCP : SYN, ACK, FIN, RST, URG, PSH, ALL, NONE

# Bloquer scans de ports
# NULL scan (tous flags à 0)
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# XMAS scan (FIN, PSH, URG)
iptables -A INPUT -p tcp --tcp-flags FIN,PSH,URG FIN,PSH,URG -j DROP

# SYN-FIN scan (illogique)
iptables -A INPUT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP

# Autoriser uniquement SYN pour nouvelles connexions
iptables -A INPUT -p tcp ! --syn -m conntrack --ctstate NEW -j DROP

# Bloquer paquets avec tous flags activés
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

**Syntaxe `--tcp-flags` :**

```bash
# Format : --tcp-flags MASQUE FLAGS_ACTIFS
# MASQUE : flags à examiner
# FLAGS_ACTIFS : flags qui doivent être à 1

# Exemple : SYN actif, ACK/FIN/RST inactifs
iptables -A INPUT -p tcp --tcp-flags SYN,ACK,FIN,RST SYN -j ACCEPT

# Équivalent à --syn (raccourci)
iptables -A INPUT -p tcp --syn -j ACCEPT
```

**Protection anti-scan :**

```bash
# Chaîne de détection de scans
iptables -N SCAN_DETECT

# NULL scan
iptables -A SCAN_DETECT -p tcp --tcp-flags ALL NONE \
  -j LOG --log-prefix "NULL-SCAN: "
iptables -A SCAN_DETECT -p tcp --tcp-flags ALL NONE -j DROP

# XMAS scan
iptables -A SCAN_DETECT -p tcp --tcp-flags FIN,PSH,URG FIN,PSH,URG \
  -j LOG --log-prefix "XMAS-SCAN: "
iptables -A SCAN_DETECT -p tcp --tcp-flags FIN,PSH,URG FIN,PSH,URG -j DROP

# SYN-FIN
iptables -A SCAN_DETECT -p tcp --tcp-flags SYN,FIN SYN,FIN \
  -j LOG --log-prefix "SYNFIN-SCAN: "
iptables -A SCAN_DETECT -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP

# Appliquer
iptables -A INPUT -p tcp -j SCAN_DETECT
```

---

### Module icmp

Le module `icmp` (implicite avec `-p icmp`) filtre par **type ICMP**.

```bash
# Syntaxe
iptables -A INPUT -p icmp --icmp-type TYPE -j ACTION

# Types courants
--icmp-type echo-request     # Ping (type 8)
--icmp-type echo-reply       # Pong (type 0)
--icmp-type destination-unreachable  # Type 3
--icmp-type time-exceeded    # Type 11 (traceroute)
--icmp-type timestamp-request  # Type 13

# Par numéro
--icmp-type 8    # Echo request
--icmp-type 0    # Echo reply
```

**Exemples :**

```bash
# 1. Autoriser ping limité
iptables -A INPUT -p icmp --icmp-type echo-request \
  -m limit --limit 1/sec -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# 2. Autoriser réponses ping
iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT

# 3. Autoriser ICMP d'erreur (important pour TCP)
iptables -A INPUT -p icmp --icmp-type destination-unreachable -j ACCEPT
iptables -A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
iptables -A INPUT -p icmp --icmp-type parameter-problem -j ACCEPT

# 4. Bloquer tout autre ICMP
iptables -A INPUT -p icmp -j DROP

# 5. Traceroute (autoriser)
iptables -A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
iptables -A INPUT -p udp --dport 33434:33524 -j ACCEPT  # Ports traceroute
```

**Types ICMP utiles :**

| Type | Nom | Usage |
|------|-----|-------|
| 0 | echo-reply | Réponse ping |
| 3 | destination-unreachable | Erreurs de routage |
| 4 | source-quench | Congestion (obsolète) |
| 5 | redirect | Redirection route |
| 8 | echo-request | Ping |
| 11 | time-exceeded | TTL expiré (traceroute) |
| 12 | parameter-problem | Problème IP header |

> [!warning] Ne bloquez pas tout ICMP
> Certains ICMP sont **nécessaires** au bon fonctionnement de TCP/IP :
> - Type 3 : Path MTU Discovery
> - Type 11 : Traceroute, debugging
> 
> Bloquer tout ICMP peut casser des connexions !

---

## Combinaison de modules

Les modules peuvent être **combinés** pour créer des règles très précises.

**Exemples de combinaisons :**

```bash
# 1. Limiter SSH par IP ET par temps
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH \
  -m limit --limit 1/min -j LOG --log-prefix "SSH-BRUTE: "
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP
iptables -A INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j REJECT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 2. Interface + état + limite + temps
iptables -A FORWARD -i eth1 -o eth0 \
  -m conntrack --ctstate NEW \
  -m limit --limit 100/sec \
  -m time --timestart 09:00 --timestop 18:00 --weekdays Mon,Tue,Wed,Thu,Fri \
  -j ACCEPT

# 3. Plage IP + multiport + connlimit
iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 \
  -p tcp -m multiport --dports 80,443,8080 \
  -m connlimit --connlimit-above 20 -j REJECT

# 4. Owner + string + état (filtrage applicatif local)
iptables -A OUTPUT -m owner --uid-owner john \
  -m conntrack --ctstate NEW \
  -m string --string "facebook.com" --algo bm -j DROP

# 5. MAC + recent + time (contrôle d'accès complexe)
iptables -A INPUT -i wlan0 \
  -m mac --mac-source 00:11:22:33:44:55 \
  -m recent --set --name DEVICE1
iptables -A INPUT -i wlan0 \
  -m recent --update --seconds 300 --name DEVICE1 \
  -m time --timestart 08:00 --timestop 20:00 -j ACCEPT

# 6. Multi-critères anti-DDoS
iptables -A INPUT -p tcp --dport 80 \
  -m conntrack --ctstate NEW \
  -m recent --set --name HTTP \
  -m limit --limit 50/sec --limit-burst 100
iptables -A INPUT -p tcp --dport 80 \
  -m recent --update --seconds 10 --hitcount 100 --name HTTP -j DROP
iptables -A INPUT -p tcp --dport 80 \
  -m connlimit --connlimit-above 50 -j REJECT
```

---

## Pièges courants

### 1. Oublier --ctstate ESTABLISHED,RELATED

```bash
# ❌ Bloque les réponses légitimes
iptables -P INPUT DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# Connexion SSH se bloque après le SYN initial !

# ✅ CORRECT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
iptables -P INPUT DROP
```

### 2. Ordre limit/action incorrect

```bash
# ❌ Le DROP est avant limit → tous les paquets sont droppés
iptables -A INPUT -p icmp -j DROP
iptables -A INPUT -p icmp -m limit --limit 1/sec -j ACCEPT

# ✅ CORRECT : limit puis DROP
iptables -A INPUT -p icmp -m limit --limit 1/sec -j ACCEPT
iptables -A INPUT -p icmp -j DROP
```

### 3. Interface dans mauvaise chaîne

```bash
# ❌ ERREUR : -o dans INPUT
iptables -A INPUT -o eth0 -j ACCEPT  # iptables: No chain/target/match by that name

# ✅ CORRECT
iptables -A INPUT -i eth0 -j ACCEPT   # INPUT = -i
iptables -A OUTPUT -o eth0 -j ACCEPT  # OUTPUT = -o
```

### 4. Connlimit sans --connlimit-mask

```bash
# ❌ Par défaut mask=32 (par IP) mais implicite
iptables -A INPUT -m connlimit --connlimit-above 10 -j REJECT

# ✅ Explicite (meilleure pratique)
iptables -A INPUT -m connlimit --connlimit-above 10 --connlimit-mask 32 -j REJECT
```

### 5. Recent sans --name

```bash
# ❌ Fonctionne mais non maintenable
iptables -A INPUT -m recent --set
iptables -A INPUT -m recent --update --seconds 60 --hitcount 4

# ✅ Avec nom explicite
iptables -A INPUT -m recent --set --name SSH
iptables -A INPUT -m recent --update --seconds 60 --hitcount 4 --name SSH
```

### 6. Owner sur INPUT

```bash
# ❌ Owner ne fonctionne pas sur INPUT
iptables -A INPUT -m owner --uid-owner john -j ACCEPT  # Jamais de match

# ✅ Owner sur OUTPUT uniquement
iptables -A OUTPUT -m owner --uid-owner john -j ACCEPT
```

### 7. String sur HTTPS

```bash
# ❌ Inefficace : HTTPS est chiffré
iptables -A INPUT -p tcp --dport 443 -m string --string "admin" --algo bm -j DROP

# ✅ String sur HTTP uniquement
iptables -A INPUT -p tcp --dport 80 -m string --string "admin" --algo bm -j DROP
```

### 8. Time sans --kerneltz

```bash
# ❌ UTC par défaut (peut ne pas correspondre à votre timezone)
iptables -A INPUT -m time --timestart 09:00 --timestop 17:00

# ✅ Utiliser timezone système
iptables -A INPUT -m time --timestart 09:00 --timestop 17:00 --kerneltz
```

---

## Bonnes pratiques

### 1. Ordre optimal des règles

```bash
#!/bin/bash
# Structure recommandée

# 1. Loopback (très fréquent)
iptables -A INPUT -i lo -j ACCEPT

# 2. Paquets invalides (sécurité)
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# 3. ESTABLISHED,RELATED (90%+ du trafic)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# 4. Protection anti-scan
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# 5. Nouvelles connexions autorisées (du plus fréquent au moins fréquent)
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# 6. Logging limité
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "FW-DROP: "

# 7. Politique par défaut
iptables -P INPUT DROP
```

### 2. Combiner limit et recent

```bash
# Protection SSH multi-niveaux
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --set --name SSH

# Niveau 1 : max 4 tentatives en 60s
iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 4 --name SSH \
  -j LOG --log-prefix "SSH-BRUTE-L1: "
iptables -A INPUT -p tcp --dport 22 -m recent --update --seconds 60 --hitcount 4 --name SSH \
  -j DROP

# Niveau 2 : max 3 connexions simultanées
iptables -A INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j REJECT

# Niveau 3 : rate limiting global
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
  -m limit --limit 5/min --limit-burst 3 -j ACCEPT
```

### 3. Toujours spécifier --connlimit-mask

```bash
# ✅ Explicite et documenté
iptables -A INPUT -m connlimit --connlimit-above 20 --connlimit-mask 32 \
  -m comment --comment "Limit 20 conn per IP"

# Par réseau /24
iptables -A INPUT -m connlimit --connlimit-above 100 --connlimit-mask 24 \
  -m comment --comment "Limit 100 conn per /24 network"
```

### 4. Utiliser des chaînes pour organisation

```bash
# Par interface
iptables -N IFACE_WAN
iptables -N IFACE_LAN
iptables -N IFACE_DMZ

iptables -A INPUT -i eth0 -j IFACE_WAN
iptables -A INPUT -i eth1 -j IFACE_LAN
iptables -A INPUT -i eth2 -j IFACE_DMZ

# Par service
iptables -N SERVICE_SSH
iptables -N SERVICE_WEB
iptables -N SERVICE_MAIL

# Protection intégrée par service
iptables -A SERVICE_SSH -m recent --set --name SSH
iptables -A SERVICE_SSH -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP
iptables -A SERVICE_SSH -m connlimit --connlimit-above 3 -j REJECT
iptables -A SERVICE_SSH -j ACCEPT
```

### 5. Documenter avec comment

```bash
# Toujours expliquer les règles complexes
iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 \
  -p tcp -m multiport --dports 80,443 \
  -m connlimit --connlimit-above 20 --connlimit-mask 32 \
  -m time --timestart 09:00 --timestop 18:00 --weekdays Mon,Tue,Wed,Thu,Fri --kerneltz \
  -j ACCEPT \
  -m comment --comment "Allow HTTP/HTTPS from DHCP range, max 20 conn/IP, business hours only"
```

### 6. Tester avant production

```bash
# Script de test avec rollback
#!/bin/bash
set -e

BACKUP="/tmp/iptables-$(date +%s).bak"

# Sauvegarder
iptables-save > "$BACKUP"

# Appliquer
source /etc/iptables/new-rules.sh

# Test
echo "Règles actives. Test SSH dans 60 secondes..."
read -t 60 -p "Valider? (ENTER=oui, timeout=rollback): " || {
    echo "Rollback..."
    iptables-restore < "$BACKUP"
    exit 1
}

echo "Règles validées. Sauvegarde permanente..."
iptables-save > /etc/iptables/rules.v4
```

### 7. Monitoring et ajustement

```bash
# Vérifier performance conntrack
watch -n 2 'cat /proc/sys/net/netfilter/nf_conntrack_count'

# Top IPs dans recent
for list in /proc/net/xt_recent/*; do
    echo "=== $(basename $list) ==="
    cat "$list" | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
done

# Vérifier les compteurs
watch -n 5 'iptables -L -n -v --line-numbers'
```

---

> [!tip] 💡 Points clés à retenir
> 1. **conntrack** avec ESTABLISHED,RELATED = performance + sécurité
> 2. Placez ESTABLISHED,RELATED en **première position** (après loopback)
> 3. **-i** pour INPUT/FORWARD, **-o** pour OUTPUT/FORWARD
> 4. **limit** = débit, **connlimit** = connexions simultanées
> 5. **recent** = tracking d'IPs, **multiport** = multiple ports
> 6. **owner** fonctionne sur OUTPUT uniquement
> 7. **Combinez** plusieurs modules pour règles sophistiquées
> 8. **Documentez** avec `-m comment`
> 9. **Testez** avec auto-rollback avant production
> 10. **Surveillez** les compteurs et conntrack régulièrement
