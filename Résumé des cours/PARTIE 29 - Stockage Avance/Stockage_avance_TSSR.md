# Stockage Avancé

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Stockage avancé - RAID / LVM / SAN & NAS  
**Date** : Janvier 2026  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction au stockage avancé|Introduction au stockage avancé]]
   - [[#Analyse des besoins en stockage|Analyse des besoins en stockage]]
   - [[#Choix des supports de stockage|Choix des supports de stockage]]
   - [[#Fiabilité et cloisonnement|Fiabilité et cloisonnement]]
   - [[#Problématiques du partitionnement classique|Problématiques du partitionnement classique]]
2. [[#RAID - Redundant Array of Independent Disks|RAID - Redundant Array of Independent Disks]]
   - [[#Concept et objectifs du RAID|Concept et objectifs du RAID]]
   - [[#Types d'implémentation RAID|Types d'implémentation RAID]]
   - [[#Les niveaux de RAID|Les niveaux de RAID]]
   - [[#Combinaisons de RAID|Combinaisons de RAID]]
   - [[#Considérations générales RAID|Considérations générales RAID]]
3. [[#LVM - Logical Volume Manager|LVM - Logical Volume Manager]]
   - [[#Concept et architecture LVM|Concept et architecture LVM]]
   - [[#Composants LVM|Composants LVM]]
   - [[#Snapshots et Copy-On-Write|Snapshots et Copy-On-Write]]
   - [[#Redimensionnement à chaud|Redimensionnement à chaud]]
4. [[#SAN & NAS - Stockage en réseau|SAN & NAS - Stockage en réseau]]
   - [[#NAS - Network Attached Storage|NAS - Network Attached Storage]]
   - [[#SAN - Storage Area Network|SAN - Storage Area Network]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction au stockage avancé

> [!abstract] Vue d'ensemble
> Le stockage avancé regroupe l'ensemble des techniques permettant d'optimiser la gestion du stockage en termes de volume, performance, fiabilité et flexibilité. Les technologies RAID et LVM constituent les piliers de cette gestion avancée.

### Questions préliminaires

> [!question] Quiz de départ
> **Question 1** : Quelles sont les différentes couches de gestion du stockage depuis la couche physique jusqu'à la couche logique (les fichiers) ?
> 
> **Réponse** : Disques physiques → Partitions → (RAID) → (LVM) → Systèmes de fichiers → Fichiers
> 
> **Question 2** : Peut-on regrouper plusieurs disques au sein d'une seule partition ?
> 
> **Réponse** : Oui, avec RAID ou LVM
> 
> **Question 3** : Peut-on mettre plusieurs systèmes de fichiers sur la même partition ?
> 
> **Réponse** : Non, une partition = un système de fichiers
> 
> **Question 4** : Peut-on regrouper plusieurs partitions dans un même système de fichiers ?
> 
> **Réponse** : Oui, avec LVM (Volume Group)

### Analyse des besoins en stockage

> [!info] Besoins variables selon les applications
> Les besoins en stockage sont très variables en fonction des applications. Pour une application et donc pour les serveurs associés, il faut analyser :

#### Les trois critères fondamentaux

| Critère | Questions à se poser |
|---------|----------------------|
| **Volume** | Quel espace de stockage est nécessaire à l'installation ? Quelle évolution dans le temps ? |
| **Performance** | Quel débit et temps d'accès requis ? L'accès aux données est-il un goulot d'étranglement ? |
| **Sûreté** | Quelle stratégie de conservation des données ? Sauvegarde ? Tolérance de panne ? |

> [!example] Exemples de besoins
> - **Serveur web statique** : Volume moyen, performance importante (temps d'accès), sûreté standard
> - **Base de données transactionnelle** : Volume important, performance critique (IOPS élevés), sûreté maximale
> - **Serveur de fichiers** : Volume très important, performance variable, sûreté importante
> - **Serveur d'archivage** : Volume très important, performance faible acceptable, sûreté critique

### Choix des supports de stockage

> [!info] Compromis volume / coût / performance
> L'offre en support de stockage est contrainte par des compromis entre volume, coût et performance.

#### Instantané technologique 2023

**Disques durs (HDD) :**
- **Volume** : 1 To → 24 To
- **Performance** : ≈ 100 Mo/s
- **Tarif** : 20-30 €/To
- **Technologies avancées** : HAMR (Heat-Assisted Magnetic Recording), SMR (Shingled Magnetic Recording) pour volumes > 30 To

**SSD (Solid State Drive) :**
- **Volume** : 250 Go → 8 To
- **Performance** : ≈ 3 à 7 Go/s
- **Tarif** : ≈ 100 €/To

**NVMe (évolution du SSD) :**
- **Performance maximale** : > 7 Go/s
- **Latence ultra-faible**
- **Coût plus élevé que SSD SATA**

> [!note] Hiérarchie de performance
> **HDD < SSD SATA < SSD NVMe**
> 
> Le coût n'est **pas linéaire** : doubler la capacité ne double pas forcément le prix, et les performances maximales sont significativement plus chères.

### Fiabilité et cloisonnement

#### Problématique de fiabilité

> [!warning] Risques de panne
> **En pratique, la fiabilité est très variable** :
> - **Panne mécanique** des disques durs (têtes de lecture, plateaux)
> - **Limite en nombre d'écritures** des SSD (usure des cellules mémoire)
> - **Probabilité de panne** : plus on a de périphériques, plus elle augmente
> - **Criticité** : contrairement au reste d'un ordinateur, changer le support de stockage ne suffit pas à relancer l'application (les données sont perdues)

#### Stratégie de cloisonnement

> [!important] Pourquoi cloisonner ?
> Quelques constats qui justifient le cloisonnement :
> - Un système de fichiers est un **espace partagé**
> - Le **contrôle applicatif de la volumétrie** est rare
> - En général : **stockage plein = système/application plante**

**Solution classique :**
> Des espaces de stockage **différents et étanches**
> 
> En cas de débordement ⇒ seul le composant qui déborde est impacté

#### Dimensionnement et découpage

> [!tip] Stratégie de partitionnement
> Isoler les répertoires à forte variation et fixer un volume maximum à l'installation.

**Exemples de répertoires à isoler :**
- **`/var/log`** : logs système et applicatifs (croissance continue)
- **Bases de données** : volumes importants et variables
- **Fichiers utilisateurs** : `/home` (croissance imprévisible)
- **Espaces de swap** : taille fixe prévisible
- **Répertoires temporaires** : `/tmp` (nettoyage régulier mais pics possibles)

> [!example] Schéma de partitionnement type
> ```
> /dev/sda1 → /boot       (500 Mo)
> /dev/sda2 → /           (20 Go)
> /dev/sda3 → /var        (10 Go)
> /dev/sda4 → /var/log    (5 Go)
> /dev/sdb1 → /home       (100 Go)
> /dev/sdb2 → swap        (8 Go)
> ```

### Problématiques du partitionnement classique

> [!warning] Limites du partitionnement traditionnel
> Le partitionnement classique présente plusieurs limitations critiques :

| Limitation | Conséquence |
|------------|-------------|
| **Nombre de partitions limité** | MBR : 4 primaires (ou 3 + 1 étendue), GPT : 128 partitions |
| **Taille limitée** | Limitée par la taille maximale des disques physiques |
| **Évolution difficile** | Modification après installation **limitée et risquée** |
| **Rigidité** | Impossible de redistribuer facilement l'espace entre partitions |

> [!important] Besoin d'évolution
> Ces limitations ont conduit au développement de solutions plus flexibles :
> - **RAID** : pour la performance et la fiabilité
> - **LVM** : pour la flexibilité de gestion
> - **Combinaison RAID + LVM** : solution optimale en entreprise

---

## RAID - Redundant Array of Independent Disks

> [!quote] Définition RAID
> **RAID** = **R**edundant **A**rray of **I**ndependent/**I**nexpensive **D**isks
> 
> Technique de virtualisation du stockage ayant pour objectifs :
> - Agrandir la taille maximum disponible
> - Améliorer les performances (débit)
> - Améliorer la fiabilité
> 
> Sans avoir à recourir à de meilleurs disques qui peuvent être plus/trop chers ou indisponibles.

### Concept et objectifs du RAID

#### Contexte historique

> [!info] Naissance du RAID
> - **1987** : Université de Berkeley (Californie)
> - **Opposition au SLED** (Single Large Expensive Disk)
> - **Objectif** : démocratiser le stockage haute performance

#### Principe général

> [!abstract] Idée centrale
> **Construire un volume/grappe/cluster RAID (array)** :
> - Espace de stockage similaire à un disque unique
> - À l'aide de **plusieurs disques physiques**
> - Selon un **niveau** déterminant les propriétés de la grappe
> 
> Concept général disposant de **nombreuses implémentations différentes**

### Types d'implémentation RAID

> [!info] Trois catégories d'implémentation
> Il existe trois types de RAID, chacun avec ses avantages et inconvénients.

#### RAID Matériel

> [!note] Caractéristiques RAID matériel
> **Principe** : Contrôleur disque (carte) installé sur un serveur

**Avantages** :
- ✅ Opérations effectuées directement au niveau du matériel
  - ⇒ **Excellente performance** & **pas de consommation CPU**
- ✅ **Transparent** pour le système d'exploitation
  - ⇒ **Boot sur la grappe RAID possible**
- ✅ Gestion au niveau BIOS/UEFI

**Inconvénients** :
- ❌ Implémentations **propriétaires** - spécifiques à chaque fabricant
  - ⇒ En général **incompatible** entre marques
- ❌ Configuration au boot (BIOS) et/ou via logiciel spécifique
- ❌ Souvent **assez coûteux**

> [!example] Cas d'usage
> **Courant sur les serveurs d'entreprise** nécessitant performances maximales et fiabilité (serveurs de bases de données, virtualisation).

#### RAID Logiciel

> [!note] Caractéristiques RAID logiciel
> **Principe** : Fonctionnalité intégrée au système d'exploitation

**Avantages** :
- ✅ **Assez souple** d'utilisation
- ✅ **Standard et compatible** (pour le même système)
- ✅ **Pas de coût matériel** supplémentaire
- ✅ Indépendant du matériel

**Inconvénients** :
- ❌ **Boot sur la grappe RAID compliqué** (nécessite partition boot séparée)
- ❌ **Consomme du CPU** (calculs de parité, reconstruction)

**Disponibilité** :
- **Linux** : outil `mdadm` (Multiple Device Administrator)
- **Windows** : Espaces de stockage (Storage Spaces)

> [!tip] Commandes Linux essentielles
> ```bash
> # Créer un RAID 1
> mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
> 
> # Vérifier l'état
> cat /proc/mdstat
> 
> # Détails d'une grappe
> mdadm --detail /dev/md0
> ```

#### RAID Hybride

> [!note] Caractéristiques RAID hybride
> **Principe** : Contrôleur physique nécessitant un logiciel (pilote)

**Avantage** :
- ✅ Permet de **booter le système** sur le RAID

**Inconvénients** :
- ❌ On retrouve plutôt les **inconvénients des 2 types** :
  - Problème de **compatibilité** (propriétaire)
  - **Consommation CPU** (pilote logiciel)
  - Fonctionnalités souvent **limitées**

> [!warning] À éviter
> Implémentations par les constructeurs de cartes mères grand public. Privilégier RAID matériel professionnel ou RAID logiciel.

### Les niveaux de RAID

> [!important] Diversité des niveaux
> Il existe plusieurs niveaux de RAID en fonction des besoins. Ces niveaux peuvent ne pas tous être disponibles sur une implémentation donnée, et il existe des **variantes** et des **combinaisons**.

#### Tableau récapitulatif des niveaux RAID

| Niveau | Nom | Disques min | Capacité | Performance | Fiabilité | Usage typique |
|--------|-----|-------------|----------|-------------|-----------|---------------|
| **JBOD** | Concaténation | 2+ | n × capacité | Standard | Médiocre | Tests, non-prod |
| **RAID 0** | Striping | 2 | n × capacité | ★★★ Excellente | ★ Mauvaise | Performance pure |
| **RAID 1** | Mirroring | 2 | 1 × capacité | Standard | ★★★ Excellente | Fiabilité max |
| **RAID 4** | Parité dédiée | 3+ | (n-1) × capacité | ★★ Bonne | ★★ Bonne | Peu utilisé |
| **RAID 5** | Parité répartie | 3+ | (n-1) × capacité | ★★ Bonne | ★★ Bonne | Compromis standard |
| **RAID 6** | Double parité | 4+ | (n-2) × capacité | ★★ Bonne | ★★★ Excellente | Fiabilité accrue |

#### JBOD (Just a Bunch Of Disks)

> [!quote] Définition JBOD
> **JBOD** = **J**ust a **B**unch **O**f **D**isks (ou **NRAID** = Non Redundant Array of Inexpensive Disks)

**Principe** : Concaténation d'un ensemble de disques en un seul volume

```
┌──────────┐ ┌──────────┐ ┌──────────┐
│  Disk 1  │ │  Disk 2  │ │  Disk 3  │
│  500 Go  │ │  500 Go  │ │  1 To    │
└──────────┘ └──────────┘ └──────────┘
      ↓            ↓            ↓
    ┌────────────────────────────────┐
    │     Volume JBOD : 2 To         │
    │  Data écrite séquentiellement  │
    └────────────────────────────────┘
```

**Caractéristiques** :
- **Capacité totale** : Somme de tous les disques
- **Tolérance aux pannes** : **Médiocre** (perte d'un disque = perte des données sur ce disque)
- **Performances** : **Identiques** à un disque seul
- **Gain** : Taille uniquement

> [!warning] Attention
> JBOD n'est **pas vraiment du RAID** (pas de redondance). À utiliser uniquement pour des données non critiques.

#### RAID 0 (Striping)

> [!quote] Définition RAID 0
> **Striping** = Entrelaçage par bande

**Principe** : Données découpées et réparties sur n disques (en général 2)

```
        ┌─────────────┐
        │    Data     │
        └─────────────┘
              │ split
        ┌─────┴─────┐
        ↓           ↓
   ┌─────────┐ ┌─────────┐
   │ Disk 1  │ │ Disk 2  │
   │ A1 B1   │ │ A2 B2   │
   │ C1 D1   │ │ C2 D2   │
   └─────────┘ └─────────┘
```

**Caractéristiques** :
- **Capacité totale** : n × capacité du plus petit disque
- **Performance** : **Gain proportionnel** au nombre de disques (≈ × n)
- **Fiabilité** : **Mauvaise** (perte d'un disque = perte totale des données)
- **Gain** : Taille + Performance

> [!warning] Danger critique
> **Fiabilité inversement proportionnelle** au nombre de disques :
> - 1 disque : probabilité de panne = P
> - 2 disques en RAID 0 : probabilité ≈ 2P
> - À utiliser **UNIQUEMENT** pour des données temporaires ou non critiques (cache, scratch)

> [!example] Cas d'usage
> - Montage vidéo (fichiers temporaires de travail)
> - Calcul scientifique (résultats intermédiaires)
> - Stations de travail graphiques (performances)

#### RAID 1 (Mirroring)

> [!quote] Définition RAID 1
> **Mirroring** = Disques miroirs (duplication)

**Principe** : Données **copiées identiquement** sur n disques (en général 2)

```
        ┌─────────────┐
        │    Data     │
        └─────────────┘
              │ duplicate
        ┌─────┴─────┐
        ↓           ↓
   ┌─────────┐ ┌─────────┐
   │ Disk 1  │ │ Disk 2  │
   │  DATA   │ │  DATA   │
   │ (copy)  │ │ (copy)  │
   └─────────┘ └─────────┘
```

**Caractéristiques** :
- **Capacité totale** : Capacité du plus petit disque (perte de n-1 disques)
- **Performance** : **Quasi identiques** (légère amélioration en lecture)
- **Fiabilité** : **Excellente** - tolère n-1 pannes
- **Gain** : Fiabilité maximale

> [!success] Avantages
> - **Sécurité maximale** des données
> - **Reconstruction rapide** (simple copie)
> - **Lecture améliorée** (possibilité de lire sur n'importe quel disque)

> [!tip] Usage recommandé
> - Serveurs de bases de données critiques
> - Systèmes d'exploitation de serveurs
> - Données irremplaçables
> - Combinaison avec RAID 0 → RAID 10

#### RAID 4 (Parité dédiée)

> [!quote] Définition RAID 4
> **Agrégation par bande avec parité dédiée**

**Principe** : Striping sur n disques + 1 disque de parité (calcul XOR)

```
     ┌──────────┐
     │   Data   │
     └──────────┘
          │ split
     ┌────┴────┐
     ↓         ↓
┌────────┐ ┌────────┐ ┌──────────┐
│ Disk 1 │ │ Disk 2 │ │  Disk 3  │
│   A1   │ │   A2   │ │ A1⊕A2=P  │
│   B1   │ │   B2   │ │ B1⊕B2=P  │
└────────┘ └────────┘ └──────────┘
                        (Parité)
```

**Caractéristiques** :
- **Disques minimum** : 3
- **Capacité totale** : (n - 1) × capacité du plus petit disque
- **Performance** : Gain proportionnel au nombre de disques (moins 1)
- **Fiabilité** : **Bonne** - tolère 1 panne
- **Inconvénient** : Le disque de parité devient un **goulot d'étranglement en écriture**

> [!note] Calcul de parité (XOR)
> La parité est calculée avec l'opérateur XOR (OU exclusif) :
> - A1 ⊕ A2 = P
> - Si perte de A1 : P ⊕ A2 = A1 (récupération)
> - Si perte de A2 : P ⊕ A1 = A2 (récupération)

> [!warning] Limitation
> **Peu utilisé en pratique** car le disque de parité est sollicité à chaque écriture. Le RAID 5 résout ce problème.

#### RAID 5 (Parité répartie)

> [!quote] Définition RAID 5
> **Agrégation par bande avec parité répartie**

**Principe** : RAID 4 avec les informations de parité **réparties** sur les disques (rotation round-robin)

```
┌────────┐ ┌────────┐ ┌────────┐
│ Disk 1 │ │ Disk 2 │ │ Disk 3 │
├────────┤ ├────────┤ ├────────┤
│   A1   │ │   A2   │ │ Parité │
│   B1   │ │ Parité │ │   B2   │
│ Parité │ │   C1   │ │   C2   │
│   D1   │ │   D2   │ │ Parité │
└────────┘ └────────┘ └────────┘
```

**Caractéristiques** :
- **Disques minimum** : 3
- **Capacité totale** : (n - 1) × capacité du plus petit disque
- **Performance** : **Un peu plus performant** que RAID 4 (pas de goulot)
- **Fiabilité** : **Bonne** - tolère 1 panne
- **Reconstruction** : Coûteuse et/ou longue

> [!important] Avantage sur RAID 4
> Évite que le disque de parité soit un **goulot d'étranglement en écriture**. La charge est répartie sur tous les disques.

> [!success] RAID 5 : le compromis standard
> **Le plus utilisé en entreprise** car offre :
> - Bon compromis **performance/fiabilité/capacité**
> - Perte d'un seul disque (≈ 25-33% selon nombre de disques)
> - Performance acceptable

> [!warning] Risques lors de reconstruction
> - **Durée longue** (plusieurs heures à jours selon capacité)
> - **Stress important** sur les disques restants
> - **Risque de seconde panne** pendant reconstruction
> - **Solution** : RAID 6 pour plus de sécurité

#### RAID 6 (Double parité)

> [!quote] Définition RAID 6
> **RAID 5 avec duplication de la parité**

**Principe** : n disques dont m (= 2) disques de parité

```
     ┌──────────┐
     │   Data   │
     └──────────┘
          │ split
     ┌────┴────┐
     ↓         ↓
┌────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐
│ Disk 1 │ │ Disk 2 │ │  Disk 3  │ │  Disk 4  │
│   A1   │ │   A2   │ │ Parité A │ │ Parité B │
└────────┘ └────────┘ └──────────┘ └──────────┘
                            ↑            ↑
                    (double parité différente)
```

**Caractéristiques** :
- **Disques minimum** : 4
- **Capacité totale** : (n - 2) × capacité du plus petit disque
- **Performance** : Gain en performance (striping)
- **Fiabilité** : **Excellente** - tolère 2 pannes simultanées
- **Reconstruction** : **Plus coûteuse et/ou longue** que RAID 5

> [!success] Tolérance accrue
> RAID 6 tolère **m pannes** (généralement 2) :
> - Sécurité pendant reconstruction
> - Grappes avec nombreux disques
> - Volumes très importants (temps de reconstruction long)

> [!tip] Quand utiliser RAID 6 ?
> - **Grappes > 6 disques**
> - **Données critiques** nécessitant haute disponibilité
> - **Environnements où reconstruction est longue** (disques > 4 To)
> - **Compensation du risque** de double panne pendant reconstruction

### Combinaisons de RAID

> [!info] RAID imbriqués
> Il est possible de combiner plusieurs niveaux de RAID pour obtenir des propriétés spécifiques.

#### RAID 0+1 (RAID 01)

**Principe** : Un **RAID 1** de grappes **RAID 0**

```
Niveau supérieur (RAID 1 - Mirroring)
         ┌──────────────────┐
         │     RAID 1       │
         └────────┬─────────┘
         ┌────────┴────────┐
         ↓                 ↓
    ┌─────────┐       ┌─────────┐
    │ RAID 0  │       │ RAID 0  │ (copies)
    └────┬────┘       └────┬────┘
    ┌────┴────┐       ┌────┴────┐
    ↓         ↓       ↓         ↓
 [Disk1] [Disk2]  [Disk3] [Disk4]
```

**Caractéristiques** :
- **Stripe first, then mirror**
- Perte d'un disque = perte d'une grappe RAID 0 complète
- **Moins robuste que RAID 10**

#### RAID 1+0 (RAID 10)

**Principe** : Un **RAID 0** de grappes **RAID 1**

```
Niveau supérieur (RAID 0 - Striping)
         ┌──────────────────┐
         │     RAID 0       │
         └────────┬─────────┘
         ┌────────┴────────┐
         ↓                 ↓
    ┌─────────┐       ┌─────────┐
    │ RAID 1  │       │ RAID 1  │
    └────┬────┘       └────┬────┘
    ┌────┴────┐       ┌────┴────┐
    ↓         ↓       ↓         ↓
 [Disk1] [Disk2]  [Disk3] [Disk4]
  (mirror)          (mirror)
```

**Caractéristiques** :
- **Mirror first, then stripe**
- Tolère la panne d'un disque par grappe RAID 1
- **Plus robuste que RAID 01**
- **Standard en entreprise** pour haute performance + haute disponibilité

> [!success] RAID 10 : le choix pro
> **Combine les avantages** :
> - **Performance** du RAID 0
> - **Fiabilité** du RAID 1
> - **Usage** : Bases de données critiques, serveurs de virtualisation

> [!note] Autres combinaisons
> De nombreuses autres variantes existent :
> - **RAID 50** : RAID 0 de RAID 5
> - **RAID 60** : RAID 0 de RAID 6
> - **RAID 100** : RAID 0 de RAID 10

### Considérations générales RAID

> [!tip] Bonnes pratiques RAID

#### Homogénéité des disques

> [!important] Utilisation de disques identiques
> **Toujours utiliser des disques identiques** au sein d'une grappe :
> - **Même capacité** (la plus petite fait référence)
> - **Même modèle** (performances homogènes)
> - **Même fabricant** (fiabilité similaire)
> - **Même lot si possible** (éviter vieillissement différent)

#### Disque de spare (secours)

> [!note] Hot spare
> **Disque de spare** = disque de secours pré-installé
> 
> Permet une **reconstruction automatique** dès détection de panne :
> 1. Panne détectée sur un disque
> 2. Disque de spare activé automatiquement
> 3. Reconstruction de la grappe lancée
> 4. Temps d'intervention réduit

En général associé à des disques **hot plug** (remplaçables à chaud, sans arrêt du système)

#### RAID et sauvegardes

> [!warning] RAID ≠ Sauvegarde
> **Le RAID ne remplace PAS les sauvegardes !**
> 
> Le RAID protège contre :
> - ✅ Pannes matérielles (disques défectueux)
> - ✅ Amélioration disponibilité
> 
> Le RAID ne protège PAS contre :
> - ❌ Suppression accidentelle de fichiers
> - ❌ Corruption de données
> - ❌ Ransomware / malware
> - ❌ Erreurs humaines
> - ❌ Catastrophes (incendie, inondation)

> [!important] Règle d'or
> **RAID = Disponibilité | Sauvegarde = Pérennité**
> 
> Une stratégie complète nécessite **RAID + Sauvegardes régulières**

---

## LVM - Logical Volume Manager

> [!quote] Définition LVM
> **LVM** = **L**ogical **V**olume **M**anager
> 
> Système de virtualisation du stockage permettant :
> - Plus de **flexibilité** et de facilité de gestion
> - Des **instantanés** (snapshots)
> - L'**entrelacement** (striping)
> - La **mise en miroir** (mirroring)
> - Opérations **à chaud** (création, redimensionnement, etc.)

### Concept et architecture LVM

> [!abstract] Architecture en couches
> LVM ajoute une couche d'abstraction entre les disques physiques et les systèmes de fichiers, offrant une flexibilité sans précédent.

#### Schéma d'architecture global

```
┌─────────────────────────────────────────────────────────┐
│                  Systèmes de Fichiers                   │
│         ext4        XFS        btrfs        NTFS         │
└───────────┬──────────────┬──────────────┬───────────────┘
            ↓              ↓              ↓
┌─────────────────────────────────────────────────────────┐
│              Logical Volumes (LV)                       │
│       /dev/vg01/lv_root    /dev/vg01/lv_home           │
└───────────────────────┬─────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│              Volume Groups (VG)                         │
│                    vg01                                 │
└───────┬────────────────────┬────────────────────────────┘
        ↓                    ↓
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ Physical Vol │      │ Physical Vol │      │ Physical Vol │
│    (PV)      │      │    (PV)      │      │    (PV)      │
│  /dev/sda1   │      │  /dev/sdb1   │      │  /dev/sdc    │
└──────┬───────┘      └──────┬───────┘      └──────┬───────┘
       ↓                     ↓                     ↓
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Disque 1   │      │   Disque 2   │      │   Disque 3   │
│   Partition  │      │   Partition  │      │Disque complet│
└──────────────┘      └──────────────┘      └──────────────┘
```

> [!info] Couches LVM
> **De bas en haut** :
> 1. **Disques/Partitions physiques**
> 2. **Physical Volumes (PV)** : disques initialisés pour LVM
> 3. **Volume Group (VG)** : pool d'espace de stockage
> 4. **Logical Volumes (LV)** : « partitions » LVM
> 5. **Systèmes de fichiers** : formatage des LV

### Composants LVM

#### Physical Volumes (PV)

> [!quote] Définition PV
> Un périphérique de stockage = Un volume physique (PV)

**Types de périphériques supportés** :
- Une **partition** (recommandé)
- Un **disque complet**
- Un **volume RAID**
- Un **stockage distant** (SAN)

> [!tip] Recommandation importante
> **Faire une partition même pour un disque complet**
> 
> Créer un MBR ou un GPT avec une partition étiquetée **LVM** :
> - Évite que les logiciels ne gérant pas LVM considèrent le disque vide
> - Permet une identification claire du disque
> - Facilite la gestion et le dépannage

> [!example] Création d'un PV
> ```bash
> # Créer une partition LVM avec fdisk/gdisk
> fdisk /dev/sdb
> # Créer partition, type 8e (LVM)
> 
> # Initialiser le PV
> pvcreate /dev/sdb1
> 
> # Lister les PV
> pvdisplay
> pvs  # version courte
> ```

#### Volume Groups (VG)

> [!quote] Définition VG
> Un groupe de volumes (VG) = un ensemble de (au moins 1) PV

**Caractéristiques** :
- **Couche d'abstraction** centrale de LVM
- **Taille du VG** = ∑ tailles des PV
- **Unité d'allocation** : **Physical Extents (PE)** (par défaut 4 Mo)

> [!important] Analogie
> Le VG est comme un **grand disque virtuel** composé de tous les PV. On y découpe ensuite des LV comme on découperait des partitions.

> [!tip] Recommandations VG
> - **Un VG par type de PV** (HDD vs SSD vs NVMe)
> - Regrouper des PV aux **caractéristiques similaires** :
>   - Performance (vitesse similaire)
>   - Fiabilité (RAID similaire)
>   - Usage (données similaires)

> [!example] Gestion des VG
> ```bash
> # Créer un VG avec 2 PV
> vgcreate vg_data /dev/sdb1 /dev/sdc1
> 
> # Ajouter un PV à un VG existant
> vgextend vg_data /dev/sdd1
> 
> # Afficher les VG
> vgdisplay
> vgs  # version courte
> ```

#### Logical Volumes (LV)

> [!quote] Définition LV
> Un VG est découpé en volumes logiques (LV) qui correspondent aux partitions habituelles.

**Caractéristiques** :
- **Taille exprimée** en nombre de **Logical Extents (LE)**
- **Taille des LE = taille des PE** (par défaut 4 Mo)
- **Supportent un système de fichiers** (ext4, XFS, NTFS, etc.)
- **Peuvent être en RAID** (0, 1, 4, 5, 6, 10) - RAID logiciel LVM
- **Peuvent être un snapshot** d'un autre LV

> [!info] Flexibilité des LV
> Les LV sont **totalement flexibles** :
> - Créés **à la demande**
> - **Redimensionnables** à chaud (agrandissement)
> - **Déplaçables** entre PV
> - **Supprimables** sans affecter les autres LV

> [!example] Gestion des LV
> ```bash
> # Créer un LV de 50 Go
> lvcreate -L 50G -n lv_home vg_data
> 
> # Créer un LV utilisant 100% de l'espace libre
> lvcreate -l 100%FREE -n lv_backup vg_data
> 
> # Formater le LV
> mkfs.ext4 /dev/vg_data/lv_home
> 
> # Monter le LV
> mount /dev/vg_data/lv_home /home
> 
> # Afficher les LV
> lvdisplay
> lvs  # version courte
> ```

### Snapshots et Copy-On-Write

#### Mécanisme Copy-On-Write (COW)

> [!quote] Définition COW
> **Copy-On-Write (COW)** = Copie à l'écriture
> 
> La création de copies instantanées est basée sur ce mécanisme ingénieux.

**Principe** :
1. **À la création du snapshot** : les deux LV (original et snapshot) **référencent les mêmes données**
   - Pas de copie physique immédiate
   - Métadonnées partagées
2. **Lors de modifications** : 
   - Données originales copiées vers le snapshot **avant** modification
   - Modifications effectuées sur le LV original
   - Le snapshot conserve l'état initial

```
État initial (création snapshot) :
┌──────────────────────────────┐
│     LV Original              │
│  Blocs: A B C D E F          │
└──────────────────────────────┘
         ↑ ↑ ↑ ↑ ↑ ↑
         └─┴─┴─┴─┴─┴──┐
                      │
┌──────────────────────────────┐
│     LV Snapshot              │
│  Référence: A B C D E F      │
│  Stocke: (rien)              │
└──────────────────────────────┘

Après modification du bloc B :
┌──────────────────────────────┐
│     LV Original              │
│  Blocs: A B' C D E F         │ (B modifié → B')
└──────────────────────────────┘

┌──────────────────────────────┐
│     LV Snapshot              │
│  Référence: A C D E F        │
│  Stocke: B (copie avant modif)│
└──────────────────────────────┘
```

> [!success] Conséquences du COW
> - **Création d'un snapshot très rapide** (quelques secondes)
> - **Snapshot consomme peu de place** au départ (métadonnées + modifications)
> - **Taille du snapshot croît** avec les modifications du LV original

> [!warning] Dimensionnement du snapshot
> La taille du snapshot doit être suffisante pour stocker :
> - Les **métadonnées** de suivi
> - **Toutes les modifications** du LV original pendant la durée de vie du snapshot
> 
> Si le snapshot est plein → **invalidation automatique** du snapshot

#### Cas d'usage des snapshots

> [!example] Utilisations pratiques
> 
> **1. Sauvegarde cohérente** :
> ```bash
> # Créer snapshot de la base de données
> lvcreate -L 10G -s -n snap_db /dev/vg_data/lv_database
> 
> # Monter le snapshot en lecture seule
> mount -o ro /dev/vg_data/snap_db /mnt/backup
> 
> # Effectuer la sauvegarde
> tar czf /backup/db_backup.tar.gz /mnt/backup
> 
> # Supprimer le snapshot
> umount /mnt/backup
> lvremove /dev/vg_data/snap_db
> ```
> 
> **2. Tests avant mise à jour** :
> ```bash
> # Snapshot avant update système
> lvcreate -L 5G -s -n snap_root /dev/vg_system/lv_root
> 
> # Effectuer la mise à jour
> apt upgrade
> 
> # Si problème : restaurer depuis snapshot
> lvconvert --merge /dev/vg_system/snap_root
> reboot
> ```
> 
> **3. Environnement de développement** :
> - Créer snapshot d'un environnement de prod
> - Tester modifications sur snapshot
> - Supprimer snapshot après tests

### Redimensionnement à chaud

> [!info] Flexibilité LVM
> LVM supporte le **redimensionnement à chaud** des LV, une fonctionnalité majeure de gestion du stockage.

#### Agrandissement d'un LV

> [!success] Opération sûre
> On peut **agrandir un LV** (dans la limite de la place disponible dans le VG)
> 
> Cette opération est **sûre** et **sans risque de perte de données**.

**Procédure** :
```bash
# 1. Vérifier l'espace disponible dans le VG
vgs

# 2. Agrandir le LV (+20 Go)
lvextend -L +20G /dev/vg_data/lv_home

# 3. Agrandir le système de fichiers
# Pour ext4 :
resize2fs /dev/vg_data/lv_home
# Pour XFS :
xfs_growfs /dev/vg_data/lv_home

# OU tout en une seule commande :
lvextend -L +20G -r /dev/vg_data/lv_home  # -r = resize FS
```

> [!note] Compatibilité systèmes de fichiers
> **Agrandissement à chaud supporté par** :
> - ✅ **ext4** (resize2fs)
> - ✅ **XFS** (xfs_growfs)
> - ✅ **btrfs**
> - ❌ **NTFS** (nécessite redémarrage Windows)

#### Réduction d'un LV

> [!warning] Opération risquée
> On peut **rétrécir un LV** mais c'est une opération **RISQUÉE**.
> 
> **Attention aux données présentes !** LVM n'a pas connaissance de l'emplacement des données.

> [!important] Limitations
> Tous les systèmes de fichiers ne supportent **PAS** la réduction :
> - ✅ **ext4** : oui (réduction possible)
> - ❌ **XFS** : **NON** (réduction impossible)
> - ✅ **btrfs** : oui

##### Procédure de réduction sécurisée

> [!tip] Bonne pratique pour réduire
> Pour limiter les risques lors de diminution de la taille :

**Étapes recommandées** :
1. **Démonter le système de fichiers** (obligatoire pour ext4)
2. **Vérifier le système de fichiers** (`e2fsck -f`)
3. **Réduire la taille du système de fichiers** (déplace les données)
   - Choisir une taille **légèrement inférieure** à la cible
4. **Réduire la taille du LV**
5. **Modifier la taille du FS** en le laissant prendre toute la place disponible

```bash
# DANGER : sauvegarde obligatoire avant !

# 1. Démonter
umount /dev/vg_data/lv_home

# 2. Vérifier le FS
e2fsck -f /dev/vg_data/lv_home

# 3. Réduire le FS à 45G (pour cible 50G)
resize2fs /dev/vg_data/lv_home 45G

# 4. Réduire le LV à 50G
lvreduce -L 50G /dev/vg_data/lv_home

# 5. Agrandir le FS pour occuper tout l'espace
resize2fs /dev/vg_data/lv_home

# 6. Remonter
mount /dev/vg_data/lv_home /home
```

> [!warning] Risque majeur
> Une **mauvaise manipulation = perte de données**
> 
> **Sauvegarde obligatoire avant toute réduction !**

#### Augmenter l'espace d'un VG

> [!tip] Ajouter de l'espace au VG
> Quand un VG n'a plus assez d'espace disponible ⇒ **Ajout d'un nouveau PV**

**Procédure** :
```bash
# 1. Créer le PV sur nouveau disque
pvcreate /dev/sde1

# 2. Étendre le VG avec ce nouveau PV
vgextend vg_data /dev/sde1

# 3. L'espace est maintenant disponible pour tous les LV du VG
vgs  # voir augmentation de VFree

# 4. Agrandir un LV si nécessaire
lvextend -L +100G /dev/vg_data/lv_home
resize2fs /dev/vg_data/lv_home
```

> [!success] Avantage majeur
> L'espace ajouté par le nouveau disque devient **disponible pour l'ensemble des LV** du VG, sans limites rigides de partitionnement.

---

## SAN & NAS - Stockage en réseau

> [!abstract] Vue d'ensemble
> Alternative au stockage local ⇒ **stockage via le réseau**
> 
> Objectifs :
> - **Mutualisation** à l'échelle d'un ensemble de machines
> - **Gestion centralisée** pour plusieurs machines
> - **Machines déplaçables** (notamment machines virtuelles)

### Contexte et approches

> [!info] Deux approches complémentaires
> Il existe deux approches principales pour le stockage en réseau :
> - **NAS** : Network Attached Storage (serveur de fichiers)
> - **SAN** : Storage Area Network (réseau de stockage)

### NAS - Network Attached Storage

> [!quote] Définition NAS
> **NAS** = **N**etwork **A**ttached **S**torage
> 
> Serveur de stockage en réseau fournissant un accès aux systèmes de fichiers via le réseau.

#### Concept

> [!info] Idée ancienne
> **Accéder à un système de fichiers distant**
> 
> Le NAS présente des **répertoires partagés** accessibles par le réseau, comme des disques locaux.

#### Protocoles NAS

> [!note] Nombreux protocoles disponibles

| Protocole | Système | Description | Usage typique |
|-----------|---------|-------------|---------------|
| **NFS** | Unix/Linux | Network File System - Classique monde Unix | Serveurs Linux, virtualisation |
| **SMB/CIFS** | Windows | Server Message Block - Approche Windows | Environnements Windows, mixtes |
| **AFP** | macOS | Apple Filing Protocol (obsolète) | Ancien macOS (< 10.9) |
| **FTP/SFTP** | Universel | File Transfer Protocol | Transferts ponctuels |
| **WebDAV** | Universel | HTTP étendu | Services cloud, web |

##### NFS (Network File System)

> [!info] Protocole Unix standard
> **Permet de monter un système de fichiers distant** comme s'il était local.

**Caractéristiques** :
- Standard dans le monde **Unix/Linux**
- Transparent pour les applications
- Performance correcte en réseau local
- Versions : NFSv3 (ancien), **NFSv4** (moderne, sécurisé)

```bash
# Exemple montage NFS
mount -t nfs 192.168.1.100:/export/data /mnt/nas

# Montage persistant dans /etc/fstab
192.168.1.100:/export/data /mnt/nas nfs defaults 0 0
```

##### SMB/CIFS (Server Message Block)

> [!info] Protocole Windows
> **Permet d'avoir des disques distants** (et plus : imprimantes, etc.)

**Caractéristiques** :
- Standard dans le monde **Windows**
- Supporté sous Linux via **Samba**
- Gestion des droits Windows (ACL)
- Protocole plus lourd que NFS

```bash
# Exemple montage SMB sous Linux
mount -t cifs //192.168.1.100/partage /mnt/nas -o username=user

# Montage persistant dans /etc/fstab
//192.168.1.100/partage /mnt/nas cifs credentials=/root/.smbcreds 0 0
```

#### Avantages et inconvénients NAS

**Avantages** :
- ✅ **Simplicité** de mise en œuvre
- ✅ **Facilité** d'administration
- ✅ **Économique** (matériel standard)
- ✅ **Partage** facile entre plusieurs OS

**Inconvénients** :
- ❌ **Impact fort sur le réseau** ⇒ nécessite un réseau de bonne qualité
- ❌ **Performance** limitée par le réseau (latence)
- ❌ **Goulot d'étranglement** si nombreux clients
- ❌ **Sécurité** : trafic sur réseau partagé

> [!warning] Exigence réseau
> Le NAS nécessite un **réseau de bonne qualité** :
> - Bande passante suffisante (Gigabit minimum, 10 Gigabit recommandé)
> - Latence faible
> - Réseau dédié recommandé pour forte charge

> [!example] Cas d'usage NAS
> - **PME** : serveur de fichiers centralisé
> - **Bureaux** : partage de documents
> - **Multimédia** : bibliothèque vidéo/photo
> - **Sauvegardes** : cible de backup

### SAN - Storage Area Network

> [!quote] Définition SAN
> **SAN** = **S**torage **A**rea **N**etwork
> 
> Réseau spécialisé (dédié) au stockage/sauvegarde.

#### Concept

> [!info] Accès bas niveau
> **Principe** : Accès distant à des **périphériques de blocs** (bas niveau)
> 
> Le SAN fournit des **volumes bruts** (comme des disques locaux virtuels) sur lesquels on crée ensuite partitions et systèmes de fichiers.

#### Architecture SAN

**Composants** :
- **Baies de stockage** : serveurs spécialisés haute performance
  - Disques nombreux en grand RAID
  - Contrôleurs redondants
  - Cache important
- **Réseau dédié** : infrastructure réseau séparée
- **HBA** : Host Bus Adapter (cartes d'interface SAN côté serveur)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Serveur 1   │    │  Serveur 2   │    │  Serveur 3   │
│     (HBA)    │    │     (HBA)    │    │     (HBA)    │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       └───────────┬───────┴───────────┬───────┘
                   │  Réseau SAN       │
       ┌───────────┴───────────────────┴───────────┐
       │        Switch Fibre Channel               │
       └───────────┬───────────────────┬───────────┘
                   │                   │
       ┌───────────┴───────┐   ┌───────┴───────────┐
       │   Baie stockage 1 │   │   Baie stockage 2 │
       │   (RAID 6)        │   │   (RAID 10)       │
       └───────────────────┘   └───────────────────┘
```

#### Protocoles SAN

> [!note] Protocoles spécialisés

| Protocole | Description | Caractéristiques |
|-----------|-------------|------------------|
| **Fibre Channel (FC)** | SCSI sur LAN spécifique | Haute performance, coûteux, jusqu'à 128 Gb/s |
| **iSCSI** | SCSI sur IP | Utilise réseau Ethernet standard, plus économique |
| **FCoE** | Fibre Channel sur Ethernet | Convergence FC et Ethernet, 10 Gigabit minimum |

##### Fibre Channel

> [!info] Standard haute performance
> **SCSI sur un LAN spécifique** (pas Ethernet standard)

**Caractéristiques** :
- **Performance maximale** (16, 32, 64, 128 Gb/s)
- **Latence ultra-faible**
- **Fiabilité excellente**
- **Coût élevé** (équipement spécialisé)
- **Câbles optiques** dédiés

> [!example] Usage
> Datacenters d'entreprise, infrastructures critiques

##### iSCSI (Internet SCSI)

> [!info] SCSI sur IP
> **Encapsulation du protocole SCSI dans TCP/IP**

**Caractéristiques** :
- **Utilise réseau Ethernet standard**
- **Plus économique** que Fibre Channel
- **Performance bonne** (limite du réseau Ethernet)
- **Facilité** de mise en œuvre
- **Sécurité** : IPsec, CHAP

```bash
# Exemple connexion iSCSI sous Linux
# 1. Découvrir les cibles iSCSI
iscsiadm -m discovery -t sendtargets -p 192.168.1.200

# 2. Se connecter à une cible
iscsiadm -m node --targetname iqn.2024.com.example:storage.disk1 --login

# 3. Le volume apparaît comme /dev/sdX
# On peut alors créer partitions, LVM, système de fichiers
```

> [!example] Usage
> PME, virtualisation, cloud privé

##### FCoE (Fibre Channel over Ethernet)

> [!info] Convergence des protocoles
> **Fibre Channel sur Ethernet** : utilise trames Ethernet pour transport FC

**Caractéristiques** :
- **10 Gigabit Ethernet minimum**
- **Convergence réseau** : données + stockage sur même infrastructure
- **Moins répandu** que FC ou iSCSI
- **Complexité** moyenne

#### Avantages et inconvénients SAN

**Avantages** :
- ✅ **Performance maximale** (accès bloc, réseau dédié)
- ✅ **Centralisation** et mutualisation du stockage
- ✅ **Flexibilité** : allocation dynamique de volumes
- ✅ **Haute disponibilité** : chemins multiples, redondance
- ✅ **Virtualisation** : migration de VM facilitée

**Inconvénients** :
- ❌ **Coût élevé** (matériel spécialisé)
- ❌ **Complexité** de mise en œuvre et administration
- ❌ **Expertise** requise
- ❌ **Évolutivité** : ajout de baies coûteux

> [!success] Cas d'usage SAN
> - **Datacenters** d'entreprise
> - **Virtualisation** massive (VMware vSphere, Hyper-V)
> - **Bases de données** critiques haute performance
> - **Clusters** haute disponibilité
> - **Cloud privé** d'entreprise

### Comparaison NAS vs SAN

| Critère | NAS | SAN |
|---------|-----|-----|
| **Accès** | Fichiers (haut niveau) | Blocs (bas niveau) |
| **Protocoles** | NFS, SMB/CIFS, FTP | FC, iSCSI, FCoE |
| **Réseau** | Ethernet standard | Dédié (FC) ou Ethernet (iSCSI/FCoE) |
| **Performance** | Moyenne (latence réseau) | Excellente (bas niveau) |
| **Coût** | Économique | Élevé |
| **Complexité** | Simple | Complexe |
| **Partage** | Multi-OS facile | Nécessite gestionnaire de volumes |
| **Usage typique** | PME, bureaux, partage | Datacenter, virtualisation, DB |

> [!tip] Choix NAS ou SAN ?
> - **NAS** : partage de fichiers, PME, simplicité, budget limité
> - **SAN** : performance maximale, haute disponibilité, virtualisation, datacenter

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Introduction au stockage avancé

- **Trois critères d'analyse** : Volume, Performance, Sûreté
- **HDD** : 20-30 €/To, ≈100 Mo/s, volumes jusqu'à 24 To
- **SSD** : ≈100 €/To, 3-7 Go/s, performances supérieures
- **Fiabilité variable** : pannes mécaniques (HDD), usure (SSD)
- **Cloisonnement nécessaire** : isoler `/var/log`, bases de données, `/home`, swap, `/tmp`
- **Limites du partitionnement classique** : nombre limité, taille fixe, évolution difficile

### RAID - Concepts essentiels

- **Objectifs RAID** : agrandir taille, améliorer performances, améliorer fiabilité
- **3 types d'implémentation** : matériel (performant, cher), logiciel (flexible, consomme CPU), hybride (à éviter)
- **Linux** : outil `mdadm` pour RAID logiciel

#### Niveaux RAID à connaître absolument

| Niveau | Disques | Capacité | Performance | Fiabilité | Usage |
|--------|---------|----------|-------------|-----------|-------|
| **JBOD** | 2+ | n×capacité | Standard | Médiocre | Non-prod |
| **RAID 0** | 2 | n×capacité | ★★★ | Danger | Scratch, cache |
| **RAID 1** | 2 | 1×capacité | Standard | ★★★ | OS, critique |
| **RAID 5** | 3+ | (n-1)×capacité | ★★ | ★★ | Compromis standard |
| **RAID 6** | 4+ | (n-2)×capacité | ★★ | ★★★ | Haute fiabilité |
| **RAID 10** | 4+ | n/2×capacité | ★★★ | ★★★ | Pro haute perf |

- **RAID 0** : Striping, performance max, perte 1 disque = tout perdu
- **RAID 1** : Mirroring, fiabilité max, perte 50% capacité
- **RAID 5** : Parité répartie, compromis standard entreprise, tolère 1 panne
- **RAID 6** : Double parité, tolère 2 pannes, grappes importantes
- **RAID 10** : Mirror puis stripe, standard pro haute performance

#### Points critiques RAID

- **Disques identiques** dans une grappe (modèle, capacité, fabricant)
- **Hot spare** : reconstruction automatique, disques hot-plug
- **RAID ≠ Sauvegarde** : RAID = disponibilité, Sauvegarde = pérennité
- **RAID ne protège PAS** : suppression accidentelle, corruption, ransomware, erreurs humaines

### LVM - Architecture et composants

- **Architecture 3 couches** : PV → VG → LV → FS
- **PV** (Physical Volume) : partition ou disque initialisé pour LVM
- **VG** (Volume Group) : pool d'espace, somme des PV, unité PE (4 Mo)
- **LV** (Logical Volume) : "partition" LVM, porte système de fichiers
- **Recommandation** : partition LVM même pour disque complet (étiquette)

#### Fonctionnalités LVM avancées

- **Snapshot (COW)** : création rapide, consomme peu, métadonnées + modifs
- **Redimensionnement à chaud** :
  - **Agrandissement** : sûr (lvextend + resize2fs/xfs_growfs)
  - **Réduction** : risqué, sauvegarde obligatoire, XFS impossible
- **Ajout de PV** : étendre VG à chaud, espace disponible pour tous LV
- **Striping/Mirroring** : LVM peut faire RAID logiciel

#### Commandes LVM essentielles

```bash
# PV
pvcreate /dev/sdb1
pvs / pvdisplay

# VG
vgcreate vg_name /dev/sdb1 /dev/sdc1
vgextend vg_name /dev/sdd1
vgs / vgdisplay

# LV
lvcreate -L 50G -n lv_name vg_name
lvextend -L +20G -r /dev/vg_name/lv_name  # -r = resize FS
lvs / lvdisplay

# Snapshot
lvcreate -L 10G -s -n snap_name /dev/vg_name/lv_name
lvremove /dev/vg_name/snap_name
```

### SAN & NAS - Stockage réseau

#### NAS (Network Attached Storage)

- **Principe** : serveur de fichiers, accès systèmes de fichiers distants
- **Protocoles** : NFS (Unix/Linux), SMB/CIFS (Windows), FTP, WebDAV
- **Avantages** : simplicité, économique, multi-OS
- **Inconvénients** : impact réseau, performance limitée
- **Usage** : PME, bureaux, partage fichiers, sauvegardes

#### SAN (Storage Area Network)

- **Principe** : réseau dédié, accès périphériques de blocs (bas niveau)
- **Protocoles** : Fibre Channel (haute perf), iSCSI (IP), FCoE (convergence)
- **Architecture** : baies de stockage + réseau dédié + HBA
- **Avantages** : performance max, centralisation, HA, virtualisation
- **Inconvénients** : coût élevé, complexité, expertise requise
- **Usage** : datacenters, virtualisation massive, bases de données critiques

#### NAS vs SAN

| Aspect | NAS | SAN |
|--------|-----|-----|
| Accès | Fichiers | Blocs |
| Niveau | Haut | Bas |
| Coût | € | €€€ |
| Complexité | Simple | Complexe |
| Performance | Moyenne | Excellente |

### Mnémotechniques RAID

> [!tip] Mémo RAID
> - **RAID 0** : **0** protection = **0** fiabilité (mais rapide !)
> - **RAID 1** : **1** miroir = **1** copie de secours
> - **RAID 5** : **5** doigts, on en perd **1** (parité) = reste **4** (données)
> - **RAID 6** : **6** doigts, on en perd **2** (double parité) = reste **4** (données)
> - **RAID 10** : **1** + **0** = Miroir (1) puis Stripe (0)

### Points transversaux

> [!important] Vision globale
> En tant que **TSSR**, tu dois :
> - **Analyser les besoins** : volume, performance, sûreté
> - **Choisir la technologie** : RAID (fiabilité/perf), LVM (flexibilité), SAN/NAS (centralisation)
> - **Combiner les solutions** : RAID + LVM = optimal
> - **Anticiper l'évolution** : LVM facilite ajout d'espace
> - **Sécuriser** : RAID ≠ sauvegarde, stratégie globale nécessaire
> - **Documenter** : configuration RAID, schéma LVM, plan de reprise

> [!warning] À ne jamais oublier
> - **Toujours faire des sauvegardes** (RAID ne suffit pas)
> - **Tester les restaurations** régulièrement
> - **Désactiver SMBv1** (faille sécurité)
> - **Disques identiques** dans grappes RAID
> - **Sauvegarde avant réduction LV**

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **AFP** | Apple Filing Protocol : ancien protocole de partage fichiers macOS |
| **Array** | Grappe ou cluster RAID : ensemble de disques constituant un volume RAID |
| **Baie de stockage** | Serveur spécialisé contenant nombreux disques en RAID, pour SAN |
| **COW** | Copy-On-Write (Copie à l'écriture) : mécanisme des snapshots LVM |
| **ext4** | Extended File System v4 : système de fichiers Linux standard |
| **FCoE** | Fibre Channel over Ethernet : protocole SAN utilisant Ethernet |
| **Fibre Channel (FC)** | Protocole SAN haute performance sur réseau dédié optique |
| **HAMR** | Heat-Assisted Magnetic Recording : technologie HDD haute capacité (>30 To) |
| **HBA** | Host Bus Adapter : carte d'interface SAN côté serveur |
| **Hot plug** | Remplacement à chaud d'un composant sans arrêt système |
| **Hot spare** | Disque de secours pré-installé pour reconstruction automatique RAID |
| **iSCSI** | Internet SCSI : protocole SAN encapsulant SCSI dans TCP/IP |
| **JBOD** | Just a Bunch Of Disks : concaténation simple de disques |
| **LE** | Logical Extent : unité d'allocation dans un LV (= taille PE) |
| **LV** | Logical Volume : "partition" LVM portant un système de fichiers |
| **LVM** | Logical Volume Manager : système de virtualisation du stockage |
| **mdadm** | Multiple Device Administrator : outil RAID logiciel sous Linux |
| **Mirroring** | Mise en miroir : duplication de données (RAID 1) |
| **NAS** | Network Attached Storage : serveur de fichiers en réseau |
| **NFS** | Network File System : protocole partage fichiers Unix/Linux |
| **NVMe** | Non-Volatile Memory Express : interface SSD ultra-rapide |
| **Parité** | Données de contrôle permettant reconstruction après panne (RAID 4/5/6) |
| **PE** | Physical Extent : unité d'allocation dans un VG (défaut 4 Mo) |
| **PV** | Physical Volume : disque ou partition initialisé pour LVM |
| **RAID** | Redundant Array of Independent Disks : virtualisation stockage multi-disques |
| **RAID 0** | Striping : entrelaçage, performance max, aucune fiabilité |
| **RAID 1** | Mirroring : duplication, fiabilité max |
| **RAID 4** | Striping avec parité dédiée : compromis, disque parité = goulot |
| **RAID 5** | Striping avec parité répartie : compromis standard entreprise |
| **RAID 6** | Striping avec double parité : tolère 2 pannes |
| **RAID 10** | Miroir puis stripe : haute performance + haute fiabilité |
| **SAN** | Storage Area Network : réseau dédié au stockage, accès bloc |
| **SCSI** | Small Computer System Interface : protocole communication disques |
| **SLED** | Single Large Expensive Disk : disque unique coûteux (opposé RAID) |
| **SMB/CIFS** | Server Message Block : protocole partage fichiers Windows |
| **SMR** | Shingled Magnetic Recording : technologie HDD haute capacité |
| **Snapshot** | Copie instantanée d'un LV à un instant T (COW) |
| **Spare** | Disque de secours pour reconstruction automatique |
| **Striping** | Entrelaçage : répartition des données sur plusieurs disques (RAID 0) |
| **VG** | Volume Group : pool d'espace LVM regroupant plusieurs PV |
| **WebDAV** | Web Distributed Authoring and Versioning : extension HTTP pour gestion documents |
| **XFS** | eXtended File System : système de fichiers Linux haute performance |
| **XOR** | OU exclusif : opération logique pour calcul parité RAID |

---

## 📚 Pour aller plus loin

> [!tip] Systèmes de fichiers nouvelle génération
> Technologies avancées combinant gestion volumes et systèmes de fichiers :
> 
> **ZFS** (Zettabyte File System) :
> - Système de fichiers + gestionnaire de volumes
> - Snapshots intégrés, compression, déduplication
> - Très utilisé dans FreeBSD, Solaris, TrueNAS
> - Protection intégrité données (checksums)
> 
> **btrfs** (B-tree File System) :
> - Alternative Linux moderne à ext4
> - Snapshots, RAID intégré, CoW natif
> - Compression transparente, déduplication
> - En développement actif

> [!note] Ressources complémentaires
> - Documentation officielle LVM : `man lvm`
> - Guide mdadm : `man mdadm`
> - RFC iSCSI : RFC 3720
> - Linux RAID Wiki : https://raid.wiki.kernel.org
> - Bonnes pratiques RAID Dell/HP/IBM (documentation constructeurs)

---

> [!success] Document de révision complet
> Ce document couvre l'intégralité du cours sur le stockage avancé (RAID/LVM/SAN-NAS). Il est conçu pour une révision efficace avant le titre RNCP TSSR. N'oublie pas de pratiquer les commandes en lab !

**Bonne révision Franck ! 💾🚀**