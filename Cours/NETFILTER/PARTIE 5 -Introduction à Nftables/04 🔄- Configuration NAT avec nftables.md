## 📑 Table des matières

- [Introduction au NAT](#introduction-au-nat)
- [SNAT et masquerading](#snat-et-masquerading)
- [DNAT et redirection de ports](#dnat-et-redirection-de-ports)
- [Syntaxe et exemples pratiques](#syntaxe-et-exemples-pratiques)
- [Comparaison avec iptables](#comparaison-avec-iptables)

---

## Introduction au NAT

Le **NAT (Network Address Translation)** permet de modifier les adresses IP des paquets qui traversent le firewall. C'est une fonction essentielle pour :
- Partager une connexion Internet (masquerading)
- Protéger les réseaux internes (masquage des adresses privées)
- Rediriger des services vers des serveurs internes (port forwarding)
- Créer des DMZ et architectures réseau complexes

> [!info] NAT dans nftables
> Avec nftables, la configuration NAT est :
> - Plus simple et lisible qu'avec iptables
> - Intégrée dans la syntaxe unifiée (plus de `-t nat`)
> - Plus flexible avec les maps pour le multi-DNAT
> - Compatible IPv4 et IPv6 avec la famille `inet`

### Types de NAT

| Type | Modification | Usage principal |
|------|--------------|-----------------|
| **SNAT** | Adresse source | Sortie Internet, connexions sortantes |
| **Masquerading** | Adresse source dynamique | Comme SNAT mais IP changeante (DHCP) |
| **DNAT** | Adresse destination | Redirection vers serveurs internes |
| **Redirection** | Port local | Proxy transparent, redirection locale |

### Prérequis pour le NAT

```bash
# 1. Activer le forwarding IP (CRUCIAL !)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Persistant via sysctl
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Pour IPv6
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p

# 2. Créer une table NAT
nft add table inet nat

# 3. Créer les chaînes de base
nft add chain inet nat prerouting { type nat hook prerouting priority dstnat \; policy accept \; }
nft add chain inet nat postrouting { type nat hook postrouting priority srcnat \; policy accept \; }
```

> [!warning] Le forwarding est obligatoire !
> Sans `ip_forward = 1`, les paquets NATés ne seront pas routés entre les interfaces. C'est l'erreur la plus fréquente en configuration NAT.

---

## SNAT et masquerading

Le **SNAT (Source NAT)** modifie l'adresse IP source des paquets sortants. C'est indispensable pour permettre à un réseau privé d'accéder à Internet.

### SNAT : adresse source fixe

Utilisez SNAT quand vous avez une **adresse IP publique fixe**.

```bash
# Configuration de base
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 snat to 203.0.113.50

# Explication :
# - oif eth0 : interface de sortie (côté Internet)
# - ip saddr 192.168.1.0/24 : réseau source à NATer
# - snat to 203.0.113.50 : adresse publique à utiliser
```

> [!info] Pourquoi utiliser SNAT ?
> - Performance légèrement meilleure que masquerading
> - Logs plus clairs (toujours la même IP source)
> - Idéal pour les serveurs avec IP fixe

#### SNAT avec plage d'adresses

```bash
# Répartir sur plusieurs IPs publiques
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 \
    snat to 203.0.113.50-203.0.113.60

# SNAT avec ports personnalisés
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 \
    snat to 203.0.113.50:1024-65535
```

> [!tip] Plage d'IPs pour la scalabilité
> Avec une seule IP publique, vous êtes limité à ~65000 connexions simultanées (limitation des ports). Utilisez plusieurs IPs pour dépasser cette limite.

#### SNAT conditionnel

```bash
# SNAT uniquement pour certains protocoles
nft add rule inet nat postrouting oif eth0 tcp dport 80 \
    ip saddr 192.168.1.0/24 snat to 203.0.113.50

# SNAT avec préservation du port source
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 \
    snat to 203.0.113.50 persistent

# SNAT avec random (répartition aléatoire)
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 \
    snat to 203.0.113.50-203.0.113.60 random

# SNAT avec fully-random (ports aléatoires)
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 \
    snat to 203.0.113.50 fully-random
```

**Options de SNAT :**
| Option | Description |
|--------|-------------|
| `persistent` | Conserve le même port source si possible |
| `random` | Choix aléatoire de l'IP dans la plage |
| `fully-random` | Randomisation complète des ports |

### Masquerading : adresse source dynamique

Utilisez **masquerading** quand votre IP publique change (DHCP, connexion mobile).

```bash
# Configuration de base
nft add rule inet nat postrouting oif eth0 masquerade

# Avec limitation de port
nft add rule inet nat postrouting oif eth0 masquerade to :1024-65535

# Avec options
nft add rule inet nat postrouting oif eth0 masquerade random
nft add rule inet nat postrouting oif eth0 masquerade fully-random
```

> [!info] Masquerading vs SNAT
> **Masquerading** :
> - ✅ S'adapte automatiquement aux changements d'IP
> - ✅ Parfait pour ADSL/Fibre résidentiel, 4G
> - ❌ Légèrement plus lent (doit récupérer l'IP à chaque paquet)
> 
> **SNAT** :
> - ✅ Plus rapide (IP en cache)
> - ✅ Idéal pour serveurs/entreprises
> - ❌ Ne s'adapte pas aux changements d'IP

### Cas pratiques SNAT/Masquerading

#### Partage de connexion Internet simple

```bash
# Réseau interne : 192.168.1.0/24
# Interface externe : eth0 (vers Internet)
# Interface interne : eth1 (vers LAN)

# Table et chaînes
nft add table inet nat
nft add chain inet nat postrouting { type nat hook postrouting priority srcnat \; policy accept \; }

# Masquerading (IP dynamique)
nft add rule inet nat postrouting oif eth0 masquerade

# OU SNAT si IP fixe (ex: 203.0.113.50)
# nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 snat to 203.0.113.50
```

#### Multi-WAN avec SNAT

```bash
# Deux connexions Internet : eth0 et eth1
# Répartition du trafic

# SNAT pour la première connexion
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/25 \
    snat to 203.0.113.50

# SNAT pour la deuxième connexion
nft add rule inet nat postrouting oif eth1 ip saddr 192.168.1.128/25 \
    snat to 198.51.100.20
```

#### SNAT avec règles de filtrage

```bash
# Table filter séparée
nft add table inet filter
nft add chain inet filter forward { type filter hook forward priority filter \; policy drop \; }

# Autoriser le forward pour le réseau NATé
nft add rule inet filter forward ip saddr 192.168.1.0/24 ct state new,established,related accept
nft add rule inet filter forward ct state established,related accept

# Table NAT
nft add table inet nat
nft add chain inet nat postrouting { type nat hook postrouting priority srcnat \; policy accept \; }
nft add rule inet nat postrouting oif eth0 masquerade
```

> [!warning] N'oubliez pas la table filter !
> Le NAT ne remplace pas les règles de filtrage. Vous devez autoriser le forward dans la table filter, sinon les paquets NATés seront bloqués.

---

## DNAT et redirection de ports

Le **DNAT (Destination NAT)** modifie l'adresse IP de destination des paquets entrants. C'est la base du **port forwarding** pour exposer des services internes.

### DNAT : redirection vers serveur interne

```bash
# Configuration de base
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100

# Explication :
# - iif eth0 : interface d'entrée (côté Internet)
# - tcp dport 80 : port public à rediriger
# - dnat to 192.168.1.100 : serveur interne cible
```

> [!info] À quoi sert le DNAT ?
> - Exposer un serveur web interne sur Internet
> - Créer des DMZ sécurisées
> - Rediriger des services (SSH, RDP, etc.)
> - Load balancing basique entre plusieurs serveurs

#### DNAT avec changement de port

```bash
# Rediriger le port 8080 externe vers le port 80 interne
nft add rule inet nat prerouting iif eth0 tcp dport 8080 dnat to 192.168.1.100:80

# Rediriger plusieurs ports vers différents serveurs
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100:80
nft add rule inet nat prerouting iif eth0 tcp dport 443 dnat to 192.168.1.100:443
nft add rule inet nat prerouting iif eth0 tcp dport 8080 dnat to 192.168.1.101:80
```

#### DNAT avec plage de ports

```bash
# Rediriger une plage de ports (ex: FTP passif)
nft add rule inet nat prerouting iif eth0 tcp dport 21000-21100 dnat to 192.168.1.50

# Rediriger vers une plage de serveurs (load balancing)
nft add rule inet nat prerouting iif eth0 tcp dport 80 \
    dnat to 192.168.1.100-192.168.1.105
```

#### DNAT conditionnel

```bash
# DNAT uniquement depuis certaines IP sources
nft add rule inet nat prerouting iif eth0 ip saddr 203.0.113.0/24 \
    tcp dport 22 dnat to 192.168.1.10

# DNAT avec limitation (protection anti-DDoS basique)
nft add rule inet nat prerouting iif eth0 tcp dport 80 \
    limit rate 100/second dnat to 192.168.1.100
```

### Redirection locale (REDIRECT)

La **redirection** change uniquement le port de destination, sans modifier l'IP. Utilisé pour les proxys transparents.

```bash
# Rediriger le trafic HTTP vers un proxy local (port 3128)
nft add rule inet nat prerouting iif eth1 tcp dport 80 redirect to 3128

# Rediriger vers une plage de ports
nft add rule inet nat prerouting iif eth1 tcp dport 80 redirect to :8080-8090

# Avec options
nft add rule inet nat prerouting iif eth1 tcp dport 80 redirect to :3128 random
```

> [!tip] Proxy transparent avec REDIRECT
> La redirection est idéale pour :
> - Proxys HTTP/HTTPS transparents (Squid)
> - Captive portals (WiFi publics)
> - Filtrage de contenu sans configuration client

### DNAT bidirectionnel (avec SNAT)

Pour que le DNAT fonctionne correctement, il faut souvent combiner avec du SNAT de retour.

```bash
# Scénario : Client Internet → Serveur web interne
# Client : 203.0.113.10
# Firewall externe : 198.51.100.50
# Serveur web : 192.168.1.100

# 1. DNAT : rediriger vers le serveur interne
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100

# 2. SNAT : masquer l'origine pour que la réponse revienne au firewall
nft add rule inet nat postrouting oif eth1 ip daddr 192.168.1.100 tcp dport 80 \
    snat to 192.168.1.1

# Sans le SNAT, le serveur répondrait directement au client, bypass du firewall !
```

> [!warning] Le piège du routage asymétrique
> Si votre serveur interne a le firewall comme passerelle par défaut, pas besoin de SNAT supplémentaire. Mais si le serveur a une autre route, le SNAT est obligatoire pour forcer le retour par le firewall.

---

## Syntaxe et exemples pratiques

### Configuration complète d'un firewall NAT

Voici une configuration réaliste pour un petit réseau d'entreprise :

```bash
#!/usr/sbin/nft -f

# Vider la configuration existante
flush ruleset

# ============================================
# TABLE FILTER - SÉCURITÉ
# ============================================
table inet filter {
    # Chaîne INPUT - trafic vers le firewall
    chain input {
        type filter hook input priority filter; policy drop;
        
        # Loopback
        iif lo accept
        
        # États établis
        ct state established,related accept
        ct state invalid drop
        
        # ICMP
        ip protocol icmp accept
        ip6 nexthdr icmpv6 accept
        
        # SSH depuis le LAN uniquement
        iif eth1 tcp dport 22 accept
        
        # Log et drop le reste
        limit rate 1/minute log prefix "INPUT-DROP: "
        drop
    }
    
    # Chaîne FORWARD - trafic routé
    chain forward {
        type filter hook forward priority filter; policy drop;
        
        # États établis
        ct state established,related accept
        ct state invalid drop
        
        # LAN vers Internet (SNAT/Masquerading)
        iif eth1 oif eth0 ip saddr 192.168.1.0/24 accept
        
        # Internet vers DMZ (DNAT)
        iif eth0 oif eth2 ct status dnat accept
        
        # Log et drop
        limit rate 1/minute log prefix "FORWARD-DROP: "
        drop
    }
    
    # Chaîne OUTPUT - trafic depuis le firewall
    chain output {
        type filter hook output priority filter; policy accept;
    }
}

# ============================================
# TABLE NAT - TRANSLATION D'ADRESSES
# ============================================
table inet nat {
    # Chaîne PREROUTING - DNAT
    chain prerouting {
        type nat hook prerouting priority dstnat; policy accept;
        
        # Serveur web DMZ (80 et 443)
        iif eth0 tcp dport 80 dnat to 192.168.2.10
        iif eth0 tcp dport 443 dnat to 192.168.2.10
        
        # Serveur mail DMZ (25, 465, 587, 993)
        iif eth0 tcp dport { 25, 465, 587, 993 } dnat to 192.168.2.20
        
        # SSH administratif vers serveur spécifique (port non-standard)
        iif eth0 tcp dport 2222 dnat to 192.168.1.50:22
    }
    
    # Chaîne POSTROUTING - SNAT/Masquerading
    chain postrouting {
        type nat hook postrouting priority srcnat; policy accept;
        
        # Masquerading LAN vers Internet
        oif eth0 ip saddr 192.168.1.0/24 masquerade
        
        # SNAT DMZ vers Internet (avec IP fixe pour les logs externes)
        oif eth0 ip saddr 192.168.2.0/24 snat to 198.51.100.50
    }
}
```

**Architecture du réseau :**
- `eth0` : Interface WAN (Internet) - 198.51.100.50
- `eth1` : Interface LAN - 192.168.1.1/24
- `eth2` : Interface DMZ - 192.168.2.1/24

### DNAT avec maps (multi-services)

Pour gérer plusieurs redirections de manière élégante :

```bash
# Création de la map port → serveur
nft add map inet nat web_servers { type inet_service : ipv4_addr \; }

# Ajout des associations
nft add element inet nat web_servers { 80 : 192.168.1.100 }
nft add element inet nat web_servers { 443 : 192.168.1.100 }
nft add element inet nat web_servers { 8080 : 192.168.1.101 }
nft add element inet nat web_servers { 8443 : 192.168.1.101 }

# Règle DNAT utilisant la map
nft add rule inet nat prerouting iif eth0 tcp dport vmap @web_servers dnat to
```

> [!tip] Avantages des maps pour le DNAT
> - Gestion centralisée des redirections
> - Ajout/suppression dynamique sans recharger les règles
> - Code plus lisible et maintenable
> - Performance optimale (lookup en O(1))

### Hairpin NAT (NAT loopback)

Permettre aux clients internes d'accéder aux services via l'IP publique :

```bash
# Configuration classique : ne fonctionne PAS depuis le LAN
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100

# Solution : ajouter une règle pour le trafic interne
nft add rule inet nat prerouting iif eth1 ip daddr 198.51.100.50 tcp dport 80 \
    dnat to 192.168.1.100

# SNAT pour forcer le retour par le firewall
nft add rule inet nat postrouting oif eth1 ip saddr 192.168.1.0/24 \
    ip daddr 192.168.1.100 tcp dport 80 snat to 192.168.1.1
```

> [!warning] Complexité du Hairpin NAT
> Le Hairpin NAT (ou NAT loopback) est complexe car :
> - Le paquet doit être NATé deux fois (DNAT puis SNAT)
> - Le routage doit être correct
> - Préférez utiliser le DNS split-horizon quand possible

### Port knocking avec sets

Implémenter un système de "port knocking" simple :

```bash
# Set pour stocker les IPs autorisées temporairement
nft add set inet filter knock_success { type ipv4_addr \; flags timeout \; }

# Chaîne de knock
nft add chain inet filter knock_chain

# Port de knock : 7000
nft add rule inet filter input tcp dport 7000 jump knock_chain
nft add rule inet filter knock_chain add @knock_success { ip saddr timeout 30s }
nft add rule inet filter knock_chain drop

# SSH accessible seulement après knock
nft add rule inet filter input tcp dport 22 ip saddr @knock_success accept
nft add rule inet filter input tcp dport 22 drop
```

### NAT avec load balancing

Répartir la charge entre plusieurs serveurs :

```bash
# DNAT avec round-robin entre 3 serveurs
nft add rule inet nat prerouting iif eth0 tcp dport 80 \
    dnat to numgen inc mod 3 map { \
        0 : 192.168.1.100, \
        1 : 192.168.1.101, \
        2 : 192.168.1.102 \
    }

# Avec poids différents (70% serveur 1, 30% serveur 2)
nft add rule inet nat prerouting iif eth0 tcp dport 80 \
    dnat to numgen random mod 10 map { \
        0-6 : 192.168.1.100, \
        7-9 : 192.168.1.101 \
    }
```

> [!info] Load balancing avec nftables
> nftables offre du load balancing basique via `numgen` :
> - `numgen inc` : round-robin
> - `numgen random` : aléatoire
> 
> Pour du load balancing avancé (health checks, sticky sessions), utilisez des solutions dédiées (HAProxy, nginx).

---

## Comparaison avec iptables

### Différences de syntaxe

#### SNAT / Masquerading

```bash
# iptables
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -j SNAT --to-source 203.0.113.50
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# nftables
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 snat to 203.0.113.50
nft add rule inet nat postrouting oif eth0 masquerade
```

#### DNAT / Port forwarding

```bash
# iptables
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80

# nftables
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100
nft add rule inet nat prerouting iif eth0 tcp dport 8080 dnat to 192.168.1.100:80
```

#### Redirection locale

```bash
# iptables
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j REDIRECT --to-ports 3128

# nftables
nft add rule inet nat prerouting iif eth1 tcp dport 80 redirect to 3128
```

### Tableau comparatif complet

| Fonctionnalité | iptables | nftables |
|----------------|----------|----------|
| **Syntaxe** | `-t nat -A PREROUTING -j DNAT --to` | `add rule inet nat prerouting dnat to` |
| **Tables séparées** | Oui (`-t nat`) | Non (table comme élément de syntaxe) |
| **Famille IP** | iptables/ip6tables séparés | `inet` pour IPv4+IPv6 simultané |
| **Load balancing** | Module `nth`, `random` | `numgen inc/random` intégré |
| **Multi-DNAT** | Règles multiples | Maps élégantes |
| **Performance** | Bonne | Meilleure (règles compilées) |
| **Atomicité** | Non (commandes successives) | Oui (fichier de config) |
| **Lisibilité** | Verbeux | Plus concis et clair |

### Avantages de nftables pour le NAT

> [!tip] Pourquoi migrer vers nftables ?
> 1. **Syntaxe unifiée** : Plus de `-t nat`, tout est cohérent
> 2. **IPv4/IPv6 simultané** : Une règle pour les deux avec `inet`
> 3. **Maps pour multi-DNAT** : Gestion centralisée, dynamique
> 4. **Meilleure performance** : Moins de traversées du kernel
> 5. **Configuration atomique** : Charger un fichier complet en une fois
> 6. **Load balancing intégré** : `numgen` plus simple que `nth`/`random`
> 7. **Pas de chaînes implicites** : Vous créez exactement ce dont vous avez besoin

### Migration d'une règle iptables complexe

**Scénario iptables :**
```bash
# Règle iptables avec SNAT conditionnel et log
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -p tcp --dport 80 \
    -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "NAT-HTTP: "
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -p tcp --dport 80 \
    -j SNAT --to-source 203.0.113.50
```

**Équivalent nftables :**
```bash
# Une seule règle fait tout !
nft add rule inet nat postrouting oif eth0 ip saddr 192.168.1.0/24 tcp dport 80 \
    limit rate 5/minute burst 10 packets log prefix "NAT-HTTP: " \
    snat to 203.0.113.50
```

> [!example] Gain en lisibilité
> nftables permet de chaîner plusieurs actions dans une règle, rendant la configuration plus compacte et lisible.

---

## 🎯 Points clés à retenir

1. **Activer le forwarding IP** : `echo 1 > /proc/sys/net/ipv4/ip_forward` est OBLIGATOIRE
2. **SNAT vs Masquerading** : SNAT pour IP fixe (plus rapide), masquerading pour IP dynamique
3. **DNAT nécessite le filter** : Autoriser le forward dans la table filter, pas seulement le NAT
4. **Chaînes prerouting/postrouting** : prerouting pour DNAT, postrouting pour SNAT
5. **Maps pour le multi-DNAT** : Plus élégant et performant que des règles multiples
6. **Hairpin NAT est complexe** : Préférez le DNS split-horizon quand possible
7. **nftables simplifie** : Syntaxe unifiée, IPv4+IPv6, meilleure performance

> [!example] Configuration minimale fonctionnelle
> ```bash
> # Activer le forwarding
> echo 1 > /proc/sys/net/ipv4/ip_forward
> 
> # Table NAT
> nft add table inet nat
> nft add chain inet nat postrouting { type nat hook postrouting priority srcnat \; policy accept \; }
> nft add rule inet nat postrouting oif eth0 masquerade
> 
> # Table filter (autoriser le forward)
> nft add table inet filter
> nft add chain inet filter forward { type filter hook forward priority filter \; policy drop \; }
> nft add rule inet filter forward ct state established,related accept
> nft add rule inet filter forward iif eth1 oif eth0 accept
> ```

> [!warning] Pièges courants
> - ❌ Oublier `ip_forward = 1` → rien ne fonctionne
> - ❌ NAT sans règles de forward → paquets bloqués
> - ❌ DNAT sans considérer le routage retour → connexions asymétriques
> - ❌ Masquerading sur interface avec IP fixe → perte de performance
> - ❌ Oublier les règles related pour les protocoles complexes (FTP, SIP)

---

*Ce document fait partie de la série sur Netfilter - Partie 5 : Introduction à nftables*