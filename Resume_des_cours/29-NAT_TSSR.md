# NAT - Network Address Translation

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : NAT - Différentes méthodes de translation d'adresse  
**Date** : Janvier 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Contexte|Contexte]]
   - [[#Problématique des adresses IP privées|Problématique des adresses IP privées]]
   - [[#Problématique des adresses publiques|Problématique des adresses publiques]]
   - [[#Définition du NAT|Définition du NAT]]
   - [[#Objectifs|Objectifs]]
   - [[#Masquer son plan d'adressage|Masquer son plan d'adressage]]
2. [[#Principes de fonctionnement|Principes de fonctionnement]]
   - [[#Critères de classification|Critères de classification]]
   - [[#Sens de traduction|Sens de traduction]]
   - [[#Mode d'association|Mode d'association]]
   - [[#Niveau de traduction|Niveau de traduction]]
3. [[#PAT/NAPT|PAT/NAPT]]
   - [[#Définition PAT|Définition]]
   - [[#Exemple PAT|Exemple]]
   - [[#Inconvénients PAT|Inconvénients]]
4. [[#DNAT|DNAT]]
   - [[#Définition DNAT|Définition]]
   - [[#Exemple DNAT|Exemple]]
   - [[#Inconvénients DNAT|Inconvénients]]
5. [[#NAT 1:1|NAT 1:1]]
   - [[#Définition NAT 1:1|Définition]]
   - [[#Exemple NAT 1:1|Exemple]]
   - [[#Inconvénients NAT 1:1|Inconvénients]]
6. [[#Aller plus loin|Aller plus loin]]
   - [[#Serveur derrière NAT|Serveur derrière NAT]]
   - [[#Depuis l'extérieur|Depuis l'extérieur]]
   - [[#Traverser des NAT|Traverser des NAT]]
7. [[#Points clés à retenir|Points clés à retenir]]
8. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le NAT (Network Address Translation) est un mécanisme essentiel permettant de traduire des adresses IP pour résoudre la pénurie d'adresses IPv4 publiques. Il fait la jonction entre les réseaux privés (LAN) utilisant des adresses RFC 1918 et les réseaux publics (WAN) nécessitant des adresses routables sur Internet.

### Pourquoi étudier le NAT ?

En tant que **TSSR**, tu dois comprendre le NAT car :
- C'est un mécanisme **omniprésent** dans les infrastructures réseaux (box Internet, routeurs d'entreprise, pare-feu)
- Il permet de **pallier la pénurie d'adresses IPv4**
- Il est configuré sur les **routeurs et pare-feu** que tu administreras
- Il est essentiel pour la **publication de services** et l'**accès Internet**

### Contexte

> [!quote] Contexte IPv4
> La problématique du NAT n'intervient que pour les adresses IPv4. Dans ce cours, la notation IP signifie implicitement IPv4.

> [!info] NAT et IPv6
> Le NAT est un mécanisme quasi-exclusif à l'IPv4. En IPv6, il existe malgré tout le NPTv6, mais il n'est plus indispensable grâce à l'abondance d'adresses IPv6.

### Problématique des adresses IP privées

Un réseau interne utilise des **adresses IP privées** définies par la RFC 1918 :

| Plage réseau | Première adresse | Dernière adresse | Nombre d'adresses |
|--------------|------------------|------------------|-------------------|
| **10.0.0.0/8** | 10.0.0.0 | 10.255.255.255 | ~16,7 millions |
| **172.16.0.0/12** | 172.16.0.0 | 172.31.255.255 | ~1 million |
| **192.168.0.0/16** | 192.168.0.0 | 192.168.255.255 | ~65 000 |

> [!warning] Limitation des adresses privées
> Ces adresses sont **non-routables sur Internet**. Elles ne peuvent pas être utilisées directement pour communiquer avec l'extérieur.

### Problématique des adresses publiques

Pour permettre :
- L'**accès à Internet** depuis le réseau interne
- La **publication de services** vers l'extérieur

Il est nécessaire d'utiliser des **adresses IP publiques**.

> [!info] Attribution des adresses publiques
> L'attribution des adresses publiques suit une hiérarchie :
> **ICANN** → **RIR** (RIPE NCC pour l'Europe) → **LIR** (fournisseurs Internet) → **Utilisateurs finaux**

#### Le problème de la rareté

IPv4 fournit environ **4,3 × 10⁹ adresses** (4,3 milliards).

Avec :
- L'**explosion du nombre d'équipements connectés**
- La croissance continue d'Internet

Les adresses IPv4 publiques sont devenues **"rares"** et précieuses.

> [!important] NAT comme solution transitoire
> Le NAT apparaît comme une solution transitoire à la pénurie d'adresses IPv4 en attendant le déploiement d'IPv6.

### Définition du NAT

> [!quote] Définition officielle
> Le **NAT** (Network Address Translation) permet de traduire une adresse IP.

Le NAT fait la jonction entre :
- Un **réseau privé LAN** contenant des adresses IP privées
- Un **réseau public WAN** contenant des adresses IP publiques

> [!info] Emplacement du NAT
> Le NAT est généralement mis en place sur un **routeur** ou un **pare-feu**.

Les mécanismes de traduction d'adresses permettent à un routeur de **modifier les paquets IP** au moment de leur transmission, en remplaçant :
- Une adresse IP **source** ou **destination**
- Éventuellement un **port** source ou destination

L'objectif est de substituer une adresse valable sur le réseau interne (privée) par une autre adresse valable sur un autre réseau (publique sur Internet).

> [!note] Terminologie alternative
> On qualifie parfois ce mécanisme de **masquage** (masquerade) d'adresse car il cache une adresse interne à un réseau externe.

> [!info] Normalisation
> Le NAT existe en plusieurs variantes et est défini notamment dans la **RFC 3022**.

### Objectifs

**Historiquement**, le NAT était utilisé pour **cacher son plan d'adressage interne** (sécurité par l'obscurité).

**Aujourd'hui**, il est massivement utilisé pour **combler le manque d'adresses IPv4 publiques**.

> [!important] Usage principal actuel
> Le NAT permet d'utiliser **une seule adresse publique** pour permettre à **plusieurs machines clientes** d'accéder à Internet.

### Masquer son plan d'adressage

#### Cas d'usage : interconnexion de deux organisations

Imaginons deux organisations qui ont chacune leur réseau :
- Chacune possède un plan d'adressage IPv4 (probablement en RFC 1918)
- Elles décident d'interconnecter leurs réseaux
- Il est très probable que leurs plans d'adressage soient **incompatibles** (utilisation des mêmes plages d'adresses)

**Problématique** : Comment connecter les 2 réseaux des 2 entreprises ?

##### Solution 1 : Faire du routage simple

Certaines plages réseaux se trouvent à plusieurs endroits ⇒ des adresses sont identiques des deux côtés.

> [!warning] Impossibilité technique
> Impossibilité technique de connecter les deux réseaux avec du routage simple classique.

##### Solution 2 : Faire du routage avec du NAT

> [!success] Solution NAT
> Mettre du NAT sur le routeur d'interconnexion permet de rendre les réseaux compatibles en **masquant les adresses incompatibles**.

---

## Principes de fonctionnement

> [!abstract] Comprendre les mécanismes du NAT
> Le NAT peut être classifié selon trois critères principaux qui peuvent être combinés pour former différents types de NAT.

### Critères de classification

Il existe **3 critères** pour caractériser un type de NAT :

1. **Le sens de traduction** → Quelle adresse est modifiée ?
2. **Le mode d'association** → Comment la traduction est-elle établie ?
3. **Le niveau de traduction** → Qu'est-ce qui est traduit ?

> [!info] Combinaisons possibles
> Ces critères peuvent être combinés pour former différents types de NAT.

### Sens de traduction

#### NAT source

> [!quote] Définition NAT source
> **NAT source** : L'adresse **source** est traduite, typiquement lors d'une sortie vers Internet.

**Caractéristiques** :
- Traduction de l'adresse (et éventuellement du port) du **client**
- Utilisé pour la **sortie vers Internet**
- Cas le plus classique : **PAT/NAPT**

#### NAT destination

> [!quote] Définition NAT destination
> **NAT destination** : L'adresse de **destination** est traduite, typiquement pour publier un service.

**Caractéristiques** :
- Traduction de l'adresse (et éventuellement du port) du **serveur**
- Utilisé pour la **publication de services**
- Également appelé : **DNAT** ou **Port forwarding**

### Mode d'association

#### Statique

> [!quote] Définition mode statique
> **Statique** : Association **fixe**, connue à l'avance, ne change pas.

**Usage typique** : Serveurs

**Caractéristiques** :
- L'association est **prédéfinie** dans la configuration
- Elle reste **permanente**
- Utilisée pour les équipements dont l'adresse doit être stable

#### Dynamique

> [!quote] Définition mode dynamique
> **Dynamique** : Association **temporaire**, créée à la demande, avec expiration après délai.

**Usage typique** : Clients

**Caractéristiques** :
- L'association est créée **automatiquement** lors de la première communication
- Elle est maintenue pendant la durée de la session
- Elle **expire** après un certain délai d'inactivité

### Niveau de traduction

#### Adresse IP uniquement

> [!quote] Traduction d'adresse simple
> **Adresse IP** : Traduction simple, **1 @IP interne ↔ 1 @IP externe**.

**Autres noms** :
- NAT simple
- **NAT 1:1**

**Caractéristiques** :
- Seule l'adresse IP est modifiée
- Les ports ne sont pas touchés
- Une adresse publique dédiée par machine interne

#### Adresse IP + Port

> [!quote] Traduction d'adresse et de port
> **Adresse IP + port** : Traduction plus fine permettant plusieurs flux via une même @IP publique.

**Autres noms** :
- **PAT** (Port Address Translation)
- **NAPT** (Network Address and Port Translation)

**Caractéristiques** :
- L'adresse IP **et** le port sont modifiés
- Permet de multiplexer plusieurs communications sur une seule IP publique
- Le port joue un rôle déterminant pour différencier les flux

---

## PAT/NAPT

> [!abstract] PAT/NAPT - Le NAT le plus courant
> Le PAT (Port Address Translation) ou NAPT (Network Address and Port Translation) est le type de NAT le plus utilisé au monde. Il permet à plusieurs machines d'un réseau interne de partager une seule adresse IP publique en utilisant différents ports.

### Définition PAT

> [!quote] Définition NAPT
> **NAPT** (Network Address and Port Translation) : Traduction de l'adresse IP **et** du port (en sortie et au retour).

> [!quote] Définition PAT
> **PAT** (Port Address Translation) : C'est le **port** qui joue un rôle déterminant pour différencier les communications.

**Autres noms** :
- NAT overload
- NAT masquerade (Linux / Netfilter / iptables / nftables)
- SNAT avec ports (Source Network Address Translation) - pfSense, Stormshield, Palo Alto

#### Caractéristiques techniques

Traduction **dynamique** de plusieurs flux internes (adresse IP et port) vers une seule IP publique - **RFC 2663**.

Les ports servent à différencier les communications.

**Classification selon les 3 critères** :
- **Sens de traduction** : Source
- **Mode d'association** : Dynamique
- **Niveau de traduction** : @IP + port

#### Usage

> [!important] Usage principal
> - Un des NAT les plus utilisés à cause de la pénurie d'@IP v4
> - Fournir un **accès Internet** à des machines clientes

### Exemple PAT

Prenons l'exemple d'un routeur avec une adresse publique **203.1.113.123** (IP WAN) :

**Scénario** :
- La machine interne **10.1.1.11** accède à Odyssey (**216.58.214.83**)
- Communication **HTTPS**, donc :
  - Port serveur = **443**
  - Port client dynamique = **52369**
- Requête de `10.1.1.11:52369` vers `216.58.214.83:443`

#### Table PAT du routeur

Le routeur note dans sa table PAT "interne ⇔ externe" :

| Adresse source (interne) | Port source | Adresse destination | Port destination |
|--------------------------|-------------|---------------------|------------------|
| **10.1.1.11** | 52369 | 216.58.214.83 | 443 |

**Association créée** : `10.1.1.11:52369` ⇔ `203.1.113.123:52369`

#### Traduction sortante (client → serveur)

**Requête transmise par le routeur sur Internet** :
- Avant NAT : `10.1.1.11:52369` → `216.58.214.83:443`
- Après NAT : `203.1.113.123:52369` → `216.58.214.83:443`

> [!info] Mécanisme
> Le routeur **remplace l'adresse source et le port source** avant l'envoi vers Internet.

#### Traduction entrante (serveur → client)

**Suite de l'exemple (retour)** :

Odyssey (216.58.214.83) reçoit une requête de 203.1.113.123 et répond :
- Réponse de `216.58.214.83:443` → `203.1.113.123:52369`

Le routeur reçoit cette réponse pour lui **MAIS** il fait du NAT :
- Le routeur cherche dans sa table une correspondance pour le port **52369**
- Il trouve l'association : **10.1.1.11**
- Le routeur se base sur le **port** pour identifier le destinataire interne

> [!important] Rôle du port
> Le port est l'élément déterminant qui permet d'identifier quelle machine interne doit recevoir la réponse.

Il transmet donc sur le réseau interne le paquet en remplaçant l'adresse de destination (la sienne) par **10.1.1.11**.

#### Gestion des collisions de ports

> [!warning] Collision de ports
> Si le port source était déjà utilisé dans la table de correspondance par une autre machine interne (collision de ports), le routeur choisit un autre port source disponible et modifie également le port.

**Exemple** :
- Si le port 52369 est déjà utilisé
- Le routeur choisit un autre port disponible (ex : **52370**)
- Association : `10.1.1.11:52369` ⇔ `203.1.113.123:52370`

C'est aussi pour cette raison que l'on parle de **PAT** (Port Address Translation).

### Inconvénients PAT

> [!warning] Limitations du PAT/NAPT

#### Limitation client/serveur

- La table étant remplie lors de la requête de l'intérieur vers l'extérieur :
  - Le **client** doit être derrière le NAT
  - **Impossible d'avoir un serveur interne** (sans DNAT ou port forwarding)

#### Complexité technique

Le paquet IP (L3) et le protocole de niveau 4 (L4) sont modifiés :
- **Lourd** : nécessite un recalcul des checksum IP et TCP/UDP
- Pas possible avec tous les protocoles de couche 4
- Incompatible avec certains protocoles (ex : **FTP actif**)
- Incompatible avec certains contrôles d'intégrité (signatures numériques)

#### Principe architectural

> [!warning] Rupture du principe de bout en bout
> Le NAT **tord le principe de bout en bout** (end-to-end principle) qui est un des principes fondateurs d'Internet. Ce principe stipule que les fonctionnalités doivent être implémentées aux extrémités du réseau, et non dans les nœuds intermédiaires.

---

## DNAT

> [!abstract] DNAT - Publication de services
> Le DNAT (Destination Network Address Translation) permet de publier des services internes vers l'extérieur en traduisant l'adresse de destination des paquets entrants.

### Définition DNAT

> [!quote] Définition DNAT
> **DNAT** (Destination Network Address Translation) : Traduction de l'adresse IP et du port de destination (en entrée et au retour).

**Autres noms** :
- **Static DNAT** (sans changement de port)
- **Port forwarding** (avec changement de port)

#### Caractéristiques techniques

Traduction **statique** de l'adresse IP de destination (et éventuellement du port destination) pour rediriger une communication entrante vers une machine interne.

**Classification selon les 3 critères** :
- **Sens de traduction** : Destination
- **Mode d'association** : Statique
- **Niveau de traduction** : @IP (+ éventuellement port)

#### Usage

> [!important] Usage principal
> - Utilisé uniquement lorsque des services sont exposés
> - **Publication de services internes** (HTTP, HTTPS, RDP, FTP, etc.)

### Exemple DNAT

Prenons l'exemple d'un routeur avec une adresse publique **203.1.113.123** (IP WAN) :

**Scénario** :
- Une machine externe **204.1.97.10** veut se connecter à un serveur sur un réseau interne (**172.16.1.15**)
- Communication **HTTP** :
  - Port serveur (destination) = **80**
  - Port client dynamique = **57221**
- Requête entrante de `204.1.97.10:57221` vers `203.1.113.123:80`

#### Table DNAT du routeur

Le routeur note dans sa table DNAT "interne ⇔ externe" :

| Adresse destination (interne) | Port destination | Adresse source | Port source |
|-------------------------------|------------------|----------------|-------------|
| **172.16.1.15** | 80 | 204.1.97.10 | 57221 |

**Association statique configurée** : `172.16.1.15:80` ⇔ `203.1.113.123:80`

#### Traduction entrante (client → serveur)

**Requête reçue par le routeur** :
`204.1.97.10:57221` → `203.1.113.123:80`

Le routeur effectue une **traduction de destination** :
- Adresse de destination : `203.1.113.123` → `172.16.1.15`
- Port de destination : `80` → `80` (conservé)

**Requête transmise en interne** :
`204.1.97.10:57221` → `172.16.1.15:80`

#### Traduction sortante (serveur → client)

**Pour le retour**, le serveur interne 172.16.1.15 répond :
- Réponse de `172.16.1.15:80` → `204.1.97.10:57221`

Le routeur reçoit la réponse et fait une **traduction inverse DNAT** :
- Adresse source remplacée : `172.16.1.15` → `203.1.113.123`
- Port source conservé : `80`

**Réponse envoyée vers l'extérieur** :
`203.1.113.123:80` → `204.1.97.10:57221`

#### Publication de plusieurs services

Le DNAT peut :
- Traduire uniquement l'adresse IP
- Ou traduire adresse IP + port

> [!tip] Mutualisation d'IP publique
> Plusieurs services peuvent être publiés sur une même IP publique à condition d'utiliser des ports différents.

**Exemple** : Un serveur qui a 2 rôles RDP (3389) et HTTP (80)

| Adresse destination (interne) | Port destination | Adresse publique | Port publique |
|-------------------------------|------------------|------------------|---------------|
| **172.16.1.15** | 80 | 203.1.113.123 | 80 |
| **172.16.1.15** | 3389 | 203.1.113.123 | 3389 |

### Inconvénients DNAT

> [!warning] Limitations et risques du DNAT

#### Exposition de services

- Expose des services internes vers l'extérieur
- Nécessite :
  - **Règles de pare-feu strictes**
  - **Durcissement des serveurs** (hardening)
  - **Surveillance et monitoring** des accès

#### Complexité technique

- **Rupture du principe de bout en bout**
- Peut poser des problèmes avec certains protocoles applicatifs

> [!important] Sécurité
> L'exposition de services via DNAT augmente la surface d'attaque de ton infrastructure. Une configuration rigoureuse du pare-feu et une surveillance constante sont indispensables.

---

## NAT 1:1

> [!abstract] NAT 1:1 - Association statique complète
> Le NAT 1:1 (One-to-One NAT) permet d'attribuer une adresse IP publique dédiée à un serveur interne de manière permanente, dans les deux sens de communication.

### Définition NAT 1:1

> [!quote] Définition NAT 1:1
> **NAT 1:1** (One-to-One Network Address Translation) : Traduction **statique** d'une adresse IP privée vers une adresse IP publique dédiée, dans les deux sens (aller et retour).

**Autre nom** :
- **Static NAT**

#### Caractéristiques techniques

Permet d'attribuer une **adresse IP publique dédiée** à un serveur interne.

**Classification selon les 3 critères** :
- **Sens de traduction** : Source ET Destination (bidirectionnel)
- **Mode d'association** : Statique
- **Niveau de traduction** : @IP uniquement

#### Usage

> [!important] Usage principal
> - Rendre un serveur joignable sur Internet
> - Concrètement : serveur en **DMZ** (Zone Démilitarisée)

### Exemple NAT 1:1

**Exemple** :
- @IP serveur interne : **172.16.1.20**
- @IP publique attribuée : **203.1.113.50** (IP WAN)
- Services hébergés : HTTP (80), HTTPS (443), RDP (3389)

#### Configuration NAT 1:1

| Adresse interne | Adresse externe (publique) |
|----------------|----------------------------|
| **172.16.1.20** | **203.1.113.50** |

**NAT 1:1 configuré sur le routeur** : `172.16.1.20` ⇔ `203.1.113.50`

#### Traduction sortante (serveur → Internet)

**Requête transmise vers le routeur** :
`172.16.1.20:443` → `203.1.113.123:443`

Les **ports ne changent pas**.

L'adresse source est remplacée : `172.16.1.20` → `203.1.113.50`

**Requête transmise par le routeur sur Internet** :
`203.1.113.50:443` → `216.58.214.83:443`

| Adresse source (interne) | Port source | Adresse publique dédiée | Adresse destination | Port destination |
|--------------------------|-------------|-------------------------|---------------------|------------------|
| **172.16.1.20** | 443 | **203.1.113.50** | 216.58.214.83 | 443 |

#### Traduction entrante (Internet → serveur)

**Réponse reçue sur le routeur** :
`216.58.214.83:443` → `203.1.113.50:443`

Les **ports ne changent pas**.

**Traduction inverse** : `203.1.113.50` → `172.16.1.20`

**Requête transmise vers le serveur interne** :
`216.58.214.83:443` → `172.16.1.20:443`

### Particularité

> [!success] Avantage du NAT 1:1
> Tous les ports sont accessibles sans configuration supplémentaire :
> - HTTP (80)
> - HTTPS (443)
> - RDP (3389)
> - Etc.

**Le port forwarding n'est pas nécessaire.**

**Pas de multiplexage de ports** : une IP publique = une machine interne.

### Inconvénients NAT 1:1

> [!warning] Limitations du NAT 1:1

#### Consommation d'adresses

- **Consommation d'une @IP publique par machine**
- Coûteux en adresses publiques (ressource rare)

#### Exposition et sécurité

- Expose directement les serveurs
- Nécessite un **pare-feu strict**
- **Durcissement des serveurs** obligatoire

#### Complexité technique

- **Rupture du principe de bout en bout**

> [!important] Quand utiliser NAT 1:1 ?
> Le NAT 1:1 est adapté pour les serveurs en DMZ nécessitant de multiples services accessibles sur leurs ports standards. Pour des besoins plus limités, le DNAT/port forwarding est plus économe en adresses publiques.

---

## Aller plus loin

### Serveur derrière NAT

Pour héberger un serveur derrière un NAT, il faut mettre en place une **correspondance statique**.

Ce genre de correspondance est un **port forwarding** (DNAT).

Elle consiste à :
1. Déclarer un **port** sur le routeur NAT
2. Lui associer une **adresse interne** (et éventuellement un port)

> [!example] Exemple : Serveur web
> Pour un serveur web, les ports **80 (TCP)** et **443 (TCP)** doivent être redirigés vers l'adresse interne du serveur.

> [!warning] Limite : Plusieurs serveurs du même type
> Dans le cas de plusieurs serveurs pour le même service, seul un d'entre eux pourra utiliser le port standard.
> 
> **Solution** : Utiliser des ports alternatifs (ex : 8080 pour HTTP, 8443 pour HTTPS).

### Depuis l'extérieur

L'utilisation de NAT implique qu'une adresse IP est utilisée par **plusieurs interfaces** de manière transparente.

#### Problématique du blocage d'IP

Dans le cas où un serveur (ou un équipement réseau) considère qu'un trafic réseau est abusif :
- Surconsommation de bande passante
- Spam
- Comportement suspect

Il est fréquent qu'il réagisse en **bloquant l'adresse**.

> [!warning] Effet de bord du NAT
> **Problème** : Plusieurs utilisateurs viennent d'être bloqués d'un coup, y compris ceux qui étaient légitimes.
> 
> Le NAT masque l'identité individuelle des machines du réseau interne, rendant impossible la discrimination entre utilisateurs légitimes et malveillants.

### Traverser des NAT

Pour permettre la traversée de NAT à certains protocoles incompatibles ou éviter d'exiger des configurations réseau aux particuliers, plusieurs techniques ont été mises au point.

> [!info] NAT Traversal
> Une première approche de ces techniques peut être consultée sur la page [NAT traversal](https://en.wikipedia.org/wiki/NAT_traversal) sur Wikipedia (🇬🇧).

**Techniques courantes** :
- **UPnP** (Universal Plug and Play) - Automatisation de la configuration NAT
- **STUN** (Session Traversal Utilities for NAT) - Découverte d'adresse publique
- **TURN** (Traversal Using Relays around NAT) - Relais de données
- **ICE** (Interactive Connectivity Establishment) - Combinaison de techniques

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **NAT** = Network Address Translation = Traduction d'adresses IP
- Mécanisme **quasi-exclusif à l'IPv4**
- Solution **transitoire** à la pénurie d'adresses IPv4
- Implémenté sur les **routeurs** et **pare-feu**
- Le NAT est **indispensable** pour continuer à utiliser IPv4

### Adressage

- **Adresses privées (RFC 1918)** : 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- Ces adresses sont **non-routables** sur Internet
- **Adresses publiques** : attribuées par ICANN → RIR (RIPE NCC) → LIR
- IPv4 fournit ~**4,3 milliards d'adresses** (ressource épuisée)

### Les 3 critères de classification

1. **Sens de traduction** : Source, Destination, ou les deux
2. **Mode d'association** : Statique (serveurs) ou Dynamique (clients)
3. **Niveau de traduction** : @IP seule ou @IP + port

### PAT/NAPT (NAT le plus courant)

- **Type** : NAT Source + Dynamique + @IP+Port
- **Usage** : Accès Internet pour les clients
- **Principe** : Plusieurs machines partagent 1 IP publique via différents ports
- **Autres noms** : NAT overload, NAT masquerade, SNAT avec ports
- **Limitation** : Client obligatoirement derrière le NAT, impossible d'héberger un serveur

### DNAT (Port Forwarding)

- **Type** : NAT Destination + Statique + @IP+Port
- **Usage** : Publication de services internes vers Internet
- **Principe** : Redirection des connexions entrantes vers un serveur interne
- **Autres noms** : Static DNAT, Port forwarding
- **Limitation** : Exposition de services, nécessite pare-feu strict

### NAT 1:1

- **Type** : NAT Source+Destination + Statique + @IP seule
- **Usage** : Serveur en DMZ avec IP publique dédiée
- **Principe** : 1 IP interne ⇔ 1 IP publique (tous les ports disponibles)
- **Autre nom** : Static NAT
- **Limitation** : Consomme 1 IP publique par machine

### Avantages du NAT

- Permet de continuer à utiliser IPv4 malgré la pénurie d'adresses
- Masque le plan d'adressage interne (sécurité par l'obscurité)
- Permet le partage d'une IP publique entre plusieurs machines
- Économise les adresses IPv4 publiques

### Inconvénients du NAT

- **Rupture du principe de bout en bout** (end-to-end)
- Incompatible avec certains protocoles (FTP actif, etc.)
- Complexité technique (recalcul checksums)
- Problèmes pour héberger des serveurs (nécessite DNAT)
- Blocages d'IP affectent plusieurs utilisateurs
- Nécessite des techniques de NAT traversal pour certaines applications

### NAT et IPv6

- Le NAT n'est **plus indispensable** avec IPv6
- IPv6 propose suffisamment d'adresses pour tous
- Le NAT existe encore en IPv6 (**NPTv6**) mais pour d'autres raisons
- L'adoption d'IPv6 réduira progressivement le besoin de NAT

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **NAT** | Network Address Translation - Mécanisme de traduction d'adresses IP entre réseaux privés et publics |
| **PAT** | Port Address Translation - Traduction incluant les ports, permettant le multiplexage |
| **NAPT** | Network Address and Port Translation - Synonyme de PAT |
| **DNAT** | Destination Network Address Translation - Traduction de l'adresse de destination |
| **SNAT** | Source Network Address Translation - Traduction de l'adresse source |
| **Port Forwarding** | Redirection de port - Technique pour publier un service interne |
| **Static NAT** | NAT avec association statique prédéfinie (NAT 1:1) |
| **Dynamic NAT** | NAT avec association temporaire créée à la demande (PAT/NAPT) |
| **NAT Overload** | Autre nom pour PAT/NAPT quand plusieurs machines partagent une IP |
| **Masquerade** | Terme Linux pour le NAT dynamique (masquage d'adresse) |
| **RFC 1918** | Norme définissant les plages d'adresses IP privées |
| **RFC 3022** | Norme définissant le NAT traditionnel |
| **RFC 2663** | Norme définissant la terminologie et les considérations du NAT |
| **Adresse privée** | Adresse IP non-routable sur Internet (10.x.x.x, 172.16-31.x.x, 192.168.x.x) |
| **Adresse publique** | Adresse IP routable sur Internet, attribuée par les RIR |
| **DMZ** | Demilitarized Zone - Zone réseau intermédiaire entre LAN et WAN pour les serveurs exposés |
| **LAN** | Local Area Network - Réseau local privé |
| **WAN** | Wide Area Network - Réseau étendu public (Internet) |
| **IP WAN** | Adresse IP publique d'un routeur côté Internet |
| **Checksum** | Somme de contrôle permettant de vérifier l'intégrité d'un paquet |
| **ICANN** | Internet Corporation for Assigned Names and Numbers - Organisme gérant les adresses IP mondiales |
| **RIR** | Regional Internet Registry - Registre régional d'attribution d'adresses (ex : RIPE NCC pour l'Europe) |
| **LIR** | Local Internet Registry - Fournisseur d'accès Internet distribuant les adresses |
| **RIPE NCC** | Réseaux IP Européens Network Coordination Centre - RIR pour l'Europe |
| **CGN** | Carrier-Grade NAT - NAT déployé au niveau des opérateurs télécom |
| **NPTv6** | Network Prefix Translation for IPv6 - Équivalent du NAT pour IPv6 |
| **NAT Traversal** | Ensemble de techniques permettant de traverser un NAT |
| **UPnP** | Universal Plug and Play - Protocole d'autoconfiguration des équipements réseau (dont NAT) |
| **STUN** | Session Traversal Utilities for NAT - Protocole de découverte d'IP publique |
| **TURN** | Traversal Using Relays around NAT - Protocole de relais pour contourner le NAT |
| **ICE** | Interactive Connectivity Establishment - Combinaison de techniques NAT traversal |
| **Table NAT** | Structure de données stockant les correspondances d'adresses/ports |
| **Session** | Communication établie entre deux machines, identifiée par IP source/dest et ports |
| **RSID** | Revision Save ID - Identifiant unique dans les documents Office pour le suivi des modifications |

---

> [!tip] Pour aller plus loin
> - Consulte la [RFC 3022](https://www.rfc-editor.org/rfc/rfc3022.html) pour les détails techniques du NAT
> - Étudie la [RFC 2663](https://www.rfc-editor.org/rfc/rfc2663.html) pour la terminologie officielle
> - Pratique la configuration NAT sur pfSense, iptables/nftables, ou des équipements Cisco
> - Analyse les tables NAT avec `conntrack` (Linux) ou `netstat` pour comprendre le fonctionnement

---

**📚 Document créé selon le format standardisé TSSR - Janvier 2025**
