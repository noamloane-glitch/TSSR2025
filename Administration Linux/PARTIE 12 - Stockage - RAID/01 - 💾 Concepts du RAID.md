

## 📋 Table des matières

```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 2 # Include headings from the specified level
maxLevel: 2 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```

---

## 🎯 Introduction au RAID

**RAID** (Redundant Array of Independent Disks) est une technologie qui combine plusieurs disques physiques en une seule unité logique pour améliorer les performances, la fiabilité, ou les deux.

> [!info] Pourquoi utiliser le RAID ?
> 
> - **Performance** : Augmenter les vitesses de lecture/écriture
> - **Redondance** : Protéger contre la perte de données en cas de défaillance disque
> - **Disponibilité** : Permettre le remplacement à chaud (hot-swap) des disques défaillants
> - **Capacité** : Agréger l'espace de plusieurs disques

---

## 🧩 Concepts fondamentaux

### Striping

Le **striping** consiste à diviser les données en blocs et à les répartir sur plusieurs disques.

```
Données : A B C D E F G H
Disque 1: A   C   E   G
Disque 2:   B   D   F   H
```

**Avantages** :

- Amélioration des performances de lecture/écriture
- Parallélisation des opérations d'I/O
- Utilisation maximale de la capacité totale

**Inconvénients** :

- Aucune redondance
- La défaillance d'un seul disque = perte totale des données

### Mirroring

Le **mirroring** copie identiquement les données sur plusieurs disques.

```
Données : A B C D E F G H
Disque 1: A B C D E F G H
Disque 2: A B C D E F G H (copie identique)
```

**Avantages** :

- Redondance complète des données
- Récupération simple en cas de panne
- Amélioration des performances de lecture

**Inconvénients** :

- Utilisation de 50% (ou moins) de la capacité totale
- Coût élevé en stockage

### Parité

La **parité** calcule des informations de contrôle permettant de reconstruire les données perdues.

```
Données : A B C → Parité: P (calculée depuis A⊕B⊕C)
Disque 1: A
Disque 2: B
Disque 3: C
Disque 4: P
```

**Principe** : Si un disque tombe en panne, ses données peuvent être recalculées à partir des autres disques et de la parité.

**Avantages** :

- Meilleur compromis capacité/redondance que le mirroring
- Récupération possible après une panne disque

**Inconvénients** :

- Calculs de parité = impact sur les performances d'écriture
- Reconstruction longue et intensive en ressources

---

## 📊 Niveaux de RAID courants

### RAID 0 - Striping

