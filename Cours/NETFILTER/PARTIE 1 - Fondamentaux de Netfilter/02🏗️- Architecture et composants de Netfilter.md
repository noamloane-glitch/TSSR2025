# 

## 📋 Table des matières

- [Les tables](#les-tables)
  - [Table filter](#table-filter)
  - [Table nat](#table-nat)
  - [Table mangle](#table-mangle)
  - [Table raw](#table-raw)
- [Les chaînes](#les-chaînes)
  - [PREROUTING](#prerouting)
  - [INPUT](#input)
  - [FORWARD](#forward)
  - [OUTPUT](#output)
  - [POSTROUTING](#postrouting)
- [Le cheminement des paquets](#le-cheminement-des-paquets)
- [Les hooks du noyau](#les-hooks-du-noyau)

---

## Les tables

Les **tables** sont des conteneurs logiques qui regroupent des chaînes de règles selon leur fonction. Chaque table a un rôle spécifique dans le traitement des paquets.

> [!info] Concept clé
> Une table définit **QUOI** faire avec les paquets (filtrer, traduire, modifier...), tandis que les chaînes définissent **QUAND** le faire (à quel moment du parcours).

### Vue d'ensemble des tables

| Table | Fonction principale | Cas d'usage typique |
|-------|-------------------|-------------------|
| **filter** | Filtrage des paquets | Autoriser/bloquer le trafic (pare-feu) |
| **nat** | Translation d'adresses | Partage de connexion, redirection de ports |
| **mangle** | Modification des en-têtes | QoS, marquage de paquets |
| **raw** | Gestion du suivi de connexion | Exclure certains flux du conntrack |

---

### Table filter

La table **filter** est la table par défaut et la plus utilisée. Son rôle est de décider si un paquet doit être accepté ou rejeté.

#### Caractéristiques

```bash
# Table par défaut : pas besoin de spécifier -t filter
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# Équivalent à :
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
```

#### Chaînes disponibles

La table filter contient **3 chaînes** :

| Chaîne | Rôle | Exemple d'utilisation |
|--------|------|---------------------|
| **INPUT** | Paquets destinés au système local | Contrôler les connexions entrantes vers les services |
| **OUTPUT** | Paquets générés par le système local | Contrôler les connexions sortantes depuis le système |
| **FORWARD** | Paquets transitant par le système | Filtrer le trafic routé (système agit comme routeur) |

> [!example] Exemple d'utilisation
> ```bash
> # Autoriser SSH entrant
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> 
> # Bloquer les connexions sortantes vers Facebook
> iptables -A OUTPUT -d 157.240.0.0/16 -j DROP
> 
> # Autoriser le trafic HTTP transitant vers le réseau interne
> iptables -A FORWARD -p tcp --dport 80 -j ACCEPT
> ```

#### Cibles principales

```bash
# ACCEPT : Accepter le paquet
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# DROP : Rejeter silencieusement (pas de réponse)
iptables -A INPUT -p tcp --dport 23 -j DROP

# REJECT : Rejeter avec notification (envoie un message ICMP)
iptables -A INPUT -p tcp --dport 23 -j REJECT

# LOG : Journaliser puis continuer le traitement
iptables -A INPUT -p tcp --dport 23 -j LOG --log-prefix "TELNET-ATTEMPT: "
```

> [!tip] Astuce professionnelle
> Utilisez **DROP** pour les scans de ports (pas de réponse = plus difficile à détecter).  
> Utilisez **REJECT** pour les services légitimes non activés (l'utilisateur comprend que le service n'est pas disponible).

> [!warning] Piège courant
> La table filter ne peut PAS faire de NAT ! Si vous essayez d'ajouter des règles SNAT/DNAT dans filter, elles seront ignorées ou provoqueront des erreurs.

---

### Table nat

La table **nat** (Network Address Translation) gère la translation d'adresses IP et de ports. Elle est essentielle pour le partage de connexion et la redirection de services.

#### Caractéristiques

```bash
# Toujours spécifier explicitement -t nat
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

#### Chaînes disponibles

La table nat contient **3 chaînes principales** :

| Chaîne | Rôle | Moment d'application |
|--------|------|---------------------|
| **PREROUTING** | Modification avant routage (DNAT) | Dès l'arrivée du paquet |
| **POSTROUTING** | Modification après routage (SNAT) | Juste avant l'envoi du paquet |
| **OUTPUT** | Paquets générés localement | Pour les paquets créés par le système |

> [!info] DNAT vs SNAT
> - **DNAT** (Destination NAT) : Change l'IP/port de **destination** → utilisé pour rediriger du trafic entrant
> - **SNAT** (Source NAT) : Change l'IP/port de **source** → utilisé pour le partage de connexion Internet

#### Types de NAT

```bash
# SNAT : Translation source avec IP fixe
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.5

# MASQUERADE : Translation source avec IP dynamique (ADSL, DHCP)
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE

# DNAT : Redirection de port (port forwarding)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080

# REDIRECT : Redirection locale (vers le système lui-même)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3128
```

> [!example] Scénario classique : Partage de connexion
> ```bash
> # Le serveur Linux a :
> # - eth0 : 203.0.113.5 (IP publique Internet)
> # - eth1 : 192.168.1.1 (réseau local)
> 
> # Activer le routage
> echo 1 > /proc/sys/net/ipv4/ip_forward
> 
> # Configurer le NAT pour partager la connexion
> iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
> 
> # Les machines du LAN (192.168.1.0/24) peuvent maintenant accéder à Internet
> # via l'IP publique du serveur
> ```

> [!warning] Attention
> Les règles NAT ne filtrent PAS le trafic ! Vous devez toujours utiliser la table **filter** en complément pour sécuriser votre système.

---

### Table mangle

La table **mangle** permet de modifier les en-têtes des paquets IP. Elle est utilisée pour des fonctionnalités avancées comme la QoS (Quality of Service) ou le marquage de paquets.

#### Caractéristiques

```bash
# Spécifier -t mangle
iptables -t mangle -A PREROUTING -p tcp --dport 22 -j TOS --set-tos 0x10
```

#### Chaînes disponibles

La table mangle contient **5 chaînes** (toutes les chaînes possibles) :

- **PREROUTING**
- **INPUT**
- **FORWARD**
- **OUTPUT**
- **POSTROUTING**

#### Utilisations courantes

| Action | Description | Exemple |
|--------|-------------|---------|
| **TOS** | Modifier le Type of Service | Prioriser le trafic SSH |
| **TTL** | Modifier le Time To Live | Masquer la présence d'un routeur |
| **MARK** | Marquer les paquets | Identification pour routage avancé |
| **DSCP** | Modifier Differentiated Services | QoS pour VoIP |

```bash
# Marquer le trafic HTTP pour routage spécial
iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-mark 1

# Modifier le TTL pour éviter la détection de NAT
iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64

# Prioriser le trafic SSH (ToS minimum delay)
iptables -t mangle -A OUTPUT -p tcp --dport 22 -j TOS --set-tos 0x10

# Configurer DSCP pour la VoIP (Expedited Forwarding)
iptables -t mangle -A POSTROUTING -p udp --dport 5060 -j DSCP --set-dscp 46
```

> [!info] Quand utiliser mangle ?
> La table mangle est rarement utilisée dans des configurations basiques. Elle devient nécessaire pour :
> - La mise en place de QoS complexe
> - Le routage basé sur des politiques (policy-based routing)
> - L'optimisation de performances réseau
> - La manipulation avancée de trafic

> [!tip] Conseil
> Si vous débutez avec Netfilter, concentrez-vous d'abord sur les tables **filter** et **nat**. La table mangle est pour des besoins avancés.

---

### Table raw

La table **raw** est la première table consultée et sert principalement à exempter certains flux du suivi de connexion (conntrack).

#### Caractéristiques

```bash
# Spécifier -t raw
iptables -t raw -A PREROUTING -p tcp --dport 80 -j NOTRACK
```

#### Chaînes disponibles

La table raw contient **2 chaînes** :

- **PREROUTING** : Paquets entrants avant toute décision de routage
- **OUTPUT** : Paquets générés localement

#### Pourquoi utiliser raw ?

Le suivi de connexion (conntrack) consomme des ressources. Sur un serveur web à fort trafic, désactiver le conntrack peut améliorer les performances.

```bash
# Désactiver le suivi pour le trafic HTTP (serveur web très chargé)
iptables -t raw -A PREROUTING -p tcp --dport 80 -j NOTRACK
iptables -t raw -A OUTPUT -p tcp --sport 80 -j NOTRACK

# Vérifier la table de connexions
conntrack -L | wc -l
```

> [!warning] Conséquence importante
> Si vous utilisez NOTRACK, vous **ne pouvez plus** utiliser le suivi d'état (state/conntrack) pour ces connexions dans vos règles de filtrage !

> [!example] Cas d'usage typique
> ```bash
> # Serveur web public recevant 100 000 requêtes/seconde
> # Le conntrack devient un goulot d'étranglement
> 
> # Solution : désactiver le suivi pour HTTP/HTTPS
> iptables -t raw -A PREROUTING -p tcp -m multiport --dports 80,443 -j NOTRACK
> iptables -t raw -A OUTPUT -p tcp -m multiport --sports 80,443 -j NOTRACK
> 
> # Impact : réduction de 30-40% de la charge CPU sur le pare-feu
> ```

> [!info] Cas d'usage avancé
> La table raw peut aussi servir à définir des règles de priorité absolue, car elle est traitée avant toutes les autres tables, y compris avant le suivi de connexion.

---

### Ordre de priorité des tables

Lorsqu'un paquet traverse un hook, les tables sont consultées dans un ordre précis :

```
1. raw         → Décisions sur le suivi
2. mangle      → Modifications d'en-têtes
3. nat (DNAT)  → Translation destination (si applicable)
4. filter      → Décisions de filtrage
5. nat (SNAT)  → Translation source (si applicable)
```

> [!tip] Mnémotechnique
> **R**aw **M**angle **N**at **F**ilter → "**R**ien **M**ieux **N**'a **F**onctionné"

---

## Les chaînes

Les **chaînes** (chains) sont des listes ordonnées de règles. Chaque paquet traverse différentes chaînes selon son parcours dans le système.

> [!info] Concept clé
> Les chaînes définissent **QUAND** appliquer les règles, selon le point du parcours où se trouve le paquet.

### Vue d'ensemble des chaînes

```
                    ┌─────────────────┐
    Paquet entrant  │   PREROUTING    │  Avant décision de routage
                    └────────┬────────┘
                             │
                   ┌─────────▼─────────┐
                   │  Décision routage │
                   └───┬───────────┬───┘
                       │           │
         Pour le système│           │Pour un autre système
                       │           │
                 ┌─────▼──────┐  ┌─▼────────┐
                 │   INPUT    │  │ FORWARD  │
                 └─────┬──────┘  └─┬────────┘
                       │            │
                       ▼            │
                 ┌──────────┐       │
                 │Processus │       │
                 │  local   │       │
                 └────┬─────┘       │
                      │             │
                 ┌────▼──────┐      │
                 │  OUTPUT   │      │
                 └────┬──────┘      │
                      │             │
                   ┌──▼─────────────▼──┐
                   │   POSTROUTING     │  Après décision de routage
                   └─────────┬─────────┘
                             │
                        Paquet sortant
```

---

### PREROUTING

La chaîne **PREROUTING** traite les paquets **immédiatement après leur réception**, avant toute décision de routage.

#### Caractéristiques

- 📍 **Position** : Première chaîne traversée par un paquet entrant
- 🎯 **Usage principal** : DNAT (redirection de trafic entrant)
- 📦 **Tables disponibles** : raw, mangle, nat

#### Quand l'utiliser ?

```bash
# DNAT : Rediriger le trafic web vers un serveur interne
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
    -j DNAT --to-destination 192.168.1.10

# Marquer le trafic avant routage
iptables -t mangle -A PREROUTING -p tcp --dport 443 -j MARK --set-mark 1

# Exclure du suivi de connexion
iptables -t raw -A PREROUTING -p tcp --dport 8080 -j NOTRACK
```

> [!example] Scénario : Serveur web en DMZ
> ```bash
> # Topologie :
> # Internet → Pare-feu (IP publique: 203.0.113.5)
> #          → Serveur web DMZ (IP privée: 10.0.0.10)
> 
> # Rediriger le port 80 public vers le serveur web interne
> iptables -t nat -A PREROUTING -d 203.0.113.5 -p tcp --dport 80 \
>     -j DNAT --to-destination 10.0.0.10:80
> 
> # Note : Le paquet n'a pas encore été routé, donc on peut changer
> # sa destination avant que le noyau décide où l'envoyer
> ```

> [!warning] Attention
> PREROUTING est exécuté AVANT la décision de routage. Donc :
> - ✅ Parfait pour modifier la destination (DNAT)
> - ❌ Ne pas utiliser pour filtrer (utilisez INPUT/FORWARD après routage)

---

### INPUT

La chaîne **INPUT** traite les paquets **destinés au système local** après la décision de routage.

#### Caractéristiques

- 📍 **Position** : Après routage, pour trafic local
- 🎯 **Usage principal** : Filtrage des connexions entrantes
- 📦 **Tables disponibles** : mangle, filter

#### Quand l'utiliser ?

```bash
# Autoriser SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Bloquer les pings (ICMP echo request)
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# Autoriser uniquement depuis une IP spécifique
iptables -A INPUT -s 192.168.1.50 -j ACCEPT

# Limiter les tentatives de connexion SSH (protection brute-force)
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
    -m recent --set
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW \
    -m recent --update --seconds 60 --hitcount 4 -j DROP
```

> [!example] Scénario : Serveur web sécurisé
> ```bash
> # Politique par défaut : tout bloquer
> iptables -P INPUT DROP
> 
> # Autoriser le loopback (obligatoire pour le bon fonctionnement)
> iptables -A INPUT -i lo -j ACCEPT
> 
> # Autoriser les connexions établies (réponses aux requêtes sortantes)
> iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
> 
> # Autoriser HTTP et HTTPS
> iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT
> 
> # Autoriser SSH uniquement depuis l'administration
> iptables -A INPUT -p tcp --dport 22 -s 203.0.113.100 -j ACCEPT
> ```

> [!tip] Bonne pratique
> Toujours autoriser le loopback en premier :
> ```bash
> iptables -A INPUT -i lo -j ACCEPT
> ```
> De nombreuses applications locales en dépendent !

---

### FORWARD

La chaîne **FORWARD** traite les paquets **routés à travers le système** (ni source, ni destination locale).

#### Caractéristiques

- 📍 **Position** : Après routage, pour trafic transité
- 🎯 **Usage principal** : Filtrage sur routeur/passerelle
- 📦 **Tables disponibles** : mangle, filter

#### Quand l'utiliser ?

Le système agit comme **routeur** ou **passerelle** entre deux réseaux.

```bash
# Activer le routage (prérequis obligatoire)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Autoriser le trafic du LAN vers Internet
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# Autoriser les réponses de retour
iptables -A FORWARD -i eth0 -o eth1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Bloquer l'accès à un site spécifique depuis le LAN
iptables -A FORWARD -d facebook.com -j REJECT
```

> [!example] Scénario : Passerelle d'entreprise
> ```bash
> # Topologie :
> # LAN (192.168.1.0/24, eth1) ← Passerelle Linux → Internet (eth0)
> 
> # Politique par défaut : bloquer le transit
> iptables -P FORWARD DROP
> 
> # Autoriser le LAN vers Internet
> iptables -A FORWARD -i eth1 -o eth0 -s 192.168.1.0/24 -j ACCEPT
> 
> # Autoriser les réponses
> iptables -A FORWARD -i eth0 -o eth1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
> 
> # Bloquer les réseaux sociaux pendant les heures de travail
> iptables -A FORWARD -m time --timestart 09:00 --timestop 18:00 \
>     --weekdays Mon,Tue,Wed,Thu,Fri \
>     -d 157.240.0.0/16 -j REJECT --reject-with icmp-host-prohibited
> ```

> [!warning] Erreur fréquente
> Si vous configurez un routeur et que rien ne passe :
> 1. Vérifiez que le routage est activé : `cat /proc/sys/net/ipv4/ip_forward`
> 2. Vérifiez les règles FORWARD : `iptables -L FORWARD -n -v`
> 3. Vérifiez le NAT : `iptables -t nat -L POSTROUTING -n -v`

---

### OUTPUT

La chaîne **OUTPUT** traite les paquets **générés par le système local** avant leur envoi.

#### Caractéristiques

- 📍 **Position** : Avant envoi, pour trafic local sortant
- 🎯 **Usage principal** : Contrôler les connexions sortantes
- 📦 **Tables disponibles** : raw, mangle, nat, filter

#### Quand l'utiliser ?

```bash
# Bloquer l'accès sortant à un domaine
iptables -A OUTPUT -d malware.example.com -j REJECT

# Autoriser uniquement certains utilisateurs à accéder au web
iptables -A OUTPUT -p tcp --dport 80 -m owner --uid-owner 1000 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j DROP

# Forcer les requêtes DNS vers un serveur spécifique
iptables -t nat -A OUTPUT -p udp --dport 53 ! -d 8.8.8.8 \
    -j DNAT --to-destination 8.8.8.8
```

> [!example] Scénario : Serveur durci
> ```bash
# Politique par défaut : bloquer les sorties
> iptables -P OUTPUT DROP
> 
> # Autoriser le loopback
> iptables -A OUTPUT -o lo -j ACCEPT
> 
> # Autoriser les réponses aux connexions entrantes
> iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
> 
> # Autoriser seulement les mises à jour système (HTTP/HTTPS vers dépôts Ubuntu)
> iptables -A OUTPUT -p tcp -m multiport --dports 80,443 \
>     -d 91.189.88.0/21 -j ACCEPT  # Adresses des serveurs Ubuntu
> 
> # Autoriser DNS
> iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
> 
> # Tout le reste est bloqué
> ```

> [!tip] Cas d'usage avancé
> La chaîne OUTPUT peut utiliser le module **owner** pour filtrer par utilisateur :
> ```bash
> # Seul root peut faire des pings
> iptables -A OUTPUT -p icmp -m owner ! --uid-owner 0 -j DROP
> ```

---

### POSTROUTING

La chaîne **POSTROUTING** traite les paquets **juste avant leur envoi sur le réseau**, après toutes les décisions de routage.

#### Caractéristiques

- 📍 **Position** : Dernière étape avant envoi
- 🎯 **Usage principal** : SNAT et masquerading
- 📦 **Tables disponibles** : mangle, nat

#### Quand l'utiliser ?

```bash
# SNAT : Remplacer l'IP source par une IP publique fixe
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.5

# MASQUERADE : Même chose avec IP dynamique (ADSL, DHCP)
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE

# Modifier le ToS en sortie
iptables -t mangle -A POSTROUTING -p tcp --sport 22 -j TOS --set-tos 0x10
```

> [!example] Scénario : Partage de connexion (NAT)
> ```bash
> # Configuration :
> # eth0 : Connexion Internet (IP publique)
> # eth1 : Réseau local (192.168.1.0/24)
> 
> # Étape 1 : Activer le routage
> echo 1 > /proc/sys/net/ipv4/ip_forward
> 
> # Étape 2 : Configurer le SNAT
> iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -j MASQUERADE
> 
> # Résultat : Les machines du LAN peuvent accéder à Internet
> # Leurs paquets sortent avec l'IP publique du serveur comme source
> ```

> [!info] SNAT vs MASQUERADE
> - **SNAT** : Plus rapide, mais nécessite une IP source fixe
> - **MASQUERADE** : Plus flexible (IP dynamique), mais légèrement plus lent
> 
> Utilisez SNAT pour des serveurs avec IP fixe, MASQUERADE pour des connexions ADSL/4G.

---

### Chaînes personnalisées

Vous pouvez créer vos propres chaînes pour organiser vos règles :

```bash
# Créer une chaîne personnalisée
iptables -N SERVICES_WEB

# Ajouter des règles dans cette chaîne
iptables -A SERVICES_WEB -p tcp --dport 80 -j ACCEPT
iptables -A SERVICES_WEB -p tcp --dport 443 -j ACCEPT

# Appeler cette chaîne depuis INPUT
iptables -A INPUT -j SERVICES_WEB
```

> [!tip] Avantage
> Les chaînes personnalisées permettent de :
> - Organiser logiquement les règles
> - Réutiliser des ensembles de règles
> - Faciliter la maintenance
> - Améliorer la lisibilité

---

## Le cheminement des paquets

Comprendre le parcours complet d'un paquet à travers Netfilter est essentiel pour créer des règles efficaces.

### Parcours détaillé avec les tables

```
                        PAQUET ENTRANT
                              │
                              ▼
        ┌──────────────────────────────────────┐
        │         PREROUTING                   │
        │  raw → mangle → nat (DNAT)           │
        └─────────────────┬────────────────────┘
                          │
                ┌─────────▼──────────┐
                │ DÉCISION DE ROUTAGE│
                │ Pour moi ? Transit? │
                └──┬──────────────┬──┘
                   │              │
        Pour ce système          Pour un autre système
                   │              │
         ┌─────────▼───────┐  ┌──▼─────────────┐
         │     INPUT       │  │    FORWARD      │
         │ mangle → filter │  │ mangle → filter │
         └────────┬────────┘  └────┬────────────┘
                  │                │
                  ▼                │
         ┌─────────────────┐       │
         │   PROCESSUS     │       │
         │     LOCAL       │       │
         └────────┬────────┘       │
                  │                │
         ┌────────▼────────┐       │
         │     OUTPUT      │       │
         │ raw → mangle    │       │
         │   → nat → filter│       │
         └────────┬────────┘       │
                  │                │
               ┌──▼────────────────▼───┐
               │ DÉCISION DE ROUTAGE   │
               │ Interface de sortie ? │
               └──────────┬─────────────┘
                          │
        ┌─────────────────▼──────────────────┐
        │         POSTROUTING                │
        │     mangle → nat (SNAT)            │
        └─────────────────┬──────────────────┘
                          │
                          ▼
                   PAQUET SORTANT
```

### Exemples de parcours complets

> [!example] Exemple 1 : Requête web entrante vers le serveur local
> ```
> Internet → eth0 → [PREROUTING/raw] → [PREROUTING/mangle] 
>          → [PREROUTING/nat] → Routage → [INPUT/mangle] 
>          → [INPUT/filter] → Apache (port 80)
> ```

> [!example] Exemple 2 : Serveur web qui répond
> ```
> Apache → [OUTPUT/raw] → [OUTPUT/mangle] → [OUTPUT/nat] 
>       → [OUTPUT/filter] → Routage → [POSTROUTING/mangle] 
>       → [POSTROUTING/nat] → eth0 → Internet
> ```

> [!example] Exemple 3 : Paquet transitant (routeur NAT)
> ```
> LAN (eth1) → [PREROUTING/raw] → [PREROUTING/mangle] 
>            → [PREROUTING/nat] → Routage → [FORWARD/mangle] 
>            → [FORWARD/filter] → Routage → [POSTROUTING/mangle] 
>            → [POSTROUTING/nat (MASQUERADE)] → eth0 → Internet
> ```

### Ordre de traitement des tables dans chaque chaîne

| Chaîne | Ordre des tables |
|--------|-----------------|
| **PREROUTING** | raw → mangle → nat |
| **INPUT** | mangle → filter |
| **FORWARD** | mangle → filter |
| **OUTPUT** | raw → mangle → nat → filter |
| **POSTROUTING** | mangle → nat |

> [!warning] Piège fréquent
> Beaucoup d'administrateurs pensent que filter s'applique partout en premier. **C'est faux !**
> 
> Par exemple, dans PREROUTING, la table nat s'exécute AVANT que le paquet atteigne filter (qui n'existe même pas dans PREROUTING).

---

## Les hooks du noyau

Les **hooks** (ou points d'ancrage) sont les emplacements dans le code du noyau Linux où Netfilter peut intercepter et traiter les paquets.

### Concept des hooks

Un hook est un point d'interception dans le parcours d'un paquet réseau au sein du noyau.

```c
// Simplifié : Code du noyau Linux (en C)
int ip_rcv(struct sk_buff *skb) {
    // Réception du paquet
    
    // HOOK NF_INET_PRE_ROUTING
    NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING, skb, ...);
    
    // Décision de routage
    ip_route_input(skb);
    
    if (paquet_pour_nous) {
        // HOOK NF_INET_LOCAL_IN
        NF_HOOK(NFPROTO_IPV4, NF_INET_LOCAL_IN, skb, ...);
    } else {
        // HOOK NF_INET_FORWARD
        NF_HOOK(NFPROTO_IPV4, NF_INET_FORWARD, skb, ...);
    }
    
    // etc.
}
```

### Les 5 hooks Netfilter

| Hook | Constante noyau | Chaîne iptables | Moment |
|------|----------------|-----------------|--------|
| **1** | NF_INET_PRE_ROUTING | PREROUTING | Dès réception du paquet |
| **2** | NF_INET_LOCAL_IN | INPUT | Paquet pour le système local |
| **3** | NF_INET_FORWARD | FORWARD | Paquet à router ailleurs |
| **4** | NF_INET_LOCAL_OUT | OUTPUT | Paquet créé localement |
| **5** | NF_INET_POST_ROUTING | POSTROUTING | Avant envoi sur le réseau |

> [!info] Fonctionnement interne
> Quand un paquet atteint un hook :
> 1. Le noyau appelle toutes les fonctions enregistrées pour ce hook
> 2. Chaque table (raw, mangle, nat, filter) enregistre ses fonctions
> 3. Les fonctions sont appelées dans l'ordre de priorité des tables
> 4. Chaque fonction examine les règles de ses chaînes
> 5. Une décision finale est prise (ACCEPT, DROP, etc.)

### Priorités des hooks

À l'intérieur d'un même hook, les tables s'exécutent selon des priorités numériques :

```c
// Priorités dans le noyau (valeurs approximatives)
NF_IP_PRI_RAW         = -300
NF_IP_PRI_CONNTRACK   = -200
NF_IP_PRI_MANGLE      = -150
NF_IP_PRI_NAT_DST     = -100  // DNAT
NF_IP_PRI_FILTER      = 0
NF_IP_PRI_NAT_SRC     = 100   // SNAT
```

> [!tip] Impact pratique
> Vous n'avez **pas besoin** de connaître ces valeurs par cœur, mais sachez que :
> - raw s'exécute toujours en premier
> - filter s'exécute au milieu
> - nat (SNAT) s'exécute après filter

### Verdicts possibles au niveau d'un hook

Quand une règle correspond à un paquet, elle retourne un verdict :

| Verdict | Signification | Action du noyau |
|---------|--------------|----------------|
| **NF_ACCEPT** | Accepter | Passe au hook suivant ou à destination |
| **NF_DROP** | Rejeter | Paquet supprimé immédiatement |
| **NF_STOLEN** | Volé | Le paquet est pris en charge ailleurs |
| **NF_QUEUE** | En file | Envoyé vers l'espace utilisateur |
| **NF_REPEAT** | Répéter | Traiter à nouveau ce hook |

```bash
# En pratique avec iptables :
iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # → NF_ACCEPT
iptables -A INPUT -p tcp --dport 23 -j DROP    # → NF_DROP
iptables -A INPUT -p tcp --dport 80 -j NFQUEUE # → NF_QUEUE
```

> [!example] NFQUEUE : Traitement dans l'espace utilisateur
> ```bash
> # Envoyer les paquets HTTP vers un programme de filtrage personnalisé
> iptables -A INPUT -p tcp --dport 80 -j NFQUEUE --queue-num 0
> 
> # Un programme en espace utilisateur (ex: avec libnetfilter_queue)
> # peut alors analyser, modifier, ou rejeter ces paquets
> ```

---

## 📝 Points clés à retenir

> [!tip] Résumé de la partie
> 
> **Les 4 tables** :
> - **filter** : Filtrage (ACCEPT/DROP/REJECT) - Table par défaut
> - **nat** : Translation d'adresses (SNAT/DNAT/MASQUERADE)
> - **mangle** : Modification d'en-têtes (ToS, TTL, MARK)
> - **raw** : Gestion du suivi de connexion (NOTRACK)
> 
> **Les 5 chaînes** :
> - **PREROUTING** : Avant routage (DNAT)
> - **INPUT** : Vers le système local
> - **FORWARD** : Transit par le système
> - **OUTPUT** : Depuis le système local
> - **POSTROUTING** : Après routage (SNAT)
> 
> **Ordre de traitement** :
> - Tables : raw → mangle → nat → filter
> - Le parcours dépend de la destination du paquet
> 
> **Hooks du noyau** :
> - 5 points d'interception dans le parcours réseau
> - Correspondent aux 5 chaînes principales
> - Permettent au noyau d'appeler les fonctions Netfilter

---

## 🎯 Bonnes pratiques

> [!warning] Pièges à éviter
> - ❌ Ne pas confondre PREROUTING (avant routage) et INPUT (après routage pour trafic local)
> - ❌ Ne pas oublier que FORWARD nécessite `ip_forward=1`
> - ❌ Ne pas essayer de faire du SNAT dans PREROUTING (c'est pour POSTROUTING)
> - ❌ Ne pas essayer de filtrer dans la table nat (utilisez filter)
> - ❌ Ne pas croire que filter s'applique partout en premier

> [!tip] Conseils professionnels
> - ✅ Utilisez la table filter pour 90% de vos besoins de pare-feu
> - ✅ DNAT va dans PREROUTING, SNAT va dans POSTROUTING
> - ✅ Dessinez le flux de vos paquets avant d'écrire les règles
> - ✅ Testez avec `iptables -L -t <table> -n -v` pour vérifier vos règles
> - ✅ Utilisez des chaînes personnalisées pour organiser des ensembles complexes de règles
> - ✅ Documentez quel hook/table/chaîne vous utilisez et pourquoi

> [!tip] Méthode de débogage
> ```bash
> # Tracer le parcours d'un paquet avec LOG
> iptables -t raw -A PREROUTING -p tcp --dport 80 -j LOG --log-prefix "RAW-PRE: "
> iptables -t mangle -A PREROUTING -p tcp --dport 80 -j LOG --log-prefix "MANGLE-PRE: "
> iptables -t nat -A PREROUTING -p tcp --dport 80 -j LOG --log-prefix "NAT-PRE: "
> iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "FILTER-IN: "
> 
> # Puis consultez les logs
> tail -f /var/log/kern.log
> ```