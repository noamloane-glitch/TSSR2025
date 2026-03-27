# 📚 Plan de Formation - Administration Windows

---

## 📘 PARTIE 1 - Fondamentaux des Systèmes Windows
Dossier Obsidian suggéré : `01-fondamentaux-windows/`

**Sujets à couvrir :**

1. Introduction aux systèmes Windows → `01-introduction-systemes-windows.md`
   - Histoire et évolution de Windows
   - Versions Windows Desktop (10, 11)
   - Versions Windows Server (2016, 2019, 2022)
   - Éditions et licences
   - Architecture matérielle requise

2. Architecture du système Windows → `02-architecture-systeme.md`
   - Architecture en couches (User Mode / Kernel Mode)
   - Le noyau Windows NT
   - Processus et threads
   - Gestion de la mémoire
   - Hardware Abstraction Layer (HAL)

3. Le registre Windows → `03-registre-windows.md`
   - Structure du registre (HKEY_*)
   - Types de données du registre
   - Manipulation avec regedit
   - Manipulation avec PowerShell
   - Sauvegarde et restauration
   - Bonnes pratiques et précautions

4. Systèmes de fichiers → `04-systemes-fichiers.md`
   - NTFS : Caractéristiques et fonctionnalités
   - ReFS : Utilisation et différences
   - FAT32 et exFAT
   - Permissions et ACL
   - Compression et chiffrement EFS
   - Quotas de disque

---

## 📘 PARTIE 2 - Installation et Configuration
Dossier Obsidian suggéré : `02-installation-configuration/`

**Sujets à couvrir :**

1. Installation de Windows Desktop → `01-installation-windows-desktop.md`
   - Prérequis matériels et logiciels
   - Création de média d'installation
   - Installation propre vs mise à niveau
   - Partitionnement de disques
   - Configuration initiale (OOBE)
   - Activation et licences

2. Installation de Windows Server → `02-installation-windows-server.md`
   - Server Core vs Desktop Experience
   - Méthodes d'installation (ISO, réseau, WDS)
   - Configuration post-installation
   - Server Manager
   - Sconfig pour Server Core
   - Conversion Core ↔ GUI

3. Configuration initiale du système → `03-configuration-initiale.md`
   - Nom d'ordinateur et groupe de travail
   - Configuration réseau (IP, DNS, passerelle)
   - Fuseau horaire et paramètres régionaux
   - Windows Update
   - Pare-feu Windows
   - Bureau à distance (RDP)

4. Gestion des pilotes de périphériques → `04-gestion-pilotes.md`
   - Device Manager
   - Installation et mise à jour de pilotes
   - Signature de pilotes
   - Dépannage des périphériques
   - Outils PnPUtil et DISM

---

## 📘 PARTIE 3 - Gestion des Utilisateurs et Sécurité
Dossier Obsidian suggéré : `03-utilisateurs-securite/`

**Sujets à couvrir :**

1. Comptes utilisateurs locaux → `01-comptes-utilisateurs-locaux.md`
   - Types de comptes (Administrateur, Standard, Invité)
   - Création et gestion via GUI
   - Création et gestion via PowerShell
   - Comptes intégrés (Administrator, Guest)
   - Profils utilisateurs locaux
   - Stratégies de mots de passe locales

2. Groupes locaux → `02-groupes-locaux.md`
   - Groupes intégrés et leurs droits
   - Création de groupes personnalisés
   - Gestion des appartenances
   - Groupes vs Utilisateurs
   - Principe du moindre privilège

3. Contrôle de compte utilisateur (UAC) → `03-uac.md`
   - Fonctionnement de l'UAC
   - Niveaux de sécurité UAC
   - Invite d'élévation
   - Configuration et désactivation (risques)
   - Contournement légitime

4. Authentification et autorisation → `04-authentification-autorisation.md`
   - Différence authentification vs autorisation
   - Types d'authentification (locale, domaine)
   - Méthodes d'authentification (password, PIN, biométrie)
   - Windows Hello
   - LAPS (Local Administrator Password Solution)