**Configuration** : Minimum 2 disques

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Disque 1│  │ Disque 2│  │ Disque 3│
│  A C E  │  │  B D F  │  │  G H I  │
└─────────┘  └─────────┘  └─────────┘
```

> [!example] Caractéristiques RAID 0
> 
> - **Capacité utilisable** : 100% (somme de tous les disques)
> - **Redondance** : Aucune
> - **Performance lecture** : Excellente (N × vitesse d'un disque)
> - **Performance écriture** : Excellente (N × vitesse d'un disque)
> - **Tolérance aux pannes** : 0 disque

**Cas d'usage** :

- Applications nécessitant des performances maximales
- Données temporaires ou facilement reconstituables
- Caches, fichiers de swap, traitement vidéo
- Environnements de développement

> [!warning] Attention ! RAID 0 n'offre AUCUNE redondance. La perte d'un seul disque = perte totale des données. Ne JAMAIS utiliser pour des données critiques.

**Exemple de calcul** :

- 3 disques de 1 To chacun
- Capacité totale : 3 To
- Si 1 disque meurt → perte des 3 To

---

### RAID 1 - Mirroring

**Configuration** : Minimum 2 disques (généralement par paires)

```
┌─────────┐  ┌─────────┐
│ Disque 1│  │ Disque 2│
│ A B C D │  │ A B C D │ (miroir exact)
└─────────┘  └─────────┘
```

> [!example] Caractéristiques RAID 1
> 
> - **Capacité utilisable** : 50% (taille du plus petit disque)
> - **Redondance** : Complète
> - **Performance lecture** : Bonne (lecture possible sur les 2 disques)
> - **Performance écriture** : Normale (limitée par l'écriture sur tous les miroirs)
> - **Tolérance aux pannes** : N-1 disques (dans une configuration à N disques)

**Cas d'usage** :

- Données critiques nécessitant une haute disponibilité
- Systèmes d'exploitation
- Bases de données (journaux de transactions)
- Serveurs de fichiers d'entreprise
- Situations où la redondance prime sur la capacité

**Exemple de calcul** :

- 2 disques de 1 To chacun
- Capacité totale : 1 To (50%)
- Si 1 disque meurt → 0 perte de données, système continue de fonctionner

> [!tip] Astuce Avec RAID 1, privilégiez des disques de même taille. Si vous utilisez un disque de 1 To et un de 2 To, seul 1 To sera utilisable.

---

### RAID 5 - Striping avec parité distribuée

**Configuration** : Minimum 3 disques

```
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ Disque 1│  │ Disque 2│  │ Disque 3│  │ Disque 4│
│  A  E  I│  │  B  F  P3│ │  C  P2 J│  │  P1 G  K│
└─────────┘  └─────────┘  └─────────┘  └─────────┘
    P = Parité (distribuée sur tous les disques)
```

> [!example] Caractéristiques RAID 5
> 
> - **Capacité utilisable** : (N-1) × taille disque
> - **Redondance** : 1 disque de parité
> - **Performance lecture** : Très bonne (lecture parallèle)
> - **Performance écriture** : Moyenne (pénalité de calcul de parité)
> - **Tolérance aux pannes** : 1 disque

**Cas d'usage** :

- Serveurs de fichiers
- Serveurs d'applications
- Stockage de données d'entreprise
- Bon compromis entre capacité, performance et redondance
- Environnements où les lectures sont plus fréquentes que les écritures

**Exemple de calcul** :

- 4 disques de 1 To chacun
- Capacité totale : 3 To (75%)
- Si 1 disque meurt → reconstruction possible, 0 perte de données

> [!warning] Pénalité d'écriture RAID 5 Chaque écriture nécessite :
> 
> 1. Lecture des anciennes données
> 2. Lecture de l'ancienne parité
> 3. Écriture des nouvelles données
> 4. Calcul et écriture de la nouvelle parité
> 
> = 4 opérations I/O pour 1 écriture !

> [!warning] Problème des disques modernes Avec des disques de grande capacité (4 To+), la reconstruction RAID 5 est :
> 
> - Très longue (plusieurs jours)
> - Très intensive
> - Risque élevé de défaillance d'un 2e disque pendant la reconstruction
> 
> → Préférer RAID 6 pour les gros disques

---

### RAID 6 - Double parité

**Configuration** : Minimum 4 disques

```
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ Disque 1│  │ Disque 2│  │ Disque 3│  │ Disque 4│  │ Disque 5│
│  A  F  │  │  B  P2 │  │  C  Q2 │  │  P1 G  │  │  Q1 H  │
└─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘
    P = Parité 1, Q = Parité 2 (2 calculs différents)
