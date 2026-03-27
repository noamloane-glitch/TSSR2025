# Filtrage réseau

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Filtrage réseau et pare-feux (Firewall)

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Définition du pare-feu|Définition du pare-feu]]
   - [[#Architecture réseau|Architecture réseau]]
   - [[#Principe du filtrage|Principe du filtrage]]
2. [[#Sécurisation réseau|Sécurisation réseau]]
   - [[#Sécurité des systèmes|Sécurité des systèmes]]
   - [[#Défense en profondeur|Défense en profondeur]]
   - [[#Politique de filtrage|Politique de filtrage]]
   - [[#Zones de confiance|Zones de confiance]]
   - [[#DMZ (DeMilitarized Zone)|DMZ]]
3. [[#Types de pare-feux|Types de pare-feux]]
   - [[#Pare-feu sans état (Stateless)|Pare-feu sans état]]
   - [[#Pare-feu à états (Stateful)|Pare-feu à états]]
   - [[#Pare-feu applicatif (DPI)|Pare-feu applicatif]]
   - [[#Pare-feux personnels|Pare-feux personnels]]
4. [[#Solutions en entreprise|Solutions en entreprise]]
   - [[#Critères de sélection|Critères de sélection]]
   - [[#Principales solutions du marché|Principales solutions du marché]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le **filtrage réseau** est un mécanisme fondamental de la sécurité informatique. Il repose sur l'utilisation de **pare-feux (firewalls)** qui contrôlent et filtrent le trafic réseau selon des règles de sécurité définies. En tant que TSSR, tu dois maîtriser les concepts de filtrage pour sécuriser les infrastructures réseau d'entreprise.

### Pourquoi étudier le filtrage réseau ?

En tant que **TSSR**, tu dois :
- Comprendre le fonctionnement des pare-feux pour sécuriser les réseaux
- Savoir concevoir une architecture réseau sécurisée avec zones de confiance
- Maîtriser les différents types de filtrage (stateless, stateful, applicatif)
- Être capable de définir et implémenter des politiques de sécurité
- Connaître les solutions du marché pour conseiller les entreprises

---

## Définition du pare-feu

> [!quote] Définition officielle
> Un **pare-feu** (firewall, garde-barrière, coupe-feu) est un nœud réseau placé à l'intersection de réseaux, en coupure, qui contrôle et filtre les paquets qui le traversent.

### Caractéristiques techniques

> [!info] Fonctionnement au niveau OSI
> - Opère **au minimum au niveau 4** du modèle OSI (couche Transport)
> - Traite les protocoles TCP, UDP et autres protocoles de transport
> - Est généralement **associé à des fonctions de routage**
> - Agit comme un point de contrôle unique entre différents réseaux

### Objectif principal

> [!important] Mission du pare-feu
> Fournir une **connectivité contrôlée et maîtrisée** entre des réseaux de **différents niveaux de confiance**.

---

## Architecture réseau

> [!example] Schéma d'architecture type
> Un pare-feu se positionne typiquement entre :
> - Le **réseau interne** (LAN de l'entreprise)
> - **Internet** (réseau non fiable)
> - D'**autres réseaux** (partenaires, filiales, etc.)

### Représentation schématique

```
┌─────────────────┐
│    Internet     │
│  (Non fiable)   │
└────────┬────────┘
         │
    ┌────▼────┐
    │ Firewall│
    │         │
    └─┬─────┬─┘
      │     │
 ┌────▼──┐ ┌▼───────────┐
 │Réseau │ │Autre réseau│
 │interne│ │(Partenaires)│
 └───────┘ └────────────┘
```

> [!tip] Positionnement stratégique
> Le pare-feu agit comme un **point de passage obligatoire** pour tout trafic entre les zones. C'est le principe de la **coupure réseau**.

---

## Principe du filtrage

> [!info] Idée générale
> Le pare-feu **inspecte tous les paquets** entrants, sortants et traversants, puis prend une décision : **accepter** (accept) ou **refuser** (drop/reject).

### Processus d'inspection

Le pare-feu effectue les opérations suivantes pour chaque paquet :

1. **Déroulement de la pile protocolaire** (au moins jusqu'au niveau 4)
2. **Analyse des en-têtes protocolaires** (adresses IP, ports, protocoles)
3. **Examen du format et contenu des données**
4. **Vérification des informations contextuelles** (comptes utilisateurs, horodatage, etc.)
5. **Décision de filtrage** selon les règles définies

### Actions possibles

> [!note] Types de réponses du pare-feu
> - **Accept** : Le paquet est autorisé à passer
> - **Drop** : Le paquet est rejeté silencieusement (aucune réponse)
> - **Reject** : Le paquet est rejeté avec notification à l'émetteur

> [!warning] Différence Drop vs Reject
> - **Drop** : Plus sécurisé car ne révèle pas l'existence du pare-feu
> - **Reject** : Plus informatif pour le débogage mais expose le pare-feu

---

## Sécurisation réseau

### Sécurité des systèmes

> [!important] Pourquoi filtrer ?
> Trois constats fondamentaux justifient le filtrage réseau :

1. **Tous les systèmes ont des failles de sécurité**
   - Aucun système n'est parfait
   - Des vulnérabilités sont découvertes régulièrement

2. **Les configurations peuvent comporter des défauts**
   - Surtout les configurations par défaut
   - Les paramètres d'usine sont rarement optimaux pour la sécurité

3. **De nombreux services inutiles sont activés par défaut**
   - Augmente la surface d'attaque
   - Services non utilisés = portes ouvertes inutiles

> [!success] Conclusion
> **Besoin impératif de sécurisation du réseau** pour compenser les faiblesses intrinsèques des systèmes.

---

### Défense en profondeur

> [!quote] Principe "Security everywhere"
> La sécurité doit être pensée à tous les niveaux, pas seulement au périmètre.

#### Défense périmétrique

> [!info] Approche traditionnelle
> **Objectif** : Seul le trafic légitime entre et sort du réseau.
> 
> Cette approche consiste à créer une **barrière de sécurité** autour du réseau à protéger.

#### Stratégie recommandée

> [!important] Ne pas faire aveuglément confiance
> **Recommandation** : Ne pas faire aveuglément confiance à nos défenses
> 
> => **Défense en profondeur** (Defense in Depth)

La défense en profondeur implique :
- Plusieurs couches de sécurité successives
- Sécurisation à différents niveaux (réseau, système, application, données)
- Principe : si une couche échoue, les autres compensent
- Monitoring et détection à tous les niveaux

> [!tip] Analogie
> Comme les couches d'un oignon : chaque couche ajoute une protection supplémentaire.

---

### Politique de filtrage

> [!question] Filtrer, mais quoi ?
> Deux approches possibles pour définir les règles de filtrage

#### Tableau comparatif des approches

| Approche | Principe | Avantages | Inconvénients | Recommandation |
|----------|----------|-----------|---------------|----------------|
| **Liste de blocage** (Blacklist) | Définir les paquets à bloquer | Simple à mettre en place initialement | • On peut en oublier<br>• Peu d'alertes en cas d'échec<br>• Approche réactive | ❌ **NON RECOMMANDÉE** |
| **Liste d'autorisation** (Whitelist) | Bloquer tout par défaut, autoriser uniquement le légitime | • Sécurité maximale<br>• Approche proactive<br>• Visibilité sur les tentatives | Plus complexe à mettre en place | ✅ **APPROCHE RECOMMANDÉE** |

#### Principe de base (Best Practice)

> [!success] Politique "Deny All"
> **Étape 1** : Tout bloquer (deny all)
> 
> **Étape 2** : Autoriser uniquement les communications légitimes
> 
> Cette approche garantit qu'aucun trafic non prévu ne peut passer.

> [!example] Exemple concret
> ```
> # Règle par défaut : tout bloquer
> iptables -P INPUT DROP
> iptables -P OUTPUT DROP
> iptables -P FORWARD DROP
> 
> # Puis autoriser explicitement
> iptables -A INPUT -p tcp --dport 443 -j ACCEPT
> iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
> ```

---

### Zones de confiance

> [!important] Organiser son réseau
> La **segmentation du réseau en zones de confiance** est une pratique fondamentale de sécurité.

#### Principe de segmentation

> [!info] Définition
> **Zones de confiance** : Regroupement de tous les nœuds ayant les **mêmes besoins en sécurité**.

### Caractéristiques

- Le **filtrage** se fait entre zones de confiance
- Souvent associé à un **cloisonnement physique** des réseaux
  - Par segmentation physique (câblage séparé)
  - Par **VLAN** (Virtual LAN)

#### Matrice des flux

> [!note] Outil de planification
> La **matrice des flux** définit les communications légitimes entre chaque zone.
> 
> Elle permet de documenter :
> - Quelles zones peuvent communiquer entre elles
> - Quels protocoles et ports sont autorisés
> - Dans quel sens (unidirectionnel ou bidirectionnel)

### Tableau exemple de matrice des flux

| Depuis / Vers | Internet | DMZ | LAN Clients | LAN Serveurs | Admin |
|---------------|----------|-----|-------------|--------------|-------|
| **Internet** | - | HTTP/HTTPS | ❌ | ❌ | ❌ |
| **DMZ** | HTTP/HTTPS | - | ❌ | MySQL | ❌ |
| **LAN Clients** | HTTP/HTTPS | ❌ | - | Tous | ❌ |
| **LAN Serveurs** | Updates | ❌ | ❌ | - | ❌ |
| **Admin** | ✅ | SSH/RDP | SSH/RDP | SSH/RDP | - |

> [!tip] Filtrage fort pour les postes clients
> De nombreuses zones peuvent être **fortement filtrées**. Par exemple, pour l'ensemble des machines clientes : **sortie uniquement** (pas d'entrée non sollicitée).

---

### DMZ (DeMilitarized Zone)

> [!quote] Une zone à surveiller
> Certains serveurs doivent être accessibles depuis l'extérieur (mail, web, DNS public...). Ils sont placés dans des zones appelées **DMZ**.

#### Caractéristiques de la DMZ

> [!info] Définition et usage
> - Zone **intermédiaire** entre Internet et le réseau interne
> - Héberge les serveurs **accessibles publiquement**
> - Fait l'objet d'une **surveillance et d'un filtrage spécifiques**
> - Considérés comme des **points d'entrée potentiels** dans le réseau

#### Serveurs typiques en DMZ

- **Serveur Web** (HTTP/HTTPS)
- **Serveur de messagerie** (SMTP, IMAP)
- **Serveur DNS** public
- **Serveur VPN** (point d'accès)
- **Serveur FTP** public
- **Serveur de proxy inverse**

> [!warning] Risques de la DMZ
> Les serveurs en DMZ peuvent être des **points d'entrée** dans le réseau :
> - Exposition directe à Internet
> - Cible privilégiée des attaquants
> - Nécessitent un durcissement (hardening) important
> - Surveillance renforcée (logs, IDS/IPS)

#### Exemple d'architecture avec DMZ

> [!example] Architecture multi-zones typique
> ```
> Internet
>    │
>    ▼
> [Firewall 1]
>    │
>    ├─────────► DMZ (Serveurs publics)
>    │              │
>    ▼              │
> [Firewall 2]      │
>    │              │
>    ├──────────────┘
>    │
>    ├─────────► LAN Interne (Postes clients)
>    │
>    ├─────────► Zone Serveurs internes
>    │
>    └─────────► Réseau WiFi Visiteurs
> ```

> [!tip] Double pare-feu
> Une architecture avec **deux pare-feux** (en cascade) offre une protection supplémentaire :
> - Firewall 1 : filtre Internet ↔ DMZ
> - Firewall 2 : filtre DMZ ↔ LAN interne

---

## Types de pare-feux

> [!abstract] Vue d'ensemble
> Il existe plusieurs types de pare-feux, classés selon leur **niveau d'inspection** et leur **gestion de l'état des connexions**.

---

### Pare-feu sans état (Stateless)

> [!info] Le plus simple et le plus ancien
> **Stateless firewall** : Filtrage simple basé uniquement sur les en-têtes des paquets, sans mémorisation des connexions.

#### Caractéristiques

**Critères de filtrage** :
- **Adresses source et/ou destination** (IP)
- **Ports** source et destination (UDP, TCP)
- **Protocoles** utilisés (TCP, UDP, ICMP, etc.)
- **Options** des protocoles
- **Interface** réseau (entrée/sortie)

#### Avantages

> [!success] Points forts
> - ✅ **Rapide** : traitement simple et direct
> - ✅ **Efficace** : faible consommation de ressources
> - ✅ **Prévisible** : comportement déterministe

#### Limites

> [!warning] Inconvénients majeurs
> - ❌ **Chaque paquet traité indépendamment** (pas de notion de session)
> - ❌ **Règles complexes et/ou nombreuses** nécessaires
> - ❌ **Pas de contexte** : ne sait pas si un paquet fait partie d'une connexion établie
> - ❌ **Difficile de filtrer finement** les protocoles à états comme TCP

> [!example] Exemple de règles stateless
> ```bash
> # Autoriser SSH entrant depuis un réseau spécifique
> iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -j ACCEPT
> 
> # Autoriser HTTP sortant
> iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
> 
> # Problème : il faut aussi autoriser les réponses !
> iptables -A INPUT -p tcp --sport 80 -j ACCEPT
> ```

---

### Pare-feu à états (Stateful)

> [!info] Avec de la mémoire
> **Stateful firewall** : Pare-feu qui conserve en mémoire l'état des connexions pour un filtrage contextuel.

#### Fonctionnement

> [!important] Suivi des connexions
> - **Suit les connexions** des protocoles à états (principalement TCP)
> - **Vérifie la conformité** d'un paquet dans son contexte
> - **Autorisation implicite des réponses** aux requêtes légitimes
> - Maintient une **table d'états** (connection tracking)

#### Avantages

> [!success] Points forts
> - ✅ **Filtrage plus poussé** et contextuel
> - ✅ **Allège l'écriture des règles** (pas besoin de règles pour les réponses)
> - ✅ **Plus sécurisé** : détecte les paquets hors contexte
> - ✅ **Gère mieux les protocoles complexes** (FTP, SIP, etc.)

#### Limites

> [!warning] Inconvénients
> - ❌ **Nécessite plus de ressources** (mémoire pour la table d'états, CPU)
> - ❌ **Vulnérable aux attaques par épuisement** de la table d'états
> - ❌ **Complexité accrue** dans la gestion

> [!example] Exemple de règles stateful
> ```bash
> # Avec iptables (stateful)
> # Autoriser les connexions établies et reliées
> iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
> 
> # Autoriser nouvelle connexion SSH depuis 192.168.1.0/24
> iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 22 -m state --state NEW -j ACCEPT
> 
> # Pas besoin de règle pour les réponses SSH !
> # Elles sont automatiquement autorisées par la première règle
> ```

> [!tip] Solution recommandée
> **pfSense** est un exemple de solution open-source utilisant le filtrage stateful.

#### États de connexion TCP

| État | Description |
|------|-------------|
| **NEW** | Nouvelle connexion initiée |
| **ESTABLISHED** | Connexion établie (échange en cours) |
| **RELATED** | Connexion liée à une connexion existante |
| **INVALID** | Paquet ne correspondant à aucune connexion connue |

---

### Pare-feu applicatif (DPI)

> [!info] Fouille complète
> **Deep Packet Inspection (DPI)** : Inspection en profondeur des paquets, jusqu'au niveau applicatif.

#### Fonctionnement

> [!important] Analyse complète de la pile
> - **Déroule l'intégralité de la pile protocolaire** (jusqu'au niveau 7)
> - **Examine le contenu des données** applicatives
> - **Vérifie la conformité** du paquet avec les protocoles utilisés
> - **Détecte les anomalies** dans les protocoles

#### Capacités avancées

- **Filtrage de protocoles difficiles** (ex : FTP avec ses connexions multiples)
- **Détection d'intrusions** au niveau applicatif
- **Blocage de contenu** spécifique (malware, scripts malveillants)
- **Contrôle d'applications** (reconnaissance des applications malgré les ports non standards)

#### Nécessite une connaissance approfondie

> [!note] Exigences techniques
> Le pare-feu doit avoir une **connaissance du protocole applicatif** pour :
> - Comprendre la structure des données
> - Valider la conformité
> - Détecter les comportements anormaux

#### Limite majeure

> [!warning] Problème avec le chiffrement
> **Inconvénient** : Impossible dans le cas de **chiffrement de bout en bout** (end-to-end encryption)
> 
> - TLS/SSL empêche l'inspection du contenu
> - Solution : déchiffrement TLS (avec problèmes de confidentialité)
> - Alternative : inspection des métadonnées (SNI, certificats)

#### Spécialisation : WAF

> [!info] Web Application Firewall
> Les **WAF** sont des pare-feux applicatifs spécialisés dans **HTTP/HTTPS** :
> - Protection contre les attaques web (SQLi, XSS, CSRF, etc.)
> - Analyse du contenu HTML, JSON, XML
> - Protection des APIs
> - Règles OWASP Top 10

> [!example] Solutions WAF connues
> - ModSecurity (open-source)
> - Cloudflare WAF
> - AWS WAF
> - F5 BIG-IP ASM

#### Implémentation avec pfSense

> [!tip] Modules additionnels pfSense
> pfSense peut faire du DPI partiellement avec des modules additionnels :
> - **Snort** : IDS/IPS avec DPI
> - **Squid** : Proxy avec filtrage de contenu
> - **pfBlockerNG** : Blocage par réputation IP/domaine

---

### Pare-feux personnels

> [!info] Chacun son pare-feu
> **Pare-feu personnel** : Logiciel filtrant le trafic entrant/sortant d'une **unique machine**.

#### Caractéristiques

- Installé directement sur le **poste de travail** ou serveur
- Filtre le trafic **local** (pas le trafic réseau global)
- Peut fonctionner en mode **interactif** (demande à l'utilisateur)

#### Avantages

> [!success] Point fort
> ✅ **Filtrage interactif** : l'utilisateur peut approuver/bloquer les connexions en temps réel

#### Inconvénients

> [!warning] Limites de sécurité
> - ❌ **Un service parmi d'autres** => moins sûr qu'un équipement dédié
> - ❌ Peut être désactivé par l'utilisateur
> - ❌ Vulnérable si le système est compromis
> - ❌ Consomme des ressources système
> - ❌ Gestion décentralisée (difficile à administrer en masse)

> [!example] Exemples de pare-feux personnels
> **Windows** :
> - Windows Defender Firewall (intégré)
> 
> **Linux** :
> - `iptables` / `nftables`
> - `ufw` (Uncomplicated Firewall)
> - `firewalld`
> 
> **Tiers** :
> - ZoneAlarm
> - Comodo Firewall
> - GlassWire

> [!tip] Usage en entreprise
> En entreprise, les pare-feux personnels sont souvent **gérés centralement** via GPO (Windows) ou outils de gestion de configuration (Ansible, Chef, etc.).

---

## Solutions en entreprise

> [!abstract] Choix d'une solution professionnelle
> Le choix d'un pare-feu en entreprise est crucial et doit être fait en fonction de critères précis, adaptés aux besoins et contraintes de l'organisation.

---

### Critères de sélection

> [!important] Comment choisir ?
> **Disclaimer** : Il n'y a pas de solution meilleure universellement qu'une autre. Tout est une question d'**adéquation avec les besoins métier**, la roadmap technique et la stratégie de sécurité.

#### Critères principaux

| Critère | Description | Questions à se poser |
|---------|-------------|---------------------|
| **Complexité de l'infrastructure** | Taille et organisation du réseau | • Combien de sites ?<br>• Quelle volumétrie de trafic ?<br>• Architecture simple ou complexe ? |
| **Intégration avec l'existant** | Compatibilité avec l'environnement actuel | • Équipements déjà en place ?<br>• Protocoles utilisés ?<br>• Outils de supervision existants ? |
| **Budget et performances** | Coût vs besoins en débit | • Budget disponible (CAPEX/OPEX) ?<br>• Débit nécessaire ?<br>• Évolution prévue ? |
| **Compétences en interne** | Capacité de gestion et maintenance | • Équipe formée ?<br>• Support nécessaire ?<br>• Complexité acceptable ? |
| **Orientation Cloud/Hybride** | Stratégie d'infrastructure | • Cloud public/privé/hybride ?<br>• Télétravail massif ?<br>• Multi-cloud ? |

> [!tip] Approche recommandée
> Réaliser une **matrice de décision** en pondérant chaque critère selon son importance pour votre organisation.

---

### Principales solutions du marché

> [!info] Panorama des leaders du marché
> Voici un comparatif des principales solutions de pare-feux professionnels.

---

#### Check Point

> [!quote] Expertise historique
> **Check Point** : Pionnier du pare-feu stateful inspection (années 90), leader historique de la sécurité réseau.

**Points forts** :
- ✅ **Expertise historique** en sécurité (depuis les années 90)
- ✅ **Gestion unifiée** via plateforme R80
- ✅ **Prévention des menaces consolidée** avec ThreatCloud (intelligence artificielle globale)
- ✅ Large gamme de produits (des PME aux très grandes entreprises)
- ✅ Écosystème riche et mature

**Technologies clés** :
- **Infinity Architecture** : plateforme de sécurité unifiée
- **ThreatCloud** : base de renseignements sur les menaces alimentée par l'IA
- **SandBlast** : protection contre les menaces zero-day

> [!success] Idéal pour
> **Grands comptes** sensibles à la conformité et à la prévention des **cyberattaques avancées** (APT, zero-day).

**Secteurs privilégiés** :
- Finance et banque
- Secteur public
- Santé
- Grandes entreprises internationales

---

#### Fortinet (FortiGate)

> [!quote] Performance et prix
> **Fortinet** : Excellent rapport qualité/prix avec une suite de sécurité complète.

**Points forts** :
- ✅ **Rapport performance/prix** excellent
- ✅ **Suite de sécurité unifiée** (Fortinet Security Fabric)
- ✅ **Gestion centralisée** avec FortiManager
- ✅ **Large gamme** de modèles (de la TPE au datacenter)
- ✅ Performances élevées (débit, latence)

**Technologies clés** :
- **Security Fabric** : intégration de l'ensemble des produits Fortinet
- **FortiGuard** : services de renseignements sur les menaces
- **ASIC propriétaires** : accélération matérielle des traitements de sécurité
- **SD-WAN** intégré

> [!success] Idéal pour
> **PME/ETI**, environnements **multi-sites**, et organisations cherchant une **protection complète** sans multiplier les solutions.

**Cas d'usage typiques** :
- PME en croissance
- Retail avec multiples magasins
- Entreprises avec sites distants
- SD-WAN sécurisé

---

#### Cisco ASA (avec FirePower)

> [!quote] Intégration réseau native
> **Cisco ASA** : Solution éprouvée pour les environnements Cisco, avec ajout de capacités avancées via FirePower.

**Points forts** :
- ✅ **Intégration réseau native** (routeurs, switches Cisco)
- ✅ **Écosystème Cisco étendu** (gestion unifiée)
- ✅ **Fiabilité éprouvée** depuis de nombreuses années
- ✅ Support et documentation de qualité
- ✅ Compatibilité avec Cisco ISE (contrôle d'accès réseau)

**Technologies clés** :
- **ASA** : pare-feu historique de Cisco
- **FirePower** : module NGFW (Next-Gen Firewall) avec IPS
- **Cisco Talos** : intelligence sur les menaces

> [!note] Évolution produit
> Nouveau produit : **Cisco Firepower Threat Defense (FTD)**
> - Remplace progressivement ASA
> - Combine ASA et FirePower dans une plateforme unifiée
> - Interface de gestion modernisée (FMC - Firepower Management Center)

> [!success] Idéal pour
> Les environnements **Cisco existants**, les grands groupes avec des besoins de sécurité avancés et de **segmentation fine** (micro-segmentation).

**Secteurs privilégiés** :
- Entreprises "full Cisco"
- Campus universitaires
- Administrations
- Grands groupes internationaux

> [!warning] Point d'attention
> Migration ASA → FTD peut être complexe et nécessite une planification rigoureuse.

---

#### Palo Alto Networks

> [!quote] Visibilité et détection avancée
> **Palo Alto Networks** : Leader du NGFW, reconnu pour sa capacité à identifier et contrôler les applications.

**Points forts** :
- ✅ **Détection avancée des menaces** avec WildFire (sandboxing cloud)
- ✅ **Politique de sécurité basée sur les applications** (App-ID)
- ✅ **Vision claire du trafic** applicatif (identification par signatures + heuristique)
- ✅ Interface intuitive et riche
- ✅ Zero Trust Architecture native

**Technologies clés** :
- **App-ID** : identification des applications (signatures + analyse comportementale)
- **User-ID** : identification des utilisateurs (intégration AD/LDAP)
- **Content-ID** : analyse du contenu (antivirus, anti-spyware, filtrage URL)
- **WildFire** : sandboxing cloud pour les menaces zero-day
- **Prisma** : SASE (Secure Access Service Edge) pour le cloud

**Philosophie** :
- Focus sur l'**identité** et l'**application** plutôt que sur les ports/IPs
- Approche **Zero Trust** (ne jamais faire confiance, toujours vérifier)

> [!success] Idéal pour
> Entreprises voulant une **visibilité applicative fine** et une **protection contre les menaces zero-day**.

**Cas d'usage typiques** :
- Organisations soucieuses de la sécurité avancée
- Entreprises avec besoins de visibility (Shadow IT)
- Migration vers le cloud avec Prisma SASE
- Environnements Zero Trust

> [!warning] Point d'attention
> **Coût élevé** (premium) - investissement important en licences (abonnements obligatoires pour les fonctionnalités avancées).

---

### Tableau comparatif récapitulatif

| Solution | Prix | Performance | Facilité | Cloud/Hybrid | Cas d'usage privilégiés |
|----------|------|-------------|----------|--------------|------------------------|
| **Check Point** | 💰💰💰💰 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | Grandes entreprises, finance |
| **Fortinet** | 💰💰💰 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | PME/ETI, multi-sites |
| **Cisco ASA/FTD** | 💰💰💰💰 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | Environnements Cisco |
| **Palo Alto** | 💰💰💰💰💰 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Sécurité avancée, Zero Trust |

---

## Point d'attention moderne

> [!warning] La sécurité ne se résume pas au pare-feu
> **Constat important** : La cybersécurité ne se résume plus à un firewall ou à un antivirus.

### Approche moderne de la sécurité

> [!important] Architecture de résilience
> Elle repose aujourd'hui sur une **architecture pensée** pour :
> - **Résister** aux attaques (durcissement, défense en profondeur)
> - **Détecter** les intrusions et comportements anormaux (SIEM, EDR, NDR)
> - **Contenir** les incidents (micro-segmentation, Zero Trust)
> - **Se relever** rapidement face aux attaques (PRA, PCA, sauvegarde)

### Stratégie recommandée

> [!success] Security by Design & Resilience by Default
> **Approche "Security by Design & Resilience by Default"**
> 
> Alignée avec les attentes des :
> - **RSSI** (Responsable de la Sécurité des Systèmes d'Information)
> - **DSI** (Directeur des Systèmes d'Information)
> - **Directions Générales**

### Composantes d'une stratégie de sécurité moderne

1. **Prévention** : Pare-feux, WAF, antimalware, durcissement
2. **Détection** : SIEM, IDS/IPS, EDR, NDR, SOC
3. **Réponse** : SOAR, playbooks, équipe incident response
4. **Récupération** : Sauvegardes, PRA/PCA, exercices de crise
5. **Gouvernance** : Politiques, formation, sensibilisation

> [!tip] Approche Zero Trust
> Le **Zero Trust** devient le nouveau paradigme :
> - Ne jamais faire confiance
> - Toujours vérifier
> - Limiter les accès au strict nécessaire (least privilege)
> - Micro-segmentation

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concept de pare-feu

- **Définition** : Nœud réseau en coupure qui filtre le trafic au minimum au niveau 4 (Transport)
- **Objectif** : Connectivité contrôlée entre réseaux de différents niveaux de confiance
- **Actions** : Accept, Drop, Reject selon les règles définies

### Architecture et zones de confiance

- **Segmentation** : Organiser le réseau en zones de confiance (LAN, DMZ, Internet, Wifi visiteurs, etc.)
- **DMZ** : Zone intermédiaire hébergeant les serveurs publics, fortement surveillée
- **Matrice des flux** : Définit les communications légitimes entre zones
- **Cloisonnement** : Physique ou logique (VLAN)

### Politique de filtrage

- **Approche recommandée** : Liste d'autorisation (Whitelist)
- **Principe de base** : Deny All par défaut, puis autoriser le strict nécessaire
- **Défense en profondeur** : Multiple couches de sécurité, ne pas se fier à une seule protection
- **Security everywhere** : Sécurité à tous les niveaux

### Types de pare-feux

| Type | Caractéristiques | Avantages | Inconvénients |
|------|------------------|-----------|---------------|
| **Stateless** | Filtre simple sur en-têtes | Rapide, efficace | Pas de contexte, règles complexes |
| **Stateful** | Suivi des connexions | Filtrage contextuel, règles simplifiées | Ressources nécessaires |
| **Applicatif (DPI)** | Inspection niveau 7 | Détection fine, protocoles complexes | Problème avec chiffrement, lourd |
| **Personnel** | Installé sur chaque machine | Filtrage interactif | Moins sûr, décentralisé |

### Solutions professionnelles

- **Check Point** : Historique, grandes entreprises, sécurité avancée
- **Fortinet** : Rapport qualité/prix, PME/ETI, multi-sites
- **Cisco ASA/FTD** : Intégration Cisco, environnements existants
- **Palo Alto** : Visibilité applicative, Zero Trust, premium

### Points de vigilance

- ⚠️ Le pare-feu ne fait pas tout : approche globale de sécurité nécessaire
- ⚠️ La DMZ nécessite une surveillance renforcée
- ⚠️ Drop vs Reject : Drop plus sécurisé (ne révèle pas le pare-feu)
- ⚠️ Chiffrement end-to-end : limite le DPI
- ⚠️ Pas de solution universellement meilleure : adapter au contexte

### Bonnes pratiques TSSR

1. **Toujours** documenter les règles de filtrage et la matrice des flux
2. **Privilégier** l'approche whitelist (deny all + allow)
3. **Segmenter** le réseau en zones de confiance logiques
4. **Surveiller** activement les logs du pare-feu
5. **Mettre à jour** régulièrement les règles et le firmware
6. **Tester** les règles après modification (validation)
7. **Sauvegarder** la configuration régulièrement
8. **Implémenter** la défense en profondeur (pas que le pare-feu)

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Pare-feu (Firewall)** | Équipement ou logiciel de sécurité qui filtre le trafic réseau selon des règles définies, placé en coupure entre réseaux |
| **Stateless** | Mode de filtrage où chaque paquet est traité indépendamment, sans mémorisation du contexte ou de l'état des connexions |
| **Stateful** | Mode de filtrage avec suivi des connexions (connection tracking), permettant un filtrage contextuel basé sur l'état de la session |
| **DPI (Deep Packet Inspection)** | Inspection en profondeur des paquets jusqu'au niveau applicatif (niveau 7 OSI) pour analyse du contenu |
| **DMZ (DeMilitarized Zone)** | Zone intermédiaire du réseau hébergeant les serveurs accessibles publiquement, positionnée entre Internet et le réseau interne |
| **Zone de confiance** | Segment réseau regroupant des équipements ayant les mêmes besoins et niveaux de sécurité |
| **Matrice des flux** | Document définissant les communications autorisées entre les différentes zones de confiance (qui peut parler à qui, comment) |
| **Accept** | Action du pare-feu autorisant le passage d'un paquet |
| **Drop** | Action rejetant silencieusement un paquet (pas de notification à l'émetteur) |
| **Reject** | Action rejetant un paquet avec envoi d'une notification à l'émetteur (ICMP unreachable) |
| **Whitelist (Liste d'autorisation)** | Approche de filtrage où tout est bloqué par défaut, seul le trafic explicitement autorisé passe |
| **Blacklist (Liste de blocage)** | Approche de filtrage où tout est autorisé par défaut, seul le trafic explicitement interdit est bloqué (non recommandée) |
| **Défense en profondeur** | Stratégie de sécurité multicouche où plusieurs niveaux de protection se complètent |
| **Défense périmétrique** | Protection du réseau au niveau de ses frontières (entrée/sortie) |
| **WAF (Web Application Firewall)** | Pare-feu applicatif spécialisé dans la protection des applications web (HTTP/HTTPS) contre les attaques type OWASP Top 10 |
| **NGFW (Next-Generation Firewall)** | Pare-feu de nouvelle génération combinant filtrage stateful, DPI, IPS, analyse de malware, etc. |
| **IDS (Intrusion Detection System)** | Système de détection d'intrusion qui analyse le trafic pour identifier des comportements suspects (alerte uniquement) |
| **IPS (Intrusion Prevention System)** | Système de prévention d'intrusion qui détecte et bloque activement les attaques |
| **Connection tracking** | Mécanisme de suivi des connexions réseau (état NEW, ESTABLISHED, RELATED, INVALID) |
| **Coupure réseau** | Positionnement du pare-feu où tout le trafic doit obligatoirement le traverser (passage forcé) |
| **Zero Trust** | Modèle de sécurité "ne jamais faire confiance, toujours vérifier", sans zone de confiance implicite |
| **SASE (Secure Access Service Edge)** | Architecture combinant fonctions réseau (SD-WAN) et sécurité (pare-feu, CASB, etc.) en mode cloud |
| **Segmentation** | Division du réseau en segments isolés pour limiter la propagation des attaques |
| **VLAN (Virtual LAN)** | Réseau local virtuel permettant de segmenter logiquement un réseau physique |
| **App-ID** | Technologie d'identification des applications indépendamment des ports utilisés (Palo Alto) |
| **Sandboxing** | Technique d'analyse de fichiers suspects dans un environnement isolé pour détecter les comportements malveillants |
| **ThreatCloud** | Base de renseignements sur les menaces alimentée par IA (Check Point) |
| **FortiGuard** | Services de renseignements sur les menaces et mises à jour de signatures (Fortinet) |
| **WildFire** | Service de sandboxing cloud pour analyse de malware (Palo Alto) |
| **Security Fabric** | Architecture unifiée intégrant l'ensemble des produits Fortinet |

---

## Conclusion

> [!abstract] Résumé final
> Le **filtrage réseau** via pare-feux est un pilier fondamental de la sécurité des infrastructures modernes.

### Ce que tu dois maîtriser en tant que TSSR

1. **Concepts fondamentaux** : Notion de pare-feu, architecture réseau, zones de confiance, DMZ
2. **Types de filtrage** : Différences entre stateless, stateful et applicatif (DPI)
3. **Politique de sécurité** : Approche whitelist, deny all, défense en profondeur
4. **Architecture** : Segmentation réseau, matrice des flux, placement des pare-feux
5. **Solutions du marché** : Critères de choix, principales solutions professionnelles

### Vision moderne

> [!important] Au-delà du pare-feu
> La cybersécurité ne se résume pas au pare-feu :
> - **Architecture de résilience** : résister, détecter, contenir, récupérer
> - **Security by Design** : intégrer la sécurité dès la conception
> - **Zero Trust** : nouveau paradigme de sécurité
> - **Approche globale** : SIEM, EDR, formation, gouvernance

### Pour aller plus loin

- Pratique sur **pfSense** (pare-feu open-source)
- Découverte des solutions professionnelles (labs, démos)
- Étude des attaques réseau pour comprendre les protections
- Approfondissement **iptables/nftables** sous Linux
- Certification **NSE (Fortinet)**, **PCNSE (Palo Alto)**, **CCNA Security (Cisco)**

---

> [!success] Prêt pour le titre RNCP !
> Tu disposes maintenant d'une vision complète du filtrage réseau et des pare-feux. Continue à pratiquer sur ton lab Proxmox avec pfSense pour consolider tes connaissances ! 🚀🔒

---

**📚 Document créé pour la préparation au titre RNCP TSSR**

**🎯 Objectif** : Maîtriser les concepts de filtrage réseau pour l'examen et la pratique professionnelle

**✅ Compatible Obsidian** avec callouts natifs et liens internes