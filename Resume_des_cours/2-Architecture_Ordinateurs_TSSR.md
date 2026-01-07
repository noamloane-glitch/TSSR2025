# Architecture des ordinateurs
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Architecture des ordinateurs - Notions de base  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Définition et concepts fondamentaux|Définition et concepts fondamentaux]]
   - [[#Qu'est-ce qu'un ordinateur ?|Qu'est-ce qu'un ordinateur ?]]
   - [[#Architecture de base|Architecture de base]]
   - [[#Architecture de Von Neumann|Architecture de Von Neumann]]
3. [[#Unité et codage de l'information|Unité et codage de l'information]]
   - [[#Le bit et l'octet|Le bit et l'octet]]
   - [[#Le binaire|Le binaire]]
4. [[#Les composants principaux|Les composants principaux]]
   - [[#Le CPU (Central Processing Unit)|Le CPU (Central Processing Unit)]]
   - [[#La RAM (Random Access Memory)|La RAM (Random Access Memory)]]
   - [[#La hiérarchie des mémoires|La hiérarchie des mémoires]]
   - [[#La carte mère|La carte mère]]
   - [[#Le firmware (BIOS/UEFI)|Le firmware (BIOS/UEFI)]]
   - [[#Le boîtier et l'alimentation|Le boîtier et l'alimentation]]
5. [[#Le stockage|Le stockage]]
   - [[#Disques durs (HDD)|Disques durs (HDD)]]
   - [[#Disques SSD|Disques SSD]]
   - [[#Secteurs et adressage|Secteurs et adressage]]
   - [[#Partitionnement|Partitionnement]]
   - [[#MBR (Master Boot Record)|MBR (Master Boot Record)]]
   - [[#GPT (GUID Partition Table)|GPT (GUID Partition Table)]]
6. [[#Les périphériques d'entrée/sortie|Les périphériques d'entrée/sortie]]
   - [[#Périphériques d'entrée|Périphériques d'entrée]]
   - [[#Périphériques de sortie|Périphériques de sortie]]
7. [[#Points clés à retenir|Points clés à retenir]]
8. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> L'architecture des ordinateurs désigne l'organisation structurelle et fonctionnelle des différents composants d'un système informatique. Cette connaissance est fondamentale pour tout technicien systèmes et réseaux.

### Pourquoi étudier l'architecture des ordinateurs ?

En tant que **TSSR**, tu dois :
- **Comprendre** le fonctionnement interne d'un ordinateur
- **Diagnostiquer** les problèmes matériels
- **Optimiser** les performances système
- **Choisir** les composants adaptés aux besoins
- **Maintenir** et dépanner les équipements informatiques

Cette connaissance est essentielle pour le titre RNCP TSSR et pour ta pratique professionnelle quotidienne.

---

## Définition et concepts fondamentaux

### Qu'est-ce qu'un ordinateur ?

> [!quote] Définition
> Un ordinateur est une **machine électronique**, **numérique** et **programmable** capable d'effectuer des **opérations arithmétiques de base**.

**Caractéristiques principales** :

| Caractéristique | Description |
|-----------------|-------------|
| **Électronique** | Fonctionne avec des circuits électroniques (transistors, circuits intégrés) |
| **Numérique** | Traite des informations sous forme binaire (0 et 1) |
| **Programmable** | Exécute des instructions stockées en mémoire (logiciels) |
| **Arithmétique** | Effectue des calculs et opérations logiques |

> [!info] Source
> Définition inspirée de Wikipedia - Ordinateur

### Architecture de base

> [!important] Modèle conceptuel fondamental

Un ordinateur est composé de **4 éléments fondamentaux** :

#### 1. UAL (Unité Arithmétique et Logique)

> [!quote] Rôle
> **Effectue les opérations** arithmétiques (addition, soustraction, multiplication, division) et logiques (ET, OU, NON, XOR).

**Fonctions** :
- Calculs mathématiques
- Opérations logiques booléennes
- Comparaisons
- Décalages binaires

#### 2. Unité de contrôle

> [!quote] Rôle
> **Séquence les opérations** - dirige et coordonne l'exécution des instructions.

**Fonctions** :
- Lecture des instructions en mémoire
- Décodage des instructions
- Coordination des autres unités
- Gestion du flux d'exécution

#### 3. Mémoire

> [!quote] Rôle
> **Stocke les données et les programmes** de manière temporaire ou permanente.

**Types** :
- Mémoire vive (RAM) : volatile, rapide
- Mémoire de masse (disques) : permanente, plus lente
- Mémoires caches : très rapides, petites

#### 4. Entrées/Sorties (I/O - Input/Output)

> [!quote] Rôle
> **Assure la communication avec l'extérieur** - permet d'échanger des données avec l'utilisateur et les périphériques.

**Exemples** :
- Entrées : clavier, souris, microphone
- Sorties : écran, imprimante, haut-parleurs
- Entrée/Sortie : carte réseau, disque dur

### Architecture de Von Neumann

> [!important] Modèle fondamental de l'informatique moderne

L'architecture des ordinateurs actuels repose sur deux modèles théoriques :

1. **Architecture de Von Neumann** (1945)
2. **Machine de Turing** (modèle théorique)

#### Schéma de l'architecture de Von Neumann

```
┌─────────────────────────────────────────────┐
│              MÉMOIRE                        │
│   (Données + Instructions)                  │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────────┐    ┌───────▼─────┐
│  Unité de  │◄───┤     UAL     │
│  Contrôle  │───►│  (Calculs)  │
└────────────┘    └─────────────┘
    │                     │
    └──────────┬──────────┘
               │
    ┌──────────┴──────────┐
    │                     │
┌───▼────┐           ┌────▼────┐
│ Entrée │           │ Sortie  │
│  (I)   │           │   (O)   │
└────────┘           └─────────┘
```

> [!info] Principe clé de Von Neumann
> Les **instructions** et les **données** sont stockées dans la **même mémoire**. C'est le concept de "programme enregistré" (stored program).

**Avantages** :
- Simplicité conceptuelle
- Flexibilité (modification des programmes)
- Universalité (une même machine peut tout faire)

**Limitation** :
- **Goulot d'étranglement de Von Neumann** : le CPU et la mémoire partagent le même bus de communication, créant un point de congestion.

---

## Unité et codage de l'information

### Le bit et l'octet

#### Le bit (Binary Digit)

> [!quote] Définition
> Le **bit** est l'**unité élémentaire** de stockage et de mesure de l'information en informatique.

**Caractéristiques** :
- **Binaire** : seulement **2 valeurs possibles** → `0` ou `1`
- Représente un état électrique : tension basse (0) ou haute (1)
- Plus petite unité d'information

> [!example] Représentation physique
> - Transistor : bloqué (0) ou passant (1)
> - Signal électrique : bas (0) ou haut (1)
> - Magnétisme : non magnétisé (0) ou magnétisé (1)

#### L'octet (Byte)

> [!quote] Définition
> Un **octet** est un groupe de **8 bits** consécutifs.

**Calcul du nombre de valeurs** :

```
1 bit  → 2 valeurs (2¹)
2 bits → 4 valeurs (2²)
3 bits → 8 valeurs (2³)
...
8 bits → 256 valeurs (2⁸)
```

> [!important] Formule générale
> **Nombre de valeurs = 2^n** où n = nombre de bits

**Pour un octet (8 bits)** :
- 2 × 2 × 2 × 2 × 2 × 2 × 2 × 2 = **2⁸ = 256 valeurs**
- Plage : **0 à 255** (en décimal)

> [!example] Exemple d'octet

```
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑                           ↑
Bit de poids fort         Bit de poids faible
   (MSB)                        (LSB)
```

### Le binaire

#### Identifier un nombre binaire vs un octet

> [!example] Exercice pratique

| Nombre | Type | Explication |
|--------|------|-------------|
| `10101010` | **Octet** | 8 bits → c'est un octet complet |
| `00001111` | **Octet** | 8 bits → c'est un octet complet |
| `-1` | **Ni l'un ni l'autre** | Les nombres négatifs ne sont pas des nombres binaires bruts |
| `0` | **Nombre binaire** | 1 bit → c'est un nombre binaire (pas un octet) |
| `00000000` | **Octet** | 8 bits → c'est un octet complet |
| `101` | **Nombre binaire** | 3 bits → nombre binaire (pas assez pour un octet) |
| `11111111` | **Octet** | 8 bits → c'est un octet complet |

> [!tip] Règle simple
> - **Octet** = exactement 8 bits
> - **Nombre binaire** = 1 bit ou plus (mais pas forcément 8)
> - Si ce n'est pas composé uniquement de 0 et 1, ce n'est pas binaire

#### Unités de mesure de l'information

> [!info] Échelle des unités

| Unité | Abréviation | Valeur (base 2) | Valeur approx. |
|-------|-------------|-----------------|----------------|
| **Bit** | b | 1 bit | 1 bit |
| **Octet** | o (ou B) | 8 bits | 8 bits |
| **Kibioctet** | Kio (KiB) | 2¹⁰ octets | ≈ 1 000 octets |
| **Mébioctet** | Mio (MiB) | 2²⁰ octets | ≈ 1 000 000 octets |
| **Gibioctet** | Gio (GiB) | 2³⁰ octets | ≈ 1 000 000 000 octets |
| **Tébioctet** | Tio (TiB) | 2⁴⁰ octets | ≈ 1 000 000 000 000 octets |
| **Pébioctet** | Pio (PiB) | 2⁵⁰ octets | ≈ 1 000 000 000 000 000 octets |
| **Exbioctet** | Eio (EiB) | 2⁶⁰ octets | - |
| **Zébioctet** | Zio (ZiB) | 2⁷⁰ octets | - |

> [!warning] Confusion courante
> Ne pas confondre :
> - **Ki/Mi/Gi** (base 2 : 1024) - notation IEC standard
> - **k/M/G** (base 10 : 1000) - notation SI
> 
> Exemple : 1 Kio = 1024 octets ≠ 1 Ko = 1000 octets

> [!note] Ressources
> Pour plus d'informations, voir Wikipedia : **Bit** et **Octet**

---

## Les composants principaux

### Le CPU (Central Processing Unit)

> [!quote] Le cerveau de l'ordinateur
> Le **CPU** (Central Processing Unit) ou **processeur** est l'**unité de calcul** centrale de l'ordinateur.

#### Composition du CPU

> [!important] Circuit intégré regroupant

Un CPU moderne est un **circuit intégré** complexe regroupant :

| Composant | Fonction |
|-----------|----------|
| **UAL** | Effectue les calculs arithmétiques et logiques |
| **Unité de contrôle** | Orchestre l'exécution des instructions |
| **Horloge** | Synchronise les opérations (fréquence en Hz) |
| **Registres** | Mémoire ultra-rapide interne au CPU |
| **Mémoires caches** | Mémoires tampons rapides (L1, L2, L3) |

#### Caractéristiques d'un CPU

> [!info] Un processeur est caractérisé par

**1. Jeu d'instructions (ISA - Instruction Set Architecture)**

C'est l'ensemble des instructions que le CPU peut exécuter.

**Jeux d'instructions courants** :
- **x86-64** (ou AMD64) : PC classiques, serveurs
- **ARM** (ARMv7, ARMv8) : smartphones, tablettes, Raspberry Pi
- **RISC-V** : architecture ouverte émergente

**2. Fréquence (vitesse d'horloge)**

Mesurée en **Hz** (Hertz) - nombre de cycles par seconde.

- **Anciens CPU** : quelques MHz (mégahertz)
- **CPU modernes** : 1 à 5 GHz (gigahertz)
- 1 GHz = 1 milliard de cycles par seconde

> [!tip] Plus de fréquence = plus rapide ?
> Pas toujours ! L'architecture, le nombre de cœurs, les caches, et l'efficacité par cycle comptent aussi.

**3. Taille des registres généraux (architecture)**

Détermine la quantité de données que le CPU peut manipuler en une opération.

- **32 bits** : peut adresser jusqu'à 4 Gio de RAM
- **64 bits** : peut adresser jusqu'à 16 Eio de RAM (théorique)

> [!example] Photo
> Photo : Priwo sur Wikimedia Commons

#### Exemples concrets de CPU

> [!example] Processeur PC classique (Intel/AMD)

**Caractéristiques typiques** :
- **Cœurs** : 1 à 16+ cores (voire 32+ pour serveurs)
- **Architecture** : 64 bits (registres d'adresse 64 bits)
- **Fréquence** : 1 à 5 GHz
- **Jeu d'instructions** : x86-64
- **Caches** :
  - **L1** : 10aines de Kio (par core)
  - **L2** : 100aines de Kio (par core)
  - **L3** : 10aines de Mio (partagé entre cores)

> [!example] Raspberry Pi 4

**Caractéristiques** :
- **Cœurs** : 4 cores
- **Architecture** : 64 bits
- **Fréquence** : 1,5 GHz
- **Jeu d'instructions** : ARMv8
- **Caches** :
  - **L1** : 80 Kio (par core)
  - **L2** : 1 Mio

> [!tip] Choix d'un CPU
> Pour choisir un CPU, considère :
> - **Usage** : bureautique, gaming, serveur, embarqué
> - **Budget** : équilibre performance/prix
> - **Compatibilité** : socket de la carte mère
> - **Consommation** : TDP (Thermal Design Power)

### La RAM (Random Access Memory)

> [!quote] La mémoire vive temporaire
> La **RAM** (Random Access Memory) est la **mémoire vive** de l'ordinateur, utilisée pour stocker temporairement les données et programmes en cours d'exécution.

#### Caractéristiques de la RAM

**Principe de fonctionnement** :

| Caractéristique | Description |
|-----------------|-------------|
| **Accès direct (Random Access)** | Contrairement à un accès séquentiel, n'importe quelle adresse peut être lue instantanément |
| **Volatile** | Le contenu est perdu à l'extinction de l'ordinateur |
| **Temporaire** | Stocke les données et programmes **en cours d'utilisation** |
| **Rapide** | Beaucoup plus rapide que les disques durs/SSD |

> [!important] RAM ≠ Stockage permanent
> - **RAM** : mémoire temporaire, volatile, rapide
> - **Disque dur/SSD** : mémoire de masse, permanente, plus lente

#### Fonctionnement conceptuel

> [!info] Idée générale

```
Mémoire RAM = Séquence d'octets numérotés

┌─────────┬──────────┐
│ Adresse │  Valeur  │
│ (index) │ (octet)  │
├─────────┼──────────┤
│ 0x0000  │ 10101010 │
│ 0x0001  │ 11110000 │
│ 0x0002  │ 00001111 │
│ 0x0003  │ 11001100 │
│   ...   │   ...    │
└─────────┴──────────┘
```

**Principe** :
- Chaque **adresse** correspond à un **mot mémoire** (généralement 1 octet)
- Le CPU peut lire/écrire n'importe quelle adresse directement
- Temps d'accès constant, peu importe l'adresse

#### Types de RAM

| Type | Description | Usage |
|------|-------------|-------|
| **SDRAM** | Synchronous Dynamic RAM | Ancienne génération |
| **DDR** | Double Data Rate | Standard actuel |
| **DDR2/3/4/5** | Évolutions avec vitesses croissantes | DDR4 courant, DDR5 récent |

> [!example] Photo
> Photo : Ryse93 sur Wikimedia Commons

### La hiérarchie des mémoires

> [!important] Rapport taille / coût / vitesse

Les mémoires d'un ordinateur sont organisées en **hiérarchie** selon trois critères inversement proportionnels :

```
┌─────────────────────────────────────┐
│         REGISTRES CPU               │  ← Plus rapide, plus cher, plus petit
│      (quelques octets)              │
├─────────────────────────────────────┤
│      CACHE L1 (par core)            │
│        (quelques Kio)               │
├─────────────────────────────────────┤
│      CACHE L2 (par core)            │
│      (centaines de Kio)             │
├─────────────────────────────────────┤
│      CACHE L3 (partagé)             │
│      (quelques Mio)                 │
├─────────────────────────────────────┤
│           RAM                       │
│     (quelques Gio)                  │
├─────────────────────────────────────┤
│        SSD / HDD                    │
│       (centaines Gio - Tio)         │
└─────────────────────────────────────┘  ← Moins rapide, moins cher, plus grand
```

#### Tableau récapitulatif des vitesses

| Niveau | Taille typique | Vitesse relative | Temps d'accès |
|--------|----------------|------------------|---------------|
| **Registres** | Données unitaires | Vitesse du CPU | < 1 ns |
| **Cache L1** | Quelques Kio | Vitesse CPU / 2 à 4 | 1-4 ns |
| **Cache L2** | Centaines Kio | Vitesse L1 / 4 | 10-20 ns |
| **Cache L3** | Quelques Mio | Vitesse L2 / 4 | 40-75 ns |
| **RAM** | Quelques Gio | Vitesse CPU / 100 | 50-100 ns |
| **SSD** | Centaines Gio - Tio | - | 0,1 ms (100 000 ns) |
| **HDD** | Plusieurs Tio | - | 10 ms (10 000 000 ns) |

> [!tip] Principe de localité
> Cette hiérarchie fonctionne grâce au **principe de localité** :
> - **Localité temporelle** : données récemment utilisées seront probablement réutilisées
> - **Localité spatiale** : données proches en mémoire seront probablement utilisées ensemble

> [!example] Analogie
> Imagine une bibliothèque :
> - **Registres** : le livre que tu lis actuellement (sur ton bureau)
> - **Caches** : les livres à portée de main (étagère proche)
> - **RAM** : les livres dans ta bibliothèque personnelle
> - **SSD/HDD** : les livres dans la bibliothèque municipale

### La carte mère

> [!quote] One motherboard to connect them all
> La **carte mère** (motherboard) est le composant central qui **interconnecte tous les autres composants** de l'ordinateur.

#### Composants et caractéristiques

> [!important] Éléments principaux d'une carte mère

**1. Format**

Standards de taille et de fixation :
- **ATX** : format standard (305 × 244 mm)
- **microATX** : format réduit (244 × 244 mm)
- **Mini-ITX** : très compact (170 × 170 mm)
- **E-ATX** : étendu pour stations de travail

**2. Chipset et socket processeur**

- **Chipset** : ensemble de circuits gérant les communications
- **Socket** : emplacement pour le(s) processeur(s)
  - Intel : LGA 1700, LGA 1200, etc.
  - AMD : AM5, AM4, TR4, etc.

**3. Connecteurs mémoire**

- **Nombre de slots** : 2, 4, 8, etc.
- **Type** : DDR4, DDR5
- **Capacité maximale** : dépend du chipset et du CPU

**4. Connecteurs pour cartes d'extension**

- **PCI Express** (PCIe) :
  - x16 : carte graphique
  - x8, x4, x1 : autres cartes (réseau, son, capture, etc.)

**5. Connecteurs de stockage**

- **M.2** : SSD compacts (NVMe ou SATA)
- **SATA** : disques durs et SSD 2,5"/3,5"
- **U.2** : SSD entreprise (rare)

**6. Périphériques E/S intégrés**

- **Cartes réseau** : Ethernet (1 Gbps, 2.5 Gbps, 10 Gbps)
- **Carte son** : audio intégré
- **WiFi/Bluetooth** : connectivité sans fil
- **USB** : contrôleurs USB 2.0, 3.0, 3.1, 3.2, USB-C

**7. Connecteurs d'alimentation**

- **ATX 24 pins** : alimentation principale
- **EPS 4+4 pins ou 8 pins** : alimentation CPU
- **Ventilateurs** : PWM ou DC

> [!example] Photo
> Photo : Smial sur Wikimedia Commons

> [!tip] Choix d'une carte mère
> Critères de sélection :
> - **Compatibilité CPU** : socket approprié
> - **Chipset** : fonctionnalités désirées
> - **RAM** : type et capacité maximale
> - **Extension** : nombre de slots PCIe, M.2
> - **Connectivité** : USB, réseau, audio
> - **Format** : compatible avec le boîtier

### Le firmware (BIOS/UEFI)

> [!quote] Un programme embarqué essentiel
> Le **firmware** est un logiciel permanent stocké dans une **mémoire morte** (ROM) sur la carte mère.

#### BIOS (Basic Input Output System)

> [!info] Système historique

**Fonctions du BIOS** :
- **Détecter les périphériques** au démarrage (POST - Power-On Self-Test)
- **Configuration générale** du matériel (date/heure, ordre de boot, etc.)
- **Périphérique de démarrage** : définir depuis quel disque/clé démarrer
- **Interface de configuration** : accès via touches spéciales au démarrage (F2, Suppr, F10, etc.)

**Caractéristiques** :
- Stocké dans une **ROM** (Read-Only Memory)
- Interface texte basique
- Limitations techniques (adressage 16 bits)

#### UEFI (Unified Extensible Firmware Interface)

> [!important] Remplaçant moderne du BIOS

**Avantages de l'UEFI** :
- **Interface graphique** : utilisation de la souris
- **Support GPT** : disques > 2 Tio
- **Secure Boot** : vérification de la signature des OS
- **Réseau** : support réseau natif (PXE boot amélioré)
- **Fonctionnalités avancées** : drivers, applications embarquées

**Compatibilité** :
- Mode **UEFI natif** : fonctionnalités complètes
- Mode **Legacy/CSM** : émulation BIOS pour anciens OS

> [!warning] Secure Boot
> Le Secure Boot peut empêcher le démarrage de certains systèmes Linux s'ils ne sont pas signés. Il peut être désactivé dans les paramètres UEFI.

> [!tip] Accès au firmware
> Pour accéder à l'UEFI/BIOS :
> - Appuyer sur **F2**, **Suppr**, **F10** ou **F12** au démarrage
> - Sous Windows : Paramètres → Récupération → Démarrage avancé

### Le boîtier et l'alimentation

#### Le boîtier

> [!quote] Sortez couvert !
> Le **boîtier** (case) protège et organise les composants de l'ordinateur.

**Formats courants** :

| Format | Description | Usage |
|--------|-------------|-------|
| **Desktop** | Horizontal, peu encombrant | Bureautique |
| **Tour** (Tower) | Vertical, plusieurs tailles (mini, mid, full) | Usage général à gaming |
| **Rackable** | Format standardisé pour racks 19" | Serveurs datacenter |
| **SFF** (Small Form Factor) | Très compact | HTPC, bureau restreint |

**Critères importants** :

**1. Refroidissement**

- **Circulation de l'air** : flux avant → arrière, bas → haut
- **Emplacements ventilateurs** : nombre et taille (80mm, 120mm, 140mm)
- **Filtres à poussière** : facilement amovibles

**2. Compatibilité**

- **Format carte mère** : ATX, microATX, Mini-ITX
- **Longueur carte graphique** : vérifier l'espace disponible
- **Hauteur ventirad CPU** : clearance suffisant
- **Emplacements disques** : nombre de baies 3,5" et 2,5"

**3. Câble management**

- **Passe-câbles** : pour organiser les câbles
- **Espace derrière la carte mère** : pour cacher les câbles

> [!example] Photos
> - Photo : Robert Kloosterhuis sur Wikimedia Commons
> - Photo : Tobias "ToMar" Maier sur Wikimedia Commons

#### L'alimentation (PSU - Power Supply Unit)

> [!important] Fournit l'énergie électrique

**Caractéristiques principales** :

**1. Puissance**

- Mesurée en **Watts (W)**
- Gamme courante : 400W à 1000W+
- Doit être suffisante pour tous les composants

> [!tip] Calcul de la puissance
> Utilise un calculateur en ligne (ex: OuterVision PSU Calculator) pour estimer la puissance nécessaire selon tes composants.

**2. Rendement (certification 80 PLUS)**

Indique l'efficacité énergétique :

| Certification | Rendement à 50% de charge |
|---------------|---------------------------|
| **80 PLUS** | ≥ 80% |
| **80 PLUS Bronze** | ≥ 82% |
| **80 PLUS Silver** | ≥ 85% |
| **80 PLUS Gold** | ≥ 87% |
| **80 PLUS Platinum** | ≥ 90% |
| **80 PLUS Titanium** | ≥ 92% |

> [!example] Calcul du rendement
> Alimentation 500W Gold (87% de rendement) :
> - Consomme ~575W sur le secteur
> - Fournit 500W aux composants
> - 75W perdus en chaleur

**3. Modularité**

- **Non-modulaire** : tous les câbles soudés
- **Semi-modulaire** : câbles principaux soudés, autres détachables
- **Modulaire** : tous les câbles détachables

> [!warning] Qualité de l'alimentation
> Ne lésine **jamais** sur la qualité de l'alimentation ! Une mauvaise PSU peut endommager tous tes composants.

---

## Le stockage

### Disques durs (HDD)

> [!quote] Stockage magnétique mécanique
> Le **disque dur** (HDD - Hard Disk Drive) est un dispositif de stockage à base de **plateaux magnétiques** rotatifs.

#### Fonctionnement du HDD

**Composants principaux** :

```
┌─────────────────────────────┐
│    ╔═══════════════╗        │
│    ║   Plateaux    ║        │  ← Plateaux magnétiques
│    ║  magnétiques  ║        │    (plusieurs empilés)
│    ╚═══════════════╝        │
│         │     ▲             │
│         │     │             │
│    ┌────▼─────┴────┐        │
│    │ Têtes de      │        │  ← Têtes de lecture/écriture
│    │ lecture       │        │
│    └───────────────┘        │
│         │                   │
│    ┌────▼─────┐             │
│    │ Moteur   │             │  ← Moteur de rotation
│    └──────────┘             │
└─────────────────────────────┘
```

**Organisation des données** :

| Concept | Description |
|---------|-------------|
| **Plateau** | Disque magnétique rotatif (un ou plusieurs) |
| **Face** | Chaque plateau a 2 faces utilisables |
| **Tête de lecture** | Lit et écrit les données (une par face) |
| **Piste** | Cercle concentrique sur une face |
| **Cylindre** | Ensemble des pistes de même rayon sur tous les plateaux |
| **Secteur** | Plus petite unité adressable (généralement 512 octets) |

**Caractéristiques** :
- **Lecture séquentielle** : plus rapide sur données consécutives
- **Latence mécanique** : temps de déplacement de la tête + rotation du plateau
- **Capacité** : jusqu'à 20+ Tio pour les modèles grand public
- **Vitesse de rotation** : 5400 RPM (lent) à 7200 RPM (standard) ou 10000 RPM (rapide)

**Avantages** :
- ✅ **Prix par Go** très compétitif
- ✅ **Grande capacité** disponible
- ✅ **Durée de vie** en écriture quasi illimitée

**Inconvénients** :
- ❌ **Lent** (80-160 Mo/s en lecture séquentielle)
- ❌ **Fragile** : sensible aux chocs (pièces mobiles)
- ❌ **Bruit** : audible (moteur, têtes)
- ❌ **Consommation** : plus élevée qu'un SSD

> [!example] Photo
> Photo : Jacek Halicki sur Wikimedia Commons

### Disques SSD

> [!quote] Stockage électronique sans pièce mobile
> Le **SSD** (Solid State Drive) est un dispositif de stockage à base de **mémoire flash** (électronique).

#### Fonctionnement du SSD

**Technologie** :
- **Mémoire flash NAND** : cellules de transistors stockant des charges électriques
- **Contrôleur** : gère les lectures/écritures et l'usure
- **DRAM cache** : accélère les opérations (sur certains modèles)

**Types de mémoire NAND** :

| Type | Bits/cellule | Vitesse | Endurance | Prix | Usage |
|------|--------------|---------|-----------|------|-------|
| **SLC** | 1 | Très rapide | Excellente | Très cher | Entreprise critique |
| **MLC** | 2 | Rapide | Bonne | Cher | Pro/Enthusiast |
| **TLC** | 3 | Bon | Moyenne | Abordable | Grand public |
| **QLC** | 4 | Correct | Faible | Économique | Stockage de masse |

**Interfaces** :
- **SATA III** : jusqu'à 600 Mo/s (limite de l'interface)
- **NVMe** (PCIe 3.0) : jusqu'à 3500 Mo/s
- **NVMe** (PCIe 4.0) : jusqu'à 7000 Mo/s
- **NVMe** (PCIe 5.0) : jusqu'à 14000 Mo/s

**Avantages** :
- ✅ **Très rapide** : 10x à 100x plus rapide qu'un HDD
- ✅ **Résistant aux chocs** : pas de pièce mobile
- ✅ **Silencieux** : aucun bruit
- ✅ **Faible consommation** : économe en énergie
- ✅ **Compact** : formats M.2, 2,5"

**Inconvénients** :
- ❌ **Prix par Go** plus élevé qu'un HDD
- ❌ **Limite d'écriture** : durée de vie en cycles d'écriture (TBW - TeraBytes Written)
- ❌ **Récupération** : plus difficile en cas de panne

> [!warning] Usure des SSD
> Les cellules flash ont une **durée de vie limitée** en nombre d'écritures. Les SSD modernes ont des mécanismes de gestion de l'usure (wear leveling) mais évite les écritures excessives inutiles.

> [!example] Photo
> Photo : Arvutistuudio sur Wikimedia Commons

### Secteurs et adressage

> [!important] Organisation logique des disques

#### Concept du secteur

> [!quote] Plus petite unité adressable
> Un **secteur** est la plus petite unité de données qu'on peut lire ou écrire sur un disque. Chaque secteur a une **adresse unique**.

**Principe** :

```
Disque = Séquence de secteurs numérotés

Secteur = Séquence binaire (généralement 512 octets)

┌──────┬──────┬──────┬──────┬──────┬─────┐
│ Sect │ Sect │ Sect │ Sect │ Sect │ ... │
│  0   │  1   │  2   │  3   │  4   │     │
└──────┴──────┴──────┴──────┴──────┴─────┘
  512o   512o   512o   512o   512o
```

#### Calcul de la capacité maximale

> [!example] Limite avec adressage 32 bits

**Avec adresses 32 bits** :
- Nombre max de secteurs : **2³² ≈ 4 milliards**

**Si secteurs de 1 octet** :
- Capacité max : 2³² octets = 4 Gio ❌ **Insuffisant !**

**Avec secteurs de 512 octets** (standard) :
- Capacité max : 2³² × 512 octets = 2 Tio ✅ **Correct**

**Avec adresses 64 bits** :
- Nombre max de secteurs : **2⁶⁴ ≈ 18 milliards de milliards**
- Avec secteurs 512 o : 2⁶⁴ × 512 = **8 Zio** (zébioctets) 🚀

> [!important] Taille standard des secteurs
> La taille habituelle des secteurs sur un disque dur est de **512 octets** (ou 4096 octets pour les nouveaux disques avec "Advanced Format").

#### Tableau récapitulatif

| Adressage | Secteurs | Taille secteur | Capacité max |
|-----------|----------|----------------|--------------|
| 32 bits | 4 milliards | 512 o | **2 Tio** |
| 32 bits | 4 milliards | 4096 o | **16 Tio** |
| 64 bits | 18 × 10¹⁸ | 512 o | **8 Zio** |
| 64 bits | 18 × 10¹⁸ | 4096 o | **64 Zio** |

### Partitionnement

> [!quote] Découper ses disques en sections logiques
> Le **partitionnement** consiste à diviser un disque physique en plusieurs **partitions** logiques indépendantes.

#### Pourquoi partitionner ?

**Raisons de créer des partitions** :
- **Organisation** : séparer système et données
- **Multi-boot** : installer plusieurs OS
- **Sécurité** : isoler les environnements
- **Sauvegarde** : faciliter les opérations de backup
- **Performance** : optimiser certains usages

#### Systèmes de partitionnement

> [!info] Deux systèmes principaux

**Utiliser un disque brut** (sans partitionnement) :
- **Rare** en pratique
- Utilisé pour certains usages spécifiques (bases de données)

**En général** : Système de partitionnement
- **Découpe** le disque en partitions
- **Informations supplémentaires** : type, flags, bootable
- **Routine de démarrage** : code pour lancer l'OS

**Deux standards** :
1. **MBR** (Master Boot Record) - ancien, limité
2. **GPT** (GUID Partition Table) - moderne, recommandé

### MBR (Master Boot Record)

> [!quote] Système de partitionnement historique Intel
> Le **MBR** (Master Boot Record) est l'ancien système de partitionnement, créé dans les années 1980.

#### Caractéristiques du MBR

**Limitations principales** :
- **Taille max des partitions** : **2 Tio** (adressage 32 bits avec secteurs 512 o)
- **Nombre de partitions** : **4 partitions primaires** maximum
- **Pas de redondance** : table de partition unique (risque de perte)

#### Structure du MBR

```
┌─────────────────────────────────────┐
│  Secteur 0 (512 octets)             │
│                                     │
│  ┌───────────────────────┐          │
│  │ Code d'amorçage       │  446 o   │
│  │ (Bootloader)          │          │
│  └───────────────────────┘          │
│  ┌───────────────────────┐          │
│  │ Table de partitions   │   64 o   │
│  │ (4 entrées × 16 o)    │          │
│  └───────────────────────┘          │
│  ┌───────────────────────┐          │
│  │ Signature (0x55AA)    │    2 o   │
│  └───────────────────────┘          │
└─────────────────────────────────────┘
```

#### Types de partitions MBR

| Type | Description |
|------|-------------|
| **Partition primaire** | Partition normale, 4 maximum |
| **Partition étendue** | Conteneur pour partitions logiques (1 seule possible) |
| **Partition logique** | Partition à l'intérieur de la partition étendue |

**Schéma typique** :

```
┌──────────────┬──────────────┬──────────────┬─────────────────────┐
│  Primaire 1  │  Primaire 2  │  Primaire 3  │  Étendue            │
│  (Système)   │  (Données)   │  (Swap)      │                     │
│              │              │              │  ┌────────┬────────┐ │
│              │              │              │  │ Log. 1 │ Log. 2 │ │
│              │              │              │  └────────┴────────┘ │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

#### Drapeaux (flags) MBR

- **Bootable** (0x80) : indique la partition de démarrage
- **Type de partition** : code hexadécimal (0x83 = Linux, 0x82 = swap, 0x07 = NTFS, etc.)

> [!warning] MBR obsolète
> Le MBR est **obsolète** pour les nouveaux systèmes. Utilise GPT à la place, sauf pour de vieux matériels ou des contraintes spécifiques.

### GPT (GUID Partition Table)

> [!quote] Format moderne de partitionnement
> Le **GPT** (GUID Partition Table) est le système de partitionnement moderne, partie de la spécification UEFI.

#### Caractéristiques du GPT

**Avantages** :
- **Taille max des partitions** : **8 Zio** (zébioctets) avec secteurs 512 o
- **Nombre de partitions** : jusqu'à **128 partitions** (standard, extensible)
- **Redondance** : tables en début et fin de disque
- **Intégrité** : CRC32 pour détecter les corruptions
- **Compatibilité UEFI** : requis pour Secure Boot

#### Structure du GPT

```
┌─────────────────────────────────────────┐
│  Secteur 0 : MBR protecteur             │  ← Compatibilité
├─────────────────────────────────────────┤
│  Secteur 1 : En-tête GPT primaire       │  ← Infos principales
├─────────────────────────────────────────┤
│  Secteurs 2-33 : Table de partitions    │  ← 128 entrées
│                  primaire (128 entrées) │
├─────────────────────────────────────────┤
│                                         │
│  Partitions de données                  │
│                                         │
├─────────────────────────────────────────┤
│  Secteurs -33 à -2 : Table partitions   │  ← Backup
│                      secondaire         │
├─────────────────────────────────────────┤
│  Secteur -1 : En-tête GPT secondaire    │  ← Backup
└─────────────────────────────────────────┘
```

#### Identification par GUID

> [!important] Chaque partition a un identifiant unique

**GUID (Globally Unique Identifier)** :
- Identifiant de **128 bits** globalement unique
- Format : `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- Deux types :
  - **Type GUID** : identifie le type de partition (système, données, swap, etc.)
  - **Unique GUID** : identifiant unique de cette partition spécifique

> [!example] Exemple de GUID
> ```
> Type : EFI System Partition
> GUID : C12A7328-F81F-11D2-BA4B-00A0C93EC93B
> ```

#### Comparaison MBR vs GPT

| Caractéristique | MBR | GPT |
|-----------------|-----|-----|
| **Année** | 1983 | 2000+ |
| **Taille max partition** | 2 Tio | 8 Zio |
| **Nombre partitions** | 4 primaires | 128 standard |
| **Redondance** | Non | Oui (début + fin) |
| **Firmware** | BIOS | UEFI (ou BIOS-CSM) |
| **Intégrité** | Non | CRC32 |
| **Identification** | Numéro | GUID unique |

> [!tip] Utiliser GPT
> Pour tout nouveau système, utilise **GPT** :
> - Meilleure fiabilité
> - Pas de limite pratique de taille
> - Support moderne
> - Requis pour Windows 11

---

## Les périphériques d'entrée/sortie

### Périphériques d'entrée

> [!quote] Amener des informations à la machine
> Les **périphériques d'entrée** (input devices) permettent de **transmettre des données** de l'utilisateur vers l'ordinateur.

#### Liste des périphériques d'entrée courants

| Périphérique | Fonction | Données transmises |
|--------------|----------|-------------------|
| **Clavier** | Saisie de texte et commandes | Caractères, touches |
| **Souris** | Pointage et sélection | Position X/Y, clics |
| **Écran tactile** | Interaction directe | Coordonnées tactiles, gestes |
| **Webcam** | Capture vidéo | Flux d'images |
| **Microphone** | Capture audio | Signal audio numérique |
| **Scanner** | Numérisation de documents | Images numériques |
| **Carte réseau** | Réception de données réseau | Paquets réseau |
| **Manette/Joystick** | Contrôle de jeux | Axes, boutons |
| **Stylet graphique** | Dessin numérique | Position, pression, inclinaison |
| **Lecteur de carte** | Lecture cartes mémoire | Données stockées |
| **Lecteur biométrique** | Authentification | Empreinte, visage, iris |

> [!example] Exemples d'usage TSSR
> - **Clavier** : administration en CLI
> - **Souris** : navigation GUI
> - **Scanner** : numérisation de documents
> - **Carte réseau** : réception de données

### Périphériques de sortie

> [!quote] Récupérer les informations de la machine
> Les **périphériques de sortie** (output devices) permettent de **restituer des données** de l'ordinateur vers l'utilisateur.

#### Liste des périphériques de sortie courants

| Périphérique | Fonction | Données restituées |
|--------------|----------|-------------------|
| **Écran** | Affichage visuel | Images, texte, vidéos |
| **Carte son** | Production audio | Signal audio analogique |
| **Haut-parleurs** | Diffusion sonore | Son audible |
| **Casque audio** | Écoute personnelle | Son audible |
| **Carte réseau** | Émission de données réseau | Paquets réseau |
| **Imprimante** | Impression papier | Documents physiques |
| **Projecteur** | Affichage grand format | Images projetées |
| **Carte graphique** | Génération d'images | Signal vidéo |
| **LED/Voyants** | Indicateurs visuels | État système |

> [!note] Périphériques mixtes (Entrée/Sortie)
> Certains périphériques sont à la fois entrée ET sortie :
> - **Carte réseau** : reçoit et envoie
> - **Écran tactile** : affiche et reçoit des touches
> - **Disque dur/SSD** : lit et écrit
> - **Clé USB** : lecture et écriture

> [!example] Exemples d'usage TSSR
> - **Écran** : affichage des commandes et résultats
> - **Imprimante** : impression de rapports
> - **Projecteur** : présentation
> - **Carte réseau** : communication client/serveur

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définition

- **Ordinateur** : machine électronique, numérique, programmable
- **Architecture** : UAL + Unité de contrôle + Mémoire + I/O
- **Modèle** : Architecture de Von Neumann, Machine de Turing

### Unité d'information

- **Bit** : unité élémentaire (0 ou 1)
- **Octet** : 8 bits = 256 valeurs (0 à 255)
- **Formule** : n bits → 2ⁿ valeurs possibles
- **Unités** : bit, octet, Kio, Mio, Gio, Tio, Pio, Eio, Zio

### CPU (Processeur)

- **Composants** : UAL, unité de contrôle, horloge, registres, caches
- **Caractéristiques** : jeu d'instructions, fréquence, taille registres
- **x86-64** : PC classiques (Intel/AMD)
- **ARM** : mobile, embarqué (Raspberry Pi)
- **Caches** : L1 (plus rapide), L2, L3 (partagé)

### Mémoires

- **RAM** : volatile, temporaire, accès direct
- **Hiérarchie** : Registres → L1 → L2 → L3 → RAM → SSD → HDD
- **Principe** : plus rapide = plus cher = plus petit

### Carte mère

- **Formats** : ATX, microATX, Mini-ITX
- **Chipset** : gestion des communications
- **Connecteurs** : CPU, RAM, PCIe, M.2, SATA
- **Firmware** : BIOS (ancien) ou UEFI (moderne)

### Stockage

**HDD** :
- Mécanique, plateaux magnétiques
- Lent mais grande capacité, économique
- Fragile (chocs)

**SSD** :
- Électronique, mémoire flash
- Très rapide, résistant, silencieux
- Plus cher, durée de vie limitée en écriture

**Secteurs** :
- Taille standard : **512 octets**
- Adressage 32 bits + 512 o = **2 Tio max**
- Adressage 64 bits + 512 o = **8 Zio max**

### Partitionnement

**MBR** (obsolète) :
- 4 partitions primaires max
- 2 Tio max par partition
- Adressage 32 bits

**GPT** (moderne) :
- 128 partitions
- 8 Zio max par partition
- Redondance, CRC32
- Requis pour UEFI/Secure Boot

### Périphériques

- **Entrée** : clavier, souris, webcam, microphone, scanner, carte réseau
- **Sortie** : écran, imprimante, haut-parleurs, carte réseau, projecteur
- **Mixtes** : écran tactile, carte réseau, disques

### Questions de révision

> [!example] Quiz rapide

**Q1** : Quel jeu d'instruction utilise un processeur AMD 64 bits ?
**R** : **x86-64**

**Q2** : Combien de valeurs différentes peut-on coder sur 8 bits ?
**R** : **256** (2⁸)

**Q3** : Quel est le successeur de MBR ?
**R** : **GPT** (GUID Partition Table)

**Q4** : Quelle est la taille habituelle des secteurs sur un disque dur ?
**R** : **512 octets**

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Ordinateur** | Machine électronique, numérique et programmable effectuant des opérations arithmétiques |
| **Architecture** | Organisation structurelle des composants d'un système informatique |
| **UAL** | Unité Arithmétique et Logique - effectue les calculs et opérations logiques |
| **Unité de contrôle** | Séquence et coordonne l'exécution des instructions |
| **Von Neumann** | Architecture où instructions et données partagent la même mémoire |
| **Bit** | Binary Digit - unité élémentaire d'information (0 ou 1) |
| **Octet** | Groupe de 8 bits (256 valeurs possibles : 0-255) |
| **CPU** | Central Processing Unit - processeur, cerveau de l'ordinateur |
| **Registre** | Mémoire ultra-rapide interne au CPU |
| **Cache** | Mémoire tampon rapide (L1, L2, L3) |
| **Horloge** | Synchronise les opérations du CPU (fréquence en Hz) |
| **Jeu d'instructions** | Ensemble des instructions qu'un CPU peut exécuter (x86-64, ARM) |
| **RAM** | Random Access Memory - mémoire vive, volatile, accès direct |
| **Volatile** | Mémoire dont le contenu est perdu à l'extinction |
| **Carte mère** | Circuit principal interconnectant tous les composants |
| **Chipset** | Ensemble de circuits gérant les communications entre composants |
| **Socket** | Emplacement pour le processeur sur la carte mère |
| **PCIe** | PCI Express - bus d'extension rapide pour cartes |
| **BIOS** | Basic Input Output System - firmware historique |
| **UEFI** | Unified Extensible Firmware Interface - firmware moderne |
| **Firmware** | Logiciel permanent stocké en ROM |
| **ROM** | Read-Only Memory - mémoire morte, non volatile |
| **POST** | Power-On Self-Test - test au démarrage |
| **HDD** | Hard Disk Drive - disque dur mécanique |
| **SSD** | Solid State Drive - disque électronique (flash) |
| **Plateau** | Disque magnétique rotatif dans un HDD |
| **Tête de lecture** | Lit et écrit les données sur les plateaux |
| **Piste** | Cercle concentrique de données sur un plateau |
| **Cylindre** | Ensemble des pistes de même rayon sur tous les plateaux |
| **Secteur** | Plus petite unité adressable sur un disque (512 o typique) |
| **Mémoire flash** | Mémoire électronique non volatile (SSD, clés USB) |
| **NAND** | Type de mémoire flash utilisée dans les SSD |
| **NVMe** | Non-Volatile Memory Express - protocole rapide pour SSD |
| **SATA** | Serial ATA - interface de connexion pour disques |
| **M.2** | Format compact pour SSD (carte sur slot) |
| **Partition** | Division logique d'un disque physique |
| **MBR** | Master Boot Record - ancien système de partitionnement (4 partitions, 2 Tio max) |
| **GPT** | GUID Partition Table - système moderne (128 partitions, 8 Zio max) |
| **GUID** | Globally Unique Identifier - identifiant unique 128 bits |
| **Bootloader** | Programme de démarrage de l'OS |
| **I/O** | Input/Output - Entrées/Sorties |
| **Périphérique d'entrée** | Dispositif transmettant des données vers l'ordinateur |
| **Périphérique de sortie** | Dispositif restituant des données de l'ordinateur |
| **ATX** | Format standard de carte mère (305 × 244 mm) |
| **PSU** | Power Supply Unit - alimentation électrique |
| **TDP** | Thermal Design Power - consommation thermique d'un composant |
| **RPM** | Rotations Per Minute - vitesse de rotation d'un HDD |
| **Latence** | Temps d'attente avant accès aux données |
| **Bande passante** | Quantité de données transférables par unité de temps |
| **Throughput** | Débit réel de transfert de données |

---

## Résumé final

> [!success] À retenir pour le titre RNCP

**Concepts fondamentaux** :
- Architecture de Von Neumann : UAL + Contrôle + Mémoire + I/O
- Bit (0/1) et octet (8 bits = 256 valeurs)
- Hiérarchie des mémoires : vitesse vs capacité vs coût

**Composants essentiels** :
- **CPU** : cerveau (UAL, contrôle, caches, registres)
- **RAM** : mémoire temporaire, volatile, rapide
- **Carte mère** : interconnexion (chipset, slots, connecteurs)
- **Firmware** : BIOS/UEFI pour initialisation

**Stockage** :
- **HDD** : mécanique, lent, grande capacité, économique
- **SSD** : électronique, rapide, cher, durée de vie limitée
- **Secteurs** : 512 octets standard
- **MBR** : obsolète (4 partitions, 2 Tio max)
- **GPT** : moderne (128 partitions, 8 Zio max)

**Périphériques** :
- Entrée : clavier, souris, scanner, carte réseau
- Sortie : écran, imprimante, haut-parleurs
- Mixtes : écran tactile, réseau, disques

---

*Document créé pour la préparation au titre RNCP Technicien Supérieur Systèmes et Réseaux (TSSR)*  
*Compatible Obsidian avec callouts natifs*