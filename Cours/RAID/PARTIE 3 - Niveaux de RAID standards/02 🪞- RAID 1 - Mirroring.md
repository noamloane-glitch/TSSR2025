## 📋 Table des matières

- [Introduction](#introduction)
- [Définition](#définition)
- [Principe de fonctionnement](#principe-de-fonctionnement)
- [Capacité totale](#capacité-totale)
- [Performance](#performance)
  - [Performance en lecture](#performance-en-lecture)
  - [Performance en écriture](#performance-en-écriture)
- [Tolérance aux pannes](#tolérance-aux-pannes)
  - [Reconstruction](#reconstruction)
  - [Scénarios de panne](#scénarios-de-panne)
- [Nombre de disques](#nombre-de-disques)
- [Cas d'usage](#cas-dusage)
- [Avantages et inconvénients](#avantages-et-inconvénients)

---

## 🎯 Introduction

Le **RAID 1** est le niveau RAID le plus simple offrant une **redondance complète**. C'est la solution de référence lorsque la **fiabilité** et la **disponibilité** des données sont prioritaires sur la capacité de stockage.

> [!info] En un mot
> RAID 1 = **Duplication totale** des données sur tous les disques de la grappe. Chaque disque contient une copie identique et complète de toutes les données.

---

## 📖 Définition

**RAID 1**, également appelé **Mirroring** (mise en miroir), est une technique qui consiste à dupliquer **intégralement** toutes les données sur plusieurs disques physiques.

Chaque disque de la grappe contient exactement les **mêmes données** :
- Si un disque tombe en panne, les autres contiennent une copie complète
- Les données restent accessibles tant qu'au moins un disque fonctionne
- Aucune reconstruction n'est nécessaire pour accéder aux données

> [!tip] Analogie
> Imaginez que vous écrivez un document important. Le RAID 1, c'est comme faire une photocopie de chaque page et la ranger dans un classeur différent. Si un classeur est détruit, l'autre contient toujours le document complet.

---

## ⚙️ Principe de fonctionnement

### Écriture des données

Lorsqu'une donnée est écrite sur un RAID 1, elle est **copiée simultanément** sur tous les disques de la grappe.

```
Données à écrire : Fichier A (100 Mo)

┌─────────────┐      ┌─────────────┐
│  Disque 1   │      │  Disque 2   │
├─────────────┤      ├─────────────┤
│ Fichier A   │  ←→  │ Fichier A   │
│  (100 Mo)   │      │  (100 Mo)   │
│             │      │             │
│   Copie 1   │      │   Copie 2   │
└─────────────┘      └─────────────┘
      ↓                     ↓
  IDENTIQUES           IDENTIQUES
```

**Processus détaillé :**

1. Le contrôleur RAID reçoit la demande d'écriture
2. Il envoie **simultanément** les données vers tous les disques
3. L'opération est considérée comme terminée quand **tous** les disques ont confirmé l'écriture
4. En cas d'échec sur un disque, l'opération continue sur les autres

> [!example] Exemple concret
> Vous sauvegardez un fichier de 500 Mo sur un RAID 1 avec 2 disques :
> - Le fichier est écrit en **parallèle** sur les deux disques
> - Chaque disque reçoit les 500 Mo complets
> - Si l'écriture échoue sur le Disque 1, elle continue sur le Disque 2
> - Le système vous avertit qu'un disque est défaillant, mais vos données sont intactes

### Lecture des données

Lors de la lecture, le contrôleur RAID peut choisir de lire depuis **n'importe quel disque** de la grappe, ou même répartir les lectures pour optimiser les performances.

```
Lecture demandée : Fichier A

Option 1 : Lire depuis Disque 1
┌─────────────┐
│  Disque 1   │  ─→  Lecture
└─────────────┘

Option 2 : Lire depuis Disque 2
┌─────────────┐
│  Disque 2   │  ─→  Lecture
└─────────────┘

Option 3 : Répartir la lecture (optimisation)
┌─────────────┐
│  Disque 1   │  ─→  Blocs 1, 3, 5...
└─────────────┘
┌─────────────┐
│  Disque 2   │  ─→  Blocs 2, 4, 6...
└─────────────┘
```

---

## 💾 Capacité totale

La capacité totale d'un RAID 1 est égale à la **taille du plus petit disque** de la grappe.

**Formule :**
```
Capacité totale = Taille du plus petit disque
```

> [!warning] Perte de capacité importante
> Le RAID 1 offre la **pire utilisation de la capacité** de tous les niveaux RAID. Avec n disques, vous n'utilisez que 1/n de la capacité totale.

### Exemples de calcul

> [!example] Cas 1 : 2 disques identiques
> **Configuration :** 2 disques de 1 To
> - Capacité brute : 2 × 1 To = 2 To
> - Capacité utilisable : **1 To**
> - Perte : 1 To (50%)

> [!example] Cas 2 : 3 disques identiques
> **Configuration :** 3 disques de 2 To
> - Capacité brute : 3 × 2 To = 6 To
> - Capacité utilisable : **2 To**
> - Perte : 4 To (67%)

> [!example] Cas 3 : Disques de tailles différentes (NON RECOMMANDÉ)
> **Configuration :** 1 disque de 1 To + 1 disque de 2 To
> - Capacité brute : 3 To
> - Capacité utilisable : **1 To** (plus petit disque)
> - Perte : 2 To (67%)
> - Le disque de 2 To n'utilise que 1 To, gaspillant 1 To

### Tableau récapitulatif

| Configuration | Capacité brute | Capacité utilisable | Efficacité |
|---------------|----------------|---------------------|------------|
| 2 × 1 To | 2 To | 1 To | 50% |
| 2 × 2 To | 4 To | 2 To | 50% |
| 3 × 1 To | 3 To | 1 To | 33% |
| 4 × 500 Go | 2 To | 500 Go | 25% |

> [!tip] Recommandation
> Utilisez **toujours des disques de taille identique** en RAID 1. Tout espace supplémentaire sur un disque plus grand sera perdu.

---

## 🚀 Performance

### Performance en lecture

Le RAID 1 peut offrir une **amélioration des performances en lecture** grâce à plusieurs stratégies :

**Stratégie 1 : Répartition des lectures**
Le contrôleur peut répartir les requêtes de lecture entre les disques disponibles :

```
Requête 1 → Disque 1
Requête 2 → Disque 2
Requête 3 → Disque 1
Requête 4 → Disque 2
```

**Gain théorique en lecture :**

| Configuration | Performance lecture séquentielle | Performance lecture aléatoire |
|---------------|----------------------------------|------------------------------|
| 2 disques | ≈ 1,5× à 2× | ≈ 1,5× à 2× |
| 3 disques | ≈ 2× à 3× | ≈ 2× à 3× |

> [!info] Gain réel
> Le gain réel dépend de :
> - L'implémentation du contrôleur RAID
> - Le type de charges (séquentiel vs aléatoire)
> - Le nombre de requêtes simultanées
> 
> En pratique, avec 2 disques, attendez-vous à un gain de **50% à 80%** en lecture.

> [!example] Exemple de performance lecture
> **Configuration :** 2 disques SATA à 150 Mo/s en RAID 1
> 
> **Performance théorique :**
> - Lecture séquentielle : 150-250 Mo/s (gain 0-70%)
> - Lecture avec multiples processus : jusqu'à 300 Mo/s
> 
> **Performance réelle mesurée :**
> - Lecture séquentielle : ~180 Mo/s
> - Lecture aléatoire : ~200 Mo/s (excellente amélioration)

### Performance en écriture

Le RAID 1 présente une **légère pénalité en écriture** car les données doivent être écrites sur tous les disques.

**Caractéristiques :**

| Opération | Performance | Explication |
|-----------|-------------|-------------|
| **Écriture séquentielle** | ≈ 1× (un disque) | Limité par le disque le plus lent |
| **Écriture aléatoire** | ≈ 0,9× (légère baisse) | Overhead de synchronisation |

**Pourquoi une pénalité ?**

1. Les données doivent être écrites sur **tous** les disques
2. L'opération se termine quand le **disque le plus lent** a fini
3. Il y a un léger overhead de synchronisation du contrôleur

> [!example] Exemple de performance écriture
> **Configuration :** 2 disques à 150 Mo/s en RAID 1
> 
> **Performance :**
> - Écriture : ~140 Mo/s (légère baisse de 7%)
> 
> La pénalité est minime et généralement **négligeable** pour la plupart des usages.

> [!tip] Optimisation
> Pour améliorer les performances en écriture, certains contrôleurs RAID matériels utilisent un **cache avec batterie** (BBU - Battery Backup Unit) qui peut accuser réception de l'écriture avant que tous les disques n'aient terminé.

### Comparaison globale

```
Performance relative (1 disque seul = 100%)

Lecture séquentielle :  ████████████████░░░░  160%
Lecture aléatoire :     █████████████████░░░  170%
Écriture séquentielle : ████████████████░░░░  95%
Écriture aléatoire :    ███████████████░░░░░  90%
```

---

## 🛡️ Tolérance aux pannes

Le RAID 1 offre **l'excellente tolérance aux pannes**, c'est son principal atout.

> [!info] Tolérance aux pannes
> **Pannes tolérées : n - 1**
> 
> où **n** = nombre total de disques dans la grappe
> 
> Tant qu'au moins **un disque fonctionne**, toutes les données restent accessibles.

### Niveaux de protection

| Configuration | Pannes tolérées | Taux de survie |
|---------------|-----------------|----------------|
| 2 disques | 1 disque | 50% des disques peuvent tomber |
| 3 disques | 2 disques | 66% des disques peuvent tomber |
| 4 disques | 3 disques | 75% des disques peuvent tomber |

> [!warning] Attention
> "n - 1 pannes tolérées" signifie que vous pouvez perdre n'importe quels (n-1) disques, **mais pas tous en même temps**.

### Reconstruction

Contrairement au RAID 0, la reconstruction d'un RAID 1 après une panne est **simple et rapide** :

**Processus de reconstruction :**

```
État initial (sain) :
Disque 1 : Données complètes ✅
Disque 2 : Données complètes ✅

Panne détectée :
Disque 1 : Données complètes ✅
Disque 2 : ❌ PANNE

Remplacement du disque :
Disque 1 : Données complètes ✅ (source)
Disque 3 : Vide (nouveau disque)

Reconstruction :
Disque 1 : Données complètes ✅ ─→ Copie ─→ Disque 3
                                              ↓
État final :                              Données complètes ✅
Disque 1 : Données complètes ✅
Disque 3 : Données complètes ✅
```

**Caractéristiques de la reconstruction :**

1. **Simple** : Copie byte-par-byte depuis un disque sain
2. **Rapide** : Vitesse = vitesse de lecture du disque source
3. **Sans calcul** : Pas de calcul de parité complexe
4. **Données disponibles** : Le système reste utilisable pendant la reconstruction

> [!example] Temps de reconstruction
> **Disque de 1 To à 100 Mo/s :**
> - Temps théorique : 1 000 000 Mo ÷ 100 Mo/s = 10 000 secondes ≈ **2h45**
> - Temps réel : **3-4 heures** (avec utilisation système en parallèle)
> 
> **Disque de 4 To à 150 Mo/s :**
> - Temps théorique : ≈ **7h30**
> - Temps réel : **8-10 heures**

> [!tip] Recommandation pendant reconstruction
> Pendant la reconstruction :
> - ✅ Le système reste **pleinement fonctionnel**
> - ⚠️ Évitez les charges intensives si possible
> - ⚠️ Le RAID est **vulnérable** (plus aucune redondance)
> - ✅ Remplacez rapidement tout autre disque défaillant

### Scénarios de panne

**Scénario 1 : Panne d'un disque sur RAID 1 à 2 disques**

```
Configuration : 2 disques de 1 To
État : 1 disque en panne

Disque 1 : ✅ Sain - Données complètes
Disque 2 : ❌ Panne

Résultat :
✅ Toutes les données accessibles
✅ Performances maintenues (lecture OK, écriture OK)
⚠️ AUCUNE redondance restante (CRITIQUE)
🔧 Remplacer le disque défaillant RAPIDEMENT
```

**Scénario 2 : Panne d'un disque sur RAID 1 à 3 disques**

```
Configuration : 3 disques de 1 To
État : 1 disque en panne

Disque 1 : ✅ Sain - Données complètes
Disque 2 : ❌ Panne
Disque 3 : ✅ Sain - Données complètes

Résultat :
✅ Toutes les données accessibles
✅ Performances maintenues
✅ Redondance restante (1 panne tolérée encore)
🔧 Remplacer le disque sous quelques jours
```

**Scénario 3 : Panne de 2 disques simultanés sur RAID 1 à 2 disques (CATASTROPHIQUE)**

```
Configuration : 2 disques de 1 To
État : 2 disques en panne

Disque 1 : ❌ Panne
Disque 2 : ❌ Panne

Résultat :
❌ PERTE TOTALE des données
❌ Aucune récupération possible
💾 Restauration depuis sauvegarde nécessaire
```

> [!warning] Double panne
> Bien que très rare, une double panne simultanée peut survenir si :
> - Les disques proviennent du même lot de fabrication (défaut série)
> - Un problème électrique endommage plusieurs disques
> - Une erreur humaine lors du remplacement
> 
> **Conclusion : RAID ≠ Sauvegarde !**

---

## 🔢 Nombre de disques

### Minimum requis

> [!info] Disques minimum
> **2 disques minimum** sont nécessaires pour créer un RAID 1.

### Configurations courantes

| Configuration | Usage typique |
|---------------|---------------|
| **2 disques** | Standard, équilibre coût/protection |
| **3 disques** | Protection renforcée, environnement critique |
| **4 disques** | Rare, très haute disponibilité |

> [!tip] Recommandation pratique
> **2 disques** suffisent dans 95% des cas. Au-delà, considérez plutôt :
> - **RAID 10** si vous avez 4+ disques (meilleure performance)
> - **RAID 6** si vous privilégiez la capacité avec redondance

### RAID 1 avec plus de 2 disques

Avec 3 disques ou plus, vous avez **plusieurs copies** des données :

```
RAID 1 avec 3 disques :

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Disque 1   │  │  Disque 2   │  │  Disque 3   │
├─────────────┤  ├─────────────┤  ├─────────────┤
│  Données A  │  │  Données A  │  │  Données A  │
│             │  │             │  │             │
│   Copie 1   │  │   Copie 2   │  │   Copie 3   │
└─────────────┘  └─────────────┘  └─────────────┘
```

**Avantages de 3+ disques :**
- ✅ Tolérance accrue (2+ pannes)
- ✅ Performances lecture encore meilleures
- ✅ Flexibilité maintenance (remplacer disques un par un)

**Inconvénients :**
- ❌ Capacité encore plus faible (33% avec 3 disques, 25% avec 4)
- ❌ Coût élevé (3-4× le prix d'un disque seul)
- ❌ Performance écriture légèrement réduite

---

## 💼 Cas d'usage

Le RAID 1 est idéal quand la **fiabilité** et la **disponibilité** sont prioritaires, et que la capacité n'est pas un problème majeur.

### ✅ Quand utiliser RAID 1

**Usage professionnel :**
- **Serveurs d'entreprise critiques**
  - Contrôleurs de domaine Active Directory
  - Serveurs DNS/DHCP
  - Serveurs de messagerie
- **Bases de données de petite/moyenne taille**
  - Données critiques avec fort taux de lecture
  - Transactions importantes mais volume modéré
- **Disques système (OS)**
  - Windows Server, Linux
  - Boot sécurisé et fiable
- **Serveurs de fichiers** (volumes système et configurations)
- **Systèmes embarqués critiques**
  - Automates industriels
  - Équipements médicaux

**Usage personnel :**
- **Station de travail professionnelle**
  - Documents importants
  - Projets critiques
- **NAS domestique** (pour données précieuses)
- **Serveur média** avec fiabilité prioritaire

### ❌ Quand NE PAS utiliser RAID 1

- **Gros volumes de stockage** (archivage, médias)
  - La capacité perdue (50%) devient inacceptable
  - Préférez RAID 5 ou RAID 6
- **Performance maximale requise**
  - Les écritures ne sont pas optimisées
  - Préférez RAID 0 (si acceptable) ou RAID 10
- **Budget très limité**
  - Doublement du coût de stockage
  - Considérez des sauvegardes régulières comme alternative économique

> [!example] Cas d'usage professionnel typique
> **Serveur d'entreprise PME :**
> 
> **Configuration :**
> - RAID 1 avec 2× SSD 500 Go : Système d'exploitation + Applications
> - RAID 5 avec 4× HDD 4 To : Données utilisateurs
> 
> **Avantages :**
> - OS et apps critiques protégés (RAID 1)
> - Boot rapide et fiable (SSD + RAID 1)
> - Grande capacité pour données (RAID 5)
> - Compromis coût/performance optimal

### Comparaison des coûts

```
Pour 1 To de stockage utilisable :

RAID 1 (2 disques) :
- Disques nécessaires : 2× 1 To
- Coût : 2× prix d'un disque
- Exemple : 2× 50€ = 100€

Disque unique + Sauvegardes :
- Disque : 1× 1 To
- Disque externe sauvegarde : 1× 1 To
- Coût : 2× prix d'un disque
- Exemple : 50€ + 50€ = 100€

Conclusion : Coût similaire, mais RAID 1 offre disponibilité immédiate.
```

---

## ⚖️ Avantages et inconvénients

### ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **🛡️ Excellente fiabilité** | Tolère n-1 pannes, données toujours accessibles |
| **⚡ Bonne performance lecture** | Jusqu'à 2× plus rapide avec 2 disques |
| **🔧 Reconstruction simple** | Copie directe, pas de calcul complexe |
| **📈 Disponibilité maximale** | Aucun temps d'arrêt en cas de panne |
| **🎯 Simplicité** | Facile à comprendre et configurer |
| **🚀 Temps reconstruction court** | Plus rapide que RAID 5/6 |

### ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **💰 Coût élevé** | Capacité utilisable = 50% (2 disques) |
| **📊 Perte de capacité** | 50% à 75% de capacité perdue |
| **✍️ Écriture non optimisée** | Légère baisse de performance |
| **🔢 Pas idéal pour gros volumes** | Coût prohibitif pour plusieurs To |

---

> [!tip] Règle d'or du RAID 1
> **RAID 1 = Assurance disponibilité, PAS remplacement sauvegarde**
> 
> Le RAID 1 protège contre :
> - ✅ Panne matérielle d'un disque
> - ✅ Secteurs défectueux
> 
> Le RAID 1 NE protège PAS contre :
> - ❌ Suppression accidentelle de fichiers
> - ❌ Corruption de données
> - ❌ Attaque ransomware
> - ❌ Erreur humaine
> - ❌ Sinistre (incendie, inondation)
> 
> **Solution :** RAID 1 + Sauvegardes externes régulières = Protection complète

---

> [!info] Points clés à retenir
> - **RAID 1** : Duplication complète sur tous les disques (mirroring)
> - **Capacité** : Taille du plus petit disque (perte 50%+ selon configuration)
> - **Performance** : Lecture améliorée (≈1,5-2×), écriture normale
> - **Fiabilité** : Excellente - tolère n-1 pannes simultanées
> - **Reconstruction** : Simple et rapide - copie directe
> - **Minimum** : 2 disques requis
> - **Usage idéal** : Serveurs critiques, OS, petites bases de données
> - **Priorité** : Fiabilité > Capacité
> - **Rappel** : RAID 1 ≠ Sauvegarde - Toujours faire des backups !
