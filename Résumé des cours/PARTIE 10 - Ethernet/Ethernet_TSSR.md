# Ethernet - Bien débuter
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Protocole Ethernet et réseaux locaux  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#La norme Ethernet|La norme Ethernet]]
   - [[#Définition|Définition]]
   - [[#Architecture IEEE|Architecture IEEE]]
   - [[#Normes PHY|Normes PHY]]
3. [[#Câblage et équipements|Câblage et équipements]]
   - [[#Câble coaxial|Câble coaxial]]
   - [[#Paire torsadée|Paire torsadée]]
   - [[#Câbles RJ45|Câbles RJ45]]
   - [[#Fibre optique|Fibre optique]]
   - [[#Équipements réseau|Équipements]]
4. [[#L'adresse MAC|L'adresse MAC]]
   - [[#Problème adressage|Problème d'adressage]]
   - [[#Format MAC|Format des adresses]]
   - [[#Consultation MAC|Consultation]]
5. [[#La trame Ethernet|La trame Ethernet]]
   - [[#Format général|Format général]]
   - [[#Composants trame|Composants]]
   - [[#Structure complète|Structure complète]]
6. [[#Switch et VLAN|Switch et VLAN]]
   - [[#Commutateur|Commutateur]]
   - [[#Gestion trames|Gestion des trames]]
   - [[#VLAN|Réseaux locaux virtuels]]
   - [[#IEEE 802.1Q|IEEE 802.1Q]]
7. [[#Points clés à retenir|Points clés]]
8. [[#Glossaire technique|Glossaire]]

---

## Introduction

> [!abstract] Vue d'ensemble
> **Ethernet** est le protocole de réseau local filaire le plus répandu au monde. Défini par la norme **IEEE 802.3**, il permet la mise en place de réseaux LAN performants, fiables et économiques.

> [!question] Différence entre Internet et Ethernet ?
> - **Ethernet** : Protocole de réseau **local** (LAN) pour communication entre équipements proches
> - **Internet** : Réseau **mondial** (WAN/GAN) interconnectant des millions de réseaux locaux
> 
> **Analogie** : Ethernet = réseau téléphonique d'un bâtiment | Internet = réseau téléphonique mondial

### Pourquoi étudier Ethernet ?

En tant que **TSSR**, vous devez :

- **Concevoir et déployer** des infrastructures réseau locales
- **Câbler** correctement avec les bonnes catégories
- **Configurer** des switches et VLANs
- **Diagnostiquer** les problèmes de connectivité
- **Optimiser** les performances réseau
- Comprendre les **trames** et l'**adressage MAC**

---

## La norme Ethernet

### Définition

> [!quote] Définition officielle
> **Ethernet** est un ensemble de protocoles permettant la mise en place de **réseaux locaux filaires** (Local Area Network ou LAN).

> [!info] Caractéristiques principales

**Standardisation** :
- Défini par la norme **IEEE 802.3**
- Protocole le plus couramment utilisé pour les réseaux filaires

**Historique** :
- Née dans les **années 1970**
- Évolution continue depuis plus de 50 ans

**Performances** :
- Débits : de **10 Mbps** à **400 Gbps**
- Objectif futur : **1,6 Tbps** (voir Ethernet Roadmap)

> [!example] Débits courants
> - **10 Mbps** : Ethernet classique (obsolète)
> - **100 Mbps** : Fast Ethernet (ancien)
> - **1 Gbps** : Gigabit Ethernet (standard actuel)
> - **10 Gbps** : 10 Gigabit Ethernet (datacenters, backbones)
> - **40/100 Gbps** : Datacenters haute performance

---

### Architecture IEEE

> [!important] Modèle en couches IEEE 802

L'architecture Ethernet est définie par **IEEE 802** et est commune aux protocoles IEEE :

```
┌──────────────────────────────────────┐
│         COUCHE 2 (Liaison)           │
│  ┌────────────────────────────────┐  │
│  │  LLC (Logical Link Control)    │  │  ← Commune à tous protocoles IEEE
│  │       IEEE 802.2               │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  MAC (Medium Access Control)   │  │  ← Spécifique à chaque protocole
│  │    (Ethernet, WiFi, etc.)      │  │     (IEEE 802.3 pour Ethernet)
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│      COUCHE 1 (Physique - PHY)       │  ← Câbles, connecteurs, signaux
└──────────────────────────────────────┘
```

**Composants** :

| Couche | Nom | Rôle |
|--------|-----|------|
| **Couche 1** | **PHY** (Physique) | Transmission physique des bits (câbles, connecteurs, signaux) |
| **Couche 2 (bas)** | **MAC** (Medium Access Control) | Contrôle d'accès au média, adressage MAC, spécifique à Ethernet |
| **Couche 2 (haut)** | **LLC** (Logical Link Control) | Contrôle de liaison logique, commune à tous les protocoles IEEE 802 |

---

### Résumé des normes IEEE

> [!note] Famille de normes IEEE 802

| Norme | Nom | Description |
|-------|-----|-------------|
| **IEEE 802** | Overview & Architecture | Vue d'ensemble et architecture générale |
| **IEEE 802.1** | Bridging & Management | Gestion des réseaux, ponts, VLANs |
| **IEEE 802.2** | Logical Link Control | Couche LLC commune à tous |
| **IEEE 802.3** | **Ethernet** | Protocole Ethernet (LAN filaire) |
| **IEEE 802.11** | **WiFi** | Wireless LAN (réseau sans fil) |
| **IEEE 802.15** | Bluetooth | Wireless PAN (réseau personnel) |
| **IEEE 802.16** | WiMAX | Broadband Wireless MAN |

> [!tip] À retenir pour l'examen
> - **802.3** = Ethernet
> - **802.11** = WiFi
> - **802.1Q** = VLANs (vu plus tard)

---

### Normes PHY

> [!important] Les normes de la couche physique

Les normes PHY (Physical Layer) d'Ethernet définissent :

**Pour la couche Physique (couche 1 OSI)** :
- Type de **médium** (câble, fibre)
- **Débit** maximal
- **Portée** maximale
- **Taux d'erreur** acceptable
- Caractéristiques électriques/optiques

**Pour la sous-couche MAC (partie basse couche 2 OSI)** :
- **Format du PDU** (Trame Ethernet)
- **Adresses** (MAC)
- Technique de partage du médium (CSMA/CD si nécessaire)

> [!note] LLC optionnelle
> Le modèle IEEE prévoit d'utiliser **LLC** entre MAC et le protocole de niveau 3 (réseau), mais dans le cas d'**IP**, elle n'est en général **pas utilisée**.

---

#### Nomenclature des normes PHY

> [!example] Comprendre la notation

**Format** : `<Débit>BASE-<Type>`

```
100BASE-T
 │   │   │
 │   │   └─ Type de medium
 │   │       T = Twisted Pairs (paires torsadées)
 │   │       F = Fibre optique
 │   │       
 │   └─ Type de codage
 │       BASE = Bande de base (le plus fréquent)
 │       
 └─ Débit maximum théorique
     100 = 100 Mbps
     1000 = 1 Gbps
     10G = 10 Gbps
```

**Exemples courants** :

| Norme | Débit | Médium | Distance max |
|-------|-------|--------|--------------|
| **10BASE-T** | 10 Mbps | Paires torsadées (Cat 3) | 100 m |
| **100BASE-TX** | 100 Mbps | Paires torsadées (Cat 5) | 100 m |
| **1000BASE-T** | 1 Gbps | Paires torsadées (Cat 5e/6) | 100 m |
| **10GBASE-T** | 10 Gbps | Paires torsadées (Cat 6a/7) | 100 m |
| **1000BASE-SX** | 1 Gbps | Fibre multimode | 550 m |
| **1000BASE-LX** | 1 Gbps | Fibre monomode | 10 km |
| **10GBASE-SR** | 10 Gbps | Fibre multimode | 300 m |
| **10GBASE-LR** | 10 Gbps | Fibre monomode | 10 km |

---

#### Détail sur le débit

> [!info] Comprendre les unités de débit

**Débit en bits par seconde (bit/s)** :

| Unité | Symbole | Valeur | Équivalent |
|-------|---------|--------|------------|
| kilo | **k** | 1 000 | 10³ |
| Mega | **M** | 1 000 000 | 10⁶ |
| Giga | **G** | 1 000 000 000 | 10⁹ |
| Tera | **T** | 1 000 000 000 000 | 10¹² |

> [!warning] Conversion bits ↔ octets
> **8 bits = 1 octet**
> 
> Donc :
> - **8 Mb = 1 Mo** (pour volume de stockage)
> - **1 Gbps = 125 Mo/s** (débit de transfert réel)
> 
> **Exemple** :
> - Connexion Gigabit Ethernet (1 Gbps) = débit théorique de **125 Mo/s**
> - En pratique : ~110-115 Mo/s (overhead protocoles)

---

## Câblage et équipements

### Câble coaxial

> [!note] Support historique d'Ethernet

**Définition** :
Le **câble coaxial** (ou ligne coaxiale) est une liaison asymétrique, utilisée en basses/hautes fréquences, composée d'un câble à **deux conducteurs** (central et extérieur).

**Caractéristiques** :
- Utilisé dans les **premières versions** d'Ethernet (10BASE2, 10BASE5)
- **Obsolète** pour Ethernet aujourd'hui
- Toujours utilisé pour TV câble, antennes

**Structure** :
```
┌─────────────────────────────────────┐
│    Gaine isolante extérieure        │
│  ┌───────────────────────────────┐  │
│  │  Blindage métallique (masse)  │  │
│  │ ┌───────────────────────────┐ │  │
│  │ │   Isolant diélectrique    │ │  │
│  │ │  ┌─────────────────────┐  │ │  │
│  │ │  │ Conducteur central  │  │ │  │
│  │ │  └─────────────────────┘  │ │  │
│  │ └───────────────────────────┘ │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

### Paire torsadée

> [!important] Standard actuel pour Ethernet

**Définition** :
Une **paire torsadée** est une ligne symétrique formée de **deux fils conducteurs enroulés en hélice** l'un autour de l'autre.

**Avantages** :
- ✅ Limitation de la sensibilité aux **interférences** électromagnétiques
- ✅ Réduction de la **diaphonie** (interférence entre paires)
- ✅ Économique et facile à installer
- ✅ Standard universel

**Composition** :
- En général **4 paires torsadées** (donc **8 fils**)
- Chaque paire a un code couleur spécifique
- Enroulées avec un pas différent pour limiter la diaphonie

```
┌──────────────────────────────────┐
│    Gaine isolante extérieure     │
│  ┌────────────────────────────┐  │
│  │  Paire 1 (Orange)          │  │  ← Torsadée
│  │  Paire 2 (Vert)            │  │  ← Torsadée
│  │  Paire 3 (Bleu)            │  │  ← Torsadée
│  │  Paire 4 (Marron)          │  │  ← Torsadée
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

---

### Câbles RJ45

> [!important] Connecteur standard Ethernet

**Définition** :
**RJ45** est le nom du **connecteur** (prise) utilisé au bout des câbles 4 paires torsadées. Ces câbles sont généralement appelés "**câbles RJ45**".

> [!warning] Ne pas confondre
> **RJ45** : 8 positions, 8 contacts (8P8C) - Ethernet
> 
> **RJ11** : 6 positions, 4 contacts (6P4C) - Téléphonie analogique (plus petit)

**Origine du nom** :
- **RJ** = Registered Jack (prise déposée aux États-Unis)
- **45** = Numéro dans le standard RJ

---

#### Câbles droits ou croisés

> [!info] Deux types de câblage

Ethernet par paires torsadées utilise des **paires différentes** pour émettre et recevoir.

**Deux types de prises/câbles** :

| Type | Nom technique | Usage | Connexion |
|------|---------------|-------|-----------|
| **Droit** | **MDI** (Medium Dependent Interface) | Hôtes (ordinateurs, serveurs) | PC → Switch |
| **Croisé** | **MDI-X** (MDI Crossover) | Équipements réseau | Switch → Switch, PC → PC |

**Principe** :
- **Câble droit** : Brochage identique aux deux extrémités (1-1, 2-2, 3-3, etc.)
- **Câble croisé** : Paires TX et RX inversées (1-3, 2-6, 3-1, 6-2)

```
CÂBLE DROIT (MDI)              CÂBLE CROISÉ (MDI-X)
PC → Switch                    PC → PC ou Switch → Switch

1 ────────────── 1             1 ───────┐  ┌───── 1
2 ────────────── 2             2 ───┐   └──┼───── 3
3 ────────────── 3             3 ───┼───┐  └───── 2
6 ────────────── 6             6 ───┘   └──────── 6
```

> [!tip] Auto MDI/MDIX
> **Auto MDI/MDIX** permet aux prises de **se croiser automatiquement** au besoin. Cette fonctionnalité rend l'utilisation de câbles croisés de **plus en plus rare**.
> 
> Disponible sur la plupart des équipements modernes (depuis ~2005).

---

#### Catégories de câbles RJ45

> [!important] Choisir la bonne catégorie

| Catégorie | Débit max | Distance max | Fréquence | Usage |
|-----------|-----------|--------------|-----------|-------|
| **CAT 5** | 100 Mbps | 100 m | 100 MHz | Obsolète (Fast Ethernet) |
| **CAT 5e** | 1 Gbps | 100 m | 100 MHz | Standard actuel (Gigabit) |
| **CAT 6** | 1 Gbps / 10 Gbps | 100 m / 50 m | 250 MHz | Recommandé (10G sur courte distance) |
| **CAT 6a** | 10 Gbps | 100 m | 500 MHz | Professionnel (10 Gigabit) |
| **CAT 7** | 10 Gbps | 100 m | 600 MHz | Haute performance |
| **CAT 8** | 25-40 Gbps | 30 m | 2000 MHz | Datacenters |

> [!tip] Recommandations TSSR
> - **Résidentiel/PME** : CAT 5e (1 Gbps) - suffisant pour la plupart des usages
> - **Entreprise** : CAT 6 (10 Gbps sur 50m) - pérennité
> - **Datacenter/Backbone** : CAT 6a ou fibre optique

> [!warning] Attention à la distance
> - CAT 6 : 10 Gbps uniquement sur **55 mètres maximum**
> - Au-delà, débit limité à 1 Gbps ou utiliser CAT 6a

---

#### Type de blindage RJ45

> [!info] Protection contre les interférences

Le **blindage** des câbles est défini par un code : **`<Lettre>/<Lettre>TP`**

**Structure** :
- **1ère lettre** : Blindage **global** du câble
- **2ème lettre** : Blindage des **paires individuelles**
- **TP** : Twisted Pairs (paires torsadées)

**Signification des lettres** :

| Lettre | Signification | Description |
|--------|---------------|-------------|
| **U** | **U**nshielded | Non blindé |
| **F** | **F**oiled | Blindage par feuillard d'aluminium |
| **S** | **S**hielded | Blindage par tresse d'aluminium ou de cuivre |
| **SF** | **S**hielded **F**oiled | Double blindage (tresse + feuille) |

---

##### Niveaux de blindage

> [!example] Configurations courantes

| Code | Nom | Blindage global | Blindage paires | Protection | Coût | Usage |
|------|-----|-----------------|-----------------|------------|------|-------|
| **U/UTP** | Non blindé | ❌ Non | ❌ Non | Minimale | € | Résidentiel |
| **F/UTP** | FTP | ✅ Feuillard | ❌ Non | Bonne | €€ | Bureaux |
| **U/FTP** | - | ❌ Non | ✅ Feuillard | Bonne | €€ | - |
| **F/FTP** | FFTP | ✅ Feuillard | ✅ Feuillard | Très bonne | €€€ | Industrie |
| **S/FTP** | SFTP | ✅ Tresse cuivre | ✅ Feuillard | Excellente | €€€€ | Environnement EMI |
| **SF/UTP** | SFUTP | ✅ Double (tresse+feuille) | ❌ Non | Excellente | €€€€ | Datacenter |

> [!tip] Choix du blindage
> - **Environnement "propre"** (bureaux) : U/UTP ou F/UTP
> - **Environnement industriel** (EMI) : F/FTP ou S/FTP
> - **Datacenter** : SF/UTP ou fibre optique

> [!warning] Inconvénient du blindage
> Le blindage nécessite une **mise à la terre correcte**. Sans cela, le blindage peut devenir une **antenne** et aggraver les interférences !

---

### Fibre optique

> [!important] Haute performance et longue distance

**Définition** :
Une **fibre optique** (FO) est un fil dont l'âme, très fine, en verre ou en plastique, a la propriété de **conduire la lumière** et sert pour la transmission de données numériques.

**Caractéristiques** :

| Aspect | Cuivre (RJ45) | Fibre Optique |
|--------|---------------|---------------|
| **Distance** | 100 m (max) | Plusieurs km à 100+ km |
| **Débit** | 10 Gbps (CAT 6a) | 10 Gbps à 400 Gbps+ |
| **Interférences** | Sensible (EMI) | Insensible (lumière) |
| **Poids** | Lourd | Léger |
| **Fragilité** | Robuste | Fragile (flexion) |
| **Coût** | Économique | Plus cher (matériel + installation) |
| **Sécurité** | Écoute possible | Très difficile à intercepter |

**Avantages** :
- ✅ **Très longues distances** (km)
- ✅ **Débits très élevés** (100+ Gbps)
- ✅ **Insensible aux interférences** électromagnétiques
- ✅ Très **sécurisée** (difficile à intercepter)
- ✅ Légère

**Inconvénients** :
- ❌ Plus **fragile** que le cuivre (rayon de courbure)
- ❌ Plus **coûteuse** (câble + équipements)
- ❌ Installation plus **complexe** (soudure, connecteurs)

---

#### Types de fibres

> [!info] Multimode vs Monomode

| Type | Symbole | Diamètre cœur | Distance | Coût | Usage |
|------|---------|---------------|----------|------|-------|
| **Multimode** | **MMF** (Multi-Mode Fiber) | 50 ou 62,5 µm | 300 m à 2 km | € | LAN, datacenters |
| **Monomode** | **SMF** (Single-Mode Fiber) | 9 µm | 10 km à 100+ km | €€€ | WAN, backbone, longue distance |

**Fibre Multimode (MMF)** :
- ✅ Moins chère
- ✅ Cœur plus large (plus facile à connecter)
- ❌ Distance limitée (300m - 2km selon débit)
- LED comme source lumineuse
- Usage : **Datacenters, bâtiments**

**Fibre Monomode (SMF)** :
- ✅ Très longue distance (10-100+ km)
- ✅ Débits très élevés
- ❌ Plus chère
- ❌ Cœur très fin (fragile, connexion délicate)
- Laser comme source lumineuse
- Usage : **Backbone, interconnexion sites, FAI**

**Connecteurs courants** :
- **LC** (Lucent Connector) : Petit, standard actuel
- **SC** (Subscriber Connector) : Carré, robuste
- **ST** (Straight Tip) : Baïonnette, ancien
- **MTP/MPO** : Multi-fibres (12, 24)

---

### Carte réseau

> [!note] Interface matérielle

**Définition** :
La **carte réseau** (ou NIC - Network Interface Card) est un périphérique informatique composé d'éléments électroniques soudés sur un circuit imprimé.

**Rôle** :
- Lien matériel **physique** pour se connecter à un réseau
- Liée à l'**adresse MAC** (identifiant unique)
- Contenue dans tous les appareils connectés modernes

**Formats** :
- **Carte PCI/PCIe** : Carte d'extension interne
- **USB** : Adaptateur externe
- **Intégrée** : Sur la carte mère (standard aujourd'hui)

**Caractéristiques** :
- Vitesse : 10/100/1000 Mbps (Gigabit), 10 Gbps, etc.
- Connecteur : RJ45, SFP, SFP+
- Support : Auto MDI/MDIX, Wake-on-LAN, PoE

---

### Émetteurs-récepteurs

> [!info] Modules pour fibre optique

Il faut un **module réseau particulier** pour émettre et recevoir sur la fibre optique :

**Types de modules** :

| Module | Nom | Débit | Format | Usage |
|--------|-----|-------|--------|-------|
| **GBIC** | Gigabit Interface Converter | 1 Gbps | Grand (obsolète) | Ancien |
| **SFP** | Small Form-factor Pluggable | 1 Gbps | Compact | Standard actuel |
| **SFP+** | Enhanced SFP | 10 Gbps | Compact | 10 Gigabit |
| **QSFP** | Quad SFP | 40 Gbps | 4 canaux | Datacenter |
| **QSFP+** | Enhanced QSFP | 40 Gbps | 4 canaux | Datacenter |
| **QSFP28** | QSFP 28Gbps | 100 Gbps | 4x25 Gbps | Datacenter |
| **CFP** | C Form-factor Pluggable | 100 Gbps+ | Grand | Backbone |
| **XFP** | 10 Gigabit SFP | 10 Gbps | Moyen | 10G |

> [!warning] Coût supplémentaire
> Ces modules doivent en général être **achetés séparément** de l'équipement (switch, routeur).
> 
> Coût : 20€ à 500€+ selon type et qualité

> [!tip] Compatibilité
> Vérifier la compatibilité :
> - Type de fibre (monomode/multimode)
> - Distance supportée
> - Constructeur de l'équipement

---

### Concentrateurs (Hubs)

> [!note] Équipement obsolète (couche 1)

**Définition** :
Les **concentrateurs** (ou hub) sont des répéteurs multiports opérant en **couche physique 1**.

**Fonctionnement** :
- Permet de simuler un **bus** sur une topologie en **étoile**
- **Répète** le signal reçu sur **tous les autres ports** (broadcast)
- Pas d'intelligence, pas de filtrage

**Caractéristiques** :
- Crée un **domaine de collision unique**
- Tous les ports **partagent** la bande passante
- Équipement d'interconnexion classique de l'**Ethernet 10BASE-T**

> [!warning] Obsolète
> Les hubs sont **obsolètes** et remplacés par les **switches** depuis les années 2000.
> 
> **Problèmes** :
> - Partage de bande passante (10 Mbps partagés)
> - Collisions fréquentes
> - Aucune sécurité (tout le monde reçoit tout)

---

### Commutateurs (Switches)

> [!important] Équipement central des réseaux modernes (couche 2)

**Définition** :
Les **commutateurs** (ou switch) sont des ponts multiports opérant en **couche liaison 2**.

**Objectif** :
Transmettre les trames **uniquement au destinataire** (communication point-à-point).

**Avantages sur le hub** :
- ✅ Chaque port = **domaine de collision séparé**
- ✅ **Full-duplex** possible (émission et réception simultanées)
- ✅ Bande passante **dédiée** par port (1 Gbps par port)
- ✅ **Filtrage** intelligent basé sur adresses MAC
- ✅ Sécurité accrue

**Équipement d'interconnexion standard** d'Ethernet aujourd'hui.

---

### Les ports

> [!tip] Diagnostic visuel

Les ports Ethernet disposent en général de **2 diodes de contrôle** :

| Diode | Nom | Signification |
|-------|-----|---------------|
| **Link** | Liaison | Indique si le lien physique est **opérationnel** (allumée = OK) |
| **Activity** | Activité | **Clignote** en fonction de l'activité réseau (transfert de données) |

> [!important] Premier diagnostic
> **Vérifier la diode "Link" est le PREMIER contrôle réseau à faire !**
> 
> - ✅ **Link allumée** : Connexion physique OK
> - ❌ **Link éteinte** : Problème physique (câble, port défectueux, équipement éteint)

**Sur les switches** :
- Les diodes peuvent être **regroupées** (1 diode pour plusieurs ports)
- Affichage possible sur écran LCD
- Consultation via interface web/CLI

---

## L'adresse MAC

### Préciser le destinataire

> [!question] Problématique

Les réseaux Ethernet comportent en général **plus de 2 hôtes**.

Historiquement en **bus** (un câble unique partagé par tous les hôtes), chaque hôte **reçoit toutes les informations**.

**Problème** :
- Comment savoir si une trame est destinée à un hôte particulier ?
- Comment identifier l'expéditeur ?

**Solution** :
Ethernet suppose que chaque interface réseau dispose d'au moins une **adresse physique unique** sur le réseau : l'**adresse MAC**.

---

### Problème d'adressage

> [!question] Comment assurer l'unicité globale ?

**Question** :
Comment s'assurer que toutes les adresses MAC utilisées sur un même réseau physique sont **uniques** ?

**Approche IEEE** :

1. Les adresses sont **stockées** dans les interfaces à la **construction** du matériel (ROM)
2. L'**IEEE attribue un préfixe unique** à chaque constructeur (OUI - Organizationally Unique Identifier)
3. Chaque constructeur est en charge d'assurer l'**unicité du suffixe** de chacune de ses cartes
4. Cela assure l'**unicité globale** des adresses

**Structure** :
```
┌─────────────────┬─────────────────┐
│   OUI (24 bits) │  NIC (24 bits)  │
│   Constructeur  │ Numéro de série │
└─────────────────┴─────────────────┘
    00:1A:2B    :    3C:4D:5E
```

---

### Problèmes restants

> [!warning] Limites du système

**Écueils** :

1. Il est **possible de changer** l'adresse MAC d'une carte (spoofing)
2. Toutes les cartes ne sont pas gérées par un constructeur
   - Exemple : Machines virtuelles (VM)
   - Cartes génériques

**Solutions IEEE** :

**Adresses universelles (globalement uniques)** :
- Gérées par IEEE
- Préfixe constructeur + numéro unique
- À condition que personne ne change les adresses sans respecter les règles

**Adresses locales (localement uniques)** :
- Gérées localement par les administrateurs réseau
- Identifiées par un bit spécifique (bit U/L)

> [!warning] Sécurité
> **Parenthèse cybersécurité** : Ne faites **pas confiance aux adresses MAC** pour la sécurité !
> 
> Les adresses MAC sont facilement **modifiables** (MAC spoofing). Ne pas les utiliser comme seul mécanisme d'authentification.

---

### Format des adresses MAC

> [!important] Structure et notation

**Caractéristiques** :
- Adresse de **48 bits** (6 octets)
- Notation : Par octet en **hexadécimal** séparé par `:` (ou `-`)
- Exemple : `00:1A:2B:3C:4D:5E`

**Structure détaillée** :

```
┌───────────────────────────────────────────────────┐
│           Adresse MAC (48 bits / 6 octets)        │
├─────────────────────────┬─────────────────────────┤
│    OUI (24 bits)        │    NIC (24 bits)        │
│    3 premiers octets    │    3 derniers octets    │
│    Constructeur         │    Numéro de série      │
└─────────────────────────┴─────────────────────────┘

   00    :    1A    :    2B    :    3C    :    4D    :    5E
   │                               │
   └─ Bits spéciaux :              └─ Identifiant unique
      - bit 7 : U/L                   du constructeur
      - bit 8 : I/G
```

**Bits spéciaux (1er octet)** :

| Bit | Nom | Valeur 0 | Valeur 1 |
|-----|-----|----------|----------|
| **7ème bit** | **U/L** (Universal/Local) | **Universelle** (globalement unique) | **Locale** (gérée localement) |
| **8ème bit** | **I/G** (Individual/Group) | **Individuelle** (adresse d'un hôte) | **Groupe** (multicast) |

**Adresses spéciales** :

| Adresse | Type | Usage |
|---------|------|-------|
| `FF:FF:FF:FF:FF:FF` | **Broadcast** | Diffusion à tous les hôtes du réseau |
| `01:00:5E:xx:xx:xx` | **Multicast IPv4** | Groupe multicast |
| `33:33:xx:xx:xx:xx` | **Multicast IPv6** | Groupe multicast |

> [!example] Exemple d'analyse
> Adresse : `D4:93:90:05:2C:1C`
> 
> - **D4:93:90** = OUI (Constructeur)
> - **05:2C:1C** = NIC (Numéro unique)
> - **D4** en binaire = `11010100`
>   - Bit 7 (U/L) = 0 → Adresse **Universelle**
>   - Bit 8 (I/G) = 0 → Adresse **Individuelle**

---

### Consultation MAC

#### Sur Linux

> [!example] Commande ip link

```bash
ip link show
```

Ou en abrégé :
```bash
ip l
```

**Exemple de sortie** :
```bash
wilder@host:~$ ip l
2: enp52s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN
    link/ether d4:93:90:05:2c:1c brd ff:ff:ff:ff:ff:ff
3: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 70:1a:b8:b1:ec:2f brd ff:ff:ff:ff:ff:ff
```

**Lecture** :
- `enp52s0` : Interface Ethernet (câblée)
  - MAC : `d4:93:90:05:2c:1c`
  - État : `DOWN` (câble non connecté)
  
- `wlp0s20f3` : Interface WiFi
  - MAC : `70:1a:b8:b1:ec:2f`
  - État : `UP` (connectée)

> [!tip] Autres commandes
> - `ip addr` : Affiche aussi les adresses IP
> - `ifconfig` : Ancien outil (encore utilisé)

---

#### Sur Windows

> [!example] PowerShell

**Commande** :
```powershell
Get-NetAdapter
```

**Exemple de sortie** :
```powershell
PS C:\Users\wilder> Get-NetAdapter

Name      InterfaceDescription          IfIndex  Status  MacAddress         LinkSpeed
----      --------------------          -------  ------  ----------         ---------
Ethernet  Intel(R) PRO/1000                   6  Up      08-00-27-BF-01-6F  1 Gbps
```

**Lecture** :
- `Ethernet` : Nom de l'interface
- MAC : `08-00-27-BF-01-6F`
- État : `Up` (connectée)
- Débit : `1 Gbps`

> [!tip] Autres commandes Windows
> - `ipconfig /all` : Affiche aussi MAC et IP
> - `getmac` : Affiche uniquement les MAC

---

## La trame Ethernet

### Format général

> [!important] Structure de la trame Ethernet

La **trame Ethernet** est constituée des éléments suivants :

**Composants** :

1. **Début de trame** (Préambule) : Synchronisation
2. **MAC header** (En-tête de liaison) :
   - Adresse MAC destination
   - Adresse MAC source
   - EtherType (type de contenu)
3. **Payload** (Données) : Le paquet IP à transporter
4. **FCS** (Frame Check Sequence) : Somme de contrôle (CRC)

**Positionnement dans les modèles** :
- **Modèle OSI** : Couche 2 (Liaison de données)
- **Modèle TCP/IP** : Couche 1 (Accès réseau)

---

### Schéma général de la trame

> [!example] Vue d'ensemble

```
┌──────────┬────────────┬────────────┬──────────┬─────────────┬─────┐
│ Préambule│  MAC Dest  │  MAC Src   │EtherType │   Payload   │ FCS │
│  (8 o)   │   (6 o)    │   (6 o)    │  (2 o)   │ (46-1500 o) │(4 o)│
└──────────┴────────────┴────────────┴──────────┴─────────────┴─────┘
    │              │                                    │           │
    └─ Sync        └─ En-tête de liaison (14 octets)   └─Données   └─ Contrôle

Taille totale : 64 à 1518 octets (sans préambule)
```

**Pour pouvoir être traités**, ces différents éléments sont de **taille et de position fixes** !

---

### Début de trame (Préambule)

> [!info] Synchronisation

**Rôle** :
Le début de la trame annonce une trame Ethernet et permet la **synchronisation** des équipements.

**Composition** :
- **7 octets** de valeur `10101010` (0xAA)
- **1 octet** de valeur `10101011` (0xAB) : **SFD** (Starting Frame Delimiter)

**SFD** :
- Doit être reçu **en entier** pour valider le début de la trame
- Marque la transition entre préambule et en-tête

**Préfixe complet** (8 octets) :
```
10101010 10101010 10101010 10101010 10101010 10101010 10101010 10101011
   AA       AA       AA       AA       AA       AA       AA       AB
```

> [!note] Synchronisation
> Le préambule permet aux équipements de **synchroniser leurs horloges** avant de recevoir les données.

---

### MAC header (En-tête de liaison)

> [!important] Adressage Ethernet

**Composition** (14 octets au total) :

| Champ | Taille | Description |
|-------|--------|-------------|
| **MAC Destination** | 6 octets (48 bits) | Adresse MAC du destinataire |
| **MAC Source** | 6 octets (48 bits) | Adresse MAC de l'émetteur |
| **EtherType** | 2 octets (16 bits) | Type de protocole de niveau supérieur |

**Exemple** :
```
┌─────────────────┬─────────────────┬──────────┐
│  FF:FF:FF:FF:FF │  D4:93:90:05:2C │  0x0800  │
│   MAC Dest      │    MAC Src      │ EtherType│
│  (Broadcast)    │  (Mon PC)       │  (IPv4)  │
└─────────────────┴─────────────────┴──────────┘
```

**Rappel** :
Les adresses MAC sont notées en hexadécimal :
- `0F:B4:AA:07:F4:1A`
- `D4:93:90:05:2C:1C`

---

### EtherType

> [!info] Identification du protocole supérieur

Le champ **EtherType** (2 octets) indique le **type de contenu** de la payload.

**Interprétation** :

| Valeur | Interprétation | Signification |
|--------|----------------|---------------|
| **≤ 1500** (0x5DC) | **Longueur** | Nombre d'octets du champ "données" (ancien format IEEE 802.3) |
| **≥ 1536** (0x600) | **Type** | Nature du protocole de niveau supérieur (Ethernet II) |

**EtherTypes courants** :

| EtherType | Valeur hexa | Protocole |
|-----------|-------------|-----------|
| **IPv4** | `0x0800` | Internet Protocol version 4 |
| **ARP** | `0x0806` | Address Resolution Protocol |
| **RARP** | `0x8035` | Reverse ARP |
| **IPv6** | `0x86DD` | Internet Protocol version 6 |
| **802.1Q** | `0x8100` | VLAN tagging |
| **LLDP** | `0x88CC` | Link Layer Discovery Protocol |

> [!tip] En pratique
> La quasi-totalité des trames modernes utilisent **Ethernet II** (EtherType ≥ 1536).
> 
> Le format avec "longueur" (≤ 1500) est **obsolète**.

---

### Données transportées (Payload)

> [!important] Charge utile de la trame

**Rôle** :
Les données sont **véhiculées** par la trame.

**Encapsulation** :
- **Émission** : Encapsulation du PDU de la couche supérieure (ex. : paquet IP)
- **Réception** : PDU transmis au protocole indiqué par le champ **EtherType**

**Taille** :

| Paramètre | Valeur |
|-----------|--------|
| **Minimum** | 46 octets |
| **Maximum** | 1500 octets |
| **Nom du maximum** | **MTU** (Maximum Transmission Unit) |

> [!warning] Padding (Bourrage)
> Si la longueur de la payload est **inférieure à 46 octets**, des octets à **0** (padding) sont ajoutés pour compléter.
> 
> Raison : Détection des collisions sur Ethernet (historique CSMA/CD).

**MTU** :
- **MTU standard** : **1500 octets**
- **Jumbo frames** : Jusqu'à **9000 octets** (datacenters)

> [!example] Exemple de payload
> Un paquet IP complet (en-tête IP + en-tête TCP + données applicatives) est encapsulé dans la payload Ethernet.

---

### En-tête IP (dans la payload)

> [!info] Début de la payload

L'**en-tête IP** constitue le début de la payload (si EtherType = 0x0800).

**Contenu** :
- Adresse IP **source**
- Adresse IP **destination**
- TTL, protocole, etc.

**Positionnement** :
- **Modèle OSI** : Couche 3 (Réseau)
- **Modèle TCP/IP** : Couche 2 (Internet)

```
┌────────────────────────────────────────┐
│         Trame Ethernet (Couche 2)      │
│  ┌──────────────────────────────────┐  │
│  │   Paquet IP (Couche 3)           │  │
│  │ ┌──────────────────────────────┐ │  │
│  │ │ Segment TCP (Couche 4)       │ │  │
│  │ │ ┌──────────────────────────┐ │ │  │
│  │ │ │ Données Application (L7) │ │ │  │
│  │ │ └──────────────────────────┘ │ │  │
│  │ └──────────────────────────────┘ │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

---

### CRC Checksum (FCS)

> [!important] Détection d'erreurs

**Nom complet** : **FCS** (Frame Check Sequence) ou **CRC** (Cyclic Redundancy Check)

**Caractéristiques** :
- Champ de **4 octets** (32 bits)
- Calcul mathématique sur l'ensemble de la trame

**Rôle** :
S'assurer que la trame a été **correctement transmise** et que les données peuvent être délivrées au protocole destinataire.

**Fonctionnement** :

1. **Émission** :
   - Calcul du CRC sur l'ensemble de la trame
   - Ajout du CRC à la fin de la trame

2. **Réception** :
   - Recalcul du CRC sur la trame reçue
   - Comparaison avec le CRC reçu
   - Si **identiques** : Trame OK
   - Si **différents** : Trame corrompue → **détruite**

> [!warning] Best Effort
> Ethernet est un protocole **"best effort"** (meilleur effort) :
> 
> En cas d'erreur de transmission (CRC incorrect), la trame est **détruite** sans notification.
> 
> Les couches supérieures (TCP) gèrent la retransmission si nécessaire.

---

### Gap inter-trame

> [!info] Séparation des trames

**Problème** :
Les trames avec un EtherType ≥ 1536 (Ethernet II) sont de taille variable (46 à 1500 octets de payload). Comment détecter la fin d'une trame ?

**Solution** :
Ethernet prévoit un **"blanc"** appelé **Interframe Gap** (IFG) entre 2 trames.

**Caractéristiques** :
- Durée : Temps d'émission de **96 bits**
- Exemple à 1 Gbps : 96 ns (nanosecondes)
- Ce gap marque la **fin de la trame**

```
┌─────────────┐   IFG   ┌─────────────┐   IFG   ┌─────────────┐
│   Trame 1   │ (96 bits)│   Trame 2   │ (96 bits)│   Trame 3   │
└─────────────┘         └─────────────┘         └─────────────┘
                │                        │
                └─ Silence (pas de signal)
```

> [!note] Rôle
> - Permet aux équipements de **traiter** la trame reçue
> - Prépare la **réception** de la trame suivante
> - **Sépare** les trames sur le medium

---

### Structure complète de la trame

> [!success] Récapitulatif avec tailles

**Trame Ethernet complète** :

```
┌──────────┬──────────┬──────────┬──────────┬─────────────┬─────┐
│Préambule │ MAC Dest │ MAC Src  │EtherType │   Payload   │ FCS │
│          │          │          │          │             │     │
│  8 oct   │  6 oct   │  6 oct   │  2 oct   │ 46-1500 oct │ 4 o │
└──────────┴──────────┴──────────┴──────────┴─────────────┴─────┘
     │            │                                           │
     │            └─── En-tête MAC (14 octets) ──────────────┘
     │
     └─ Non compté dans taille trame (ajouté par couche physique)

Taille trame (sans préambule) : 64 à 1518 octets
Taille trame (avec préambule) : 72 à 1526 octets
```

**Détail des tailles** :

| Champ | Taille | Cumul |
|-------|--------|-------|
| Préambule + SFD | 8 octets | 8 |
| MAC Destination | 6 octets | 14 |
| MAC Source | 6 octets | 20 |
| EtherType | 2 octets | 22 |
| **Payload** (données) | **46 à 1500 octets** | 68 à 1522 |
| FCS (CRC) | 4 octets | 72 à 1526 |

**Limites** :
- **Taille minimale** : 64 octets (sans préambule) ou 72 octets (avec)
- **Taille maximale** : 1518 octets (sans préambule) ou 1526 octets (avec)
- **MTU standard** : 1500 octets (payload uniquement)

> [!tip] Mnémotechnique
> **1 octet = 1 byte = 8 bits**
> 
> Trame standard : **64 à 1518 octets** (hors préambule)

---

## Switch et VLAN

### Commutateur

> [!important] Équipement intelligent de niveau 2

**Définition** :
Un **commutateur** (switch) est un équipement de **niveau 2** (travaille avec les **trames**) - C'est un pont multiports.

**Rôle principal** :
Créer un **canal spécifique** pour chaque communication (commutation).

**Moyen** :
- Utilise les **adresses MAC**
- Construit une **table de correspondance** MAC ↔ ports (table CAM)

**Avantages** :
- ✅ Sépare les **domaines de collision** (un par port)
- ✅ Communication **point-à-point** efficace
- ✅ **Full-duplex** possible
- ✅ Bande passante **dédiée** par port

---

### Gestion des trames

> [!info] Algorithmes du switch

Le switch utilise **5 mécanismes** pour gérer les trames :

#### 1. Apprentissage (Learning)

**Fonctionnement** :
À l'arrivée d'une trame sur un port :
1. Le switch **regarde l'adresse MAC source**
2. **Enregistre** dans sa table MAC la relation : `Adresse MAC ↔ Port`

**Exemple** :
```
Trame reçue sur port 3 :
  MAC Src: AA:BB:CC:DD:EE:01
  
Table MAC mise à jour :
┌─────────────────────┬──────┐
│ Adresse MAC         │ Port │
├─────────────────────┼──────┤
│ AA:BB:CC:DD:EE:01   │  3   │ ← Nouvelle entrée
└─────────────────────┴──────┘
```

---

#### 2. Inondation (Flooding)

**Fonctionnement** :
À l'arrivée d'une trame dont la **destination est inconnue** (absente de la table MAC) :
- Envoi de la trame à **tous les ports** (sauf le port source)
- C'est un **broadcast** au niveau du switch

**Cas d'usage** :
- Première trame vers un nouvel hôte
- Table MAC vidée (aging)
- Broadcast Ethernet (`FF:FF:FF:FF:FF:FF`)

**Exemple** :
```
Trame reçue sur port 3 :
  MAC Dest: XX:YY:ZZ:11:22:33 (inconnu)
  
Action : Envoi sur TOUS les ports (1, 2, 4, 5, 6...) sauf 3
```

---

#### 3. Réexpédition (Forwarding)

**Fonctionnement** :
À l'arrivée d'une trame dont la **destination est connue** :
- La trame est envoyée **uniquement sur le port destinataire**
- **Principe de la commutation** (différent du hub qui broadcast tout)

**Exemple** :
```
Table MAC :
┌─────────────────────┬──────┐
│ Adresse MAC         │ Port │
├─────────────────────┼──────┤
│ AA:BB:CC:DD:EE:01   │  3   │
│ 11:22:33:44:55:66   │  5   │
└─────────────────────┴──────┘

Trame reçue sur port 3 :
  MAC Dest: 11:22:33:44:55:66
  
Action : Envoi UNIQUEMENT sur port 5
```

---

#### 4. Filtrage (Filtering)

**Fonctionnement** :
Si l'adresse **source** et **destination** sont sur le **même port** :
- **Pas de transmission** de la trame
- La communication est locale au segment

**Cas d'usage** :
- Un port est relié à un **hub** (plusieurs hôtes sur le même port)
- Communication entre deux hôtes du même hub

**Exemple** :
```
Table MAC :
┌─────────────────────┬──────┐
│ Adresse MAC         │ Port │
├─────────────────────┼──────┤
│ AA:BB:CC:DD:EE:01   │  3   │
│ AA:BB:CC:DD:EE:02   │  3   │ ← Même port
└─────────────────────┴──────┘

Trame de EE:01 vers EE:02 :
  
Action : Pas de transmission (déjà sur le même segment)
```

---

#### 5. Vieillissement (Aging)

**Fonctionnement** :
- La table MAC est **vidée cycliquement** pour chaque entrée
- Disparition des entrées **sans activité** après un délai (timeout)
- Permet de libérer la mémoire (table limitée)

**Raison** :
- Mémoire **limitée** du switch
- Adaptation aux **changements** de topologie (hôte déplacé)

**Timeout typique** : **300 secondes** (5 minutes)

**Exemple** :
```
Table MAC (5 minutes sans activité) :
┌─────────────────────┬──────┬────────────┐
│ Adresse MAC         │ Port │ Âge        │
├─────────────────────┼──────┼────────────┤
│ AA:BB:CC:DD:EE:01   │  3   │ 30 sec     │ ← OK
│ 11:22:33:44:55:66   │  5   │ 310 sec    │ ← SUPPRIMÉ
└─────────────────────┴──────┴────────────┘
```

---

### Autres fonctions d'un commutateur

> [!tip] Fonctionnalités avancées

Les switches modernes offrent de nombreuses fonctionnalités avancées :

| Fonction | Norme | Description |
|----------|-------|-------------|
| **Empilage** (Stack) | - | Regrouper plusieurs switches en un seul logique |
| **Agrégation de liens** | LACP (802.3ad) | Regrouper plusieurs ports pour augmenter bande passante |
| **SNMP/SMON** | - | Protocole de supervision réseau |
| **Redondance** | SPB (802.1aq), STP | Gestion de la redondance (boucles) |
| **Port mirroring** | - | Duplication de trafic (pour analyse) |
| **Authentification** | 802.1X | Contrôle d'accès au réseau (NAC) |
| **Filtrage** | ACL | Listes de contrôle d'accès |
| **VLAN** | 802.1Q | Cloisonnement logique (voir section suivante) |
| **QoS** | 802.1p | Support de la qualité de service (priorisation) |
| **PoE** | 802.3af/at/bt | Alimentation électrique via câble Ethernet |
| **Routage** (L3) | - | Routage IP (switch multicouche) |

> [!example] Cas d'usage TSSR
> - **PME** : VLAN, QoS, PoE
> - **Entreprise** : + Empilage, Agrégation, Redondance, 802.1X
> - **Datacenter** : + Routage L3, Port mirroring, QoS avancé

---

### VLAN (Réseaux locaux virtuels)

> [!important] Segmentation logique du réseau

**Définition** :
Les **VLAN** (Virtual LAN) permettent de **segmenter** un réseau Ethernet :

**Objectifs** :
- Séparer les **domaines de diffusion** (broadcast)
- Les hôtes sur des VLANs différents **ne peuvent plus communiquer** directement
- Sans avoir besoin d'avoir des **réseaux physiques séparés**

**Principe** :
Sur un switch, on affecte un ensemble de **ports** à un **VLAN**.

---

#### Avantages des VLANs

> [!success] Bénéfices

**Sécurité** :
- ✅ **Isolation** du trafic entre services
- ✅ Limitation de la propagation (broadcast, malware)

**Organisation** :
- ✅ Regroupement **logique** par fonction (compta, RH, production)
- ✅ Indépendant de la **localisation physique**

**Performance** :
- ✅ Réduction des **domaines de broadcast**
- ✅ Optimisation du trafic

**Flexibilité** :
- ✅ Reconfiguration **sans recâblage**
- ✅ Gestion centralisée

> [!example] Cas d'usage typique
> **Entreprise** :
> - VLAN 10 : Direction
> - VLAN 20 : Comptabilité
> - VLAN 30 : Production
> - VLAN 40 : Invités (WiFi)
> 
> Chaque VLAN = réseau IP distinct (ex: 192.168.10.0/24, 192.168.20.0/24, etc.)

---

### IEEE 802.1Q

> [!important] Standard de tagging VLAN

**Problème** :
Comment transmettre l'information de VLAN **entre switches** ?

**Solution** :
Utiliser **IEEE 802.1Q** - Standard de **tagging VLAN**.

**Principe** :
Ajouter un **en-tête supplémentaire** (4 octets) dans la trame Ethernet.

---

#### Structure du tag 802.1Q

> [!info] En-tête VLAN

**Position** :
Inséré **après l'en-tête Ethernet** standard (après MAC source, avant EtherType).

**Composition** (4 octets) :

| Champ | Taille | Valeur | Description |
|-------|--------|--------|-------------|
| **TPID** | 2 octets | `0x8100` | Tag Protocol Identifier (EtherType de 802.1Q) |
| **TCI** | 2 octets | Variable | Tag Control Information |

**TCI décomposé** :

| Sous-champ | Bits | Description |
|------------|------|-------------|
| **PCP** | 3 bits | Priority Code Point (QoS - 802.1p) |
| **DEI** | 1 bit | Drop Eligible Indicator |
| **VID** | 12 bits | **VLAN Identifier** (0-4095) |

**Structure complète** :
```
┌──────────┬──────────┬──────────┬──────────┬──────────┬─────────────┬─────┐
│ MAC Dest │ MAC Src  │   TPID   │   TCI    │EtherType │   Payload   │ FCS │
│  (6 o)   │  (6 o)   │  (2 o)   │  (2 o)   │  (2 o)   │ (46-1500 o) │(4 o)│
│          │          │  0x8100  │  VLAN ID │          │             │     │
└──────────┴──────────┴──────────┴──────────┴──────────┴─────────────┴─────┘
                          │                     │
                          └─ Tag 802.1Q (4 o) ──┘
```

> [!warning] Taille de trame
> Avec le tag 802.1Q, la trame Ethernet fait **4 octets de plus** :
> - Sans tag : 64-1518 octets
> - Avec tag : 68-1522 octets

---

#### VLAN inter-switch

> [!note] Trunk ports

**Principe** :
Les trames **802.1Q** ne circulent que sur les **liens inter-switch** (trunk).

**Fonctionnement** :

**À l'émission** (switch A) :
1. Le switch **ajoute** le tag 802.1Q en fonction du port émetteur (et du VLAN auquel il appartient)
2. **Recalcule** le CRC (FCS)
3. Transmet sur le trunk

**À la réception** (switch B) :
1. Le switch **lit** le tag 802.1Q
2. **Retire** l'en-tête 802.1Q
3. **Recalcule** le CRC
4. Transmet à la destination (selon le VLAN)

**Configuration des ports** :

| Type de port | Mode | Fonction |
|--------------|------|----------|
| **Access** | Sans tag | Port pour hôte (PC, serveur) - appartient à UN VLAN |
| **Trunk** | Avec tag | Port inter-switch - transporte TOUS les VLANs |

```
┌─────────────┐ Trunk 802.1Q ┌─────────────┐
│  Switch A   │◄─────────────►│  Switch B   │
│             │  (Tagged)     │             │
│ Port 1 ─────┤               ├───── Port 1 │
│ (VLAN 10)   │               │   (VLAN 10) │
│             │               │             │
│ Port 2 ─────┤               ├───── Port 2 │
│ (VLAN 20)   │               │   (VLAN 20) │
└─────────────┘               └─────────────┘
   Access                          Access
  (Untagged)                     (Untagged)
```

---

### Autres extensions notables

> [!tip] Technologies complémentaires

**Autonégociation** :
- Apparu avec **Fast Ethernet** (100 Mbps)
- Les cartes **négocient** automatiquement :
  - Débit (10/100/1000 Mbps)
  - Duplex (half/full)

**Power over Ethernet (PoE)** :
- Permet l'**alimentation électrique** via les câbles Ethernet
- Standards :
  - **802.3af** (PoE) : 15,4W
  - **802.3at** (PoE+) : 30W
  - **802.3bt** (PoE++) : 60W / 100W
- Usages : Téléphones IP, caméras, points d'accès WiFi

**Jumbo Frames** :
- Augmentation du **MTU** de 1500 jusqu'à **9000 octets**
- Avantages :
  - Réduction de l'overhead (moins de trames)
  - Augmentation des performances (datacenters)
- Inconvénient :
  - Nécessite support par tous les équipements

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définition Ethernet

1. **Ethernet** = Protocole de réseau **local filaire** (LAN)
2. **Norme** : IEEE 802.3
3. **Débits** : 10 Mbps à 400 Gbps (objectif 1,6 Tbps)
4. **Standard** : Protocole le plus utilisé pour réseaux locaux

### Architecture IEEE

```
┌──────────────────────────────┐
│  LLC (802.2) - Commune       │
├──────────────────────────────┤  Couche 2
│  MAC (802.3) - Spécifique    │
├──────────────────────────────┤
│  PHY - Physique              │  Couche 1
└──────────────────────────────┘
```

### Normes PHY

**Format** : `<Débit>BASE-<Type>`
- Exemple : **1000BASE-T** = 1 Gbps, Bande de base, Paires torsadées

### Câblage

**Paires torsadées** :
- **CAT 5e** : 1 Gbps, 100m (standard actuel)
- **CAT 6** : 10 Gbps, 55m (recommandé)
- **CAT 6a** : 10 Gbps, 100m (professionnel)

**Fibre optique** :
- **Multimode** : 300m-2km, moins cher (datacenters)
- **Monomode** : 10-100 km, plus cher (backbone)

**Blindage** :
- **U/UTP** : Non blindé (résidentiel)
- **F/UTP** : Blindage global (bureaux)
- **S/FTP** : Blindage optimal (industriel)

### Câbles droits vs croisés

| Type | Usage |
|------|-------|
| **Droit (MDI)** | PC → Switch |
| **Croisé (MDI-X)** | PC → PC, Switch → Switch |

> **Auto MDI/MDIX** rend les câbles croisés obsolètes

### Adresse MAC

**Format** : 48 bits (6 octets) en hexadécimal
- Exemple : `D4:93:90:05:2C:1C`
- Structure : **OUI** (3 octets constructeur) + **NIC** (3 octets unique)
- Bits spéciaux :
  - Bit 7 : U/L (Universelle/Locale)
  - Bit 8 : I/G (Individuelle/Groupe)
- Broadcast : `FF:FF:FF:FF:FF:FF`

**Consultation** :
- Linux : `ip link show` ou `ip l`
- Windows : `Get-NetAdapter` ou `ipconfig /all`

### Trame Ethernet

**Structure** (Ethernet II) :
```
┌──────────┬──────────┬──────────┬──────────┬─────────────┬─────┐
│Préambule │ MAC Dest │ MAC Src  │EtherType │   Payload   │ FCS │
│  8 oct   │  6 oct   │  6 oct   │  2 oct   │ 46-1500 oct │ 4 o │
└──────────┴──────────┴──────────┴──────────┴─────────────┴─────┘

Taille : 64 à 1518 octets (sans préambule)
```

**Composants clés** :

| Champ | Taille | Rôle |
|-------|--------|------|
| Préambule + SFD | 8 o | Synchronisation |
| MAC Destination | 6 o | Adresse destinataire |
| MAC Source | 6 o | Adresse expéditeur |
| EtherType | 2 o | Type protocole (0x0800 = IPv4) |
| Payload | 46-1500 o | Données (MTU = 1500) |
| FCS (CRC) | 4 o | Détection d'erreurs |

**MTU** : Maximum Transmission Unit = **1500 octets**

### Switch (Commutateur)

**Niveau** : Couche 2 (Liaison)

**5 mécanismes** :
1. **Learning** : Apprentissage MAC ↔ Port
2. **Flooding** : Envoi à tous si destination inconnue
3. **Forwarding** : Envoi au port destinataire
4. **Filtering** : Pas de transmission si même port
5. **Aging** : Vieillissement table MAC (~300s)

### VLAN

**Définition** : Virtual LAN = Segmentation logique du réseau

**Avantages** :
- ✅ Sécurité (isolation)
- ✅ Organisation (par service)
- ✅ Performance (réduction broadcast)
- ✅ Flexibilité (sans recâblage)

**IEEE 802.1Q** :
- Tag de **4 octets** (TPID + TCI)
- TPID = `0x8100`
- VID = 12 bits (4096 VLANs possibles)
- Circule sur **trunk** (inter-switch)
- Retiré sur **access** (vers hôte)

### EtherTypes courants

| EtherType | Protocole |
|-----------|-----------|
| `0x0800` | IPv4 |
| `0x0806` | ARP |
| `0x86DD` | IPv6 |
| `0x8100` | 802.1Q (VLAN) |

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Access port** | Port switch en mode non-tagué, appartient à un VLAN |
| **Aging** | Vieillissement des entrées de table MAC |
| **ARP** | Address Resolution Protocol - Résolution IP → MAC |
| **Auto MDI/MDIX** | Détection et croisement automatique des paires TX/RX |
| **Broadcast** | Diffusion à tous les hôtes (MAC: FF:FF:FF:FF:FF:FF) |
| **CAM table** | Content Addressable Memory - Table MAC du switch |
| **CAT 5e/6/6a** | Catégories de câbles paires torsadées |
| **Coaxial** | Câble à deux conducteurs (obsolète pour Ethernet) |
| **CRC** | Cyclic Redundancy Check - Détection d'erreurs |
| **CSMA/CD** | Carrier Sense Multiple Access / Collision Detection |
| **Domaine de collision** | Zone où les trames peuvent entrer en collision |
| **Domaine de broadcast** | Zone où les broadcasts sont propagés |
| **EtherType** | Champ identifiant le protocole de niveau supérieur |
| **Ethernet** | Protocole LAN filaire (IEEE 802.3) |
| **FCS** | Frame Check Sequence - Somme de contrôle (4 octets) |
| **Filtering** | Pas de transmission si src et dest sur même port |
| **Flooding** | Envoi à tous les ports (destination inconnue) |
| **Forwarding** | Envoi au port destinataire (destination connue) |
| **Full-duplex** | Émission et réception simultanées |
| **Half-duplex** | Émission ou réception (alternée) |
| **Hub** | Concentrateur (couche 1, obsolète) |
| **IEEE 802.3** | Norme Ethernet |
| **IEEE 802.1Q** | Norme VLAN tagging |
| **Interframe Gap (IFG)** | Blanc de 96 bits entre trames |
| **Jumbo Frame** | Trame avec MTU > 1500 octets (jusqu'à 9000) |
| **LACP** | Link Aggregation Control Protocol (802.3ad) |
| **Learning** | Apprentissage MAC ↔ Port par le switch |
| **LLC** | Logical Link Control - Sous-couche liaison commune IEEE |
| **MAC** | Medium Access Control - Sous-couche liaison spécifique |
| **Adresse MAC** | Adresse physique 48 bits (6 octets hexa) |
| **MDI** | Medium Dependent Interface - Câble droit |
| **MDI-X** | MDI Crossover - Câble croisé |
| **Monomode (SMF)** | Fibre optique longue distance (9 µm) |
| **MTU** | Maximum Transmission Unit - Taille max payload (1500) |
| **Multimode (MMF)** | Fibre optique courte distance (50/62.5 µm) |
| **NIC** | Network Interface Card - Carte réseau |
| **OUI** | Organizationally Unique Identifier - Préfixe constructeur MAC |
| **Padding** | Octets de bourrage (si payload < 46 octets) |
| **Paire torsadée** | Deux fils enroulés en hélice (limitation interférences) |
| **Payload** | Charge utile de la trame (données) |
| **PHY** | Physical Layer - Couche physique |
| **PoE** | Power over Ethernet - Alimentation via câble |
| **Préambule** | 8 octets de synchronisation (début trame) |
| **QoS** | Quality of Service - Qualité de service (802.1p) |
| **RJ45** | Connecteur 8P8C pour câbles paires torsadées |
| **SFD** | Starting Frame Delimiter - Délimiteur début trame |
| **SFP** | Small Form-factor Pluggable - Module fibre optique |
| **Switch** | Commutateur (couche 2, pont multiports) |
| **TPID** | Tag Protocol Identifier - Identifiant 802.1Q (0x8100) |
| **Trame** | PDU de la couche liaison (Ethernet) |
| **Trunk port** | Port switch en mode tagué 802.1Q (inter-switch) |
| **U/UTP** | Unshielded/Unshielded Twisted Pair - Non blindé |
| **F/UTP** | Foiled/Unshielded TP - Blindage global |
| **S/FTP** | Shielded/Foiled TP - Blindage tresse + paires |
| **VID** | VLAN Identifier - Numéro de VLAN (12 bits, 0-4095) |
| **VLAN** | Virtual LAN - Réseau local virtuel (segmentation) |

---

**Document créé le** : 21 novembre 2025  
**Version** : 1.0  
**Source** : Cours "Ethernet - Bien débuter" - Formation TSSR

> [!success] ✅ BON COURAGE POUR VOTRE TITRE RNCP TSSR !

---
