# 

## 📋 Table des matières

- [Introduction](#introduction)
- [JBOD - Just a Bunch Of Disks](#jbod---just-a-bunch-of-disks)
  - [Définition](#définition)
  - [Principe de fonctionnement](#principe-de-fonctionnement-jbod)
  - [Capacité totale](#capacité-totale-jbod)
  - [Performance](#performance-jbod)
  - [Tolérance aux pannes](#tolérance-aux-pannes-jbod)
  - [Cas d'usage](#cas-dusage-jbod)
- [RAID 0 - Striping](#raid-0---striping)
  - [Définition](#définition-raid-0)
  - [Principe de fonctionnement](#principe-de-fonctionnement-raid-0)
  - [Capacité totale](#capacité-totale-raid-0)
  - [Performance](#performance-raid-0)
  - [Tolérance aux pannes](#tolérance-aux-pannes-raid-0)
  - [Cas d'usage](#cas-dusage-raid-0)
- [Comparaison JBOD vs RAID 0](#comparaison-jbod-vs-raid-0)

---

## 🎯 Introduction

JBOD et RAID 0 sont les deux configurations les plus simples pour agréger plusieurs disques physiques. Bien qu'ils permettent tous deux d'augmenter la capacité de stockage disponible, ils fonctionnent de manière très différente et n'offrent **aucune tolérance aux pannes**.

> [!warning] Avertissement crucial
> Ni JBOD ni RAID 0 ne fournissent de redondance. La perte d'un seul disque entraîne la perte de toutes les données de la grappe. Ces configurations ne doivent être utilisées que pour des données non critiques ou temporaires.

---

## 📦 JBOD - Just a Bunch Of Disks

### Définition

**JBOD** (Just a Bunch Of Disks) ou **NRAID** (Non-Redundant Array of Inexpensive Disks) est une méthode qui consiste à regrouper plusieurs disques physiques en un seul volume logique par **concaténation linéaire**.

> [!info] Terminologie
> Le terme "JBOD" n'est pas officiellement un niveau RAID, mais il est souvent mentionné dans le contexte du RAID car il utilise une infrastructure similaire pour agréger des disques.

### Principe de fonctionnement (JBOD)

Le JBOD fonctionne en **concaténant** les disques les uns après les autres. Les données sont écrites séquentiellement :
- Le premier disque est rempli complètement
- Puis le deuxième disque commence à être utilisé
- Et ainsi de suite

```
┌─────────────┐
│   Disque 1  │ ← Données A, B, C (jusqu'à ce qu'il soit plein)
│   (1 To)    │
└─────────────┘
       ↓
┌─────────────┐
│   Disque 2  │ ← Données D, E, F (commence quand Disque 1 plein)
│   (1 To)    │
└─────────────┘
       ↓
┌─────────────┐
│   Disque 3  │ ← Données G, H, I (commence quand Disque 2 plein)
│   (500 Go)  │
└─────────────┘

Volume logique résultant : 2,5 To
```

> [!example] Exemple concret
> Vous avez 3 disques de tailles différentes : 1 To, 1 To et 500 Go. En JBOD, le système voit un seul volume de 2,5 To. Lorsque vous copiez un gros fichier de 1,2 To, les premiers 1 To vont sur le disque 1, et les 200 Go restants sur le disque 2.

### Capacité totale (JBOD)

La capacité totale d'un JBOD est la **somme de tous les disques**, quelle que soit leur taille :

**Formule :**
```
Capacité totale = Disque 1 + Disque 2 + ... + Disque n
```

> [!example] Exemples de calcul
> - 3 disques de 1 To : **3 To** disponibles
> - 2 disques de 2 To + 1 disque de 500 Go : **4,5 To** disponibles
> - 4 disques de tailles variées (1 To, 2 To, 500 Go, 3 To) : **6,5 To** disponibles

**Avantage :** Aucune perte de capacité, tous les disques sont utilisés à 100%.

### Performance (JBOD)

Les performances d'un JBOD sont **identiques à celles d'un disque seul** car il n'y a pas de parallélisation des opérations.

**Caractéristiques de performance :**

| Opération | Performance | Explication |
|-----------|-------------|-------------|
| **Lecture séquentielle** | = 1 disque | Un seul disque actif à la fois |
| **Écriture séquentielle** | = 1 disque | Un seul disque actif à la fois |
| **Lecture aléatoire** | = 1 disque | Pas d'amélioration |
| **Écriture aléatoire** | = 1 disque | Pas d'amélioration |

> [!info] Pourquoi pas d'amélioration ?
> Contrairement au RAID 0, le JBOD n'utilise qu'un seul disque à la fois. Il n'y a donc aucun parallélisme et aucun gain de performance par rapport à un disque unique.

### Tolérance aux pannes (JBOD)

> [!warning] Tolérance aux pannes : AUCUNE
> **Pannes tolérées : 0**
> 
> Si **n'importe quel disque** de la grappe tombe en panne :
> - Les données présentes sur ce disque sont **perdues**
> - Selon la configuration, le volume entier peut devenir **inaccessible**
> - Même si les autres disques fonctionnent, la reconstruction n'est pas possible

**Impact d'une panne :**

```
Avant panne :
Disque 1 : Données A, B, C ✅
Disque 2 : Données D, E, F ✅
Disque 3 : Données G, H, I ✅

Après panne Disque 2 :
Disque 1 : Données A, B, C ✅ (toujours accessibles)
Disque 2 : ❌ PANNE - Données D, E, F PERDUES
Disque 3 : Données G, H, I ✅ (toujours accessibles)
```

> [!tip] Note importante
> Avec certaines implémentations JBOD, seules les données du disque défaillant sont perdues. Avec d'autres, l'ensemble du volume devient inutilisable. Cela dépend du système d'exploitation et du contrôleur utilisé.

### Cas d'usage (JBOD)

Le JBOD est adapté dans des situations très spécifiques :

✅ **Quand utiliser JBOD :**
- **Archivage de données peu importantes** (collections de médias non critiques)
- **Agrégation de disques de tailles différentes** (réutilisation de vieux disques)
- **Stockage temporaire** (fichiers de travail, cache)
- **Budget limité** avec besoin d'espace maximum

❌ **Quand NE PAS utiliser JBOD :**
- Données critiques ou importantes
- Environnement de production
- Besoin de performance
- Besoin de fiabilité

> [!tip] Astuce
> Le JBOD est souvent utilisé pour donner une "seconde vie" à d'anciens disques de capacités différentes, en les regroupant pour créer un grand volume de stockage bon marché pour des données non critiques.

---

## ⚡ RAID 0 - Striping

### Définition (RAID 0)

**RAID 0**, également appelé **Striping** (entrelaçage par bandes), est une technique qui répartit les données de manière **alternée** sur plusieurs disques pour améliorer les performances.

> [!info] RAID 0 = "RAID" ?
> Le terme "RAID 0" est paradoxal car l'acronyme RAID signifie "Redundant Array" (matrice redondante), alors que le RAID 0 n'offre **aucune redondance**. Il est néanmoins considéré comme un niveau RAID pour des raisons historiques.

### Principe de fonctionnement (RAID 0)

Le RAID 0 divise les données en **blocs** (appelés "stripes" ou "bandes") et les répartit de manière **alternée** sur tous les disques de la grappe.

```
Données à écrire : A B C D E F G H

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Disque 1   │  │  Disque 2   │  │  Disque 3   │
├─────────────┤  ├─────────────┤  ├─────────────┤
│  Bloc A     │  │  Bloc B     │  │  Bloc C     │
│  Bloc D     │  │  Bloc E     │  │  Bloc F     │
│  Bloc G     │  │  Bloc H     │  │     ...     │
└─────────────┘  └─────────────┘  └─────────────┘
```

**Processus d'écriture :**
1. Les données sont divisées en blocs de taille fixe (généralement 64 Ko, 128 Ko ou 256 Ko)
2. Les blocs sont écrits en **parallèle** sur les différents disques
3. Bloc 1 → Disque 1, Bloc 2 → Disque 2, Bloc 3 → Disque 3, etc.
4. Le cycle recommence : Bloc 4 → Disque 1, Bloc 5 → Disque 2...

> [!example] Exemple concret
> Vous copiez un fichier vidéo de 1 Go sur un RAID 0 de 2 disques avec des blocs de 128 Ko :
> - Bloc 1 (128 Ko) → Disque 1
> - Bloc 2 (128 Ko) → Disque 2
> - Bloc 3 (128 Ko) → Disque 1
> - Bloc 4 (128 Ko) → Disque 2
> - ...et ainsi de suite jusqu'à la fin du fichier
> 
> Résultat : Les deux disques travaillent en **parallèle**, divisant par 2 le temps d'écriture.

### Capacité totale (RAID 0)

La capacité totale d'un RAID 0 dépend du nombre de disques et de leur taille :

**Formule :**
```
Capacité totale = n × taille du plus petit disque
```
où **n** = nombre de disques dans la grappe

> [!warning] Important
> Si les disques ont des tailles différentes, la capacité supplémentaire du disque le plus grand est **perdue** ou reste **inutilisée**.

> [!example] Exemples de calcul
> **Cas 1 : Disques identiques**
> - 3 disques de 1 To
> - Capacité = 3 × 1 To = **3 To**
> 
> **Cas 2 : Disques différents**
> - 2 disques : 1 To et 2 To
> - Capacité = 2 × 1 To (plus petit) = **2 To**
> - Perte : 1 To du disque de 2 To reste inutilisé
> 
> **Cas 3 : Configuration optimale**
> - 4 disques de 500 Go
> - Capacité = 4 × 500 Go = **2 To**

> [!tip] Recommandation
> Utilisez toujours des disques de **taille identique** en RAID 0 pour éviter le gaspillage de capacité.

### Performance (RAID 0)

Le RAID 0 offre les **meilleures performances** de tous les niveaux RAID car toutes les opérations sont parallélisées sans surcharge liée à la redondance.

**Gains théoriques :**

| Opération | Performance théorique | Explication |
|-----------|----------------------|-------------|
| **Lecture séquentielle** | n × vitesse d'1 disque | Tous les disques lisent en parallèle |
| **Écriture séquentielle** | n × vitesse d'1 disque | Tous les disques écrivent en parallèle |
| **Lecture aléatoire** | ≈ n × vitesse d'1 disque | Parallélisation des requêtes |
| **Écriture aléatoire** | ≈ n × vitesse d'1 disque | Parallélisation des requêtes |

où **n** = nombre de disques

> [!example] Exemple de performance
> **Configuration :** 3 disques SATA à 150 Mo/s chacun en RAID 0
> 
> **Performance théorique :**
> - Lecture séquentielle : 3 × 150 = **450 Mo/s**
> - Écriture séquentielle : 3 × 150 = **450 Mo/s**
> 
> **Performance réelle :** Environ 400-420 Mo/s (overhead du contrôleur RAID)

**Facteurs affectant les performances :**

```bash
# Taille des blocs (stripe size)
Petits blocs (64 Ko)  : Meilleur pour petits fichiers et accès aléatoire
Gros blocs (256 Ko+)  : Meilleur pour gros fichiers et streaming vidéo

# Nombre de disques
2 disques  : Performance × 2
4 disques  : Performance × 4
8 disques  : Performance × 8 (mais risque accru de panne)
```

> [!tip] Optimisation
> Pour du **streaming vidéo** ou du **montage**, privilégiez des blocs de 128 Ko ou 256 Ko. Pour des **bases de données** avec beaucoup de petites transactions, privilégiez 64 Ko.

### Tolérance aux pannes (RAID 0)

> [!warning] Tolérance aux pannes : AUCUNE (pire que JBOD)
> **Pannes tolérées : 0**
> 
> La défaillance d'un **seul disque** entraîne la **perte totale** de toutes les données de la grappe.

**Pourquoi RAID 0 est plus risqué que JBOD :**

En JBOD, seules les données du disque défaillant sont potentiellement perdues. En RAID 0, comme les données sont entrelacées sur tous les disques, **chaque fichier est fragmenté** entre tous les disques. La perte d'un seul disque rend **tous les fichiers** irrécupérables.

```
Fichier vidéo.mp4 (1 Go) en RAID 0 avec 3 disques :

Disque 1 : Blocs 1, 4, 7, 10, 13... ✅
Disque 2 : Blocs 2, 5, 8, 11, 14... ❌ PANNE
Disque 3 : Blocs 3, 6, 9, 12, 15... ✅

Résultat : Le fichier est PERDU car il manque les blocs 2, 5, 8, 11, 14...
```

**Probabilité de panne :**

Plus vous ajoutez de disques en RAID 0, plus le risque de panne augmente :

| Configuration | Risque relatif |
|---------------|----------------|
| 1 disque seul | 1× (référence) |
| RAID 0 - 2 disques | ≈ 2× |
| RAID 0 - 4 disques | ≈ 4× |
| RAID 0 - 8 disques | ≈ 8× |

> [!warning] Danger exponentiel
> Chaque disque ajouté à la grappe RAID 0 augmente proportionnellement le risque qu'un disque tombe en panne, entraînant la perte de **toutes** les données.

### Cas d'usage (RAID 0)

Le RAID 0 est adapté uniquement dans des situations très spécifiques où la **performance est critique** et les données sont **non critiques ou facilement récupérables**.

✅ **Quand utiliser RAID 0 :**
- **Stations de montage vidéo** (fichiers de travail temporaires)
- **Rendu 3D** (fichiers temporaires de calcul)
- **Cache système** (données volatiles)
- **Jeux vidéo** (réinstallables facilement)
- **Environnements de développement** (code sauvegardé dans Git)
- **Scratch disks** pour applications (Photoshop, After Effects)

❌ **Quand NE JAMAIS utiliser RAID 0 :**
- Serveurs de production
- Bases de données
- Stockage de fichiers utilisateurs
- Documents importants
- Tout système où la perte de données est inacceptable

> [!tip] Bonne pratique
> Si vous utilisez le RAID 0, mettez en place une **sauvegarde automatique** très fréquente (idéalement en temps réel) vers un autre support. Le RAID 0 doit être considéré comme du stockage **jetable**.

> [!example] Cas d'usage professionnel
> **Studio de production vidéo :**
> - RAID 0 en SSD NVMe pour le montage en temps réel (performances maximales)
> - Sauvegarde automatique toutes les 15 minutes vers un NAS en RAID 5
> - Après finalisation, export vers stockage long terme (RAID 6 ou cloud)
> 
> En cas de panne du RAID 0, seules 15 minutes de travail maximum sont perdues.

---

## 📊 Comparaison JBOD vs RAID 0

| Critère | JBOD | RAID 0 |
|---------|------|--------|
| **Méthode** | Concaténation séquentielle | Entrelaçage parallèle (striping) |
| **Nombre de disques minimum** | 2+ | 2+ |
| **Capacité totale** | Somme de tous les disques | n × plus petit disque |
| **Performance lecture** | = 1 disque | ≈ n × 1 disque |
| **Performance écriture** | = 1 disque | ≈ n × 1 disque |
| **Tolérance aux pannes** | ❌ Aucune | ❌ Aucune (pire) |
| **Perte de données si 1 panne** | Partielle (1 disque) | Totale (tous les fichiers) |
| **Disques de tailles différentes** | ✅ Optimal | ⚠️ Gaspillage |
| **Complexité** | Très simple | Simple |
| **Coût** | Minimal | Minimal |
| **Usage typique** | Agrégation économique | Performance maximale |

---

> [!warning] Rappel de sécurité
> **JBOD et RAID 0 = AUCUNE PROTECTION**
> 
> Ces deux configurations doivent être utilisées uniquement pour :
> - Données **temporaires** ou **non critiques**
> - Avec une **sauvegarde externe** obligatoire si les données ont une quelconque valeur
> 
> Pour des données importantes, privilégiez les niveaux RAID avec redondance (RAID 1, 5, 6, 10).

---

> [!info] Points clés à retenir
> - **JBOD** : Concatène les disques séquentiellement, pas de gain de performance, perte limitée à un disque
> - **RAID 0** : Entrelaçage parallèle, excellentes performances, perte totale si une panne
> - **Les deux** : Aucune tolérance aux pannes, réservés aux données non critiques
> - **Capacité JBOD** : Somme de tous les disques
> - **Capacité RAID 0** : n × plus petit disque
> - **Performance RAID 0** : Multipliée par le nombre de disques
> - **Règle d'or** : Sauvegarde obligatoire, jamais en production pour données importantes
