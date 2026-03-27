## 📋 Table des matières

- [Structure des commandes nft](#structure-des-commandes-nft)
- [Tables et chaînes dans nftables](#tables-et-chaînes-dans-nftables)
- [Familles d'adresses](#familles-dadresses)
- [Types de chaînes](#types-de-chaînes)

---

## Structure des commandes nft

### Anatomie d'une commande nft

La syntaxe de `nft` suit une structure logique et cohérente, contrairement aux options multiples d'iptables.

```bash
# Structure générale
nft [options] COMMANDE [OBJET] [PARAMÈTRES]

# Exemple concret
nft add rule inet filter input tcp dport 22 accept
#   └─┬─┘ └┬┘ └──┬──┘ └─┬──┘ └───┬──────────────┘ └──┬──┘
#   cmd   obj famille  table  chain  critères      action
```

### Les commandes principales

| Commande | Description | Équivalent iptables |
|----------|-------------|-------------------|
| **add** | Ajouter un objet (règle, table, chaîne...) | -A (append) |
| **insert** | Insérer un objet à une position | -I (insert) |
| **delete** | Supprimer un objet | -D (delete) |
| **replace** | Remplacer un objet | - |
| **list** | Afficher les objets | -L (list) |
| **flush** | Vider des objets | -F (flush) |
| **create** | Créer (erreur si existe déjà) | -N (new chain) |

> [!info] Différence add vs create
> - **add** : Ajoute, même si l'objet existe déjà (pas d'erreur)
> - **create** : Crée uniquement si n'existe pas (erreur si existe)

### Les objets manipulables

```bash
# Table
nft add table inet filter

# Chaîne
nft add chain inet filter input

# Règle
nft add rule inet filter input tcp dport 22 accept

# Set (ensemble)
nft add set inet filter blacklist { type ipv4_addr\; }

# Map (dictionnaire)
nft add map inet nat portmap { type inet_service : ipv4_addr\; }

# Element (élément d'un set/map)
nft add element inet filter blacklist { 192.168.1.100 }

# Counter (compteur nommé)
nft add counter inet filter ssh_counter

# Quota (quota de bande passante)
nft add quota inet filter monthly_quota { over 100 mbytes }
```

### Options globales

```bash
# Mode interactif
nft -i
nft> add table inet filter
nft> list ruleset
nft> quit

# Lire depuis un fichier
nft -f /etc/nftables.conf

# Mode numérique (pas de résolution DNS/services)
nft -n list ruleset

# Sortie JSON (pour parsing automatique)
nft -j list ruleset

# Mode verbeux
nft -v add rule inet filter input tcp dport 22 accept

# Vérifier sans appliquer (dry-run)
nft -c -f /etc/nftables.conf
```

> [!tip] Mode interactif pour l'apprentissage
> ```bash
> # Lancez nft en mode interactif pour tester
> nft -i
> 
> # Tapez vos commandes
> nft> list ruleset
> nft> add table inet test
> nft> list tables
> 
> # Quittez avec Ctrl+D ou quit
> nft> quit
> ```

### Syntaxe des règles

```bash
# Structure complète d'une règle
nft add rule [famille] table chaîne [handle HANDLE] critère1 critère2 ... action

# Exemples progressifs
# Simple
nft add rule inet filter input accept

# Avec critère de protocole
nft add rule inet filter input tcp dport 22 accept

# Avec multiples critères
nft add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 22 accept

# Avec compteur
nft add rule inet filter input tcp dport 80 counter accept

# Avec log
nft add rule inet filter input tcp dport 23 log prefix "TELNET: " drop
```

> [!example] Comparaison syntaxe iptables vs nftables
> ```bash
> # iptables : options avec tirets
> iptables -t filter -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT
> 
> # nftables : syntaxe naturelle sans tirets
> nft add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 22 accept
> 
> # Plus lisible, plus proche de la langue naturelle
> ```

### Gestion des handles (identifiants de règles)

Chaque règle possède un **handle** (identifiant unique) attribué automatiquement.

```bash
# Afficher les handles
nft -a list ruleset
# Sortie :
# table inet filter { # handle 1
#   chain input { # handle 1
#     type filter hook input priority 0; policy accept;
#     tcp dport 22 accept # handle 5
#     tcp dport 80 accept # handle 6
#   }
# }

# Supprimer une règle par handle
nft delete rule inet filter input handle 5

# Insérer une règle avant un handle spécifique
nft insert rule inet filter input position 6 tcp dport 443 accept

# Remplacer une règle
nft replace rule inet filter input handle 6 tcp dport 8080 accept
```

> [!warning] Handles changeants
> Les handles peuvent changer après un flush ou un rechargement. Ne les hardcodez pas dans vos scripts !

---

## Tables et chaînes dans nftables

### Concept de table

Une **table** est un conteneur pour des chaînes. Contrairement à iptables, il n'y a **pas de tables prédéfinies**.

> [!info] Liberté totale
> Avec nftables, VOUS décidez :
> - Combien de tables créer
> - Comment les nommer
> - Ce qu'elles contiennent
> 
> Il n'y a pas de tables "filter", "nat", "mangle" obligatoires.

### Créer et gérer des tables

```bash
# Créer une table
nft add table inet filter
nft add table inet ma_table_personnalisee
nft add table ip firewall_ipv4_only

# Lister les tables
nft list tables
# Sortie :
# table inet filter
# table inet ma_table_personnalisee

# Lister le contenu d'une table
nft list table inet filter

# Vider une table (supprimer toutes ses chaînes et règles)
nft flush table inet filter

# Supprimer une table (doit être vide)
nft delete table inet filter
```

> [!tip] Convention de nommage
> Bien qu'il n'y ait pas d'obligation, beaucoup d'administrateurs utilisent des noms familiers :
> ```bash
> nft add table inet filter    # Pour le filtrage
> nft add table inet nat        # Pour le NAT
> nft add table inet mangle     # Pour la modification
> ```
> C'est optionnel mais aide à la compréhension !

### Attributs d'une table

```bash
# Afficher les tables avec leur famille
nft list tables
# table ip filter      ← IPv4 seulement
# table ip6 filter     ← IPv6 seulement
# table inet filter    ← IPv4 + IPv6
# table arp filter     ← ARP
# table bridge filter  ← Bridge

# Une table appartient à UNE seule famille
```

### Concept de chaîne

Une **chaîne** est une liste ordonnée de règles à l'intérieur d'une table.

> [!info] Types de chaînes
> Il existe deux types de chaînes :
> - **Chaînes de base** : Attachées à un hook Netfilter (input, output, etc.)
> - **Chaînes normales** : Pour organiser vos règles (appelées depuis d'autres chaînes)

### Chaînes de base (base chains)

Les chaînes de base sont le point d'entrée du trafic dans votre ruleset.

```bash
# Syntaxe complète
nft add chain [famille] table chaîne '{ type TYPE hook HOOK priority PRIORITÉ ; policy POLITIQUE ; }'

# Exemple concret
nft add chain inet filter input '{ type filter hook input priority 0 ; policy accept ; }'
#                                 └─type─┘ └──hook──┘ └priority┘  └─policy─┘
```

#### Paramètres d'une chaîne de base

| Paramètre | Description | Valeurs possibles |
|-----------|-------------|------------------|
| **type** | Type de traitement | filter, nat, route |
| **hook** | Point d'accroche Netfilter | prerouting, input, forward, output, postrouting |
| **priority** | Ordre d'exécution | -400 à 400 (0 = standard) |
| **policy** | Politique par défaut | accept, drop |

```bash
# Créer les chaînes de base classiques
nft add chain inet filter input '{
    type filter hook input priority 0 ; policy drop ;
}'

nft add chain inet filter forward '{
    type filter hook forward priority 0 ; policy drop ;
}'

nft add chain inet filter output '{
    type filter hook output priority 0 ; policy accept ;
}'
```

> [!warning] Syntaxe des accolades
> Les accolades `{ }` doivent être **entre guillemets simples** pour éviter l'interprétation par le shell :
> ```bash
> # CORRECT ✅
> nft add chain inet filter input '{ type filter hook input priority 0 ; }'
> 
> # INCORRECT ❌ (le shell interprète les accolades)
> nft add chain inet filter input { type filter hook input priority 0 ; }
> ```

### Chaînes normales (regular chains)

Les chaînes normales servent à organiser vos règles, comme des fonctions.

```bash
# Créer une chaîne normale (sans hook)
nft add chain inet filter services_web

# Ajouter des règles dans cette chaîne
nft add rule inet filter services_web tcp dport 80 accept
nft add rule inet filter services_web tcp dport 443 accept

# Appeler cette chaîne depuis une chaîne de base
nft add rule inet filter input jump services_web

# Ou avec "goto" (ne revient pas)
nft add rule inet filter input goto services_web
```

> [!info] jump vs goto
> - **jump** : Exécute la chaîne cible puis revient continuer
> - **goto** : Exécute la chaîne cible et ne revient PAS
> 
> Similaire à l'appel de fonction vs goto en programmation.

### Gérer les chaînes

```bash
# Lister les chaînes d'une table
nft list chains inet filter

# Afficher le contenu d'une chaîne
nft list chain inet filter input

# Vider une chaîne (supprimer toutes ses règles)
nft flush chain inet filter input

# Supprimer une chaîne (doit être vide et non utilisée)
nft delete chain inet filter services_web

# Renommer une chaîne
nft rename chain inet filter input nouveau_nom
```

### Exemple complet de structure

```bash
# Créer une table
nft add table inet firewall

# Créer des chaînes de base
nft add chain inet firewall input '{
    type filter hook input priority 0 ; policy drop ;
}'

nft add chain inet firewall output '{
    type filter hook output priority 0 ; policy accept ;
}'

# Créer des chaînes normales d'organisation
nft add chain inet firewall ssh_rules
nft add chain inet firewall web_rules
nft add chain inet firewall icmp_rules

# Peupler les chaînes normales
nft add rule inet firewall ssh_rules tcp dport 22 accept
nft add rule inet firewall web_rules tcp dport { 80, 443 } accept
nft add rule inet firewall icmp_rules icmp type echo-request accept

# Appeler ces chaînes depuis input
nft add rule inet firewall input ct state established,related accept
nft add rule inet firewall input jump ssh_rules
nft add rule inet firewall input jump web_rules
nft add rule inet firewall input jump icmp_rules
```

> [!tip] Avantage des chaînes normales
> Organisation modulaire de vos règles :
> - Plus facile à maintenir
> - Réutilisable
> - Lisibilité améliorée
> - Modifications isolées

---

## Familles d'adresses

Les **familles** définissent le type de trafic réseau traité par une table.

### Vue d'ensemble des familles

| Famille | Protocole | Description | Équivalent iptables |
|---------|-----------|-------------|-------------------|
| **ip** | IPv4 | Trafic IPv4 uniquement | iptables |
| **ip6** | IPv6 | Trafic IPv6 uniquement | ip6tables |
| **inet** | IPv4 + IPv6 | Dual-stack (les deux) | iptables + ip6tables |
| **arp** | ARP | Protocole de résolution d'adresse | arptables |
| **bridge** | Bridge | Trafic de pont Ethernet | ebtables |
| **netdev** | Device | Niveau très bas (ingress) | - (nouveau) |

### Famille ip (IPv4)

Traite uniquement le trafic IPv4.

```bash
# Créer une table IPv4
nft add table ip filter

# Créer une chaîne
nft add chain ip filter input '{ type filter hook input priority 0 ; }'

# Ajouter des règles IPv4
nft add rule ip filter input ip saddr 192.168.1.0/24 accept
nft add rule ip filter input ip daddr 203.0.113.5 tcp dport 80 accept

# Critères disponibles spécifiques à ip
nft add rule ip filter input ip saddr 10.0.0.0/8 counter drop
nft add rule ip filter input ip protocol tcp accept
nft add rule ip filter input ip ttl 64 accept
```

> [!example] Cas d'usage
> Utilisez la famille **ip** quand :
> - Vous gérez un réseau IPv4 pur (pas d'IPv6)
> - Vous avez besoin de critères très spécifiques à IPv4
> - Vous voulez des rulesets séparés pour IPv4 et IPv6

### Famille ip6 (IPv6)

Traite uniquement le trafic IPv6.

```bash
# Créer une table IPv6
nft add table ip6 filter

# Créer une chaîne
nft add chain ip6 filter input '{ type filter hook input priority 0 ; }'

# Ajouter des règles IPv6
nft add rule ip6 filter input ip6 saddr 2001:db8::/32 accept
nft add rule ip6 filter input ip6 daddr fe80::/10 accept

# Critères spécifiques IPv6
nft add rule ip6 filter input ip6 nexthdr tcp accept
nft add rule ip6 filter input ip6 hoplimit 64 accept
nft add rule ip6 filter input icmpv6 type echo-request accept
```

> [!warning] Ne pas oublier IPv6
> Erreur fréquente : configurer uniquement IPv4 et laisser IPv6 ouvert !
> ```bash
> # ❌ Dangereux : IPv4 protégé, IPv6 ouvert
> nft add table ip filter
> # ... règles IPv4 seulement ...
> 
> # ✅ Correct : protéger les deux
> nft add table ip filter
> nft add table ip6 filter
> # ... règles pour chaque famille ...
> ```

### Famille inet (Dual-stack IPv4+IPv6)

**La killer feature de nftables** : gérer IPv4 ET IPv6 avec les mêmes règles !

```bash
# Créer une table dual-stack
nft add table inet filter

# Créer une chaîne
nft add chain inet filter input '{ type filter hook input priority 0 ; }'

# UNE règle pour IPv4 ET IPv6
nft add rule inet filter input tcp dport 22 accept
# Fonctionne pour 192.168.1.100:22 ET 2001:db8::1:22

# Critères communs
nft add rule inet filter input ct state established,related accept
nft add rule inet filter input tcp dport { 80, 443 } accept

# Critères spécifiques (préfixe ip ou ip6)
nft add rule inet filter input ip saddr 192.168.1.0/24 accept
nft add rule inet filter input ip6 saddr 2001:db8::/32 accept
```

> [!tip] Recommandation
> **Utilisez inet par défaut** pour vos pare-feu modernes :
> - Moins de code à maintenir
> - Pas de duplication de règles
> - Cohérence garantie entre IPv4 et IPv6
> - Simplifie l'administration

#### Comparaison inet vs ip+ip6

```bash
# Approche ancienne (ip + ip6) : duplication
nft add table ip filter
nft add chain ip filter input '{ type filter hook input priority 0 ; }'
nft add rule ip filter input tcp dport 22 accept
nft add rule ip filter input tcp dport 80 accept
nft add rule ip filter input tcp dport 443 accept

nft add table ip6 filter
nft add chain ip6 filter input '{ type filter hook input priority 0 ; }'
nft add rule ip6 filter input tcp dport 22 accept
nft add rule ip6 filter input tcp dport 80 accept
nft add rule ip6 filter input tcp dport 443 accept

# Approche moderne (inet) : factorisation
nft add table inet filter
nft add chain inet filter input '{ type filter hook input priority 0 ; }'
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input tcp dport 80 accept
nft add rule inet filter input tcp dport 443 accept
# Une seule définition pour les deux protocoles !
```

### Famille arp

Gère le protocole ARP (résolution adresse MAC ↔ IP).

```bash
# Créer une table ARP
nft add table arp filter

# Créer une chaîne
nft add chain arp filter input '{ type filter hook input priority 0 ; }'

# Règles ARP
nft add rule arp filter input arp saddr ip 192.168.1.1 accept
nft add rule arp filter input arp operation request accept
nft add rule arp filter input arp operation reply drop

# Filtrer par MAC
nft add rule arp filter input arp saddr ether aa:bb:cc:dd:ee:ff accept
```

> [!info] Cas d'usage ARP
> - Protection contre ARP spoofing
> - Filtrage sur réseaux locaux sensibles
> - Contrôle d'accès par MAC
> - Rarement utilisé en pratique

### Famille bridge

Gère le trafic sur des ponts Ethernet (bridges).

```bash
# Créer une table bridge
nft add table bridge filter

# Créer une chaîne
nft add chain bridge filter forward '{ type filter hook forward priority 0 ; }'

# Filtrer le trafic ponté
nft add rule bridge filter forward ether type ip accept
nft add rule bridge filter forward ether type ip6 accept
nft add rule bridge filter forward ether type arp accept

# Filtrer par VLAN
nft add rule bridge filter forward vlan id 100 accept
```

> [!example] Cas d'usage bridge
> - Serveurs avec interfaces bridgées (VMs, conteneurs)
> - Filtrage sur switches Linux
> - Contrôle VLAN
> - Cas spécifiques de virtualisation

### Famille netdev

Traitement au niveau le plus bas (ingress = à l'entrée de l'interface réseau).

```bash
# Créer une table netdev (nécessite spécifier l'interface)
nft add table netdev filter

# Créer une chaîne ingress sur une interface spécifique
nft add chain netdev filter ingress_eth0 '{
    type filter hook ingress device eth0 priority 0 ;
}'

# Protection DDoS très précoce
nft add rule netdev filter ingress_eth0 ip saddr 203.0.113.0/24 drop

# Rate limiting avant le stack réseau
nft add rule netdev filter ingress_eth0 limit rate 1000 mbytes/second accept
```

> [!warning] Cas d'usage avancé
> La famille **netdev** est pour des besoins très spécifiques :
> - Protection DDoS au niveau interface
> - Performance maximale (avant le stack IP)
> - Rate limiting hardware-proche
> 
> Pour 99% des cas, utilisez **inet** ou **ip/ip6**.

### Tableau récapitulatif

| Famille | Commande | Protocoles | Utilisation |
|---------|----------|------------|-------------|
| **ip** | `nft add table ip ...` | IPv4 | Réseau IPv4 pur |
| **ip6** | `nft add table ip6 ...` | IPv6 | Réseau IPv6 pur |
| **inet** | `nft add table inet ...` | IPv4 + IPv6 | **Recommandé** (dual-stack) |
| **arp** | `nft add table arp ...` | ARP | Anti-spoofing, contrôle MAC |
| **bridge** | `nft add table bridge ...` | Bridge | Virtualisation, VLAN |
| **netdev** | `nft add table netdev ...` | Device | DDoS, performance extrême |

---

## Types de chaînes

Les chaînes de base doivent spécifier un **type** qui définit le traitement appliqué aux paquets.

### Les 3 types principaux

| Type | Rôle | Hooks compatibles | Usage |
|------|------|------------------|-------|
| **filter** | Filtrage (accepter/rejeter) | Tous | Pare-feu classique |
| **nat** | Translation d'adresses | prerouting, input, output, postrouting | NAT, redirection |
| **route** | Modification de routage | output | Routage avancé |

### Type filter

Le type **filter** est utilisé pour les décisions d'acceptation/rejet de paquets.

```bash
# Type filter peut être utilisé sur tous les hooks
nft add chain inet filter input '{
    type filter hook input priority 0 ;
}'

nft add chain inet filter forward '{
    type filter hook forward priority 0 ;
}'

nft add chain inet filter output '{
    type filter hook output priority 0 ;
}'
```

#### Actions disponibles avec filter

```bash
# Accept : Accepter le paquet
nft add rule inet filter input tcp dport 22 accept

# Drop : Rejeter silencieusement
nft add rule inet filter input tcp dport 23 drop

# Reject : Rejeter avec notification
nft add rule inet filter input tcp dport 23 reject

# Return : Sortir de la chaîne actuelle
nft add rule inet filter input tcp dport 80 return

# Jump : Sauter vers une autre chaîne
nft add rule inet filter input jump ma_chaine

# Goto : Aller vers une chaîne (sans retour)
nft add rule inet filter input goto ma_chaine
```

> [!tip] Type filter : Le plus courant
> 90% de vos chaînes seront de type **filter** pour implémenter vos règles de pare-feu.

### Type nat

Le type **nat** est utilisé pour la translation d'adresses et de ports.

```bash
# Type nat sur les hooks de NAT
nft add chain inet nat prerouting '{
    type nat hook prerouting priority 0 ;
}'

nft add chain inet nat postrouting '{
    type nat hook postrouting priority 0 ;
}'

# Aussi possible (moins courant)
nft add chain inet nat output '{
    type nat hook output priority 0 ;
}'

nft add chain inet nat input '{
    type nat hook input priority 0 ;
}'
```

#### Actions NAT disponibles

```bash
# SNAT : Changer l'adresse source
nft add rule inet nat postrouting ip saddr 192.168.1.0/24 snat to 203.0.113.5

# Masquerade : SNAT avec IP dynamique
nft add rule inet nat postrouting masquerade

# DNAT : Changer l'adresse destination
nft add rule inet nat prerouting tcp dport 80 dnat to 192.168.1.10

# Redirect : Redirection locale
nft add rule inet nat prerouting tcp dport 80 redirect to 8080
```

> [!warning] Restriction importante
> Le type **nat** ne peut être utilisé QUE sur certains hooks :
> - ✅ prerouting (DNAT)
> - ✅ input (DNAT rare)
> - ✅ output (DNAT local)
> - ✅ postrouting (SNAT)
> - ❌ forward (impossible !)

### Type route

Le type **route** permet de re-router les paquets basé sur des critères.

```bash
# Type route uniquement sur output
nft add chain inet mangle output '{
    type route hook output priority 0 ;
}'

# Changer la marque de routage
nft add rule inet mangle output tcp dport 80 mark set 1

# Le système peut alors utiliser ces marques pour le routage
# (via ip rule et tables de routage)
```

> [!info] Usage avancé
> Le type **route** est pour le routage basé sur des politiques (policy-based routing). C'est un cas d'usage avancé rarement nécessaire.

### Priorités des chaînes

Le paramètre **priority** définit l'ordre d'exécution quand plusieurs chaînes sont sur le même hook.

```bash
# Priorités numériques : -400 à 400
# Plus le nombre est PETIT, plus c'est exécuté TÔT

# Exécutée en premier (priorité -100)
nft add chain inet filter input_early '{
    type filter hook input priority -100 ;
}'

# Exécutée ensuite (priorité 0, standard)
nft add chain inet filter input '{
    type filter hook input priority 0 ;
}'

# Exécutée en dernier (priorité 100)
nft add chain inet filter input_late '{
    type filter hook input priority 100 ;
}'
```

#### Priorités standard recommandées

| Priorité | Nom symbolique | Usage |
|----------|---------------|-------|
| **-400** | raw | Pas de suivi de connexion |
| **-300** | - | Pré-traitement |
| **-225** | conntrack | Suivi de connexion |
| **-200** | mangle | Modification d'en-têtes |
| **-100** | dstnat | DNAT (destination NAT) |
| **0** | filter | Filtrage standard |
| **100** | srcnat | SNAT (source NAT) |
| **200** | - | Post-traitement |

```bash
# Exemple : reproduire l'architecture iptables
nft add chain inet filter input '{
    type filter hook input priority 0 ;
}'

nft add chain inet nat prerouting '{
    type nat hook prerouting priority -100 ;
}'

nft add chain inet nat postrouting '{
    type nat hook postrouting priority 100 ;
}'
```

> [!tip] Recommandation
> Pour débuter, utilisez toujours **priority 0** sauf besoin spécifique.

### Politique par défaut (policy)

Le paramètre **policy** définit l'action par défaut si aucune règle ne matche.

```bash
# Policy accept (permissif)
nft add chain inet filter input '{
    type filter hook input priority 0 ; policy accept ;
}'

# Policy drop (restrictif)
nft add chain inet filter input '{
    type filter hook input priority 0 ; policy drop ;
}'
```

> [!warning] Attention en production
> ```bash
> # ❌ Dangereux : policy drop sans règles
> nft add chain inet filter input '{ type filter hook input priority 0 ; policy drop ; }'
> # Vous venez de vous couper l'accès SSH !
> 
> # ✅ Correct : autoriser d'abord, puis définir policy drop
> nft add chain inet filter input '{ type filter hook input priority 0 ; policy accept ; }'
> nft add rule inet filter input ct state established,related accept
> nft add rule inet filter input tcp dport 22 accept
> nft chain inet filter input '{ policy drop ; }'  # Modifier ensuite
> ```

### Compatibilité hooks et types

| Hook | Type filter | Type nat | Type route |
|------|------------|---------|-----------|
| **prerouting** | ✅ | ✅ | ❌ |
| **input** | ✅ | ✅ | ❌ |
| **forward** | ✅ | ❌ | ❌ |
| **output** | ✅ | ✅ | ✅ |
| **postrouting** | ✅ | ✅ | ❌ |

> [!info] Règle simple
> - **filter** : Tous les hooks ✅
> - **nat** : Tous sauf forward
> - **route** : Uniquement output

---

## 📝 Points clés à retenir

> [!tip] Résumé de la partie
> 
> **Commandes nft** :
> - Syntaxe : `nft COMMANDE OBJET PARAMÈTRES`
> - Commandes principales : add, delete, list, flush
> - Options : -i (interactif), -f (fichier), -n (numérique), -a (handles)
> 
> **Tables et chaînes** :
> - Tables : conteneurs personnalisables (pas de noms imposés)
> - Chaînes de base : attachées à un hook (avec type, priority, policy)
> - Chaînes normales : pour organiser les règles
> 
> **Familles d'adresses** :
> - **ip** : IPv4 seulement
> - **ip6** : IPv6 seulement
> - **inet** : IPv4 + IPv6 (recommandé !)
> - **arp** : Protocole ARP
> - **bridge** : Trafic ponté
> - **netdev** : Très bas niveau
> 
> **Types de chaînes** :
> - **filter** : Filtrage (accept/drop/reject)
> - **nat** : Translation d'adresses
> - **route** : Modification routage
> - Priority : -400 à 400 (0 = standard)
> - Policy : accept ou drop (par défaut)

---

## 🎯 Bonnes pratiques

> [!warning] Pièges à éviter
> - ❌ Ne pas oublier les guillemets autour des accolades : `'{ ... }'`
> - ❌ Ne pas mélanger les familles (une table = une famille)
> - ❌ Ne pas utiliser type nat sur le hook forward
> - ❌ Ne pas définir policy drop sans règles de base (vous perdez l'accès SSH !)
> - ❌ Ne pas hardcoder les handles dans vos scripts

> [!tip] Conseils professionnels
> - ✅ Utilisez la famille **inet** par défaut (dual-stack)
> - ✅ Utilisez des noms de tables explicites (filter, nat, mangle)
> - ✅ Organisez vos règles dans des chaînes normales (modulaire)
> - ✅ Commencez avec priority 0 sauf besoin spécifique
> - ✅ Testez en mode interactif (`nft -i`) avant de scripter
> - ✅ Utilisez `nft -c -f` pour valider vos fichiers de config
> - ✅ Documentez vos choix de structure dans des commentaires

> [!example] Exemple de structure recommandée
> ```bash
> # Table principale avec famille inet (dual-stack)
> nft add table inet filter
> 
> # Chaînes de base avec noms standards
> nft add chain inet filter input '{ type filter hook input priority 0 ; policy drop ; }'
> nft add chain inet filter forward '{ type filter hook forward priority 0 ; policy drop ; }'
> nft add chain inet filter output '{ type filter hook output priority 0 ; policy accept ; }'
> 
> # Chaînes normales pour organisation
> nft add chain inet filter services_essentiels
> nft add chain inet filter services_web
> nft add chain inet filter admin_access
> 
> # Structure claire, maintenable, évolutive
> ```