5. Stratégies de sécurité locales → `05-strategies-securite-locales.md`
   - Local Security Policy
   - Stratégies de mots de passe
   - Stratégies de verrouillage de compte
   - Stratégies d'audit
   - Attribution des droits utilisateur
   - Options de sécurité

6. BitLocker et chiffrement → `06-bitlocker-chiffrement.md`
   - Principe de BitLocker
   - BitLocker To Go (USB)
   - Configuration et activation
   - TPM (Trusted Platform Module)
   - Clés de récupération
   - Gestion et déverrouillage

---

## 📘 PARTIE 4 - Gestion du Réseau
Dossier Obsidian suggéré : `04-gestion-reseau/`

**Sujets à couvrir :**

1. Configuration réseau TCP/IP → `01-configuration-tcpip.md`
   - Adressage IPv4 et IPv6
   - Configuration IP statique vs DHCP
   - Masque de sous-réseau et passerelle
   - Serveurs DNS
   - Configuration via GUI et PowerShell
   - Commandes réseau de base (ipconfig, ping, tracert)

2. Cartes réseau et pilotes → `02-cartes-reseau.md`
   - Gestion des cartes réseau physiques
   - Cartes réseau virtuelles
   - Teaming et agrégation de liens
   - Configuration avancée (MTU, VLAN)
   - Dépannage des cartes réseau

3. Pare-feu Windows Defender → `03-parefeu-windows.md`
   - Profils de pare-feu (Domaine, Privé, Public)
   - Règles entrantes et sortantes
   - Configuration via GUI (Windows Defender Firewall)
   - Configuration via PowerShell (NetSecurity)
   - Journalisation et dépannage
   - Bonnes pratiques de sécurité

4. Partage de fichiers et imprimantes → `04-partage-fichiers-imprimantes.md`
   - Partages SMB/CIFS
   - Permissions de partage vs permissions NTFS
   - Partages administratifs ($)
   - Gestion des sessions et fichiers ouverts
   - Partage d'imprimantes réseau
   - Mappage de lecteurs réseau

5. Résolution de noms → `05-resolution-noms.md`
   - Fichier hosts
   - DNS client
   - Cache DNS
   - NetBIOS et WINS (legacy)
   - Commandes de diagnostic (nslookup, Resolve-DnsName)

---

## 📘 PARTIE 5 - Active Directory - Concepts et Gestion
Dossier Obsidian suggéré : `05-active-directory/`

**Sujets à couvrir :**

1. Introduction à Active Directory → `01-introduction-active-directory.md`
   - Qu'est-ce qu'Active Directory
   - Forêt, arborescence, domaine
   - Contrôleurs de domaine (DC)
   - Avantages d'Active Directory
   - LDAP et Kerberos

2. Installation d'Active Directory Domain Services → `02-installation-adds.md`
   - Prérequis pour un DC
   - Promotion d'un serveur en DC
   - Création d'une nouvelle forêt
   - Ajout d'un DC à un domaine existant
   - Niveaux fonctionnels de forêt et domaine
   - DNS et Active Directory

3. Unités d'organisation (OU) → `03-unites-organisation.md`
   - Concept des OU
   - Création et structure d'OU
   - Délégation de contrôle
   - Bonnes pratiques d'organisation
   - OU vs groupes de sécurité

4. Utilisateurs et groupes de domaine → `04-utilisateurs-groupes-domaine.md`
   - Création d'utilisateurs de domaine
   - Attributs utilisateurs (UPN, SamAccountName)
   - Groupes de sécurité et de distribution
   - Étendues de groupe (Local, Global, Universal)
   - Stratégies de nommage
   - Groupes intégrés du domaine

5. Comptes ordinateurs → `05-comptes-ordinateurs.md`
   - Jonction au domaine (Windows 10/11)
   - Jonction au domaine (Windows Server)
   - Gestion des comptes ordinateurs dans AD
   - Réinitialisation de comptes ordinateurs
   - Déplacement entre OU

