# DHCP (Dynamic Host Configuration Protocol)

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : DHCP - Configuration automatique des réseaux IP

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Problématique de l'adressage statique|Problématique de l'adressage statique]]
   - [[#Limites de l'adressage statique|Limites de l'adressage statique]]
   - [[#Définition du DHCP|Définition du DHCP]]
   - [[#Objectifs du DHCP|Objectifs du DHCP]]

2. [[#Fonctionnement général|Fonctionnement général]]
   - [[#Architecture client-serveur|Architecture client-serveur]]
   - [[#Le client DHCP|Le client DHCP]]
   - [[#Identification des clients|Identification des clients]]
   - [[#Le serveur DHCP|Le serveur DHCP]]
   - [[#Paramètres obligatoires et optionnels|Paramètres obligatoires et optionnels]]
   - [[#Synthèse du fonctionnement|Synthèse du fonctionnement]]

3. [[#Séquence DHCP|Séquence DHCP]]
   - [[#Les messages DHCP|Les messages DHCP]]
   - [[#Séquence DORA|Séquence DORA]]
   - [[#Messages complémentaires|Messages complémentaires]]
   - [[#Cas d'usage des demandes|Cas d'usage des demandes]]

4. [[#Fonctionnalités avancées|Fonctionnalités avancées]]
   - [[#Paramètres de configuration|Paramètres de configuration]]
   - [[#Pour aller plus loin|Pour aller plus loin]]

5. [[#Points clés à retenir|Points clés à retenir]]

6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le **DHCP** (Dynamic Host Configuration Protocol) est un protocole réseau essentiel qui permet l'attribution **automatique** de paramètres réseau IP aux machines d'un réseau. Il résout les problèmes liés à la configuration manuelle des adresses IP.

### Pourquoi étudier le DHCP ?

En tant que **TSSR**, tu dois comprendre :
- Comment les adresses IP sont attribuées automatiquement sur un réseau
- Le fonctionnement du protocole client-serveur DHCP
- La séquence d'échanges entre client et serveur (DORA)
- La configuration et le dépannage d'un serveur DHCP
- La gestion des baux et des réservations

---

### Problématique de l'adressage statique

> [!info] Le problème initial
> Dans **IPv4**, il n'existe pas de mécanisme de configuration automatique des interfaces réseau prévu dans le protocole de base.

**Conséquence :**
- Configuration **manuelle** nécessaire
- Utilisation de l'**adressage statique** (configuration fixe sur chaque machine)

> [!quote] Configuration manuelle
> Sans DHCP, chaque machine doit être configurée individuellement avec :
> - Son adresse IP
> - Son masque de sous-réseau
> - Sa passerelle par défaut
> - Ses serveurs DNS
> - Éventuellement d'autres paramètres réseau

**Contexte historique :**
- À l'origine d'Internet, les réseaux étaient petits et stables
- La configuration manuelle était gérable
- Avec la croissance exponentielle des réseaux, cette approche est devenue obsolète

---

### Limites de l'adressage statique

> [!warning] Problèmes majeurs de l'adressage statique
> L'adressage statique présente de nombreuses limitations qui le rendent inadapté aux réseaux modernes.

**Problèmes identifiés :**

| Problème | Impact |
|----------|--------|
| **Erreurs humaines** | Très fréquentes lors de la saisie manuelle |
| **Temps d'administration** | Configuration individuelle de chaque machine |
| **Conflits d'adresses IP** | Difficile d'éviter les doublons (deux machines avec la même IP) |
| **Changements de plan d'adressage** | Risque majeur lors d'une modification globale |
| **Documentation** | Complexité de la mise à jour et maintien de la cohérence |
| **Passage à l'échelle** | Impossible à gérer pour de grands parcs machines |

> [!important] Problème du passage à l'échelle
> La configuration manuelle ne permet pas de gérer efficacement des réseaux comportant de nombreuses machines, surtout si elles sont nomades ou évoluent fréquemment.

**Dépendances et contraintes :**
- **Dépendance aux administrateurs** : intervention humaine pour chaque ajout/modification
- **Temps de déploiement** : lent et coûteux en ressources humaines
- **Risque de dysfonctionnement** : une erreur peut paralyser une machine ou créer des conflits réseau

---

### Limites de l'adressage statique (suite)

> [!example] Exemple concret de problème
> Méthode manuelle évidemment **"impossible"** dans le cas de **large parc de machines** éventuellement **nomades** et **évoluant dans le temps** !

**Scénario type :**

```
Réseau d'entreprise en 192.168.0.0/24 comportant déjà 50 machines.
Ajout de 10 nouvelles machines.

Problème : Comment attribuer les adresses ?
```

**Étapes nécessaires en adressage statique :**

1. **Vérifier les adresses déjà utilisées** :
   - Consultation manuelle de la documentation (si elle existe et est à jour)
   - Scan réseau (nmap, arp-scan, etc.)
   - Vérification auprès des collègues
   - Risque : la doc n'est jamais complètement à jour

2. **Identifier les adresses disponibles** :
   - Dans une plage /24, 254 adresses utilisables
   - 50 déjà utilisées = 204 disponibles
   - Mais lesquelles exactement ?

3. **Attribuer manuellement** :
   - Se connecter sur chaque nouvelle machine
   - Configurer l'IP, le masque, la passerelle, les DNS
   - Tester la connectivité
   - Mettre à jour la documentation

4. **Gérer les erreurs** :
   - Conflit d'IP découvert après coup
   - Obligation de reconfigurer
   - Perte de temps et frustration

> [!warning] Machines nomades
> Dans le cas de machines portables qui se connectent à différents réseaux (bureau, domicile, clients), l'adressage statique impose de reconfigurer à chaque changement de réseau !

**Bilan :**
- **Temps d'administration** : plusieurs heures pour 10 machines
- **Risque d'erreur** : élevé
- **Maintenabilité** : très faible
- **Solution** : DHCP !

---

### Définition du DHCP

> [!quote] Définition officielle
> Le **DHCP** (Dynamic Host Configuration Protocol) est défini par les **RFC 2131** et **RFC 2132**. Il permet l'attribution **automatique** de paramètres réseau IP aux machines.

**Standards et évolution :**

| Version | RFC | Description |
|---------|-----|-------------|
| **DHCPv4** | RFC 2131 | Protocole DHCP pour IPv4 (définition) |
| **DHCPv4** | RFC 2132 | Options DHCP pour IPv4 |
| **DHCPv6** | RFC 8415 | DHCP pour IPv6 (remplace RFC 3315) |

**Origine et évolution :**
```
RARP (Reverse ARP)
    ↓
BOOTP (Bootstrap Protocol)
    ↓
DHCP (Dynamic Host Configuration Protocol)
```

> [!info] Héritage de BOOTP
> DHCP est une évolution de BOOTP, qui permettait déjà le démarrage réseau des machines sans disque (diskless workstations). DHCP conserve cette fonctionnalité tout en ajoutant la gestion dynamique des adresses.

**Principe de base :**
- Un **serveur centralisé** gère les paramètres réseau
- Les **clients** demandent automatiquement leur configuration
- Attribution **dynamique** d'adresses IP via un système de **baux** (leases)

> [!important] Protocole standardisé
> DHCP est un protocole standardisé par l'IETF, garantissant l'interopérabilité entre différents équipements (serveurs Linux, Windows, routeurs Cisco, etc.).

---

### Objectifs du DHCP

> [!success] Objectifs principaux
> DHCP répond à plusieurs objectifs essentiels pour la gestion moderne des réseaux.

**1. Centralisation de la configuration**

> [!info] Un seul point de gestion
> Un seul serveur gère l'ensemble des paramètres réseau pour tout le réseau.

**Avantages :**
- **Facilitation de gestion d'un parc** de machines
- **Mise à jour simple et rapide** des paramètres (passerelle, DNS, etc.)
- Modification centralisée : un changement sur le serveur = effet immédiat sur tous les clients au prochain renouvellement

**Exemple :**
```
Changement de serveur DNS :
- Sans DHCP : reconfigurer 200 machines manuellement
- Avec DHCP : modifier la config du serveur DHCP
  → les clients obtiennent le nouveau DNS au renouvellement de bail
```

---

**2. Configuration automatique**

> [!info] Automatisation complète
> Paramétrage IP automatique et dynamique de tous les éléments réseau.

**Paramètres attribués automatiquement :**
- **Adresse IP** unique pour chaque client
- **Masque de sous-réseau** (CIDR)
- **Passerelle par défaut**
- **Serveurs DNS**
- Et bien d'autres paramètres optionnels

**Avantages :**
- **Pas de doublons d'adresses** : le serveur gère l'unicité
- **Réduction des erreurs humaines** : pas de saisie manuelle
- **Gestion automatique des entrées/sorties** du réseau via les baux (leases)
- **Plug and play** : un nouvel équipement obtient automatiquement sa configuration

> [!tip] Déploiement rapide
> Brancher un nouvel ordinateur → allumer → automatiquement configuré et connecté au réseau en quelques secondes !

---

**3. Économie de ressources**

> [!success] Optimisation des ressources
> DHCP permet d'économiser à la fois des adresses IP et du temps d'administration.

**Économie d'adresses IP :**
- Seules les machines **"actives"** ont des paramètres attribués
- Utilisation du système de **baux IP** (pluriel de bail = lease)
- Une machine éteinte ou déconnectée libère son adresse
- Cette adresse peut être réattribuée à une autre machine

**Exemple de gain d'adresses :**
```
Entreprise avec 200 employés et 200 ordinateurs portables
Taux de présence moyen : 70% (télétravail, congés, déplacements)

Sans DHCP : besoin de 200 adresses IP statiques
Avec DHCP : besoin d'environ 150 adresses en pratique
Gain : 50 adresses réutilisables = 25% d'économie
```

**Économie de temps d'administration :**
- **Pas de configuration manuelle** de chaque poste
- **Moins d'interventions** pour dépannage (conflits IP, mauvaise config)
- **Mise à jour centralisée** des paramètres
- **Documentation automatique** via les logs du serveur DHCP

> [!tip] Temps gagné
> Configuration manuelle : 10-15 minutes/machine
> Configuration DHCP : 0 minute/machine (automatique)
> Sur 100 machines : gain de 15-25 heures de travail !

---

**4. Support des machines diskless**

> [!info] Support des hôtes sans disque
> DHCP hérite de BOOTP la capacité de supporter les **hôtes sans disque** (diskless workstations).

**Fonctionnalités de démarrage réseau :**
- Support du **démarrage réseau** (PXE - Preboot Execution Environment)
- Attribution d'adresse IP avant même le chargement du système d'exploitation
- Indication du serveur TFTP et du fichier de boot à charger

**Évolution historique :**
```
RARP (Reverse ARP)
  ↓  Limité : seulement adresse IP
BOOTP (Bootstrap Protocol)
  ↓  Amélioration : paramètres réseau + boot
DHCP (Dynamic Host Configuration Protocol)
  ↓  Ajout : attribution dynamique + baux
```

**Cas d'usage modernes :**
- **Thin clients** : postes de travail légers sans disque
- **Déploiement d'OS** : installation réseau via PXE
- **Infrastructure VDI** : postes virtualisés
- **Containers** et machines virtuelles éphémères

> [!example] Déploiement d'OS via PXE
> 1. Machine démarre → requête DHCP
> 2. Serveur DHCP fournit IP + serveur TFTP + fichier boot
> 3. Machine télécharge le bootloader via TFTP
> 4. Lancement de l'installation réseau de l'OS

---

## Fonctionnement général

> [!abstract] Section : Fonctionnement général
> Cette section détaille l'architecture et le fonctionnement du protocole DHCP, incluant le rôle du client et du serveur.

---

### Architecture client-serveur

> [!quote] Le fonctionnement
> DHCP utilise une **architecture client/serveur** avec un protocole de communication basé sur UDP.

**Protocole de communication :**

| Élément | Port UDP | Description |
|---------|----------|-------------|
| **Serveur DHCP** | **67** | Port d'écoute du serveur |
| **Client DHCP** | **68** | Port utilisé par le client |

**Protocole :** UDP (User Datagram Protocol)
- **Sans connexion** : pas d'établissement de session
- **Sans garantie de livraison** : pas d'accusé de réception au niveau transport
- **Rapide et léger** : idéal pour la configuration réseau

> [!important] Ports DHCP à connaître
> **UDP 67** : serveur DHCP
> **UDP 68** : client DHCP
> Ces ports sont standardisés et doivent être connus pour l'examen TSSR.

**Fonctionnement du serveur :**
- Un serveur DHCP gère **une (ou des) plage(s) d'adresses** (scope/pool)
- Plusieurs plages possibles pour différents sous-réseaux
- Peut distribuer des paramètres différents selon les plages

> [!example] Configuration typique
> ```
> Serveur DHCP : 192.168.1.1
> Plage d'adresses : 192.168.1.100 - 192.168.1.200 (101 adresses)
> Baux : 8 heures
> DNS : 8.8.8.8, 8.8.4.4
> Passerelle : 192.168.1.1
> ```

---

### Le client DHCP

> [!info] Caractéristiques du client DHCP
> Le client DHCP est le composant logiciel présent sur chaque machine qui souhaite obtenir une configuration réseau automatique.

**État initial du client :**
- **Pas de configuration IP initiale**
- Adresse IP source : **`0.0.0.0`** (adresse nulle)
- Adresse IP destination : **`255.255.255.255`** (broadcast)
- S'identifie via son **adresse MAC**

**Processus de demande :**
1. Le client fait une demande de configuration réseau
2. Utilisation du **broadcast IP** (255.255.255.255)
3. La requête est envoyée à **tous** les équipements du segment réseau
4. Un ou plusieurs serveurs DHCP peuvent répondre

> [!warning] Notion de domaine de diffusion (broadcast domain)
> Les requêtes DHCP utilisent le broadcast (diffusion), elles sont donc limitées au **domaine de collision** local.

**Limitation du broadcast :**
- **Passage de routeur interdit** par défaut
- Les broadcasts ne traversent pas les routeurs
- Problème : serveur DHCP sur un réseau, clients sur un autre réseau

**Solution : DHCP Relay / IP Helper**
- Configurable sur les routeurs via **ip-helper** (Cisco) ou **DHCP relay**
- Le routeur relaie les requêtes DHCP vers un serveur distant
- Permet de centraliser le serveur DHCP

> [!example] Schéma avec DHCP Relay
> ```
> Réseau A (192.168.1.0/24)          Réseau B (192.168.2.0/24)
>      |                                      |
>   Clients -------- Routeur -------- Serveur DHCP
>                  (DHCP Relay)
> 
> Le routeur relaie les broadcasts DHCP du réseau A vers le serveur du réseau B
> ```

**Renouvellement de bail :**
- Le client peut **renouveler son bail** avant expiration
- Évite de perdre la connectivité réseau
- Le renouvellement se fait automatiquement à mi-parcours du bail

---

### Identification des clients

> [!warning] Fiabilité de l'identification par MAC
> L'identification des clients DHCP pose des questions de sécurité et de fiabilité.

**Méthode traditionnelle : adresse MAC**

Un serveur DHCP utilise les **adresses MAC** pour reconnaître les clients.

**Avantages de l'identification par MAC :**
- Permet de **distinguer une demande de renouvellement** d'une nouvelle demande
- Permet de **réserver des adresses** pour certaines interfaces connues (réservation DHCP)
- Permet d'**identifier certains comportements anormaux**

> [!example] Réservation DHCP
> ```
> MAC: 00:1A:2B:3C:4D:5E → toujours attribuer 192.168.1.50
> Utile pour : serveurs, imprimantes, équipements réseau
> ```

**Limitations de l'adresse MAC :**

> [!warning] Les adresses MAC ne sont PAS des identificateurs fiables !
> Problèmes :
> - **MAC spoofing** : falsification facile de l'adresse MAC
> - **Machines virtuelles** : génération de nouvelles adresses MAC
> - **Changement de carte réseau** : nouvelle MAC pour la même machine
> - **Dual boot** : même MAC, systèmes différents

---

### Identification des clients (suite)

**Solutions modernes d'identification :**

**En IPv4 : Option DHCP 61 - Client Identifier**

> [!info] RFC 2132 §9.14 - Client Identifier
> Option permettant une identification plus fiable que la simple adresse MAC.

**Contenu possible de l'option 61 :**
- **Adresse MAC** (comportement par défaut)
- **Une chaîne aléatoire** générée par le client
- **Un identifiant aléatoire généré par l'OS** (Windows, Linux)

**Avantages :**
- Plus stable que la MAC seule
- Peut être persistant même en cas de changement de carte réseau
- Permet de mieux identifier les machines en dual-boot

---

**En IPv6 : DUID (DHCP Unique Identifier)**

> [!info] RFC 8415 §11 - DUID
> Identifiant unique utilisé en DHCPv6 pour identifier de manière fiable les clients.

**Types de DUID :**

| Type | Nom | Description |
|------|-----|-------------|
| **DUID-LLT** | Link-Layer + Time | Basé sur adresse MAC + horodatage |
| **DUID-EN** | Enterprise Number | Basé sur numéro d'entreprise |
| **DUID-LL** | Link-Layer | Basé uniquement sur adresse MAC |
| **DUID-UUID** | UUID | Identifiant universel unique |

**Génération du DUID :**
- **À partir du matériel** : combinaison MAC + timestamp ou UUID matériel
- **À partir d'un identifiant logiciel** : UUID généré par l'OS

**Persistance :**
- Le DUID est **stocké** sur la machine cliente
- Reste **identique** même après réinstallation (dans certains cas)
- Offre une meilleure identification qu'une simple adresse MAC

> [!tip] Bonne pratique
> Pour les réservations DHCP importantes (serveurs, imprimantes), utiliser l'option Client Identifier en plus de la MAC pour plus de fiabilité.

---

### Le serveur DHCP

> [!quote] Rôle du serveur DHCP
> Le serveur DHCP est le composant central qui gère l'attribution des configurations réseau aux clients.

**Fonctions principales du serveur :**

1. **Écoute les requêtes** en provenance des clients DHCP (port UDP 67)

2. **Propose une configuration IP disponible** :
   - Sélectionne une adresse dans la plage configurée
   - Vérifie qu'elle n'est pas déjà attribuée
   - Vérifie qu'elle n'est pas dans les exclusions

3. **Gère les étendues d'adresses** (scopes/pools) :
   - Définit les plages IP distribuables
   - Peut gérer plusieurs plages pour différents sous-réseaux

4. **Attribue les paramètres IP** via les **Options DHCP** :
   - Adresse IP
   - Masque de sous-réseau (CIDR)
   - Passerelle par défaut
   - Serveurs DNS
   - Nom de domaine
   - Serveur NTP
   - Serveur TFTP (pour PXE)
   - Et de nombreuses autres options...

5. **Gère les baux et réservations** :
   - **Baux dynamiques** : attribution temporaire avec durée
   - **Réservations** : attribution fixe basée sur MAC/Client ID

6. **Peut imposer des options spécifiques** :
   - Options différentes selon les clients
   - Possibilité de créer des politiques d'attribution

> [!info] Étendue (Scope/Pool)
> Une étendue est une **plage d'adresses IP** que le serveur DHCP peut distribuer aux clients d'un sous-réseau spécifique.

**Exemple de configuration d'étendue :**
```
Nom de l'étendue : "LAN Principal"
Plage d'adresses : 192.168.1.100 - 192.168.1.200
Masque : 255.255.255.0 (/24)
Exclusions : 192.168.1.150-192.168.1.160 (réservé pour d'autres usages)
Durée du bail : 8 heures
Passerelle : 192.168.1.1
DNS : 8.8.8.8, 8.8.4.4
Domaine : entreprise.local
```

**Types d'attribution :**

| Type | Description | Usage |
|------|-------------|-------|
| **Automatique** | Adresse attribuée aléatoirement dans la plage | Postes utilisateurs |
| **Dynamique** | Adresse avec bail temporaire, libérée après expiration | Usage standard |
| **Réservation** | Adresse fixe basée sur MAC/Client ID | Serveurs, imprimantes |

> [!tip] Gestion des exclusions
> Il est recommandé d'exclure de la plage DHCP les adresses utilisées pour :
> - Le serveur DHCP lui-même
> - Les routeurs et passerelles
> - Les serveurs ayant des IP statiques
> - Les équipements réseau (switches manageable, etc.)

---

### Paramètres obligatoires et optionnels

> [!important] Paramètres distribués par DHCP
> Le serveur DHCP peut distribuer de nombreux paramètres réseau, certains obligatoires, d'autres optionnels.

**Paramètres OBLIGATOIRES :**

> [!quote] Minimum vital
> Le serveur DHCP envoie **obligatoirement** les paramètres suivants.

| Paramètre | Description |
|-----------|-------------|
| **Adresse IP** | Adresse IPv4 unique attribuée au client |
| **Masque de sous-réseau** | CIDR définissant le réseau local |
| **Bail** | Durée de validité de l'attribution (lease time) |

Sans ces trois paramètres, le client ne peut pas avoir une configuration réseau fonctionnelle.

---

**Paramètres OPTIONNELS :**

> [!info] Options DHCP
> Les paramètres optionnels sont **multiples** et permettent une configuration complète du client.

**Principaux paramètres optionnels :**

| Option | Nom | Description | Numéro d'option |
|--------|-----|-------------|-----------------|
| **Router** | Passerelle par défaut | Route vers Internet | 3 |
| **DNS** | Serveurs DNS | Adresses IP des serveurs DNS | 6 |
| **Domain Name** | Nom de domaine | Domaine de recherche DNS | 15 |
| **NTP** | Serveur de temps | Synchronisation horaire | 42 |
| **TFTP Server** | Serveur TFTP | Pour démarrage réseau (PXE) | 66 |
| **Bootfile Name** | Fichier de boot | Nom du fichier à charger via TFTP | 67 |
| **Vendor Options** | Options constructeur | Paramètres spécifiques | 43 |

> [!example] Options DHCP courantes
> ```
> Option 1  : Masque de sous-réseau
> Option 3  : Passerelle par défaut (router)
> Option 6  : Serveurs DNS
> Option 15 : Nom de domaine DNS
> Option 42 : Serveurs NTP (temps)
> Option 51 : Durée du bail
> Option 66 : Serveur TFTP
> Option 67 : Nom du fichier de boot
> ```

**Exemples de configuration complète :**

```
Configuration DHCP distribuée à un client :

Adresse IP       : 192.168.1.150
Masque          : 255.255.255.0 (/24)
Bail            : 86400 secondes (24 heures)
Passerelle      : 192.168.1.1
DNS primaire    : 8.8.8.8
DNS secondaire  : 8.8.4.4
Domaine         : entreprise.local
Serveur NTP     : 192.168.1.10
```

> [!tip] Configuration minimale vs complète
> - **Minimale** : IP + masque + bail = connexion au réseau local
> - **Standard** : + passerelle + DNS = accès Internet et résolution de noms
> - **Complète** : + NTP + domaine + options avancées = configuration professionnelle optimale

**Options avancées pour cas spécifiques :**
- **Option 119** : Domain Search List (liste de domaines de recherche DNS)
- **Option 121** : Classless Static Routes (routes statiques spécifiques)
- **Option 150** : Serveur TFTP (alternatif pour Cisco)
- **Option 252** : WPAD (Web Proxy Auto-Discovery)

> [!note] RFC 2132
> La RFC 2132 définit toutes les options DHCP disponibles pour IPv4. Il en existe plus de 250 options différentes !

---

### Synthèse du fonctionnement

> [!success] Fonctionnement complet DHCP
> Synthèse du processus d'attribution d'une configuration réseau par DHCP.

**Processus complet :**

1. **Le client envoie une requête** DHCPDISCOVER (broadcast)
   - Demande : "Y a-t-il un serveur DHCP disponible ?"
   - Envoyée à tous les équipements du réseau local

2. **Le(s) serveur(s) DHCP répond(ent)** avec DHCPOFFER
   - Proposition : "Voici une adresse IP disponible et les paramètres associés"
   - Plusieurs serveurs peuvent répondre

3. **Le client choisit une offre** et envoie DHCPREQUEST
   - Demande de réservation de la configuration proposée
   - Si plusieurs offres : généralement la première reçue

4. **Si le client accepte une proposition :**
   - Il envoie une **demande de réservation**
   - Le serveur **réserve cette adresse** dans sa base
   - La réservation est valable pour une **durée donnée** (bail/lease)

5. **Le serveur confirme** avec DHCPACK
   - Envoie les **informations de paramétrage complètes**
   - Réserve l'adresse pour la durée du bail
   - Le client configure son interface réseau

6. **Le client utilise la configuration**
   - L'adresse est active pendant toute la durée du bail
   - Le client tentera de renouveler le bail avant expiration

> [!important] Système de baux (leases)
> Le **bail** (lease) est la durée pendant laquelle une adresse IP est attribuée à un client. C'est un élément fondamental de DHCP qui permet la réutilisation des adresses.

**Cycle de vie d'un bail :**

```
Durée du bail : 24 heures (86400 secondes)

0h       12h           18h           24h
|--------|-------------|-------------|
Début    T1 (50%)      T2 (87.5%)    Expiration
         Renouvellement Renouvellement  Fin du bail
         auprès du     auprès de       Perte de config
         même serveur  n'importe quel
                       serveur
```

**Phases du bail :**
- **T1 (50% du bail)** : le client tente de renouveler auprès du serveur qui lui a attribué l'adresse
- **T2 (87.5% du bail)** : si T1 échoue, le client tente de renouveler auprès de n'importe quel serveur DHCP
- **Expiration (100%)** : si aucun renouvellement, le client perd sa configuration et doit redemander

> [!tip] Durée de bail recommandée
> - **Réseau stable** (bureaux fixes) : 1 à 7 jours
> - **Réseau avec rotations** (WiFi visiteurs, hotspot) : 1 à 8 heures
> - **Réseau très dynamique** (conférence, événement) : 30 minutes à 2 heures

---

## Séquence DHCP

> [!abstract] Section : Séquence DHCP
> Cette section détaille les différents messages DHCP et les séquences d'échanges entre client et serveur.

---

### Les messages DHCP

> [!info] Messages de communication DHCP
> DHCP utilise **8 types de messages** différents pour la communication entre client et serveur.

**Vue d'ensemble des messages :**

| Message | Direction | Rôle |
|---------|-----------|------|
| **DHCPDISCOVER** | Client → Broadcast | Recherche de serveur DHCP |
| **DHCPOFFER** | Serveur → Client | Proposition de configuration |
| **DHCPREQUEST** | Client → Serveur | Demande de réservation |
| **DHCPACK** | Serveur → Client | Confirmation d'attribution |
| **DHCPNAK** | Serveur → Client | Refus de réservation |
| **DHCPDECLINE** | Client → Serveur | Refus de l'offre (IP déjà utilisée) |
| **DHCPRELEASE** | Client → Serveur | Résiliation du bail |
| **DHCPINFORM** | Client → Serveur | Demande d'infos sans réservation d'IP |

> [!important] Classification des messages
> Les 4 premiers messages (DISCOVER, OFFER, REQUEST, ACK) constituent la séquence normale d'attribution.
> Les 4 autres sont des messages complémentaires pour gérer les cas particuliers.

---

### Séquence DORA

> [!success] Séquence DORA - Fonctionnement optimal
> La séquence **DORA** représente le fonctionnement normal et optimal de DHCP.

**DORA = Acronyme mnémotechnique**
- **D**iscover
- **O**ffer
- **R**equest
- **A**ck (Acknowledge)

---

**Messages de communication fonctionnelle :**

**1. DHCPDISCOVER**

> [!info] Client → Broadcast
> Le client **recherche** un serveur DHCP disponible sur le réseau.

**Caractéristiques :**
- **Direction** : Client vers broadcast (255.255.255.255)
- **Contenu** : adresse MAC du client, demande de configuration
- **Objectif** : "Y a-t-il un serveur DHCP disponible ?"

```
Source IP      : 0.0.0.0 (client n'a pas encore d'IP)
Destination IP : 255.255.255.255 (broadcast)
Source MAC     : XX:XX:XX:XX:XX:XX (MAC du client)
Message        : DHCPDISCOVER
```

---

**2. DHCPOFFER**

> [!info] Serveur → Client
> Le serveur **propose** une configuration IP disponible au client.

**Caractéristiques :**
- **Direction** : Serveur vers client (unicast ou broadcast selon le client)
- **Contenu** : 
  - Adresse IP proposée
  - Masque de sous-réseau
  - Durée du bail
  - Adresse du serveur DHCP
  - Options DHCP (passerelle, DNS, etc.)

```
Message        : DHCPOFFER
Adresse offerte: 192.168.1.150
Masque         : 255.255.255.0
Bail           : 86400 secondes
Serveur DHCP   : 192.168.1.1
Options        : passerelle, DNS, domaine, etc.
```

> [!note] Plusieurs offres possibles
> Si plusieurs serveurs DHCP sont présents sur le réseau, le client peut recevoir plusieurs DHCPOFFER. Il choisira généralement la première offre reçue.

---

**3. DHCPREQUEST**

> [!info] Client → Serveur/Broadcast
> Le client **demande la réservation** de la configuration proposée par un serveur.

**Caractéristiques :**
- **Direction** : Client vers broadcast (ou vers le serveur spécifique)
- **Contenu** : 
  - Adresse IP demandée
  - Identifiant du serveur DHCP choisi
  - Adresse MAC du client

```
Message        : DHCPREQUEST
Adresse voulue : 192.168.1.150
Serveur choisi : 192.168.1.1
Client MAC     : XX:XX:XX:XX:XX:XX
```

**Pourquoi en broadcast ?**
- Informe **tous les serveurs DHCP** que le client a fait un choix
- Les serveurs qui n'ont pas été choisis peuvent **libérer** l'adresse qu'ils avaient proposée

---

**4. DHCPACK**

> [!success] Serveur → Client
> Le serveur **confirme** la réservation et envoie le paramétrage complet.

**Caractéristiques :**
- **Direction** : Serveur vers client (unicast)
- **Contenu** : Configuration réseau complète
  - Confirmation de l'adresse IP attribuée
  - Tous les paramètres réseau (masque, passerelle, DNS, etc.)
  - Durée du bail

```
Message          : DHCPACK
Adresse attribuée: 192.168.1.150
Masque           : 255.255.255.0
Bail             : 86400 secondes (24h)
Passerelle       : 192.168.1.1
DNS              : 8.8.8.8, 8.8.4.4
Domaine          : entreprise.local
```

**Après réception du DHCPACK :**
- Le client **configure son interface** réseau avec les paramètres reçus
- Le client **vérifie** que l'adresse n'est pas déjà utilisée (ARP gratuit)
- Si l'adresse est déjà utilisée : le client envoie un DHCPDECLINE

---

**Schéma de la séquence DORA :**

```
     Client                                Serveur DHCP
       |                                         |
       |  1. DHCPDISCOVER (broadcast)           |
       |--------------------------------------->|
       |     "Y a-t-il un serveur DHCP ?"       |
       |                                         |
       |  2. DHCPOFFER                           |
       |<---------------------------------------|
       |     "Voici une IP disponible"           |
       |                                         |
       |  3. DHCPREQUEST (broadcast)             |
       |--------------------------------------->|
       |     "Je veux cette IP"                  |
       |                                         |
       |  4. DHCPACK                             |
       |<---------------------------------------|
       |     "OK, c'est à toi"                   |
       |                                         |
    Configure                               Enregistre
    l'interface                             le bail
```

> [!success] Séquence DORA complète
> La séquence DORA est la **séquence normale** et **optimale** pour obtenir une configuration réseau via DHCP. C'est celle qui se produit lors d'une première connexion ou après expiration d'un bail.

---

### Messages complémentaires

> [!info] Autres messages DHCP
> En plus de la séquence DORA, DHCP dispose de 4 messages complémentaires pour gérer des situations spécifiques.

---

**5. DHCPNAK (Negative Acknowledge)**

> [!warning] Serveur → Client
> Le serveur **refuse** la réservation demandée par le client.

**Caractéristiques :**
- **Direction** : Serveur vers client
- **Raisons possibles** :
  - L'adresse demandée n'est plus disponible
  - L'adresse n'appartient pas à l'étendue du serveur
  - Le client est sur le mauvais sous-réseau
  - Le bail a expiré et l'adresse a été réattribuée

**Conséquence :**
- Le client doit recommencer le processus depuis le début (DHCPDISCOVER)

```
Scénario : expiration de bail
Client  : DHCPREQUEST (demande de renouvellement)
Serveur : DHCPNAK (bail expiré, adresse réattribuée)
Client  : DHCPDISCOVER (nouvelle demande)
```

---

**6. DHCPDECLINE**

> [!warning] Client → Serveur
> Le client **refuse** l'adresse proposée car elle est déjà utilisée.

**Caractéristiques :**
- **Direction** : Client vers serveur
- **Moment** : Après réception du DHCPACK
- **Raison** : Le client détecte que l'adresse est déjà utilisée (via ARP)

**Processus :**
1. Client reçoit DHCPACK avec l'adresse 192.168.1.150
2. Client envoie un **ARP gratuit** (Gratuitous ARP) pour vérifier
3. Si une autre machine répond avec cette IP : **conflit détecté**
4. Client envoie DHCPDECLINE
5. Client recommence avec DHCPDISCOVER

```
Détection de conflit d'IP :
Client  : DHCPACK reçu (192.168.1.150)
Client  : ARP gratuit "Qui a 192.168.1.150 ?"
Machine : "Moi ! (réponse ARP)"
Client  : DHCPDECLINE "Cette IP est déjà utilisée"
Serveur : Marque l'IP comme problématique
Client  : DHCPDISCOVER (nouvelle demande)
```

> [!tip] Protection contre les conflits
> Le DHCPDECLINE est un mécanisme de sécurité pour éviter les conflits d'IP sur le réseau. C'est rare mais important.

---

**7. DHCPRELEASE**

> [!info] Client → Serveur
> Le client **résilie son bail** volontairement et libère son adresse IP.

**Caractéristiques :**
- **Direction** : Client vers serveur
- **Moment** : Quand le client n'a plus besoin de l'adresse
- **Effet** : Libération immédiate de l'adresse dans le pool DHCP

**Cas d'usage :**
- Arrêt propre de la machine (shutdown)
- Libération manuelle de la config DHCP
- Changement de réseau (portable)

```bash
# Linux : libérer l'adresse DHCP
sudo dhclient -r eth0

# Windows : libérer l'adresse DHCP
ipconfig /release
```

**Avantages :**
- Libération **immédiate** de l'adresse (pas besoin d'attendre l'expiration du bail)
- L'adresse redevient **immédiatement disponible** pour d'autres clients
- Bonne pratique pour optimiser l'utilisation des adresses

> [!note] Pas toujours envoyé
> En pratique, le DHCPRELEASE n'est pas toujours envoyé (extinction brutale, perte réseau). Le serveur DHCP doit donc aussi gérer les expirations de bail.

---

**8. DHCPINFORM**

> [!info] Client → Serveur
> Le client **demande des paramètres** de configuration **sans demander d'adresse IP**.

**Caractéristiques :**
- **Direction** : Client vers serveur
- **Particularité** : Le client a **déjà une adresse IP** (statique ou autre source)
- **Objectif** : Obtenir uniquement les options DHCP (DNS, passerelle, etc.)

**Cas d'usage :**
- Machine avec IP statique qui veut les paramètres DHCP (DNS, domaine, etc.)
- Récupération d'options après un changement réseau
- Machines avec configuration hybride

```
Scénario : serveur avec IP statique
Machine : IP statique configurée manuellement = 192.168.1.10
Machine : DHCPINFORM "Quels sont les DNS et la passerelle ?"
Serveur : Réponse avec options DHCP (DNS, passerelle, domaine, etc.)
Machine : Utilise ces paramètres sans changer son IP
```

> [!example] Configuration hybride
> Certaines machines (serveurs, équipements réseau) ont besoin d'une IP fixe mais veulent récupérer dynamiquement d'autres paramètres comme les DNS. DHCPINFORM permet cela.

---

### Cas d'usage des demandes

> [!info] Différents scénarios d'attribution DHCP
> Selon le contexte, la séquence d'échanges DHCP peut varier. Voici les principaux cas d'usage.

---

**Cas 1 : Première connexion**

> [!example] Machine neuve ou première connexion au réseau
> C'est la séquence DORA complète, cas le plus courant.

**Séquence :**
```
DHCPDISCOVER → DHCPOFFER → DHCPREQUEST → DHCPACK
```

**Contexte :**
- Machine jamais connectée au réseau
- Première connexion après installation
- Aucune configuration DHCP précédente en mémoire

**Déroulement :**
1. Le client ne connaît aucun serveur DHCP
2. Il fait un broadcast pour découvrir les serveurs (DISCOVER)
3. Un ou plusieurs serveurs répondent avec des offres (OFFER)
4. Le client choisit une offre et la demande (REQUEST)
5. Le serveur confirme (ACK)

---

**Cas 2 : Redémarrage avec une IP toujours valable**

> [!success] Optimisation : séquence raccourcie
> Le client connaît déjà le serveur DHCP et son ancienne IP est toujours valable.

**Séquence raccourcie :**
```
DHCPREQUEST → DHCPACK  (ou DHCPNAK si problème)
```

**Conditions pour cette séquence :**
Le client envoie un DHCPREQUEST **directement** uniquement s'il connaît déjà :
- **L'adresse IP qu'il utilisait** précédemment
- **L'identifiant du serveur** qui lui a attribué cette adresse IP

**Réponses possibles du serveur :**

| Réponse | Condition | Action client |
|---------|-----------|---------------|
| **DHCPACK** | IP toujours valide et dans l'étendue | Utilise la configuration |
| **DHCPNAK** | IP n'est plus valable | Recommence avec DHCPDISCOVER |

**Exemple :**
```
Client redémarre après 2 heures d'arrêt (bail de 24h)
Client : DHCPREQUEST "Je veux 192.168.1.150 du serveur 192.168.1.1"
Serveur : DHCPACK "OK, toujours disponible"
Client : Configure l'interface (très rapide !)
```

**Avantages :**
- **Plus rapide** : 2 messages au lieu de 4
- **Moins de trafic réseau**
- **Réduction de charge** sur le serveur DHCP

---

**Cas 3 : Expiration du bail**

> [!warning] Bail expiré
> Le client tente de renouveler son bail mais celui-ci a expiré.

**Séquence :**
```
DHCPREQUEST → DHCPNAK → DHCPDISCOVER → DHCPOFFER → DHCPREQUEST → DHCPACK
```

**Scénario :**
1. Le client n'a pas renouvelé son bail à temps (machine éteinte longtemps)
2. Le bail a expiré côté serveur
3. Le serveur a **libéré l'adresse** dans son pool
4. L'adresse a potentiellement été **réattribuée** à un autre client

**Déroulement :**
```
Client : DHCPREQUEST "Je veux renouveler 192.168.1.150"
Serveur : DHCPNAK "Désolé, bail expiré ou adresse non disponible"
Client : DHCPDISCOVER "OK, je cherche un nouveau serveur"
Serveur : DHCPOFFER "Voici 192.168.1.151"
Client : DHCPREQUEST "Je prends 192.168.1.151"
Serveur : DHCPACK "Confirmé"
```

**Causes d'expiration :**
- Machine éteinte pendant plus longtemps que la durée du bail
- Perte de connexion réseau prolongée
- Hibernation prolongée de la machine

> [!note] Délai de renouvellement
> Normalement, le client tente de renouveler son bail à T1 (50% du bail) et T2 (87.5% du bail). L'expiration ne devrait arriver que si la machine est hors ligne.

---

**Cas 4 : Changement de réseau**

> [!warning] Mauvais sous-réseau
> Le client se connecte à un réseau différent avec son ancienne configuration.

**Séquence :**
```
DHCPREQUEST → DHCPNAK → DHCPDISCOVER → ...
```

**Scénario typique : machine portable**
```
Bureau (réseau A) : 192.168.1.0/24 → IP obtenue : 192.168.1.150
---
Machine déplacée vers salle de réunion (réseau B) : 192.168.2.0/24
---
Client : DHCPREQUEST "Je veux 192.168.1.150"
Serveur réseau B : DHCPNAK "Cette IP n'est pas dans mon sous-réseau"
Client : DHCPDISCOVER "Je cherche une nouvelle IP"
Serveur réseau B : DHCPOFFER "Voici 192.168.2.100"
...
```

**Pourquoi DHCPNAK ?**
- L'adresse demandée (192.168.1.150) n'appartient **pas au sous-réseau** actuel
- Le serveur DHCP du réseau B gère le réseau 192.168.2.0/24
- Il ne peut pas attribuer une adresse du réseau A

**Détection automatique :**
Le client peut détecter le changement de réseau :
- Via la **passerelle par défaut** différente
- Via le **préfixe réseau** différent
- Via l'absence de réponse du serveur DHCP d'origine

> [!tip] Mobilité des portables
> C'est un cas très fréquent avec les machines portables qui se déplacent entre différents réseaux (bureau, domicile, WiFi public, etc.). DHCP gère cela automatiquement.

---

## Fonctionnalités avancées

> [!abstract] Section : Fonctionnalités avancées
> Cette section couvre les paramètres avancés et les fonctionnalités étendues de DHCP.

---

### Paramètres de configuration

> [!info] Distribution de nombreux paramètres
> Les serveurs DHCP peuvent distribuer bien plus que des adresses IP. Ils peuvent configurer complètement un client réseau.

**Catégories de paramètres :**

**1. Paramètres réseau de base**
- **Adresse IP** et **masque de sous-réseau**
- **Informations de routage** :
  - Route par défaut (passerelle)
  - Routes statiques spécifiques (option 121)

**2. Résolution de noms**
- **Adresses de serveurs DNS** (récursifs)
- **Nom de domaine** pour les recherches DNS
- **Liste de recherche de domaines** (domain search list)

**3. Services réseau**
- **Serveurs de temps (NTP)** pour synchronisation horaire
- **Serveurs WINS** (Windows Internet Name Service) pour réseaux Windows
- **Serveurs d'impression** (option 9)

**4. Démarrage réseau (PXE)**
- **Serveur TFTP** (Trivial File Transfer Protocol)
- **Nom du fichier de boot** à télécharger
- **Options PXE** pour démarrage sans disque

**5. Options VoIP et téléphonie**
- **Serveurs SIP** pour téléphonie IP
- **Serveurs d'approvisionnement** pour téléphones IP

**6. Autres paramètres avancés**
- **Proxy web automatique (WPAD)**
- **Options vendeur spécifiques** (option 43)
- **Paramètres de mobilité IP**

> [!example] Configuration complète d'un poste de travail
> ```
> === Paramètres réseau ===
> Adresse IP      : 192.168.1.150
> Masque          : 255.255.255.0
> Passerelle      : 192.168.1.1
> Bail            : 86400 secondes (24h)
> 
> === DNS ===
> DNS primaire    : 192.168.1.10
> DNS secondaire  : 8.8.8.8
> Domaine         : entreprise.local
> Recherche       : entreprise.local, prod.entreprise.local
> 
> === Services ===
> Serveur NTP     : 192.168.1.10
> Serveur WINS    : 192.168.1.11
> 
> === PXE (si applicable) ===
> Serveur TFTP    : 192.168.1.20
> Fichier boot    : pxelinux.0
> ```

**Tableau des principales options DHCP :**

| Option | Nom | Description | Usage |
|--------|-----|-------------|-------|
| 1 | Subnet Mask | Masque de sous-réseau | Obligatoire |
| 3 | Router | Passerelle par défaut | Standard |
| 6 | DNS Server | Serveurs DNS | Standard |
| 15 | Domain Name | Nom de domaine | Standard |
| 42 | NTP Server | Serveur de temps | Recommandé |
| 44 | WINS Server | Serveur WINS | Réseaux Windows |
| 51 | Lease Time | Durée du bail | Obligatoire |
| 66 | TFTP Server | Serveur TFTP | PXE/Boot réseau |
| 67 | Bootfile Name | Nom fichier boot | PXE/Boot réseau |
| 119 | Domain Search | Liste recherche DNS | Avancé |
| 121 | Classless Static Routes | Routes statiques | Avancé |
| 252 | WPAD | Proxy auto config | Avancé |

> [!tip] Options les plus utilisées
> Dans un réseau d'entreprise standard, les options les plus couramment configurées sont :
> - 1 (Masque), 3 (Passerelle), 6 (DNS), 15 (Domaine), 42 (NTP), 51 (Bail)

**Configuration différenciée par client :**
Les serveurs DHCP peuvent distribuer des options différentes selon :
- Le **sous-réseau** d'origine du client
- L'**adresse MAC** ou le **Client ID**
- Des **classes de clients** (Option 60 - Vendor Class Identifier)
- Des **politiques** définies par l'administrateur

> [!example] Configuration différenciée
> ```
> Réseau bureaux (VLAN 10) :
>   → DNS interne : 192.168.10.1
>   → Bail : 24 heures
> 
> Réseau invités (VLAN 20) :
>   → DNS public : 8.8.8.8
>   → Bail : 2 heures
>   → Pas de serveur WINS
>   → Proxy web obligatoire
> 
> Téléphones IP (identifiés par Option 60) :
>   → Serveur TFTP : 192.168.1.50
>   → Serveur SIP : 192.168.1.51
>   → VLAN voix : 100
> ```

---

### Pour aller plus loin

> [!tip] Approfondir DHCP
> Ressources et fonctionnalités avancées pour aller plus loin dans la maîtrise de DHCP.

**Documentation de référence :**

**RFC officielles :**
- **IPv4 :**
  - **RFC 2131** : Dynamic Host Configuration Protocol (définition du protocole)
  - **RFC 2132** : DHCP Options and BOOTP Vendor Extensions (toutes les options)
- **IPv6 :**
  - **RFC 8415** : Dynamic Host Configuration Protocol for IPv6 (DHCPv6)

**Documentation complémentaire :**
- **Wikipédia** : 
  - [En français](https://fr.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) : vue d'ensemble
  - [En anglais](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) : beaucoup plus complet et détaillé
- **Guides d'administration** :
  - Documentation Microsoft (Windows Server DHCP)
  - Documentation ISC (ISC DHCP Server - Linux)
  - Documentation Cisco (DHCP Server et DHCP Relay)

---

**Fonctionnalités avancées à étudier :**

**1. Tolérance de pannes (High Availability)**

> [!info] Multiples serveurs DHCP coordonnés
> Pour garantir la disponibilité du service DHCP, plusieurs serveurs peuvent être déployés de manière coordonnée.

**Approches possibles :**

| Méthode | Description | Avantages | Inconvénients |
|---------|-------------|-----------|---------------|
| **Split-scope** | Chaque serveur gère une partie de la plage | Simple | Pas de vraie HA |
| **DHCP Failover** | Deux serveurs partagent les mêmes plages | Vraie HA | Configuration complexe |
| **Anycast DHCP** | Plusieurs serveurs avec la même IP | Transparent | Nécessite routage avancé |

**Exemple de split-scope :**
```
Plage totale : 192.168.1.100 - 192.168.1.200 (101 adresses)

Serveur DHCP 1 : 192.168.1.100 - 192.168.1.150 (51 adresses = 50%)
Serveur DHCP 2 : 192.168.1.151 - 192.168.1.200 (50 adresses = 50%)

Si un serveur tombe, l'autre continue avec sa moitié
```

---

**2. Relais DHCP (DHCP Relay / IP Helper)**

> [!info] Routage des requêtes DHCP
> Permet à des clients sur un réseau de communiquer avec un serveur DHCP sur un autre réseau.

**Problème :**
- Les requêtes DHCP utilisent le **broadcast** (255.255.255.255)
- Les broadcasts ne traversent **pas les routeurs** par défaut
- Les clients sur un réseau distant ne peuvent pas atteindre le serveur DHCP

**Solution : DHCP Relay**
- Le **routeur** fait office de relais
- Il **reçoit** les broadcasts DHCP des clients
- Il les **transforme** en unicast
- Il les **transfère** vers le serveur DHCP distant
- Il **retransmet** les réponses aux clients

```
Réseau A              Routeur avec           Réseau B
(192.168.1.0/24)      DHCP Relay         (192.168.2.0/24)
                      
Client -------- Broadcast DHCP --------> Routeur
                                           |
                                    Unicast vers
                                    Serveur DHCP
                                           |
                                           v
                                    Serveur DHCP
                                           |
                                    Réponse unicast
                                           |
                                           v
Client <------- Réponse relayée -------- Routeur
```

**Configuration Cisco IOS :**
```cisco
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 192.168.2.10
 ! 192.168.2.10 = adresse du serveur DHCP
```

**Configuration Linux (isc-dhcp-relay) :**
```bash
# /etc/default/isc-dhcp-relay
SERVERS="192.168.2.10"
INTERFACES="eth0"
```

**Avantages du DHCP Relay :**
- **Centralisation** : un seul serveur DHCP pour plusieurs réseaux
- **Économie** : pas besoin d'un serveur par réseau
- **Gestion simplifiée** : configuration centralisée

---

**3. Démarrage de machines avec PXE**

> [!info] Preboot Execution Environment
> PXE permet le démarrage d'une machine via le réseau sans système d'exploitation installé.

**Principe de PXE :**
1. Machine démarre → BIOS/UEFI en mode réseau
2. Requête DHCP avec options PXE
3. Serveur DHCP répond avec :
   - Adresse IP
   - Adresse du serveur TFTP (option 66)
   - Nom du fichier de boot (option 67)
4. Machine télécharge le bootloader via TFTP
5. Bootloader télécharge le kernel et initramfs
6. Démarrage du système d'exploitation réseau

**Cas d'usage :**
- **Déploiement d'OS** : installation automatique
- **Maintenance système** : boot sur un OS de diagnostic
- **Thin clients** : démarrage sans disque dur
- **Laboratoires** : déploiement rapide de postes identiques

**Options DHCP pour PXE :**
```
Option 66 : next-server = 192.168.1.20 (serveur TFTP)
Option 67 : filename = "pxelinux.0" (bootloader)
```

**Exemple de configuration ISC DHCP :**
```
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8;
    
    # Configuration PXE
    next-server 192.168.1.20;       # Serveur TFTP
    filename "pxelinux.0";          # Bootloader
}
```

---

**4. Sécurisation DHCP**

**Problématiques de sécurité :**
- **Rogue DHCP Server** : serveur DHCP pirate sur le réseau
- **DHCP Starvation** : épuisement du pool d'adresses
- **DHCP Spoofing** : usurpation d'identité du serveur

**Solutions :**
- **DHCP Snooping** : fonctionnalité des switches pour filtrer les serveurs DHCP autorisés
- **Port Security** : limitation du nombre de MAC par port
- **Authentication** : authentification des clients (802.1X)
- **MAC Filtering** : liste blanche/noire de MAC autorisées

---

**5. DHCPv6 et SLAAC**

> [!info] DHCP pour IPv6
> IPv6 propose deux mécanismes d'auto-configuration : SLAAC et DHCPv6.

**SLAAC (StateLess Address Auto-Configuration) :**
- Configuration automatique **sans serveur DHCP**
- Basé sur les **Router Advertisements** (RA)
- Le client génère son adresse à partir du préfixe réseau + son interface ID

**DHCPv6 :**
- Équivalent DHCP pour IPv6
- Peut fonctionner en mode **stateful** (avec attribution d'adresse) ou **stateless** (uniquement options)
- Utilise les ports UDP 546 (client) et 547 (serveur)

**Combinaison possible :**
- SLAAC pour l'adresse IP
- DHCPv6 pour les options (DNS, domaine, etc.)

> [!note] Différence IPv4/IPv6
> IPv6 a intégré l'auto-configuration dans le protocole de base (SLAAC), contrairement à IPv4 qui nécessite absolument DHCP pour l'auto-configuration.

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP
> Voici les éléments essentiels à maîtriser sur DHCP pour ton titre TSSR.

### Concepts fondamentaux

- **DHCP** = Dynamic Host Configuration Protocol
- **Objectif** : attribution **automatique** de paramètres réseau IP aux machines
- **Standardisé** par RFC 2131 et RFC 2132 (IPv4) / RFC 8415 (IPv6)
- **Architecture client-serveur** basée sur **UDP**
- Résout les problèmes de l'**adressage statique** : erreurs humaines, temps d'administration, conflits d'IP

### Protocole de communication

- **Port UDP 67** : serveur DHCP (à écouter)
- **Port UDP 68** : client DHCP
- Communication basée sur **UDP** : rapide, sans connexion
- Utilisation du **broadcast** (255.255.255.255) pour les découvertes
- Les broadcasts ne traversent **pas les routeurs** par défaut

### Séquence DORA

**Mnémotechnique essentiel : D.O.R.A.**

1. **DISCOVER** : Client cherche un serveur DHCP (broadcast)
2. **OFFER** : Serveur propose une configuration
3. **REQUEST** : Client demande la réservation
4. **ACK** : Serveur confirme l'attribution

**C'est la séquence normale et optimale** lors d'une première connexion.

### Messages DHCP

**8 types de messages au total :**

| Message | Direction | Usage |
|---------|-----------|-------|
| DISCOVER | C → Broadcast | Recherche de serveur |
| OFFER | S → C | Proposition de config |
| REQUEST | C → S | Demande de réservation |
| ACK | S → C | Confirmation |
| NAK | S → C | Refus |
| DECLINE | C → S | Refus (IP déjà utilisée) |
| RELEASE | C → S | Résiliation du bail |
| INFORM | C → S | Demande d'options sans IP |

### Système de baux (leases)

- Un **bail** (lease) est la durée pendant laquelle une adresse est attribuée
- Permet la **réutilisation** des adresses IP (économie d'adresses)
- **T1 (50% du bail)** : tentative de renouvellement auprès du serveur d'origine
- **T2 (87,5% du bail)** : tentative auprès de n'importe quel serveur
- **Expiration (100%)** : perte de la configuration, nouvelle demande nécessaire

### Paramètres distribués

**Obligatoires :**
- Adresse IP
- Masque de sous-réseau
- Durée du bail

**Optionnels courants :**
- Passerelle par défaut (option 3)
- Serveurs DNS (option 6)
- Nom de domaine (option 15)
- Serveur NTP (option 42)
- Serveur TFTP pour PXE (option 66)
- Nom du fichier de boot (option 67)

### Identification des clients

- **Traditionnellement** : adresse MAC
- **Problème** : MAC non fiable (spoofing, VM, changement de carte)
- **IPv4** : Option 61 - Client Identifier (plus fiable)
- **IPv6** : DUID (DHCP Unique Identifier)
- Permet les **réservations** : attribuer toujours la même IP à un client spécifique

### Cas d'usage pratiques

**Cas 1 : Première connexion**
- Séquence DORA complète

**Cas 2 : Redémarrage rapide**
- REQUEST direct → ACK (séquence raccourcie)

**Cas 3 : Bail expiré**
- REQUEST → NAK → DISCOVER (nouvelle séquence DORA)

**Cas 4 : Changement de réseau**
- REQUEST → NAK (mauvais sous-réseau) → DISCOVER

### Rôle du client DHCP

- **Pas de configuration initiale** : IP source 0.0.0.0
- S'identifie par son **adresse MAC** (ou Client ID)
- Envoie des **broadcasts** pour trouver un serveur
- Peut **renouveler** son bail automatiquement
- **Vérifie** l'unicité de l'IP reçue (ARP gratuit)

### Rôle du serveur DHCP

- **Écoute** sur port UDP 67
- Gère des **étendues** (scopes/pools) d'adresses
- **Attribue** adresses et paramètres via options DHCP
- Gère les **baux** et **réservations**
- Peut imposer des **options spécifiques** selon les clients

### Fonctionnalités avancées

- **DHCP Relay / IP Helper** : routage des requêtes DHCP entre réseaux
- **Tolérance de pannes** : plusieurs serveurs coordonnés (failover, split-scope)
- **PXE** : démarrage réseau via TFTP (options 66 et 67)
- **DHCPv6** : équivalent pour IPv6 (ports 546/547)
- **SLAAC** : auto-configuration IPv6 sans DHCP

### Avantages de DHCP

- **Centralisation** : un seul point de gestion
- **Automatisation** : pas de configuration manuelle
- **Économie d'IP** : réutilisation via les baux
- **Économie de temps** : gain administratif majeur
- **Réduction d'erreurs** : pas de saisie manuelle
- **Pas de doublons** : gestion de l'unicité par le serveur
- **Flexibilité** : adaptation aux machines nomades

### Limitations

- **Dépendance** : panne du serveur = plus de nouvelles attributions
- **Broadcast limité** : nécessite DHCP Relay pour multi-réseaux
- **Sécurité** : risque de rogue DHCP server (solution : DHCP snooping)
- **MAC non fiable** : nécessite Client ID ou DUID

### Configuration réseau typique

```
Adresse IP    : 192.168.1.150 (attribuée dynamiquement)
Masque        : 255.255.255.0
Bail          : 86400 secondes (24 heures)
Passerelle    : 192.168.1.1
DNS           : 8.8.8.8, 8.8.4.4
Domaine       : entreprise.local
Serveur NTP   : 192.168.1.10
```

### Points d'attention pour l'examen

- Connaître la **séquence DORA** par cœur (mnémotechnique)
- Savoir expliquer le **système de baux** et le processus de renouvellement
- Connaître les **ports UDP 67 et 68**
- Comprendre pourquoi les broadcasts ne traversent **pas les routeurs**
- Connaître les **3 paramètres obligatoires** : IP, masque, bail
- Savoir identifier les **cas d'usage** (première connexion, renouvellement, expiration, changement réseau)
- Comprendre le rôle du **DHCP Relay** pour les réseaux multiples
- Connaître les principales **options DHCP** (3, 6, 15, 42, 51, 66, 67)

### Commandes utiles à connaître

**Linux :**
```bash
# Obtenir une configuration DHCP
sudo dhclient eth0

# Renouveler le bail
sudo dhclient -r eth0 && sudo dhclient eth0

# Voir la configuration actuelle
ip addr show eth0
cat /var/lib/dhcp/dhclient.leases
```

**Windows :**
```cmd
# Obtenir une configuration DHCP
ipconfig /renew

# Libérer l'adresse
ipconfig /release

# Voir la configuration
ipconfig /all
```

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR
> Termes techniques importants à maîtriser pour comprendre et travailler avec DHCP.

| Terme | Définition |
|-------|------------|
| **DHCP** | Dynamic Host Configuration Protocol - Protocole d'attribution automatique de paramètres réseau IP |
| **RFC 2131** | RFC définissant le protocole DHCP pour IPv4 |
| **RFC 2132** | RFC définissant les options DHCP pour IPv4 |
| **RFC 8415** | RFC définissant DHCPv6 (DHCP pour IPv6) |
| **Adressage statique** | Configuration manuelle fixe des paramètres réseau sur chaque machine |
| **Adressage dynamique** | Attribution automatique et temporaire de paramètres réseau via DHCP |
| **Bail (Lease)** | Durée pendant laquelle une adresse IP est attribuée à un client DHCP |
| **Renouvellement de bail** | Processus par lequel un client prolonge la durée de son bail avant expiration |
| **Expiration de bail** | Fin de validité d'un bail, l'adresse est libérée et doit être redemandée |
| **T1** | Moment à 50% du bail où le client tente de renouveler auprès du serveur d'origine |
| **T2** | Moment à 87,5% du bail où le client tente de renouveler auprès de n'importe quel serveur |
| **DORA** | Discover, Offer, Request, Acknowledge - Séquence normale d'attribution DHCP |
| **DHCPDISCOVER** | Message broadcast du client recherchant un serveur DHCP |
| **DHCPOFFER** | Message du serveur proposant une configuration au client |
| **DHCPREQUEST** | Message du client demandant la réservation d'une configuration |
| **DHCPACK** | Message du serveur confirmant l'attribution des paramètres |
| **DHCPNAK** | Message du serveur refusant une demande de réservation |
| **DHCPDECLINE** | Message du client refusant une offre (IP déjà utilisée) |
| **DHCPRELEASE** | Message du client libérant volontairement son bail |
| **DHCPINFORM** | Message du client demandant des options sans demander d'IP |
| **Broadcast** | Diffusion d'un message à tous les équipements d'un réseau (255.255.255.255) |
| **Unicast** | Envoi d'un message à un destinataire unique spécifique |
| **UDP** | User Datagram Protocol - Protocole de transport sans connexion utilisé par DHCP |
| **Port 67** | Port UDP sur lequel écoute le serveur DHCP |
| **Port 68** | Port UDP utilisé par les clients DHCP |
| **Serveur DHCP** | Serveur gérant l'attribution automatique de paramètres réseau |
| **Client DHCP** | Machine demandant une configuration réseau automatique |
| **Stub resolver** | Composant réseau minimal sur le client gérant les requêtes DNS locales |
| **Étendue (Scope)** | Plage d'adresses IP qu'un serveur DHCP peut distribuer |
| **Pool d'adresses** | Ensemble d'adresses IP disponibles pour attribution dynamique |
| **Réservation DHCP** | Attribution d'une adresse IP fixe à un client spécifique basée sur MAC/Client ID |
| **Exclusion** | Adresse(s) IP retirée(s) d'une étendue et non distribuée(s) par DHCP |
| **Options DHCP** | Paramètres supplémentaires distribués avec l'adresse IP (DNS, passerelle, etc.) |
| **Option 1** | Masque de sous-réseau (subnet mask) |
| **Option 3** | Passerelle par défaut (router) |
| **Option 6** | Serveurs DNS |
| **Option 15** | Nom de domaine DNS |
| **Option 42** | Serveurs NTP (Network Time Protocol) |
| **Option 51** | Durée du bail (lease time) |
| **Option 61** | Client Identifier - Identifiant unique du client plus fiable que la MAC |
| **Option 66** | Serveur TFTP (pour boot réseau/PXE) |
| **Option 67** | Nom du fichier de boot (bootfile name) |
| **Adresse MAC** | Media Access Control - Adresse physique unique de la carte réseau |
| **MAC spoofing** | Falsification de l'adresse MAC d'une interface réseau |
| **Client ID** | Identifiant unique du client DHCP (option 61) plus fiable que la MAC |
| **DUID** | DHCP Unique Identifier - Identifiant unique en DHCPv6 |
| **DHCP Relay** | Routeur relayant les requêtes DHCP entre réseaux |
| **IP Helper** | Fonctionnalité Cisco permettant le relais DHCP |
| **Domaine de diffusion** | Segment réseau où les broadcasts sont propagés |
| **Domaine de collision** | Même concept que domaine de diffusion dans ce contexte |
| **ARP gratuit** | Requête ARP envoyée pour vérifier l'unicité d'une adresse IP |
| **Conflit d'IP** | Deux machines utilisant la même adresse IP simultanément |
| **BOOTP** | Bootstrap Protocol - Ancêtre de DHCP pour démarrage réseau |
| **RARP** | Reverse ARP - Protocole primitif précédant BOOTP |
| **PXE** | Preboot Execution Environment - Démarrage d'une machine via le réseau |
| **TFTP** | Trivial File Transfer Protocol - Protocole simple de transfert de fichiers |
| **Diskless** | Machine sans disque dur démarrant via le réseau |
| **Thin client** | Poste de travail léger avec ressources minimales |
| **Split-scope** | Méthode de haute disponibilité avec deux serveurs gérant des plages différentes |
| **DHCP Failover** | Mécanisme de haute disponibilité avec deux serveurs synchronisés |
| **Rogue DHCP** | Serveur DHCP non autorisé sur le réseau (pirate ou mal configuré) |
| **DHCP Starvation** | Attaque épuisant le pool d'adresses DHCP |
| **DHCP Snooping** | Fonctionnalité switch filtrant les serveurs DHCP autorisés |
| **DHCPv6** | DHCP pour IPv6 (ports UDP 546 et 547) |
| **SLAAC** | StateLess Address Auto-Configuration - Auto-configuration IPv6 sans serveur |
| **Router Advertisement** | Annonce du routeur en IPv6 contenant le préfixe réseau |
| **Stateful DHCPv6** | DHCPv6 attribuant les adresses IP |
| **Stateless DHCPv6** | DHCPv6 fournissant uniquement les options (pas d'attribution d'IP) |
| **ISC DHCP** | Internet Systems Consortium DHCP - Serveur DHCP open source pour Linux/Unix |
| **Windows DHCP** | Serveur DHCP intégré à Windows Server |
| **dnsmasq** | Serveur léger DNS et DHCP pour petits réseaux |
| **Vendor Class** | Identifiant du type/modèle de client (option 60) |
| **Domain Search List** | Liste de domaines DNS pour recherche automatique (option 119) |
| **Classless Static Routes** | Routes statiques spécifiques distribuées via DHCP (option 121) |
| **WPAD** | Web Proxy Auto-Discovery Protocol (option 252) |
| **NTP** | Network Time Protocol - Synchronisation de l'heure |
| **WINS** | Windows Internet Name Service - Résolution de noms NetBIOS |
| **Passage à l'échelle** | Capacité d'un système à gérer une croissance importante |
| **Centralisation** | Gestion depuis un point unique |
| **Automatisation** | Processus se déroulant sans intervention manuelle |
| **Résilience** | Capacité à continuer de fonctionner malgré des pannes |

---

**Fin du document de révision DHCP - TSSR**

*Document créé pour la préparation du titre RNCP Technicien Supérieur Systèmes et Réseaux*

*Janvier 2026*
