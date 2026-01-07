# Internet Protocol version 4 (IPv4)
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Internet Protocol version 4 - Adresse et paquet  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Le protocole IP|Le protocole IP]]
   - [[#L'IETF et les RFC|L'IETF et les RFC]]
   - [[#Historique et versions d'IP|Historique et versions d'IP]]
   - [[#Prérequis et glossaire|Prérequis et glossaire]]
   - [[#Objectifs d'IP|Objectifs d'IP]]
   - [[#Architecture réseau IP|Architecture réseau IP]]
2. [[#Les adresses IPv4|Les adresses IPv4]]
   - [[#Modèle d'adressage|Modèle d'adressage]]
   - [[#Structure d'une adresse IPv4|Structure d'une adresse IPv4]]
   - [[#Notation et représentation|Notation et représentation]]
   - [[#Masque de sous-réseau|Masque de sous-réseau]]
   - [[#Classes d'adresses|Classes d'adresses]]
   - [[#Adresses spéciales|Adresses spéciales]]
3. [[#Configuration réseau|Configuration réseau]]
   - [[#Paramètres de configuration|Paramètres de configuration]]
   - [[#Calculs d'adresses|Calculs d'adresses]]
   - [[#Routage|Routage]]
4. [[#Le paquet IPv4|Le paquet IPv4]]
   - [[#Structure de l'en-tête|Structure de l'en-tête]]
   - [[#Champs de l'en-tête|Champs de l'en-tête]]
   - [[#Fragmentation|Fragmentation]]
   - [[#Numéros de protocoles|Numéros de protocoles]]
5. [[#Protocoles connexes|Protocoles connexes]]
   - [[#ICMP - Internet Control Message Protocol|ICMP - Internet Control Message Protocol]]
   - [[#ARP - Address Resolution Protocol|ARP - Address Resolution Protocol]]
6. [[#Points clés à retenir|Points clés à retenir]]
7. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> L'Internet Protocol version 4 (IPv4) est le protocole fondamental qui permet l'interconnexion de réseaux physiques au niveau de la couche 3 du modèle OSI. C'est grâce à IP qu'Internet existe et fonctionne.

### Le protocole IP

> [!quote] Définition
> **IP** signifie **Internet Protocol**. C'est le protocole d'interconnexion de réseaux physiques standardisé par l'IETF, fonctionnant à la couche 3 (couche réseau) du modèle OSI.

**Caractéristiques principales** :
- Standard **IETF** (Internet Engineering Task Force)
- Couche **3 du modèle OSI** (couche réseau)
- **2 versions** actuellement en activité :
  - IP version 4 (IPv4) - la plus répandue
  - IP version 6 (IPv6) - en déploiement progressif
- Permet **Internet** : réseau mondial accessible à tous

> [!info] Rôle fondamental
> IP est le protocole qui permet à différents réseaux physiques (Ethernet, WiFi, etc.) de communiquer entre eux, créant ainsi une interconnexion de réseaux : un "internet".

### L'IETF et les RFC

#### Internet Engineering Task Force (IETF)

> [!quote] Définition de l'IETF
> L'**IETF** (Internet Engineering Task Force) est un groupe international **ouvert à tout individu** dont l'objectif est d'élaborer les protocoles d'Internet (couche 3 et supérieures).

**Objectif** : Élaborer les protocoles d'Internet (couche 3 et +)

**Production** : Request For Comments (RFC)

#### Les RFC (Request For Comments)

> [!important] Documents RFC
> Les **RFC** sont des documents numérotés (RFC \<numéro\>) qui décrivent les techniques pour faire de l'Internet (interconnexion de réseaux physiques).

**Processus de standardisation** :

| Statut | Description |
|--------|-------------|
| **Internet Draft** | Brouillon, support à discussion |
| **Proposed Standard** | Stable, en général avec au moins 1 implémentation |
| **Draft Standard** | Finale, au moins 2 implémentations |
| **Internet Standard (STD)** | Standard de fait, officiel |
| **RFC** | Peut être : Experimental, Informational, Historic |
| **Best Current Practice (BCP)** | Meilleures pratiques actuelles |

> [!example] RFC importante
> **RFC 2026 (BCP 9)** : The Internet Standards Process - Revision 3
> 
> Cette RFC décrit le processus de standardisation de l'IETF.

### Historique et versions d'IP

> [!info] Évolution du protocole IP

**Chronologie historique** :

| Année | Événement |
|-------|-----------|
| **1974** | *A Protocol for Packet Network Intercommunication* - Vinton G. Cerf, Robert E. Kahn (Mai 1974) |
| **1977** | TCP version 2 (IEN 5) - Vinton G. Cerf (Mai 1977) |
| **1978** | TCP version 3 (IEN 21) - Vinton G. Cerf, Jonathan B. Postel |
| **1980** | TCP version 4 - RFC 761 (1980) puis **RFC 793** (1981 - STD 7) |
| **1980** | **IP version 4** - RFC 760 (1980) puis **RFC 791** (1981 - STD 5) |
| **1995** | IP version 6 - RFC 1883 (1995) puis **RFC 2460** (1998 - Draft Standard) |

> [!note] RFC de référence pour IPv4
> **RFC 791** (1981 - STD 5) est le document de référence pour IPv4.

### Prérequis et glossaire

#### Prérequis d'IP

> [!important] Concepts fondamentaux
> IP n'est **pas** un protocole de niveau 1 (Physique). Il suppose l'existence de protocoles de réseaux physiques sous-jacents.

**IP s'appuie sur des protocoles de couche inférieure** :
- **Quelconques** : Ethernet, WiFi, ATM, etc.
- **Caractéristiques nécessaires** :
  - Adresses physiques (ex. : MAC pour Ethernet)
  - MTU (Maximum Transmission Unit)
  - Capacité à envoyer des trames entre interfaces d'un même réseau physique directement reliées

> [!example] Exemple de stack
> ```
> Application (HTTP, FTP...)
>      ↓
> Transport (TCP, UDP)
>      ↓
> Réseau (IP) ← Nous sommes ici
>      ↓
> Liaison (Ethernet, WiFi)
>      ↓
> Physique (câbles, ondes...)
> ```

#### Glossaire IP

> [!note] Termes essentiels pour comprendre IP

| Terme | Définition |
|-------|------------|
| **Nœud** | Équipement supportant IP (ordinateur, serveur, routeur, etc.) |
| **Routeur** | Nœud transmettant les paquets dont il n'est **pas** le destinataire |
| **Hôte** | Nœud qui n'est **pas** un routeur (ordinateur, serveur final) |
| **Couche supérieure** | Protocole transporté par IP (TCP, UDP, ICMP, etc.) |
| **Lien** | Protocole sous IP (Ethernet, WiFi, ATM, etc.) |
| **Voisins** | Nœuds attachés au **même lien** (peuvent communiquer directement) |
| **Interface** | Moyen d'accès au lien d'un nœud (carte réseau) |
| **Adresse** | Identifiant IP d'une interface (ou d'un ensemble d'interfaces) |
| **Paquet** | En-tête IP + charge utile (données) |

### Objectifs d'IP

> [!important] Services rendus par IP

Le protocole IP a pour objectifs de fournir :

1. **Transfert non fiable** de paquets indépendants
   - Pas de garantie de livraison (best effort)
   - Pas de garantie d'ordre de livraison
   
2. **Adresses logiques**
   - Identification unique des interfaces
   
3. **Routage**
   - Acheminement des paquets à travers les réseaux
   
4. **Qualité de service (QoS)**
   - Gestion des priorités de trafic
   
5. **Durée de vie des paquets**
   - Évite les boucles infinies (TTL)
   
6. **Somme de contrôle**
   - Détection d'erreurs dans l'en-tête
   
7. **Gestion de la fragmentation**
   - Adaptation aux liens à petit MTU

> [!warning] IP est non fiable !
> IP ne garantit **pas** la livraison des paquets, ni leur ordre d'arrivée. C'est le rôle des protocoles de couche supérieure (comme TCP) d'assurer la fiabilité si nécessaire.

### Architecture réseau IP

#### Le réseau au sens IP

> [!important] Distinction fondamentale

IP fait une distinction entre deux concepts :

| Concept | Définition |
|---------|------------|
| **Liens (réseaux physiques)** | Ensemble d'interfaces **directement connectées** pouvant communiquer à l'aide d'un protocole de couche 1 et 2 |
| **Réseaux (logiques)** | Ensemble d'interfaces appartenant **nécessairement au même lien** |

> [!info] Interconnexion
> Les interfaces de réseaux logiques différents peuvent communiquer via une **passerelle** (routeur) pour former une **interconnexion de réseaux** (un "internet").

#### Conception d'architecture réseau

> [!tip] Rôle de l'administrateur réseau

Un administrateur va donc :

1. **Concevoir ses réseaux IP** en fonction de ses réseaux physiques
   - Un réseau IP appartient à un réseau physique
   - Un réseau physique peut contenir plusieurs réseaux IP

2. **Interconnecter les réseaux IP** devant communiquer à l'aide de passerelles (routeurs) ayant :
   - 2 interfaces physiques pour le cas de 2 réseaux physiques distincts
   - OU 2 configurations IP distinctes pouvant être sur 1 même interface physique (VLAN)

> [!example] Schéma conceptuel
> ```
> Réseau IP A (192.168.1.0/24)    Réseau IP B (192.168.2.0/24)
>        |                                 |
>    [Switch]                          [Switch]
>        |                                 |
>    Ethernet A                        Ethernet B
>        |                                 |
>        └────────[Routeur]────────────────┘
>              (2 interfaces)
> ```

#### Configuration locale

> [!warning] Pas de coordinateur global
> Il n'y a **pas de coordinateur global** => pas de configuration globale.

**Conséquences** :
- Chaque interface dispose de sa **propre configuration locale**
- Donc de sa **propre vision des réseaux logiques**
- Pour pouvoir communiquer, ces configurations doivent être **compatibles**

> [!tip] Vision décentralisée
> Chaque nœud a sa propre vision du réseau. C'est l'ensemble cohérent de ces configurations locales qui forme l'architecture réseau globale.

---

## Les adresses IPv4

### Modèle d'adressage

#### Contexte historique

> [!info] Le contexte - IPv4 : fin des années 70
> **À l'époque d'IPv4** :
> - Début du Minitel
> - Passage du 8086 (4,77 - 10 MHz) au 80286 (6-20 MHz)
> - Le premier Macintosh (128K) n'apparaîtra qu'en 1984
> 
> Les contraintes techniques étaient très différentes d'aujourd'hui !

#### Principes d'adressage

> [!important] Adresses d'interfaces
> - Une adresse IPv4 identifie une **interface**, pas une machine
> - Une interface physique peut avoir **plusieurs adresses logiques**
> - Potentiellement **plusieurs interfaces par machine**

> [!example] Cas pratiques
> - Un serveur avec 2 cartes réseau = 2 interfaces = 2 adresses IP minimum
> - Une interface peut avoir une IP principale + des IP secondaires
> - L'interface de loopback (127.0.0.1) est une interface virtuelle

### Structure d'une adresse IPv4

> [!quote] Définition
> Une adresse IPv4 est codée sur **32 bits (4 octets)**, ce qui représente environ **4,3 milliards d'adresses** (2³² = 4 294 967 296).

> [!warning] En pratique moins d'adresses disponibles
> Certaines adresses sont réservées à des usages spéciaux, réduisant le nombre d'adresses utilisables.

#### Composition d'une adresse

> [!important] Structure en deux parties
> Une adresse IPv4 comporte **2 parties** :
> 1. **Identifiant de réseau** (préfixe) - `n` bits
> 2. **Identifiant d'interface** (hôte) - `32-n` bits

```
┌─────────────┬──────────────────┐
│  ID Réseau  │  ID Interface    │
│   (n bits)  │  (32-n bits)     │
└─────────────┴──────────────────┘
       32 bits au total
```

#### Classes historiques

> [!note] Système de classes (historique - obsolète)
> Historiquement, IPv4 utilisait un système de **3 classes principales** :

| Classe | Premiers bits | Bits réseau | Bits hôte | Plage |
|--------|---------------|-------------|-----------|-------|
| **Classe A** | `0` | 8 bits | 24 bits | 0.0.0.0 à 127.255.255.255 |
| **Classe B** | `10` | 16 bits | 16 bits | 128.0.0.0 à 191.255.255.255 |
| **Classe C** | `110` | 24 bits | 8 bits | 192.0.0.0 à 223.255.255.255 |

> [!warning] Système obsolète
> Ce système de classes est **obsolète** depuis l'introduction du CIDR (Classless Inter-Domain Routing) en 1993. Aujourd'hui, on utilise les masques de sous-réseau.

#### Exemples de décomposition

> [!example] Exemple 1 - Classe A

```
Adresse binaire : 00000001 00000000 00000000 00000011
                  ↑
                  Premier bit = 0 → Classe A

ID réseau    : 00000001 (8 bits)
ID interface : 00000000 00000000 00000011 (24 bits)
```

> [!example] Exemple 2 - Classe C

```
Adresse binaire : 11000000 10101000 00000000 11111110
                  ↑↑↑
                  Premiers bits = 110 → Classe C

ID réseau    : 11000000 10101000 00000000 (24 bits)
ID interface : 11111110 (8 bits)
```

### Notation et représentation

#### Notation décimale pointée

> [!quote] Notation décimale pointée - RFC 1166
> On ne manipule pas directement les adresses en binaire. On utilise la **notation décimale pointée** :
> - Chaque octet est converti en base 10
> - 4 octets = 4 nombres décimaux
> - Ils sont séparés par des points (`.`)

**Syntaxe** : `A.B.C.D` où A, B, C et D sont des nombres de 0 à 255

> [!example] Exemples de conversion

```
Binaire : 00000001 00000000 00000000 00000011
Décimal : 1.0.0.3

Binaire : 11000000 10101000 00000000 11111110
Décimal : 192.168.0.254

Binaire : 10101100 00010000 00000001 00000001
Décimal : 172.16.1.1
```

> [!tip] Astuce de conversion
> Pour convertir du binaire au décimal :
> - Chaque bit a une valeur : 128, 64, 32, 16, 8, 4, 2, 1
> - Additionne les valeurs des bits à 1
> 
> Exemple : `11000000` = 128 + 64 = **192**

### Masque de sous-réseau

> [!quote] Définition
> Le **masque de sous-réseau** (subnet mask) permet de séparer la partie réseau de la partie hôte dans une adresse IP.

#### Structure du masque

> [!important] Composition du masque
> - Le masque est également sur **32 bits**
> - Partie réseau : bits à **1**
> - Partie hôte : bits à **0**
> - Les bits à 1 sont **contigus**, suivis des bits à 0

```
Exemple de masque classe C :
Binaire : 11111111 11111111 11111111 00000000
Décimal : 255.255.255.0
          └─────────┬─────────┘  └───┬───┘
            Réseau (24 bits)    Hôte (8 bits)
```

#### Notation CIDR

> [!info] Notation CIDR (Classless Inter-Domain Routing)
> On note souvent le masque avec un **/n** où `n` est le nombre de bits à 1 du masque.

**Exemples** :

| Notation CIDR | Masque décimal | Bits réseau | Bits hôte |
|---------------|----------------|-------------|-----------|
| `/8` | 255.0.0.0 | 8 | 24 |
| `/16` | 255.255.0.0 | 16 | 16 |
| `/24` | 255.255.255.0 | 24 | 8 |
| `/30` | 255.255.255.252 | 30 | 2 |

> [!example] Notation complète d'une adresse
> - Adresse : `192.168.1.10`
> - Masque : `255.255.255.0` ou `/24`
> - Notation complète : **192.168.1.10/24**

#### Opération avec le masque

> [!important] ET logique (AND)
> Pour extraire l'adresse réseau d'une adresse IP, on effectue un **ET logique** entre l'adresse et le masque.

> [!example] Calcul de l'adresse réseau

```
Adresse IP    : 192.168.1.10    11000000.10101000.00000001.00001010
Masque        : 255.255.255.0   11111111.11111111.11111111.00000000
              ─────────────────────────────────────────────────────── (ET)
Adresse réseau: 192.168.1.0     11000000.10101000.00000001.00000000
```

### Classes d'adresses

#### Tableau récapitulatif des classes

| Classe | 1er octet | Masque par défaut | Nombre de réseaux | Hôtes par réseau | Usage |
|--------|-----------|-------------------|-------------------|------------------|-------|
| **A** | 1-126 | /8 (255.0.0.0) | 126 | 16 777 214 | Très grands réseaux |
| **B** | 128-191 | /16 (255.255.0.0) | 16 384 | 65 534 | Réseaux moyens |
| **C** | 192-223 | /24 (255.255.255.0) | 2 097 152 | 254 | Petits réseaux |
| **D** | 224-239 | N/A | N/A | N/A | Multicast |
| **E** | 240-255 | N/A | N/A | N/A | Réservé/expérimental |

> [!note] Note sur le nombre d'hôtes
> Le nombre d'hôtes utilisables = 2^(bits hôte) - 2
> - On retire l'adresse réseau (tous les bits hôte à 0)
> - On retire l'adresse de broadcast (tous les bits hôte à 1)

### Adresses spéciales

> [!important] Adresses réservées

#### Adresse réseau et broadcast

| Type | Description | Exemple (réseau 192.168.1.0/24) |
|------|-------------|----------------------------------|
| **Adresse réseau** | Tous les bits hôte à **0** | 192.168.1.0 |
| **Adresse broadcast** | Tous les bits hôte à **1** | 192.168.1.255 |

> [!warning] Ces adresses ne peuvent PAS être attribuées à des hôtes
> - L'adresse réseau identifie le réseau lui-même
> - L'adresse broadcast permet d'envoyer à tous les hôtes du réseau

#### Adresses privées (RFC 1918)

> [!info] Plages d'adresses privées
> Ces adresses sont **non routables sur Internet** et réservées aux réseaux privés.

| Classe | Plage | Notation CIDR | Nombre d'adresses |
|--------|-------|---------------|-------------------|
| **Classe A** | 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | 16 777 216 |
| **Classe B** | 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | 1 048 576 |
| **Classe C** | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 65 536 |

> [!tip] Utilisation
> Ces adresses sont utilisées dans les réseaux locaux (LAN) des entreprises et domiciles. L'accès à Internet se fait via NAT (Network Address Translation).

#### Autres adresses spéciales

| Adresse | Signification |
|---------|---------------|
| **0.0.0.0** | Adresse "toutes interfaces" ou route par défaut |
| **127.0.0.0/8** | Loopback (boucle locale) - 127.0.0.1 typiquement |
| **169.254.0.0/16** | APIPA - Auto-configuration en l'absence de DHCP |
| **255.255.255.255** | Broadcast limité (réseau local uniquement) |

> [!example] Adresse de loopback
> **127.0.0.1** (ou **localhost**) : permet à une machine de communiquer avec elle-même. Utilisé pour les tests locaux.

---

## Configuration réseau

### Paramètres de configuration

> [!important] Configuration minimale d'une interface IPv4

Pour configurer une interface réseau en IPv4, il faut au minimum :

1. **Adresse IP** : identifiant unique de l'interface
2. **Masque de sous-réseau** : définit la partie réseau/hôte

> [!tip] Configuration complète recommandée

Pour une connexion fonctionnelle, on ajoute généralement :

3. **Passerelle par défaut (Gateway)** : routeur permettant de sortir du réseau local
4. **Serveurs DNS** : pour la résolution de noms de domaine

> [!example] Exemple de configuration

```
Interface     : eth0
Adresse IP    : 192.168.1.100
Masque        : 255.255.255.0 (/24)
Passerelle    : 192.168.1.1
DNS primaire  : 8.8.8.8
DNS secondaire: 8.8.4.4
```

#### Méthodes de configuration

| Méthode | Description | Usage |
|---------|-------------|-------|
| **Statique** | Configuration manuelle fixe | Serveurs, équipements réseau |
| **DHCP** | Configuration automatique dynamique | Postes clients, mobilité |
| **APIPA** | Auto-configuration (169.254.x.x) | Échec DHCP, réseau local minimal |

### Calculs d'adresses

#### Calculer le nombre d'hôtes

> [!tip] Formule du nombre d'hôtes utilisables

```
Nombre d'hôtes = 2^(32-n) - 2
```

Où `n` est le nombre de bits du masque (notation /n)

> [!example] Exemples de calculs

```
Réseau 192.168.1.0/24 :
- Bits hôte : 32 - 24 = 8 bits
- Nombre d'hôtes : 2^8 - 2 = 256 - 2 = 254 hôtes

Réseau 10.0.0.0/16 :
- Bits hôte : 32 - 16 = 16 bits
- Nombre d'hôtes : 2^16 - 2 = 65 536 - 2 = 65 534 hôtes

Réseau 172.16.0.0/30 :
- Bits hôte : 32 - 30 = 2 bits
- Nombre d'hôtes : 2^2 - 2 = 4 - 2 = 2 hôtes (lien point-à-point)
```

#### Identifier l'adresse réseau et broadcast

> [!example] Réseau 192.168.10.0/24

```
Adresse réseau : 192.168.10.0
Première IP utilisable : 192.168.10.1
Dernière IP utilisable : 192.168.10.254
Adresse broadcast : 192.168.10.255
Nombre d'hôtes : 254
```

> [!example] Réseau 172.16.5.128/25

```
Masque : /25 = 255.255.255.128

Adresse réseau : 172.16.5.128
Première IP utilisable : 172.16.5.129
Dernière IP utilisable : 172.16.5.254
Adresse broadcast : 172.16.5.255
Nombre d'hôtes : 2^7 - 2 = 126 hôtes
```

#### Découpage en sous-réseaux (subnetting)

> [!info] Principe du subnetting
> Le **subnetting** consiste à diviser un réseau en plusieurs sous-réseaux plus petits en empruntant des bits à la partie hôte.

> [!example] Découpage d'un /24 en 4 sous-réseaux

```
Réseau initial : 192.168.1.0/24 (254 hôtes)

On emprunte 2 bits : /24 → /26 (4 sous-réseaux de 62 hôtes chacun)

Sous-réseau 1 : 192.168.1.0/26   (192.168.1.0 - 192.168.1.63)
Sous-réseau 2 : 192.168.1.64/26  (192.168.1.64 - 192.168.1.127)
Sous-réseau 3 : 192.168.1.128/26 (192.168.1.128 - 192.168.1.191)
Sous-réseau 4 : 192.168.1.192/26 (192.168.1.192 - 192.168.1.255)
```

> [!tip] Formule pour le subnetting
> - Nombre de sous-réseaux : **2^bits empruntés**
> - Hôtes par sous-réseau : **2^(bits hôtes restants) - 2**

### Routage

#### Principe du routage

> [!quote] Définition
> Le **routage** est le processus de sélection du chemin qu'un paquet IP doit emprunter pour atteindre sa destination.

> [!important] Table de routage
> Chaque nœud IP possède une **table de routage** qui contient :
> - Les réseaux connus (directement connectés ou appris)
> - L'interface de sortie pour chaque réseau
> - La passerelle (next hop) si le réseau n'est pas directement connecté

#### Processus de routage

> [!info] Décision de routage

Lorsqu'un nœud veut envoyer un paquet :

1. **Consultation de la table de routage**
   - Compare l'adresse destination avec les entrées de la table
   
2. **Sélection de la route** :
   - Si destination dans le réseau local → **livraison directe**
   - Sinon → envoi vers la **passerelle par défaut**

3. **Transmission** :
   - Le paquet est envoyé à l'interface appropriée

> [!example] Exemple de décision

```
Configuration locale :
- IP : 192.168.1.10/24
- Gateway : 192.168.1.1

Destination : 192.168.1.50
→ Même réseau (192.168.1.0/24) → Livraison directe (ARP)

Destination : 8.8.8.8
→ Réseau différent → Envoi vers gateway 192.168.1.1
```

#### Route par défaut

> [!important] Route par défaut (Default Gateway)
> La **route par défaut** (`0.0.0.0/0`) est utilisée quand aucune route spécifique ne correspond à la destination.

```
Notation : 0.0.0.0/0 via 192.168.1.1
```

> [!tip] Configuration courante
> Sur un poste client, on configure généralement :
> - Une route vers le réseau local (automatique)
> - Une route par défaut vers le routeur/box

---

## Le paquet IPv4

### Structure de l'en-tête

> [!important] En-tête IPv4
> L'en-tête IPv4 a une taille minimale de **20 octets** (160 bits) et peut aller jusqu'à **60 octets** avec les options.

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if IHL > 5)                       |
|                           + Padding                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Champs de l'en-tête

#### Première ligne (32 bits)

> [!note] Champs de contrôle de base

| Champ | Taille | Position | Description |
|-------|--------|----------|-------------|
| **Version** | 4 bits | Bits 0-3 | Version du protocole IP (4 pour IPv4) |
| **IHL** | 4 bits | Bits 4-7 | Internet Header Length - Longueur de l'en-tête en mots de 32 bits (min: 5, max: 15) |
| **Type of Service (ToS)** | 8 bits | Bits 8-15 | Qualité de service (QoS), priorité du paquet |
| **Total Length** | 16 bits | Bits 16-31 | Longueur totale du paquet (en-tête + données) en octets (max: 65 535) |

> [!example] IHL (Internet Header Length)
> - Valeur minimale : **5** (5 × 4 = 20 octets)
> - Valeur maximale : **15** (15 × 4 = 60 octets)
> - Si IHL = 5 : pas d'options, en-tête standard de 20 octets

#### Deuxième ligne (32 bits)

> [!note] Champs de fragmentation

| Champ | Taille | Position | Description |
|-------|--------|----------|-------------|
| **Identification** | 16 bits | Bits 32-47 | Identifiant unique du paquet (pour le réassemblage des fragments) |
| **Flags** | 3 bits | Bits 48-50 | Indicateurs de fragmentation |
| **Fragment Offset** | 13 bits | Bits 51-63 | Position du fragment dans le paquet original (en blocs de 8 octets) |

**Détail des Flags** :

| Bit | Nom | Signification |
|-----|-----|---------------|
| Bit 0 | Réservé | Toujours à 0 |
| Bit 1 | **DF** (Don't Fragment) | 0 = peut fragmenter, 1 = ne pas fragmenter |
| Bit 2 | **MF** (More Fragments) | 0 = dernier fragment, 1 = d'autres fragments suivent |

> [!important] Rôle des flags
> - **DF = 1** : Si fragmentation nécessaire, le paquet est **rejeté** et un message ICMP est envoyé
> - **MF = 1** : Indique que ce paquet est un fragment et que d'autres fragments suivent

#### Troisième ligne (32 bits)

> [!note] Champs de contrôle et protocole

| Champ | Taille | Position | Description |
|-------|--------|----------|-------------|
| **Time To Live (TTL)** | 8 bits | Bits 64-71 | Durée de vie du paquet (nombre de sauts max) |
| **Protocol** | 8 bits | Bits 72-79 | Numéro du protocole encapsulé (TCP=6, UDP=17, ICMP=1) |
| **Header Checksum** | 16 bits | Bits 80-95 | Somme de contrôle de l'en-tête IP (recalculée à chaque hop car TTL change) |

> [!important] Time To Live (TTL)
> Le **TTL** est **décrémenté de 1** à chaque passage par un routeur. Si TTL atteint 0, le paquet est **détruit** et un message ICMP "Time Exceeded" est envoyé à l'émetteur.

> [!tip] Usage du TTL
> - Évite les **boucles infinies** de routage
> - Utilisé par `traceroute` pour découvrir le chemin réseau
> - Valeur initiale typique : 64, 128 ou 255

#### Quatrième et cinquième lignes (64 bits)

> [!note] Adresses source et destination

| Champ | Taille | Position | Description |
|-------|--------|----------|-------------|
| **Source Address** | 32 bits | Bits 96-127 | Adresse IP source (émetteur du paquet) |
| **Destination Address** | 32 bits | Bits 128-159 | Adresse IP destination (destinataire du paquet) |

#### Options et padding

> [!info] En-tête variable
> L'en-tête IP peut comporter des **options** (rarement utilisées en pratique).

**Si options présentes** :
- Le champ **IHL** est > 5
- On ajoute du **bourrage (padding)** pour que l'en-tête soit un multiple de 32 bits

> [!example] Options possibles
> - Record Route : enregistrer les routeurs traversés
> - Timestamp : horodatage des passages
> - Source Routing : spécifier le chemin à suivre

### Fragmentation

> [!quote] Problématique de la fragmentation
> Le paquet IP doit être transmis dans la trame du lien sous-jacent. Cette trame a un **MTU** (Maximum Transmission Unit) en général très inférieur à la limite de 65 535 octets d'IP.

#### Contexte de la fragmentation

> [!important] Scénario nécessitant la fragmentation

Les passerelles IP (routeurs) peuvent parfois :
1. **Recevoir** un paquet IP sur un lien avec un MTU donné
2. Devoir le **transférer** sur un autre lien avec un MTU **plus petit**
3. Ne pas pouvoir transmettre le paquet en l'état

**Solution** : Le routeur doit **fragmenter** le paquet.

> [!warning] Fragmentation multiple
> Cette fragmentation peut avoir lieu **plusieurs fois** sur le chemin entre source et destination.

> [!example] Exemple de MTU par technologie

| Technologie | MTU typique |
|-------------|-------------|
| **Ethernet** | 1500 octets |
| **WiFi** | 1500 octets (généralement) |
| **PPPoE (ADSL)** | 1492 octets |
| **VPN/Tunnel** | Variable (souvent < 1500) |

#### Mécanisme de fragmentation

> [!important] Processus de fragmentation

**Quand le MTU est trop petit** :

**Si Don't Fragment (DF) = 1** :
- Le paquet est **jeté**
- L'émetteur est averti via **ICMP "Fragmentation Needed"**

**Si DF = 0** (fragmentation autorisée) :

1. **Découpe du paquet** en plusieurs fragments
2. **Recopie de l'en-tête** pour chaque fragment
3. **Modification des champs** :
   - **Total Length** : taille du fragment
   - **More Fragments (MF)** :
     - Fragments 1 à n-1 : MF = 1
     - Dernier fragment : MF = 0
   - **Fragment Offset** : position dans le paquet original (en blocs de 8 octets)
   - **Identification** : même valeur pour tous les fragments

> [!important] Réassemblage
> Le **réassemblage** du paquet se fait **uniquement par le destinataire final**, pas par les routeurs intermédiaires !

> [!example] Exemple de fragmentation

```
Paquet original : 3000 octets de données + 20 octets d'en-tête = 3020 octets
MTU du lien : 1500 octets

Fragment 1 :
- En-tête : 20 octets
- Données : 1480 octets (1500 - 20)
- MF = 1, Offset = 0

Fragment 2 :
- En-tête : 20 octets
- Données : 1480 octets
- MF = 1, Offset = 185 (1480/8)

Fragment 3 :
- En-tête : 20 octets
- Données : 40 octets (reste)
- MF = 0, Offset = 370 (2960/8)
```

> [!tip] Path MTU Discovery
> Pour éviter la fragmentation, on peut utiliser **Path MTU Discovery** :
> - Émettre des paquets avec DF = 1
> - Réduire progressivement la taille jusqu'à ce qu'il passe
> - Détermine le MTU minimal du chemin

### Numéros de protocoles

> [!important] Champ Protocol
> Le champ **Protocol** (8 bits) contient le numéro du protocole encapsulé dans IP.

> [!info] Standardisation IANA
> Ces numéros sont **standardisés** et maintenus par l'**IANA** (Internet Assigned Numbers Authority - composante de l'ICANN).

#### Protocoles courants

| Numéro | Protocole | Description |
|--------|-----------|-------------|
| **1** | ICMP | Internet Control Message Protocol |
| **2** | IGMP | Internet Group Management Protocol |
| **4** | IPv4 | IPv4 encapsulé (tunneling) |
| **6** | TCP | Transmission Control Protocol |
| **17** | UDP | User Datagram Protocol |
| **41** | IPv6 | IPv6 encapsulé (tunneling 6in4) |
| **47** | GRE | Generic Routing Encapsulation |
| **50** | ESP | Encapsulating Security Payload (IPsec) |
| **51** | AH | Authentication Header (IPsec) |
| **89** | OSPF | Open Shortest Path First |

> [!note] Réutilisation
> On retrouve cette même numérotation dans les en-têtes **IPv6** ainsi qu'en général chaque fois qu'un protocole IETF a besoin de référencer un protocole.

> [!tip] Liste complète
> La liste complète est disponible sur le site de l'IANA : **Protocol Numbers**

---

## Protocoles connexes

### ICMP - Internet Control Message Protocol

> [!quote] Définition - RFC 792
> **ICMP** (Internet Control Message Protocol) est le **protocole de contrôle** associé à IP. Quand IP a besoin de communiquer un message d'erreur ou de contrôle, il utilise un paquet ICMP.

#### Caractéristiques d'ICMP

> [!important] Points clés
> - ICMP circule **dans IP** (numéro de protocole = **1**)
> - Utilisé pour les **messages d'erreur** et de **contrôle**
> - Ne garantit pas la livraison (comme IP)
> - Essentiel au bon fonctionnement d'IP

#### Structure d'un paquet ICMP

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Code      |          Checksum             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### Types ICMP courants

> [!example] Types de messages ICMP

| Type | Code | Message | Usage |
|------|------|---------|-------|
| **0** | 0 | Echo Reply | Réponse à un ping |
| **3** | 0-15 | Destination Unreachable | Destination inaccessible (réseau, hôte, port, etc.) |
| **5** | 0-3 | Redirect | Redirection vers un meilleur routeur |
| **8** | 0 | Echo Request | Requête ping |
| **11** | 0-1 | Time Exceeded | TTL expiré ou timeout de réassemblage |

> [!tip] Liste complète
> La liste complète des types ICMP est disponible sur le site de l'**IANA**.

#### Utilisations pratiques d'ICMP

> [!example] Commande `ping`
> ```bash
> ping 8.8.8.8
> ```
> - Envoie des **ICMP Echo Request** (type 8)
> - Attend des **ICMP Echo Reply** (type 0)
> - Permet de tester la connectivité réseau

> [!example] Commande `traceroute`
> ```bash
> traceroute google.com
> ```
> - Envoie des paquets avec TTL croissant (1, 2, 3...)
> - Chaque routeur renvoie **ICMP Time Exceeded** (type 11)
> - Permet de découvrir le chemin réseau

> [!warning] ICMP et sécurité
> Certains administrateurs **bloquent ICMP** pour des raisons de sécurité, ce qui peut rendre le diagnostic réseau plus difficile. C'est une mauvaise pratique : il vaut mieux filtrer intelligemment.

### ARP - Address Resolution Protocol

> [!quote] Définition - RFC 826
> **ARP** (Address Resolution Protocol) est le protocole permettant de faire le lien entre la **couche 2 (liaison)** et la **couche 3 (réseau)**.

#### Problématique résolue par ARP

> [!important] Le problème
> Lors d'une communication IP :
> 1. L'**adresse IP** de destination est connue (couche 3)
> 2. Mais pour insérer le paquet IP dans une **trame Ethernet**, il faut l'**adresse MAC** de destination (couche 2) !
> 
> **ARP** résout ce problème : il traduit une adresse IP en adresse MAC.

> [!info] Résolution d'adresse
> ARP = Association entre adresse IP (logique) et adresse MAC (physique)

#### Structure du paquet ARP

> [!note] Encapsulation ARP
> - ARP est encapsulé **directement dans Ethernet**
> - **EtherType** : `0x0806`
> - Adapté à différents protocoles réseau (couche 3) sur différents liens (couche 2)

**Champs principaux** :

| Champ | Taille | Description |
|-------|--------|-------------|
| **Hardware Type** | 16 bits | Type de réseau physique (Ethernet = 1) |
| **Protocol Type** | 16 bits | Type de protocole réseau (IPv4 = 0x0800) |
| **Hardware Length** | 8 bits | Longueur adresse physique (6 pour MAC) |
| **Protocol Length** | 8 bits | Longueur adresse logique (4 pour IPv4) |
| **Operation** | 16 bits | Type d'opération (1 = requête, 2 = réponse) |
| **Sender Hardware Address** | Variable | Adresse MAC source |
| **Sender Protocol Address** | Variable | Adresse IP source |
| **Target Hardware Address** | Variable | Adresse MAC destination (0 pour requête) |
| **Target Protocol Address** | Variable | Adresse IP destination |

> [!note] Adresse cible à 0
> Dans une requête ARP, l'**adresse MAC destination** est mise à **00:00:00:00:00:00** car elle est justement inconnue (c'est ce qu'on cherche !).

#### Fonctionnement général d'ARP

> [!example] Processus de résolution ARP

**Scénario** : Machine A (192.168.1.10) veut communiquer avec Machine B (192.168.1.20)

**Étapes** :

1. **Machine A vérifie son cache ARP**
   - Si l'association IP↔MAC existe déjà : utilisation directe
   - Sinon : envoi d'une requête ARP

2. **Requête ARP en broadcast**
   ```
   ARP Request (broadcast sur le réseau) :
   "Qui a l'adresse IP 192.168.1.20 ? Répondez à 192.168.1.10 (MAC: AA:BB:CC:DD:EE:FF)"
   ```

3. **Machine B répond**
   ```
   ARP Reply (unicast vers Machine A) :
   "192.168.1.20 est à l'adresse MAC 11:22:33:44:55:66"
   ```

4. **Machine A met en cache la réponse**
   - Stockage dans la **table ARP** (cache temporaire)
   - Durée de vie : quelques minutes typiquement

5. **Communication possible**
   - Machine A peut maintenant encapsuler ses paquets IP dans des trames Ethernet avec la bonne MAC

> [!tip] Commandes utiles

```bash
# Voir la table ARP
arp -a
ip neigh show

# Vider le cache ARP (Linux)
ip neigh flush all

# Ajouter une entrée statique
arp -s 192.168.1.20 11:22:33:44:55:66
```

#### Cache ARP

> [!info] Table ARP
> Chaque machine maintient une **table ARP** (cache) pour ne pas redemander à chaque fois.

**Caractéristiques du cache** :
- **Temporaire** : les entrées expirent après un certain temps (timeout)
- **Dynamique** : mis à jour automatiquement
- **Consultable** : visible avec `arp -a` ou `ip neigh`

> [!example] Exemple de table ARP

```bash
$ arp -a
? (192.168.1.1) at 00:11:22:33:44:55 [ether] on eth0
? (192.168.1.20) at aa:bb:cc:dd:ee:ff [ether] on eth0
? (192.168.1.254) at 11:22:33:44:55:66 [ether] on eth0
```

#### Problèmes de sécurité ARP

> [!warning] Vulnérabilités d'ARP
> ARP n'a **aucun mécanisme d'authentification**. Il est vulnérable à plusieurs attaques :

| Attaque | Description |
|---------|-------------|
| **ARP Spoofing** | Usurpation d'identité en envoyant de fausses réponses ARP |
| **ARP Poisoning** | Empoisonnement du cache ARP des machines du réseau |
| **Man-in-the-Middle** | Interception du trafic en se faisant passer pour la passerelle |

> [!tip] Protection
> - **ARP statique** : entrées fixes pour équipements critiques
> - **Port Security** : limitation des MAC par port sur les switches
> - **Dynamic ARP Inspection** (DAI) : validation des paquets ARP par le switch
> - **Segmentation réseau** : limiter la portée des attaques

> [!info] Pour aller plus loin
> Pour approfondir les problèmes de sécurité liés à ARP, voir les ressources spécialisées en sécurité réseau.

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Introduction et concepts

- **IP** = Internet Protocol, protocole de **couche 3** (réseau) du modèle OSI
- Standardisé par l'**IETF** via des **RFC** (RFC 791 pour IPv4)
- IPv4 créé en **1980**, toujours majoritairement utilisé aujourd'hui
- Permet l'**interconnexion de réseaux physiques** hétérogènes
- **Non fiable** : pas de garantie de livraison (best effort)

### Structure des adresses IPv4

- Adresse sur **32 bits (4 octets)** = environ 4,3 milliards d'adresses
- **2 parties** : ID réseau (préfixe) + ID interface (hôte)
- Notation **décimale pointée** : A.B.C.D (ex: 192.168.1.10)
- Notation **CIDR** : adresse/masque (ex: 192.168.1.0/24)

### Masque de sous-réseau

- **Sépare** la partie réseau de la partie hôte
- Bits à **1** = réseau, bits à **0** = hôte
- Notation **/n** : nombre de bits à 1 (ex: /24 = 255.255.255.0)
- **ET logique** entre adresse et masque = adresse réseau

### Classes d'adresses (historique)

| Classe | 1er octet | Masque | Hôtes/réseau |
|--------|-----------|--------|--------------|
| A | 1-126 | /8 | 16 777 214 |
| B | 128-191 | /16 | 65 534 |
| C | 192-223 | /24 | 254 |

### Adresses spéciales

- **Adresse réseau** : tous bits hôte à 0 (ex: 192.168.1.0)
- **Broadcast** : tous bits hôte à 1 (ex: 192.168.1.255)
- **Privées RFC 1918** : 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- **Loopback** : 127.0.0.0/8 (typiquement 127.0.0.1)
- **APIPA** : 169.254.0.0/16 (auto-configuration)

### Configuration réseau

- **Minimum** : Adresse IP + Masque
- **Complet** : + Passerelle + DNS
- **Méthodes** : Statique, DHCP, APIPA
- Formule **nombre d'hôtes** : 2^(32-n) - 2

### Structure du paquet IPv4

- **En-tête minimale** : 20 octets (peut aller jusqu'à 60 avec options)
- **Champs essentiels** :
  - Version (4), IHL, ToS, Total Length
  - Identification, Flags, Fragment Offset
  - TTL, Protocol, Header Checksum
  - Source Address, Destination Address

### Champs importants

- **TTL** : Durée de vie (décrément à chaque hop), évite boucles infinies
- **Protocol** : Numéro du protocole encapsulé (1=ICMP, 6=TCP, 17=UDP)
- **Flags** : DF (Don't Fragment), MF (More Fragments)
- **Total Length** : Maximum 65 535 octets

### Fragmentation

- Nécessaire quand **paquet > MTU** du lien
- Si **DF = 1** : paquet rejeté, ICMP envoyé
- Si **DF = 0** : fragmentation en plusieurs paquets
- **Réassemblage** uniquement par le **destinataire final**
- MTU Ethernet standard : **1500 octets**

### Routage

- **Table de routage** : détermine l'interface/passerelle pour chaque destination
- **Livraison directe** : destination dans le réseau local
- **Routage indirect** : via passerelle par défaut (0.0.0.0/0)
- Décision basée sur l'**adresse réseau** (IP dest AND masque)

### ICMP (Protocol 1)

- Protocole de **contrôle et erreur** d'IP
- Types courants :
  - **Type 8** : Echo Request (ping)
  - **Type 0** : Echo Reply (pong)
  - **Type 3** : Destination Unreachable
  - **Type 11** : Time Exceeded (TTL expiré)
- Utilisé par **ping** et **traceroute**

### ARP (EtherType 0x0806)

- Résout **IP → MAC** (couche 3 → couche 2)
- **Requête** en broadcast : "Qui a cette IP ?"
- **Réponse** en unicast : "Moi, voici ma MAC"
- **Cache ARP** : stockage temporaire des associations
- **Vulnérable** : ARP Spoofing, ARP Poisoning, MitM

### Glossaire technique - Rappel des termes clés

- **Nœud** : équipement supportant IP
- **Routeur** : nœud transmettant les paquets (passerelle)
- **Hôte** : nœud qui n'est pas un routeur
- **Interface** : point d'accès au réseau (carte réseau)
- **Lien** : réseau physique (Ethernet, WiFi)
- **Réseau** : ensemble logique d'interfaces sur le même lien
- **Paquet** : en-tête IP + données

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **IP (Internet Protocol)** | Protocole de couche 3 permettant l'interconnexion de réseaux physiques |
| **IPv4** | Version 4 d'IP, adresses sur 32 bits, RFC 791 (1981) |
| **IETF** | Internet Engineering Task Force - organisme de standardisation d'Internet |
| **RFC** | Request For Comments - documents décrivant les standards Internet |
| **Adresse IP** | Identifiant logique unique d'une interface réseau (32 bits pour IPv4) |
| **Masque de sous-réseau** | Détermine la séparation entre partie réseau et partie hôte |
| **CIDR** | Classless Inter-Domain Routing - notation adresse/masque (ex: /24) |
| **Classe A/B/C** | Ancien système d'adressage hiérarchique (obsolète, remplacé par CIDR) |
| **Adresse réseau** | Identifie un réseau logique (bits hôte à 0) |
| **Adresse broadcast** | Adresse de diffusion vers tous les hôtes d'un réseau (bits hôte à 1) |
| **Adresse privée** | Adresse non routable sur Internet (RFC 1918) |
| **Nœud** | Équipement supportant le protocole IP |
| **Routeur** | Nœud transmettant les paquets entre réseaux différents |
| **Hôte** | Nœud terminal (non-routeur) |
| **Interface** | Point d'accès d'un nœud à un lien réseau |
| **Lien** | Réseau physique (Ethernet, WiFi, etc.) |
| **Voisins** | Nœuds sur le même lien (communication directe possible) |
| **Paquet** | Unité de données IP (en-tête + charge utile) |
| **En-tête IP** | Informations de contrôle du paquet (minimum 20 octets) |
| **TTL** | Time To Live - nombre de sauts maximum avant destruction du paquet |
| **MTU** | Maximum Transmission Unit - taille maximale d'une trame |
| **Fragmentation** | Découpage d'un paquet IP pour l'adapter au MTU du lien |
| **DF** | Don't Fragment - flag interdisant la fragmentation |
| **MF** | More Fragments - flag indiquant qu'il y a d'autres fragments |
| **IHL** | Internet Header Length - longueur de l'en-tête en mots de 32 bits |
| **ToS** | Type of Service - champ de qualité de service (QoS) |
| **Checksum** | Somme de contrôle pour détecter les erreurs dans l'en-tête |
| **Protocol** | Numéro identifiant le protocole encapsulé (1=ICMP, 6=TCP, 17=UDP) |
| **Passerelle** | Routeur permettant de sortir d'un réseau local (default gateway) |
| **Route par défaut** | Route 0.0.0.0/0 utilisée quand aucune route spécifique ne correspond |
| **Table de routage** | Liste des réseaux connus et des chemins pour les atteindre |
| **Subnetting** | Découpage d'un réseau en sous-réseaux plus petits |
| **ICMP** | Internet Control Message Protocol - protocole de contrôle et erreur d'IP |
| **ARP** | Address Resolution Protocol - résout IP → MAC |
| **Loopback** | Interface virtuelle permettant la communication interne (127.0.0.1) |
| **APIPA** | Automatic Private IP Addressing - auto-configuration (169.254.x.x) |
| **DHCP** | Dynamic Host Configuration Protocol - attribution automatique d'adresses |
| **Unicast** | Communication point à point (1 émetteur → 1 destinataire) |
| **Broadcast** | Communication vers tous (1 émetteur → tous sur le réseau) |
| **Multicast** | Communication de groupe (1 émetteur → groupe d'abonnés) |
| **NAT** | Network Address Translation - traduction d'adresses (privé ↔ public) |
| **Cache ARP** | Table temporaire des associations IP ↔ MAC |
| **IANA** | Internet Assigned Numbers Authority - gère les numéros de protocoles |
| **Best effort** | Mode de livraison non garanti (IP fait de son mieux) |

---

## Résumé final

> [!success] À retenir pour le titre RNCP

**Adresse et Réseau IP version 4** :
- Adresses sur 32 bits avec structure réseau/hôte
- Notation décimale pointée et CIDR
- Masque de sous-réseau pour séparer réseau/hôte
- Adresses spéciales : réseau, broadcast, privées
- Calculs : nombre d'hôtes, sous-réseaux

**Le paquet IPv4** :
- En-tête de 20 octets minimum avec champs essentiels
- TTL pour éviter les boucles
- Fragmentation pour s'adapter au MTU
- Protocol pour identifier l'encapsulation
- Checksum pour la détection d'erreurs

**Protocoles complémentaires nécessaires à IP** :
- **ICMP** : contrôle et erreurs (ping, traceroute)
- **ARP** : résolution IP → MAC (lien couche 2/3)

**Architecture et routage** :
- Configuration locale : IP, masque, passerelle, DNS
- Table de routage pour décisions d'acheminement
- Interconnexion de réseaux via routeurs

---

*Document créé pour la préparation au titre RNCP Technicien Supérieur Systèmes et Réseaux (TSSR)*  
*Compatible Obsidian avec callouts natifs*