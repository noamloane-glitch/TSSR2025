# WIFI - Réseaux Sans Fil 802.11
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)
**Sujet** : Réseaux WiFi - Normes IEEE 802.11
**Date** : Mars 2026
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Les normes WiFi|Les normes WiFi]]
   - [[#Définition et historique|Définition et historique]]
   - [[#Architecture en couches IEEE|Architecture en couches IEEE]]
   - [[#Objectifs de 802.11|Objectifs de 802.11]]
3. [[#Les normes physiques|Les normes physiques]]
   - [[#Medium hertzien|Medium hertzien]]
   - [[#Régulation des fréquences|Régulation des fréquences]]
   - [[#Interférences et canaux|Interférences et canaux]]
   - [[#Agrégation de canaux et MIMO|Agrégation de canaux et MIMO]]
   - [[#Les antennes|Les antennes]]
   - [[#Tableau récapitulatif des normes|Tableau récapitulatif des normes]]
4. [[#Les modes opératoires|Les modes opératoires]]
   - [[#Mode ad hoc (IBSS)|Mode ad hoc (IBSS)]]
   - [[#Mode infrastructure BSS|Mode infrastructure BSS]]
   - [[#Mode infrastructure étendu ESS|Mode infrastructure étendu ESS]]
   - [[#Réseau mesh|Réseau mesh]]
5. [[#La couche liaison|La couche liaison]]
   - [[#Adresses MAC|Adresses MAC]]
   - [[#Sous-couche LLC|Sous-couche LLC]]
   - [[#CSMA/CA - Accès au médium|CSMA/CA - Accès au médium]]
   - [[#RTS et CTS|RTS et CTS]]
   - [[#PCF et HCF|PCF et HCF]]
6. [[#La sécurité|La sécurité]]
   - [[#Protocoles obsolètes|Protocoles obsolètes]]
   - [[#Protocoles recommandés|Protocoles recommandés]]
   - [[#WPS et recommandations ANSSI|WPS et recommandations ANSSI]]
   - [[#Failles récentes|Failles récentes]]
7. [[#Points clés à retenir|Points clés à retenir]]
8. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le **WiFi** est une technologie de réseau local sans fil basée sur les normes **IEEE 802.11**. Apparu en 1997, il offre des débits allant de 11 Mbps à plus de 10 Gbps selon la génération. Ce cours couvre les normes physiques, les modes de déploiement, le fonctionnement de la couche liaison et les protocoles de sécurité.

### Pourquoi maîtriser le WiFi en tant que TSSR ?

En tant que **TSSR**, tu seras amené à :
- Déployer et configurer des réseaux sans fil en entreprise
- Choisir la bonne norme et le bon mode selon le contexte
- Sécuriser les accès WiFi (WPA2/WPA3, RADIUS)
- Diagnostiquer les problèmes d'interférences et de performance

---

## Les normes WiFi

### Définition et historique

> [!quote] Définition officielle
> **WiFi** est une **marque et un label** pour les équipements suivant les normes **IEEE 802.11**. Ce n'est pas un acronyme — c'est un nom commercial déposé.

> [!info] Historique
> - Apparu en **1997**
> - Débits : de **11 Mbps** (802.11b, 1999) jusqu'à **10,5 Gbps** (802.11ax, 2021)
> - Les normes 802.11 représentent l'approche IEEE pour les **réseaux locaux sans fil (WLAN)**

### Architecture en couches IEEE

> [!important] Modèle en couches spécifique au WiFi
> L'architecture IEEE 802.11 reprend le modèle OSI avec des particularités :

```
┌─────────────────────────────────┐
│     Couche 3 et supérieures     │
├─────────────────────────────────┤
│  LLC (Logical Link Control)     │  ← Couche 2 - COMMUNE à tous IEEE
├─────────────────────────────────┤
│  MAC (Medium Access Control)    │  ← Couche 2 - SPÉCIFIQUE à chaque protocole
├─────────────────────────────────┤
│  PHY (Physique)                 │  ← Couche 1
└─────────────────────────────────┘
```

> [!warning] Différence clé avec Ethernet (802.3)
> Contrairement à Ethernet, **la couche LLC est présente et nécessaire** en WiFi à cause de la moindre fiabilité du support physique sans fil.

### Objectifs de 802.11

> [!note] Les 7 objectifs fondamentaux d'IEEE 802.11
> 1. **Fonctionnement** avec et sans infrastructure
> 2. **Gestion de réseaux multiples** et des interférences
> 3. **DFS** (Dynamic Frequency Selection) - gestion dynamique des fréquences
> 4. **TPC** (Transmit Power Control) - gestion de la puissance d'émission
> 5. **QoS** pour le transport voix/vidéo
> 6. **Gestion de la mobilité** (roaming)
> 7. **Sécurité**

---

## Les normes physiques

### Medium hertzien

> [!info] Caractéristiques du médium radio vs câble
> Le WiFi utilise des **ondes radio** — un support très différent du câble :
> - **Pas de frontières** physiques (le signal traverse les murs)
> - **Interférences** d'autres signaux (WiFi, Bluetooth, RFID, micro-ondes...)
> - **Fiabilité nettement inférieure** au câble
> - **Topologie dynamique** (les stations se déplacent)
> - **Pas de connectivité globale** garantie
> - **Propriétés variables** au cours du temps

> [!tip] Anecdote historique
> Il a existé une norme WiFi sur **infrarouge** à 1 ou 2 Mbps — aujourd'hui abandonnée.

### Régulation des fréquences

> [!important] Organismes de régulation
> L'utilisation des fréquences radio est **réglementée** :
> - **France** : **ANFR** (Agence Nationale des Fréquences) + **ARCEP** (Autorité de régulation des communications électroniques, des postes et de la distribution de la presse)
> - **USA** : FCC

> [!note] Bandes ISM (Industrial, Scientific and Medical)
> Ce sont des **bandes de fréquences libres** — aucune licence de radiocommunication nécessaire :

| Bande | Zone | Statut |
|-------|------|--------|
| 890-915 MHz | Europe | Utilisée par GSM (indisponible WiFi) |
| 902-928 MHz | USA | Disponible |
| **2,4 - 2,835 GHz** | **Mondiale** | **Disponible WiFi** |
| **5,725 - 5,85 GHz** | **Mondiale** | **Disponible WiFi** |
| **6 GHz** | **Europe** | **Autorisée depuis 2021** |

### Interférences et canaux

> [!warning] Sources d'interférences WiFi
> Le WiFi 2,4 GHz subit des interférences avec :
> - Autres réseaux WiFi
> - **Bluetooth**
> - **RFID**
> - Périphériques sans fil (micros, enceintes, télécommandes)
> - **Fours à micro-ondes** (même bande de fréquence !)

> [!info] Découpage en canaux - Bande 2,4 GHz (802.11b)
> - **14 canaux** de 22 MHz espacés de **5 MHz** (avec recouvrements)
> - Seulement **3 canaux utilisables simultanément** sans interférence
> - Canaux recommandés : **1, 6, 11**
> - Le canal 14 est légal **uniquement au Japon**

> [!example] Déploiement multi-AP sans interférence
> Pour déployer plusieurs points d'accès dans une même zone, utiliser les canaux **1, 6 et 11** en bande 2,4 GHz afin d'éviter les chevauchements.

> [!info] Bande 5 GHz
> - **22 canaux de 20 MHz** utilisables simultanément en Europe sans condition
> - Numérotés à partir de **32** (puis 36, 40, 44...)
> - Utilisée par : 802.11a, 802.11n, **802.11ac** (exclusivement), 802.11ax

### Agrégation de canaux et MIMO

> [!important] Agrégation de canaux (Channel Bonding)
> Pour augmenter les débits, les normes récentes permettent d'agréger des canaux contigus :

| Norme | Canaux agrégeables | Largeur totale |
|-------|-------------------|----------------|
| 802.11n | 2 canaux | 40 MHz |
| 802.11ac | 2, 4 ou 8 canaux | 40, 80 ou 160 MHz |
| 802.11ax | 2, 4 ou 8 canaux | 40, 80 ou 160 MHz |

> [!important] MIMO (Multiple-Input Multiple-Output)
> Technique apparue avec **802.11n** :
> - Utilisation **simultanée de plusieurs antennes** en émission et/ou réception
> - Permet d'augmenter débits **et** portée
> - Un équipement avec plus d'antennes peut gérer des **connexions simultanées**
> - **Nécessite** des antennes supplémentaires sur tous les équipements

### Les antennes

> [!note] Types d'antennes WiFi

| Type | Caractéristique | Usage typique |
|------|----------------|---------------|
| **Omnidirectionnelle** | Zone de couverture en "donut" autour de l'antenne | La plus courante, déploiement général |
| **Directionnelle** | Émission/réception dans **une seule direction** | Liaisons point-à-point, longue portée |

> [!tip] Règle de portée
> La portée est **proportionnelle à la puissance du signal**. Les puissances maximum sont réglementées (souvent inférieures aux recommandations). La puissance est ajustée dynamiquement pour économiser l'énergie.

### Tableau récapitulatif des normes

> [!important] Comparatif des normes 802.11

| Protocole | Date | Fréquence | Largeur | Débit max théorique | MIMO max |
|-----------|------|-----------|---------|---------------------|----------|
| **802.11b** | 1999 | 2,4 GHz | 22 MHz | 11 Mbps | 1 |
| **802.11a** | 1999 | 5 GHz | 20 MHz | 54 Mbps | 1 |
| **802.11g** | 2003 | 2,4 GHz | 20 MHz | 54 Mbps | 1 |
| **802.11n** | 2009 | 2,4 / 5 GHz | 20 / 40 MHz | 72/150 Mbps par antenne | 4 |
| **802.11ac** | 2015 | 5 GHz | 20 à 160 MHz | 433 Mbps à 6,93 Gbps | 8 |
| **802.11ax** | 2021 | 1 à 7,1 GHz | 20 à 160 MHz | 1,1 à 10,5 Gbps | 8 |
| **802.11be** | 2024 (prévu) | 2,4/5/6 GHz | 320 MHz | objectif 40 Gbps | 16 |

> [!tip] Mnémotechnique pour retenir l'ordre des normes
> **"B A G N AC AX BE"** → **"Bah, a gagné en accélérant axé bien"**
> Ou par génération WiFi : WiFi 1 (b) → WiFi 2 (a) → WiFi 3 (g) → WiFi 4 (n) → WiFi 5 (ac) → WiFi 6 (ax) → WiFi 7 (be)

> [!warning] Les débits indiqués sont des **maximums théoriques**
> En conditions réelles, les débits effectifs sont nettement inférieurs (interférences, distance, nombre d'utilisateurs, obstacles...).

---

## Les modes opératoires

> [!abstract] Vue d'ensemble des modes de déploiement
> 802.11 propose plusieurs modes pour déployer un réseau WiFi, adaptés à différents besoins et contextes d'infrastructure.

### Mode ad hoc (IBSS)

> [!quote] Définition
> **IBSS** = Independent Basic Service Set

> [!info] Caractéristiques du mode ad hoc
> - Communication **point à point directe** entre stations
> - **Pas de point d'accès**, pas d'infrastructure
> - Chaque terminal définit sa propre **BSA** (Basic Service Area = cellule de couverture)
> - Mise en place simple

> [!example] Usage typique
> Déploiement d'un réseau temporaire au besoin (ex : partage de connexion entre smartphones, réseau de chantier provisoire).

> [!warning] Limite
> Non adapté aux déploiements professionnels pérennes — pas de centralisation, pas de sécurité avancée.

### Mode infrastructure BSS

> [!quote] Définition
> **BSS** = Basic Service Set

> [!info] Mode infrastructure avec un seul AP
> - **1 point d'accès (AP)** = 1 cellule = 1 BSS
> - Toutes les communications **passent par le point d'accès**
> - Les stations **partagent le canal** de communication (et donc le débit)
> - **SSID** (Service Set Identifier) : nom du réseau WiFi
> - **BSSID** : adresse MAC du point d'accès

### Mode infrastructure étendu ESS

> [!quote] Définition
> **ESS** = Extended Service Set

> [!info] Mode infrastructure avec plusieurs AP
> - **Plusieurs AP** connectés via un **DS** (Distribution System, en général Ethernet)
> - **SSID commun** à tous les AP pour le même réseau
> - Chaque AP a son propre **BSSID** (adresse MAC)

> [!note] Topologies ESS

| Topologie | Cellules | Mobilité |
|-----------|----------|----------|
| **Cellules non recouvrantes** | Sans chevauchement | ❌ Impossible |
| **Cellules recouvrantes** | Avec chevauchement (15-20%) | ✅ Via roaming (802.11f) |

> [!tip] Optimisation avec cellules recouvrantes
> - Limiter la puissance du signal améliore le débit
> - Plus grand nombre de connexions sans perte de performance
> - **Roaming (802.11f)** : échange d'informations entre AP via le DS pour éviter les coupures lors des déplacements

### Réseau mesh

> [!info] Mode réseau maillé
> Le **mesh** offre les avantages de l'ESS **sans câblage** (DS) entre les AP :
> - L'AP primaire est relié aux autres réseaux (Internet/LAN)
> - Peut disposer d'un **failover** (redondance)
> - Tous les AP relaient le trafic → nécessite un **protocole de routage dynamique**

> [!example] Usage typique
> Déploiement WiFi dans de grands espaces sans possibilité de passer du câble (entrepôts, bâtiments anciens, espaces ouverts).

---

## La couche liaison

### Adresses MAC

> [!info] Adresses MAC en WiFi
> 802.11 utilise les **mêmes adresses MAC** (Medium Access Control) que 802.3 (Ethernet).
> Chaque interface WiFi possède une adresse MAC attribuée par le constructeur (ou choisie manuellement).

### Sous-couche LLC

> [!important] Rôle de la LLC (Logical Link Control - 802.2)
> En WiFi, la LLC est **indispensable** (contrairement à Ethernet) en raison de la moindre fiabilité du support physique sans fil.

> [!note] Modes opératoires LLC

| Type | Mode | Acquittement |
|------|------|-------------|
| **Type 1** | Sans connexion | Sans acquittement |
| **Type 2** | Mode connecté | Avec n° de séquence des trames |
| **Type 3** | Sans connexion | Avec acquittement |

> Services assurés : **contrôle et reprise des erreurs** + **contrôle de flux**

### CSMA/CA - Accès au médium

> [!important] Pourquoi CSMA/CD ne fonctionne pas en WiFi ?
> Le **CSMA/CD** (Collision Detection) est impossible en WiFi car une station ne peut pas détecter une collision pendant qu'elle émet sur un médium radio.
> → Solution : **CSMA/CA** (Collision **Avoidance**)

> [!quote] Définition CSMA/CA
> **CSMA/CA** = Carrier-Sense Multiple Access with Collision Avoidance

> [!info] Fonctionnement CSMA/CA (DCF - Distributed Coordination Function)

```
Station veut émettre
       │
       ▼
  Écoute du médium (Carrier-Sense)
       │
  ┌────┴────┐
  │ Occupé? │
  └────┬────┘
  OUI ─┤ Attente (Back-off aléatoire - BEB)
       │
  NON ─┤ Émission de la trame
       │
       ▼
  Attente de l'ACK
       │
  ┌────┴────────────┐
  │ ACK reçu?       │
  └────┬────────────┘
  OUI ─┤ Succès ✓
  NON ─┤ Réémission après BEB (Binary Exponential Backoff)
```

> [!note] BEB (Binary Exponential Backoff)
> En cas de collision ou d'absence d'ACK, la station **double** son temps d'attente avant de réessayer. Cela permet de réduire les collisions dans un réseau chargé.

### RTS et CTS

> [!info] Mécanisme RTS/CTS (optionnel)
> Pour les trames de grande taille, 802.11 propose un mécanisme de **réservation du canal** :

```
Station A             AP              Autres stations
    │                  │                    │
    │──── RTS ────────►│                    │
    │                  │──── CTS ──────────►│ (canal réservé - NAV)
    │◄──── CTS ────────│                    │
    │                  │                    │
    │══════ DATA ══════►│                    │ (silence obligatoire)
    │◄───── ACK ────────│                    │
```

> [!note] Priorités des trames et InterFrame Spacing

| Type | Intervalle | Priorité | Usage |
|------|-----------|----------|-------|
| **SIFS** | Short IFS | ⬆️ Haute | RTS, CTS, ACK |
| **PIFS** | PCF IFS | ↔️ Moyenne | Mode PCF |
| **DIFS** | DCF IFS | ⬇️ Basse | Trames de données classiques |

> [!tip] Mnémotechnique
> **SIFS < PIFS < DIFS** → Plus l'intervalle est court, plus la trame est prioritaire.

> [!info] NAV (Network Allocation Vector)
> Pendant le NAV calculé par les autres stations après réception du CTS, seule la station ayant réservé le canal peut émettre — les autres **ne sondent plus le canal**.

### PCF et HCF

> [!note] PCF (Point Coordinated Function)
> - C'est le **point d'accès qui distribue la parole** via des trames CF Poll Frame
> - Utilise le **PIFS** (prioritaire sur le DIFS)
> - **Rarement implémenté** dans les équipements réels

> [!info] HCF (Hybrid Coordination Function) - IEEE 802.11e
> Mode hybride entre DCF et PCF avec gestion des **priorités QoS** :

| Catégorie | Usage | Priorité |
|-----------|-------|----------|
| Voix | VoIP | ⬆️⬆️ Très haute |
| Vidéo | Streaming | ⬆️ Haute |
| Best effort | Navigation web | Normale |
| Background | Téléchargements | ⬇️ Basse |

---

## La sécurité

> [!abstract] Enjeux de sécurité WiFi
> Le support radio implique des risques spécifiques : l'écoute ne nécessite que d'être à portée du signal, sans accès physique au réseau.

### Protocoles obsolètes

> [!warning] WEP - À ne jamais utiliser
> **WEP** (Wired Equivalent Privacy) :
> - Utilise **RC4** — algorithme cryptographique obsolète et mal implémenté
> - Des outils clé en main permettent de le **casser en quelques instants**
> - **Complètement abandonné** — ne jamais configurer WEP

> [!warning] WPA - Insuffisant
> **WPA** (Wi-Fi Protected Access) :
> - Solution **intermédiaire temporaire** en attente de WPA2
> - Utilise encore RC4 avec **TKIP** (Temporal Key Integrity Protocol)
> - Génère des clés temporaires depuis une **PSK** (Pre-Shared Key / passphrase)
> - **Insuffisant** — à éviter également

### Protocoles recommandés

> [!important] WPA2 (2004) — Recommandé minimum
> - Remplace RC4 par **AES** (bien plus robuste)
> - Utilise **CCMP** (Counter-Mode/CBC-Mac Protocol) au lieu de TKIP

| Mode | Nom | Mécanisme | Usage |
|------|-----|-----------|-------|
| **WPA2-Personal** | WPA2-PSK | Passphrase partagée | Domicile, petites structures |
| **WPA2-Enterprise** | WPA2-EAP | Serveur RADIUS + EAP | Entreprises |

> [!tip] Bonne pratique WPA2-PSK
> Utiliser des passphrases **longues (> 20 caractères) et complexes**.

> [!info] EAP (Extensible Authentication Protocol)
> Permet différentes stratégies d'authentification. Nécessite un **serveur d'authentification** (ex : **RADIUS**).

> [!note] WPA3 (2018)
> - Encore peu répandu mais recommandé pour les nouveaux déploiements
> - Résout les faiblesses de WPA2 (notamment les attaques par dictionnaire)

### WPS et recommandations ANSSI

> [!warning] WPS - À désactiver impérativement
> **WPS** (Wi-Fi Protected Setup) est un mécanisme d'autoconfiguration entre AP et terminal.
> Le code PIN WPS a été démontré comme facilement contournable.
> **Recommandation : désactiver WPS sur tous les équipements.**

> [!tip] Référence officielle
> Pour les recommandations de sécurité WiFi : **Guide ANSSI** (Agence Nationale de la Sécurité des Systèmes d'Information).
> Bien que datant de 2013, il reste en grande partie d'actualité.

### Failles récentes

> [!info] Failles majeures WiFi

| Faille | Année | Protocole affecté | Description |
|--------|-------|-------------------|-------------|
| **KRACK** | 2017 | WPA2 | Key Reinstallation Attack |
| **Krœœk** | 2019 | WPA2 | Faille sur certaines puces spécifiques |
| **Dragonblood** | 2019 | WPA3 | Faiblesses dans l'implémentation initiale |

> [!success] Ces failles ont été corrigées
> Par amélioration du protocole et/ou correction des bugs d'implémentation.

> [!important] Leçon essentielle
> **La mise à jour des équipements est primordiale !** Un équipement non mis à jour reste vulnérable même si la faille est connue et corrigée.

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Normes et histoire
- WiFi = marque pour les équipements conformes aux normes **IEEE 802.11**
- Évolution : 802.11b (1999, 11 Mbps) → 802.11ax (2021, 10,5 Gbps)
- Noms commerciaux : WiFi 4 (n), WiFi 5 (ac), WiFi 6 (ax), WiFi 7 (be)

### Fréquences et canaux
- **2,4 GHz** : 3 canaux non chevauchants (1, 6, 11), portée supérieure, plus d'interférences
- **5 GHz** : 22 canaux disponibles en Europe, débits supérieurs, moins d'interférences
- **6 GHz** : autorisé depuis 2021, réservé aux normes récentes (ax/be)

### Modes de déploiement
- **Ad hoc (IBSS)** : sans infrastructure, temporaire
- **BSS** : 1 AP, petite zone
- **ESS** : plusieurs AP + DS (Ethernet), roaming possible avec 802.11f
- **Mesh** : plusieurs AP sans câblage entre eux, routage dynamique

### Couche liaison
- **LLC obligatoire** en WiFi (gestion d'erreurs)
- **CSMA/CA** (et non CD) : écoute avant émission + attente ACK
- **RTS/CTS** : réservation optionnelle du canal pour grandes trames
- **SIFS < PIFS < DIFS** : ordre de priorité des intervalles

### Sécurité
- **WEP** : ❌ interdit (RC4, cassable instantanément)
- **WPA** : ❌ insuffisant (RC4 + TKIP)
- **WPA2-PSK** : ✅ minimum acceptable (AES + CCMP, passphrase > 20 car.)
- **WPA2-Enterprise** : ✅ recommandé en entreprise (EAP + RADIUS)
- **WPA3** : ✅ idéal si matériel compatible
- **WPS** : ❌ **toujours désactiver**
- **Mise à jour** : primordiale contre KRACK, Krœœk, Dragonblood

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|-----------|
| **802.11** | Ensemble de normes IEEE pour les réseaux locaux sans fil (WiFi) |
| **ANFR** | Agence Nationale des Fréquences — régule l'usage des fréquences radio en France |
| **ARCEP** | Autorité de régulation des communications électroniques, des postes et de la distribution de la presse |
| **ANSSI** | Agence Nationale de la Sécurité des Systèmes d'Information — publie les recommandations sécurité |
| **AP** | Access Point — point d'accès WiFi |
| **BSA** | Basic Service Area — zone de couverture d'un point d'accès |
| **BSS** | Basic Service Set — réseau WiFi avec un seul AP |
| **BSSID** | Identifiant du BSS = adresse MAC du point d'accès |
| **CCMP** | Counter-Mode/CBC-Mac Protocol — protocole de chiffrement utilisé par WPA2 (remplace TKIP) |
| **DCF** | Distributed Coordination Function — méthode d'accès CSMA/CA |
| **DFS** | Dynamic Frequency Selection — gestion dynamique des fréquences |
| **DS** | Distribution System — système de distribution reliant les AP (généralement Ethernet) |
| **EAP** | Extensible Authentication Protocol — protocole d'authentification flexible |
| **ESS** | Extended Service Set — réseau WiFi avec plusieurs AP connectés |
| **HCF** | Hybrid Coordination Function (802.11e) — gestion hybride avec QoS |
| **IBSS** | Independent Basic Service Set — réseau ad hoc sans infrastructure |
| **ISM** | Industrial, Scientific and Medical — bandes de fréquences libres |
| **LLC** | Logical Link Control (802.2) — sous-couche liaison assurant contrôle d'erreurs et de flux |
| **MAC** | Medium Access Control — sous-couche gérant l'accès au médium partagé |
| **MIMO** | Multiple-Input Multiple-Output — utilisation simultanée de plusieurs antennes |
| **NAV** | Network Allocation Vector — timer indiquant la durée de réservation du canal |
| **PCF** | Point Coordinated Function — mode où l'AP distribue la parole |
| **PSK** | Pre-Shared Key — clé partagée (passphrase) utilisée en WPA/WPA2-Personal |
| **RADIUS** | Remote Authentication Dial-In User Service — serveur d'authentification centralisé |
| **RC4** | Algorithme de chiffrement symétrique — utilisé par WEP et WPA, aujourd'hui obsolète |
| **SSID** | Service Set Identifier — nom du réseau WiFi |
| **TKIP** | Temporal Key Integrity Protocol — protocole de gestion de clés temporaires (WPA) |
| **TPC** | Transmit Power Control — gestion dynamique de la puissance d'émission |
| **WEP** | Wired Equivalent Privacy — premier protocole de sécurité WiFi, **obsolète et non sécurisé** |
| **WPA** | Wi-Fi Protected Access — protocole de sécurité intermédiaire, **insuffisant** |
| **WPA2** | Wi-Fi Protected Access 2 — standard actuel recommandé (AES/CCMP) |
| **WPA3** | Wi-Fi Protected Access 3 — protocole le plus récent, résout les faiblesses de WPA2 |
| **WPS** | Wi-Fi Protected Setup — mécanisme d'autoconfiguration, **à désactiver impérativement** |

---

*Document généré pour révision TSSR - Titre RNCP | Mars 2026*
