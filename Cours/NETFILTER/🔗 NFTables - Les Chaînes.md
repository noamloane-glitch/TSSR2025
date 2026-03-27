
## 📋 Table des matières

- [Introduction](#introduction)
- [Chaînes de base (Base Chains)](#chaînes-de-base-base-chains)
- [Chaînes utilisateur (Regular Chains)](#chaînes-utilisateur-regular-chains)
- [Types de chaînes](#types-de-chaînes)
- [Hooks disponibles](#hooks-disponibles)
- [Priorités des chaînes](#priorités-des-chaînes)

---

## Introduction

Les chaînes (chains) dans NFTables sont des conteneurs logiques qui regroupent des règles de filtrage. Elles constituent l'élément central de l'organisation du pare-feu et déterminent **où** et **quand** les paquets seront inspectés.

> [!info] Analogie
> Pensez aux chaînes comme à des **postes de contrôle** sur une autoroute : chaque poste (chaîne) inspecte les véhicules (paquets) à un moment précis de leur trajet, et applique des règles spécifiques selon leur destination.

---

## Chaînes de base (Base Chains)

### Définition et rôle

Les **chaînes de base** sont les chaînes qui sont directement **rattachées à un hook du noyau**. Ce sont les points d'entrée du système de filtrage : tout paquet qui traverse le système passe obligatoirement par au moins une chaîne de base.

> [!warning] Point critique
> Sans chaîne de base, vos règles NFTables ne seront jamais évaluées ! Les chaînes de base sont **indispensables** pour que le pare-feu fonctionne.

### Caractéristiques essentielles

Une chaîne de base possède **trois attributs obligatoires** :

1. **Type** : détermine la nature du traitement (filter, nat, route)
2. **Hook** : définit le point d'accrochage dans le parcours du paquet
3. **Priority** : détermine l'ordre d'exécution si plusieurs chaînes utilisent le même hook

### Syntaxe de création

```bash
# Structure générale
nft add chain [famille] [table] [nom_chaîne] { \
    type [type] hook [hook] priority [priorité] \; \
    policy [politique_par_défaut] \; \
}

# Exemple concret : chaîne d'entrée pour filtrer le trafic entrant
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
    policy drop \; \
}
```

> [!example] Exemple détaillé
> ```bash
> # Création d'une chaîne de base pour le filtrage en sortie
> nft add chain inet mon_firewall output_filter {
>     type filter hook output priority 0 ;
>     policy accept ;
> }
> 
> # Cette chaîne va :
> # - Filtrer les paquets (type filter)
> # - Au moment où ils sortent du système (hook output)
> # - Avec une priorité standard (0)
> # - Par défaut accepter ce qui n'est pas explicitement bloqué (policy accept)
> ```

### Politiques par défaut

La **policy** définit le comportement par défaut lorsqu'aucune règle ne correspond au paquet.

| Politique | Comportement | Usage recommandé |
|-----------|--------------|------------------|
| `accept` | Accepte le paquet | Approche permissive (liste noire) |
| `drop` | Rejette silencieusement | Approche restrictive (liste blanche) - **recommandé pour la sécurité** |

> [!tip] Bonne pratique
> Pour un pare-feu sécurisé, utilisez `policy drop` et autorisez explicitement uniquement les flux nécessaires. C'est le principe du **moindre privilège**.

---

## Chaînes utilisateur (Regular Chains)

### Définition et utilité

Les **chaînes utilisateur** (ou chaînes régulières) sont des chaînes **non rattachées à un hook**. Elles ne sont donc jamais exécutées automatiquement par le noyau.

> [!info] Pourquoi utiliser des chaînes utilisateur ?
> Elles permettent d'**organiser et modulariser** vos règles. C'est comme créer des fonctions réutilisables en programmation : vous évitez la duplication de code et améliorez la lisibilité.

### Mécanisme de fonctionnement

Les chaînes utilisateur sont appelées **explicitement** depuis une chaîne de base via l'action `jump` ou `goto`.

```bash
# Création d'une chaîne utilisateur (pas de hook, type, ni priority)
nft add chain inet filtre regles_ssh

# Ajout de règles dans la chaîne utilisateur
nft add rule inet filtre regles_ssh tcp dport 22 accept

# Appel depuis une chaîne de base
nft add rule inet filtre input jump regles_ssh
```

### Jump vs Goto

| Commande | Comportement après exécution | Usage |
|----------|------------------------------|-------|
| `jump` | Retourne à la chaîne appelante et continue l'évaluation | Sous-routine - **usage courant** |
| `goto` | Ne retourne PAS, s'arrête après la chaîne ciblée | Branchement définitif - usage spécifique |

> [!example] Illustration jump vs goto
> ```bash
> # Avec JUMP : le paquet continue après la chaîne utilisateur
> nft add chain inet filtre input { type filter hook input priority 0 \; }
> nft add chain inet filtre check_ssh
> nft add rule inet filtre check_ssh tcp dport 22 log prefix "SSH: "
> nft add rule inet filtre input jump check_ssh
> nft add rule inet filtre input counter  # Cette règle sera exécutée après le jump
> 
> # Avec GOTO : le paquet s'arrête à la chaîne ciblée
> nft add rule inet filtre input tcp dport 80 goto web_rules
> nft add rule inet filtre input counter  # Cette règle NE sera PAS exécutée pour le port 80
> ```

### Cas d'usage des chaînes utilisateur

1. **Organisation thématique** : regrouper les règles par service
   ```bash
   nft add chain inet filtre regles_web
   nft add chain inet filtre regles_mail
   nft add chain inet filtre regles_dns
   ```

2. **Réutilisation de règles complexes** : éviter la duplication
   ```bash
   # Chaîne de validation d'adresses IP suspectes
   nft add chain inet filtre check_blacklist
   nft add rule inet filtre input jump check_blacklist
   nft add rule inet filtre forward jump check_blacklist
   ```

3. **Simplification de la maintenance** : modifier un ensemble de règles en un seul endroit

> [!tip] Astuce d'organisation
> Préfixez vos chaînes utilisateur avec un identifiant : `usr_ssh`, `usr_web`, `usr_vpn`. Cela permet de les distinguer rapidement des chaînes de base dans la configuration.

---

## Types de chaînes

Le **type** d'une chaîne de base détermine quelle catégorie de traitement elle effectue sur les paquets.

### Filter

**Objectif** : Filtrage classique du trafic (autoriser/bloquer).

> [!info] C'est quoi ?
> Le type `filter` permet d'inspecter les paquets et de décider de leur sort : accept, drop, reject, log, etc. C'est le type **le plus couramment utilisé** pour un pare-feu.

```bash
# Chaîne de filtrage en entrée
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
    policy drop \; \
}
```

**Hooks compatibles** : tous (prerouting, input, forward, output, postrouting)

**Actions typiques** : accept, drop, reject, log, counter, jump, goto

### NAT

**Objectif** : Translation d'adresses réseau (Network Address Translation).

> [!info] C'est quoi ?
> Le type `nat` permet de **modifier les adresses IP source ou destination** des paquets. Essentiel pour le partage de connexion Internet, le port forwarding, ou masquer un réseau privé.

```bash
# NAT sortant (masquerading) pour partager la connexion Internet
nft add chain inet filtre postrouting { \
    type nat hook postrouting priority 100 \; \
}
nft add rule inet filtre postrouting oifname "eth0" masquerade
```

**Hooks compatibles** : 
- `prerouting` : DNAT (Destination NAT - redirection de ports entrants)
- `postrouting` : SNAT/masquerade (Source NAT - partage de connexion)
- `output` : NAT pour le trafic local

**Actions typiques** : snat, dnat, masquerade

> [!warning] Attention
> Pour le NAT, seul le **premier paquet** d'une connexion traverse les règles. Les paquets suivants utilisent le suivi de connexion (conntrack). Vos règles NAT ne doivent donc matcher que les nouveaux flux.

### Route

**Objectif** : Modifier la décision de routage d'un paquet.

> [!info] C'est quoi ?
> Le type `route` permet de **re-marquer** un paquet ou de modifier sa destination **avant** la décision de routage finale. Utilisé pour le routage avancé basé sur des règles (policy-based routing).

```bash
# Marquer les paquets pour un routage alternatif
nft add chain inet filtre output { \
    type route hook output priority -150 \; \
}
nft add rule inet filtre output tcp dport 443 mark set 10
```

**Hooks compatibles** : `output` uniquement

**Actions typiques** : mark, meta (pour marquer les paquets)

> [!tip] Cas d'usage avancé
> Le type `route` est rarement utilisé sauf pour des configurations complexes comme le multi-homing (plusieurs connexions Internet), le load balancing, ou le VPN avec routage conditionnel.

---

## Hooks disponibles

Les **hooks** sont des points d'accrochage dans le parcours d'un paquet à travers le système. Ils déterminent **à quel moment** une chaîne de base sera exécutée.

### Schéma du parcours d'un paquet

```
Paquet entrant
     |
     v
[PREROUTING] -----> Décision de routage
     |                      |
     |                      v
     |              Paquet pour cette machine ?
     |                 /          \
     |               OUI          NON
     |                |            |
     v                v            v
 [INPUT]         Processus     [FORWARD]
     |              local          |
     |                |            |
     |                v            |
     |           [OUTPUT]          |
     |                |            |
     |                v            |
     +---------> Décision de routage
                      |
                      v
                [POSTROUTING]
                      |
                      v
                Paquet sortant
```

### Prerouting

**Moment** : Avant la décision de routage, dès l'arrivée du paquet sur une interface.

**Usage principal** :
- DNAT (redirection de ports)
- Marquage de paquets pour le routage
- Filtrage très précoce

```bash
# Redirection de port (DNAT)
nft add chain inet filtre prerouting { \
    type nat hook prerouting priority -100 \; \
}
nft add rule inet filtre prerouting tcp dport 8080 dnat to 192.168.1.10:80
```

> [!warning] Point d'attention
> À ce stade, le système ne sait pas encore si le paquet est destiné à la machine locale ou doit être routé. Utilisez ce hook avec précaution.

### Input

**Moment** : Après la décision de routage, pour les paquets **destinés à la machine locale**.

**Usage principal** :
- Filtrage des services locaux
- Protection de la machine elle-même
- **Le hook le plus important pour protéger un serveur**

```bash
# Filtrage d'accès SSH
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
    policy drop \; \
}
nft add rule inet filtre input iifname "lo" accept
nft add rule inet filtre input ct state established,related accept
nft add rule inet filtre input tcp dport 22 accept
```

> [!tip] Bonne pratique
> Toujours autoriser l'interface loopback (`lo`) en premier dans la chaîne input, sinon des services locaux risquent de dysfonctionner.

### Forward

**Moment** : Après la décision de routage, pour les paquets **qui transitent** par la machine (routage).

**Usage principal** :
- Filtrage inter-réseaux
- Pare-feu pour un réseau derrière la machine
- Routeur/Gateway

```bash
# Autoriser le forward entre deux réseaux
nft add chain inet filtre forward { \
    type filter hook forward priority 0 \; \
    policy drop \; \
}
nft add rule inet filtre forward iifname "lan0" oifname "wan0" accept
nft add rule inet filtre forward ct state established,related accept
```

> [!info] Prérequis système
> Pour que le hook `forward` fonctionne, le **forwarding IP** doit être activé dans le noyau :
> ```bash
> # Méthode permanente (Debian 13)
> echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
> echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
> reboot
> ```

### Output

**Moment** : Avant la décision de routage, pour les paquets **générés localement** par la machine.

**Usage principal** :
- Filtrage du trafic sortant de la machine
- Contrôle des connexions initiées par les processus locaux
- NAT pour le trafic local

```bash
# Restreindre les connexions sortantes
nft add chain inet filtre output { \
    type filter hook output priority 0 \; \
    policy accept \; \
}
nft add rule inet filtre output tcp dport 25 drop  # Bloquer SMTP sortant
```

> [!tip] Cas d'usage
> Utile pour empêcher un serveur compromis d'établir des connexions sortantes malveillantes, ou pour forcer l'utilisation d'un proxy.

### Postrouting

**Moment** : Après la décision de routage finale, juste avant l'envoi sur l'interface de sortie.

**Usage principal** :
- SNAT / Masquerading (partage de connexion)
- Marquage final des paquets
- Modification des paquets juste avant émission

```bash
# Masquerading pour partager la connexion Internet
nft add chain inet filtre postrouting { \
    type nat hook postrouting priority 100 \; \
}
nft add rule inet filtre postrouting oifname "eth0" masquerade
```

> [!info] Différence SNAT vs Masquerade
> - **SNAT** : spécifie une adresse IP source fixe (IP publique statique)
> - **Masquerade** : utilise automatiquement l'IP de l'interface de sortie (IP dynamique - DHCP)

---

## Priorités des chaînes

### Concept et importance

La **priorité** détermine l'**ordre d'exécution** lorsque plusieurs chaînes sont rattachées au même hook. Une priorité **plus basse** est exécutée **en premier**.

> [!warning] Point critique
> Si plusieurs chaînes utilisent le même hook, l'ordre d'exécution peut être déterminant pour le résultat final. Une règle de DROP exécutée avant une règle d'ACCEPT bloquera le paquet.

### Valeurs de priorité

Les priorités sont des **entiers signés** (peuvent être négatifs).

| Plage | Usage typique |
|-------|---------------|
| `-400` à `-300` | Priorités très hautes (connexion tracking, mangling précoce) |
| `-150` à `-100` | NAT destination (DNAT), routage alternatif |
| `-1` à `0` | **Filtrage standard** (le plus courant) |
| `100` à `300` | NAT source (SNAT/masquerade) |
| `1000+` | Logging, mangling final |

### Syntaxe

```bash
# Priorité explicite (valeur numérique)
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
}

# Priorité avec constante symbolique (depuis nftables 0.9.6)
nft add chain inet filtre input { \
    type filter hook input priority filter \; \
}
```

### Constantes symboliques de priorité

NFTables définit des constantes pour faciliter la compréhension :

| Constante | Valeur | Usage |
|-----------|--------|-------|
| `raw` | -300 | Avant le connexion tracking |
| `mangle` | -150 | Modification de paquets |
| `dstnat` | -100 | DNAT (avant routage) |
| `filter` | 0 | **Filtrage standard** |
| `security` | 50 | SELinux/Apparmor |
| `srcnat` | 100 | SNAT/masquerade |

> [!example] Exemple avec plusieurs chaînes sur le même hook
> ```bash
> # Chaîne de DNAT (priorité -100, exécutée en premier)
> nft add chain inet filtre prert_nat { \
>     type nat hook prerouting priority dstnat \; \
> }
> 
> # Chaîne de filtrage (priorité 0, exécutée après)
> nft add chain inet filtre prert_filter { \
>     type filter hook prerouting priority filter \; \
> }
> 
> # Ordre d'exécution : prert_nat → prert_filter
> ```

### Cas pratiques d'utilisation des priorités

**Scénario 1** : Logging avant filtrage

```bash
# Log en priorité -1 (avant le filtrage)
nft add chain inet filtre input_log { \
    type filter hook input priority -1 \; \
}
nft add rule inet filtre input_log log prefix "INPUT: "

# Filtrage en priorité 0 (après le log)
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
    policy drop \; \
}
```

**Scénario 2** : NAT puis filtrage

```bash
# DNAT en priorité -100
nft add chain inet filtre prerouting { \
    type nat hook prerouting priority -100 \; \
}

# Filtrage en priorité 0 (voit les paquets APRÈS modification NAT)
nft add chain inet filtre input { \
    type filter hook input priority 0 \; \
}
```

> [!tip] Bonne pratique
> Pour un pare-feu simple, restez sur la priorité `0` (ou `filter`). N'utilisez des priorités différentes que si vous avez un besoin précis de contrôler l'ordre d'exécution.

---

## 🎯 Points clés à retenir

1. **Chaînes de base** : rattachées à un hook, obligatoires pour que les règles soient exécutées
2. **Chaînes utilisateur** : modulaires, appelées via jump/goto, organisent le code
3. **Types** : filter (filtrage), nat (translation d'adresse), route (routage avancé)
4. **Hooks** : définissent le moment d'intervention (prerouting, input, forward, output, postrouting)
5. **Priorité** : contrôle l'ordre d'exécution (valeur basse = exécution précoce)

> [!tip] Conseil final
> Commencez simple : une table `inet`, des chaînes de base avec `type filter` et `priority 0` sur les hooks `input`, `forward`, et `output`. Ajoutez de la complexité (NAT, chaînes utilisateur, priorités multiples) uniquement quand vous en avez besoin.