```

> [!example] Caractéristiques RAID 6
> 
> - **Capacité utilisable** : (N-2) × taille disque
> - **Redondance** : 2 disques de parité
> - **Performance lecture** : Très bonne
> - **Performance écriture** : Inférieure à RAID 5 (double calcul de parité)
> - **Tolérance aux pannes** : 2 disques

**Cas d'usage** :

- Stockage de données critiques avec gros volumes
- Serveurs avec disques de grande capacité (4 To+)
- Environnements nécessitant une sécurité accrue
- Systèmes où le temps de reconstruction est long
- Archives à long terme

**Exemple de calcul** :

- 5 disques de 4 To chacun
- Capacité totale : 12 To (60%)
- Si 2 disques meurent simultanément → reconstruction possible, 0 perte

> [!tip] Quand choisir RAID 6 plutôt que RAID 5 ?
> 
> - Disques > 2 To
> - Temps de reconstruction > 24h estimé
> - Données très critiques
> - Faible fréquence de remplacement des disques
> - Budget permettant de "perdre" 2 disques en capacité

---

### RAID 10 - Mirroring + Striping

**Configuration** : Minimum 4 disques (nombre pair requis)

```
      RAID 0 (Stripe)
    ┌─────────┴─────────┐
    │                   │
 RAID 1              RAID 1
┌────┴────┐      ┌────┴────┐
│         │      │         │
Disque 1  Disque 2  Disque 3  Disque 4
  A C E     A C E     B D F     B D F
(miroir)          (miroir)
```

> [!example] Caractéristiques RAID 10
> 
> - **Capacité utilisable** : 50% (N/2 × taille disque)
> - **Redondance** : Complète par paire
> - **Performance lecture** : Excellente (meilleure que RAID 5/6)
> - **Performance écriture** : Très bonne (meilleure que RAID 5/6)
> - **Tolérance aux pannes** : 1 disque par paire (jusqu'à N/2 disques si bien répartis)

**Cas d'usage** :

- Bases de données à haute performance
- Serveurs de messagerie
- Applications transactionnelles
- Environnements nécessitant performances ET redondance
- Machines virtuelles à forte sollicitation I/O

**Exemple de calcul** :

- 4 disques de 1 To chacun
- Capacité totale : 2 To (50%)
- Performances : proche de RAID 0
- Redondance : protection complète

> [!warning] Attention à la topologie RAID 10 peut tolérer plusieurs pannes, MAIS :
> 
> - ✅ Si les disques 1 et 3 tombent → OK (paires différentes)
> - ❌ Si les disques 1 et 2 tombent → PERTE TOTALE (même paire)
> 
> La probabilité dépend de la disposition des pannes.

---

## ⚙️ RAID matériel vs RAID logiciel

### RAID Matériel

Le RAID est géré par une carte contrôleur dédiée.

**Fonctionnement** :

- Carte RAID PCI/PCIe avec processeur dédié
- Cache mémoire embarqué (BBU - Battery Backup Unit)
- BIOS propre pour configuration
- Systèmes voit un seul disque logique

> [!example] Avantages du RAID matériel
> 
> - **Performance** : Processeur dédié, pas de charge CPU système
> - **Cache** : Mémoire cache avec batterie de secours
> - **Indépendance** : Fonctionne quel que soit l'OS
> - **Bootable** : Peut héberger le système d'exploitation
> - **Hot-swap** : Remplacement à chaud facilité
> - **Fonctionnalités avancées** : Gestion fine, monitoring matériel

> [!warning] Inconvénients du RAID matériel
> 
> - **Coût** : Cartes de qualité = plusieurs centaines d'euros
> - **Point de défaillance** : Si la carte meurt, besoin d'une carte identique pour récupérer les données
> - **Vendor lock-in** : Dépendance au fabricant
> - **Mise à jour firmware** : Complexe et risquée
> - **Compatibilité** : Tous les serveurs ne supportent pas toutes les cartes

**Cartes populaires** :

- LSI MegaRAID
- Dell PERC
- HP Smart Array
- Adaptec

### RAID Logiciel

Le RAID est géré par le système d'exploitation (Linux : mdadm, LVM, etc.).

**Fonctionnement** :

- Utilise le CPU système pour les calculs
- Configuration via outils logiciels
- Stockage des métadonnées sur les disques

> [!example] Avantages du RAID logiciel
> 
> - **Coût** : Gratuit, pas de matériel supplémentaire
> - **Flexibilité** : Configuration très fine et modifiable
> - **Portabilité** : Les disques peuvent être lus sur n'importe quel système Linux
> - **Pas de vendor lock-in** : Indépendant du matériel
> - **Évolutivité** : Ajout/suppression de disques plus simple
> - **Intégration** : Bien intégré aux outils système (LVM, etc.)

> [!warning] Inconvénients du RAID logiciel
> 
> - **Performance** : Utilise le CPU système (impact variable selon charge)
> - **Boot** : Configuration plus complexe pour booter sur RAID
> - **Dépendance OS** : Nécessite que l'OS soit fonctionnel
> - **Pas de cache matériel** : Sauf si ajout d'une carte HBA avec cache
> - **Gestion hot-swap** : Moins automatisée qu'avec matériel dédié

**Outils Linux** :

```bash
# mdadm - outil principal pour RAID logiciel Linux
mdadm --create /dev/md0 --level=5 --raid-devices=4 /dev/sd[abcd]

