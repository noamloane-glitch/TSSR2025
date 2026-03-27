# 

## 📋 Table des matières

- [Définition et rôle de Netfilter](#définition-et-rôle-de-netfilter)
- [Historique et évolution](#historique-et-évolution)
- [Architecture générale](#architecture-générale)
- [Relation avec iptables](#relation-avec-iptables)

---

## Définition et rôle de Netfilter

### Qu'est-ce que Netfilter ?

**Netfilter** est un framework intégré au noyau Linux qui permet la manipulation, le filtrage et la transformation des paquets réseau. C'est un composant fondamental pour la sécurité et la gestion du trafic réseau sous Linux.

> [!info] Netfilter dans le noyau
> Netfilter n'est pas un programme externe mais une partie intégrante du noyau Linux. Il fonctionne au niveau 3 (couche réseau) et 4 (couche transport) du modèle OSI.

### Rôle principal

Netfilter joue plusieurs rôles critiques dans le système :

| Fonction | Description |
|----------|-------------|
| **Filtrage de paquets** | Autoriser ou bloquer le trafic réseau selon des règles définies |
| **Translation d'adresses (NAT)** | Modifier les adresses IP source ou destination des paquets |
| **Modification de paquets** | Altérer les en-têtes ou le contenu des paquets |
| **Suivi de connexion** | Maintenir l'état des connexions réseau (stateful firewall) |
| **Journalisation** | Enregistrer les événements réseau pour analyse |

### Pourquoi Netfilter est important ?

```bash
# Sans Netfilter, un système Linux serait :
# ❌ Exposé à toutes les connexions entrantes
# ❌ Incapable de partager une connexion Internet (pas de NAT)
# ❌ Sans contrôle sur le trafic réseau
# ❌ Vulnérable aux attaques réseau

# Avec Netfilter :
# ✅ Contrôle total du trafic entrant/sortant
# ✅ Protection contre les intrusions
# ✅ Partage de connexion possible
# ✅ Gestion avancée du routage
```

> [!tip] Cas d'usage courants
> - **Serveur web** : Autoriser uniquement HTTP/HTTPS et SSH
> - **Passerelle réseau** : Partager une connexion Internet via NAT
> - **DMZ** : Isoler les serveurs publics du réseau interne
> - **VPN** : Filtrer et router le trafic chiffré

---

## Historique et évolution

### Chronologie de Netfilter

**1998 - ipfwadm** 📅
- Premier système de filtrage Linux (noyau 2.0)
- Fonctionnalités limitées
- Interface complexe

**1999 - ipchains** 📅
- Amélioration pour le noyau 2.2
- Introduction du concept de chaînes
- Toujours limité en termes de fonctionnalités

**2000 - Netfilter/iptables** 🎯
- Réécriture complète par Rusty Russell et l'équipe Netfilter
- Intégré au noyau Linux 2.4
- Architecture modulaire et extensible
- Support du suivi de connexion (conntrack)
- Devient le standard de facto

**2014 - nftables** 🆕
- Successeur moderne de iptables
- Syntaxe simplifiée et unifiée
- Meilleures performances
- Toujours basé sur le framework Netfilter

> [!info] Compatibilité
> Netfilter reste le framework sous-jacent pour iptables ET nftables. Le passage de l'un à l'autre est une question d'outil d'administration, pas d'architecture noyau.

### Pourquoi cette évolution ?

```bash
# ipfwadm (1998)
# - Fonctionnalités basiques
# - Peu flexible

# ipchains (1999)
# - Chaînes de règles
# - Toujours des limitations

# iptables (2000)
# - Tables multiples (filter, nat, mangle)
# - Suivi de connexion
# - Architecture extensible
# ⭐ Devient le standard pendant 15 ans

# nftables (2014+)
# - Syntaxe modernisée
# - Un seul outil pour tout (pas de séparation iptables/ip6tables/arptables)
# - Meilleures performances
```

> [!warning] Attention
> Dans une formation TSSR, vous travaillerez principalement avec **iptables** car il reste très largement utilisé en production. nftables est l'avenir mais la transition est progressive.

---

## Architecture générale

### Vue d'ensemble du framework

Netfilter est organisé selon une architecture en couches bien définie :

```
┌─────────────────────────────────────────────────┐
│          ESPACE UTILISATEUR (userspace)         │
│                                                 │
│  iptables │ nftables │ autres outils            │
└────────────────────┬────────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │    Bibliothèques      │
         │   (libiptc, libnftnl) │
         └───────────┬───────────┘
                     │
═════════════════════╪═══════════════════════════════
                     │
         ┌───────────▼───────────┐
         │    NOYAU LINUX        │
         │                       │
         │   ┌───────────────┐   │
         │   │   Netfilter   │   │
         │   │   Framework   │   │
         │   └───────┬───────┘   │
         │           │           │
         │   ┌───────▼───────┐   │
         │   │     Hooks     │   │
         │   └───────┬───────┘   │
         │           │           │
         │   ┌───────▼───────┐   │
         │   │  Modules      │   │
         │   │  (xt_state,   │   │
         │   │   nf_nat...)  │   │
         │   └───────────────┘   │
         └───────────────────────┘
```

### Les composants clés

> [!example] Composants de l'architecture
> 
> **1. Hooks (points d'accroche)**
> - Points d'interception dans le parcours des paquets
> - 5 hooks principaux : PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING
> 
> **2. Tables**
> - Conteneurs organisés par fonction
> - filter (filtrage), nat (translation), mangle (modification), raw (suivi)
> 
> **3. Chaînes**
> - Listes de règles attachées aux hooks
> - Chaque table contient des chaînes spécifiques
> 
> **4. Règles**
> - Instructions de traitement des paquets
> - Critères de correspondance + action (cible)
> 
> **5. Modules**
> - Extensions chargeables
> - Ajoutent des fonctionnalités (ex: suivi de connexion, correspondance de chaînes)

### Flux de traitement d'un paquet

```bash
# Paquet ENTRANT (vers le système local)
Internet → [PREROUTING] → Routage → [INPUT] → Application

# Paquet TRANSITÉ (routé à travers le système)
Internet → [PREROUTING] → Routage → [FORWARD] → [POSTROUTING] → Internet

# Paquet SORTANT (depuis le système local)
Application → [OUTPUT] → Routage → [POSTROUTING] → Internet
```

> [!info] Point clé
> À chaque hook, Netfilter peut appliquer les règles de plusieurs tables dans un ordre spécifique. C'est ce qu'on appelle l'ordre de priorité des tables.

### Ordre de traitement des tables

Lorsqu'un paquet traverse un hook, les tables sont consultées dans cet ordre :

1. **raw** - Décisions sur le suivi de connexion
2. **mangle** - Modifications des en-têtes
3. **nat** - Translation d'adresses (DNAT pour paquets entrants)
4. **filter** - Décisions de filtrage (accepter/rejeter)
5. **nat** - Translation d'adresses (SNAT pour paquets sortants)

> [!tip] Astuce mémorisation
> Pensez à "**R**aw **M**angle **N**at **F**ilter" pour l'ordre principal. Le NAT apparaît à deux moments différents selon le type de translation.

---

## Relation avec iptables

### Netfilter vs iptables : Clarification

```bash
# NETFILTER (dans le noyau)
# - Framework de manipulation de paquets
# - Code qui s'exécute au niveau du noyau
# - Gère réellement le trafic réseau
# - Ne peut pas être directement manipulé par l'utilisateur

# IPTABLES (espace utilisateur)
# - Outil en ligne de commande
# - Interface d'administration
# - Permet de configurer les règles Netfilter
# - Programme que vous utilisez : /sbin/iptables
```

> [!info] Analogie
> **Netfilter** est comme le moteur d'une voiture (invisible, fait le travail).  
> **iptables** est comme le volant et les pédales (interface pour contrôler le moteur).

### Comment iptables communique avec Netfilter

```
┌──────────────────────────────────────────┐
│  Administrateur tape une commande        │
│  $ iptables -A INPUT -p tcp --dport 22   │
│            -j ACCEPT                     │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  iptables (programme userspace)          │
│  - Parse la commande                     │
│  - Valide la syntaxe                     │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  libiptc (bibliothèque)                  │
│  - Convertit en format compréhensible    │
│    par le noyau                          │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  Appel système (ioctl/netlink)           │
│  - Communication avec le noyau           │
└──────────────┬───────────────────────────┘
               │
═══════════════╪═══════════════════════════
               │  ESPACE NOYAU
               ▼
┌──────────────────────────────────────────┐
│  Netfilter                               │
│  - Enregistre la règle                   │
│  - Applique au trafic réseau             │
└──────────────────────────────────────────┘
```

### Variantes d'iptables

Netfilter peut être administré par différents outils selon les besoins :

| Outil | Description | Usage |
|-------|-------------|-------|
| **iptables** | Gestion des règles IPv4 | Standard pour le trafic IPv4 |
| **ip6tables** | Gestion des règles IPv6 | Spécifique au trafic IPv6 |
| **arptables** | Gestion du protocole ARP | Filtrage au niveau liaison |
| **ebtables** | Gestion bridging Ethernet | Filtrage pour les ponts réseau |

> [!warning] Piège courant
> Beaucoup d'administrateurs oublient de configurer **ip6tables** et laissent ainsi leur système vulnérable via IPv6, même si IPv4 est correctement filtré !

### iptables : Syntaxe de base

```bash
# Structure générale d'une commande iptables
iptables [-t table] COMMANDE CHAINE [critères] -j ACTION

# Exemple concret
iptables -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
#        └─ table  └─ cmd  └─ chain  └─ critères  └─ action

# Détails :
# -t filter  : Spécifie la table (filter par défaut si omis)
# -A INPUT   : Ajoute (Append) la règle à la chaîne INPUT
# -p tcp     : Protocole TCP
# --dport 80 : Port de destination 80
# -j ACCEPT  : Action = accepter le paquet
```

> [!example] Commandes les plus courantes
> ```bash
> # Lister les règles
> iptables -L -n -v
> 
> # Ajouter une règle
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> 
> # Supprimer une règle
> iptables -D INPUT 1
> 
> # Vider toutes les règles
> iptables -F
> 
> # Définir la politique par défaut
> iptables -P INPUT DROP
> ```

### L'écosystème autour de Netfilter

Au-delà d'iptables, plusieurs outils interagissent avec Netfilter :

**Outils de gestion** :
- `iptables-save` : Exporte les règles actuelles
- `iptables-restore` : Importe des règles depuis un fichier
- `iptables-persistent` : Service pour rendre les règles persistantes au démarrage

**Outils d'analyse** :
- `conntrack` : Examine la table de suivi de connexion
- `nflog` : Journalisation avancée des paquets
- `tcpdump` : Capture de paquets (utilise indirectement Netfilter)

**Interfaces graphiques** :
- `ufw` : Uncomplicated Firewall (Ubuntu)
- `firewalld` : Gestion dynamique du pare-feu (RHEL/CentOS)
- `fwbuilder` : Constructeur graphique de règles

> [!tip] Pour les débutants
> Si iptables vous semble complexe, vous pouvez commencer avec `ufw` qui utilise Netfilter/iptables en arrière-plan avec une syntaxe simplifiée :
> ```bash
> ufw allow 22/tcp
> ufw enable
> ```
> Mais dans un contexte professionnel TSSR, la maîtrise d'iptables reste indispensable !

### Pourquoi apprendre iptables en 2025 ?

Bien que nftables soit la technologie "officielle" depuis plusieurs années, iptables reste omniprésent :

✅ **Présent dans la majorité des distributions en production**  
✅ **Documentation abondante et communauté large**  
✅ **Scripts et configurations existantes basées dessus**  
✅ **Compétence recherchée par les employeurs**  
✅ **Base solide pour comprendre nftables ensuite**

> [!info] Transition progressive
> La plupart des distributions Linux modernes incluent `iptables-nft`, une version d'iptables qui utilise nftables comme backend tout en conservant la syntaxe iptables. Cela facilite la migration progressive.

---

## 📝 Points clés à retenir

> [!tip] Résumé de la partie
> 
> **Netfilter** :
> - Framework dans le noyau Linux pour la manipulation de paquets
> - Intégré depuis le noyau 2.4 (2000)
> - Fonctionne via un système de hooks, tables, chaînes et règles
> 
> **iptables** :
> - Outil en ligne de commande pour administrer Netfilter
> - Interface entre l'administrateur et le framework noyau
> - Syntaxe : `iptables [-t table] COMMANDE CHAINE [critères] -j ACTION`
> 
> **Architecture** :
> - 5 hooks : PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING
> - 4 tables principales : raw, mangle, nat, filter
> - Traitement des paquets selon un ordre précis
> 
> **Évolution** :
> - ipfwadm → ipchains → iptables → nftables
> - Netfilter reste la base commune, seuls les outils changent

---

## 🎯 Bonnes pratiques

> [!warning] Pièges à éviter
> - ❌ Ne jamais tester des règles de pare-feu sans accès de secours (console physique ou IPMI)
> - ❌ Ne pas oublier de configurer ip6tables en plus d'iptables
> - ❌ Ne pas confondre Netfilter (noyau) et iptables (outil)
> - ❌ Ne pas appliquer de règles sans les tester d'abord sur un environnement de test

> [!tip] Conseils professionnels
> - ✅ Toujours commenter vos règles pour faciliter la maintenance
> - ✅ Documenter les changements effectués
> - ✅ Sauvegarder les règles avant toute modification
> - ✅ Utiliser des scripts pour déployer des configurations cohérentes
> - ✅ Tester progressivement en mode "log" avant de bloquer réellement