6. Stratégies de groupe (GPO) - Fondamentaux → `06-gpo-fondamentaux.md`
   - Concept des GPO
   - Structure d'une GPO
   - Liaison de GPO (Link) aux OU
   - Héritage et ordre de traitement
   - Forcer et bloquer l'héritage
   - Filtrage de sécurité
   - Outils de gestion (GPMC)

7. Stratégies de groupe (GPO) - Configuration → `07-gpo-configuration.md`
   - Configuration ordinateur vs utilisateur
   - Paramètres de sécurité via GPO
   - Redirection de dossiers
   - Scripts de démarrage/arrêt/ouverture/fermeture de session
   - Préférences de stratégie de groupe
   - Installation de logiciels via GPO

8. Dépannage Active Directory → `08-depannage-ad.md`
   - Outils de diagnostic (dcdiag, repadmin)
   - Problèmes de réplication
   - Résultats de stratégie de groupe (gpresult, rsop.msc)
   - Journaux d'événements AD
   - Synchronisation horaire (W32Time)

---

## 📘 PARTIE 6 - Services Réseau Windows
Dossier Obsidian suggéré : `06-services-reseau/`

**Sujets à couvrir :**

1. Service DNS → `01-service-dns.md`
   - Rôle DNS Server
   - Zones DNS (primaire, secondaire, stub)
   - Enregistrements DNS (A, AAAA, CNAME, MX, SRV, PTR)
   - DNS et Active Directory (zones intégrées AD)
   - Transferts de zone
   - Redirecteurs DNS
   - Dépannage DNS

2. Service DHCP → `02-service-dhcp.md`
   - Rôle DHCP Server
   - Étendues et plages d'adresses
   - Réservations DHCP
   - Options DHCP (routeur, DNS, domaine)
   - Bail DHCP
   - Haute disponibilité DHCP (failover)
   - Dépannage DHCP

3. Service de fichiers DFS → `03-service-dfs.md`
   - DFS Namespaces
   - DFS Replication
   - Configuration d'un namespace
   - Cibles de dossier
   - Réplication bidirectionnelle
   - Topologies de réplication

4. Service d'impression → `04-service-impression.md`
   - Print Server role
   - Installation d'imprimantes réseau
   - Gestion des pilotes d'imprimantes
   - Files d'attente et priorités
   - Déploiement via GPO
   - Suivi et rapports d'impression

5. Service de déploiement Windows (WDS) → `05-service-wds.md`
   - Rôle WDS
   - Images de démarrage (boot) et d'installation
   - Configuration de WDS
   - PXE Boot
   - Déploiement réseau de Windows
   - Intégration avec MDT

---

## 📘 PARTIE 7 - PowerShell pour l'Administration
Dossier Obsidian suggéré : `07-powershell/`

**Sujets à couvrir :**

1. Introduction à PowerShell → `01-introduction-powershell.md`
   - Qu'est-ce que PowerShell
   - PowerShell vs CMD
   - Versions de PowerShell (5.1, 7+)
   - Console vs ISE vs VS Code
   - Syntaxe de base

2. Cmdlets et pipeline → `02-cmdlets-pipeline.md`
   - Structure Verb-Noun
   - Get-Command, Get-Help, Get-Member
   - Pipeline (|)
   - Objets .NET
   - Filtrage avec Where-Object
   - Sélection avec Select-Object

3. Variables et types de données → `03-variables-types.md`
   - Déclaration de variables
   - Types de données (String, Int, Array, Hashtable)
   - Variables automatiques ($_, $PSItem, $env:)
   - Variables de préférence

4. Structures de contrôle → `04-structures-controle.md`
   - Conditions (if, else, elseif, switch)
   - Boucles (for, foreach, while, do-while)
   - Opérateurs de comparaison
   - Opérateurs logiques

5. Fonctions et scripts → `05-fonctions-scripts.md`
   - Création de fonctions
   - Paramètres et arguments
   - Return et sortie
   - Scripts .ps1
   - Execution policy
   - Dot sourcing

6. Modules PowerShell → `06-modules-powershell.md`
   - Qu'est-ce qu'un module
   - Modules intégrés (ActiveDirectory, NetAdapter, etc.)
   - Installation de modules (PowerShell Gallery)
   - Import-Module
   - Création de modules personnalisés

