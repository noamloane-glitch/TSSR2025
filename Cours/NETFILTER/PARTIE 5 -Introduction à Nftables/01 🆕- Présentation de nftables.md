## 📋 Table des matières

- [Qu'est-ce que nftables](#quest-ce-que-nftables)
- [Historique et motivation](#historique-et-motivation)
- [Différences architecturales avec iptables](#différences-architecturales-avec-iptables)
- [Avantages et inconvénients](#avantages-et-inconvénients)

---

## Qu'est-ce que nftables

### Définition

**nftables** est le successeur moderne d'iptables pour le filtrage de paquets et la gestion du pare-feu sous Linux. Il s'agit d'un nouveau framework de classification de paquets qui utilise toujours l'infrastructure **Netfilter** du noyau, mais avec une interface utilisateur complètement repensée.

> [!info] Point clé
> nftables n'est PAS un remplacement de Netfilter, mais un remplacement d'**iptables**. Le framework Netfilter dans le noyau reste identique - seul l'outil d'administration change.

```bash
# Architecture simplifiée
┌─────────────────────────────────────┐
│     ESPACE UTILISATEUR              │
│                                     │
│  iptables    │    nftables          │
│  ip6tables   │    (nft)             │
│  arptables   │                      │
│  ebtables    │                      │
└───────┬──────────────┬──────────────┘
        │              │
═══════╪══════════════╪═════════════════
        │              │
        └──────┬───────┘
               │
    ┌──────────▼──────────┐
    │   NOYAU LINUX       │
    │                     │
    │   Netfilter         │
    │   Framework         │
    └─────────────────────┘
```

### Outil en ligne de commande : nft

L'outil principal pour administrer nftables est la commande **nft** :

```bash
# Syntaxe générale
nft [options] commande

# Exemples basiques
nft list ruleset                    # Afficher toutes les règles
nft add table inet filter          # Créer une table
nft add chain inet filter input    # Créer une chaîne
nft flush ruleset                  # Vider toutes les règles
```

> [!tip] Première différence notable
> Avec iptables, vous utilisiez : `iptables`, `ip6tables`, `arptables`, `ebtables`  
> Avec nftables, vous utilisez : **uniquement `nft`** pour tout !

### Installation et disponibilité

```bash
# Vérifier si nftables est installé
nft --version

# Installation sur Debian/Ubuntu
sudo apt install nftables

# Installation sur RHEL/CentOS/Rocky
sudo dnf install nftables

# Activer le service
sudo systemctl enable nftables
sudo systemctl start nftables
```

> [!warning] Compatibilité
> nftables est disponible depuis le noyau Linux **3.13** (2014), mais est pleinement fonctionnel à partir du noyau **4.14** (2017). La plupart des distributions modernes l'incluent par défaut depuis 2019-2020.

### Composants principaux

| Composant | Description | Équivalent iptables |
|-----------|-------------|-------------------|
| **nft** | Outil CLI principal | iptables, ip6tables, etc. |
| **libnftables** | Bibliothèque C | libiptc |
| **nf_tables** | Module noyau | modules iptables (xt_*, ipt_*) |

---

## Historique et motivation

### Chronologie du développement

**2008** 📅
- Patrick McHardy commence le développement de nftables
- Objectif : Résoudre les limitations d'iptables

**2013** 📅
- Première inclusion dans le noyau Linux 3.13
- Version initiale, encore immature

**2014** 📅
- nftables intégré officiellement au projet Netfilter
- Développement actif par la communauté

**2018** 📅
- Debian 10 "Buster" adopte nftables par défaut
- Maturité suffisante pour la production

**2019-2020** 📅
- La plupart des distributions Linux passent à nftables par défaut
- RHEL 8, Ubuntu 20.04, etc.

**2021-2025** 📅
- nftables devient le standard recommandé
- iptables reste disponible mais en mode "legacy" ou via compatibilité

> [!info] État actuel (2025)
> - **nftables** est le standard officiel et recommandé
> - **iptables** reste largement utilisé en production
> - De nombreux systèmes utilisent **iptables-nft** : syntaxe iptables avec backend nftables

### Pourquoi créer nftables ?

Les développeurs ont identifié plusieurs **limitations d'iptables** :

#### 1. Multiplicité des outils

```bash
# Avec iptables : 4 outils différents
iptables   # IPv4
ip6tables  # IPv6
arptables  # ARP
ebtables   # Ethernet bridging

# Avec nftables : un seul outil
nft        # Gère tout !
```

#### 2. Architecture rigide

```bash
# iptables : tables et chaînes prédéfinies
# Vous DEVEZ utiliser : filter, nat, mangle, raw
# Vous DEVEZ utiliser : INPUT, OUTPUT, FORWARD, etc.

# nftables : tables et chaînes totalement personnalisables
# Vous créez ce dont vous avez besoin, comme vous voulez
```

#### 3. Performance et mises à jour

> [!example] Problème iptables
> ```bash
> # Avec iptables, chaque modification nécessite de :
> # 1. Télécharger TOUTES les règles du noyau
> # 2. Modifier les règles en mémoire
> # 3. Renvoyer TOUTES les règles au noyau
> 
> # Pour 1000 règles, modifier une seule règle = 
> # transférer 1000 règles deux fois !
> ```

> [!example] Solution nftables
> ```bash
> # Avec nftables :
> # Modifications atomiques et incrémentielles
> # Modifier une règle = modifier UNE règle
> # Bien plus efficace !
> ```

#### 4. Complexité du code

| Aspect | iptables | nftables |
|--------|----------|----------|
| **Lignes de code noyau** | ~30 000 | ~6 000 |
| **Modules noyau** | Des dizaines | Un seul |
| **Maintenabilité** | Difficile | Simplifiée |

#### 5. Manque de fonctionnalités avancées

```bash
# iptables : pas de variables, pas de structures de données

# nftables : sets, maps, dictionnaires
nft add set inet filter blacklist { type ipv4_addr\; }
nft add element inet filter blacklist { 192.168.1.100, 10.0.0.50 }
```

### Objectifs de nftables

Les développeurs visaient à créer un outil :

✅ **Unifié** : Un seul outil pour IPv4, IPv6, ARP, bridge  
✅ **Performant** : Mises à jour atomiques, moins de surcharge  
✅ **Flexible** : Tables et chaînes personnalisables  
✅ **Simple** : Syntaxe cohérente et lisible  
✅ **Moderne** : Support des structures de données avancées  
✅ **Maintenable** : Code noyau simplifié

> [!tip] Citation officielle
> "nftables aims to replace the existing {ip,ip6,arp,eb}tables framework" - Projet Netfilter

---

## Différences architecturales avec iptables

### 1. Structure des tables et chaînes

#### Avec iptables : Structure rigide

```bash
# Tables PRÉDÉFINIES (vous ne pouvez pas en créer d'autres)
filter, nat, mangle, raw, security

# Chaînes PRÉDÉFINIES (obligatoires dans chaque table)
INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING

# Exemple : vous DEVEZ utiliser ces noms
iptables -t filter -A INPUT -j ACCEPT
```

#### Avec nftables : Structure libre

```bash
# Tables : VOUS les créez comme vous voulez
nft add table inet ma_table
nft add table ip pare_feu_ipv4
nft add table ip6 filtrage_v6

# Chaînes : VOUS les nommez comme vous voulez
nft add chain inet ma_table entrees
nft add chain inet ma_table sorties
nft add chain inet ma_table mon_filtrage_personnel

# Exemple : totale liberté
nft add chain inet pare_feu acces_web
```

> [!info] Flexibilité
> Dans nftables, **vous définissez votre propre architecture**. Pas de contraintes prédéfinies, sauf si vous le souhaitez explicitement.

### 2. Familles d'adresses

#### Avec iptables : Outils séparés

```bash
# IPv4
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# IPv6  
ip6tables -A INPUT -s 2001:db8::/32 -j ACCEPT

# ARP
arptables -A INPUT --source-mac aa:bb:cc:dd:ee:ff -j ACCEPT

# Bridge
ebtables -A INPUT -p IPv4 -j ACCEPT
```

#### Avec nftables : Une commande unifiée

```bash
# Famille ip (IPv4 uniquement)
nft add rule ip filter input ip saddr 192.168.1.0/24 accept

# Famille ip6 (IPv6 uniquement)
nft add rule ip6 filter input ip6 saddr 2001:db8::/32 accept

# Famille inet (IPv4 ET IPv6 ensemble !)
nft add rule inet filter input ip saddr 192.168.1.0/24 accept
nft add rule inet filter input ip6 saddr 2001:db8::/32 accept

# Famille arp
nft add rule arp filter input arp saddr ip 192.168.1.1 accept

# Famille bridge
nft add rule bridge filter input ether type ip accept
```

| Famille | Description | Équivalent iptables |
|---------|-------------|-------------------|
| **ip** | IPv4 uniquement | iptables |
| **ip6** | IPv6 uniquement | ip6tables |
| **inet** | IPv4 + IPv6 (dual-stack) | iptables + ip6tables |
| **arp** | Protocole ARP | arptables |
| **bridge** | Bridging Ethernet | ebtables |
| **netdev** | Ingress (très bas niveau) | - (nouveau) |

> [!tip] Famille inet : La killer feature
> ```bash
> # Au lieu de dupliquer les règles :
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> ip6tables -A INPUT -p tcp --dport 22 -j ACCEPT
> 
> # Une seule règle nftables :
> nft add rule inet filter input tcp dport 22 accept
> # Fonctionne pour IPv4 ET IPv6 !
> ```

### 3. Syntaxe des règles

#### Avec iptables : Syntaxe à options

```bash
# Structure : commande -options --long-options valeur
iptables -t filter -A INPUT -p tcp -s 192.168.1.0/24 --dport 80 -j ACCEPT
#        └─table └─cmd └─chain └─proto └─source      └─port    └─action

# Beaucoup de tirets, longues options
```

#### Avec nftables : Syntaxe naturelle

```bash
# Structure : nft commande famille table chaîne expression action
nft add rule inet filter input ip saddr 192.168.1.0/24 tcp dport 80 accept
#   └─cmd    └─famille └─tab └─chain └─source           └─port    └─action

# Plus lisible, plus proche du langage naturel
```

> [!example] Comparaison côte à côte
> ```bash
> # iptables : syntaxe technique
> iptables -A INPUT -p tcp -m multiport --dports 80,443 -m state --state NEW -j ACCEPT
> 
> # nftables : syntaxe descriptive
> nft add rule inet filter input tcp dport { 80, 443 } ct state new accept
> ```

### 4. Gestion des ensembles (sets)

#### Avec iptables : Utilisation du module ipset (séparé)

```bash
# Créer un set avec ipset (outil séparé)
ipset create blacklist hash:ip

# Ajouter des adresses
ipset add blacklist 192.168.1.100
ipset add blacklist 10.0.0.50

# Utiliser dans iptables
iptables -A INPUT -m set --match-set blacklist src -j DROP

# Problème : deux outils différents, syntaxes différentes
```

#### Avec nftables : Sets intégrés

```bash
# Tout dans nft, syntaxe cohérente
nft add set inet filter blacklist { type ipv4_addr\; }

# Ajouter des adresses
nft add element inet filter blacklist { 192.168.1.100, 10.0.0.50 }

# Utiliser dans les règles
nft add rule inet filter input ip saddr @blacklist drop

# Un seul outil, une seule syntaxe
```

> [!info] Sets anonymes
> nftables permet aussi des sets "inline" :
> ```bash
> # Set anonyme directement dans la règle
> nft add rule inet filter input tcp dport { 22, 80, 443 } accept
> ```

### 5. Modifications atomiques

#### Avec iptables : Modifications non-atomiques

```bash
# Problème : fenêtre de vulnérabilité lors des modifications

# Étape 1 : Flush les règles
iptables -F
# → Pendant un instant, AUCUNE protection !

# Étape 2 : Recharger les nouvelles règles
iptables-restore < /etc/iptables/rules.v4
# → Fenêtre de risque entre flush et restore
```

#### Avec nftables : Modifications atomiques

```bash
# Solution : remplacements atomiques

# Charger un nouveau ruleset complet
nft -f /etc/nftables.conf
# → Remplacement atomique, pas de fenêtre de vulnérabilité

# Ou utiliser des transactions
nft -i
nft> flush ruleset
nft> include "/etc/nftables.conf"
nft> commit
# → Tout est appliqué en une seule opération atomique
```

> [!tip] Avantage critique
> En production, les modifications atomiques éliminent les fenêtres de vulnérabilité lors des mises à jour du pare-feu.

### 6. Performance du moteur de règles

#### Architecture du matching

```bash
# iptables : Matching linéaire
Règle 1 → test → non → Règle 2 → test → non → ... → Règle 1000 → match !
# Pour 1000 règles, peut nécessiter 1000 tests

# nftables : Structures optimisées (maps, sets avec hash/rbtree)
Règle avec @set → lookup hash → O(1) ou O(log n)
# Beaucoup plus rapide pour de grands ensembles
```

| Aspect | iptables | nftables |
|--------|----------|----------|
| **Algorithme matching** | Linéaire | Hash/RBTree optimisé |
| **1000 règles simples** | ~1000 tests | ~1000 tests |
| **1000 IPs dans un set** | ~1000 tests | ~1 lookup |
| **Mise à jour ruleset** | Reload complet | Incrémentiel |

### 7. Compteurs et statistiques

#### Avec iptables : Compteurs par défaut

```bash
# Tous les compteurs sont toujours actifs
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Voir les compteurs
iptables -L -v -n
# Consomme de la mémoire et des ressources même si inutilisé
```

#### Avec nftables : Compteurs optionnels

```bash
# Sans compteur (plus léger)
nft add rule inet filter input tcp dport 80 accept

# Avec compteur (explicite)
nft add rule inet filter input tcp dport 80 counter accept

# Vous choisissez quand compter
```

> [!tip] Performance
> Sur un système à très fort trafic, désactiver les compteurs inutiles peut améliorer les performances.

---

## Avantages et inconvénients

### Avantages de nftables

#### ✅ 1. Unification des outils

```bash
# Avant (iptables)
iptables -A INPUT ...
ip6tables -A INPUT ...
arptables -A INPUT ...
ebtables -A INPUT ...

# Maintenant (nftables)
nft add rule inet filter input ...
# Un seul outil, une seule syntaxe !
```

#### ✅ 2. Syntaxe plus claire et cohérente

```bash
# iptables : difficile à lire
iptables -t nat -A PREROUTING -p tcp -d 203.0.113.5 --dport 80 -j DNAT --to-destination 192.168.1.10:8080

# nftables : plus lisible
nft add rule inet nat prerouting ip daddr 203.0.113.5 tcp dport 80 dnat to 192.168.1.10:8080
```

#### ✅ 3. Structures de données avancées

```bash
# Sets nommés
nft add set inet filter blacklist { type ipv4_addr\; flags interval\; }

# Maps (dictionnaires)
nft add map inet nat portmap { type inet_service : ipv4_addr\; }
nft add element inet nat portmap { 80 : 192.168.1.10, 443 : 192.168.1.11 }

# Utilisation
nft add rule inet nat prerouting dnat to tcp dport map @portmap
```

#### ✅ 4. Meilleures performances

| Scénario | iptables | nftables | Gain |
|----------|----------|----------|------|
| Lookup 10k IPs | Linéaire | Hash O(1) | ~100x |
| Modification règle | Reload full | Incrémentiel | ~10x |
| Mémoire utilisée | Compteurs forcés | Optionnels | ~20% |

#### ✅ 5. Gestion par fichier de configuration

```bash
# nftables : configuration déclarative
cat /etc/nftables.conf
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state established,related accept
        tcp dport 22 accept
    }
}

# Rechargement simple
nft -f /etc/nftables.conf
```

#### ✅ 6. Architecture flexible

```bash
# Vous créez votre propre structure
nft add table inet pare_feu_entreprise
nft add chain inet pare_feu_entreprise regles_dmz
nft add chain inet pare_feu_entreprise regles_admin
nft add chain inet pare_feu_entreprise regles_visiteurs

# Pas de contraintes prédéfinies
```

#### ✅ 7. Modifications atomiques

```bash
# Remplacement complet sans fenêtre de vulnérabilité
nft -f /etc/nftables.conf

# Ou transactions
nft -i
nft> begin transaction
nft> ...modifications...
nft> commit
```

#### ✅ 8. Famille inet (dual-stack)

```bash
# Une règle pour IPv4 ET IPv6
nft add rule inet filter input tcp dport { 22, 80, 443 } accept

# Équivalent iptables (deux règles) :
iptables -A INPUT -p tcp -m multiport --dports 22,80,443 -j ACCEPT
ip6tables -A INPUT -p tcp -m multiport --dports 22,80,443 -j ACCEPT
```

---

### Inconvénients de nftables

#### ❌ 1. Courbe d'apprentissage

> [!warning] Nouvelle syntaxe
> Si vous maîtrisez iptables, vous devez réapprendre une syntaxe différente.
> ```bash
> # Muscle memory iptables
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> 
> # Nouvelle syntaxe nftables
> nft add rule inet filter input tcp dport 22 accept
> ```

#### ❌ 2. Documentation et exemples moins abondants

| Ressource | iptables | nftables |
|-----------|----------|----------|
| **Tutoriels en ligne** | Des milliers | Quelques centaines |
| **Livres publiés** | Nombreux | Rares |
| **Stack Overflow** | ~15k questions | ~500 questions |
| **Scripts existants** | Millions | Milliers |

> [!info] Situation qui s'améliore
> La documentation nftables s'étoffe progressivement, mais iptables reste plus documenté pour l'instant (2025).

#### ❌ 3. Compatibilité avec anciens systèmes

```bash
# nftables nécessite :
# - Noyau Linux >= 3.13 (idéalement >= 4.14)
# - Distributions récentes

# Problème : systèmes legacy
# - RHEL 6 / CentOS 6 : pas de nftables
# - Debian 8 et antérieurs : pas de nftables
# - Ubuntu 16.04 et antérieurs : support limité
```

#### ❌ 4. Scripts et outils existants basés sur iptables

```bash
# Problème : nombreux outils utilisent iptables

# Docker
docker run ... → utilise iptables pour le NAT

# Kubernetes
kube-proxy → utilise iptables par défaut

# Fail2ban
Configuration par défaut → iptables

# UFW, firewalld
Historiquement basés sur iptables

# Migration nécessaire pour tout l'écosystème
```

> [!warning] Cohabitation délicate
> iptables et nftables peuvent cohabiter, mais :
> - Risque de conflits
> - Gestion complexe
> - Recommandé : choisir l'un OU l'autre

#### ❌ 5. Manque de support dans certains environnements

```bash
# Environnements qui peuvent poser problème :

# Conteneurs légers
# - Certains ne incluent pas nft par défaut
# - Alpine Linux : nécessite installation manuelle

# Systèmes embarqués
# - Mémoire limitée
# - Parfois iptables préféré pour sa légèreté

# Pare-feu matériels avec Linux embarqué
# - Firmwares pas toujours à jour
```

#### ❌ 6. Transition en cours dans l'industrie

> [!info] État actuel (2025)
> - **Distributions** : nftables par défaut
> - **Production** : iptables encore dominant
> - **Documentation entreprise** : souvent basée sur iptables
> - **Compétences TSSR** : iptables plus demandé actuellement

---

### Tableau comparatif global

| Critère | iptables | nftables | Gagnant |
|---------|----------|----------|---------|
| **Unification IPv4/v6** | ❌ Outils séparés | ✅ Un seul outil | nftables |
| **Syntaxe** | ⚠️ Options complexes | ✅ Naturelle | nftables |
| **Performance** | ⚠️ Linéaire | ✅ Optimisée | nftables |
| **Structures données** | ❌ Limitées | ✅ Avancées | nftables |
| **Flexibilité** | ❌ Structure rigide | ✅ Architecture libre | nftables |
| **Atomicité** | ❌ Non-atomique | ✅ Atomique | nftables |
| **Documentation** | ✅ Abondante | ⚠️ Croissante | iptables |
| **Adoption production** | ✅ Massive | ⚠️ Progressive | iptables |
| **Compatibilité** | ✅ Partout | ⚠️ Noyau >= 3.13 | iptables |
| **Courbe apprentissage** | ⚠️ Complexe | ⚠️ Différente | Égalité |
| **Écosystème outils** | ✅ Mature | ⚠️ En développement | iptables |
| **Futur** | ⚠️ Legacy | ✅ Standard | nftables |

---

## 📝 Points clés à retenir

> [!tip] Résumé de la partie
> 
> **nftables** :
> - Successeur moderne d'iptables (pas de Netfilter)
> - Disponible depuis 2014, mature depuis 2019
> - Un seul outil `nft` pour tout (IPv4/v6/ARP/bridge)
> - Standard par défaut sur distributions récentes
> 
> **Motivation** :
> - Simplifier et unifier les outils
> - Améliorer les performances
> - Flexibiliser l'architecture
> - Moderniser la syntaxe
> 
> **Différences majeures** :
> - Syntaxe naturelle vs options techniques
> - Tables/chaînes personnalisables vs prédéfinies
> - Famille `inet` pour dual-stack IPv4/v6
> - Sets/maps intégrés vs ipset séparé
> - Modifications atomiques vs recharges complètes
> 
> **Avantages nftables** :
> - ✅ Plus performant, plus flexible, plus moderne
> - ✅ Syntaxe cohérente, structures avancées
> - ✅ Architecture libre, modifications atomiques
> 
> **Inconvénients nftables** :
> - ❌ Courbe d'apprentissage, moins documenté
> - ❌ Adoption progressive, compatibilité limitée
> - ❌ Écosystème en transition

---

## 🎯 Bonnes pratiques

> [!warning] Pièges à éviter
> - ❌ Ne pas mélanger iptables et nftables sur le même système (risques de conflits)
> - ❌ Ne pas supposer que la syntaxe iptables fonctionne en nftables
> - ❌ Ne pas négliger la formation : nftables n'est PAS juste une mise à jour d'iptables
> - ❌ Ne pas migrer en production sans tests approfondis

> [!tip] Conseils professionnels
> - ✅ Apprenez **d'abord iptables** (encore dominant en production, base conceptuelle)
> - ✅ Puis apprenez **nftables** (l'avenir, compétence différenciante)
> - ✅ Pour nouveaux déploiements : privilégiez nftables
> - ✅ Pour systèmes existants : restez sur iptables sauf besoin spécifique
> - ✅ Utilisez `iptables-translate` pour la migration
> - ✅ Testez en environnement de dev avant la production
> - ✅ Documentez votre choix (iptables vs nftables) dans votre infrastructure

> [!info] Recommandation TSSR
> **Formation** : Maîtriser iptables reste essentiel (90% des systèmes production actuels)  
> **Veille** : Suivre nftables pour être prêt à la transition  
> **Certification** : Les deux sont valorisés sur le marché du travail