# LVM peut aussi faire du RAID
lvcreate --type raid5 -L 10G -n raid5_lv vg_name
```

### Comparaison synthétique

|Critère|RAID Matériel|RAID Logiciel|
|---|---|---|
|**Coût initial**|Élevé (200-2000€)|Gratuit|
|**Performance**|Excellente|Bonne (dépend CPU)|
|**Charge CPU**|Nulle|Variable|
|**Complexité setup**|Moyenne|Moyenne à élevée|
|**Portabilité**|Faible|Élevée|
|**Fiabilité**|Très bonne|Bonne|
|**Flexibilité**|Moyenne|Élevée|
|**Boot system**|Simple|Plus complexe|

> [!tip] Quel RAID choisir ?
> 
> **Choisir RAID matériel si :**
> 
> - Budget disponible
> - Performance critique (bases de données haute charge)
> - Environnement de production critique
> - Infrastructure existante avec cartes RAID
> 
> **Choisir RAID logiciel si :**
> 
> - Budget limité
> - Flexibilité et portabilité importantes
> - Charge I/O modérée
> - Besoin de configurations personnalisées
> - Infrastructure cloud/virtualisée

---

## 📈 Redondance et performance

### Impact sur les performances

Le choix du niveau RAID a un impact direct sur les performances :

```
Performance LECTURE (du meilleur au moins bon) :
RAID 0 > RAID 10 > RAID 5 ≈ RAID 6 > RAID 1

