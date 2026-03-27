# Conteneurs et Docker

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Conteneurs et Docker - Isolation des processus et orchestration

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#L'idée générale|L'idée générale]]
   - [[#Processus et système de fichiers|Processus et système de fichiers]]
   - [[#Le cloisonnement|Le cloisonnement]]
   - [[#Qu'est-ce qu'un conteneur|Qu'est-ce qu'un conteneur]]
   - [[#Conteneur vs Machine Virtuelle|Conteneur vs Machine Virtuelle]]
2. [[#Le point de vue développeur|Le point de vue développeur]]
   - [[#Problème initial côté dev|Problème initial côté dev]]
   - [[#Diversité des environnements|Diversité des environnements]]
   - [[#Ce qu'apporte la conteneurisation|Ce qu'apporte la conteneurisation aux développeurs]]
3. [[#Le point de vue infrastructure|Le point de vue infrastructure]]
   - [[#Problème initial côté infra|Problème initial côté infra]]
   - [[#Sans les conteneurs|Sans les conteneurs]]
   - [[#Rôle du TSSR|Rôle et missions d'un TSSR]]
   - [[#Avantages pour l'infrastructure|Avantages pour l'infrastructure]]
4. [[#Les solutions de conteneurisation|Les solutions de conteneurisation]]
   - [[#Conteneurisation bas niveau|Conteneurisation bas niveau]]
   - [[#Conteneurisation système (LXC)|Conteneurisation système]]
   - [[#Conteneurisation applicative (Docker)|Conteneurisation applicative]]
   - [[#Orchestration de conteneurs|Orchestration de conteneurs]]
5. [[#Docker en détail|Docker en détail]]
   - [[#Concepts fondamentaux|Concepts fondamentaux]]
   - [[#Les images Docker|Les images]]
   - [[#Les conteneurs Docker|Les conteneurs]]
   - [[#Les dépôts|Les dépôts]]
   - [[#Les volumes|Les volumes]]
   - [[#Les réseaux|Les réseaux]]
   - [[#Considérations générales|Considérations générales]]
6. [[#Docker en pratique|Docker en pratique]]
   - [[#Composants Docker|Composants]]
   - [[#Outils supplémentaires|Outils supplémentaires]]
   - [[#Docker CLI|Docker CLI]]
   - [[#Gestion des images|Gestion des images]]
   - [[#Lancer un conteneur|Lancer un conteneur]]
   - [[#Gestion des conteneurs|Gestion des conteneurs]]
   - [[#Gestion des volumes Docker|Gestion des volumes]]
   - [[#Gestion des réseaux Docker|Gestion des réseaux]]
   - [[#Gestion globale|Gestion globale]]
   - [[#Le Dockerfile|Le Dockerfile]]
   - [[#Générer une image|Générer une image]]
   - [[#Cas d'usage|Cas d'usage]]
7. [[#Points clés à retenir|Points clés à retenir]]
8. [[#Glossaire technique|Glossaire technique]]
9. [[#Ressources et références|Ressources et références]]

---

## Introduction

> [!abstract] Vue d'ensemble
> La **conteneurisation** est une technologie d'isolation des processus qui permet de déployer et d'exécuter des applications dans des environnements cloisonnés et reproductibles. En tant que TSSR, tu dois comprendre comment les conteneurs répondent aux problématiques modernes de déploiement, de sécurité et de gestion des infrastructures.

### Pourquoi étudier les conteneurs et Docker ?

En tant que **TSSR**, tu dois :
- Comprendre l'isolation des processus et ses enjeux de sécurité
- Maîtriser Docker pour déployer des applications modernes
- Savoir gérer des infrastructures conteneurisées
- Faire le lien entre développeurs et infrastructure (approche DevOps)
- Optimiser l'utilisation des ressources serveur

---

## L'idée générale

> [!important] Contexte moderne
> Les **infrastructures modernes** doivent déployer et maintenir des applications complexes qui reposent sur :
> - Des **processus** multiples
> - Des **dépendances** spécifiques (bibliothèques, frameworks)
> - Des **configurations** particulières

> [!success] Solution
> La **conteneurisation** est une réponse système aux problématiques de **déploiement applicatif**.

---

## Processus et système de fichiers

> [!question] Un moment de réflexion
> Sur un OS, à quoi a accès un processus ?

### Ressources accessibles par un processus

Un processus standard peut accéder à :
- **Fichiers** (système, configuration, données)
- **Bibliothèques** partagées (librairies système)
- **Interfaces réseau** (connexions, ports)
- **Ressources CPU et mémoire** (potentiellement toutes)

> [!warning] Problème fondamental
> **Par défaut, un processus voit PLUS que ce qui lui est strictement nécessaire.**
> 
> Cela pose des problèmes de :
> - **Sécurité** : accès à des ressources sensibles
> - **Stabilité** : un processus peut affecter le système complet
> - **Isolation** : difficile de cloisonner les applications

### Composition du système de fichiers

> [!info] Contenu du FS (File System)
> Le système de fichiers d'un OS contient :
> - **Fichiers système** (`/bin`, `/sbin`, `/lib`, etc.)
> - **Bibliothèques partagées** (`.so` sous Linux, `.dll` sous Windows)
> - **Données applicatives** (configurations, bases de données)

> [!note] Insuffisance des permissions classiques
> Les mécanismes classiques de permissions (UNIX : `rwx`, ACL) sont nécessaires mais **souvent insuffisants** pour cloisonner finement les processus.

> [!important] Principe d'administration système
> En tant qu'administrateur système, on doit **réduire la surface d'attaque** en limitant strictement l'accès des processus aux ressources nécessaires.

---

## Le cloisonnement

> [!info] Une solution : le cloisonnement
> C'est une **vieille méthode** qui a évolué vers la conteneurisation moderne.

### Mécanismes historiques

| Système | Mécanisme | Description |
|---------|-----------|-------------|
| **Linux** | `chroot` | Change Root - modifie la racine du système de fichiers pour un processus |
| **BSD** | `jails` | Prisons - environnements isolés plus avancés que chroot |

### Objectifs du cloisonnement

> [!success] Double objectif
> 1. **Limiter ce qu'un processus peut voir** (principe du moindre privilège)
> 2. **Limiter l'impact d'un incident** ou d'une brèche de sécurité

> [!tip] Fondation de la conteneurisation
> Ces mécanismes constituent les **bases de la conteneurisation** moderne.

---

## Qu'est-ce qu'un conteneur

> [!quote] Définition
> Un **conteneur** est un ensemble de processus s'exécutant dans un **environnement isolé**.

### Caractéristiques d'un conteneur

```
┌─────────────────────────────────┐
│       CONTENEUR                 │
├─────────────────────────────────┤
│ • Propre système de fichiers   │
│ • Ressources limitées (CPU/RAM)│
│ • S'appuie sur le noyau hôte   │
│ • Isolation réseau              │
│ • Namespace dédié               │
└─────────────────────────────────┘
```

> [!important] Points clés
> - **Dispose de son propre FS** (système de fichiers isolé)
> - **Ressources limitées** (quotas CPU, RAM, disque)
> - **S'appuie sur le noyau du système hôte** (pas de noyau dédié)

---

## La conteneurisation

> [!info] Concept général
> L'idée générale de la conteneurisation :

### Principes fondamentaux

1. **Cloisonner un processus** (et ses processus fils)
2. **Déclarer explicitement** les ressources accessibles et l'environnement d'exécution
3. **Créer des instances séparées** reproductibles et indépendantes

> [!note] Extension moderne
> La conteneurisation est une **extension moderne** du concept de **prison logicielle** (jail, chroot).

### Avantages de la conteneurisation

- ✅ **Isolation forte** entre les applications
- ✅ **Reproductibilité** des environnements
- ✅ **Portabilité** entre différents systèmes
- ✅ **Légèreté** comparée aux machines virtuelles
- ✅ **Déploiement rapide**

---

## Conteneur vs Machine Virtuelle

> [!important] Différence fondamentale
> Même si à l'utilisation les conteneurs **ressemblent** à des machines virtuelles (VM), c'est en fait **très différent** techniquement.

### Machine Virtuelle (VM)

> [!info] Fonctionnement VM
> Pour avoir une VM, un **hyperviseur** propose une couche d'abstraction qui :
> - **Reproduit le comportement d'une machine réelle**
> - Permet l'**installation d'un OS complet**
> - Chaque VM a son **propre noyau**

### Conteneur

> [!info] Fonctionnement conteneur
> Avec les conteneurs :
> - Il y a **un seul noyau** qui tourne (celui de l'hôte)
> - On s'appuie sur des **fonctionnalités du noyau** (namespaces, cgroups)
> - On **isole des processus** dans des conteneurs
> - **Moins consommateur** en ressources (CPU, mémoire, disque)

### Tableau comparatif

| Critère | Machine Virtuelle | Conteneur |
|---------|-------------------|-----------|
| **Noyau** | Noyau dédié par VM | Noyau partagé (hôte) |
| **OS** | OS complet | Pas d'OS complet |
| **Démarrage** | Minutes | Secondes |
| **Taille** | Go (gigaoctets) | Mo (mégaoctets) |
| **Isolation** | Forte (matérielle) | Moyenne (logicielle) |
| **Performance** | Overhead hyperviseur | Native (quasi) |
| **Ressources** | Lourdes | Légères |
| **Densité** | 10-50 VM/serveur | 100-1000 conteneurs/serveur |

### Schéma des différences

```
┌─────────────────────────────────────────────────────────┐
│              MACHINES VIRTUELLES                        │
├─────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C  │                          │
│  Libs   │  Libs   │  Libs   │                          │
│  OS 1   │  OS 2   │  OS 3   │                          │
├─────────┴─────────┴─────────┤                          │
│     Hyperviseur (Type 1/2)   │                          │
├──────────────────────────────┤                          │
│     OS Hôte (si Type 2)      │                          │
├──────────────────────────────┤                          │
│         Matériel             │                          │
└──────────────────────────────┘                          │

┌─────────────────────────────────────────────────────────┐
│                   CONTENEURS                            │
├─────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C  │                          │
│  Libs   │  Libs   │  Libs   │                          │
├─────────┴─────────┴─────────┤                          │
│     Docker Engine            │                          │
├──────────────────────────────┤                          │
│     OS Hôte (noyau partagé)  │                          │
├──────────────────────────────┤                          │
│         Matériel             │                          │
└──────────────────────────────┘                          │
```

> [!tip] Quand utiliser quoi ?
> - **VM** : Isolation forte, OS différents, sécurité maximale, ressources dédiées
> - **Conteneur** : Déploiement rapide, densité élevée, microservices, même OS

---

## Le point de vue développeur

> [!abstract] Problématiques du développement moderne
> La conteneurisation répond à des problèmes concrets rencontrés par les développeurs au quotidien.

---

## Problème initial côté dev

> [!warning] Complexité des dépendances
> Pour qu'une **application fonctionne**, elle a besoin de :
> - **Versions spécifiques** de langages (Node.js, Python, Java, etc.)
> - **Bibliothèques** et frameworks
> - **Variables d'environnement**
> - **Configurations système**
> - **Services externes** (bases de données, cache, etc.)

### L'environnement d'exécution varie

> [!info] Multiplicité des environnements
> L'environnement peut être **différent** selon :
> - **Machine de développement** (poste du développeur)
> - **Serveur de test** (environnement QA)
> - **Serveur de pré-production** (staging)
> - **Serveur de production** (prod)
> - **Poste d'un autre développeur**

---

## Diversité des environnements

> [!warning] Sources de problèmes
> La diversité crée des difficultés majeures :

### Variations entre environnements

- **Dev, test, pré-prod, prod** : configurations différentes
- **OS différents** : Linux, Windows, macOS
- **Versions de bibliothèques** : conflits de dépendances
- **Configurations réseau** : ports, proxy, pare-feu
- **Permissions** : utilisateurs, droits d'accès différents

### Conséquences

> [!warning] Impact sur le développement
> - ❌ **Tests et debug compliqués** et difficiles à mettre en œuvre
> - ❌ **Déploiement à risque** (instabilité, comportements imprévisibles)
> - ❌ **Perte de temps** en configuration et résolution de problèmes
> - ❌ **Frustration** des équipes

### La phrase culte

> [!example] Situation typique
> **Développeur** : "Ça marche pas sur ton PC ?"
> 
> **Autre développeur** : "C'est bizarre, ça marche sur ma machine..."
> 
> 😤 **Frustration garantie !**

---

## Ce qu'apporte la conteneurisation aux développeurs

> [!success] Solution miracle
> Avec la conteneurisation, l'application est **livrée avec** :
> - **Ses propres dépendances** (bibliothèques, frameworks)
> - **Sa propre configuration** (variables, paramètres)
> - **Son environnement d'exécution complet**

> [!important] Principe fondamental
> L'application a le **même comportement partout**, quel que soit :
> - La machine
> - L'environnement
> - L'OS sous-jacent (tant que Docker est installé)

### Avantages pour les développeurs

| Avant (sans conteneur) | Après (avec conteneur) |
|------------------------|------------------------|
| Configuration manuelle longue | Lancement en une commande |
| "Ça marche sur ma machine" | "Ça marche partout" |
| Conflits de versions | Isolation complète |
| Documentation complexe | Dockerfile = documentation |
| Onboarding difficile | Nouveau dev opérationnel en minutes |

> [!tip] DevOps et CI/CD
> Les conteneurs facilitent l'intégration continue (CI) et le déploiement continu (CD) en garantissant la reproductibilité.

---

## Le point de vue infrastructure

> [!abstract] Enjeux pour l'administrateur système
> La conteneurisation répond également aux défis de gestion d'infrastructure moderne.

---

## Problème initial côté infra

> [!warning] Défis de l'hébergement multiple
> L'infrastructure doit héberger **plusieurs applications** (ou services) avec les impératifs suivants :

### Exigences

1. **Hébergement multiple** : plusieurs services sur les mêmes serveurs
2. **Garanties de sécurité** : isolation entre les applications
3. **Garanties d'accès aux ressources** : QoS, pas de famine
4. **Optimisation des ressources** : maximiser l'utilisation serveur

---

## Sans les conteneurs

> [!warning] Problèmes classiques
> Sans conteneurisation, on fait face à :

### Difficultés de gestion

- **Services installés directement** sur l'OS (liés au système)
- **Dépendances partagées** entre les services (conflits potentiels)
- **Conflits de versions** (OS, bibliothèques, services)
- **Difficultés d'isolation** d'un service compromis (propagation d'attaque)
- **Mises à jour risquées** (impact global sur le système)
- **Déploiement lent** (installation manuelle, configuration)

> [!example] Exemple de conflit
> Deux applications nécessitent des versions différentes de Python :
> - App A : Python 3.8
> - App B : Python 3.11
> 
> Sans conteneur = **problème insoluble** sans solutions complexes (virtualenv, pyenv, etc.)

---

## Rôle et missions d'un TSSR

> [!important] Responsabilités du TSSR
> En tant que TSSR, tes missions incluent :

### Missions principales

1. **Réduire la surface d'attaque**
   - Limiter les services exposés
   - Cloisonner les applications
   - Appliquer le principe du moindre privilège

2. **Gérer les ressources** (CPU, RAM, espace disque)
   - Optimisation de l'utilisation
   - Prévention de la surcharge
   - Garanties de performance

3. **Déploiement fiable et reproductible**
   - Minimiser les erreurs humaines
   - Standardiser les processus
   - Documenter les configurations

4. **Mises à jour sans problème**
   - Déploiements sans interruption (rolling update)
   - Rollback rapide en cas de problème
   - Tests avant production

> [!success] La conteneurisation répond à TOUTES ces missions !

---

## Avantages pour l'infrastructure

> [!success] Ce qu'apporte la conteneurisation aux gestionnaires d'infra

### Bénéfices majeurs

| Avantage | Description | Impact |
|----------|-------------|--------|
| **Cloisonnement strict** | Isolation des processus | 🔒 Sécurité renforcée |
| **Déploiements rapides** | Clonage/lancement en secondes | ⚡ Agilité |
| **Indépendant de l'OS** | Même conteneur sur différents OS | 🌍 Portabilité |
| **Pas de trace disque** | Suppression propre | 🧹 Maintenance simplifiée |
| **Scalabilité** | Ajout/retrait facile de conteneurs | 📈 Élasticité |
| **Densité élevée** | Nombreux conteneurs par serveur | 💰 Économies |

> [!success] Alternative aux VM
> Les conteneurs constituent une **bonne alternative aux VM** pour de nombreux cas d'usage, avec :
> - Démarrage quasi-instantané
> - Consommation mémoire réduite
> - Meilleure densité (plus de conteneurs que de VM par serveur)

> [!warning] Attention
> Les conteneurs ne remplacent pas totalement les VM. Ils répondent à des besoins différents et sont souvent **complémentaires**.

---

## Les solutions de conteneurisation

> [!abstract] Panorama des solutions
> La conteneurisation est un **concept**, pas un produit. Docker est une implémentation parmi d'autres.

> [!info] Différentes approches
> Les approches possibles de la conteneurisation :
> 1. **Conteneurisation bas niveau** (outils système)
> 2. **Conteneurisation système** (LXC/LXD)
> 3. **Conteneurisation applicative** (Docker, Podman)
> 4. **Orchestration de conteneurs** (Kubernetes, Docker Swarm)

---

## Conteneurisation bas niveau

> [!info] Approche manuelle
> On peut cloisonner des ressources avec quelques commandes système de base.

### Outils de base

| Outil | Système | Description |
|-------|---------|-------------|
| **chroot** | Linux/Unix | Change Root - isole le système de fichiers |
| **namespaces** | Linux | Isole différentes ressources système (réseau, PID, IPC, etc.) |
| **cgroups** | Linux | Limite et contrôle les ressources (CPU, RAM, I/O) |
| **jails** | BSD | Environnements d'exécution isolés |

> [!note] Fondations
> Ces mécanismes sont les **briques de base** utilisées par Docker et autres solutions de conteneurisation.

### Namespaces Linux

> [!example] Types de namespaces
> - **PID** : Isolation des processus
> - **Network** : Stack réseau isolée
> - **Mount** : Points de montage isolés
> - **UTS** : Hostname isolé
> - **IPC** : Inter-Process Communication isolée
> - **User** : Utilisateurs et groupes isolés
> - **Cgroup** : Contrôle des ressources isolé

### Cgroups (Control Groups)

> [!example] Limitations possibles
> - **CPU** : limiter le % CPU utilisable
> - **Memory** : limiter la RAM allouée
> - **Disk I/O** : contrôler les accès disque
> - **Network** : limiter la bande passante

> [!warning] Complexité
> L'utilisation directe de ces outils est **complexe** et **sujette à erreurs**. D'où l'intérêt des solutions de plus haut niveau.

---

## Conteneurisation système (LXC)

> [!info] LXC : Linux Containers
> **LXC** permet de créer des conteneurs système "ressemblant" à une VM, mais **sans hyperviseur** !

### Principe

- On dispose d'une **machine Linux complète**
- Utilise **namespaces, cgroups** et le **noyau Linux de l'hôte**
- Mais **sans la surcharge** d'un hyperviseur

### Contenu d'un conteneur LXC

> [!note] Composition
> Un conteneur LXC contient :
> - Un **système d'init** (systemd, OpenRC)
> - **Plusieurs services** (SSH, Apache, MySQL, etc.)
> - **Plusieurs utilisateurs**
> - Une **arborescence Linux complète** (`/bin`, `/etc`, `/var`, etc.)

### Public cible

> [!success] Parfait pour
> Approche **Infra / Système / DevOps / Cybersécurité**
> 
> Idéal quand on veut :
> - Une machine Linux légère
> - Plusieurs services dans le même conteneur
> - Un environnement proche d'une VM
> - Gérer des utilisateurs, permissions classiques

### LXD : la couche de gestion

> [!info] Qu'est-ce que LXD ?
> **LXD** est la **couche de gestion** d'un conteneur LXC.
> 
> Avec LXD, on gère :
> - **Images** (templates de conteneurs)
> - **Réseaux** (bridges, VLAN)
> - **Stockage** (pools, volumes)
> - **Snapshots** et migrations
> - **API REST** pour l'automatisation

> [!tip] Proxmox et LXC
> **Proxmox VE** utilise **LXC** pour ses conteneurs ! C'est ce que tu utilises dans ton homelab.

---

## Conteneurisation applicative (Docker)

> [!info] Solution Docker
> **Docker** se concentre sur la conteneurisation **applicative**.

### Caractéristiques

- **Isolation application/service** (un service par conteneur)
- **Un processus principal** par conteneur
- **Très léger** car pas d'OS complet (pas d'init, pas de systemd)
- **Orienté microservices**

### Philosophie Docker

> [!important] Un conteneur = Un service
> Docker recommande :
> - **1 conteneur = 1 application/service**
> - Pas d'init system (systemd)
> - Processus principal en PID 1
> - Conteneur éphémère (immutable infrastructure)

### Différence avec LXC

| Critère | LXC | Docker |
|---------|-----|--------|
| **Type** | Conteneur système | Conteneur applicatif |
| **Contenu** | OS complet | Application + dépendances |
| **Init** | systemd/OpenRC | Pas d'init |
| **Services** | Multiples | Un principal |
| **Usage** | Remplacer VM | Déployer applications |
| **Taille** | Centaines de Mo | Dizaines de Mo |

> [!tip] Complémentarité
> LXC et Docker ne sont pas concurrents mais **complémentaires** :
> - **LXC** : Infrastructure as Code (conteneurs système)
> - **Docker** : Application deployment (conteneurs applicatifs)

---

## Orchestration de conteneurs

> [!info] Au-delà du conteneur unique
> L'orchestration gère **plusieurs conteneurs** sur **plusieurs machines**.

### Problématique

> [!warning] Complexité à grande échelle
> **De base** : 1 conteneur sur 1 machine pour 1 application/service
> 
> **Si on augmente** l'un des éléments, la gestion se complique :
> - 10 conteneurs sur 1 machine
> - 1 conteneur sur 10 machines
> - 100 conteneurs sur 10 machines

### Fonctions de l'orchestration

> [!important] Capacités d'orchestration
> L'orchestration permet :
> - **Gestion de plusieurs conteneurs** simultanés
> - **Répartition de charge** (load balancing)
> - **Démarrage, arrêt** (gestion du cycle de vie)
> - **Scalabilité horizontale** (ajout de conteneurs)
> - **Scalabilité verticale** (ajout de ressources - non dynamique)
> - **Auto-healing** (redémarrage automatique)
> - **Service discovery** (découverte des services)
> - **Rolling updates** (mises à jour progressives)

### Solutions d'orchestration

| Solution | Description | Complexité |
|----------|-------------|------------|
| **Docker Compose** | Multi-conteneurs sur 1 machine | ⭐ Facile |
| **Docker Swarm** | Cluster Docker natif | ⭐⭐ Moyen |
| **Kubernetes (K8s)** | Standard de l'industrie | ⭐⭐⭐⭐⭐ Complexe |
| **Nomad** | Alternative HashiCorp | ⭐⭐⭐ Moyen |

> [!warning] Complexité Kubernetes
> **Kubernetes** est extrêmement puissant mais très complexe. Ne pas se lancer sans formation appropriée.

---

## Docker en détail

> [!abstract] Focus sur Docker
> Docker est devenu le **standard de facto** de la conteneurisation applicative.

---

## Concepts fondamentaux

> [!quote] Idée générale de Docker
> L'idée générale de Docker est de :
> - **Construire** (build) ou **récupérer** (pull) des **images**
> - Les stocker dans un **dépôt** (repository)
> - Les **exécuter** (run) sous la forme de **conteneurs isolés**

### Communication entre conteneurs

> [!info] Mécanismes de communication
> Pour permettre aux conteneurs de communiquer, Docker propose :
> - **Volumes** : partage de répertoires entre conteneurs (et avec l'hôte)
> - **Réseaux virtuels** : communication réseau entre conteneurs

### Schéma des concepts

```
┌─────────────────────────────────────────────────┐
│              WORKFLOW DOCKER                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  [Dockerfile] ──build──> [Image]               │
│                             │                   │
│                             │ run               │
│                             ▼                   │
│                        [Conteneur 1]            │
│                        [Conteneur 2]            │
│                        [Conteneur 3]            │
│                             │                   │
│                             │                   │
│  [Docker Hub] ◄──pull/push──┘                  │
│                                                 │
│  [Volume] ◄─────── partagé entre conteneurs    │
│  [Network] ◄────── communication inter-conteneur│
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Les images Docker

> [!quote] Définition d'une image
> Avec Docker, une **image** est l'ensemble des **dossiers et fichiers** qui constitue le système de fichiers racine d'un conteneur, ainsi que les **informations de configuration** indiquant notamment quel programme doit être exécuté.

### Origine des images

> [!info] Deux sources
> Une image peut être :
> 1. **Récupérée** dans un dépôt (repository) via `docker pull`
> 2. **Construite** (build) à l'aide d'un fichier **Dockerfile**

### Structure en couches

> [!important] Architecture en layers
> Les images Docker sont construites en **couches** (layers) :
> - Chaque couche peut être **conservée en cache**
> - Les couches sont **partagées** entre différentes images
> - **Économie** d'espace disque et de temps de build

```
┌─────────────────────────────┐
│   Layer 4: Application      │
├─────────────────────────────┤
│   Layer 3: Dépendances      │
├─────────────────────────────┤
│   Layer 2: Packages         │
├─────────────────────────────┤
│   Layer 1: OS de base       │
└─────────────────────────────┘
```

> [!example] Exemple
> Si 3 images utilisent toutes Ubuntu 22.04 comme base, cette couche n'est stockée qu'une seule fois.

### Analogie

> [!tip] Comparaison
> **Une image équivaut à un programme avec toutes ses dépendances.**
> 
> C'est le "template" ou "modèle" à partir duquel on crée des conteneurs.

---

## Les conteneurs Docker

> [!quote] Définition
> Exécuter du code avec Docker consiste à créer un **conteneur** à partir d'une **image**.

### Relation image ↔ conteneur

> [!important] Analogie fondamentale
> **Un conteneur est à l'image ce que le processus est au programme.**
> 
> - **Image** = Template statique (comparable à un fichier exécutable)
> - **Conteneur** = Instance en exécution (comparable à un processus)

### Multiplicité

> [!info] Plusieurs instances possibles
> **Plusieurs conteneurs** peuvent être créés à partir de la **même image** pour être exécutés :
> - **Simultanément** (parallèle)
> - **Non simultanément** (séquentiel)

> [!example] Exemple pratique
> À partir de l'image `nginx:latest`, on peut lancer :
> - 1 conteneur pour le site web de production
> - 1 conteneur pour le site de staging
> - 1 conteneur pour les tests
> 
> Chacun avec sa propre configuration, ses propres données.

### Isolation et modifications

> [!note] Isolation des modifications
> - Les processus d'un conteneur ne peuvent accéder qu'aux **fichiers de son image**
> - Les **modifications faites** restent dans le conteneur
> - Elles ne sont **pas répercutées** sur l'image
> - Suppression du conteneur = perte des modifications (sauf volumes)

> [!warning] Principe d'immutabilité
> Les conteneurs sont **éphémères** : on les supprime et recrée plutôt que de les modifier en place.

---

## Les dépôts

> [!info] Partage d'images
> Si Docker permet la construction d'images via Dockerfile, il est courant de partager ses images via des **dépôts** (registry/repository).

### Docker Hub

> [!quote] Dépôt officiel
> **Docker Hub** est un service web géré par Docker Inc. qui héberge :
> - Des **dépôts publics** (gratuits)
> - Des **dépôts privés** (payants ou limités)

> [!success] Richesse du catalogue
> Beaucoup d'applications courantes proposent sur Docker Hub des images **utilisables directement** :
> - Serveurs web (nginx, Apache)
> - Bases de données (MySQL, PostgreSQL, MongoDB)
> - Langages (Python, Node.js, Java)
> - Outils (Redis, Elasticsearch, etc.)

> [!example] Adresses Docker Hub
> Format : `registry/repository:tag`
> - `nginx:latest` (image officielle nginx, dernière version)
> - `ubuntu:22.04` (Ubuntu 22.04 LTS)
> - `mysql:8.0` (MySQL version 8.0)

### Dépôt personnel (Private Registry)

> [!info] Héberger son propre dépôt
> Il est aussi possible d'héberger son **propre dépôt** (on-premise ou cloud).

**Avantages** :
- ✅ **Confidentialité** : images privées
- ✅ **Performance** : accès rapide (réseau local)
- ✅ **Contrôle** : gestion complète
- ✅ **Sécurité** : pas d'exposition externe

**Solutions de registry privé** :
- Docker Registry (solution officielle)
- Harbor (CNCF - fonctionnalités avancées)
- GitLab Container Registry
- GitHub Container Registry
- Cloud providers (AWS ECR, Azure ACR, Google GCR)

### Actions sur les dépôts

> [!note] Opérations
> On peut :
> - **Récupérer** (pull) des images depuis un dépôt
> - **Publier** (push) des images vers un dépôt

```bash
# Pull : récupérer une image
docker pull nginx:latest

# Push : publier une image (nécessite authentification)
docker tag mon-app:latest registry.exemple.com/mon-app:latest
docker push registry.exemple.com/mon-app:latest
```

---

## Les volumes

> [!quote] Stockage persistant
> Docker propose la création de **volumes** qui permettent de partager des fichiers entre le système hôte et les conteneurs.

### Principe des volumes

> [!important] Idée générale
> Les volumes sont des **espaces de stockage persistants** :
> - **Gérés par Docker**
> - **Partagés** entre conteneurs
> - **Persistent** après la destruction du conteneur
> - Permettent le **stockage de données persistantes**

### Cas d'usage

> [!example] Utilisations typiques
> - **Bases de données** : stocker les données en dehors du conteneur
> - **Logs** : centraliser les logs applicatifs
> - **Configuration** : injecter des fichiers de config
> - **Upload** : stocker les fichiers uploadés par les utilisateurs
> - **Développement** : hot-reload du code

### Types de montage

| Type | Description | Usage |
|------|-------------|-------|
| **Volume** | Géré par Docker | Production (recommandé) |
| **Bind mount** | Dossier hôte monté | Développement |
| **tmpfs** | En mémoire (RAM) | Données temporaires |

### Volumes vs Bind mounts

> [!info] Volumes (recommandés)
> ```bash
> docker volume create mon-volume
> docker run -v mon-volume:/data mon-image
> ```
> - Gérés par Docker
> - Indépendants du filesystem hôte
> - Sauvegardes facilitées
> - Portables

> [!info] Bind mounts
> ```bash
> docker run -v /chemin/hote:/chemin/conteneur mon-image
> ```
> - Montage direct d'un dossier de l'hôte
> - Utile en développement
> - Dépendant du filesystem hôte

> [!warning] Persistence
> Sans volume, les données d'un conteneur sont **perdues** à sa suppression !

---

## Les réseaux

> [!info] Communication réseau
> Docker permet la **communication réseau** entre conteneurs et avec l'extérieur.

### Publication de ports

> [!important] Port mapping
> Au lancement d'un conteneur, Docker peut **lier un port du système hôte** avec un **port du conteneur**.
> 
> Cela permet facilement l'utilisation de conteneurs pour exécuter des **serveurs**.

```bash
# Exposer le port 80 du conteneur sur le port 8080 de l'hôte
docker run -p 8080:80 nginx
```

### Réseau par défaut (Bridge)

> [!note] Configuration automatique
> Par défaut, les conteneurs ont :
> - Une **interface réseau** sur un **pont virtuel (bridge)** géré par Docker
> - Une **configuration réseau** fournie via DHCP
> - Un accès sortant via **NAT** (Network Address Translation)

```
┌────────────────────────────────────┐
│         Réseau hôte                │
│                                    │
│  ┌──────────────────────────┐     │
│  │   Docker Bridge (NAT)    │     │
│  │                          │     │
│  │  ┌────────┐  ┌────────┐ │     │
│  │  │ Cont.1 │  │ Cont.2 │ │     │
│  │  │172.17.x│  │172.17.y│ │     │
│  │  └────────┘  └────────┘ │     │
│  └──────────────────────────┘     │
└────────────────────────────────────┘
```

### Réseaux personnalisés

> [!info] Créer des réseaux
> Docker permet la création d'**autres réseaux virtuels** auxquels plusieurs conteneurs peuvent être connectés.

**Types de réseaux** :
- **bridge** : réseau isolé par défaut
- **host** : utilise le réseau de l'hôte directement
- **overlay** : multi-hôtes (Swarm, Kubernetes)
- **macvlan** : assigne une adresse MAC au conteneur
- **none** : pas de réseau

> [!example] Exemple d'usage
> ```bash
> # Créer un réseau
> docker network create mon-reseau
> 
> # Lancer des conteneurs sur ce réseau
> docker run --network mon-reseau --name db mysql
> docker run --network mon-reseau --name app mon-application
> 
> # L'app peut contacter la DB via "db:3306"
> ```

> [!tip] DNS automatique
> Les conteneurs sur le même réseau personnalisé peuvent se contacter par leur **nom** (résolution DNS automatique).

---

## Considérations générales

> [!important] Bonnes pratiques Docker
> Recommandations de Docker Inc. pour une utilisation optimale.

### Un conteneur = Un service

> [!quote] Philosophie Docker
> Docker recommande qu'une **image corresponde à un seul service** :
> - Une application web
> - Un serveur de base de données
> - Un proxy
> - Un cache (Redis)
> - Etc.

> [!info] Application multi-services
> Pour mettre en place une **application complète**, on déploie potentiellement **plusieurs conteneurs** :
> - 1 conteneur pour le frontend (React, Vue.js)
> - 1 conteneur pour l'API backend (Node.js, Python)
> - 1 conteneur pour la base de données (PostgreSQL)
> - 1 conteneur pour le cache (Redis)
> - 1 conteneur pour le reverse proxy (nginx)

### Conteneurs éphémères

> [!important] Principe d'immutabilité
> Un conteneur **n'est pas censé durer dans le temps** mais plutôt être **redéployé fréquemment**.

**Raisons de redéploiement** :
- **Mise à jour** de l'application
- **Changement de configuration**
- **Scaling** (ajout/retrait d'instances)
- **Problème** rencontré (auto-healing)

> [!success] Infrastructure as Code
> Cette approche favorise :
> - **Reproductibilité** : même résultat à chaque déploiement
> - **Rollback** facile : retour à la version précédente
> - **Immutabilité** : on ne modifie pas, on recrée
> - **Automatisation** : déploiements scriptés

> [!warning] Conséquence
> Les **données persistantes** doivent être stockées dans des **volumes**, pas dans le conteneur lui-même.

---

## Docker en pratique

> [!abstract] Mise en œuvre concrète
> Passons à l'utilisation pratique de Docker avec ses commandes et outils.

---

## Composants Docker

> [!info] Écosystème Docker
> Docker est constitué de plusieurs composants complémentaires.

### Docker Engine

> [!important] Cœur de Docker
> **Docker Engine** comprend :
> - **dockerd** : daemon de gestion des conteneurs (serveur)
> - **docker** : interface en ligne de commande (CLI)
> - **containerd** : runtime de conteneurs
> - **runc** : exécution de conteneurs (bas niveau)

### Docker Desktop

> [!info] Interface graphique
> **Docker Desktop** :
> - Interface graphique pour gérer Docker
> - **Nécessaire sous Windows** (et macOS)
> - Inclut Docker Engine, Docker CLI, Docker Compose
> - Dashboard visuel

> [!warning] Attention Linux
> ⚠️ **Docker Desktop sous Linux est une VM** !
> - Pas recommandé sous Linux
> - Préférer l'installation native de Docker Engine

### Éditions

> [!note] Versions disponibles
> Docker est disponible en :
> - **CE (Community Edition)** : gratuite, open-source
> - **Souscriptions payantes** : support, fonctionnalités entreprise

---

## Outils supplémentaires

> [!info] Outils de l'écosystème Docker
> Au-delà de Docker Engine, plusieurs outils facilitent la gestion de conteneurs.

### Docker Compose

> [!important] Multi-conteneurs simplifié
> **Docker Compose** permet le lancement d'un **ensemble de conteneurs** (application multi-conteneurs) décrit par un fichier `docker-compose.yml`.

**Avantages** :
- ✅ **Déclaratif** : infrastructure as code
- ✅ **Reproductible** : même config partout
- ✅ **Simple** : une commande pour tout lancer
- ✅ **Gestion globale** : start/stop/logs de tous les conteneurs

> [!example] Exemple docker-compose.yml
> ```yaml
> version: '3.8'
> services:
>   web:
>     image: nginx
>     ports:
>       - "8080:80"
>   db:
>     image: mysql:8.0
>     environment:
>       MYSQL_ROOT_PASSWORD: secret
> ```

### Docker Swarm

> [!info] Cluster Docker natif
> **Docker Swarm** : regroupement de plusieurs **Docker Engine** en un **cluster**.

**Fonctionnalités** :
- Orchestration native Docker
- Load balancing intégré
- Scaling automatique
- Service discovery
- Rolling updates

> [!note] Alternative
> Moins utilisé que Kubernetes mais plus simple à mettre en place.

### Kubernetes (K8s)

> [!important] Standard de l'orchestration
> **Kubernetes** : système d'**orchestration poussée** de conteneurs, devenu le standard de l'industrie.

**Capacités** :
- Orchestration à très grande échelle
- Auto-healing avancé
- Scaling automatique (HPA, VPA, Cluster Autoscaler)
- Service mesh
- Écosystème riche (Helm, Operators, etc.)

> [!warning] Complexité élevée
> Kubernetes est **très puissant** mais **très complexe**. Formation nécessaire.

---

## Docker CLI

> [!info] Interface en ligne de commande
> L'utilisation de Docker en ligne de commande passe par la commande `docker`.

### Syntaxe générale

```bash
docker [OPTIONS] COMMAND [ARG...]
```

### Prérequis

> [!warning] Permissions nécessaires
> Nécessite d'être dans le groupe `docker` (Linux) :
> ```bash
> sudo usermod -aG docker $USER
> # Déconnexion/reconnexion nécessaire
> ```

### Aide

```bash
# Aide générale
docker help

# Aide sur une commande spécifique
docker COMMAND --help

# Exemples
docker run --help
docker network --help
```

### Structure des commandes

> [!note] Organisation
> Les commandes Docker sont organisées en **catégories** :
> - **Management Commands** : gestion des ressources (image, container, volume, network)
> - **Commands** : actions directes (run, build, pull, push)

---

## Gestion des images

> [!info] Commandes images
> La gestion des images se fait via : `docker image COMMAND`

### Commandes principales

| Commande | Description | Exemple |
|----------|-------------|---------|
| `ls` | Afficher les images disponibles | `docker image ls` |
| `pull` | Récupérer une image depuis un dépôt | `docker image pull nginx` |
| `push` | Publier une image vers un dépôt | `docker image push mon-image` |
| `rm` | Supprimer une image | `docker image rm nginx` |
| `tag` | Créer un tag (alias) | `docker image tag nginx mon-nginx` |
| `build` | Construire une image | `docker build -t mon-app .` |
| `inspect` | Détails sur une image | `docker image inspect nginx` |
| `prune` | Supprimer images inutilisées | `docker image prune` |

### Alias courants

> [!tip] Raccourcis
> Des alias existent pour simplifier :
> ```bash
> docker images     # équivalent à docker image ls
> docker rmi nginx  # équivalent à docker image rm nginx
> docker pull nginx # équivalent à docker image pull nginx
> ```

### Exemples pratiques

```bash
# Lister les images locales
docker image ls
docker images  # alias

# Télécharger une image
docker pull ubuntu:22.04

# Supprimer une image
docker image rm ubuntu:22.04
docker rmi ubuntu:22.04  # alias

# Afficher l'historique des layers
docker history nginx

# Nettoyer les images non utilisées
docker image prune -a
```

---

## Lancer un conteneur

> [!important] Commande docker run
> Création et démarrage d'un conteneur : `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

### Options principales

| Option | Description | Exemple |
|--------|-------------|---------|
| `-d, --detach` | Exécuter en arrière-plan | `docker run -d nginx` |
| `-i, --interactive` | Garder STDIN ouvert | `docker run -it ubuntu bash` |
| `-t, --tty` | Allouer un pseudo-TTY | `docker run -it ubuntu bash` |
| `-p, --publish` | Publier un port (hôte:conteneur) | `docker run -p 8080:80 nginx` |
| `--name` | Nommer le conteneur | `docker run --name web nginx` |
| `-e, --env` | Variable d'environnement | `docker run -e API_KEY=abc nginx` |
| `-v, --volume` | Monter un volume | `docker run -v data:/data nginx` |
| `--network` | Connecter à un réseau | `docker run --network mon-reseau nginx` |
| `--rm` | Supprimer auto après arrêt | `docker run --rm nginx` |
| `--restart` | Politique de redémarrage | `docker run --restart always nginx` |

### Exemples pratiques

```bash
# Lancer nginx en arrière-plan sur le port 8080
docker run -d -p 8080:80 --name mon-nginx nginx

# Lancer Ubuntu en mode interactif
docker run -it ubuntu bash

# Lancer avec variables d'environnement
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Lancer avec volume
docker run -d -v /data-hote:/data-conteneur ubuntu

# Combiner plusieurs options
docker run -d \
  --name mon-app \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -v app-data:/app/data \
  --network app-network \
  --restart unless-stopped \
  mon-image:latest
```

### Command et Args

> [!note] Surcharger la commande
> `[COMMAND] [ARG...]` remplace la commande par défaut de l'image.

```bash
# Commande par défaut
docker run ubuntu
# Exécute la commande définie dans l'image

# Commande personnalisée
docker run ubuntu echo "Hello Docker"
# Exécute echo au lieu de la commande par défaut

# Mode interactif avec bash
docker run -it ubuntu bash
```

---

## Gestion des conteneurs

> [!info] Commandes conteneurs
> Gestion des conteneurs : `docker container COMMAND`

### Commandes principales

| Commande | Description | Exemple |
|----------|-------------|---------|
| `ls` | Lister les conteneurs | `docker container ls` |
| `start` | Démarrer un conteneur arrêté | `docker container start mon-conteneur` |
| `stop` | Arrêter un conteneur | `docker container stop mon-conteneur` |
| `restart` | Redémarrer un conteneur | `docker container restart mon-conteneur` |
| `rm` | Supprimer un conteneur | `docker container rm mon-conteneur` |
| `exec` | Exécuter commande dans conteneur | `docker exec -it mon-conteneur bash` |
| `logs` | Afficher les logs | `docker container logs mon-conteneur` |
| `inspect` | Détails sur un conteneur | `docker container inspect mon-conteneur` |
| `stats` | Statistiques d'utilisation | `docker container stats` |
| `top` | Processus en cours | `docker container top mon-conteneur` |

### Alias courants

```bash
docker ps        # équivalent à docker container ls
docker ps -a     # tous les conteneurs (y compris arrêtés)
docker stop nginx # équivalent à docker container stop nginx
```

### Exemples pratiques

```bash
# Lister les conteneurs en cours d'exécution
docker ps
docker container ls  # équivalent

# Lister TOUS les conteneurs (même arrêtés)
docker ps -a
docker container ls -a  # équivalent

# Arrêter un conteneur
docker stop mon-conteneur

# Démarrer un conteneur arrêté
docker start mon-conteneur

# Redémarrer
docker restart mon-conteneur

# Supprimer un conteneur (doit être arrêté)
docker rm mon-conteneur

# Supprimer un conteneur en cours (force)
docker rm -f mon-conteneur

# Exécuter une commande dans un conteneur en cours
docker exec mon-conteneur ls /app
docker exec -it mon-conteneur bash  # shell interactif

# Afficher les logs
docker logs mon-conteneur
docker logs -f mon-conteneur  # suivre en temps réel (tail -f)

# Statistiques en temps réel
docker stats

# Inspecter un conteneur (JSON)
docker inspect mon-conteneur

# Copier des fichiers
docker cp mon-conteneur:/app/log.txt ./log.txt
docker cp ./config.json mon-conteneur:/app/
```

---

## Gestion des volumes Docker

> [!info] Commandes volumes
> Gestion des volumes : `docker volume COMMAND`

### Commandes principales

| Commande | Description | Exemple |
|----------|-------------|---------|
| `create` | Créer un volume | `docker volume create mon-volume` |
| `ls` | Lister les volumes | `docker volume ls` |
| `inspect` | Détails sur un volume | `docker volume inspect mon-volume` |
| `rm` | Supprimer un volume | `docker volume rm mon-volume` |
| `prune` | Supprimer volumes inutilisés | `docker volume prune` |

### Exemples pratiques

```bash
# Créer un volume
docker volume create mon-volume

# Lister les volumes
docker volume ls

# Utiliser un volume dans un conteneur
docker run -v mon-volume:/data nginx

# Inspecter un volume (voir où il est stocké)
docker volume inspect mon-volume

# Supprimer un volume
docker volume rm mon-volume

# Nettoyer les volumes non utilisés
docker volume prune
```

> [!warning] Attention à prune
> `docker volume prune` supprime **tous** les volumes non utilisés par au moins un conteneur. Utiliser avec précaution en production !

---

## Gestion des réseaux Docker

> [!info] Commandes réseaux
> Gestion des réseaux : `docker network COMMAND`

### Commandes principales

| Commande | Description | Exemple |
|----------|-------------|---------|
| `create` | Créer un réseau | `docker network create mon-reseau` |
| `ls` | Lister les réseaux | `docker network ls` |
| `inspect` | Détails sur un réseau | `docker network inspect mon-reseau` |
| `connect` | Connecter conteneur à réseau | `docker network connect mon-reseau conteneur` |
| `disconnect` | Déconnecter conteneur | `docker network disconnect mon-reseau conteneur` |
| `rm` | Supprimer un réseau | `docker network rm mon-reseau` |
| `prune` | Supprimer réseaux inutilisés | `docker network prune` |

### Exemples pratiques

```bash
# Créer un réseau
docker network create mon-reseau

# Lister les réseaux
docker network ls

# Créer réseau avec configuration spécifique
docker network create --driver bridge --subnet 172.25.0.0/16 mon-reseau-custom

# Lancer conteneurs sur le même réseau
docker run -d --name db --network mon-reseau mysql
docker run -d --name app --network mon-reseau mon-application

# Connecter un conteneur existant à un réseau
docker network connect mon-reseau mon-conteneur

# Inspecter un réseau (voir conteneurs connectés)
docker network inspect mon-reseau

# Supprimer un réseau
docker network rm mon-reseau

# Nettoyer réseaux non utilisés
docker network prune
```

---

## Gestion globale

> [!info] Commandes système
> Gestion globale de Docker : `docker system COMMAND`

### Commandes principales

| Commande | Description | Exemple |
|----------|-------------|---------|
| `df` | Utilisation disque de Docker | `docker system df` |
| `info` | Informations système | `docker system info` |
| `events` | Événements en temps réel | `docker system events` |
| `prune` | Supprimer données inutilisées | `docker system prune` |

### Nettoyage

> [!important] docker system prune
> La commande `prune` permet un **nettoyage global** des éléments inutilisés.

```bash
# Nettoyer (conteneurs arrêtés, réseaux, images dangling)
docker system prune

# Nettoyer TOUT (incluant volumes)
docker system prune -a --volumes

# Afficher ce qui serait supprimé sans le faire
docker system prune --dry-run
```

> [!warning] Danger
> `docker system prune -a --volumes` supprime **tout** ce qui n'est pas utilisé actuellement. **Très destructif** !

### Utilisation disque

```bash
# Voir l'utilisation disque
docker system df

# Sortie exemple :
# TYPE            TOTAL   ACTIVE   SIZE      RECLAIMABLE
# Images          15      5        2.5GB     1.8GB (72%)
# Containers      10      2        500MB     300MB (60%)
# Local Volumes   5       1        1GB       800MB (80%)
# Build Cache     0       0        0B        0B
```

### Informations système

```bash
# Informations complètes sur Docker
docker system info

# Inclut : version, stockage, plugins, runtime, etc.
```

---

## Le Dockerfile

> [!quote] Fichier de construction
> Le **Dockerfile** est un fichier texte contenant les instructions pour **construire une image Docker**.

### Instructions principales

| Instruction | Description | Exemple |
|-------------|-------------|---------|
| `FROM` | Image de base (obligatoire, 1 seul) | `FROM ubuntu:22.04` |
| `WORKDIR` | Définir le répertoire de travail | `WORKDIR /app` |
| `COPY` | Copier fichiers locaux vers image | `COPY . /app` |
| `ADD` | Copier + extraction (tar, URL) | `ADD app.tar.gz /app` |
| `RUN` | Exécuter une commande (build-time) | `RUN apt-get update` |
| `CMD` | Commande par défaut (1 seul) | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | Point d'entrée principal | `ENTRYPOINT ["nginx"]` |
| `EXPOSE` | Ports en écoute (documentation) | `EXPOSE 80 443` |
| `ENV` | Variables d'environnement | `ENV NODE_ENV=production` |
| `ARG` | Arguments de build | `ARG VERSION=1.0` |
| `LABEL` | Métadonnées | `LABEL maintainer="email@example.com"` |
| `VOLUME` | Déclarer un volume | `VOLUME ["/data"]` |
| `USER` | Utilisateur d'exécution | `USER node` |

### Exemple complet

> [!example] Application Node.js
> ```dockerfile
> # Image de base
> FROM node:18-alpine
> 
> # Répertoire de travail
> WORKDIR /app
> 
> # Copier package.json et package-lock.json
> COPY package*.json ./
> 
> # Installer les dépendances
> RUN npm install --production
> 
> # Copier le reste de l'application
> COPY . .
> 
> # Exposer le port
> EXPOSE 3000
> 
> # Utilisateur non-root (sécurité)
> USER node
> 
> # Commande par défaut
> CMD ["node", "src/index.js"]
> ```

### Bonnes pratiques

> [!tip] Optimisation des layers
> - **Ordre des instructions** : mettre ce qui change peu au début
> - **Minimiser les layers** : combiner les `RUN` quand possible
> - **Nettoyer dans le même layer** : `RUN apt-get update && apt-get install && rm -rf /var/lib/apt/lists/*`
> - **Utiliser .dockerignore** : exclure fichiers inutiles
> - **Images de base légères** : alpine, slim
> - **Multi-stage builds** : réduire taille finale

> [!example] Multi-stage build
> ```dockerfile
> # Stage 1 : Build
> FROM node:18 AS builder
> WORKDIR /app
> COPY package*.json ./
> RUN npm install
> COPY . .
> RUN npm run build
> 
> # Stage 2 : Production
> FROM node:18-alpine
> WORKDIR /app
> COPY --from=builder /app/dist ./dist
> COPY package*.json ./
> RUN npm install --production
> CMD ["node", "dist/index.js"]
> ```

### Référence complète

> [!note] Documentation officielle
> Dockerfile référence : https://docs.docker.com/engine/reference/builder/

---

## Générer une image

> [!info] Commande docker build
> Création d'une image à partir d'un Dockerfile : `docker build [OPTIONS] PATH`

### Syntaxe

```bash
docker build [OPTIONS] PATH | URL | -
```

### Options principales

| Option | Description | Exemple |
|--------|-------------|---------|
| `-t, --tag` | Nommer et tagger l'image | `docker build -t mon-app:1.0 .` |
| `-f, --file` | Spécifier le Dockerfile | `docker build -f Dockerfile.prod .` |
| `--no-cache` | Ne pas utiliser le cache | `docker build --no-cache .` |
| `--build-arg` | Passer un argument de build | `docker build --build-arg VERSION=1.0 .` |
| `--target` | Stage cible (multi-stage) | `docker build --target builder .` |

### Exemples pratiques

```bash
# Build basique (Dockerfile dans le répertoire courant)
docker build -t mon-app .

# Build avec tag de version
docker build -t mon-app:1.0 .
docker build -t mon-app:latest .

# Build avec un Dockerfile spécifique
docker build -f Dockerfile.prod -t mon-app:prod .

# Build sans cache (reconstruction complète)
docker build --no-cache -t mon-app .

# Build avec arguments
docker build --build-arg VERSION=2.0 --build-arg ENV=prod -t mon-app .

# Build multi-stage (cibler un stage)
docker build --target builder -t mon-app-builder .

# Build depuis un dépôt Git
docker build -t mon-app https://github.com/user/repo.git#branch

# Build avec progression détaillée
docker build --progress=plain -t mon-app .
```

### Fichier .dockerignore

> [!tip] Exclure des fichiers
> Créer un fichier `.dockerignore` pour exclure fichiers/dossiers du contexte de build :

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
*.log
dist
coverage
```

> [!success] Avantages
> - ⚡ **Build plus rapide** (contexte plus léger)
> - 🔒 **Sécurité** (pas de secrets dans l'image)
> - 💾 **Image plus petite**

---

## Cas d'usage

> [!success] Utilisations de Docker
> Docker s'applique à de nombreux scénarios professionnels.

### Applications principales

1. **Création de packages de déploiement d'application**
   - Image = artefact déployable
   - Versionning via tags
   - Distribution via registry

2. **Homogénéisation des environnements**
   - Dev = Test = Pré-prod = Prod
   - Même image partout
   - "It works on my machine" résolu

3. **Déploiements rapides et reproductibles**
   - Secondes vs minutes/heures
   - Rollback instantané
   - Blue/Green, Canary déployments

4. **Automatisation**
   - CI/CD facilitée
   - Tests automatisés dans conteneurs
   - Infrastructure as Code

### Cas d'usage spécifiques

> [!example] Exemples concrets
> - **Développement local** : environnement complet en une commande
> - **Tests** : environnements éphémères pour chaque test
> - **Microservices** : déploiement indépendant de chaque service
> - **Legacy modernization** : conteneuriser des applications anciennes
> - **Edge computing** : déploiement sur IoT, edge devices
> - **Machine Learning** : environnements reproductibles pour ML

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

- **Conteneur** : environnement isolé pour processus, partageant le noyau hôte
- **Image** : template statique (programme + dépendances)
- **Conteneur** : instance d'une image en exécution (processus)
- **Différence VM/Conteneur** : VM = noyau dédié / Conteneur = noyau partagé
- **Isolation** : namespaces (PID, network, mount) + cgroups (CPU, RAM)

### Avantages de la conteneurisation

**Pour les développeurs** :
- ✅ Même environnement partout (dev = prod)
- ✅ Déploiement simplifié
- ✅ Fin du "ça marche sur ma machine"

**Pour l'infrastructure** :
- ✅ Cloisonnement strict (sécurité)
- ✅ Déploiement rapide (secondes)
- ✅ Densité élevée (économies)
- ✅ Alternative légère aux VM

### Solutions de conteneurisation

| Type | Solution | Usage |
|------|----------|-------|
| **Bas niveau** | chroot, namespaces, cgroups | Fondations |
| **Système** | LXC/LXD | Conteneur = VM légère |
| **Applicative** | Docker, Podman | 1 conteneur = 1 service |
| **Orchestration** | Kubernetes, Swarm | Multi-conteneurs, multi-hôtes |

### Composants Docker

- **Image** : construite en layers, récupérée ou créée via Dockerfile
- **Conteneur** : instance éphémère d'une image
- **Volume** : stockage persistant, survit à la suppression du conteneur
- **Réseau** : communication entre conteneurs (bridge, host, overlay)
- **Registry** : dépôt d'images (Docker Hub, Harbor, etc.)

### Commandes Docker essentielles

```bash
# Images
docker pull nginx                    # Télécharger
docker build -t mon-app .           # Construire
docker images                        # Lister

# Conteneurs
docker run -d -p 8080:80 nginx      # Lancer
docker ps                            # Lister (actifs)
docker ps -a                         # Lister (tous)
docker stop mon-conteneur            # Arrêter
docker rm mon-conteneur              # Supprimer
docker exec -it conteneur bash       # Shell interactif
docker logs -f conteneur             # Logs en temps réel

# Volumes
docker volume create mon-volume      # Créer
docker run -v mon-volume:/data nginx # Utiliser

# Réseaux
docker network create mon-reseau     # Créer
docker run --network mon-reseau nginx # Utiliser

# Nettoyage
docker system prune                  # Nettoyer
```

### Dockerfile : instructions clés

```dockerfile
FROM ubuntu:22.04                # Image de base
WORKDIR /app                     # Répertoire de travail
COPY . .                         # Copier fichiers
RUN apt-get update              # Exécuter commande (build)
CMD ["python", "app.py"]        # Commande par défaut (run)
EXPOSE 80                        # Port documenté
ENV NODE_ENV=production         # Variable d'environnement
```

### Bonnes pratiques

1. **Un conteneur = un service** (principe de responsabilité unique)
2. **Conteneurs éphémères** : redéployer plutôt que modifier
3. **Données persistantes dans volumes** : jamais dans le conteneur
4. **Images légères** : alpine, slim, multi-stage builds
5. **Sécurité** : utilisateur non-root, scanner les images
6. **.dockerignore** : exclure fichiers inutiles
7. **Tags explicites** : éviter `:latest` en production
8. **Logs vers stdout/stderr** : pas dans fichiers

### Points d'attention

- ⚠️ **Isolation moins forte que VM** : partage du noyau
- ⚠️ **Données perdues** sans volume (conteneur éphémère)
- ⚠️ **Sécurité kernel** : vulnérabilité noyau = impact tous conteneurs
- ⚠️ **Permissions Docker** : groupe docker = accès root
- ⚠️ **Complexité orchestration** : Kubernetes a une courbe d'apprentissage raide

### Pour le TSSR

> [!important] Compétences à maîtriser
> 1. **Comprendre** l'isolation (namespaces, cgroups)
> 2. **Installer et configurer** Docker Engine
> 3. **Lancer et gérer** des conteneurs
> 4. **Créer des Dockerfiles** optimisés
> 5. **Gérer volumes et réseaux**
> 6. **Utiliser Docker Compose** pour multi-conteneurs
> 7. **Intégrer Docker** dans une infrastructure existante
> 8. **Sécuriser** les déploiements Docker
> 9. **Monitorer** les conteneurs (logs, stats)
> 10. **Troubleshooter** les problèmes courants

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Conteneur** | Environnement isolé pour exécuter des processus, partageant le noyau du système hôte |
| **Image** | Template en lecture seule contenant le code, les bibliothèques et dépendances nécessaires pour exécuter une application |
| **Layer (Couche)** | Une des couches composant une image Docker, chaque instruction Dockerfile créant un nouveau layer |
| **Dockerfile** | Fichier texte contenant les instructions pour construire automatiquement une image Docker |
| **Registry (Dépôt)** | Service de stockage et distribution d'images Docker (ex: Docker Hub, Harbor) |
| **Docker Hub** | Registry public officiel de Docker Inc., hébergeant des images publiques et privées |
| **Volume** | Mécanisme de stockage persistant géré par Docker, survivant à la suppression des conteneurs |
| **Bind mount** | Montage direct d'un répertoire de l'hôte dans un conteneur |
| **Network (Réseau)** | Réseau virtuel permettant la communication entre conteneurs |
| **Bridge** | Type de réseau par défaut créant un pont virtuel entre conteneurs |
| **Host network** | Mode réseau où le conteneur utilise directement la stack réseau de l'hôte |
| **Port mapping** | Association d'un port de l'hôte avec un port du conteneur (ex: 8080:80) |
| **Docker Engine** | Runtime Docker comprenant dockerd (daemon) et la CLI |
| **dockerd** | Daemon Docker, processus serveur gérant les conteneurs |
| **Docker CLI** | Interface en ligne de commande pour interagir avec Docker |
| **Docker Compose** | Outil pour définir et lancer des applications multi-conteneurs via fichier YAML |
| **Docker Swarm** | Solution native d'orchestration de conteneurs Docker en cluster |
| **Kubernetes (K8s)** | Système d'orchestration de conteneurs à grande échelle, standard de l'industrie |
| **Namespace** | Mécanisme du noyau Linux isolant les ressources système (PID, network, mount, etc.) |
| **Cgroups** | Control Groups - mécanisme Linux limitant et contrôlant les ressources (CPU, RAM, I/O) |
| **chroot** | Commande Unix changeant la racine du système de fichiers pour un processus |
| **Jail** | Environnement d'isolation sur BSD, ancêtre des conteneurs |
| **LXC** | Linux Containers - solution de conteneurisation système créant des environnements complets |
| **LXD** | Gestionnaire et couche d'administration pour conteneurs LXC |
| **Hyperviseur** | Logiciel créant et gérant des machines virtuelles (Type 1: bare-metal, Type 2: hosted) |
| **VM (Virtual Machine)** | Machine virtuelle complète avec son propre noyau OS |
| **Conteneur système** | Conteneur ressemblant à une VM complète (LXC) avec init, services multiples |
| **Conteneur applicatif** | Conteneur léger exécutant une seule application (Docker) |
| **Tag** | Étiquette versionnant une image (ex: nginx:1.21, mon-app:latest) |
| **latest** | Tag par défaut, désignant la version la plus récente (à éviter en production) |
| **Pull** | Télécharger une image depuis un registry |
| **Push** | Publier une image vers un registry |
| **Build** | Construire une image à partir d'un Dockerfile |
| **Run** | Créer et démarrer un conteneur à partir d'une image |
| **Exec** | Exécuter une commande dans un conteneur en cours d'exécution |
| **Commit** | Créer une nouvelle image à partir des modifications d'un conteneur |
| **ENTRYPOINT** | Instruction Dockerfile définissant le point d'entrée principal non modifiable |
| **CMD** | Instruction Dockerfile définissant la commande par défaut, modifiable au run |
| **EXPOSE** | Instruction Dockerfile documentant les ports sur lesquels le conteneur écoute |
| **ENV** | Variable d'environnement définie dans l'image ou le conteneur |
| **WORKDIR** | Répertoire de travail dans le conteneur |
| **Multi-stage build** | Technique Dockerfile utilisant plusieurs FROM pour optimiser la taille finale |
| **.dockerignore** | Fichier listant les fichiers/dossiers à exclure du contexte de build |
| **Build context** | Ensemble des fichiers envoyés au daemon Docker lors du build |
| **Dangling image** | Image sans tag, généralement ancienne version remplacée |
| **Ephemeral (Éphémère)** | Qualifie un conteneur conçu pour être supprimé et recréé facilement |
| **Immutability** | Principe d'infrastructure immuable : on ne modifie pas, on recrée |
| **Orchestration** | Gestion automatisée du cycle de vie de multiples conteneurs sur plusieurs hôtes |
| **Service discovery** | Mécanisme permettant aux services de se découvrir automatiquement |
| **Load balancing** | Répartition de charge entre plusieurs instances d'un service |
| **Scaling** | Ajustement du nombre d'instances d'un service (horizontal) ou des ressources (vertical) |
| **Rolling update** | Mise à jour progressive remplaçant les instances une par une |
| **Rollback** | Retour à une version précédente en cas de problème |
| **Health check** | Vérification périodique de l'état de santé d'un conteneur |
| **Restart policy** | Politique de redémarrage automatique d'un conteneur (no, always, unless-stopped, on-failure) |
| **Detached mode** | Exécution en arrière-plan (option -d) |
| **Interactive mode** | Exécution interactive avec terminal (options -it) |
| **Stateless** | Application sans état, ne stockant pas de données localement |
| **Stateful** | Application avec état, nécessitant stockage persistant |
| **Microservices** | Architecture applicative découpée en services indépendants et déployables séparément |
| **CI/CD** | Continuous Integration / Continuous Deployment - pratiques DevOps d'automatisation |
| **DevOps** | Culture et pratiques rapprochant développement (Dev) et opérations (Ops) |
| **Infrastructure as Code** | Gestion de l'infrastructure via code versionné et reproductible |

---

## Ressources et références

> [!info] Documentation et apprentissage
> Ressources officielles et recommandées pour approfondir Docker.

### Documentation officielle Docker

> [!important] Sources principales
> **Site officiel** : https://www.docker.com
> 
> **Documentation** : https://docs.docker.com
> - **Guides** : https://docs.docker.com/guides/
> - **Manuals** : https://docs.docker.com/manuals/
> - **Reference** : https://docs.docker.com/reference/
> 
> **Docker 101 Tutorial** : https://www.docker.com/101-tutorial/

### Sécurité

> [!warning] Guide ANSSI
> **Guide de sécurisation ANSSI** : https://www.ssi.gouv.fr/
> 
> Recommandations de sécurité pour Docker :
> - Durcissement des images
> - Gestion des secrets
> - Contrôle d'accès
> - Monitoring et logs

### Outils avancés

> [!info] Aller plus loin
> - **Docker Compose** : https://docs.docker.com/compose/
> - **Docker Swarm** : https://docs.docker.com/engine/swarm/
> - **Kubernetes** : https://kubernetes.io/fr/docs/home/

### Ressources communautaires francophones

> [!tip] Experts francophones
> - **Site d'Hadrien Pélissier** : expertise DevOps et conteneurs
> - **Site de Stéphane Robert** : tutoriels et guides pratiques

### Formation continue

> [!note] Apprentissage pratique
> Pour ton homelab Proxmox :
> - Installer Docker sur VM Debian/Ubuntu
> - Pratiquer les commandes de base
> - Créer tes propres Dockerfiles
> - Tester Docker Compose
> - Expérimenter avec les réseaux et volumes

---

## Conclusion

> [!abstract] Résumé final
> La **conteneurisation** et **Docker** sont devenus incontournables dans les infrastructures modernes.

### Ce que tu dois maîtriser en tant que TSSR

1. **Comprendre l'isolation des processus** (namespaces, cgroups)
2. **Différencier conteneurs et VMs** (usages, avantages, limites)
3. **Maîtriser Docker CLI** (images, conteneurs, volumes, réseaux)
4. **Créer des Dockerfiles** optimisés et sécurisés
5. **Gérer le stockage** (volumes, bind mounts)
6. **Configurer les réseaux** Docker
7. **Déployer des applications** multi-conteneurs (Docker Compose)
8. **Appliquer les bonnes pratiques** (sécurité, performance, maintenance)

### Points clés à retenir

> [!success] Essentiels
> - **Conteneurisation** = isolation des processus (pas une VM)
> - **Docker** = standard de facto de la conteneurisation applicative
> - **Image** = template / **Conteneur** = instance en exécution
> - **Un conteneur = Un service** (philosophie microservices)
> - **Éphémère** : on supprime et recrée, on ne modifie pas
> - **Volumes** pour la persistence, **réseaux** pour la communication
> - **Dockerfile** = Infrastructure as Code
> - **Orchestration** (K8s) pour passer à l'échelle

### Pour aller plus loin

> [!tip] Prochaines étapes
> 1. **Pratique intensive** sur ton lab Proxmox
> 2. **Docker Compose** pour applications complètes
> 3. **Sécurisation** (scan d'images, users non-root, secrets)
> 4. **Monitoring** (Prometheus, Grafana pour conteneurs)
> 5. **CI/CD** (intégration Docker dans pipelines)
> 6. **Kubernetes** (quand tu maîtriseras bien Docker)

---

> [!success] Prêt pour le titre RNCP !
> Tu disposes maintenant d'une vision complète de la conteneurisation et de Docker. Pratique intensivement sur ton homelab pour consolider ces connaissances avant ton examen en mars 2026 ! 🐳🚀

---

**📚 Document créé pour la préparation au titre RNCP TSSR**

**🎯 Objectif** : Maîtriser les conteneurs et Docker pour l'examen et la pratique professionnelle

**✅ Compatible Obsidian** avec callouts natifs et liens internes