# Les outils de l'administrateur
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Les outils de l'administrateur - Les éléments de base  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Ordinateur|Ordinateur]]
   - [[#Différents types d'ordinateurs|Différents types d'ordinateurs]]
   - [[#Administrer un serveur|Administrer un serveur]]
   - [[#Choix de systèmes d'exploitation|Choix de systèmes d'exploitation]]
3. [[#Des logiciels|Des logiciels]]
   - [[#Command-Line Interpreter - Shell|Command-Line Interpreter - Shell]]
   - [[#Éditeurs de texte CLI|Éditeurs de texte CLI]]
   - [[#La boîte à outils de l'administrateur|La boîte à outils de l'administrateur]]
4. [[#Points clés à retenir|Points clés à retenir]]
5. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Ce document présente les **éléments de base** dont dispose un administrateur système et réseau pour effectuer son travail au quotidien. Il couvre les différents types d'ordinateurs, les interfaces d'administration, les systèmes d'exploitation et les outils logiciels essentiels.

### Pourquoi étudier les outils de l'administrateur ?

En tant que **TSSR**, tu dois :
- Comprendre la différence entre **postes clients** et **serveurs**
- Maîtriser les **interfaces d'administration** (console, terminal, SSH, web)
- Connaître les **systèmes d'exploitation** disponibles (Windows, Linux, BSD, macOS)
- Utiliser efficacement les **outils en ligne de commande** (shell, éditeurs de texte)
- Constituer ta **boîte à outils** d'administrateur professionnel

> [!important] Objectif principal
> Identifier et utiliser les bons outils pour chaque tâche d'administration système et réseau, en fonction du contexte (poste client vs serveur, interface graphique vs ligne de commande).

---

## Ordinateur

> [!quote] Définition
> Un **ordinateur** est une machine électronique capable de traiter des informations de manière automatique selon des programmes préalablement enregistrés.

### Différents types d'ordinateurs

En tant qu'administrateur système, tu vas travailler avec **deux grandes catégories** d'ordinateurs aux usages très différents.

#### Postes clients (ordinateur de bureau - laptop - smartphone)

> [!info] Caractéristiques des postes clients
> Les postes clients sont les machines utilisées directement par les utilisateurs finaux pour effectuer leur travail quotidien.

**Caractéristiques principales** :
- **Équipements** : Clavier, écran, souris (ou écran tactile pour smartphone/tablette)
- **Localisation** : Installés dans l'espace de travail de l'utilisateur ou portables
- **Utilisateurs** : 1 utilisateur simultané à la fois
- **Disponibilité** : Allumés au besoin (pas nécessairement 24h/24)
- **Interface** : Interface graphique (GUI) principalement utilisée
- **Exemples** : PC de bureau Windows, MacBook, smartphone Android/iOS

> [!example] Cas d'usage typiques
> - Poste de travail d'un employé de bureau
> - Laptop d'un commercial en déplacement
> - Smartphone d'un technicien sur le terrain

#### Serveurs / Équipements réseaux

> [!info] Caractéristiques des serveurs
> Les serveurs sont des machines dédiées à fournir des services à plusieurs utilisateurs simultanément, généralement sans interaction humaine directe.

**Caractéristiques principales** :
- **Disponibilité** : Allumés en permanence (24h/24, 7j/7)
- **Utilisateurs** : Utilisés par **plusieurs utilisateurs simultanément**
- **Localisation** : Regroupés dans un **local dédié** (salle serveurs) sur site ou distant (datacenter, cloud)
- **Équipements** : Rarement équipés de clavier, écran et souris directement
- **Administration** : Gérés à distance via console réseau, SSH, interface web
- **Exemples** : Serveur web Apache, serveur de fichiers NAS, contrôleur de domaine Active Directory

> [!warning] Point d'attention
> Les serveurs étant critiques pour l'entreprise, leur administration nécessite des précautions particulières :
> - Toujours avoir un plan de sauvegarde avant intervention
> - Privilégier les interventions en dehors des heures de production
> - Documenter toutes les modifications effectuées

> [!example] Équipements réseaux associés
> Les **équipements réseaux** (switchs, routeurs, pare-feu) partagent les mêmes caractéristiques que les serveurs :
> - Allumés en permanence
> - Administration à distance (console série, SSH, web)
> - Aucun périphérique d'affichage direct

**Tableau comparatif** :

| Critère | Poste client | Serveur / Équipement réseau |
|---------|--------------|------------------------------|
| **Disponibilité** | Au besoin | 24h/24 |
| **Utilisateurs simultanés** | 1 | Plusieurs |
| **Localisation** | Bureau / Portable | Salle serveurs / Datacenter |
| **Périphériques** | Clavier, écran, souris | Rarement équipé |
| **Interface principale** | Graphique (GUI) | Ligne de commande (CLI) |
| **Administration** | Locale | À distance |

---

### Administrer un serveur

> [!question] Quelle interface utiliser ?
> Comment administrer un serveur qui n'a pas de clavier, d'écran ni de souris directement connectés ?

#### Historique : Le terminal

> [!note] Contexte historique
> Historiquement, l'accès à la **console système** se faisait via un **terminal** physique connecté au serveur.

Un **terminal** était un périphérique composé :
- D'un **écran** (souvent monochrome)
- D'un **clavier**
- **Sans** unité de calcul (le traitement se fait sur le serveur)

> [!info] Photo de référence
> Sur la photo Wikimedia présentée dans le cours, on voit une **baie de serveurs** dans un datacenter avec un **terminal intégré** (écran + clavier) permettant d'administrer les serveurs de la baie sans avoir besoin de connecter un équipement externe.
> 
> Source : Photo de David Martín :: Suki_ :: sur Wikimedia

#### Solutions modernes d'administration

Aujourd'hui, plusieurs solutions existent pour administrer un serveur à distance :

##### 1. Switch KVM

> [!info] Switch KVM
> Un **Switch KVM** (Keyboard, Video, Mouse) est un boîtier matériel permettant de contrôler plusieurs serveurs avec un seul clavier, un seul écran et une seule souris.

**Fonctionnement** :
- Connexion physique au serveur via câbles VGA/HDMI + USB
- Basculement d'un serveur à l'autre par raccourci clavier
- Idéal pour administration locale en salle serveurs

**Lien** : [Console système](https://fr.wikipedia.org/wiki/Console_système) (mentionné dans le cours)

##### 2. Émulateur de terminal et connexion directe

> [!info] Console série
> Certains serveurs disposent d'un **port série** (RJ45 ou USB) permettant une connexion directe à la console système.

**Configuration** :
- Connexion via câble **RJ45** (avec adaptateur série) ou **USB**
- Utilisation d'un **émulateur de terminal** (PuTTY, minicom, screen)
- Accès de secours en cas de panne réseau

> [!tip] Documentation utile
> Le cours mentionne "une petite doc" pour la connexion directe. Cela fait référence aux guides de connexion série spécifiques à chaque constructeur (Dell iDRAC, HP iLO, Cisco console, etc.).

##### 3. Console réseau via SSH

> [!important] SSH - La méthode privilégiée
> **SSH** (Secure Shell) est le protocole standard pour l'administration à distance des serveurs Linux/Unix.

**Avantages** :
- Connexion **sécurisée** et **chiffrée** sur le réseau
- Accès à distance depuis n'importe où
- Authentification par mot de passe ou clé publique
- Transfert de fichiers via SCP/SFTP

**Commande typique** :
```bash
ssh utilisateur@adresse-serveur
```

##### 4. Interface web

> [!info] Administration web
> De nombreux serveurs et équipements proposent une **interface web** d'administration.

**Exemples** :
- **Webmin** / **Cockpit** (administration Linux)
- **Proxmox** (virtualisation)
- **pfSense** (pare-feu)
- **UniFi Controller** (équipements réseau Ubiquiti)
- **iLO** / **iDRAC** (gestion matérielle HP/Dell)

**Avantages** :
- Interface graphique intuitive
- Pas besoin de client spécifique (navigateur web)
- Idéal pour tâches ponctuelles

**Inconvénients** :
- Moins efficace pour tâches répétitives ou automatisation
- Consomme plus de ressources que CLI

##### 5. Ordinateur portable de l'administrateur

> [!success] L'outil universel
> Un **ordinateur portable** est l'outil indispensable de l'administrateur système moderne.

**Utilisations** :
- Connexion SSH aux serveurs
- Accès aux interfaces web d'administration
- Connexion directe via câble série/console
- Documentation et scripts d'automatisation
- Tests et diagnostics réseau

**Configuration recommandée** :
- Système d'exploitation : Linux ou Windows avec WSL
- Clients SSH : OpenSSH, PuTTY, MobaXterm
- Émulateurs de terminal : Terminator, Windows Terminal
- Navigateur web : Firefox, Chromium
- Outils réseau : Wireshark, nmap, iperf

---

### Choix de systèmes d'exploitation

> [!abstract] Diversité des OS
> Un administrateur système doit connaître plusieurs systèmes d'exploitation et savoir choisir le bon outil pour chaque usage.

> [!quote] Citation
> "Une histoire d'habitude"
> 
> Le choix d'un système d'exploitation dépend souvent du contexte de l'entreprise, des habitudes de l'équipe et des compétences disponibles.

#### Systèmes d'exploitation disponibles

Le cours présente visuellement les principaux systèmes d'exploitation :

##### 1. GNU/Linux

> [!info] GNU/Linux
> **Linux** est un noyau libre développé par Linus Torvalds en 1991, combiné avec les outils du projet GNU (Richard Stallman, 1983).

**Caractéristiques** :
- **Libre et open source**
- Très utilisé pour les **serveurs**
- Grande variété de distributions (Debian, Ubuntu, Red Hat, CentOS, Arch...)
- Ligne de commande puissante (bash, shell)
- Gratuit

**Usages typiques** :
- Serveurs web (Apache, Nginx)
- Serveurs de bases de données (MySQL, PostgreSQL)
- Conteneurs et cloud (Docker, Kubernetes)
- Équipements réseau (routeurs, pare-feu)

##### 2. MS Windows

> [!info] Microsoft Windows
> **Windows** est le système d'exploitation propriétaire de Microsoft, dominant sur les postes clients.

**Caractéristiques** :
- **Propriétaire et commercial**
- Très utilisé sur les **postes clients**
- Interface graphique intuitive
- Large compatibilité logicielle
- Windows Server pour les serveurs d'entreprise

**Usages typiques** :
- Postes de travail utilisateurs
- Serveurs Active Directory (annuaire d'entreprise)
- Serveurs de fichiers Windows (partages SMB/CIFS)
- Applications métier Windows

##### 3. BSD (FreeBSD, OpenBSD, NetBSD)

> [!info] BSD - Berkeley Software Distribution
> **BSD** est une famille de systèmes UNIX libres, dérivés du système UNIX de Berkeley.

**Variantes principales** :
- **FreeBSD** : Performance et stabilité (serveurs)
- **OpenBSD** : Sécurité avant tout
- **NetBSD** : Portabilité (fonctionne sur de nombreux matériels)

**Caractéristiques** :
- Très **stable** et **sécurisé**
- Licence BSD (plus permissive que GPL)
- Utilisé dans certains équipements réseau et pare-feu

**Usages typiques** :
- Serveurs haute disponibilité
- Pare-feu (pfSense basé sur FreeBSD)
- Systèmes embarqués

##### 4. macOS (OS X)

> [!info] macOS
> **macOS** est le système d'exploitation d'Apple pour ordinateurs Mac, basé sur un noyau UNIX (Darwin).

**Caractéristiques** :
- **Propriétaire** (Apple)
- Basé sur **BSD/UNIX**
- Interface graphique élégante et intuitive
- Terminal Unix complet
- Matériel Apple uniquement

**Usages typiques** :
- Postes de travail créatifs (graphisme, vidéo, audio)
- Développement logiciel (iOS, macOS)
- Administration système (terminal Unix)

> [!warning] Limitation
> macOS est **exclusivement** disponible sur du matériel Apple, ce qui limite son utilisation en entreprise aux profils spécifiques (designers, développeurs iOS).

#### Tableau comparatif des systèmes d'exploitation

| Système | Type | Licence | Usage principal | Coût | Interface |
|---------|------|---------|-----------------|------|-----------|
| **GNU/Linux** | Libre | GPL/MIT/BSD | Serveurs | Gratuit | CLI + GUI |
| **Windows** | Propriétaire | Commercial | Postes clients | Payant | GUI |
| **Windows Server** | Propriétaire | Commercial | Serveurs entreprise | Payant | GUI + CLI |
| **FreeBSD** | Libre | BSD | Serveurs | Gratuit | CLI + GUI |
| **OpenBSD** | Libre | BSD | Sécurité/Pare-feu | Gratuit | CLI |
| **macOS** | Propriétaire | Commercial | Postes créatifs | Payant (avec Mac) | GUI + CLI |

> [!tip] Conseil pour le TSSR
> En tant que TSSR, tu dois être **polyvalent** :
> - Maîtriser **Linux** (Debian/Ubuntu, Red Hat/CentOS) pour les serveurs
> - Connaître **Windows** (client et serveur) pour l'environnement bureautique
> - Comprendre les bases de **BSD** pour les pare-feu (pfSense)
> - Savoir qu'il existe des alternatives (macOS, autres distributions)

---

## Des logiciels

> [!abstract] Outils logiciels de l'administrateur
> Au-delà du système d'exploitation, l'administrateur utilise de nombreux **logiciels spécialisés** pour accomplir ses tâches quotidiennes.

### Command-Line Interpreter - Shell

> [!quote] Définition
> Un **shell** (ou **interpréteur de commandes**) est un programme qui permet de communiquer avec le système d'exploitation en mode texte, sans interface graphique.

#### En direct avec l'OS

> [!info] Communication directe
> Le shell permet de **communiquer directement avec l'OS** sans passer par une interface graphique.

**Principe de fonctionnement** :
1. L'utilisateur tape une **commande** au clavier
2. Le shell **interprète** la commande
3. Le shell demande à l'**OS d'exécuter** l'action
4. Le résultat est **affiché** à l'écran en mode texte

#### Avantages du shell

> [!success] Pourquoi utiliser le shell ?

| Avantage | Explication |
|----------|-------------|
| **Rapidité** | Taper une commande est souvent plus rapide que naviguer dans des menus graphiques |
| **Faible consommation de ressources** | Pas de chargement d'interface graphique (idéal pour serveurs) |
| **Très efficace** | Automatisation possible (scripts), traitement par lot, pipelines |
| **Reproductible** | Les commandes peuvent être documentées et répétées à l'identique |
| **Administration à distance** | Fonctionne même sur connexion lente (SSH) |

> [!example] Exemple concret
> Créer 100 utilisateurs avec interface graphique : cliquer 100 fois dans des formulaires (long et fastidieux).
> 
> Créer 100 utilisateurs en shell : 1 script de quelques lignes exécuté en quelques secondes.

#### Inconvénients du shell

> [!warning] Limitations

| Inconvénient | Explication |
|--------------|-------------|
| **Peu intuitif de prime abord** | Nécessite de connaître les commandes et leur syntaxe |
| **Courbe d'apprentissage** | Temps d'apprentissage initial plus long qu'une interface graphique |
| **Risque d'erreur** | Une commande mal tapée peut avoir des conséquences graves (suppression de fichiers, etc.) |

> [!tip] Conseil d'apprentissage
> La maîtrise du shell est **essentielle** pour un TSSR. Plus tu pratiqueras, plus tu deviendras efficace. Commence par les commandes de base et progresse graduellement.

#### Shells les plus courants

**Sous Linux/Unix** :
- **bash** (Bourne Again Shell) : le plus répandu sur Linux
- **zsh** (Z Shell) : moderne, puissant, nombreuses fonctionnalités
- **sh** (Bourne Shell) : le shell historique UNIX
- **fish** (Friendly Interactive Shell) : convivial pour débutants

**Sous Windows** :
- **CMD** (Command Prompt) : ancien, limité
- **PowerShell** : moderne, très puissant, orienté objet
- **WSL Bash** (Windows Subsystem for Linux) : bash Linux dans Windows

> [!example] Capture d'écran du cours
> Le cours montre une fenêtre de **terminal** avec un shell en ligne de commande (fond sombre, texte clair), affichant simplement un prompt avec la commande `true` (qui ne fait rien mais retourne un code de succès).

---

### Éditeurs de texte CLI

> [!quote] Définition
> Un **éditeur de texte CLI** (Command-Line Interface) est un logiciel permettant de **créer et éditer des fichiers texte** directement en ligne de commande, sans interface graphique.

#### Une interface texte

> [!info] Pourquoi des éditeurs en mode texte ?
> Sur un serveur sans interface graphique, il est nécessaire de pouvoir **éditer des fichiers de configuration** directement en ligne de commande.

**Cas d'usage typiques** :
- Modifier un fichier de configuration (`/etc/ssh/sshd_config`, `/etc/fstab`, `/etc/hosts`)
- Créer un script shell
- Éditer un fichier de logs
- Modifier un code source directement sur le serveur

#### Les trois principaux éditeurs CLI

Le cours présente visuellement trois éditeurs majeurs :

##### 1. ed / sed

> [!note] ed - L'éditeur historique
> **ed** est l'éditeur de texte originel d'UNIX, créé dans les années 1970.

**Caractéristiques** :
- Très minimaliste
- Ligne par ligne
- Peu utilisé directement aujourd'hui
- **sed** (Stream Editor) en dérive : édition de flux de texte

**Usage moderne** :
- **sed** est très utilisé pour traiter des fichiers en scripts (recherche/remplacement)

```bash
# Exemple sed : remplacer "old" par "new" dans un fichier
sed 's/old/new/g' fichier.txt
```

##### 2. Vim

> [!important] Vim - L'éditeur puissant
> **Vim** (Vi IMproved) est l'éditeur le plus puissant et le plus utilisé par les administrateurs système.

**Caractéristiques** :
- Présent sur **presque tous les systèmes** Linux/Unix
- Très **puissant** et **configurable**
- Modes de fonctionnement : Normal, Insertion, Visuel, Commande
- Courbe d'apprentissage **abrupte**
- Très efficace une fois maîtrisé

**Commandes de base** :
```bash
vim fichier.txt    # Ouvrir un fichier

# Dans Vim :
i                  # Passer en mode insertion (taper du texte)
Esc                # Revenir en mode normal
:w                 # Enregistrer (write)
:q                 # Quitter (quit)
:wq                # Enregistrer et quitter
:q!                # Quitter sans enregistrer
```

> [!tip] Apprendre Vim
> Pour apprendre Vim, tape `vimtutor` dans un terminal. C'est un tutoriel interactif de 30 minutes qui t'apprendra les bases.

**Lien de référence** : [Text Editor Homepage](https://www.vim.org/) (mentionné visuellement dans le cours)

##### 3. Emacs

> [!info] Emacs - L'éditeur extensible
> **Emacs** est un éditeur de texte extrêmement puissant et extensible, presque un système d'exploitation à lui seul.

**Caractéristiques** :
- **Extensible** à l'extrême (langage Emacs Lisp)
- Très **personnalisable**
- Raccourcis clavier basés sur **Ctrl** et **Meta** (Alt)
- Peut faire presque tout (éditeur, IDE, email, calendrier, jeux...)
- Communauté très active

**Commandes de base** :
```bash
emacs fichier.txt  # Ouvrir un fichier

# Dans Emacs :
Ctrl+x Ctrl+s      # Enregistrer
Ctrl+x Ctrl+c      # Quitter
Ctrl+g             # Annuler une commande
```

> [!note] Guerre des éditeurs
> Il existe une "guerre" amicale entre utilisateurs de **Vim** et **Emacs** sur lequel est le meilleur éditeur. Les deux sont excellents, c'est une question de préférence personnelle.

#### Éditeur pour débutants : nano

> [!tip] nano - L'éditeur simple
> **nano** est un éditeur simple et intuitif, idéal pour débuter.

**Avantages** :
- Interface **intuitive** avec aide en bas d'écran
- Pas de modes (on tape directement du texte)
- Parfait pour les modifications simples

**Commandes de base** :
```bash
nano fichier.txt   # Ouvrir un fichier

# Dans nano (indiqué en bas de l'écran) :
Ctrl+O             # Enregistrer (WriteOut)
Ctrl+X             # Quitter (eXit)
Ctrl+K             # Couper une ligne
Ctrl+U             # Coller
```

> [!success] Recommandation pour débuter
> Commence par **nano** pour les éditions simples, puis apprends progressivement **vim** qui est le standard de l'industrie.

#### Tableau comparatif des éditeurs CLI

| Éditeur | Difficulté | Puissance | Disponibilité | Usage recommandé |
|---------|-----------|-----------|---------------|------------------|
| **nano** | ⭐ Facile | ⭐⭐ Basique | Très répandu | Débutants, éditions simples |
| **vim** | ⭐⭐⭐ Difficile | ⭐⭐⭐⭐⭐ Très puissant | Universel | Administration système |
| **emacs** | ⭐⭐⭐⭐ Très difficile | ⭐⭐⭐⭐⭐ Extensible | Courant | Développement, power users |
| **ed/sed** | ⭐⭐ Moyen | ⭐⭐⭐ Spécialisé | Universel | Scripts automatiques |

---

### La boîte à outils de l'administrateur

> [!abstract] Des outils variés pour des tâches variées
> L'administrateur système dispose d'une **boîte à outils** riche et diversifiée pour accomplir ses missions quotidiennes.

#### Catégories d'outils essentiels

Le cours mentionne plusieurs catégories d'outils :

##### 1. Terminal / Émulateur de terminal

> [!info] Le terminal - Interface essentielle
> Un **émulateur de terminal** est un logiciel qui simule un terminal physique et permet d'accéder au shell.

**Exemples mentionnés dans le cours** :
- **xterm** : terminal basique sous X Window (Linux)
- **PuTTY** : client SSH/terminal très populaire sous Windows
- **MobaXterm** : terminal avancé sous Windows (SSH, X11, SFTP intégré)
- **Terminator** : terminal avancé sous Linux (splits, onglets)
- **screen** : multiplexeur de terminal (plusieurs terminaux dans une seule fenêtre)
- **tmux** : alternative moderne à screen

> [!tip] Recommandations par OS
> - **Windows** : MobaXterm (tout-en-un) ou Windows Terminal + OpenSSH
> - **Linux** : Terminator, Tilix, ou le terminal par défaut de ta distribution
> - **macOS** : iTerm2 ou Terminal natif

**Lien mentionné** : [puTTY](https://www.putty.org/), [MobaXterm](https://mobaxterm.mobatek.net/), [terminator](https://gnome-terminator.org/), [screen](https://www.gnu.org/software/screen/)

##### 2. Client SSH

> [!important] SSH - Outil indispensable
> Un **client SSH** permet de se connecter à distance à un serveur de manière sécurisée.

**Clients SSH mentionnés** :
- **OpenSSH** : client/serveur SSH standard sous Linux/Unix/macOS
- **PuTTY** : client SSH graphique sous Windows

**Utilisation basique** :
```bash
# Connexion SSH simple
ssh utilisateur@192.168.1.10

# Connexion SSH sur un port spécifique
ssh -p 2222 utilisateur@serveur.com

# Copie de fichiers via SCP
scp fichier.txt utilisateur@serveur:/chemin/destination/

# Tunnel SSH (redirection de port)
ssh -L 8080:localhost:80 utilisateur@serveur
```

> [!success] SSH est omniprésent
> En tant que TSSR, tu utiliseras SSH **quotidiennement** pour administrer tes serveurs Linux. C'est l'outil #1 de l'administrateur système.

**Lien** : [puTTY](https://www.putty.org/)

##### 3. Navigateur web

> [!info] Navigateur - Administration moderne
> De nombreuses interfaces d'administration modernes sont **web-based** (basées sur le web).

**Navigateurs recommandés** :
- **Firefox** : open source, respectueux de la vie privée
- **Chromium** : version open source de Chrome

**Usages** :
- Accès aux interfaces web des serveurs (Webmin, Cockpit, Proxmox)
- Interfaces des équipements réseau (routeurs, switchs, pare-feu)
- Outils de monitoring (Grafana, Zabbix)
- Documentation en ligne

##### 4. Outils réseaux

> [!info] Diagnostics et analyse réseau
> Les **outils réseau** permettent de diagnostiquer et analyser le trafic et la connectivité réseau.

**Outils essentiels (non détaillés dans ce cours)** :
- **ping** : tester la connectivité réseau
- **traceroute** / **tracert** : tracer le chemin réseau
- **nslookup** / **dig** : requêtes DNS
- **netstat** / **ss** : état des connexions réseau
- **tcpdump** / **Wireshark** : capture et analyse de paquets
- **nmap** : scan de ports et découverte réseau
- **iperf** : test de bande passante

> [!note] Cours dédié
> Les outils réseau font probablement l'objet d'un cours séparé dans ta formation TSSR. Ils sont essentiels pour diagnostiquer les problèmes de connectivité.

##### 5. Outils en ligne de commande

> [!success] La puissance de la ligne de commande
> **Beaucoup, beaucoup d'outils en ligne de commande !**

C'est la citation du cours, qui souligne que l'administrateur système dispose d'une **quantité impressionnante** d'outils CLI pour toutes sortes de tâches.

**Catégories d'outils CLI** :

**Gestion de fichiers** :
- `ls`, `cd`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `touch`
- `find`, `locate`, `which`, `whereis`
- `cat`, `less`, `more`, `head`, `tail`
- `grep`, `sed`, `awk`, `cut`, `sort`, `uniq`

**Gestion système** :
- `top`, `htop`, `ps`, `kill`, `killall`
- `systemctl`, `service` (gestion de services)
- `df`, `du` (espace disque)
- `free` (mémoire)
- `uname`, `hostname`, `uptime`

**Gestion utilisateurs** :
- `useradd`, `usermod`, `userdel`
- `passwd`, `chage`
- `groupadd`, `groupmod`, `groupdel`
- `who`, `w`, `last`

**Permissions** :
- `chmod`, `chown`, `chgrp`
- `umask`

**Réseau** :
- `ip`, `ifconfig` (configuration réseau)
- `ping`, `traceroute`, `mtr`
- `netstat`, `ss` (connexions)
- `curl`, `wget` (téléchargement)

**Archivage / Compression** :
- `tar`, `gzip`, `bzip2`, `xz`
- `zip`, `unzip`

**Et bien d'autres...** 🚀

> [!tip] Apprendre progressivement
> Ne cherche pas à mémoriser tous les outils d'un coup. Apprends-les **au fur et à mesure** de tes besoins et pratiques. Utilise le `man` (manuel) pour comprendre les options de chaque commande :
> ```bash
> man ls    # Affiche le manuel de la commande ls
> ```

#### Récapitulatif de la boîte à outils

> [!success] Les 5 piliers de l'administrateur

| Catégorie | Outils | Usage |
|-----------|--------|-------|
| **Terminal** | xterm, PuTTY, MobaXterm, Terminator, screen | Interface de commande |
| **SSH** | OpenSSH, PuTTY | Connexion à distance sécurisée |
| **Navigateur** | Firefox, Chromium | Interfaces web d'administration |
| **Outils réseau** | ping, traceroute, nmap, Wireshark | Diagnostic et analyse réseau |
| **CLI** | Centaines de commandes | Toutes les tâches d'administration |

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Ordinateurs

- **Deux types** : Postes clients (1 utilisateur, allumés au besoin) vs Serveurs (multi-utilisateurs, allumés 24h/24)
- **Serveurs** : Rarement équipés clavier/écran/souris, administrés à distance
- **Localisation** : Postes clients dans bureaux, serveurs en salle dédiée/datacenter

### Administration serveur

- **Méthodes historiques** : Terminal physique, console système
- **Méthodes modernes** :
  - Switch KVM (basculement entre serveurs)
  - Console série (RJ45/USB avec émulateur terminal)
  - **SSH** (méthode privilégiée pour Linux/Unix)
  - Interface web (administration graphique)
  - Ordinateur portable (outil universel de l'admin)

### Systèmes d'exploitation

- **GNU/Linux** : Libre, gratuit, dominant sur serveurs
- **Windows** : Propriétaire, dominant sur postes clients
- **BSD** (FreeBSD/OpenBSD/NetBSD) : Libre, stable, sécurisé
- **macOS** : Propriétaire Apple, basé UNIX, postes créatifs
- **Choix** : Dépend du contexte, des habitudes et compétences

### Logiciels

- **Shell** : Interface en ligne de commande pour communiquer avec l'OS
  - Avantages : Rapide, efficace, faible consommation, automatisable
  - Inconvénients : Peu intuitif au début
- **Éditeurs CLI** : nano (simple), vim (puissant standard), emacs (extensible)
- **Boîte à outils** : Terminal, SSH, navigateur, outils réseau, nombreuses commandes CLI

### Compétences TSSR essentielles

1. Distinguer postes clients et serveurs
2. Maîtriser SSH pour administration à distance
3. Connaître Linux (serveurs) et Windows (clients)
4. Utiliser efficacement le shell (bash)
5. Éditer des fichiers en ligne de commande (vim/nano)
6. Constituer sa boîte à outils personnalisée

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Poste client** | Ordinateur utilisé par un utilisateur final pour son travail quotidien (PC, laptop, smartphone) |
| **Serveur** | Ordinateur fournissant des services à plusieurs utilisateurs simultanément, allumé en permanence |
| **Console système** | Interface directe (clavier + écran) pour administrer un ordinateur ou serveur |
| **Terminal** | Périphérique ou logiciel permettant d'interagir avec un système en mode texte |
| **Émulateur de terminal** | Logiciel simulant un terminal physique (xterm, PuTTY, MobaXterm) |
| **Switch KVM** | Boîtier matériel permettant de contrôler plusieurs serveurs avec un seul clavier/écran/souris |
| **Console série** | Port de connexion directe (RJ45/USB) pour administrer un serveur sans réseau |
| **SSH** | Secure Shell - Protocole de connexion à distance sécurisée et chiffrée |
| **CLI** | Command-Line Interface - Interface en ligne de commande (mode texte) |
| **GUI** | Graphical User Interface - Interface graphique (fenêtres, boutons, menus) |
| **Shell** | Interpréteur de commandes permettant de communiquer avec l'OS en mode texte |
| **bash** | Bourne Again Shell - Shell le plus répandu sous Linux |
| **Éditeur CLI** | Logiciel d'édition de texte en ligne de commande (vim, nano, emacs) |
| **vim** | Vi IMproved - Éditeur de texte puissant, standard sous Linux/Unix |
| **nano** | Éditeur de texte simple et intuitif pour débutants |
| **emacs** | Éditeur de texte extensible et très puissant |
| **GNU/Linux** | Système d'exploitation libre combinant le noyau Linux et les outils GNU |
| **Distribution Linux** | Variante de Linux (Debian, Ubuntu, Red Hat, CentOS, Arch...) |
| **BSD** | Berkeley Software Distribution - Famille de systèmes UNIX libres |
| **FreeBSD** | Variante BSD orientée performance et stabilité |
| **OpenBSD** | Variante BSD orientée sécurité |
| **macOS** | Système d'exploitation propriétaire d'Apple, basé sur UNIX (Darwin) |
| **Système d'exploitation** | Logiciel de base gérant le matériel et fournissant une interface aux applications |
| **Interface web** | Interface d'administration accessible via un navigateur web |
| **Salle serveurs** | Local dédié abritant les serveurs et équipements réseau de l'entreprise |
| **Datacenter** | Centre de données hébergeant de nombreux serveurs (sur site ou distant) |
| **Administration à distance** | Gestion d'un serveur sans être physiquement devant la machine |
| **Outil réseau** | Logiciel de diagnostic et analyse réseau (ping, traceroute, nmap, Wireshark) |
| **Client SSH** | Logiciel permettant d'établir une connexion SSH (OpenSSH, PuTTY) |
| **Port série** | Port de communication (RS-232) permettant une connexion console directe |

---

## Ressources complémentaires

> [!tip] Atelier pratique

### Atelier Wild Code School

Le cours propose un atelier pratique pour s'entraîner au terminal :

**Lien** : [https://wildcodeschool.github.io/workshop-terminal/README-FR](https://wildcodeschool.github.io/workshop-terminal/README-FR)

> [!example] Contenu de l'atelier
> Cet atelier interactif te permettra de pratiquer les commandes de base du terminal Linux. Il est recommandé de le compléter pour renforcer tes compétences en ligne de commande.

### Documentation et tutoriels

**Shell et ligne de commande** :
- `man bash` : Manuel de bash sur ton système
- [The Linux Command Line (livre gratuit)](http://linuxcommand.org/tlcl.php)
- [ExplainShell](https://explainshell.com/) : Décortique les commandes shell

**Éditeurs CLI** :
- `vimtutor` : Tutoriel interactif de Vim (tape la commande dans un terminal)
- [OpenVim](https://www.openvim.com/) : Tutoriel Vim interactif en ligne
- [GNU Emacs Manual](https://www.gnu.org/software/emacs/manual/)

**SSH** :
- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [PuTTY Documentation](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html)

**Systèmes d'exploitation** :
- [Debian Administrator's Handbook](https://debian-handbook.info/)
- [The FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/)

---

## Conclusion

> [!success] Les fondamentaux de l'administrateur système

Ce cours a présenté les **éléments de base** dont dispose un administrateur système et réseau :

### Ce que tu dois retenir

1. **Types d'ordinateurs** :
   - Postes clients pour utilisateurs finaux
   - Serveurs pour services partagés (allumés 24h/24)

2. **Méthodes d'administration** :
   - Console physique (historique)
   - SSH (méthode moderne privilégiée)
   - Interface web (pour tâches ponctuelles)
   - Ordinateur portable (outil universel)

3. **Systèmes d'exploitation** :
   - Linux dominant sur serveurs
   - Windows dominant sur postes clients
   - BSD pour systèmes critiques
   - macOS pour créatifs

4. **Outils essentiels** :
   - Shell (bash) pour administration en ligne de commande
   - Éditeurs CLI (vim/nano) pour éditer fichiers de config
   - SSH pour connexion à distance
   - Terminal / Navigateur web
   - Centaines de commandes CLI

### Prochaines étapes

> [!tip] Développer tes compétences
> - **Pratique** l'atelier terminal proposé
> - **Installe** une VM Linux et entraîne-toi aux commandes
> - **Apprends** progressivement vim (commence par vimtutor)
> - **Configure** ton environnement de travail (terminal, SSH, outils)
> - **Documente** tes configurations et scripts

> [!quote] Citation finale
> "L'administrateur système efficace est celui qui maîtrise ses outils et sait choisir le bon outil pour chaque tâche."

**La ligne de commande est ton alliée. Pratique, pratique, pratique !** 🚀💻

---

*Document créé pour la préparation au titre RNCP Technicien Supérieur Systèmes et Réseaux (TSSR)*  
*Compatible Obsidian avec callouts natifs*