7. Gestion à distance avec PowerShell → `07-gestion-distance.md`
   - PowerShell Remoting
   - Enable-PSRemoting
   - Enter-PSSession
   - Invoke-Command
   - Sessions persistantes (New-PSSession)
   - WinRM et sécurité

8. Automatisation courante → `08-automatisation-courante.md`
   - Gestion des utilisateurs AD
   - Gestion des groupes AD
   - Gestion des ordinateurs
   - Gestion des services Windows
   - Gestion du système de fichiers
   - Interrogation du registre
   - Planification de tâches

---

## 📘 PARTIE 8 - Virtualisation avec Hyper-V
Dossier Obsidian suggéré : `08-hyperv/`

**Sujets à couvrir :**

1. Introduction à Hyper-V → `01-introduction-hyperv.md`
   - Qu'est-ce que la virtualisation
   - Hyperviseur Type 1 vs Type 2
   - Hyper-V sur Windows Server
   - Hyper-V sur Windows 10/11 Pro
   - Prérequis matériels (VT-x/AMD-V, SLAT)

2. Installation et configuration d'Hyper-V → `02-installation-configuration.md`
   - Activation du rôle Hyper-V
   - Hyper-V Manager
   - Configuration de l'hôte
   - Chemins par défaut
   - Migration en direct (configuration)

3. Commutateurs virtuels → `03-commutateurs-virtuels.md`
   - Types de commutateurs (Externe, Interne, Privé)
   - Création de vSwitch
   - VLAN tagging
   - Équipes NIC sur l'hôte
   - Gestion via PowerShell

4. Création et gestion de machines virtuelles → `04-creation-gestion-vm.md`
   - Assistant de création de VM
   - Génération 1 vs Génération 2
   - Configuration matérielle virtuelle (CPU, RAM, disques)
   - Mémoire dynamique
   - Points de contrôle (snapshots)
   - Export et import de VM

5. Disques virtuels → `05-disques-virtuels.md`
   - Formats VHD vs VHDX
   - Disques dynamiques vs fixes vs différentiels
   - Création et gestion de VHD/VHDX
   - Montage de disques virtuels
   - Redimensionnement et conversion
   - Disques pass-through

6. Intégration Services et Enhanced Session → `06-integration-services.md`
   - Composants Integration Services
   - Installation dans les VM invités
   - Enhanced Session Mode
   - Partage du presse-papier
   - Redirection de périphériques
   - Time synchronization

7. Haute disponibilité Hyper-V → `07-haute-disponibilite.md`
   - Clustering de basculement pour Hyper-V
   - Stockage partagé (CSV)
   - Migration en direct (Live Migration)
   - Migration de stockage
   - Réplication Hyper-V
   - Scénarios de disaster recovery

---

## 📘 PARTIE 9 - Surveillance et Performance
Dossier Obsidian suggéré : `09-surveillance-performance/`

**Sujets à couvrir :**

1. Gestionnaire des tâches → `01-gestionnaire-taches.md`
   - Onglet Processus
   - Onglet Performances
   - Onglet Utilisateurs
   - Onglet Services
   - Onglet Démarrage
   - Analyse des ressources

2. Moniteur de ressources → `02-moniteur-ressources.md`
   - Vue d'ensemble
   - CPU et threads
   - Mémoire et processus
   - Disque et E/S
   - Réseau et connexions
   - Identification des goulots d'étranglement

3. Analyseur de performances → `03-analyseur-performances.md`
   - Compteurs de performances
   - Création de rapports personnalisés
   - Ensemble de collecteurs de données
   - Alertes de performances
   - Analyse historique
   - Compteurs Windows et applications

4. Observateur d'événements → `04-observateur-evenements.md`
   - Structure des journaux (Application, Sécurité, Système)
   - Types d'événements (Information, Avertissement, Erreur, Critique)
   - Filtrage d'événements
   - Abonnements aux événements
   - Journaux personnalisés
   - Corrélation avec Event ID

