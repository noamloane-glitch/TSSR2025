# Internet Protocol version 6 (IPv6)
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Internet Protocol version 6 - Adresses et paquets  
**Date** : Décembre 2024  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction]]
2. [[#Les adresses IPv6]]
3. [[#Autoconfiguration]]
4. [[#Le paquet IPv6]]
5. [[#Protocoles associés]]
6. [[#Synthèse IPv4 vs IPv6]]
7. [[#Points clés à retenir]]
8. [[#Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> IPv6 résout le problème d'épuisement des adresses IPv4 avec des adresses sur 128 bits et apporte de nombreuses améliorations.

### Pourquoi IPv6 ?

**Limites d'IPv4 :**
- 4,3 milliards d'adresses insuffisantes
- Explosion : smartphones, IoT, cloud, 4G/5G
- NAT = solution temporaire mais ajoute complexité

**IPv6 rétablit le principe end-to-end d'Internet**

---

### Définition

> [!quote] IPv6
> Nouvelle version du protocole IP avec adresses sur **128 bits**, standardisée par l'IETF.

**Caractéristiques :**
- Successeur d'IPv4
- Compatible avec protocoles modernes (DNS, HTTP, etc.)
- Coexiste avec IPv4 (dual-stack)

---

### Objectifs principaux

1. **Étendre l'adressage** (32→128 bits)
2. **Simplifier les en-têtes**
3. **Automatiser la configuration** (SLAAC)
4. **Renforcer la sécurité** (IPsec intégré)
5. **Réduire la fragmentation** (PMTUd)
6. **Optimiser le routage**

---

### Déploiement actuel (2025)

- **~50% du trafic Google** en IPv6
- **85% d'adoption en France**
- Tous les OS modernes supportent IPv6
- Tous les FAI français attribuent IPv4 + IPv6

---

## Les adresses IPv6

> [!abstract] Adresses sur 128 bits
> ≈ 3,4 × 10³⁸ adresses disponibles

### Structure

**3 catégories d'adresses :**
- **Unicast** : une interface
- **Multicast** : groupe d'interfaces  
- **Anycast** : une parmi un groupe

> [!warning] Pas de broadcast en IPv6 !

---

### Notation hexadécimale

**Format :** 8 groupes de 16 bits séparés par `:`

**Exemple :**
```
2001:0db8:0000:85a3:0000:0000:ac1f:8001
```

**Simplifications (RFC 5952) :**
1. Omission des zéros non significatifs
2. Compression d'une suite de zéros avec `::`

**Résultat :**
```
2001:db8:0:85a3::ac1f:8001
```

> [!warning] Le `::` ne peut apparaître qu'UNE SEULE FOIS

---

### Le préfixe réseau

**Notation CIDR :** `adresse/longueur_préfixe`

**Exemple :** `2001:db8::1/64`

> [!important] Standard /64
> - 64 bits pour le préfixe réseau
> - 64 bits pour l'identifiant d'interface

---

### Adresses particulières

| Préfixe | Type | Usage | IPv4 équivalent |
|---------|------|-------|-----------------|
| `::1/128` | Loopback | Boucle locale | 127.0.0.1 |
| `::` | Indéfinie | Non spécifiée | 0.0.0.0 |
| `ff00::/8` | Multicast | Groupes | 224.0.0.0/4 |
| `fe80::/10` | Link-Local | Lien local **OBLIGATOIRE** | 169.254.0.0/16 |
| `fc00::/7` | Unique Local | Privées | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 |
| `2000::/3` | Global Unicast | Publiques | Adresses publiques |

---

#### Adresse Link-Local

> [!important] fe80::/64
> **Obligatoire sur toute interface IPv6** pour le fonctionnement du protocole.

**Structure :**
```
fe80::/64 + identifiant d'interface (64 bits)
```

---

#### Adresse Unique Local

> [!info] fc00::/7 (en pratique fd00::/8)
> Adresses privées IPv6, routables localement mais pas sur Internet.

**Structure :**
```
fd + Global ID aléatoire (40 bits) + Subnet (16 bits) + Interface ID (64 bits)
```

---

#### Adresse Global Unicast

> [!info] 2000::/3 (en pratique 2001::/16)
> Adresses publiques routables sur Internet.

**Hiérarchie d'attribution :**
```
IANA → RIR (ex: RIPE NCC) → LIR (FAI) → Site → Abonné
```

---

### Politique d'attribution

**RIPE NCC (Europe) :**
- Allocation minimum : /32
- Sans justification : /29 à /32
- Plus de /29 si justifié

**IETF RFC 6177 :**
- Un site = plusieurs /64
- Attribution /48 non systématique

---

## Autoconfiguration

> [!abstract] SLAAC
> StateLess Address AutoConfiguration = configuration automatique sans serveur

### Objectifs

- **Configuration manuelle optionnelle**
- **DHCP optionnel** (DHCPv6)
- **Re-numérotation facile**
- **Adresses multiples** par interface
- **Plug-and-Play**

---

### Identifiant d'interface

**3 méthodes de génération (64 bits) :**

| Méthode | RFC | Avantage | Inconvénient |
|---------|-----|----------|--------------|
| Tirage au sort | RFC 8981 | Vie privée | Traçabilité difficile |
| EUI-64 (MAC) | RFC 4291 | Stable | Vie privée compromise |
| Cryptographique (CGA) | RFC 3972 | Sécurité forte | Complexe, rare |

> [!tip] Recommandé
> **Privacy Extensions** (tirage au sort) pour les postes clients

---

### Préfixe réseau

**Pour Link-Local :**
- Préfixe connu : `fe80::/64`
- Détection d'adresse dupliquée (DAD)

**Pour autres adresses :**
- Router Advertisement (RA) des routeurs
- Contient : préfixe(s), durée de vie, paramètres
- Multicast : `ff02::1` (all-nodes) ou `ff02::2` (all-routers)

---

### Portée et durée de vie

**Portées (Scope) :**
- Interface-Local (hôte) : ::1
- Link-Local : fe80::/10
- Global : 2000::/3

**États d'une adresse :**
```
Provisoire → Préférée → Dépréciée → Invalide
```

**Permet le renumbering sans interruption de service**

---

## Le paquet IPv6

> [!important] En-tête fixe de 40 octets
> Plus simple et plus efficace qu'IPv4

### Structure de l'en-tête

```
Version (4) | Traffic Class (8) | Flow Label (20)
Payload Length (16) | Next Header (8) | Hop Limit (8)
Source Address (128 bits)
Destination Address (128 bits)
```

**Champs principaux :**
- **Version** = 6
- **Traffic Class** = QoS
- **Flow Label** = identification de flux ⚡ NOUVEAUTÉ
- **Payload Length** = taille charge utile
- **Next Header** = protocole suivant
- **Hop Limit** = TTL

---

### Simplifications vs IPv4

> [!success] Supprimé de l'en-tête
> - Header Length (en-tête fixe)
> - Identification/Flags/Fragment Offset (fragmentation différente)
> - **Header Checksum** (améliore performances)
> - Options (remplacé par en-têtes d'extension)

---

### Fragmentation

> [!important] Changement majeur
> Les routeurs **NE FRAGMENTENT JAMAIS** en IPv6

**PMTUd (Path MTU Discovery) :**
1. Émetteur envoie paquet
2. Routeur drop si trop grand
3. Routeur envoie ICMPv6 "Packet Too Big" + MTU
4. Émetteur ajuste la taille

> [!warning] MTU minimale IPv6 = 1280 octets

---

## Protocoles associés

### ICMPv6

> [!quote] RFC 4443
> Regroupe ICMP, ARP et IGMP d'IPv4

**Next Header = 58**

**Messages principaux :**
- Type 1 : Destination Unreachable
- Type 2 : Packet Too Big (PMTUd)
- Type 3 : Time Exceeded
- Type 128/129 : Echo Request/Reply (ping)
- Type 133/134 : RS/RA (Router Solicitation/Advertisement)
- Type 135/136 : NS/NA (Neighbor Solicitation/Advertisement)

---

#### NDP (Neighbor Discovery Protocol)

> [!important] Remplace ARP
> Utilise ICMPv6 pour découverte de voisins

**Fonctions :**
1. Router Discovery (RS/RA)
2. Neighbor Discovery (NS/NA) = ex-ARP
3. Détection d'adresses dupliquées (DAD)
4. Redirection

---

### DHCPv6

> [!info] Optionnel grâce à SLAAC

**Ports :** UDP 546 (client) / 547 (serveur)  
**Multicast :** `ff02::1:2`

**Modes :**
- **Stateful** : adresses + paramètres
- **Stateless** : paramètres uniquement (adresses par SLAAC)

**Flags dans RA :**
- M=1 : DHCPv6 pour adresses
- O=1 : DHCPv6 pour paramètres

---

### IPsec

> [!info] Intégré nativement (mais optionnel)

**Composants :**
- **AH** (Authentication Header) : authentification + intégrité
- **ESP** (Encapsulating Security Payload) : + confidentialité
- **IKE** : échange de clés

**Modes :**
- Transport : sécurise charge utile
- Tunnel : encapsule paquet complet (VPN)

---

### Mobile IPv6 (MIPv6)

> [!info] RFC 6275
> Maintien des connexions lors de changements de réseau

**Composants :**
- **Home Address** : permanente
- **Care-of Address** : temporaire sur réseau actuel
- **Home Agent** : intercepte et redirige via tunnel

> [!warning] Peu déployé en pratique

---

## Synthèse IPv4 vs IPv6

| Caractéristique | IPv4 | IPv6 |
|-----------------|------|------|
| **Adresses** | 32 bits (4,3 × 10⁹) | 128 bits (3,4 × 10³⁸) |
| **Notation** | Décimal pointé | Hexadécimal `:` |
| **En-tête** | Variable 20-60 octets | Fixe 40 octets |
| **Configuration** | DHCP obligatoire | SLAAC automatique |
| **Broadcast** | Oui | **Non** (multicast) |
| **Fragmentation** | Par routeurs | Par émetteur uniquement |
| **Checksum** | Oui | **Non** |
| **NAT** | Indispensable | Inutile |
| **ARP** | Protocole séparé | ICMPv6 (NDP) |

---

## Points clés à retenir

> [!success] Pour le titre RNCP

### Adresses essentielles

| Préfixe | Usage |
|---------|-------|
| `::1/128` | Loopback |
| `fe80::/10` | Link-Local **OBLIGATOIRE** |
| `fd00::/8` | Unique Local (privé) |
| `2001::/16` | Global Unicast (public) |
| `ff00::/8` | Multicast |

### Concepts clés

- **128 bits** = 3,4 × 10³⁸ adresses
- **Notation hexa** : 8 groupes séparés par `:`
- **Simplification** : omission zéros, compression `::`
- **Pas de broadcast** → multicast
- **SLAAC** = autoconfiguration automatique
- **En-tête fixe** 40 octets
- **PMTUd** : routeurs ne fragmentent pas
- **ICMPv6** regroupe ICMP+ARP+IGMP
- **DHCPv6** optionnel
- **IPsec** intégré nativement

---

## Glossaire technique

| Terme | Définition |
|-------|------------|
| **Anycast** | Adresse de groupe, le paquet va à la plus proche |
| **CIDR** | Notation adresse/préfixe (ex: 2001:db8::/32) |
| **DAD** | Duplicate Address Detection - détection doublons |
| **DHCPv6** | DHCP pour IPv6, optionnel grâce à SLAAC |
| **Dual-stack** | Support simultané IPv4 + IPv6 |
| **EUI-64** | Méthode génération ID interface depuis MAC |
| **Flow Label** | Étiquette de flux pour traitement spécifique (nouveauté IPv6) |
| **Global Unicast** | Adresses publiques routables (2000::/3) |
| **Hop Limit** | Nombre max de sauts (= TTL IPv4) |
| **IANA** | Internet Assigned Numbers Authority |
| **ICMPv6** | Protocole contrôle IPv6 (Next Header 58) |
| **IETF** | Internet Engineering Task Force |
| **IPsec** | Suite protocoles sécurité IP |
| **Link-Local** | Adresse non routable locale (fe80::/10), OBLIGATOIRE |
| **MIPv6** | Mobile IPv6 - maintien connexions lors déplacements |
| **MTU** | Maximum Transmission Unit (min 1280 en IPv6) |
| **Multicast** | Diffusion à un groupe (ff00::/8) |
| **NA** | Neighbor Advertisement (Type 136) |
| **NAT** | Network Address Translation - inutile en IPv6 |
| **NDP** | Neighbor Discovery Protocol - remplace ARP |
| **Next Header** | Type prochain en-tête (= Protocol IPv4) |
| **NS** | Neighbor Solicitation (Type 135) - ex-ARP Request |
| **PMTUd** | Path MTU Discovery - découverte MTU minimale |
| **Privacy Extensions** | Génération aléatoire ID interface (RFC 8981) |
| **RA** | Router Advertisement (Type 134) |
| **RFC** | Request For Comments - standards IETF |
| **RIR** | Regional Internet Registry (ex: RIPE NCC) |
| **RIPE NCC** | RIR pour Europe, Moyen-Orient, Asie centrale |
| **RS** | Router Solicitation (Type 133) |
| **SLAAC** | StateLess Address AutoConfiguration |
| **Traffic Class** | Champ QoS (= ToS IPv4) |
| **Unicast** | Communication point-à-point |
| **Unique Local** | Adresses privées (fc00::/7, pratique fd00::/8) |

---

## Conclusion

> [!success] IPv6 : protocole du présent et du futur

**En tant que TSSR, maîtriser IPv6 est indispensable pour :**
- Administrer les réseaux modernes
- Comprendre les architectures dual-stack
- Diagnostiquer les problèmes de connectivité
- Implémenter des solutions sécurisées

**Ressources officielles :**
- IETF IPv6 : https://www.ietf.org/topics/ipv6/
- RIPE NCC IPv6 : https://www.ripe.net/publications/ipv6-info-centre
- Test IPv6 : https://test-ipv6.com

**RFCs importantes :**
- RFC 4291 : Architecture d'adressage
- RFC 4862 : SLAAC
- RFC 4861 : Neighbor Discovery (NDP)
- RFC 8200 : Spécification IPv6
- RFC 8201 : PMTUd

---

**Bon courage pour ton titre RNCP TSSR ! 🚀📚**

