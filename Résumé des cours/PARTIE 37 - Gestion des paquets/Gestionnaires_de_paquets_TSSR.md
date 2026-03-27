# Les Gestionnaires de Paquets
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)
**Sujet** : Les gestionnaires de paquets Linux
**Date** : Février 2026
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Historique|Historique]]
   - [[#Définition|Définition]]
   - [[#Notions fondamentales|Notions fondamentales]]
2. [[#Principaux gestionnaires|Principaux gestionnaires]]
   - [[#Vue d'ensemble par distribution|Vue d'ensemble par distribution]]
   - [[#APT — Debian et dérivés|APT — Debian et dérivés]]
   - [[#apt vs apt-get|apt vs apt-get]]
   - [[#Interfaces graphiques|Interfaces graphiques]]
   - [[#Gestionnaires universels avec isolation|Gestionnaires universels avec isolation]]
3. [[#Architecture et fonctionnement|Architecture et fonctionnement]]
   - [[#Bas niveau vs Haut niveau|Bas niveau vs Haut niveau]]
   - [[#Les dépôts|Les dépôts]]
   - [[#Le fichier sources.list|Le fichier sources.list]]
4. [[#Commandes de base|Commandes de base]]
   - [[#Commandes dpkg|Commandes dpkg]]
   - [[#Commandes apt|Commandes apt]]
5. [[#Sécurité|Sécurité]]
6. [[#Points clés à retenir|Points clés à retenir]]
7. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Les gestionnaires de paquets sont des outils fondamentaux sur Linux. Ils automatisent l'installation, la désinstallation et la mise à jour des logiciels. Chaque distribution Linux a le sien, en fonction de sa "souche" d'origine.

### Historique

> [!info] Évolution historique
> Initialement, l'installation et la mise à jour des logiciels sous Linux étaient des processus **manuels et fastidieux**.
>
> Des outils bas niveau ont d'abord été développés :
> - **Debian** → **DPKG**
> - **Red Hat** → **RPM**
>
> Ces outils ont évolué vers de véritables gestionnaires de paquets haut niveau :
> - **Debian** → **APT** (basé sur DPKG)
> - **Red Hat** → **YUM** (basé sur RPM)

### Définition

> [!quote] Définition officielle
> Un **gestionnaire de paquets** est un outil automatisant le processus d'installation, désinstallation et mise à jour de logiciels installés sur un système Linux de manière organisée et cohérente, souvent à partir de **dépôts**. Il comprend également la **gestion des dépendances**.

### Notions fondamentales

> [!important] Les 3 concepts essentiels

- **Paquet** : archive contenant des fichiers binaires, des bibliothèques, des scripts et des métadonnées
- **Dépendances** : les paquets peuvent dépendre d'autres paquets pour fonctionner correctement
- **Dépôt** : référentiel centralisé de logiciels vérifiés et compatibles avec la distribution

> [!tip] Mnémotechnique
> **P**aquet + **D**épendances + **D**épôt = les **3D** des gestionnaires de paquets

---

## Principaux gestionnaires

### Vue d'ensemble par distribution

> [!important] Tableau récapitulatif — À mémoriser absolument !

| Distribution | Gestionnaire de paquets | Type de paquet |
|-------------|------------------------|----------------|
| **Ubuntu** | APT | `.deb` |
| **Debian** | APT | `.deb` |
| **Linux Mint** | APT | `.deb` |
| **Kali Linux** | APT | `.deb` |
| **Fedora** | DNF | `.rpm` |
| **CentOS** | YUM | `.rpm` |
| **Red Hat** | YUM | `.rpm` |
| **OpenSUSE** | Zypper | `.rpm` |
| **Arch Linux** | Pacman | `.pkg` |
| **Manjaro** | Pacman | `.pck` |

> [!note] Pourquoi tant de gestionnaires différents ?
> - Philosophies et approches différentes
> - Ancienneté et continuité (héritage historique)
> - Adaptation aux besoins spécifiques de chaque distribution

### APT — Debian et dérivés

> [!info] APT (Advanced Package Tool)
> APT est le gestionnaire de paquets avancé utilisé principalement dans les distributions **Debian et dérivées** (Ubuntu, Mint, Kali, Tails…). Il est **basé sur dpkg** (bas niveau).

**Caractéristiques principales :**
- Gestion automatique des dépendances
- Commandes simples et intuitives
- Gestion des sources de logiciels (dépôts)
- Mises à jour régulières disponibles
- Vérification de l'authenticité des paquets (GPG)

### apt vs apt-get

> [!info] Deux outils, une même famille
> `apt` (2014) et `apt-get` (1998) sont deux outils de gestion de paquets sur les systèmes Debian. `apt` est plus récent et dispose de plus de fonctionnalités.

| Fonctionnalité | `apt` | `apt-get` |
|---------------|-------|-----------|
| **Recherche** | `apt search <paquet>` | `apt-cache search <paquet>` |
| **Gestion des dépendances** | Complexe et avancée | Basique |
| **Mise à jour des versions** | `apt upgrade` (gestion intelligente) | `apt-get upgrade` (pas de suppression des anciennes versions) |
| **Barre de progression** | ✅ Oui | ❌ Non |

> [!tip] Conseil pratique
> Préférer **`apt`** dans vos scripts et en ligne de commande : plus moderne, plus lisible, plus complet.

### Interfaces graphiques

> [!note] Pour ceux qui préfèrent le graphique
> - **Aptitude** : interface semi-graphique en mode texte (terminal)
> - **Synaptic** : interface graphique complète (GUI) — `apt` en mode visuel

### Gestionnaires universels avec isolation

> [!info] Snap, Flatpak, AppImage
> Ces gestionnaires universels **isolent les applications** en les empaquetant avec leurs propres dépendances (comme Docker pour les conteneurs).
>
> **Avantages :**
> - Pas de conflits entre différentes versions de bibliothèques
> - Applications portables entre distributions
> - MAJ continues et indépendantes de la distribution
>
> **Inconvénients :**
> - Consomment plus d'espace disque et de mémoire (dépendances dupliquées)

---

## Architecture et fonctionnement

### Bas niveau vs Haut niveau

> [!important] Les 2 couches de gestion des paquets

| Niveau | Outil (Debian) | Outil (Red Hat) | Rôle |
|--------|---------------|-----------------|------|
| **Bas niveau** | `dpkg` | `rpm` | Installation au niveau fichier (extraction, placement) |
| **Haut niveau** | `apt` | `yum` / `dnf` | Gestion des dépendances, accès aux dépôts, MAJ automatique |

> [!example] Analogie
> `dpkg` = le maçon qui pose les briques
> `apt` = l'architecte qui commande les matériaux et coordonne les travaux

### Les dépôts

> [!info] Qu'est-ce qu'un dépôt ?
> Les **dépôts** (repositories) sont des serveurs ou collections de logiciels accessibles via les gestionnaires de paquets. Ils assurent la disponibilité des logiciels, des MAJ et des informations sur les dépendances.

**Processus d'accès au dépôt :**
```
Utilisateur → Gestionnaire de paquets → Dépôt (métadonnées + paquet)
                     ↓
           Résolution des dépendances
                     ↓
           Téléchargement et installation
```

### Le fichier sources.list

> [!important] /etc/apt/sources.list — Fichier critique pour APT
> Ce fichier est **essentiel** : il spécifie les dépôts depuis lesquels APT va chercher les paquets à installer ou mettre à jour.
>
> **Emplacement :** `/etc/apt/sources.list`

**Structure d'une ligne du sources.list :**

```
<type> <URL> <distribution> <composantes>
```

| Champ | Valeurs possibles | Description |
|-------|------------------|-------------|
| **Type** | `deb` ou `deb-src` | Binaires ou sources |
| **URL** | `http://deb.debian.org/...` | Adresse du dépôt |
| **Distribution** | `stable`, `bullseye`, `jammy`… | Version de la distro |
| **Composantes** | `main`, `contrib`, `non-free`, `restricted`… | Catégorie de paquets |

**Signification des composantes :**

| Composante | Description |
|-----------|-------------|
| `main` | Logiciels libres respectant les DFSG (Debian Free Software Guidelines) |
| `contrib` | Logiciels libres mais dépendant de paquets non-libres |
| `non-free` | Logiciels propriétaires |
| `restricted` | Logiciels propriétaires (Ubuntu) |
| `universe` | Logiciels libres maintenus par la communauté |
| `multiverse` | Logiciels non libres selon la définition Ubuntu |

> [!example] Exemple de sources.list — Ubuntu 22.04 (Jammy)
> ```
> deb http://fr.archive.ubuntu.com/ubuntu/ jammy main restricted
> deb http://fr.archive.ubuntu.com/ubuntu/ jammy-updates multiverse
> deb http://fr.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
> ```
> - `jammy` : Ubuntu 22.04
> - `jammy-updates` : MAJ publiées après la sortie
> - `jammy-backports` : versions plus récentes de certains logiciels

> [!example] Exemple de sources.list — Debian stable
> ```bash
> # Dépôt principal
> deb http://deb.debian.org/debian/ stable main
> # Contributions et non-libre
> deb http://deb.debian.org/debian/ stable contrib non-free
> # MAJ de sécurité
> deb http://security.debian.org/debian-security stable-security main contrib non-free
> ```

---

## Commandes de base

### Commandes dpkg

> [!info] dpkg — Gestion bas niveau (fichiers .deb)

```bash
# Installation d'un paquet .deb local
sudo dpkg -i dropbox_2022.12.05_amd64.deb

# Désinstallation d'un paquet
sudo dpkg -r dropbox_2022.12.05_amd64.deb

# Lister tous les paquets installés (format long)
dpkg -l

# Lister tous les paquets installés (format court)
dpkg --get-selections

# Vérifier si un paquet est installé
dpkg -s nano

# Lister les fichiers installés par un paquet
dpkg -L nano
```

**Lecture de `dpkg -l` :**
```
||/ Nom        Version    Architecture  Description
ii  nano       6.2-1      amd64         ...
^^
|+-- État (i = installé)
+--- Souhait (i = install)
```

### Commandes apt

> [!important] apt — Commandes essentielles à maîtriser

```bash
# Mettre à jour la liste des paquets disponibles (toujours faire avant d'installer)
sudo apt update

# Installer les MAJ disponibles
sudo apt upgrade

# Raccourci — update + upgrade
sudo apt update && sudo apt upgrade

# Installer un paquet
sudo apt install curl

# Désinstaller un paquet (conserve les fichiers de configuration)
sudo apt remove curl

# Désinstaller complètement (supprime aussi les fichiers de config)
sudo apt purge curl

# Supprimer les paquets de dépendances orphelins
sudo apt autoremove

# Désinstallation complète en une commande
sudo apt autoremove --purge curl

# Rechercher un paquet
apt search baobab

# Afficher les infos d'un paquet
apt show nano
```

> [!tip] Ordre des opérations pour installer un logiciel
> 1. `sudo apt update` — toujours mettre à jour les listes en premier
> 2. `sudo apt install <paquet>` — installation

> [!warning] Différence apt search vs apt-cache search
> ```bash
> # apt-cache search (ancienne méthode)
> apt-cache search baobab
> # → Résultat simple
>
> # apt search (nouvelle méthode)
> apt search baobab
> # → Résultat plus riche avec statut d'installation [installé]
> ```

---

## Sécurité

> [!important] Vérification GPG des dépôts
> APT utilise des **signatures GPG** (GNU Privacy Guard) pour vérifier l'authenticité des dépôts et des paquets. Chaque dépôt possède une clé GPG que le système doit reconnaître.

**Exemple : ajout de la clé GPG de Google Chrome**
```bash
sudo wget -O- https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /usr/share/keyrings/google-chrome.gpg
```

> [!warning] Sécurité des sources
> - N'ajouter que des dépôts **officiels et de confiance** dans `sources.list`
> - Toujours vérifier la présence d'une clé GPG avant d'ajouter un dépôt tiers
> - Un dépôt non sécurisé peut compromettre l'intégralité du système

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Architecture
- **2 niveaux** : bas niveau (dpkg/rpm) et haut niveau (apt/yum)
- APT = surcouche de dpkg avec gestion des dépendances et des dépôts
- Liste des dépôts dans `/etc/apt/sources.list`

### Commandes incontournables
- `apt update` → mise à jour des listes (avant toute action)
- `apt install` / `apt remove` / `apt purge` → gestion des paquets
- `apt upgrade` → installation des MAJ disponibles
- `dpkg -l` → liste des paquets installés
- `dpkg -s <paquet>` → statut d'un paquet

### Gestionnaires par distribution (à mémoriser)
- **Debian/Ubuntu/Mint/Kali** → **APT** (`.deb`)
- **Red Hat/CentOS** → **YUM** (`.rpm`)
- **Fedora** → **DNF** (`.rpm`)
- **Arch/Manjaro** → **Pacman** (`.pkg`)
- **OpenSUSE** → **Zypper** (`.rpm`)

### Sécurité
- GPG pour l'authenticité des paquets
- Ne jamais ajouter de dépôts non fiables

> [!warning] Pièges à éviter
> - Ne **jamais** installer de paquet sans faire `apt update` avant
> - Ne pas confondre `apt remove` (conserve la config) et `apt purge` (supprime tout)
> - Ne pas ajouter des dépôts tiers sans vérifier leur clé GPG

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|-----------|
| **APT** | Advanced Package Tool — gestionnaire haut niveau pour Debian et dérivés |
| **dpkg** | Debian Package — gestionnaire bas niveau, base d'APT |
| **RPM** | Red Hat Package Manager — format de paquet et outil bas niveau Red Hat |
| **YUM** | Yellowdog Updater Modified — gestionnaire haut niveau Red Hat (remplacé par DNF) |
| **DNF** | Dandified YUM — successeur de YUM pour Fedora/Red Hat moderne |
| **Pacman** | Gestionnaire de paquets d'Arch Linux |
| **Zypper** | Gestionnaire de paquets d'OpenSUSE |
| **Paquet** | Archive contenant binaires, bibliothèques, scripts et métadonnées d'un logiciel |
| **Dépendance** | Pré-requis logiciel — autre paquet nécessaire au bon fonctionnement |
| **Dépôt** | Serveur centralisé hébergeant des paquets vérifiés et leurs métadonnées |
| **sources.list** | Fichier `/etc/apt/sources.list` listant les dépôts APT à utiliser |
| **GPG** | GNU Privacy Guard — système de signature cryptographique pour valider les paquets |
| **DFSG** | Debian Free Software Guidelines — critères définissant un logiciel libre selon Debian |
| **Snap** | Gestionnaire universel avec isolation (Canonical/Ubuntu) |
| **Flatpak** | Gestionnaire universel avec isolation (multi-distribution) |
| **AppImage** | Format d'application portable, auto-contenue, sans installation |
| **Aptitude** | Interface semi-graphique pour APT en mode console |
| **Synaptic** | Interface graphique (GUI) pour APT |
