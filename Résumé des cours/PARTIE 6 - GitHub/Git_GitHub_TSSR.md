# Git & GitHub
## Versionnement de projets

---

**Formation TSSR - Titre RNCP**  
**Date de révision :** Novembre 2025  
**Contexte :** Préparation titre Technicien Supérieur Systèmes et Réseaux

---

## 📋 Sommaire

1. [Git](#git)
   - [La gestion de version](#la-gestion-de-version)
   - [Concept et histoire](#concept-et-histoire)
   - [Pourquoi Git ?](#pourquoi-git-)
   - [Centralisé vs Décentralisé](#centralisé-vs-décentralisé)
   - [Base de fonctionnement](#base-de-fonctionnement)
   - [Les 3 états des fichiers](#base-de-fonctionnement-1)
   - [Commandes indispensables](#les-commandes-indispensables)
   - [Les branches](#les-branches)
   - [Workflow](#workflow)
2. [GitHub](#github)
   - [Qu'est-ce que c'est ?](#github-quest-ce-que-cest-)
   - [Fonctionnalités](#fonctionnalités)
   - [Fork et Pull Request](#fork)
3. [Conclusion](#conclusion)
4. [Points clés à retenir](#points-clés-à-retenir)
5. [Glossaire technique](#glossaire-technique)
6. [Ressources](#ressources)

---

## 🔧 Git

### La gestion de version

#### Pourquoi gérer des versions ?

**3 raisons principales :**

1. 👥 **Travailler à plusieurs** sur un même projet
   - Collaboration efficace
   - Travail simultané sans conflits

2. ⏮️ **Revenir à une version antérieure** d'un fichier ou du projet complet
   - En cas de bugs
   - Pour comparer les évolutions
   - Pour annuler des modifications

3. 🚀 **Développer des fonctionnalités plus facilement**
   - Isolation des développements
   - Tests sans risque
   - Intégration progressive

---

### Concept et histoire

#### Qu'est-ce qu'un logiciel de gestion de versions ?

> **Définition :** Un logiciel de gestion de versions permet de **stocker un ensemble de fichiers** en conservant la **chronologie de toutes les modifications** qui ont été effectuées dessus.

---

#### 📅 Quelques dates-clés

| Année | Événement | Description |
|-------|-----------|-------------|
| **Années 90** | **CVS** | Création du système de versionnement CVS (Concurrent Versions System) |
| **Début 2000** | **Apache SVN** | Subversion - Amélioration de CVS |
| **2005** | **GIT** | Création de Git par **Linus Torvalds** (créateur de Linux) |

---

#### 🎯 Évolution des systèmes de versionnement

```
CVS (1990)
  │
  ├─ Concurrent Versions System
  ├─ Premier système de version populaire
  └─ Centralisé
      │
      ▼
SVN (2000)
  │
  ├─ Subversion
  ├─ Amélioration de CVS
  └─ Centralisé
      │
      ▼
GIT (2005)
  │
  ├─ Créé par Linus Torvalds
  ├─ Pour gérer le code source de Linux
  └─ Décentralisé ✅
```

---

### Pourquoi Git ?

#### Les avantages

#### ✅ Décentralisé

- **Permet de commencer à travailler tout de suite** (pas de serveur requis)
- Chaque développeur a une copie complète de l'historique
- Travail possible hors ligne

#### ✅ Très rapide

- Parce que décentralisé
- Opérations locales (pas d'attente réseau)
- Algorithmes optimisés

#### ✅ "Relativement" simple

- Gestion des branches facilitée
- Création, fusion, suppression de branches rapide
- Workflow flexible

---

#### 🔄 Mais il existe d'autres gestionnaires de versions

| Outil | Type | Particularité |
|-------|------|---------------|
| **SVN** | Centralisé | Subversion, successeur de CVS |
| **CVS** | Centralisé | Concurrent Versions System, ancien |
| **Mercurial** | Décentralisé | Alternative à Git, Python |
| **Bazaar** | Décentralisé | Par Canonical (Ubuntu) |
| **Perforce** | Centralisé | Pour grandes entreprises |
| **Git** | **Décentralisé** | **Le plus populaire** ✅ |

---

### Centralisé vs Décentralisé

#### 🏢 Système Centralisé

```
┌─────────────────────────────────┐
│      SERVEUR CENTRAL            │
│   (Unique source de vérité)     │
│                                 │
│    ┌─────────────────────┐     │
│    │  Historique complet │     │
│    │  Toutes les versions│     │
│    └─────────────────────┘     │
└────────┬──────────┬─────────────┘
         │          │
    ┌────▼────┐ ┌──▼──────┐
    │ Client 1│ │ Client 2│
    │ (copie  │ │ (copie  │
    │ locale) │ │ locale) │
    └─────────┘ └─────────┘
```

**Caractéristiques :**
- ✅ Contrôle centralisé
- ❌ Nécessite une connexion au serveur
- ❌ Point de défaillance unique
- ❌ Performances dépendantes du réseau
- Exemple : **SVN**, **CVS**

---

#### 🌐 Système Décentralisé

```
┌─────────────────┐       ┌─────────────────┐
│  Développeur 1  │       │  Développeur 2  │
│                 │       │                 │
│ ┌─────────────┐ │       │ ┌─────────────┐ │
│ │  Dépôt local│ │       │ │  Dépôt local│ │
│ │  (complet)  │ │◄─────►│ │  (complet)  │ │
│ └─────────────┘ │       │ └─────────────┘ │
└────────┬────────┘       └────────┬────────┘
         │                         │
         │      ┌─────────────┐    │
         └─────►│   Serveur   │◄───┘
                │  (optionnel)│
                │ ┌─────────┐ │
                │ │  Dépôt  │ │
                │ │(complet)│ │
                │ └─────────┘ │
                └─────────────┘
```

**Caractéristiques :**
- ✅ Chaque client a l'historique complet
- ✅ Travail hors ligne possible
- ✅ Pas de point de défaillance unique
- ✅ Performances excellentes (local)
- ✅ Flexibilité dans les workflows
- Exemple : **Git**, **Mercurial**

---

### Base de fonctionnement

#### Les 3 étapes du versionnement

**Processus de travail avec Git :**

1. 📝 **Tu fais une modification** dans ton répertoire de travail (Working Directory)

2. ➕ **Tu ajoutes (`add`)** tes fichiers à l'index (snapshots de tes fichiers)

3. 💾 **Tu fais un commit** lorsque ton projet est dans un état "propre"
   - Nouvelle fonctionnalité complète
   - Sous-partie fonctionnelle
   - Sauvegarde avec un **ID** unique et un **commentaire**

---

#### 🌳 Git gère 3 "arbres" principaux

```
┌─────────────────────────────────────────────────────────┐
│                  RÉPERTOIRE DE TRAVAIL                  │
│                    (Working Directory)                   │
│                                                          │
│  Fichiers sur lesquels tu travailles actuellement       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ file1.js │  │ file2.js │  │ file3.js │             │
│  └──────────┘  └──────────┘  └──────────┘             │
└───────────────────────┬──────────────────────────────────┘
                        │ git add
                        ▼
┌─────────────────────────────────────────────────────────┐
│                      INDEX (STAGE)                       │
│                                                          │
│  Fichiers prêts pour le prochain commit                 │
│  ┌──────────┐  ┌──────────┐                            │
│  │ file1.js │  │ file2.js │   (snapshots)              │
│  └──────────┘  └──────────┘                            │
└───────────────────────┬──────────────────────────────────┘
                        │ git commit
                        ▼
┌─────────────────────────────────────────────────────────┐
│                         HEAD                             │
│                  (Dernier commit)                        │
│                                                          │
│  État validé de ton projet                               │
│  ┌─────────────────────────────────────────┐            │
│  │  Commit abc123: "Add new features"      │            │
│  │  Date: 2025-11-19                       │            │
│  │  Files: file1.js, file2.js              │            │
│  └─────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────┘
```

---

### Base de fonctionnement

#### États des fichiers

**Les fichiers dans le répertoire de travail ne peuvent avoir que 2 états :**

1. 🔍 **Tracked** (Suivis par Git)
2. ❓ **Untracked** (Non suivis par Git)

---

**Les fichiers "tracked" peuvent avoir 3 états :**

| État | Description | Action nécessaire |
|------|-------------|-------------------|
| **Modified** | Modifiés mais **pas** ajoutés à l'index | `git add` pour passer en Staged |
| **Unmodified** | Commités, aucune modification | Aucune action |
| **Staged** | Modifiés **et** ajoutés à l'index | `git commit` pour valider |

---

#### 🔄 Cycle de vie d'un fichier

```
┌──────────────┐
│  UNTRACKED   │  Nouveau fichier créé
└──────┬───────┘
       │ git add
       ▼
┌──────────────┐
│    STAGED    │  Fichier ajouté à l'index
└──────┬───────┘
       │ git commit
       ▼
┌──────────────┐
│  UNMODIFIED  │  Fichier commité
└──────┬───────┘
       │ Modification du fichier
       ▼
┌──────────────┐
│   MODIFIED   │  Fichier modifié
└──────┬───────┘
       │ git add
       ▼
┌──────────────┐
│    STAGED    │  Prêt pour commit
└──────┬───────┘
       │ git commit
       ▼
┌──────────────┐
│  UNMODIFIED  │  Retour à l'état commité
└──────────────┘
```

---

### Les commandes indispensables

#### Connaître le statut de ton repository

**Commande la plus importante :**

```bash
git status
```

**Informations fournies :**
- ✅ Quels sont les fichiers **untracked** (non suivis)
- ✅ Quels sont les fichiers **modified** ou **supprimés** non versionnés
- ✅ Quels sont les **conflits** éventuels
- ✅ Sur quelle **branche** tu es
- ✅ État de synchronisation avec le **remote**

---

#### Exemple de git status

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   index.html
        new file:   style.css

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   script.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md
```

> 💡 **Conseil :** Use et abuse de cette commande !

---

### Les commandes indispensables (suite)

#### Voir l'historique des modifications

```bash
git log
```

**Affiche :**
- ✅ Liste des **commits** effectués
- ✅ **Auteur** de chaque commit
- ✅ **Date** de chaque commit
- ✅ **Message** de commit
- ✅ **Hash** (identifiant unique) du commit

---

#### Exemple de git log

```bash
$ git log
commit a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0 (HEAD -> main, origin/main)
Author: John Doe <john.doe@example.com>
Date:   Tue Nov 19 10:30:45 2025 +0100

    Add new feature X

commit b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1
Author: Jane Smith <jane.smith@example.com>
Date:   Mon Nov 18 15:22:30 2025 +0100

    Fix bug in authentication

commit c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2
Author: John Doe <john.doe@example.com>
Date:   Mon Nov 18 09:15:10 2025 +0100

    Initial commit
```

**Options utiles :**
```bash
git log --oneline           # Format condensé
git log --graph             # Affichage graphique des branches
git log --all               # Tous les commits (toutes branches)
git log -n 5                # Les 5 derniers commits
git log --author="John"     # Commits d'un auteur
```

---

### Les commandes indispensables (suite)

#### 📚 Commandes essentielles

| Commande | Fonction principale |
|----------|---------------------|
| **init** | Initialiser un nouveau dépôt Git |
| **add** | Ajouter des fichiers à l'index (staging) |
| **commit** | Enregistrer les modifications dans l'historique |
| **status** | Voir l'état du dépôt |
| **checkout** | Changer de branche ou restaurer des fichiers |
| **branch** | Gérer les branches |
| **push** | Envoyer les commits vers un dépôt distant |
| **pull** | Récupérer et fusionner depuis un dépôt distant |
| **merge** | Fusionner des branches |
| **clone** | Cloner un dépôt distant |

---

#### 📚 Commandes avancées

| Commande | Fonction principale |
|----------|---------------------|
| **remote** | Gérer les dépôts distants |
| **log** | Voir l'historique des commits |
| **diff** | Voir les différences entre versions |
| **stash** | Mettre de côté temporairement des modifications |
| **reset** | Annuler des commits (attention : destructif) |
| **revert** | Annuler un commit en créant un nouveau commit |
| **config** | Configurer Git |
| **fetch** | Récupérer depuis un dépôt distant (sans fusionner) |

---

### Exemples de commandes Git

#### Initialiser et configurer

```bash
# Initialiser un nouveau dépôt
git init

# Configurer son identité
git config --global user.name "John Doe"
git config --global user.email "john.doe@example.com"

# Voir la configuration
git config --list
```

---

#### Workflow de base

```bash
# Voir l'état
git status

# Ajouter un fichier à l'index
git add fichier.txt

# Ajouter tous les fichiers modifiés
git add .

# Commiter avec un message
git commit -m "Add new feature"

# Voir l'historique
git log

# Voir les différences
git diff
```

---

#### Travailler avec les branches

```bash
# Créer une nouvelle branche
git branch feature-x

# Changer de branche
git checkout feature-x

# Créer et changer en une seule commande
git checkout -b feature-y

# Lister les branches
git branch

# Fusionner une branche
git checkout main
git merge feature-x

# Supprimer une branche
git branch -d feature-x
```

---

#### Travailler avec un dépôt distant

```bash
# Cloner un dépôt
git clone https://github.com/user/repo.git

# Ajouter un dépôt distant
git remote add origin https://github.com/user/repo.git

# Voir les dépôts distants
git remote -v

# Envoyer ses commits
git push origin main

# Récupérer et fusionner
git pull origin main

# Seulement récupérer (sans fusionner)
git fetch origin
```

---

#### Annuler des modifications

```bash
# Annuler les modifications d'un fichier (non staged)
git restore fichier.txt

# Retirer un fichier de l'index (unstage)
git restore --staged fichier.txt

# Annuler le dernier commit (garde les modifs)
git reset --soft HEAD~1

# Annuler le dernier commit (supprime les modifs)
git reset --hard HEAD~1

# Créer un commit qui annule un commit précédent
git revert <commit-hash>
```

---

#### Mettre de côté temporairement

```bash
# Mettre de côté les modifications
git stash

# Lister les stash
git stash list

# Réappliquer le dernier stash
git stash pop

# Réappliquer un stash spécifique
git stash apply stash@{0}

# Supprimer un stash
git stash drop stash@{0}
```

---

### Les branches

#### Définition

> **Par convention**, la branche principale d'un projet se nomme **`main`** (anciennement `master`).

> ⚠️ **Règle d'or :** Elle DOIT TOUJOURS être pleinement **FONCTIONNELLE** !

---

#### 🌳 Types de branches

**On distingue 2 types de branches :**

#### 1. Branches permanentes (toujours existantes)

| Branche | Rôle |
|---------|------|
| **main** | Production, code stable |
| **develop** | Développement en cours |
| **release** | Préparation d'une nouvelle version |
| **preprod** | Pré-production, tests finaux |
| **recette** | Tests de validation |

---

#### 2. Branches temporaires (amenées à disparaître)

| Type | Exemple | Usage |
|------|---------|-------|
| **Feature** | `feature/login` | Nouvelle fonctionnalité |
| **Bugfix** | `bugfix/auth-error` | Correction de bug |
| **Hotfix** | `hotfix/critical-fix` | Correction urgente en prod |
| **User Story** | `us/user-registration` | Développement d'une user story |

---

#### 📝 Convention de nommage

**Format recommandé :**
```
<type>/<description-courte>

Exemples :
- feature/add-user-authentication
- bugfix/fix-login-error
- hotfix/security-patch
- release/v1.2.0
```

---

#### 🔀 Opérations sur les branches

```bash
# Créer une branche
git branch feature/login

# Changer de branche
git checkout feature/login

# Créer et changer (raccourci)
git checkout -b feature/login

# Lister toutes les branches
git branch -a

# Renommer une branche
git branch -m ancien-nom nouveau-nom

# Supprimer une branche (si fusionnée)
git branch -d feature/login

# Forcer la suppression (même non fusionnée)
git branch -D feature/login

# Fusionner une branche
git checkout main
git merge feature/login

# Voir les branches fusionnées
git branch --merged

# Voir les branches non fusionnées
git branch --no-merged
```

---

### Workflow

#### Par où commencer ?

**Exemple de workflow Git Flow :**

```
         main (production)
           │
           │  ┌─────────── hotfix ─────────┐
           │  │                             │
           ├──┴───────────────────────────┬─┤
           │                              │
           │         release              │
           │            │                 │
      ┌────┴────┐      │                 │
      │  Tag    │      ▼                 │
      │ v1.0.0  │   develop              │
      └─────────┘      │                 │
                       │                 │
         ┌─────────────┼────────┐        │
         │             │        │        │
    feature/A     feature/B  bugfix     │
         │             │        │        │
         └─────────────┴────────┴────────┘
                       │
                    (merge)
                       │
                       ▼
                    develop
                       │
                       ▼
                    release
                       │
                       ▼
                     main
```

> 📌 **Note :** Ce schéma est un exemple de workflow, non celui utilisé durant la formation.

---

#### 🔄 Workflow simplifié

**Pour les petits projets ou équipes réduites :**

```
1. main (branche principale)
   └─> feature/nouvelle-fonctionnalite
       │
       ├─> Développement
       │
       └─> Merge vers main
```

**Étapes :**
1. Créer une branche depuis `main`
2. Développer la fonctionnalité
3. Tester
4. Merger dans `main`
5. Supprimer la branche temporaire

---

#### 🏢 Git Flow (workflow complet)

**Pour les projets d'entreprise :**

```
Branches permanentes :
- main (production)
- develop (développement)

Branches temporaires :
- feature/* (fonctionnalités)
- release/* (versions)
- hotfix/* (corrections urgentes)
```

**Processus :**
1. Créer `feature/*` depuis `develop`
2. Développer
3. Merger dans `develop`
4. Créer `release/*` depuis `develop`
5. Tester et corriger dans `release/*`
6. Merger `release/*` dans `main` ET `develop`
7. Tag la version dans `main`

---

## 🐙 GitHub

### GitHub, qu'est-ce que c'est ?

#### Définition

> **GitHub** est un **service web de gestion de version**, créé en **2008**.

**Caractéristiques :**
- ✅ Basé sur **Git**
- ✅ Modèle **gratuit** pour des dépôts publics/privés (open-source)
- ✅ Modèle **payant** pour des fonctionnalités avancées
- ✅ Hébergement dans le cloud

---

#### ⚠️ Attention à la confusion !

```
┌─────────────────────────────────────────────────┐
│  GIT  ≠  GITHUB                                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  GIT :                                          │
│  - Logiciel de gestion de versions             │
│  - Installé localement                          │
│  - Fonctionne en ligne de commande             │
│  - Open source et gratuit                       │
│                                                 │
│  GITHUB :                                       │
│  - Plateforme web d'hébergement                │
│  - Service en ligne                             │
│  - Interface graphique                          │
│  - Gratuit (avec limites) ou payant           │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 🔄 Il existe d'autres services similaires

| Service | Propriétaire | Particularité |
|---------|--------------|---------------|
| **GitHub** | Microsoft | Le plus populaire, communauté énorme |
| **GitLab** | GitLab Inc. | Auto-hébergeable, CI/CD intégré |
| **Bitbucket** | Atlassian | Intégration avec Jira |
| **Gitea** | Open source | Léger, auto-hébergeable |
| **SourceForge** | Slashdot Media | Ancien, pour projets open source |

---

### Fonctionnalités

#### 🌐 Véritable réseau social des développeurs

**Fonctionnalités sociales :**
- ✅ **Suivi de projets** (repositories)
- ✅ **Suivi de personnes** (followers/following)
- ✅ **Création d'équipes** (organizations)
- ✅ **Stars** (favoris)
- ✅ **Discussions** (issues, pull requests)
- ✅ **Profil développeur** (contributions, statistiques)

---

#### 🛠️ Services pour les projets

| Service | Description |
|---------|-------------|
| **Wiki** | Documentation du projet |
| **Pages** | Hébergement de site web statique |
| **Issues** | Suivi de problèmes et tâches |
| **Projects** | Gestion de projet (Kanban) |
| **Actions** | CI/CD (Intégration/Déploiement continu) |
| **Packages** | Hébergement de packages |
| **Security** | Analyse de sécurité, Dependabot |

---

#### 🔌 Intégration de services externes

**Exemples d'intégrations :**
- **Slack** : Notifications
- **Discord** : Webhooks
- **Travis CI** : Intégration continue
- **Heroku** : Déploiement automatique
- **Code Climate** : Qualité du code
- **Codecov** : Couverture de tests

---

### Fork

#### GitHub incite au fork

> **Définition du fork :** Copier un projet (open-source), l'installer sur son propre compte GitHub et le modifier selon ses besoins.

---

#### 🔱 Processus de contribution

```
┌─────────────────────────────────────────────────┐
│  1. FORK                                        │
│                                                 │
│  Projet Original        →    Ton Fork           │
│  (github.com/auteur)        (github.com/toi)    │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  2. CLONE                                       │
│                                                 │
│  Ton Fork  →  Dépôt Local                       │
│                                                 │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  3. DÉVELOPPEMENT                               │
│                                                 │
│  - Créer une branche                            │
│  - Faire des modifications                      │
│  - Commit                                       │
│                                                 │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  4. PUSH                                        │
│                                                 │
│  Dépôt Local  →  Ton Fork                       │
│                                                 │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│  5. PULL REQUEST                                │
│                                                 │
│  Ton Fork  →  Projet Original                   │
│  (Demande de contribution)                      │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

#### 📋 Qu'est-ce qu'une Pull Request ?

> **Pull Request (PR)** : Demande de contribution qui peut être réalisée facilement vers le dépôt du contributeur initial.

**Une PR permet de :**
- ✅ Proposer des modifications
- ✅ Discuter des changements
- ✅ Réviser le code (code review)
- ✅ Tester avant d'intégrer
- ✅ Documenter les contributions

---

#### 🔄 Workflow de Pull Request

```bash
# 1. Fork le projet sur GitHub (via l'interface)

# 2. Clone ton fork
git clone https://github.com/ton-username/projet.git

# 3. Ajouter l'upstream (projet original)
git remote add upstream https://github.com/auteur-original/projet.git

# 4. Créer une branche pour ta modification
git checkout -b feature/ma-modification

# 5. Faire tes modifications et commit
git add .
git commit -m "Add new feature"

# 6. Push vers ton fork
git push origin feature/ma-modification

# 7. Créer une Pull Request sur GitHub (via l'interface)

# 8. Attendre la revue et l'acceptation

# 9. Une fois acceptée, mettre à jour ton fork
git checkout main
git pull upstream main
git push origin main
```

---

## ✅ Conclusion

### Avantages de Git

| Avantage | Explication |
|----------|-------------|
| **🌿 Système de branches** | Bascule facile entre features, sans interférence |
| **🔌 Décentralisé** | Tout en local, pas besoin de connexion serveur pour les fusions |
| **⚡ Algorithmes efficaces** | Fusion très performante, détection automatique des conflits |
| **🚀 Commits rapides** | Opérations instantanées (local) |
| **📦 Historique complet** | Chaque développeur a tout l'historique |
| **🔄 Flexibilité** | Nombreux workflows possibles |

---

### Contraintes de Git

| Contrainte | Solution |
|------------|----------|
| **📚 Commandes peu intuitives** | Pratique, formation, documentation |
| **🤝 Nécessité d'un workflow commun** | Définir des conventions d'équipe |
| **🆕 Courbe d'apprentissage** | Débuter avec les commandes de base |
| **⚠️ Risque d'erreurs** | Prudence avec `reset --hard`, backups |

---

## 🎯 Points clés à retenir

### ✅ Concepts fondamentaux

- [ ] **Git** = Logiciel de gestion de versions décentralisé
- [ ] **GitHub** = Service web d'hébergement de dépôts Git
- [ ] **Décentralisé** = Chaque développeur a l'historique complet
- [ ] **3 états** : Working Directory, Index (Stage), HEAD
- [ ] **Fichiers** : Untracked → Modified → Staged → Unmodified

### ✅ Commandes essentielles

- [ ] `git init` : Initialiser un dépôt
- [ ] `git clone` : Cloner un dépôt distant
- [ ] `git add` : Ajouter à l'index
- [ ] `git commit` : Enregistrer les modifications
- [ ] `git status` : Voir l'état du dépôt
- [ ] `git log` : Voir l'historique
- [ ] `git diff` : Voir les différences

### ✅ Branches

- [ ] `git branch` : Gérer les branches
- [ ] `git checkout` : Changer de branche
- [ ] `git merge` : Fusionner des branches
- [ ] Convention : **main** = branche principale (toujours stable)
- [ ] Branches permanentes : main, develop, release
- [ ] Branches temporaires : feature/*, bugfix/*, hotfix/*

### ✅ Travail en équipe

- [ ] `git push` : Envoyer vers le dépôt distant
- [ ] `git pull` : Récupérer et fusionner
- [ ] `git fetch` : Récupérer sans fusionner
- [ ] `git remote` : Gérer les dépôts distants
- [ ] **Fork** : Copier un projet sur son compte
- [ ] **Pull Request** : Demander l'intégration de modifications

### ✅ GitHub

- [ ] Service web ≠ Git (logiciel)
- [ ] Hébergement de code
- [ ] Fonctionnalités sociales (follow, star, fork)
- [ ] Services : Wiki, Pages, Issues, Projects, Actions
- [ ] Alternatives : GitLab, Bitbucket

### ✅ Workflow

- [ ] Définir un workflow commun en équipe
- [ ] Git Flow : branches permanentes + temporaires
- [ ] Toujours garder `main` fonctionnelle
- [ ] Créer des branches pour chaque feature/bugfix
- [ ] Faire des commits atomiques avec messages clairs
- [ ] Tester avant de merger dans main

---

## 📊 Tableaux récapitulatifs

### Centralisé vs Décentralisé

| Aspect | Centralisé (SVN) | Décentralisé (Git) |
|--------|------------------|---------------------|
| **Historique** | Uniquement sur le serveur | Sur chaque client |
| **Connexion** | Nécessaire pour la plupart des opérations | Optionnelle |
| **Performance** | Dépend du réseau | Très rapide (local) |
| **Point de défaillance** | Serveur unique | Aucun (multiples copies) |
| **Travail hors ligne** | Limité | Complet |
| **Complexité** | Plus simple | Plus complexe |

---

### États des fichiers

| État | Description | Commande pour passer à l'état suivant |
|------|-------------|--------------------------------------|
| **Untracked** | Nouveau fichier, non suivi | `git add` |
| **Modified** | Modifié, non indexé | `git add` |
| **Staged** | Indexé, prêt pour commit | `git commit` |
| **Unmodified** | Commité, aucune modification | (modifier le fichier) |

---

### Commandes par catégorie

#### Configuration
```bash
git config --global user.name "Nom"
git config --global user.email "email@example.com"
git config --list
```

#### Création/Clone
```bash
git init
git clone <url>
```

#### Modifications
```bash
git add <fichier>
git add .
git commit -m "message"
git status
git diff
```

#### Branches
```bash
git branch
git branch <nom>
git checkout <branche>
git checkout -b <nouvelle-branche>
git merge <branche>
git branch -d <branche>
```

#### Remote
```bash
git remote add origin <url>
git push origin <branche>
git pull origin <branche>
git fetch origin
```

#### Historique
```bash
git log
git log --oneline
git log --graph
```

#### Annulation
```bash
git restore <fichier>
git restore --staged <fichier>
git reset --soft HEAD~1
git reset --hard HEAD~1
git revert <commit>
```

---

## 📖 Glossaire technique

| Terme | Définition |
|-------|------------|
| **Git** | Logiciel de gestion de versions décentralisé créé par Linus Torvalds |
| **GitHub** | Service web d'hébergement de dépôts Git |
| **Repository (dépôt)** | Projet géré par Git contenant l'historique complet |
| **Commit** | Enregistrement d'un état du projet avec un message |
| **Branch (branche)** | Ligne de développement indépendante |
| **main** | Branche principale (anciennement master) |
| **Working Directory** | Répertoire de travail contenant les fichiers actuels |
| **Index (Stage)** | Zone intermédiaire avant le commit |
| **HEAD** | Pointeur vers le dernier commit de la branche actuelle |
| **Tracked** | Fichier suivi par Git |
| **Untracked** | Fichier non suivi par Git |
| **Modified** | Fichier modifié mais non indexé |
| **Staged** | Fichier indexé, prêt pour commit |
| **Unmodified** | Fichier commité sans modification |
| **Remote** | Dépôt distant (ex: sur GitHub) |
| **Origin** | Nom par défaut du dépôt distant |
| **Push** | Envoyer les commits vers un dépôt distant |
| **Pull** | Récupérer et fusionner depuis un dépôt distant |
| **Fetch** | Récupérer depuis un dépôt distant sans fusionner |
| **Clone** | Copier un dépôt distant en local |
| **Fork** | Copier un projet sur son propre compte GitHub |
| **Pull Request (PR)** | Demande d'intégration de modifications |
| **Merge** | Fusionner deux branches |
| **Conflict** | Conflit lors d'une fusion (modifications incompatibles) |
| **Stash** | Mettre de côté temporairement des modifications |
| **Tag** | Marquer un commit spécifique (ex: version) |
| **Hash** | Identifiant unique d'un commit (SHA-1) |
| **Upstream** | Dépôt original (dans le cas d'un fork) |
| **Workflow** | Processus de travail défini pour utiliser Git |

---

## 📚 Ressources

### Documentation Git

- **Documentation officielle :** https://git-scm.com/docs
- **Git - guide simple (français) :** http://rogerdudler.github.io/git-guide/index.fr.html
- **OpenClassrooms - Git :** https://openclassrooms.com/courses/gerez-vos-codes-source-avec-git

### GitHub

- **OpenClassrooms - Git et GitHub :** https://openclassrooms.com/courses/gerer-son-code-avec-git-et-github/qu-est-ce-que-versionner-son-code

### Git Workflows

- **Quel Git workflow pour mon projet :** http://www.nicoespeon.com/fr/2013/08/quel-git-workflow-pour-mon-projet/
- **Git workflow efficace :** https://makina-corpus.com/blog/metier/2014/un-workflow-git-efficace-pour-les-projets-a-moyen-long-terme

### Ressources additionnelles

- **Pro Git Book (gratuit) :** https://git-scm.com/book/fr/v2
- **Git Cheat Sheet :** https://education.github.com/git-cheat-sheet-education.pdf
- **Learn Git Branching (interactif) :** https://learngitbranching.js.org/?locale=fr_FR
- **Oh Shit, Git!? :** https://ohshitgit.com/fr

---

## 📝 Checklist de révision

### Avant l'examen TSSR, je dois savoir :

#### Concepts généraux
- [ ] Définir ce qu'est la gestion de versions
- [ ] Expliquer pourquoi utiliser Git
- [ ] Différencier Git et GitHub
- [ ] Comprendre centralisé vs décentralisé
- [ ] Expliquer les 3 "arbres" de Git

#### États et cycle de vie
- [ ] Identifier les états d'un fichier (Untracked, Modified, Staged, Unmodified)
- [ ] Comprendre le cycle de vie d'un fichier dans Git
- [ ] Savoir quand utiliser `git add` vs `git commit`

#### Commandes de base
- [ ] Initialiser un dépôt : `git init`
- [ ] Cloner un dépôt : `git clone`
- [ ] Voir l'état : `git status`
- [ ] Ajouter à l'index : `git add`
- [ ] Commiter : `git commit`
- [ ] Voir l'historique : `git log`
- [ ] Voir les différences : `git diff`

#### Branches
- [ ] Créer une branche : `git branch`
- [ ] Changer de branche : `git checkout`
- [ ] Créer et changer : `git checkout -b`
- [ ] Fusionner : `git merge`
- [ ] Supprimer une branche : `git branch -d`
- [ ] Comprendre la convention de nommage
- [ ] Savoir que `main` doit toujours être stable

#### Travail en équipe
- [ ] Ajouter un remote : `git remote add`
- [ ] Pousser : `git push`
- [ ] Tirer : `git pull`
- [ ] Récupérer : `git fetch`
- [ ] Comprendre le workflow de fork/PR

#### GitHub
- [ ] Créer un compte GitHub
- [ ] Créer un repository
- [ ] Forker un projet
- [ ] Créer une Pull Request
- [ ] Connaître les fonctionnalités (Issues, Wiki, Pages)

#### Bonnes pratiques
- [ ] Messages de commit clairs et descriptifs
- [ ] Commits atomiques (une seule chose à la fois)
- [ ] Ne jamais commiter directement dans `main`
- [ ] Tester avant de merger
- [ ] Définir un workflow avec l'équipe

---

**🎓 Bon courage pour ta préparation au titre RNCP TSSR !**

---

*Document de révision créé pour la formation TSSR - Novembre 2025*  
*Git & GitHub : Versionnement de projets*
