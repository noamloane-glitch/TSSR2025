## 📑 Table des matières

- [Introduction](#introduction)
- [Syntaxe des règles](#syntaxe-des-règles)
- [Expressions de correspondance](#expressions-de-correspondance)
- [Verdicts et actions](#verdicts-et-actions)
- [Sets et maps](#sets-et-maps)

---

## Introduction

Les règles constituent le cœur opérationnel de nftables. Chaque règle définit **une condition de filtrage** et **une action à effectuer** lorsque cette condition est remplie. Contrairement à iptables qui nécessitait différentes commandes selon les tables, nftables unifie tout avec une syntaxe cohérente et plus lisible.

> [!info] Pourquoi c'est important
> La maîtrise des règles et correspondances vous permet de :
> - Créer des politiques de sécurité précises et efficaces
> - Filtrer le trafic avec une granularité fine
> - Optimiser les performances avec les sets et maps
> - Déboguer rapidement les problèmes de connectivité

---

## Syntaxe des règles

### Structure générale d'une règle

```bash
nft add rule <famille> <table> <chaîne> <correspondances> <verdict>
```

**Décomposition :**
- `<famille>` : ip, ip6, inet, arp, bridge, netdev
- `<table>` : nom de la table contenant la chaîne
- `<chaîne>` : nom de la chaîne où ajouter la règle
- `<correspondances>` : critères de filtrage (expressions)
- `<verdict>` : action à effectuer (accept, drop, reject, etc.)

### Exemples de syntaxe de base

```bash
# Règle simple : autoriser le trafic SSH entrant
nft add rule inet filter input tcp dport 22 accept

# Règle avec plusieurs correspondances
nft add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 80 accept

# Règle avec correspondance d'état
nft add rule inet filter input ct state established,related accept
```

> [!tip] Ordre des éléments
> L'ordre des correspondances n'affecte pas le fonctionnement, mais par convention :
> 1. Adresses source/destination
> 2. Protocole
> 3. Ports
> 4. États de connexion
> 5. Verdict

### Positionnement des règles

```bash
# Ajouter à la fin (par défaut)
nft add rule inet filter input tcp dport 443 accept

# Insérer au début
nft insert rule inet filter input tcp dport 443 accept

# Insérer à une position spécifique (après handle X)
nft add rule inet filter input position 5 tcp dport 443 accept

# Lister avec les handles pour référence
nft -a list chain inet filter input
```

> [!warning] Attention à l'ordre des règles
> Les règles sont évaluées **séquentiellement**. Une règle trop permissive placée avant une règle restrictive rendra la seconde inefficace !

### Gestion des règles existantes

```bash
# Lister toutes les règles d'une chaîne avec leurs handles
nft -a list chain inet filter input

# Supprimer une règle par son handle
nft delete rule inet filter input handle 10

# Remplacer une règle
nft replace rule inet filter input handle 10 tcp dport 8080 accept

# Vider toutes les règles d'une chaîne
nft flush chain inet filter input
```

---

## Expressions de correspondance

Les expressions de correspondance définissent les **critères de filtrage** des paquets. nftables offre une grande richesse d'expressions.

### Correspondances par adresse IP

```bash
# Adresse source IPv4
nft add rule inet filter input ip saddr 192.168.1.100 drop

# Adresse destination IPv4
nft add rule inet filter output ip daddr 10.0.0.0/8 accept

# Adresse source IPv6
nft add rule inet filter input ip6 saddr 2001:db8::/32 accept

# Plage d'adresses
nft add rule inet filter input ip saddr 192.168.1.10-192.168.1.50 accept

# Négation (tout sauf cette adresse)
nft add rule inet filter input ip saddr != 192.168.1.100 drop
```

> [!info] Famille inet pour IPv4 et IPv6
> Avec la famille `inet`, utilisez :
> - `ip saddr/daddr` pour IPv4
> - `ip6 saddr/daddr` pour IPv6
> 
> Une règle peut cibler les deux versions simultanément !

### Correspondances par protocole

```bash
# Protocole par nom
nft add rule inet filter input ip protocol tcp accept
nft add rule inet filter input ip protocol udp accept
nft add rule inet filter input ip protocol icmp accept

# Protocole par numéro
nft add rule inet filter input ip protocol 6 accept  # TCP = 6

# Négation
nft add rule inet filter input ip protocol != tcp drop
```

**Protocoles courants :**
| Nom | Numéro | Usage |
|-----|--------|-------|
| tcp | 6 | Connexions fiables |
| udp | 17 | Connexions rapides |
| icmp | 1 | Ping, diagnostics |
| icmpv6 | 58 | Ping IPv6 |
| esp | 50 | IPsec |
| gre | 47 | Tunnels |

### Correspondances par port

```bash
# Port destination simple
nft add rule inet filter input tcp dport 22 accept

# Port source
nft add rule inet filter output tcp sport 1024-65535 accept

# Ports multiples (liste)
nft add rule inet filter input tcp dport { 22, 80, 443 } accept

# Plage de ports
nft add rule inet filter input tcp dport 8000-8999 accept

# Négation
nft add rule inet filter input tcp dport != 22 drop
```

> [!tip] Optimisation avec les sets anonymes
> La syntaxe `{ 22, 80, 443 }` crée un set anonyme, bien plus performant que trois règles distinctes !

### Correspondances par interface réseau

```bash
# Interface d'entrée
nft add rule inet filter input iif eth0 accept

# Interface de sortie
nft add rule inet filter output oif wlan0 accept

# Interface par nom avec wildcard
nft add rule inet filter input iifname "eth*" accept

# Négation
nft add rule inet filter input iif != lo drop
```

> [!warning] iif vs iifname
> - `iif` : utilise l'index de l'interface (plus rapide)
> - `iifname` : utilise le nom (nécessaire pour les interfaces dynamiques comme Docker)

### Correspondances TCP/UDP avancées

```bash
# Flags TCP
nft add rule inet filter input tcp flags syn tcp flags != ack drop

# Filtrer les SYN flood (nouvelle connexion)
nft add rule inet filter input tcp flags & (fin|syn|rst|ack) == syn \
    ct state new limit rate 10/second accept

# Bloquer les paquets invalides
nft add rule inet filter input tcp flags & (fin|syn|rst|psh|ack|urg) == fin|psh|urg drop

# Fragments IP
nft add rule inet filter input ip frag-off & 0x1fff != 0 drop
```

**Flags TCP courants :**
| Flag | Signification |
|------|---------------|
| syn | Demande de connexion |
| ack | Accusé de réception |
| fin | Fermeture de connexion |
| rst | Réinitialisation |
| psh | Données à traiter immédiatement |
| urg | Données urgentes |

### Correspondances par état de connexion (conntrack)

```bash
# États établis et connexes
nft add rule inet filter input ct state established,related accept

# Nouvelle connexion uniquement
nft add rule inet filter input ct state new tcp dport 80 accept

# Bloquer les paquets invalides
nft add rule inet filter input ct state invalid drop

# État DNAT (après translation)
nft add rule inet filter forward ct status dnat accept
```

**États de connexion :**
| État | Description |
|------|-------------|
| new | Premier paquet d'une nouvelle connexion |
| established | Paquet appartenant à une connexion établie |
| related | Paquet lié à une connexion existante (FTP data, ICMP error) |
| invalid | Paquet ne correspondant à aucune connexion |

> [!tip] La règle d'or du stateful firewall
> Placez toujours en début de chaîne input :
> ```bash
> nft add rule inet filter input ct state established,related accept
> ```
> Cela autorise automatiquement les réponses aux connexions sortantes !

### Correspondances par limite de débit (rate limiting)

```bash
# Limiter le nombre de nouvelles connexions SSH
nft add rule inet filter input tcp dport 22 ct state new \
    limit rate 3/minute accept

# Limiter avec burst
nft add rule inet filter input tcp dport 22 ct state new \
    limit rate 10/minute burst 5 packets accept

# Limiter la bande passante (avec quota)
nft add rule inet filter forward quota 10 gbytes drop
```

> [!info] Rate vs Burst
> - **rate** : taux moyen autorisé
> - **burst** : pics temporaires tolérés au-delà du taux moyen

### Correspondances par longueur de paquet

```bash
# Bloquer les paquets trop petits
nft add rule inet filter input ip length < 60 drop

# Bloquer les paquets trop grands
nft add rule inet filter input ip length > 1500 drop

# Plage de longueur
nft add rule inet filter input ip length 100-1400 accept
```

### Correspondances par marquage (marks)

```bash
# Définir un mark
nft add rule inet filter forward meta mark set 1

# Filtrer par mark
nft add rule inet filter output meta mark 1 accept

# Mark combiné avec masque
nft add rule inet filter forward meta mark and 0xff == 0x01 accept
```

> [!info] À quoi servent les marks ?
> Les marks permettent de "marquer" des paquets pour :
> - Le routage avancé (policy routing)
> - La QoS (Quality of Service)
> - Partager des informations entre tables

---

## Verdicts et actions

Les verdicts déterminent **ce qui arrive au paquet** une fois qu'il correspond à une règle.

### Verdicts terminaux

Ces verdicts **arrêtent l'évaluation** des règles dans la chaîne courante.

```bash
# ACCEPT : autoriser le paquet
nft add rule inet filter input tcp dport 22 accept

# DROP : rejeter silencieusement (pas de réponse)
nft add rule inet filter input ip saddr 10.0.0.5 drop

# REJECT : rejeter avec message d'erreur
nft add rule inet filter input tcp dport 23 reject

# REJECT avec type de réponse personnalisé
nft add rule inet filter input tcp dport 23 reject with tcp reset
nft add rule inet filter input udp dport 53 reject with icmp type port-unreachable
```

> [!warning] DROP vs REJECT
> - **DROP** : le paquet disparaît, pas de réponse → l'attaquant ne sait pas si le service existe
> - **REJECT** : envoie un message d'erreur → plus convivial pour les utilisateurs légitimes
> 
> Utilisez DROP pour les règles anti-scan, REJECT pour les services fermés aux utilisateurs.

### Verdicts de saut (non-terminaux)

Ces verdicts **transfèrent l'évaluation** vers une autre chaîne.

```bash
# JUMP : aller vers une chaîne, puis revenir
nft add rule inet filter input tcp dport 80 jump web_rules

# GOTO : aller vers une chaîne sans retour
nft add rule inet filter input tcp dport 443 goto web_rules

# RETURN : revenir à la chaîne appelante
nft add rule inet filter web_rules tcp flags syn accept
nft add rule inet filter web_rules return  # Retour après traitement
```

> [!tip] JUMP vs GOTO
> - **JUMP** : comme un appel de fonction, retourne après traitement
> - **GOTO** : comme un goto en programmation, pas de retour
> 
> JUMP est plus flexible et généralement recommandé.

### Verdicts de logging

```bash
# LOG : journaliser puis continuer
nft add rule inet filter input tcp dport 22 log prefix "SSH: " accept

# LOG avec niveau et groupe
nft add rule inet filter input log level warn group 2 prefix "ALERT: " drop

# LOG avec options avancées
nft add rule inet filter input log prefix "DROP: " \
    log level info flags all drop
```

**Niveaux de log :**
- `emerg`, `alert`, `crit`, `err`, `warn`, `notice`, `info`, `debug`

> [!warning] Performance du logging
> Le logging peut **ralentir significativement** le firewall. Utilisez-le avec modération, idéalement avec rate limiting :
> ```bash
> nft add rule inet filter input limit rate 1/minute log prefix "DROP: " drop
> ```

### Verdicts de compteur

```bash
# COUNTER : compter les paquets et octets
nft add rule inet filter input tcp dport 80 counter accept

# Combinaison avec d'autres verdicts
nft add rule inet filter input tcp dport 22 counter log prefix "SSH: " accept

# Lister les statistiques
nft list chain inet filter input
```

### Verdicts de manipulation (NAT/Modification)

```bash
# SNAT : changer l'adresse source
nft add rule inet nat postrouting oif eth0 snat to 203.0.113.1

# MASQUERADE : SNAT dynamique (IP changeante)
nft add rule inet nat postrouting oif eth0 masquerade

# DNAT : changer l'adresse destination
nft add rule inet nat prerouting iif eth0 tcp dport 80 dnat to 192.168.1.100

# REDIRECT : rediriger vers un port local
nft add rule inet nat prerouting tcp dport 80 redirect to 8080
```

> [!info] Ces verdicts seront détaillés dans la partie NAT
> Ils sont mentionnés ici pour compléter la liste des verdicts possibles.

### Verdicts de métadonnées

```bash
# Définir un mark
nft add rule inet filter forward meta mark set 1 accept

# Définir une priority (QoS)
nft add rule inet filter forward meta priority set 1 accept

# Modifier le TTL
nft add rule inet filter forward ip ttl set 64 accept
```

---

## Sets et maps

Les sets et maps permettent de **regrouper des éléments** (adresses, ports, etc.) pour créer des règles plus efficaces et maintenables.

### Sets : ensembles d'éléments

Les sets stockent des **listes d'éléments** qu'on peut réutiliser dans plusieurs règles.

#### Création d'un set

```bash
# Set d'adresses IPv4
nft add set inet filter blacklist { type ipv4_addr \; }

# Set de ports
nft add set inet filter allowed_ports { type inet_service \; }

# Set avec timeout automatique
nft add set inet filter temp_block { type ipv4_addr \; timeout 1h \; }

# Set avec flag constant (non modifiable après création)
nft add set inet filter trusted_ips { type ipv4_addr \; flags constant \; }

# Set avec intervalle (plages)
nft add set inet filter ip_ranges { type ipv4_addr \; flags interval \; }
```

**Types de données courants :**
| Type | Description | Exemple |
|------|-------------|---------|
| ipv4_addr | Adresse IPv4 | 192.168.1.1 |
| ipv6_addr | Adresse IPv6 | 2001:db8::1 |
| ether_addr | Adresse MAC | aa:bb:cc:dd:ee:ff |
| inet_proto | Protocole | tcp, udp |
| inet_service | Port | 22, 80, 443 |
| mark | Marqueur | 0x1 |

#### Ajout d'éléments dans un set

```bash
# Ajouter un élément
nft add element inet filter blacklist { 10.0.0.5 }

# Ajouter plusieurs éléments
nft add element inet filter blacklist { 10.0.0.5, 10.0.0.6, 10.0.0.7 }

# Ajouter avec timeout individuel
nft add element inet filter temp_block { 192.168.1.50 timeout 30m }

# Ajouter un intervalle (nécessite flag interval)
nft add element inet filter ip_ranges { 192.168.1.0-192.168.1.255 }
nft add element inet filter ip_ranges { 10.0.0.0/8 }
```

#### Utilisation d'un set dans une règle

```bash
# Bloquer toutes les IP du set blacklist
nft add rule inet filter input ip saddr @blacklist drop

# Autoriser les ports du set
nft add rule inet filter input tcp dport @allowed_ports accept

# Négation
nft add rule inet filter input ip saddr != @blacklist accept
```

> [!tip] Performance des sets
> Les sets utilisent des **structures de données optimisées** (hash tables, arbres) :
> - Recherche en O(1) ou O(log n)
> - Bien plus rapide que des règles multiples
> - Idéal pour les grandes listes (blacklists, etc.)

#### Gestion des sets

```bash
# Lister les sets
nft list sets

# Lister le contenu d'un set
nft list set inet filter blacklist

# Supprimer un élément
nft delete element inet filter blacklist { 10.0.0.5 }

# Vider un set
nft flush set inet filter blacklist

# Supprimer un set (doit être vide ou sans règles l'utilisant)
nft delete set inet filter blacklist
```

### Sets anonymes

Pour des listes courtes utilisées une seule fois, utilisez des **sets anonymes** directement dans la règle :

```bash
# Set anonyme d'adresses
nft add rule inet filter input ip saddr { 192.168.1.10, 192.168.1.20 } accept

# Set anonyme de ports
nft add rule inet filter input tcp dport { 22, 80, 443 } accept

# Set anonyme avec CIDR
nft add rule inet filter input ip saddr { 10.0.0.0/8, 172.16.0.0/12 } drop
```

> [!info] Sets nommés vs anonymes
> - **Sets nommés** : réutilisables, modifiables dynamiquement, meilleurs pour de grandes listes
> - **Sets anonymes** : pratiques pour des listes courtes statiques

### Maps : dictionnaires clé → valeur

Les maps associent des **clés à des valeurs** pour effectuer des actions différentes selon l'élément.

#### Création d'une map

```bash
# Map IP → verdict
nft add map inet filter ip_verdict { type ipv4_addr : verdict \; }

# Map IP → mark
nft add map inet filter ip_marks { type ipv4_addr : mark \; }

# Map port → adresse (pour DNAT)
nft add map inet nat port_forward { type inet_service : ipv4_addr \; }
```

**Types de valeurs courants :**
- `verdict` : accept, drop, jump chain_name
- `mark` : marqueur numérique
- `ipv4_addr`, `ipv6_addr` : adresse IP
- `inet_service` : numéro de port

#### Ajout d'éléments dans une map

```bash
# Ajouter des associations IP → verdict
nft add element inet filter ip_verdict { 192.168.1.10 : accept }
nft add element inet filter ip_verdict { 10.0.0.5 : drop }

# Ajouter des associations IP → mark
nft add element inet filter ip_marks { 192.168.1.0/24 : 0x1 }
nft add element inet filter ip_marks { 192.168.2.0/24 : 0x2 }

# Ajouter des associations port → IP (DNAT)
nft add element inet nat port_forward { 8080 : 192.168.1.100 }
nft add element inet nat port_forward { 8443 : 192.168.1.101 }
```

#### Utilisation d'une map dans une règle

```bash
# Appliquer le verdict selon l'IP source
nft add rule inet filter input ip saddr vmap @ip_verdict

# Définir le mark selon l'IP source
nft add rule inet filter forward ip saddr map @ip_marks meta mark set

# DNAT selon le port
nft add rule inet nat prerouting tcp dport vmap @port_forward
```

> [!warning] map vs vmap
> - `map` : retourne une valeur utilisée dans l'expression
> - `vmap` : retourne un verdict (action terminale)
> 
> ```bash
> # map : assigne une valeur
> ip saddr map @ip_marks meta mark set
> 
> # vmap : exécute un verdict
> ip saddr vmap @ip_verdict
> ```

### Cas d'usage pratiques des sets et maps

#### Blacklist dynamique

```bash
# Création
nft add set inet filter blacklist { type ipv4_addr \; flags timeout \; }

# Règle
nft add rule inet filter input ip saddr @blacklist drop

# Ajout dynamique (via script)
nft add element inet filter blacklist { 203.0.113.45 timeout 1h }
```

#### Whitelist de services

```bash
# Set d'IPs autorisées pour SSH
nft add set inet filter ssh_allowed { type ipv4_addr \; }
nft add element inet filter ssh_allowed { 192.168.1.0/24, 10.0.0.0/8 }

# Règle
nft add rule inet filter input tcp dport 22 ip saddr @ssh_allowed accept
nft add rule inet filter input tcp dport 22 drop
```

#### Redirection multi-ports (DNAT)

```bash
# Map port → serveur interne
nft add map inet nat web_servers { type inet_service : ipv4_addr \; }
nft add element inet nat web_servers { 80 : 192.168.1.10 }
nft add element inet nat web_servers { 8080 : 192.168.1.11 }
nft add element inet nat web_servers { 8443 : 192.168.1.12 }

# Règle DNAT
nft add rule inet nat prerouting tcp dport vmap @web_servers dnat to
```

#### QoS par réseau source

```bash
# Map réseau → priority
nft add map inet filter qos_marks { type ipv4_addr : mark \; flags interval \; }
nft add element inet filter qos_marks { 192.168.1.0/24 : 0x1 }  # Prioritaire
nft add element inet filter qos_marks { 192.168.2.0/24 : 0x2 }  # Normal
nft add element inet filter qos_marks { 192.168.3.0/24 : 0x3 }  # Bas

# Règle
nft add rule inet filter forward ip saddr map @qos_marks meta mark set
```

> [!tip] Gestion centralisée
> Les sets et maps permettent de **centraliser la configuration** :
> - Modifier un set sans recharger toutes les règles
> - Scripts d'administration simplifiés
> - Maintenance facilitée

### Bonnes pratiques avec les sets et maps

> [!tip] Recommandations
> 1. **Utilisez des sets nommés** pour les listes réutilisées ou modifiables
> 2. **Utilisez des sets anonymes** pour les listes courtes statiques
> 3. **Ajoutez des timeouts** pour les blacklists temporaires
> 4. **Documentez vos sets** avec des noms explicites (`ssh_allowed`, pas `set1`)
> 5. **Utilisez les intervals** pour les plages IP/ports (plus efficace que des éléments multiples)

> [!warning] Pièges à éviter
> - Ne supprimez pas un set utilisé par des règles actives
> - Attention aux timeouts : vérifiez régulièrement les expirations
> - Les sets avec `flags constant` ne peuvent plus être modifiés après création
> - Les maps avec type `verdict` nécessitent `vmap`, pas `map`

---

## 🎯 Points clés à retenir

1. **Syntaxe unifiée** : Une seule structure pour toutes les règles, plus lisible qu'iptables
2. **Ordre des règles** : Évaluation séquentielle, l'ordre est crucial
3. **Correspondances riches** : Adresses, ports, protocoles, états, interfaces, limites, etc.
4. **Verdicts variés** : Terminaux (accept, drop, reject) et non-terminaux (jump, goto)
5. **Sets pour l'efficacité** : Recherche optimisée, meilleur que des règles multiples
6. **Maps pour la flexibilité** : Associations clé→valeur, idéales pour le DNAT et le marquage
7. **Performance** : Privilégiez les sets, limitez le logging, utilisez les états conntrack

> [!example] Règle complète typique
> ```bash
> # Autoriser SSH depuis le réseau admin, avec rate limiting et log
> nft add rule inet filter input \
>     ip saddr @admin_network \
>     tcp dport 22 \
>     ct state new \
>     limit rate 3/minute \
>     log prefix "SSH-ACCEPT: " \
>     counter \
>     accept
> ```

---

*Ce document fait partie de la série sur Netfilter - Partie 5 : Introduction à nftables*