5. Outils de diagnostic système → `05-outils-diagnostic.md`
   - Informations système (msinfo32)
   - Moniteur de fiabilité
   - Rapports de diagnostic
   - Windows Memory Diagnostic
   - Vérificateur de fichiers système (sfc)
   - DISM (Deployment Image Servicing)

6. Optimisation des performances → `06-optimisation-performances.md`
   - Gestion des services de démarrage
   - Défragmentation et optimisation de disques
   - Nettoyage de disque
   - ReadyBoost (si applicable)
   - Fichier d'échange (pagefile)
   - Indexation Windows Search
   - Effets visuels

---

## 📘 PARTIE 10 - Sauvegarde et Récupération
Dossier Obsidian suggéré : `10-sauvegarde-recuperation/`

**Sujets à couvrir :**

1. Stratégie de sauvegarde → `01-strategie-sauvegarde.md`
   - Types de sauvegardes (complète, incrémentielle, différentielle)
   - Règle 3-2-1
   - RPO et RTO
   - Planification des sauvegardes
   - Rotation des médias
   - Tests de restauration

2. Windows Server Backup → `02-windows-server-backup.md`
   - Installation de la fonctionnalité
   - Configuration de sauvegardes planifiées
   - Sauvegarde manuelle unique
   - Destinations de sauvegarde (disque local, réseau)
   - Bare Metal Recovery (BMR)
   - Restauration de fichiers et volumes

3. Historique des fichiers (Windows Desktop) → `03-historique-fichiers.md`
   - Configuration de l'historique des fichiers
   - Destinations de sauvegarde
   - Fréquence et rétention
   - Restauration de fichiers
   - Exclusions

4. Clichés instantanés (Shadow Copies) → `04-cliches-instantanes.md`
   - Volume Shadow Copy Service (VSS)
   - Configuration sur volumes partagés
   - Planification des clichés
   - Restauration via "Versions précédentes"
   - Espace disque et rétention

5. Points de restauration système → `05-points-restauration.md`
   - Configuration de la protection système
   - Création manuelle de points
   - Restauration du système
   - Annulation de restauration
   - Limitations et bonnes pratiques

6. Récupération et dépannage → `06-recuperation-depannage.md`
   - Options de démarrage avancées
   - Mode sans échec
   - Environnement de récupération Windows (WinRE)
   - Réparation du démarrage
   - Invite de commandes de récupération
   - Réinitialisation et actualisation de Windows

---

## 📘 PARTIE 11 - Sécurité Avancée
Dossier Obsidian suggéré : `11-securite-avancee/`

**Sujets à couvrir :**

1. Windows Defender et antivirus → `01-windows-defender.md`
   - Windows Defender Antivirus
   - Analyses (rapide, complète, personnalisée)
   - Mises à jour des définitions
   - Exclusions
   - Protection en temps réel
   - Windows Defender Offline

2. Windows Defender Firewall avancé → `02-firewall-avance.md`
   - Console MMC (wf.msc)
   - Règles entrantes/sortantes détaillées
   - Règles de sécurité de connexion
   - IPsec et authentification
   - Profils multiples
   - Import/export de règles

3. AppLocker et contrôle d'applications → `03-applocker.md`
   - Contrôle d'applications avec AppLocker
   - Règles par éditeur, chemin, hash
   - Stratégies par défaut
   - Mode audit vs application
   - WDAC (Windows Defender Application Control)

4. Audit et journalisation de sécurité → `04-audit-journalisation.md`
   - Configuration des stratégies d'audit
   - Audit avancé
   - Journaux de sécurité
   - Analyse des connexions
   - Détection d'anomalies
   - SIEM et corrélation

5. Durcissement de Windows → `05-durcissement-windows.md`
   - Recommandations ANSSI/CIS
   - Désactivation de services inutiles
   - Réduction de la surface d'attaque
   - Configuration des protocoles réseau
   - SMBv1 et protocoles legacy
   - Sécurité du protocole RDP

6. Gestion des certificats → `06-gestion-certificats.md`
   - PKI (Public Key Infrastructure)
   - Magasin de certificats Windows
   - Certificats personnels vs racine de confiance
   - Importation et exportation
   - Renouvellement de certificats
   - Révocation (CRL, OCSP)