Performance ÉCRITURE (du meilleur au moins bon) :
RAID 0 > RAID 10 > RAID 1 > RAID 5 > RAID 6
```

**Facteurs influençant les performances** :

- Type de charge (lecture vs écriture)
- Taille des blocs
- Nombre de disques
- Type de contrôleur (matériel vs logiciel)
- Type de disques (HDD vs SSD)

### Pénalité d'écriture par niveau RAID

|Niveau RAID|Opérations I/O par écriture|Explication|
|---|---|---|
|**RAID 0**|1|Écriture directe|
|**RAID 1**|2|Écriture sur 2 disques (miroir)|
|**RAID 10**|2|Écriture sur 2 disques par paire|
|**RAID 5**|4|Read-Modify-Write (lecture ancienne data + parité, écriture nouvelle data + parité)|
|**RAID 6**|6|Comme RAID 5 mais avec double parité|

> [!info] Comprendre la pénalité RAID 5 Pour écrire 1 bloc de données en RAID 5 :
> 
> 1. Lire l'ancien bloc de données
> 2. Lire l'ancien bloc de parité
> 3. Calculer la nouvelle parité : P_new = P_old ⊕ Data_old ⊕ Data_new
> 4. Écrire le nouveau bloc de données
> 5. Écrire le nouveau bloc de parité
> 
> = 2 lectures + 2 écritures = 4 opérations I/O

### Protection des données

|Niveau RAID|Disques tolérés en panne|Capacité utilisable|Redondance|
|---|---|---|---|
|**RAID 0**|0|100%|Aucune|
|**RAID 1**|N-1|50%|Totale|
|**RAID 5**|1|(N-1)/N|Bonne|
|**RAID 6**|2|(N-2)/N|Excellente|
|**RAID 10**|1 par paire|50%|Totale|

### Temps de reconstruction

Le temps nécessaire pour reconstruire un array après une panne :

> [!warning] Facteurs impactant la reconstruction
> 
> - **Taille des disques** : 10 To = beaucoup plus long que 1 To
> - **Charge système** : Reconstruction ralentie si système en production
> - **Type de RAID** : RAID 6 plus lent que RAID 5
> - **Type de disques** : SSD plus rapide que HDD
> - **Vitesse du contrôleur**

**Estimations approximatives** (disques HDD 7200 RPM) :

- RAID 1 (1 To) : 2-4 heures
- RAID 5 (4×1 To) : 4-8 heures
- RAID 5 (4×4 To) : 20-40 heures
- RAID 6 (6×4 To) : 30-60 heures

> [!warning] Risque pendant la reconstruction Pendant la reconstruction :
> 
> - Les performances sont dégradées
> - Les disques restants sont fortement sollicités
> - Risque accru de défaillance d'un 2e disque
> - RAID 5 : si 2e disque tombe = PERTE TOTALE
> - RAID 6 : si 3e disque tombe = PERTE TOTALE

---

## ✅ Bonnes pratiques

### Choix du niveau RAID

> [!tip] Guide de décision rapide
> 
> **RAID 0** :
> 
> - Vous avez besoin de performances maximales
> - Les données sont temporaires ou sauvegardées ailleurs
> - Budget disque limité
> 
> **RAID 1** :
> 
> - Redondance critique
> - Disque système/boot
> - Petit nombre de disques (2-4)
> - Besoin de simplicité
> 
> **RAID 5** :
> 
> - Bon compromis capacité/redondance
> - Disques ≤ 2 To
> - Lectures >> Écritures
> - Budget moyen
> 
> **RAID 6** :
> 
> - Disques > 2 To
> - Données très critiques
> - Tolérance de 2 pannes nécessaire
> - Temps de reconstruction long anticipé
> 
> **RAID 10** :
> 
> - Performance ET redondance critiques
> - Base de données haute performance
> - Budget disque confortable
> - Charge d'écritures importante

### Configuration et maintenance

> [!tip] Conseils de configuration
> 
> **Disques** :
> 
> - Utiliser des disques de même taille (même modèle si possible)
> - Privilégier des disques "enterprise" ou "NAS" pour RAID
> - Éviter de mélanger SSD et HDD dans le même array
> - Prévoir un disque de spare (hot spare)
> 
> **Contrôleur** :
> 
> - Activer le cache en écriture si BBU présente
> - Configurer le stripe size selon la charge (64-256 KB typique)
> - Vérifier régulièrement le firmware
> 
> **Système** :
> 
> - Activer le monitoring (smartd, mdadm monitoring)
> - Configurer les alertes email
> - Tester régulièrement les sauvegardes
> - Documenter la configuration

### Monitoring et alertes

```bash
# Surveillance RAID logiciel avec mdadm
cat /proc/mdstat

# Configuration des alertes email
echo "MAILADDR votre@email.com" >> /etc/mdadm/mdadm.conf

# Vérification de l'état SMART des disques
smartctl -a /dev/sda

