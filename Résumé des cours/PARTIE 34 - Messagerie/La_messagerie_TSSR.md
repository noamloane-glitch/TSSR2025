## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : La messagerie - Un outil de communication  
**Date** : Février 2026  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Les origines|Les origines]]
   - [[#La fonction|La fonction]]
   - [[#Analogie avec le courrier postal|Analogie avec le courrier postal]]
   - [[#L'adresse électronique|L'adresse électronique]]
   - [[#Le courrier électronique|Le courrier électronique]]
   - [[#La boîte aux lettres électronique|La boîte aux lettres électronique]]
   - [[#En entreprise|En entreprise]]
   - [[#Les messageries instantanées|Les messageries instantanées]]
   - [[#On-premises ou cloud|On-premises ou cloud]]
   - [[#Terminologie|Terminologie]]

2. [[#Clients et serveurs|Clients et serveurs]]
   - [[#Glossaire technique messagerie|Glossaire technique messagerie]]
   - [[#Fonctionnement de l'infrastructure|Fonctionnement de l'infrastructure]]
   - [[#Le client de messagerie|Le client de messagerie]]
   - [[#Le serveur de messagerie|Le serveur de messagerie]]
   - [[#Exemple pratique Bob et Alice|Exemple pratique : Bob et Alice]]
   - [[#Serveur MX|Serveur MX]]
   - [[#Fonctionnement détaillé|Fonctionnement détaillé]]

3. [[#Les protocoles|Les protocoles]]
   - [[#SMTP|SMTP]]
   - [[#POP3|POP3]]
   - [[#IMAP|IMAP]]
   - [[#Comparaison POP vs IMAP|Comparaison POP vs IMAP]]

4. [[#Bonnes pratiques|Bonnes pratiques]]
   - [[#Gestion des emails|Gestion des emails]]
   - [[#Sécurité|Sécurité]]
   - [[#Confidentialité|Confidentialité]]
   - [[#Utilisation responsable|Utilisation responsable]]

5. [[#Points clés à retenir|Points clés à retenir]]

6. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> La messagerie électronique est un système de communication essentiel dans le monde professionnel moderne. Ce document synthétise les concepts fondamentaux, les protocoles et les bonnes pratiques nécessaires pour la maîtrise de cet outil en tant que Technicien Supérieur Systèmes et Réseaux.

### Pourquoi étudier la messagerie ?

En tant que **TSSR**, tu dois maîtriser la messagerie électronique car :
- C'est un **système critique** en entreprise qui nécessite une surveillance continue
- Tu seras responsable de l'installation, la configuration et la maintenance des serveurs de messagerie
- Tu dois comprendre les protocoles pour résoudre les problèmes de communication
- La sécurité des emails est un enjeu majeur pour la protection des données

---

## Les origines

> [!info] Histoire de la messagerie électronique
> La messagerie électronique a évolué depuis les années 1960 pour devenir l'outil de communication professionnel incontournable d'aujourd'hui.

**Chronologie historique** :

| Année | Événement |
|-------|-----------|
| **1965** | Création du courrier électronique entre le SDC et le MIT |
| **1969** | Développement du courrier électronique via ARPANET |
| **1971** | Création du signe arobase "@" par **Ray Tomlinson** |

> [!example] Première adresse email
> La toute première adresse de courrier électronique était : `tomlinson@bbn-tenexa`

---

## La fonction

> [!quote] Définition officielle
> La messagerie électronique est un **système de communication en mode texte de type asynchrone** utilisé sur un réseau informatique.

### Caractéristiques principales

**Mode asynchrone** :
- Les messages sont **envoyés et reçus avec une latence** qui peut être plus ou moins importante
- Contrairement à la messagerie instantanée, la réception n'est pas immédiate ni garantie en temps réel

**Contenu** :
- Messages en mode texte (brut ou HTML)
- Possibilité d'adjoindre des **images** et des **fichiers**

> [!example] Exemples de systèmes de messagerie
> - **Microsoft Outlook** : client de messagerie professionnel
> - **Gmail** : service de messagerie web de Google
> - **Mozilla Thunderbird** : client de messagerie open source

---

## Analogie avec le courrier postal

> [!tip] Mnémotechnique
> Pour comprendre le fonctionnement de la messagerie électronique, pense au système postal classique !

### Tableau comparatif

| Étape | Courrier postal | Courrier électronique |
|-------|-----------------|----------------------|
| **1. Rédaction** | Écriture d'une lettre et mise sous enveloppe | Rédaction d'un email dans une interface dédiée |
| **2. Dépôt** | Le courrier est posté dans une boîte aux lettres d'un bureau de La Poste | L'email est envoyé sur le serveur SMTP de l'expéditeur (= La Poste) |
| **3. Acheminement** | La Poste envoie le courrier au destinataire | Le serveur SMTP envoie l'email au serveur du destinataire |
| **4. Réception** | Le destinataire trouve son courrier à son adresse postale, dans sa boîte aux lettres | Le destinataire se connecte à sa boîte mail et consulte l'email (en POP ou en IMAP) |

> [!important] Concept clé
> Le **serveur SMTP** joue le rôle de **La Poste** dans le système de messagerie électronique : il achemine les messages du serveur expéditeur vers le serveur destinataire.

---

## L'adresse électronique

> [!quote] Définition RFC 5322
> L'adresse électronique (ou adresse email) est composée d'une chaîne de caractères permettant de recevoir du courrier électronique dans une boîte aux lettres électronique. Elle est définie par la **RFC 5322**.

### Structure d'une adresse électronique

Une adresse email est composée de **3 éléments** :

```
partie_locale@nom_de_domaine
```

| Élément | Description | Exemple |
|---------|-------------|---------|
| **Partie locale** | Identifiant de l'utilisateur | `jean.dupont` |
| **Séparateur @** | Arobase qui signifie "à" ou "chez" | `@` |
| **Nom de domaine** | Domaine de l'organisation | `entreprise.fr` |

> [!example] Exemple complet
> `jean.dupont@entreprise.fr`
> - **Partie locale** : `jean.dupont`
> - **Séparateur** : `@`
> - **Domaine** : `entreprise.fr`

---

## Le courrier électronique

> [!info] Structure d'un email
> Un courrier électronique (communément appelé **email**) est composé de 2 parties principales.

### Composition d'un email

**1. L'en-tête (Header)**
- Contient les **informations contextuelles** :
  - Expéditeur (From)
  - Destinataire(s) (To, Cc, Bcc)
  - Objet (Subject)
  - Date d'envoi
  - Serveurs de transit
  - Identifiants techniques

**2. Le corps du message (Body)**
- Le **message en lui-même**
- Codé sous forme de texte :
  - **Texte brut** (plain text)
  - **HTML** (formatage enrichi)

> [!note] Terminologie
> Le terme "email" sera utilisé dans la suite de ce document pour désigner le courrier électronique.

---

## La boîte aux lettres électronique

> [!quote] Définition RFC 5322
> Une boîte aux lettres électronique est la **destination** vers laquelle les courriers électroniques sont livrés. Elle est identifiée par une **adresse électronique**.

### Caractéristiques

**Double signification** du terme "boîte aux lettres" :
1. Le **volume de stockage** dédié localement ou distant (sur serveur)
2. Une **entité conceptuelle** qui ne concerne pas nécessairement le stockage de fichiers (selon RFC 5322)

**Gestion des accès** :
- Des accès en **Lecture/Écriture** peuvent être mis en place
- Permet la création de **boîtes aux lettres partagées** (BAL partagées)

> [!example] Cas d'usage des BAL partagées
> - `support@entreprise.fr` : accessible par toute l'équipe support
> - `contact@entreprise.fr` : accessible par plusieurs collaborateurs
> - `commercial@entreprise.fr` : partagée entre commerciaux

---

## En entreprise

> [!important] Système critique
> La messagerie électronique est un des **systèmes de communication de base** en entreprise, l'autre étant le téléphone. C'est un **système critique** qui doit être **surveillé en continu**.

### Fonctionnalités additionnelles

En général, le service de messagerie est accompagné d'autres fonctionnalités :

| Fonctionnalité | Description |
|----------------|-------------|
| **Contacts partagés** | Annuaires d'entreprise accessibles à tous |
| **Gestion des ressources** | Réservation de salles de réunion, équipements |
| **Webmail** | Accès web à la messagerie depuis n'importe où |
| **Calendrier partagé** | Planification et organisation d'événements |

> [!warning] Responsabilité du TSSR
> En tant que TSSR, tu devras assurer :
> - La **disponibilité** du service (24/7)
> - La **sécurité** des communications
> - La **sauvegarde** des données
> - La **maintenance** et les mises à jour

---

## Les messageries instantanées

> [!quote] Définition
> Une messagerie instantanée (ou "chat") est un système de communication **en temps réel** pour l'échange de messages de type texte.

### Différences avec l'email

| Critère | Email | Messagerie instantanée |
|---------|-------|------------------------|
| **Mode** | Asynchrone | **Synchrone** (temps réel) |
| **Latence** | Variable (secondes à heures) | Immédiate |
| **Usage** | Communication formelle, documentation | Communication rapide, informelle |
| **Stockage** | Long terme | Court terme généralement |

> [!example] Exemples de messageries instantanées
> - **WhatsApp** : messagerie grand public
> - **Slack** : communication d'équipe professionnelle
> - **Microsoft Teams** : plateforme collaborative
> - **Skype** : visioconférence et chat
> - **Messenger** : messagerie Facebook

> [!note] Distinction importante
> La messagerie instantanée permet une **communication rapide entre utilisateurs** grâce à l'envoi et la réception **immédiats** des messages.

---

## On-premises ou cloud

> [!info] Modes de déploiement
> Il existe 2 modes principaux de déploiement et d'hébergement pour les systèmes de messagerie.

### Tableau comparatif

| Critère | On-premises (Local) | Cloud (Distant) |
|---------|---------------------|-----------------|
| **Infrastructure** | Serveurs locaux appartenant à l'entreprise | Serveurs distants gérés par un fournisseur |
| **Investissement initial** | Achats matériels importants | Abonnement mensuel/annuel |
| **Installation** | À charge de l'entreprise | Gérée par le fournisseur |
| **Configuration** | Personnalisable à 100% | Limitée aux options du fournisseur |
| **Maintenance** | Gestion interne (sauvegarde, MAJ, monitoring) | Gérée par le fournisseur |
| **Infrastructure réseau** | Nécessaire (serveurs, liens, sécurité) | Minimale (connexion Internet) |
| **Accessibilité** | Dépend du réseau de l'entreprise | Depuis n'importe quel terminal autorisé |
| **Contrôle des données** | Total | Partagé avec le fournisseur |
| **Dépendance** | Aucune (autonomie) | Dépendance vis-à-vis du fournisseur cloud |

> [!tip] Choix stratégique
> **On-premises** : pour les organisations nécessitant un contrôle total des données et de l'infrastructure
> 
> **Cloud** : pour les organisations privilégiant la flexibilité, la simplicité de gestion et l'accessibilité

> [!warning] Points d'attention Cloud
> - Dépendance au fournisseur (vendor lock-in)
> - Nécessité d'une connexion Internet stable
> - Questions de souveraineté des données
> - Conformité RGPD à vérifier

---

## Terminologie

> [!note] Variations linguistiques
> Selon la région et la langue, différents termes sont utilisés pour désigner le courrier électronique.

### Termes en français

**En France et pays francophones** :
- **email** ou **e-mail** (le plus courant)
- **mail** (abus de langage mais très répandu)
- **courriel** (origine québécoise, recommandation officielle)
- **courrier électronique** (terme formel)
- **message électronique** (terme générique)

**Dans les pays anglo-saxons** :
- **email** (terme standard)
- **mail** désigne uniquement le courrier postal physique

> [!important] Convention du cours
> Dans la suite de ce cours, le courrier électronique sera désigné par le terme **"email"**.

---

# Clients et serveurs

> [!abstract] Infrastructure de messagerie
> Dans une infrastructure réseau d'entreprise, le service de messagerie électronique repose sur une **architecture clients/serveurs**.

## Glossaire technique messagerie

> [!note] Définitions essentielles pour le TSSR
> Ces acronymes et termes sont fondamentaux pour comprendre l'architecture de messagerie.

| Terme | Signification | Description |
|-------|---------------|-------------|
| **BAL** ou **BALE** | Boîte Aux Lettres (Électronique) | Espace de stockage des emails d'un utilisateur |
| **SMTP** | Simple Mail Transfer Protocol | Protocole d'envoi des emails |
| **POP** | Post Office Protocol | Protocole de réception des emails (téléchargement) |
| **IMAP** | Internet Message Access Protocol | Protocole de réception des emails (synchronisation) |
| **MUA** | Mail User Agent | Client de messagerie (logiciel client) |
| **MSA** | Mail Submission Agent | Composant qui accepte les emails du MUA et les transmet au MTA |
| **MTA** | Mail Transfer Agent | Élément principal d'un serveur SMTP qui transmet les emails d'un serveur à un autre |
| **MDA** | Mail Delivery Agent | Service de remise des emails dans les BAL |

> [!important] À retenir
> - **MUA** = ce que l'utilisateur utilise (Outlook, Thunderbird)
> - **MTA** = le cœur du routage des emails entre serveurs
> - **MDA** = la livraison finale dans la boîte aux lettres

---

## Fonctionnement de l'infrastructure

> [!info] Architecture client/serveur
> L'utilisateur n'est pas directement en contact avec le serveur de messagerie, mais passe par le **client** pour la gestion de sa messagerie.

**Rôle du serveur** :
- Contient un **logiciel** qui gère les messages
- Gère les **boîtes aux lettres** (stockage)
- Achemine les emails (routage)

**Rôle du client** :
- **Interface utilisateur** pour la consultation et l'envoi
- Communication avec le serveur via protocoles standardisés

---

## Le client de messagerie

> [!quote] Définition
> Le client de messagerie est le **logiciel** qui sert d'interface à l'utilisateur pour lui permettre l'envoi et la réception d'emails.

### Les 2 catégories de clients

#### 1. Client lourd (Client mail)

> [!info] Client lourd
> Logiciel installé localement sur un appareil (ordinateur, smartphone, tablette).

**Caractéristiques** :
- Installation nécessaire sur l'appareil
- Peut dépendre de l'OS hôte installé
- Fonctionnement possible hors ligne (après synchronisation)
- Stockage local des emails

**Exemples** :
- **Microsoft Outlook** (Windows, macOS)
- **Mozilla Thunderbird** (multiplateforme, open source)
- **Apple Mail** (macOS, iOS)
- **IBM Lotus Notes** (entreprise)

#### 2. Client web (Webmail)

> [!info] Webmail
> Interface accessible à partir d'un **navigateur web**, sans installation.

**Caractéristiques** :
- Aucune installation nécessaire
- Accessible depuis n'importe quel navigateur
- Nécessite une connexion Internet permanente
- Stockage sur le serveur

**Exemples** :
- **Gmail** (Google)
- **Outlook.com / Office 365** (Microsoft)
- **Yahoo! Mail**
- **La Poste.net**

> [!tip] Bonne pratique
> Un webmail peut être mis en place **en complément** d'un client lourd pour offrir plus de flexibilité aux utilisateurs.

---

## Le serveur de messagerie

> [!important] Types de serveurs
> Par **abus de langage**, on distingue 2 types de serveurs selon les protocoles utilisés.

### Classification des serveurs

| Type | Protocole | Rôle |
|------|-----------|------|
| **Serveur SMTP** | SMTP | Serveur **sortant** (envoi) |
| **Serveur POP/IMAP** | POP3 ou IMAP | Serveur **entrant** (réception) |

> [!note] Précision technique
> En réalité, il s'agit souvent du **même serveur physique** qui héberge plusieurs services (SMTP, POP, IMAP) avec des processus distincts.

**Les principaux protocoles de messagerie** :
- **SMTP** : Simple Mail Transfer Protocol
- **POP** : Post Office Protocol
- **IMAP** : Internet Message Access Protocol

---

## Exemple pratique : Bob et Alice

> [!example] Scénario
> Prenons 2 utilisateurs pour illustrer le fonctionnement complet de la messagerie :
> - **Bob** : adresse email dans le domaine `domaine1.fr` → `bob@domaine1.fr`
> - **Alice** : adresse email dans le domaine `domaine2.fr` → `alice@domaine2.fr`
> 
> **Question** : Que se passe-t-il au niveau des clients et des serveurs lorsque Bob envoie un mail à Alice ?

### Étape 1 : Du côté de l'expéditeur (Bob)

1. **Bob écrit un mail** sur son logiciel de messagerie et clique sur "Envoyer"
2. Le logiciel contacte le **serveur SMTP** du domaine de Bob → `smtp.domaine1.fr`
3. Le SMTP **lit l'adresse de destination** et en extrait le domaine :
   - Si destination = `domaine1.fr` → le mail est traité localement sur ce SMTP
   - Si destination = autre domaine → contact du SMTP de l'autre domaine
4. Dans notre exemple, la destination est `alice@domaine2.fr` :
   - Le SMTP cherche à contacter `smtp.domaine2.fr`
   - Si le SMTP `domaine2.fr` **existe** → transfert du mail
   - Sinon → message d'erreur à l'expéditeur

### Étape 2 : Du côté du serveur destinataire

1. Le **SMTP de domaine2.fr** reçoit le mail
2. Il **vérifie** dans sa liste d'utilisateurs que `alice` existe :
   - S'il n'existe **pas** → message d'erreur au SMTP d'origine
   - S'il **existe** → le mail est placé dans l'**espace mémoire** dédié à Alice sur le serveur

> [!success] Objectif atteint
> Le mail est ainsi **arrivé à destination**. L'objectif du protocole SMTP est atteint.

### Étape 3 : Du côté du client destinataire (Alice)

1. **Alice veut voir** si elle a reçu des mails
2. Elle **ouvre son logiciel** de messagerie
3. Le logiciel utilise **POP ou IMAP** pour vérifier si des mails sont en attente dans l'espace mémoire dédié sur le serveur
4. S'il y en a, le serveur **envoie les mails** vers le logiciel de messagerie d'Alice
5. **Alice reçoit le mail de Bob** dans sa boîte aux lettres

---

## Serveur MX

> [!quote] Définition
> Un serveur **MX (Mail Exchange)** est un serveur de messagerie qui **reçoit et achemine** les mails.

### Enregistrement MX (MX record)

> [!info] MX record
> Un **enregistrement MX** correspond aux adresses des serveurs sur lesquels sont envoyés les mails de destination.

**Fonctionnement** :
1. Lors de l'envoi d'un email, le serveur SMTP **interroge le DNS**
2. Il obtient la **liste des enregistrements MX** du domaine destinataire
3. Ensuite, une **requête A** et/ou **AAAA** est faite pour récupérer les adresses IP

**Gestion de la priorité** :
- Il peut y avoir **plusieurs MX** par nom de domaine
- Chaque MX possède une **priorité** (plus le chiffre est bas, plus la priorité est élevée)

> [!example] Exemple de MX record
> ```
> MX 10 mx1.nom-domaine.fr
> MX 20 mx2.nom-domaine.fr
> MX 30 mx3.nom-domaine.fr
> ```
> 
> Le serveur SMTP tentera d'abord `mx1.nom-domaine.fr` (priorité 10). En cas d'échec, il utilisera `mx2.nom-domaine.fr` (priorité 20), et ainsi de suite.

> [!tip] Redondance
> Avoir plusieurs enregistrements MX assure la **haute disponibilité** du service de messagerie.

---

## Fonctionnement détaillé

> [!abstract] Processus complet
> Voici le schéma détaillé du cheminement d'un email entre Bob et Alice.

### Schéma du flux d'emails

```
Domaine1.fr                           Domaine2.fr
┌─────────────┐                       ┌─────────────┐
│             │                       │             │
│  Bob        │                       │  Alice      │
│  MUA        │                       │  MUA        │
│             │                       │             │
└──────┬──────┘                       └──────▲──────┘
       │ SMTP                                │ POP/IMAP
       ▼                                     │
┌──────────────┐                      ┌──────────────┐
│ MSA          │                      │              │
│ (vérif.      │                      │              │
│ contenu)     │                      │              │
└──────┬───────┘                      └──────┬───────┘
       │ SMTP                                │
       ▼                                     │
┌──────────────┐   Recherche MX      ┌──────▼───────┐
│ MTA          ├───────────────────►  │ Serveur DNS  │
│ (vérif. mail)│                      └──────────────┘
└──────┬───────┘                             
       │ SMTP                                
       │                                     
       ▼                                     
┌──────────────┐                             
│ smtp.        │                             
│ domaine1.fr  │                             
│ (MTA)        │                             
└──────┬───────┘                             
       │ SMTP (Internet)
       │                                     
       ▼                                     
┌──────────────┐                             
│ smtp.        │                             
│ domaine2.fr  │                             
│ (MTA)        │                             
└──────┬───────┘                             
       │ SMTP                                
       ▼                                     
┌──────────────┐                             
│ MDA          │                             
│ (vérif. BAL) │                             
└──────┬───────┘                             
       │                                     
       ▼                                     
┌──────────────┐                             
│ BDD de BAL   │                             
│              │                             
└──────────────┘                             
```

### Détail des étapes

1. **Bob (MUA)** rédige et envoie l'email
2. **MSA** (Mail Submission Agent) reçoit l'email et vérifie son contenu
3. **MTA** (Mail Transfer Agent) vérifie le mail et interroge le DNS pour trouver le MX de `domaine2.fr`
4. Le **Serveur DNS** retourne les enregistrements MX
5. **MTA de domaine1.fr** envoie l'email via SMTP au **MTA de domaine2.fr**
6. Le **MTA de domaine2.fr** reçoit l'email
7. Le **MDA** (Mail Delivery Agent) vérifie que la BAL d'Alice existe
8. L'email est stocké dans la **base de données de BAL**
9. **Alice (MUA)** se connecte via POP ou IMAP pour récupérer ses emails

> [!important] Points clés
> - Les **MTA** assurent le **routage** des emails entre serveurs
> - Le **MDA** assure la **livraison finale** dans la boîte aux lettres
> - Le **MSA** est le point d'entrée depuis le client de l'utilisateur

---

# Les protocoles

> [!abstract] Protocoles de messagerie
> Les protocoles de messagerie sont des ensembles de règles standardisées qui permettent l'envoi, le routage et la réception des emails.

## SMTP

> [!quote] Définition
> Le protocole **SMTP (Simple Mail Transfer Protocol)** est utilisé pour **envoyer des emails** sur un réseau informatique.

### Caractéristiques techniques

| Caractéristique | Valeur |
|-----------------|--------|
| **RFC** | RFC 821 et RFC 5321 |
| **Port par défaut** | 25 |
| **Port avec chiffrement** | 465 (SMTPS) ou 587 (SMTP avec STARTTLS) |
| **Direction** | Envoi (sortant) |
| **Type** | Protocole de transfert |

> [!info] Rôle du SMTP
> Son rôle est de **router les mails** à partir de l'adresse du destinataire vers le domaine de destination.

### Fonctionnement

```
Client SMTP → Serveur SMTP local → Serveur SMTP distant → Boîte aux lettres
```

> [!note] Abus de langage
> Par abus de langage, on parle de **"serveur SMTP"** pour désigner un serveur de messagerie configuré pour envoyer des emails.

> [!warning] Sécurité
> Le port 25 transmet les données **en clair** (non chiffré). Pour sécuriser les communications :
> - Utiliser le **port 465** (SMTPS - chiffrement SSL/TLS direct)
> - Utiliser le **port 587** (SMTP + STARTTLS - chiffrement opportuniste)

---

## POP3

> [!quote] Définition
> Le protocole **POP (Post Office Protocol)** est actuellement en version 3, d'où le nom **POP3** souvent utilisé.

### Caractéristiques techniques

| Caractéristique | Valeur |
|-----------------|--------|
| **Version actuelle** | POP3 |
| **Port par défaut** | 110 |
| **Port sécurisé** | 995 (POP3S) |
| **Direction** | Réception (entrant) |
| **Type** | Protocole de téléchargement |

> [!info] Rôle de POP3
> Son rôle est de permettre à un utilisateur de **relever son courrier** sur un serveur POP. POP établit un dialogue entre l'hôte qui ne contient pas la BAL mais sur lequel on trouve le MUA, et la BAL sur le serveur.

### Fonctionnalités de base

POP est un **protocole simple** qui donne accès à des fonctionnalités de base :

- ✉️ **Comptage** des emails disponibles
- 📊 **Calcul de volumes** (taille totale des emails)
- 🗑️ **Suppression** d'emails
- ⬇️ **Téléchargement** des emails de la BAL vers le client local

### Schéma de fonctionnement POP3

```
┌─────────────────────────────────────────────────┐
│     Serveur de messagerie                       │
│  ┌─────────────────────────┐                    │
│  │  Nouveaux mails         │                    │
│  └─────────────────────────┘                    │
└────────────┬────────────────────────────────────┘
             │
             │ POP3 - Téléchargement
             ▼
┌────────────────────────┐
│  Client 1              │
│  ┌──────────────────┐  │
│  │ Stockage local   │  │
│  │ des mails        │  │
│  └──────────────────┘  │
└────────────────────────┘

┌────────────────────────┐
│  Client 2              │
│  ┌──────────────────┐  │
│  │ Pas de nouveaux  │  │
│  │ messages         │  │
│  └──────────────────┘  │
└────────────────────────┘
```

> [!warning] Problématique POP3
> **Limitation importante** : Une fois les mails téléchargés sur Client 1, ils sont **supprimés du serveur** (comportement par défaut). Le Client 2 ne verra donc **pas** ces nouveaux messages.

> [!tip] Option de configuration
> La plupart des clients de messagerie proposent l'option : **"conserver une copie des messages sur le serveur"**. Cette option permet de contourner la limitation, mais n'offre pas de synchronisation réelle entre les clients.

---

## IMAP

> [!quote] Définition
> Le protocole **IMAP (Internet Message Access Protocol)** est un protocole de récupération de mails **avancé** et constitue une alternative à POP par l'ensemble des services évolués qu'il propose.

### Caractéristiques techniques

| Caractéristique | Valeur |
|-----------------|--------|
| **Version actuelle** | IMAP4 |
| **Port par défaut** | 143 |
| **Port sécurisé** | 993 (IMAPS) |
| **Direction** | Réception (entrant) |
| **Type** | Protocole de synchronisation |

> [!important] Innovation majeure
> Une des principales nouveautés est la possibilité de pouvoir lire **uniquement les en-têtes** des messages (sans le corps). Ainsi, on peut par exemple **effacer des messages sans les avoir lus**.

### Services avancés IMAP

IMAP offre des fonctionnalités bien plus évoluées que POP :

| Fonctionnalité | Description |
|----------------|-------------|
| **Gestion des en-têtes** | Lecture uniquement des objets des messages (sans le corps) → effacement/déplacement des mails sans les avoir lus |
| **Création de dossiers** | Création de dossiers directement sur le serveur |
| **Lecture non destructive** | Lecture des mails **en les laissant sur le serveur** |
| **Marquage** | Marquage des mails (lu/non lu, important, etc.) |
| **Synchronisation** | Synchronisation bidirectionnelle entre les mails locaux et sur le serveur |
| **Multi-appareils** | Accès cohérent depuis plusieurs clients simultanément |

### Schéma de fonctionnement IMAP

```
┌─────────────────────────────────────────────────┐
│     Serveur de messagerie                       │
│  ┌─────────────────────────┐                    │
│  │  Tous les mails         │                    │
│  │  (source de vérité)     │                    │
│  └─────────────────────────┘                    │
└────────┬────────────────┬───────────────────────┘
         │                │
         │                │
    Sync │                │ Sync
         ▼                ▼
┌────────────────┐   ┌────────────────┐
│  Client 1      │   │  Client 2      │
│  ┌──────────┐  │   │  ┌──────────┐  │
│  │ Copie    │  │   │  │ Copie    │  │
│  │ locale   │  │   │  │ locale   │  │
│  └──────────┘  │   │  └──────────┘  │
└────────────────┘   └────────────────┘
```

> [!success] Avantage IMAP
> Client 1 et Client 2 sont **synchronisés** avec le serveur et sont dans le **même état**. Toute modification (lecture, suppression, déplacement) sur un client est **répercutée** sur tous les autres.

### Exemple de synchronisation

**Scénario** :
1. Un email arrive sur le serveur
2. Alice consulte l'email depuis son smartphone (Client 1)
3. Alice marque l'email comme "lu"
4. La synchronisation IMAP met à jour le serveur
5. Alice ouvre ensuite son ordinateur portable (Client 2)
6. L'email apparaît déjà comme "lu" sur Client 2

> [!note] Mode asynchrone
> IMAP est **asynchrone**, il permet donc d'être utilisé avec une connexion à **faible débit**. Les synchronisations se font progressivement sans bloquer l'utilisateur.

---

## Comparaison POP vs IMAP

> [!important] Choix du protocole
> Le choix entre POP et IMAP dépend des besoins de l'organisation et des utilisateurs.

### Tableau comparatif détaillé

| Critère | POP3 | IMAP |
|---------|------|------|
| **Type** | Téléchargement | Synchronisation |
| **Stockage des emails** | Local (sur le client) | Serveur (copie locale possible) |
| **Multi-appareils** | ❌ Problématique | ✅ Optimal |
| **Accès hors ligne** | ✅ Total (après téléchargement) | ⚠️ Partiel (emails synchronisés) |
| **Espace serveur** | 💾 Libéré rapidement | 💾 Utilisé en continu |
| **Bande passante** | 📶 Téléchargement complet | 📶 Optimisé (en-têtes d'abord) |
| **Gestion des dossiers** | ❌ Locale uniquement | ✅ Serveur + locale |
| **Sauvegarde** | ⚠️ Dépend du client | ✅ Centralisée sur serveur |
| **Complexité** | 🟢 Simple | 🟡 Avancé |

### Cas d'usage recommandés

#### Utiliser POP3 quand :

> [!tip] POP3 recommandé pour
> - Organisation **privée ou professionnelle** sans besoins avancés
> - Utilisateurs **mono-poste** (un seul appareil)
> - Besoin d'**accès hors ligne** permanent
> - **Espace serveur limité** (libération rapide)
> - Connexion Internet **peu fiable**

#### Utiliser IMAP quand :

> [!tip] IMAP recommandé pour
> - Utilisation **professionnelle** avec gestion fine des BAL
> - Solution **multi-postes** (ordinateur, smartphone, tablette)
> - Besoin de **synchronisation** entre appareils
> - Importance de la **sauvegarde centralisée**
> - Connexion **à faible débit** (mode asynchrone)
> - **Collaboration** (boîtes aux lettres partagées)

> [!success] Recommandation TSSR
> Pour un environnement professionnel moderne, **IMAP** est généralement le choix privilégié car il offre :
> - Flexibilité d'accès multi-appareils
> - Sauvegarde centralisée
> - Meilleure expérience utilisateur
> - Gestion avancée des emails

---

# Bonnes pratiques

> [!abstract] Utilisation professionnelle de la messagerie
> En tant que TSSR, tu dois non seulement administrer les systèmes de messagerie, mais aussi promouvoir les bonnes pratiques auprès des utilisateurs.

## Gestion des emails

> [!important] Hygiène de la boîte aux lettres
> Une bonne gestion des emails améliore les performances du système et facilite le travail quotidien.

### Règles de gestion

- 🗑️ **Ne pas faire du stockage "longue durée"** des emails
  - Les emails ne sont pas conçus pour l'archivage long terme
  - Utiliser des solutions d'archivage dédiées si nécessaire

- 🧹 **Faire régulièrement le tri** des emails
  - Supprimer les emails qui ne sont plus nécessaires
  - Archiver les emails importants dans un système dédié
  - Vider régulièrement la corbeille

- 📁 **Utiliser des dossiers** pour classer les emails par catégorie
  - Exemples : "Travail", "Personnel", "Projets", "Archivage"
  - Créer une arborescence logique
  - Utiliser les fonctionnalités de filtrage automatique

- 🔒 **Ne pas garder d'emails sensibles** ou confidentiels dans la BAL
  - Risque en cas de compromission du compte
  - Transférer vers un stockage sécurisé si nécessaire
  - Chiffrer les emails contenant des données sensibles

- 💾 **Sauvegarder régulièrement** les emails importants
  - Éviter toute perte de données en cas de panne
  - Protéger contre les suppressions accidentelles
  - Exporter les emails critiques hors de la messagerie

> [!tip] Quota de boîte aux lettres
> La plupart des systèmes de messagerie ont des **quotas de stockage**. Une gestion rigoureuse évite de saturer l'espace et de bloquer la réception de nouveaux emails.

---

## Sécurité

> [!warning] La sécurité est critique
> La messagerie est un vecteur d'attaque privilégié (phishing, malware, ransomware). La sécurité doit être une priorité absolue.

### Mesures de sécurité essentielles

#### Authentification

- 🔐 **Utiliser des mots de passe forts et uniques**
  - Minimum 12 caractères
  - Combinaison de majuscules, minuscules, chiffres et caractères spéciaux
  - Ne jamais réutiliser un mot de passe

- 🔑 **Activer l'authentification à deux facteurs (2FA)**
  - Ajoute une couche de sécurité supplémentaire
  - Même si le mot de passe est compromis, le compte reste protégé
  - Utiliser une application d'authentification (Google Authenticator, Microsoft Authenticator)

#### Vigilance sur le contenu

> [!danger] Pièces jointes suspectes
> - **Ne jamais ouvrir** de pièces jointes provenant d'expéditeurs inconnus
> - Vérifier l'extension du fichier (éviter .exe, .bat, .scr, .js)
> - Scanner systématiquement avec un antivirus
> - En cas de doute, contacter l'expéditeur par un autre canal

> [!danger] Liens malveillants
> - **Survoler** les liens sans cliquer pour voir l'URL réelle
> - Vérifier la cohérence entre le texte du lien et l'URL
> - Ne jamais cliquer sur des liens dans des emails non sollicités
> - Attention aux techniques de phishing (URLs similaires à des sites légitimes)

#### Spam et emails non sollicités

- ⚠️ **Faire attention aux emails non sollicités (spam)**
  - Utiliser les filtres anti-spam
  - Marquer comme spam plutôt que simplement supprimer
  - Ne jamais répondre à un spam

- 👤 **Messages provenant d'expéditeurs inconnus**
  - Vérifier l'adresse email complète de l'expéditeur
  - Attention aux techniques d'usurpation d'identité (spoofing)
  - En cas de doute, ne pas interagir avec l'email

> [!example] Exemple de phishing
> Email reçu : "Votre compte Netflix a été suspendu. Cliquez ici pour réactiver votre compte."
> 
> **Signes suspects** :
> - Adresse expéditeur : `netflix-support@secure-netflx.com` (faute d'orthographe)
> - Urgence artificielle
> - Lien vers un site qui n'est pas netflix.com
> - Demande d'informations de paiement

---

## Confidentialité

> [!info] Protection des données
> La confidentialité des communications professionnelles est une obligation légale et éthique.

### Règles de confidentialité

- 💼 **Utiliser une adresse électronique professionnelle** pour les communications d'entreprise
  - Séparer vie professionnelle et vie privée
  - Respecter la politique de l'entreprise
  - Faciliter la gestion et l'archivage

- 🔒 **Éviter d'envoyer des informations sensibles** par courrier électronique
  - Mots de passe, numéros de carte bancaire, données personnelles
  - Utiliser des canaux sécurisés dédiés (coffre-fort numérique, messagerie chiffrée)
  - Chiffrer les emails contenant des données sensibles (S/MIME, PGP)

- ✅ **Vérifier la liste des destinataires** avant d'envoyer un email
  - Attention aux champs "Cc" et "Cci" (Bcc)
  - Éviter les fuites d'informations involontaires
  - Utiliser "Cci" pour les envois groupés (protection de la vie privée)

> [!warning] RGPD
> En Europe, le **Règlement Général sur la Protection des Données (RGPD)** impose des obligations strictes :
> - Minimisation des données
> - Consentement des personnes
> - Droit à l'effacement
> - Notification des violations de données

> [!tip] Bonne pratique Cc vs Cci
> - **Cc (Copie Carbone)** : tous les destinataires voient qui a reçu l'email
> - **Cci (Copie Carbone Invisible / Bcc)** : chaque destinataire ne voit que sa propre adresse
> 
> Utiliser **Cci** pour les envois groupés à des personnes qui ne se connaissent pas (respect de la vie privée).

---

## Utilisation responsable

> [!important] Professionnalisme
> L'utilisation de la messagerie en entreprise engage la responsabilité de l'utilisateur et de l'organisation.

### Principes d'utilisation responsable

- 📜 **Respecter la politique de courrier électronique** de l'entreprise
  - Lire et comprendre la charte informatique
  - Respecter les règles d'utilisation
  - Connaître ses droits et obligations

- 🤝 **Éviter d'utiliser des termes inappropriés ou offensants** dans les courriels professionnels
  - Langage professionnel et courtois
  - Éviter l'humour qui pourrait être mal interprété
  - Pas d'insultes, de discrimination, de harcèlement

- ⚡ **Utiliser le courrier électronique de manière efficace et efficiente**
  - Objets clairs et concis
  - Messages structurés et concis
  - Répondre dans des délais raisonnables
  - Éviter les échanges interminables (privilégier le téléphone/réunion si nécessaire)

> [!tip] Rédaction d'un email professionnel
> **Structure recommandée** :
> 1. **Objet** : clair et informatif
> 2. **Salutation** : formule de politesse adaptée
> 3. **Corps** : message structuré, paragraphes courts
> 4. **Signature** : nom, fonction, coordonnées
> 
> **Exemple** :
> ```
> Objet : Demande de validation - Budget Q1 2026
> 
> Bonjour Marie,
> 
> Je te transmets en pièce jointe le projet de budget pour le
> premier trimestre 2026.
> 
> Pourrais-tu le valider avant vendredi 14 février ?
> 
> Merci d'avance,
> 
> Cordialement,
> Jean Dupont
> Technicien Systèmes et Réseaux
> Service Informatique
> jean.dupont@entreprise.fr | 01 23 45 67 89
> ```

> [!warning] Attention
> - Les emails professionnels peuvent être lus par l'employeur (selon la législation)
> - Les emails engagent l'entreprise
> - Tout écrit peut être utilisé comme preuve légale

---

# Points clés à retenir

> [!success] Synthèse pour le titre RNCP
> Voici les éléments essentiels à maîtriser pour ton examen TSSR.

## Introduction et concepts

- 📧 La **messagerie électronique** est un système de communication **asynchrone** en mode texte
- ⏰ **Historique** : création en 1965, arobase "@" en 1971 par Ray Tomlinson
- 📮 **Analogie postale** : le SMTP joue le rôle de La Poste pour acheminer les emails
- ✉️ Une **adresse email** se compose de : `partie_locale@nom_de_domaine`
- 📄 Un **email** contient : en-tête (métadonnées) + corps (message)
- 📬 Une **BAL** (Boîte Aux Lettres) est l'espace de stockage des emails
- 🏢 En entreprise, la messagerie est un **système critique** nécessitant une surveillance continue
- ⚡ Différence avec la **messagerie instantanée** : synchrone vs asynchrone

## Architecture et infrastructure

- 🏗️ **Architecture client/serveur** : l'utilisateur passe par un client pour accéder au serveur
- 💻 **Client lourd** (Outlook, Thunderbird) vs **Webmail** (Gmail, interface web)
- 🌐 **On-premises** (serveurs locaux) vs **Cloud** (serveurs distants)
- 🔄 **Enregistrement MX** : indique les serveurs de messagerie pour un domaine (avec priorité)

## Agents de messagerie (à connaître absolument)

| Agent | Rôle |
|-------|------|
| **MUA** (Mail User Agent) | Client de messagerie (interface utilisateur) |
| **MSA** (Mail Submission Agent) | Accepte les emails du MUA et les transmet au MTA |
| **MTA** (Mail Transfer Agent) | Transmet les emails d'un serveur à un autre (routage) |
| **MDA** (Mail Delivery Agent) | Remet les emails dans les boîtes aux lettres |

## Protocoles (CRUCIAL)

### SMTP (Simple Mail Transfer Protocol)

- 📤 **Rôle** : envoi/routage des emails
- 🔌 **Ports** : 25 (non sécurisé), 465 (SMTPS), 587 (SMTP+STARTTLS)
- 📋 **RFC** : RFC 821 et RFC 5321
- ⚠️ Protocole **sortant** (expéditeur → serveur → destinataire)

### POP3 (Post Office Protocol v3)

- 📥 **Rôle** : téléchargement des emails
- 🔌 **Ports** : 110 (non sécurisé), 995 (POP3S)
- 📋 **RFC** : RFC 1939
- ⚠️ **Limitation** : téléchargement + suppression du serveur (par défaut)
- 💡 **Usage** : mono-poste, accès hors ligne

### IMAP (Internet Message Access Protocol)

- 🔄 **Rôle** : synchronisation des emails
- 🔌 **Ports** : 143 (non sécurisé), 993 (IMAPS)
- 📋 **RFC** : RFC 3501
- ✅ **Avantages** : multi-appareils, gestion de dossiers serveur, synchronisation bidirectionnelle
- 💡 **Usage** : professionnel, multi-postes, accès depuis plusieurs appareils

### POP vs IMAP - Comparaison

| Critère | POP3 | IMAP |
|---------|------|------|
| **Mode** | Téléchargement | Synchronisation |
| **Stockage** | Local | Serveur |
| **Multi-appareils** | ❌ | ✅ |
| **Dossiers serveur** | ❌ | ✅ |
| **Sauvegarde** | Cliente | Serveur |

> [!important] Choix du protocole
> - **POP3** → utilisateur mono-poste, accès hors ligne prioritaire
> - **IMAP** → environnement professionnel, multi-appareils, synchronisation

## Processus d'envoi d'email (à savoir expliquer)

1. **Bob** rédige un email à `alice@domaine2.fr`
2. Le **MUA** de Bob se connecte au **serveur SMTP** de `domaine1.fr`
3. Le **SMTP** interroge le **DNS** pour obtenir les **enregistrements MX** de `domaine2.fr`
4. Le **MTA** de `domaine1.fr` transfère l'email au **MTA** de `domaine2.fr` via SMTP
5. Le **MDA** de `domaine2.fr` vérifie que la BAL d'Alice existe et y dépose l'email
6. **Alice** utilise **POP** ou **IMAP** pour récupérer l'email depuis le serveur

## Bonnes pratiques (pour l'examen et la vie pro)

### Gestion

- 🧹 Ne pas faire de stockage long terme dans la BAL
- 📁 Classer avec des dossiers
- 🗑️ Supprimer régulièrement les emails inutiles
- 💾 Sauvegarder les emails importants

### Sécurité

- 🔐 Mots de passe forts + authentification 2FA
- ⚠️ Vigilance sur les pièces jointes et liens
- 🚫 Attention au phishing et au spam
- 🔍 Vérifier l'expéditeur réel

### Confidentialité

- 💼 Utiliser une adresse professionnelle pour le travail
- 🔒 Ne pas envoyer d'informations sensibles par email
- ✅ Vérifier les destinataires (Cc vs Cci)

### Utilisation responsable

- 📜 Respecter la charte de l'entreprise
- 🤝 Langage professionnel et courtois
- ⚡ Communication efficace et efficiente

---

# Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **BAL / BALE** | Boîte Aux Lettres (Électronique) - Espace de stockage des emails d'un utilisateur sur le serveur |
| **SMTP** | Simple Mail Transfer Protocol - Protocole d'envoi/routage des emails (port 25, 465, 587) |
| **POP3** | Post Office Protocol version 3 - Protocole de téléchargement des emails (port 110, 995) |
| **IMAP** | Internet Message Access Protocol - Protocole de synchronisation des emails (port 143, 993) |
| **MUA** | Mail User Agent - Client de messagerie utilisé par l'utilisateur final (Outlook, Thunderbird, etc.) |
| **MSA** | Mail Submission Agent - Composant qui accepte les emails du MUA et les transmet au MTA |
| **MTA** | Mail Transfer Agent - Agent qui transmet les emails entre serveurs (routage) |
| **MDA** | Mail Delivery Agent - Agent qui délivre les emails dans les boîtes aux lettres |
| **MX Record** | Mail Exchange Record - Enregistrement DNS indiquant les serveurs de messagerie d'un domaine |
| **RFC** | Request For Comments - Documents décrivant les standards Internet |
| **Email** | Courrier électronique, message électronique, mail, courriel |
| **Webmail** | Interface web d'accès à la messagerie (Gmail, Outlook.com) |
| **Client lourd** | Logiciel de messagerie installé localement (Outlook, Thunderbird) |
| **Asynchrone** | Mode de communication où l'émission et la réception ne sont pas simultanées |
| **Synchrone** | Mode de communication en temps réel (messagerie instantanée) |
| **En-tête** | Partie d'un email contenant les métadonnées (expéditeur, destinataire, objet, date) |
| **Corps** | Partie d'un email contenant le message en lui-même |
| **On-premises** | Infrastructure hébergée localement par l'entreprise |
| **Cloud** | Infrastructure hébergée à distance par un fournisseur de services |
| **Phishing** | Technique d'hameçonnage visant à obtenir des informations sensibles |
| **Spam** | Courrier électronique non sollicité (courrier indésirable) |
| **2FA** | Two-Factor Authentication - Authentification à deux facteurs |
| **SSL/TLS** | Protocoles de chiffrement des communications |
| **SMTPS** | SMTP Sécurisé (port 465) |
| **POP3S** | POP3 Sécurisé (port 995) |
| **IMAPS** | IMAP Sécurisé (port 993) |
| **STARTTLS** | Commande permettant de passer d'une connexion non chiffrée à chiffrée |
| **Cc** | Copie Carbone - Destinataires en copie (visible par tous) |
| **Cci / Bcc** | Copie Carbone Invisible / Blind Carbon Copy - Destinataires en copie cachée |
| **RGPD** | Règlement Général sur la Protection des Données |
| **DNS** | Domain Name System - Système de résolution de noms de domaine |
| **Arobase (@)** | Symbole "@" séparant la partie locale du nom de domaine dans une adresse email |

---

> [!success] Document de révision complet
> Ce document couvre l'ensemble du cours sur la messagerie électronique. Il est conçu pour être importé dans Obsidian et utilisé comme support de révision pour le titre RNCP TSSR.
> 
> **Prochaines étapes recommandées** :
> - Réviser les protocoles (SMTP, POP3, IMAP)
> - S'entraîner à expliquer le processus complet d'envoi d'email
> - Maîtriser les agents de messagerie (MUA, MSA, MTA, MDA)
> - Connaître les ports et RFC associés
> - Comprendre les différences POP vs IMAP et savoir argumenter le choix

**Bonne révision ! 🎓**