7. Credential Guard et Device Guard → `07-credential-device-guard.md`
   - Windows Defender Credential Guard
   - Protection des informations d'identification
   - Virtualization-based Security (VBS)
   - Device Guard / WDAC
   - Secure Boot et TPM
   - Configuration et prérequis

---

## 📘 PARTIE 12 - Bureaux à Distance et Accès Distant
Dossier Obsidian suggéré : `12-bureaux-distance/`

**Sujets à couvrir :**

1. Bureau à distance (RDP) → `01-bureau-distance-rdp.md`
   - Activation du Bureau à distance
   - Client Bureau à distance (mstsc)
   - Connexion RDP de base
   - Options de connexion avancées
   - Fichiers .rdp
   - Redirection de ressources (lecteurs, imprimantes, audio)
   - Sécurité RDP (NLA, encryption)

2. Remote Desktop Services → `02-remote-desktop-services.md`
   - Architecture RDS
   - Session Host
   - Connection Broker
   - Web Access
   - RemoteApp
   - Licensing
   - Passerelle RD (RD Gateway)

3. Windows Admin Center → `03-windows-admin-center.md`
   - Présentation de WAC
   - Installation et configuration
   - Gestion de serveurs via WAC
   - Gestion de clusters
   - Extension et plugins
   - Administration via navigateur web

4. VPN Windows → `04-vpn-windows.md`
   - Types de VPN (PPTP, L2TP/IPsec, SSTP, IKEv2)
   - Configuration client VPN
   - Profils VPN
   - Always On VPN
   - Serveur VPN Windows Server (RRAS)
   - Authentification RADIUS

5. DirectAccess → `05-directaccess.md`
   - Concept de DirectAccess
   - Différences avec VPN traditionnel
   - Architecture et composants
   - Configuration serveur
   - Configuration client
   - Dépannage DirectAccess

---

## 📘 PARTIE 13 - Clustering et Haute Disponibilité
Dossier Obsidian suggéré : `13-clustering-ha/`

**Sujets à couvrir :**

1. Introduction au clustering → `01-introduction-clustering.md`
   - Qu'est-ce qu'un cluster
   - Types de clusters (Failover, Load Balancing)
   - Concepts : Quorum, témoin, heartbeat
   - Scénarios de haute disponibilité
   - Limitations et coûts

2. Failover Clustering → `02-failover-clustering.md`
   - Prérequis matériels et logiciels
   - Installation du rôle Failover Clustering
   - Validation du cluster
   - Création d'un cluster de basculement
   - Configuration du quorum
   - Rôles clusterisés (File Server, Hyper-V, SQL)

3. Stockage pour clusters → `03-stockage-clusters.md`
   - Types de stockage partagé (SAN, iSCSI, SMB 3.0)
   - Cluster Shared Volumes (CSV)
   - Storage Spaces Direct
   - Témoin de partage de fichiers
   - Témoin de disque
   - Témoin cloud Azure

4. Network Load Balancing → `04-network-load-balancing.md`
   - Concept de NLB
   - Configuration de NLB
   - Modes de distribution (unicast, multicast)
   - Affinité de sessions
   - Scénarios d'utilisation (serveurs web)
   - Différences avec Failover Clustering

---

## 📘 PARTIE 14 - Maintenance et Mises à Jour
Dossier Obsidian suggéré : `14-maintenance-maj/`

**Sujets à couvrir :**

1. Windows Update → `01-windows-update.md`
   - Types de mises à jour (qualité, fonctionnalités, pilotes)
   - Configuration de Windows Update
   - Heures d'activité et planification
   - Historique des mises à jour
   - Résolution des problèmes de mises à jour
   - Désinstallation de mises à jour

2. Windows Server Update Services (WSUS) → `02-wsus.md`
   - Rôle WSUS
   - Architecture WSUS
   - Installation et configuration
   - Synchronisation avec Microsoft Update
   - Approbation et déploiement de mises à jour
   - Groupes d'ordinateurs
   - Rapports WSUS