# Test de lecture de l'array (scrubbing)
echo check > /sys/block/md0/md/sync_action
```

> [!warning] Erreurs courantes à éviter
> 
> ❌ **Ne JAMAIS** :
> 
> - Considérer RAID comme une sauvegarde (RAID ≠ Backup)
> - Ignorer les alertes de disque défaillant
> - Retirer plusieurs disques simultanément
> - Négliger les tests de restauration
> - Mélanger des disques de tailles très différentes
> - Faire confiance à RAID 5 avec disques > 4 To
> 
> ✅ **TOUJOURS** :
> 
> - Avoir une vraie stratégie de sauvegarde (3-2-1)
> - Remplacer immédiatement les disques défaillants
> - Tester la reconstruction avant la mise en production
> - Documenter la configuration RAID
> - Monitorer l'état de l'array

### Pièges courants

> [!warning] Le mythe du RAID comme sauvegarde
> 
> **RAID protège contre** :
> 
> - Défaillance matérielle des disques
> - Temps d'arrêt lié à une panne disque
> 
> **RAID NE protège PAS contre** :
> 
> - Suppression accidentelle de fichiers
> - Corruption de données
> - Virus/ransomware
> - Vol/incendie/désastre
> - Erreur humaine (rm -rf /)
> - Défaillance du contrôleur RAID
> 
> → **Règle 3-2-1 des sauvegardes** :
> 
> - **3** copies de vos données
> - Sur **2** supports différents
> - Dont **1** copie hors site

> [!warning] URE (Unrecoverable Read Error)
> 
> Les disques modernes ont un taux URE typique de 10^-14 à 10^-15.
> 
> **Signification** : 1 erreur tous les 12,5 To lus (pour 10^-14).
> 
> **Problème** : Lors de la reconstruction d'un RAID 5 avec 4 disques de 4 To :
> 
> - Volume total à lire : 12 To
> - Probabilité d'URE : ~96% !
> - Si URE survient → reconstruction échoue
> 
> **Solution** : Utiliser RAID 6 pour les gros disques.

### Dimensionnement

**Calculateur de capacité** :

```
RAID 0 : Capacité = N × Taille_disque
RAID 1 : Capacité = Taille_disque (plus petit)
RAID 5 : Capacité = (N-1) × Taille_disque
RAID 6 : Capacité = (N-2) × Taille_disque
RAID 10: Capacité = (N/2) × Taille_disque
```

**Exemple** : 6 disques de 2 To

| RAID    | Calcul       | Capacité utilisable |
| ------- | ------------ | ------------------- |
| RAID 0  | 6 × 2 To     | 12 To               |
| RAID 1  | 2 To         | 2 To                |
| RAID 5  | (6-1) × 2 To | 10 To               |
| RAID 6  | (6-2) × 2 To | 8 To                |
| RAID 10 | (6/2) × 2 To | 6 To                |
|         |              |                     |

---

## 🎯 Tableau récapitulatif

| Niveau      | Disques min | Capacité | Pannes tolérées | Perf. lecture | Perf. écriture | Cas d'usage                      |
| ----------- | ----------- | -------- | --------------- | ------------- | -------------- | -------------------------------- |
| **RAID 0**  | 2           | 100%     | 0               | ⭐⭐⭐⭐⭐         | ⭐⭐⭐⭐⭐          | Performance pure                 |
| **RAID 1**  | 2           | 50%      | N-1             | ⭐⭐⭐⭐          | ⭐⭐⭐            | OS, données critiques            |
| **RAID 5**  | 3           | (N-1)/N  | 1               | ⭐⭐⭐⭐          | ⭐⭐             | Stockage général                 |
| **RAID 6**  | 4           | (N-2)/N  | 2               | ⭐⭐⭐⭐          | ⭐⭐             | Gros volumes critiques           |
| **RAID 10** | 4           | 50%      | 1/paire         | ⭐⭐⭐⭐⭐         | ⭐⭐⭐⭐           | Hautes performances + redondance |
|             |             |          |                 |               |                |                                  |

---

> [!tip] Pensée finale Le RAID améliore la disponibilité et les performances, mais n'est PAS une sauvegarde. Une stratégie de stockage robuste combine :
> 
> - RAID pour la disponibilité et la performance
> - Sauvegardes régulières pour la protection des données
> - Tests de restauration pour la tranquillité d'esprit