3. Maintenance du système → `03-maintenance-systeme.md`
   - Tâches de maintenance planifiées
   - Nettoyage de disque automatique
   - Défragmentation planifiée
   - Vérification de l'intégrité des fichiers
   - Surveillance de l'espace disque
   - Rotation des journaux

4. DISM et réparation d'image → `04-dism-reparation.md`
   - Deployment Image Servicing and Management
   - DISM /Online vs /Image
   - Vérification de l'intégrité (CheckHealth, ScanHealth)
   - Réparation d'image (RestoreHealth)
   - Sources de réparation
   - Intégration avec SFC

---

## 📘 PARTIE 15 - Dépannage et Résolution de Problèmes
Dossier Obsidian suggéré : `15-depannage/`

**Sujets à couvrir :**

1. Méthodologie de dépannage → `01-methodologie-depannage.md`
   - Approche structurée de résolution
   - Identifier le problème
   - Établir une théorie de cause probable
   - Tester la théorie
   - Établir un plan d'action
   - Vérifier la fonctionnalité complète
   - Documentation

2. Dépannage du démarrage → `02-depannage-demarrage.md`
   - Séquence de démarrage Windows
   - POST et BIOS/UEFI
   - Chargeur de démarrage (BOOTMGR, BCD)
   - Erreurs courantes de démarrage
   - Outils de récupération (WinRE)
   - Réparation du BCD
   - bootrec et bcdedit

3. Dépannage des performances → `03-depannage-performances.md`
   - Identification des goulots (CPU, RAM, Disque, Réseau)
   - Processus consommateurs de ressources
   - Fuites de mémoire
   - Problèmes de disque (E/S élevées)
   - Latence réseau
   - Outils de diagnostic de performances

4. Dépannage réseau → `04-depannage-reseau.md`
   - Pas de connectivité réseau
   - Diagnostic des cartes réseau
   - Problèmes DHCP
   - Problèmes DNS
   - Connectivité intermittente
   - Outils (ping, tracert, pathping, nslookup, Test-NetConnection)

5. Dépannage Active Directory → `05-depannage-ad.md`
   - Problèmes d'authentification
   - Réplication AD en échec
   - Problèmes de GPO non appliquées
   - Problèmes de DNS et AD
   - Outils (dcdiag, repadmin, nltest, gpresult)
   - Journaux d'événements AD

6. Blue Screen of Death (BSOD) → `06-bsod.md`
   - Compréhension des BSOD
   - Codes d'erreur communs
   - Analyse de fichiers dump (minidump)
   - WinDbg basics
   - Causes fréquentes (pilotes, matériel, RAM)
   - Prévention et résolution

7. Outils en ligne de commande → `07-outils-ligne-commande.md`
   - CMD vs PowerShell
   - Commandes réseau (ipconfig, netstat, nslookup, route)
   - Commandes système (sfc, chkdsk, tasklist, taskkill)
   - Commandes de fichiers (robocopy, xcopy, attrib)
   - Commandes AD (net user, net group, dsquery)
   - Scripts batch utiles

---

## 📘 PARTIE 16 - Projet Pratique Intégrateur
Dossier Obsidian suggéré : `16-projet-pratique/`

**Sujets à couvrir :**

1. Cahier des charges → `01-cahier-charges.md`
   - Contexte de l'entreprise fictive
   - Besoins identifiés
   - Contraintes techniques
   - Livrables attendus
   - Planning de réalisation

2. Conception de l'infrastructure → `02-conception-infrastructure.md`
   - Architecture réseau
   - Plan d'adressage IP
   - Structure Active Directory
   - Choix technologiques
   - Schémas et documentation

3. Mise en œuvre → `03-mise-en-oeuvre.md`
   - Installation des serveurs
   - Configuration AD DS
   - Déploiement des services (DNS, DHCP, etc.)
   - Configuration des GPO
   - Tests fonctionnels

4. Documentation finale → `04-documentation-finale.md`
   - Documentation technique
   - Procédures d'exploitation
   - Guide utilisateur
   - Plan de sauvegarde et de récupération
   - Bilan du projet

---

