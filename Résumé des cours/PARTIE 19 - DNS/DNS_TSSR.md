# Domain Name System (DNS)

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Domain Name System - Résolution de noms sur Internet

**Date** : Janvier 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Le problème - Humain vs ordinateur|Le problème - Humain vs ordinateur]]
   - [[#Un système de correspondances|Un système de correspondances]]
   - [[#HOSTS.TXT - L'ancêtre du DNS|HOSTS.TXT - L'ancêtre du DNS]]
   - [[#DNS - Le système de noms de domaine|DNS - Le système de noms de domaine]]
   - [[#Idée générale|Idée générale]]

2. [[#Les noms de domaine|Les noms de domaine]]
   - [[#Une définition|Une définition]]
   - [[#Arborescence des noms de domaine|Arborescence des noms de domaine]]
   - [[#Les composants d'un nom de domaine|Les composants d'un nom de domaine]]
   - [[#Des exemples de noms de domaine|Des exemples de noms de domaine]]

3. [[#Le protocole DNS|Le protocole DNS]]
   - [[#La communication DNS|La communication DNS]]
   - [[#La résolution de nom|La résolution de nom]]
   - [[#Les serveurs faisant autorité|Les serveurs faisant autorité]]
   - [[#Les serveurs racines|Les serveurs racines]]
   - [[#Les résolveurs|Les résolveurs]]
   - [[#Les résolveurs publiques|Les résolveurs publiques]]
   - [[#Stub resolver (DNS local)|Stub resolver (DNS local)]]
   - [[#Configuration DNS d'un hôte|Configuration DNS d'un hôte]]

4. [[#Les enregistrements|Les enregistrements]]
   - [[#Les enregistrements DNS (Resource Records)|Les enregistrements DNS (Resource Records)]]
   - [[#La résolution inverse|La résolution inverse]]
   - [[#DNS round-robin|DNS round-robin]]

5. [[#Enregistrer un nom de domaine|Enregistrer un nom de domaine]]
   - [[#Racine et TLD|Racine et TLD]]
   - [[#Registre de noms de domaine|Registre de noms de domaine]]
   - [[#Bureau d'enregistrement|Bureau d'enregistrement]]

6. [[#Outils|Outils]]
   - [[#dig - Domain Information Groper|dig - Domain Information Groper]]

7. [[#Points clés à retenir|Points clés à retenir]]

8. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le **Domain Name System (DNS)** est un système fondamental d'Internet qui permet de convertir des noms de domaine faciles à mémoriser (comme `wildcodeschool.com`) en adresses IP utilisables par les machines. C'est une infrastructure invisible mais essentielle au fonctionnement quotidien d'Internet.

### Pourquoi étudier le DNS ?

En tant que **TSSR**, tu dois comprendre :
- Comment les noms sont résolus en adresses IP
- L'architecture distribuée et hiérarchique du DNS
- Le fonctionnement des serveurs DNS et résolveurs
- La configuration et le dépannage des problèmes de résolution de noms

---

### Le problème - Humain vs ordinateur

> [!info] Le problème fondamental
> Pour communiquer sur un réseau IP, il est nécessaire de connaître l'**adresse IP du destinataire** d'un message.

**Les contraintes techniques :**
- Les adresses IP sont des **séquences binaires** (32 bits pour IPv4, 128 bits pour IPv6)
- Elles disposent d'une notation standard (ex : `192.168.1.1` ou `2001:db8::1`)
- Cette notation les rend plus faciles à manipuler qu'en binaire pur

**Le problème humain :**
- Il reste **difficile** de manipuler ces adresses au quotidien
- Il est **très difficile** de s'en souvenir
- En revanche, il est **aisé de retenir des noms textuels** comme `google.com` ou `wikipedia.org`

> [!important] Besoin fondamental
> Les humains ont besoin de noms mémorisables, les ordinateurs ont besoin d'adresses IP. Le DNS fait le pont entre ces deux mondes.

---

### Un système de correspondances

> [!quote] Principe de base
> Pour pouvoir utiliser des noms à la place des adresses, il faut un **système de correspondances** (mapping).

**Caractéristiques de ce système :**
- Doit pouvoir faire correspondre des **noms d'hôtes** (hostname) à leur **adresse IP**
- Ce système est apparu très tôt dans l'histoire d'Internet
- Était historiquement géré **à la main** !

> [!note] Évolution historique
> Le besoin de nommer les machines sur un réseau est apparu dès les débuts d'Internet (ARPANET). La solution initiale était manuelle et centralisée.

---

### HOSTS.TXT - L'ancêtre du DNS

> [!info] Le premier système de nommage
> Dès les débuts du réseau qui deviendra Internet, un **fichier texte** contenant des adresses IP et les noms de machine correspondants est apparu.

**Historique du fichier HOSTS.TXT :**
- **Standardisé** par la **RFC 608** de 1974
- **Maintenu** par le **NIC** (Network Information Center)
- **Copié** sur chaque machine via transfert de fichier (**FTP**)

**Limitations :**
- L'augmentation rapide du nombre d'hôtes a rendu cette gestion **obsolète**
- Impossible de maintenir manuellement un fichier pour des milliers de machines
- Problèmes de cohérence et de synchronisation

> [!warning] Passage à l'échelle impossible
> Le système HOSTS.TXT ne pouvait pas supporter la croissance explosive d'Internet. Un système distribué et automatisé était nécessaire.

**Héritage moderne :**
Le fichier `/etc/hosts` sur les systèmes Unix/Linux est un vestige de cette époque. Il est toujours consulté en priorité avant les requêtes DNS.

---

### DNS - Le système de noms de domaine

> [!quote] Naissance du DNS
> Le système des noms de domaine (**Domain Name System - DNS**) a vu le jour au NIC pour permettre de répondre à cet enjeu majeur.

**Historique :**
- **Standardisé** par **Paul V. Mockapetris** dans les **RFC 882 et 883** en **1983**
- Aujourd'hui basé sur les **RFC 1034 et 1035**, toujours valables
- Complété par de nombreuses autres RFC au fil des années

> [!important] RFC fondamentales
> Les RFC 1034 et 1035 datant de 1987 sont toujours la base du DNS moderne. Elles définissent l'architecture et le protocole DNS.

---

### Idée générale

> [!abstract] Qu'est-ce que DNS ?
> DNS est un système complexe qui peut être défini de plusieurs façons complémentaires.

**DNS est :**

1. **Une base de données répartie et décentralisée**
   - Garantit la **robustesse** du système
   - Permet le **passage à l'échelle** (scaling) pour des milliards de noms

2. **Une technologie d'infrastructure**
   - Fonctionne de manière **invisible** pour l'utilisateur final
   - Essentielle mais transparente dans l'utilisation quotidienne d'Internet

3. **Un ensemble de noms de domaine auxquels sont associées des données**
   - Pas uniquement des adresses IP
   - Différents types d'enregistrements (Resource Records)

> [!success] Architecture distribuée
> La force du DNS réside dans sa nature distribuée : aucun serveur unique ne contient toutes les informations, ce qui garantit la résilience et les performances à l'échelle mondiale.

---

## Les noms de domaine

> [!abstract] Section : Les noms de domaine
> Cette section explique la structure, la syntaxe et les caractéristiques des noms de domaine utilisés sur Internet.

---

### Une définition

> [!quote] Définition : Nom de domaine
> Un **nom de domaine** est un identifiant textuel unique qui désigne un ensemble de ressources Internet.

**Caractéristiques d'un nom de domaine :**

| Caractéristique | Description |
|-----------------|-------------|
| **Identifiant unique** | Un nom de domaine identifie de manière unique une entité sur Internet |
| **Textuel** | Composé de caractères (lettres, chiffres, tirets) |
| **Non sensible à la casse** | `WildCodeSchool.com` = `wildcodeschool.com` = `WILDCODESCHOOL.COM` |
| **Structure hiérarchique** | Organisation en arborescence de type parent/enfant |
| **Composite** | Composé de plusieurs labels séparés par des points |

**Propriétés fonctionnelles :**
- Un **domaine peut contenir des sous-domaines** (relation parent-enfant)
- Chaque domaine **appartient à un domaine parent**, sauf la racine
- Désigne un **ensemble de ressources Internet** (serveurs, services, etc.)

**Avantages :**
- **Assimilable à l'identité** d'une personne/structure/ressource sur Internet
- **Plus stable que les adresses IP** (un site peut changer d'IP sans changer de nom)
- **Au choix de l'utilisateur** => vecteur d'image et de communication

> [!tip] Stabilité des noms
> Un nom de domaine reste identique même si les serveurs changent d'adresse IP. C'est un avantage majeur pour la pérennité des services.

---

### Arborescence des noms de domaine

> [!important] Structure hiérarchique
> Les noms de domaine suivent une structure arborescente, lue de droite à gauche, depuis la racine vers les feuilles.

**Composants de la hiérarchie :**
- **Racine** : appelée **point** (`.`)
- **TLD** : Top Level Domain (domaine de premier niveau)
- **Sous-domaines** : et ainsi de suite, séparés par des points (`.`)

**FQDN (Fully Qualified Domain Name) :**
- Nom de domaine complet incluant tous les niveaux jusqu'à la racine
- Le **point final** est souvent **omis** en pratique
- Exemple : `www.wildcodeschool.com.` (avec le point final explicite)

```
Arborescence DNS (simplifiée) :

                        . (racine)
                          |
        +-----------------+-----------------+
        |                 |                 |
       COM               FR               ORG
        |                 |                 |
   wildcodeschool       gouv          wikipedia
        |                 |                 |
     odyssey          ecologie             fr
        |                 |                 |
      www               www               www
```

> [!example] Lecture d'un nom de domaine
> Pour `www.odyssey.wildcodeschool.com.` :
> - `.` = racine (souvent omise)
> - `com` = TLD (Top Level Domain)
> - `wildcodeschool` = domaine de second niveau
> - `odyssey` = sous-domaine
> - `www` = sous-sous-domaine (souvent un alias)

**Exemples de TLD :**
- **Génériques** : `.com`, `.org`, `.net`
- **Nationaux** : `.fr` (France), `.香港` (Hong Kong en sinogrammes)
- **Nouveaux gTLD** : `.pro`, `.museum`, `.name`

> [!note] TLD internationalisés
> Depuis quelques années, les TLD peuvent utiliser des caractères non-latins. Par exemple `.香港` (Hong Kong) ou `.рф` (Russie).

---

### Les composants d'un nom de domaine

> [!info] Labels - Les composants d'un nom de domaine
> Chaque partie d'un nom de domaine séparée par un point est appelée un **label**.

**Règles pour les labels :**

| Règle | Détail |
|-------|--------|
| **Nombre** | En nombre quelconque (mais avoir un seul label est compliqué - TLD) |
| **Nombre minimal** | En général au moins 2 labels (domaine + TLD) |
| **Caractères** | Peuvent utiliser un jeu de caractères étendus - **IDN** (Internationalized Domain Name) |
| **Longueur maximale** | **63 caractères maximum** par label |
| **Longueur totale** | **255 caractères maximum** pour l'ensemble du nom de domaine |

**Caractères autorisés (traditionnellement) :**
- Lettres : `a-z` (non sensible à la casse)
- Chiffres : `0-9`
- Tiret : `-` (mais pas en début ou fin de label)

**IDN (Internationalized Domain Name) :**
- Permet l'utilisation de caractères non-ASCII
- Exemple : caractères accentués, cyrilliques, arabes, chinois, etc.
- Encodage technique en **Punycode** pour la compatibilité

> [!example] Exemple d'IDN
> Le nom de domaine `नेपाल.icom.museum` utilise des caractères en écriture devanagari (utilisée pour le népalais et d'autres langues).

> [!warning] Longueur maximale
> Attention à la limite de 63 caractères par label et 255 caractères au total. Ces limites sont rarement atteintes mais doivent être connues.

---

### Des exemples de noms de domaine

> [!example] Exemples variés de noms de domaine
> Voici différents types de noms de domaine illustrant la diversité des structures possibles.

**Exemples analysés :**

```
odyssey.wildcodeschool.com.
├─ . (racine - souvent omise)
├─ com (TLD)
├─ wildcodeschool (domaine de second niveau)
└─ odyssey (sous-domaine)
```

| Nom de domaine | Analyse |
|----------------|---------|
| `odyssey.wildcodeschool.com` | Sous-domaine de Wild Code School |
| `www.wildcodeschool.com` | Alias courant pour le site web principal |
| `fr.wikipedia.org` | Version française de Wikipédia |
| `नेपाल.icom.museum` | Nom de domaine internationalisé (IDN) avec caractères devanagari |
| `ietf.org` | Domaine de second niveau simple |
| `dany.wilder.name.fr` | Plusieurs niveaux de sous-domaines |
| `tssr.pro` | TLD spécialisé `.pro` pour professionnels |

> [!tip] Convention `www`
> Le sous-domaine `www` est une convention historique pour désigner le serveur web principal d'un domaine. Aujourd'hui, beaucoup de sites sont accessibles avec ou sans `www`.

> [!note] Diversité des structures
> Il n'y a pas de limite au nombre de sous-domaines que l'on peut créer (dans la limite des 255 caractères totaux). L'organisation dépend des besoins de l'entité propriétaire du domaine.

---

## Le protocole DNS

> [!abstract] Section : Le protocole DNS
> Cette section détaille le fonctionnement technique du protocole DNS, l'architecture client-serveur, et les différents types de serveurs impliqués dans la résolution de noms.

---

### La communication DNS

> [!quote] Base du protocole
> DNS est un **protocole client-serveur** de niveau applicatif (**couche 7** du modèle OSI).

**Caractéristiques du protocole :**

| Aspect | Détail |
|--------|--------|
| **Couche OSI** | Couche 7 (Application) |
| **Transport** | **UDP** (en général) ou **TCP** |
| **Port** | **53** (standard) |
| **Port sécurisé** | **853** (DNS over TLS - DoT) |
| **Pattern de communication** | En général : une requête → une réponse → fin de communication |

> [!info] UDP vs TCP
> - **UDP** est utilisé par défaut car plus rapide et suffisant pour des requêtes courtes
> - **TCP** est utilisé pour les transferts de zone, les réponses trop grandes (>512 octets), ou DoT/DoH

**Différents types de serveurs DNS :**

1. **Serveurs faisant autorité** (authoritative server)
   - Contiennent les données officielles pour une zone

2. **Résolveurs** (resolver) ou serveur DNS récursif
   - Aussi appelés "serveur de cache DNS"
   - Interrogent les serveurs autoritaires au nom des clients

> [!important] Port 53
> Le port 53 est un port standard à connaître absolument pour un TSSR. C'est le port utilisé par défaut pour toutes les communications DNS.

**Protocoles sécurisés modernes :**
- **DNS over TLS (DoT)** : Port 853
- **DNS over HTTPS (DoH)** : Port 443
- Ces protocoles chiffrent les requêtes DNS pour protéger la vie privée

---

### La résolution de nom

> [!info] Processus de résolution DNS
> La résolution d'un nom de domaine en adresse IP suit un processus en plusieurs étapes impliquant différents acteurs.

**Schéma du processus de résolution :**

```
┌──────────┐                                             
│  Client  │  1. Requête : odyssey.wildcodeschool.com ?
│          │────────────────────────────────────────────┐
└──────────┘                                            │
                                                        ▼
┌─────────────────┐                           ┌─────────────────┐
│  Stub resolver  │  8. Réponse IP            │   Résolveur     │
│  (lib sur l'OS) │◄──────────────────────────│   (FAI/LAN)     │
└─────────────────┘                           └─────────────────┘
                                                        │
                                      2. odyssey.wildcodeschool.com ?
                                                        ▼
                                              ┌──────────────────────┐
                                              │ Serveur DNS racine   │
                                              │  (root-servers.net)  │
                                              └──────────────────────┘
                                                        │
                                          3. NS de .com │
                                                        ▼
                                              ┌──────────────────────┐
                                              │ Serveur DNS de .com  │
                                              └──────────────────────┘
                                                        │
                                5. NS de wildcodeschool.com
                                                        ▼
                                       ┌─────────────────────────────────┐
                                       │ Serveur DNS de                  │
                                       │ wildcodeschool.com              │
                                       │ (serveur faisant autorité)      │
                                       └─────────────────────────────────┘
                                                        │
                          7. Réponse : 2a00:1450:4007:815::2013
```

**Étapes détaillées de la résolution :**

1. **Le client** demande l'adresse de `odyssey.wildcodeschool.com` au **stub resolver** local
2. Le **stub resolver** transmet la requête au **résolveur récursif** (FAI ou réseau local)
3. Le **résolveur** interroge un **serveur racine** (`.`)
4. Le **serveur racine** répond avec l'adresse des **serveurs NS du TLD .com**
5. Le **résolveur** interroge le **serveur .com**
6. Le **serveur .com** répond avec les **serveurs NS de wildcodeschool.com**
7. Le **résolveur** interroge le **serveur autoritaire de wildcodeschool.com**
8. Le **serveur autoritaire** répond avec l'**adresse IP** de `odyssey.wildcodeschool.com`
9. Le **résolveur** retourne la réponse au **stub resolver**
10. Le **stub resolver** retourne la réponse au **client**

> [!tip] Optimisation par cache
> En pratique, le résolveur met en cache les réponses. Si le TLD `.com` ou le domaine `wildcodeschool.com` ont déjà été consultés récemment, certaines étapes peuvent être évitées.

> [!important] Résolution récursive
> Le résolveur fait le travail récursif d'interroger successivement les différents serveurs. Le client n'a qu'à faire une seule requête au résolveur.

---

### Les serveurs faisant autorité

> [!quote] La source de vérité
> Un **serveur faisant autorité** (authoritative server) est un serveur contenant les informations officielles pour une (ou plusieurs) zone(s) DNS.

**Définition de zone :**
- Une **zone** est une **partie de l'arborescence des noms de domaine**
- Correspond généralement à un domaine et ses sous-domaines
- Délimite la responsabilité administrative d'un serveur

**Caractéristiques des serveurs autoritaires :**

| Aspect | Détail |
|--------|--------|
| **Contenu** | Données officielles pour une ou plusieurs zones |
| **Redondance** | En général plusieurs serveurs pour une même zone (tolérance de panne) |
| **Hiérarchie** | Serveur **primaire** et serveurs **secondaires** |
| **Synchronisation** | Via **transfert de zone** (zone transfer) en utilisant le protocole DNS |

**Serveur primaire vs secondaires :**
- **Primaire (master)** : contient la copie originale de la zone, où se font les modifications
- **Secondaires (slaves)** : répliques du primaire, se synchronisent automatiquement

**Solutions logicielles principales :**
- **BIND** (Berkeley Internet Name Domain) - le plus utilisé historiquement
- **NSD** (Name Server Daemon) - performant et sécurisé
- **Knot DNS** - moderne et rapide
- **PowerDNS** - flexible avec différents backends
- **Microsoft DNS** - intégré à Windows Server

> [!important] Redondance obligatoire
> Pour tout domaine public, il est **obligatoire** d'avoir au moins 2 serveurs DNS autoritaires (généralement un primaire et un secondaire) pour garantir la disponibilité.

> [!example] Exemple pratique
> Pour le domaine `wildcodeschool.com`, il pourrait y avoir :
> - `ns1.wildcodeschool.com` (primaire)
> - `ns2.wildcodeschool.com` (secondaire)
> - Tous deux contiennent les enregistrements DNS pour la zone `wildcodeschool.com`

---

### Les serveurs racines

> [!quote] Le départ
> Les **serveurs racines** (root servers) sont au sommet de la hiérarchie DNS. Ils connaissent les serveurs autoritaires de tous les TLD.

**Gestion des serveurs racines :**
- Gérés par l'**ICANN** (Internet Corporation for Assigned Names and Numbers)
- Administrés par 12 organisations différentes
- Dont le **RIPE NCC** (serveur K) et l'**ICANN** directement (serveur L)

**Les 13 domaines racines :**
- Nommés de `a.root-servers.net` à `m.root-servers.net`
- Où `<lettre>` = [a-m]
- Ces 13 adresses IP doivent être connues par tous les résolveurs

> [!important] 13 adresses, pas 13 serveurs !
> En réalité, il y a **bien plus de 13 serveurs physiques** grâce à la technique de l'**anycast**.

**Répartition mondiale :**
- Plus de **130 sites** répartis sur toute la planète
- L'anycast route automatiquement vers le serveur le plus proche
- Garantit performances et résilience

**Contenu de la zone racine :**
- Liste des **TLD** (Top Level Domains)
- Adresses des **serveurs NS** pour chaque TLD
- C'est le point de départ de toute résolution DNS

> [!tip] Ressource complémentaire
> Pour plus d'informations : [Serveur racine du DNS sur Wikipédia](https://fr.wikipedia.org/wiki/Serveur_racine_du_DNS)

> [!note] Anycast
> L'anycast est une technique réseau permettant à plusieurs serveurs de partager la même adresse IP. Les requêtes sont automatiquement routées vers le serveur le plus proche géographiquement.

---

### Les résolveurs

> [!quote] Relai et mémoire
> Les **résolveurs** (recursive resolvers) sont les serveurs DNS que les clients interrogent directement. Ils font le travail récursif de résolution de noms.

**Fonctionnement des résolveurs :**

**Initialisation :**
- À l'initialisation, contiennent **uniquement les 13 root-servers**
- Construisent progressivement leur cache en interrogeant la hiérarchie DNS

**Processus de résolution :**
1. Reçoivent une requête d'un client
2. Interrogent les **serveurs faisant autorité** pour obtenir l'information demandée
3. Suivent la hiérarchie DNS (racine → TLD → domaine)
4. Retournent la réponse au client

**Cache DNS :**
- Les résolveurs **mettent en cache** les réponses reçues
- Évite de refaire les mêmes requêtes à répétition
- Les informations en cache sont **temporaires** (contrôlées par le **TTL** - Time To Live)

> [!important] TTL (Time To Live)
> Chaque enregistrement DNS possède un TTL indiquant combien de temps il peut être conservé en cache. Après expiration, le résolveur doit redemander l'information.

**Localisation des résolveurs :**
- En général **chez les FAI** (Fournisseurs d'Accès à Internet)
- Ou sur un **réseau privé** et dédiés à un usage interne
- Configurés dans les paramètres réseau des clients

**Solutions logicielles pour résolveurs :**
- **Unbound** - résolveur récursif performant et sécurisé
- **BIND** - peut aussi faire office de résolveur
- **PowerDNS Recursor** - résolveur performant
- **Microsoft DNS** - sur Windows Server

> [!example] Scénario typique
> 1. Vous tapez `www.google.com` dans votre navigateur
> 2. Votre ordinateur interroge le résolveur de votre FAI (ex : Free, Orange)
> 3. Le résolveur consulte son cache ou interroge la hiérarchie DNS
> 4. Le résolveur vous répond avec l'adresse IP de Google
> 5. Votre navigateur se connecte à cette adresse IP

---

### Les résolveurs publiques

> [!info] DNS publique
> Certains résolveurs ne sont pas privés et sont **accessibles à tous** sur Internet. Ils offrent des alternatives aux résolveurs des FAI.

**Principaux résolveurs publics :**

| Fournisseur | Pays | Adresses IPv4 | Adresses IPv6 |
|-------------|------|---------------|---------------|
| **Quad9** | Suisse | `9.9.9.9`<br>`149.112.112.112` | `2620:fe::fe`<br>`2620:fe::9` |
| **Cloudflare** | US | `1.1.1.1`<br>`1.0.0.1` | `2606:4700:4700::1111`<br>`2606:4700:4700::1001` |
| **Google** | US | `8.8.8.8`<br>`8.8.4.4` | `2001:4860:4860::8888`<br>`2001:4860:4860::8844` |
| **FDN** | FR | `80.67.169.12`<br>`80.67.169.40` | `2001:910:800::12`<br>`2001:910:800::40` |

> [!tip] Avantages des DNS publics
> - Souvent **plus rapides** que les DNS des FAI
> - Meilleure **disponibilité**
> - Certains offrent des fonctionnalités supplémentaires (filtrage de malwares, blocage de publicités)
> - **Confidentialité** variable selon le fournisseur

> [!warning] Considérations de confidentialité
> Utiliser un DNS public signifie que ce fournisseur peut voir tous les noms de domaine que vous résolvez. Choisissez un fournisseur de confiance.

**Caractéristiques par fournisseur :**

- **Quad9** : basé en Suisse, focus sur la sécurité (bloque les domaines malveillants)
- **Cloudflare** : promet de ne pas enregistrer les logs, focus sur la vitesse
- **Google** : très rapide, mais Google collecte des données anonymisées
- **FDN** : associatif français, respectueux de la vie privée

> [!example] Configuration d'un DNS public
> Sur Linux, pour utiliser Cloudflare DNS :
> ```bash
> # Modifier /etc/resolv.conf
> nameserver 1.1.1.1
> nameserver 1.0.0.1
> ```

---

### Stub resolver (DNS local)

> [!quote] La résolution DNS en local
> Un **stub resolver** (résolveur minimal) est le composant DNS présent sur chaque machine cliente.

**Caractéristiques du stub resolver :**

| Fonctionnalité | Description |
|----------------|-------------|
| **Résolution récursive** | **NE gère PAS** la partie récursive (délègue au résolveur complet) |
| **Cache local** | Gère (en général) un **cache** pour économiser les requêtes |
| **Configuration** | Doit connaître l'adresse d'**au moins un résolveur récursif** |
| **Intégration système** | En général **intégré au système d'exploitation** |

**Fonctionnement :**
1. Reçoit les requêtes DNS des applications locales
2. Consulte son cache local
3. Consulte le fichier `/etc/hosts` (sur Unix/Linux)
4. Si non trouvé, transmet la requête à un résolveur récursif
5. Met en cache la réponse reçue

**Implémentations courantes :**
- **systemd-resolved** (Linux moderne avec systemd)
- **libc resolver** (bibliothèque C standard)
- **Windows DNS Client** (service Windows)

> [!important] Stub resolver = client DNS
> Le stub resolver est le "client DNS" de votre machine. C'est lui qui fait l'interface entre les applications (navigateur, mail, etc.) et le résolveur récursif.

> [!example] Fichier /etc/hosts
> Sur Linux, le fichier `/etc/hosts` est consulté avant toute requête DNS :
> ```
> 127.0.0.1       localhost
> 192.168.1.10    serveur-local.lan
> ```

> [!tip] Ordre de résolution
> Ordre typique de résolution sur Linux :
> 1. Cache du stub resolver
> 2. Fichier `/etc/hosts`
> 3. Requête au résolveur DNS configuré

---

### Configuration DNS d'un hôte

> [!info] Et sur un client ?
> Chaque machine cliente doit être configurée pour savoir quel(s) résolveur(s) DNS utiliser.

**Configuration réseau :**
- La configuration réseau d'une machine indique **au moins une** (mais en général **plusieurs**) adresse(s) de résolveur(s) DNS
- Ces informations sont en général **fournies par DHCP**

**Processus de résolution sur un client :**

1. **Une application** veut accéder à une information DNS (ex : récupérer l'adresse via le nom)

2. **Elle questionne le stub resolver du système** (la libc, par exemple)

3. **Le stub resolver :**
   - Regarde dans son **cache**
   - Consulte le fichier **`/etc/hosts`** (sous Unix/Linux)

4. **Si absent :**
   - S'adresse au **résolveur récursif** configuré dans sa config
   - Si pas de réponse, essaie un autre résolveur configuré
   - Continue jusqu'à obtenir une réponse ou épuisement des résolveurs

> [!example] Configuration DNS sur Linux
> Fichier `/etc/resolv.conf` :
> ```bash
> # DNS Cloudflare
> nameserver 1.1.1.1
> nameserver 1.0.0.1
> # Domaine de recherche
> search wildcodeschool.com
> ```

> [!example] Configuration DNS sur Windows
> Via PowerShell :
> ```powershell
> # Voir la configuration DNS
> Get-DnsClientServerAddress
> 
> # Configurer un DNS
> Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "1.1.1.1","1.0.0.1"
> ```

**Méthodes de configuration :**

| Méthode | Description | Usage |
|---------|-------------|-------|
| **DHCP** | Configuration automatique par le serveur DHCP | Standard en entreprise et chez les FAI |
| **Statique** | Configuration manuelle | Serveurs, postes de travail spécifiques |
| **VPN** | Configuration fournie par le serveur VPN | Connexions VPN |

> [!important] Redondance DNS
> Il est recommandé de configurer au moins 2 serveurs DNS pour garantir la disponibilité en cas de panne de l'un d'eux.

> [!warning] Ordre des serveurs DNS
> Le premier serveur DNS configuré sera utilisé en priorité. Le second n'est interrogé qu'en cas d'échec du premier (timeout ou erreur).

---

## Les enregistrements

> [!abstract] Section : Les enregistrements DNS
> Cette section présente les différents types d'enregistrements DNS (Resource Records) et leur utilisation.

---

### Les enregistrements DNS (Resource Records)

> [!quote] Pas que des adresses IP
> DNS associe à un nom de domaine des **Resource Record (RR)**. Il existe de nombreux types d'enregistrements permettant de stocker différentes informations.

**Principaux types d'enregistrements :**

| Type | Nom complet | Description |
|------|-------------|-------------|
| **A** | Address Record | Associe un nom à une ou plusieurs **adresses IPv4** |
| **AAAA** | IPv6 Address Record | Associe un nom à une ou plusieurs **adresses IPv6** |
| **NS** | Name Server | Indique le **serveur faisant autorité** sur ce domaine |
| **CNAME** | Canonical Name | Définit le **nom canonique** d'un alias (pointeur vers un autre nom) |
| **SOA** | Start of Authority | Définit le **serveur primaire** d'une zone et ses paramètres |
| **PTR** | Pointer | Utilisé pour la **résolution inverse** (IP → nom) |
| **MX** | Mail Exchanger | Indique le **serveur de courrier** du domaine |
| **TXT** | Text | Contient du **texte libre** (souvent pour validation, SPF, DKIM) |
| **SRV** | Service | Indique l'emplacement de **services spécifiques** |

> [!important] TTL (Time To Live)
> Tous les Resource Records ont un **TTL** (Time To Live) qui indique le **temps maximum de validité** dans un cache. Exprimé en secondes.

**Exemples d'enregistrements :**

```dns
; Enregistrement A (IPv4)
www.wildcodeschool.com.    3600    IN    A    192.0.2.1

; Enregistrement AAAA (IPv6)
www.wildcodeschool.com.    3600    IN    AAAA    2001:db8::1

; Enregistrement CNAME (alias)
ftp.wildcodeschool.com.    3600    IN    CNAME    www.wildcodeschool.com.

; Enregistrement MX (mail)
wildcodeschool.com.        3600    IN    MX    10 mail.wildcodeschool.com.

; Enregistrement TXT (texte libre)
wildcodeschool.com.        3600    IN    TXT    "v=spf1 include:_spf.google.com ~all"
```

> [!tip] Format d'un enregistrement DNS
> Format général : `nom TTL classe type valeur`
> - **nom** : nom de domaine concerné
> - **TTL** : durée de vie en cache (en secondes)
> - **classe** : presque toujours `IN` (Internet)
> - **type** : type d'enregistrement (A, AAAA, CNAME, etc.)
> - **valeur** : donnée associée

**Enregistrements spéciaux :**

> [!info] SOA (Start of Authority)
> L'enregistrement SOA est obligatoire pour chaque zone. Il contient :
> - Le nom du serveur primaire
> - L'email de l'administrateur
> - Le numéro de série de la zone
> - Les paramètres de rafraîchissement et d'expiration

> [!info] NS (Name Server)
> Les enregistrements NS indiquent quels serveurs sont autoritaires pour une zone. Chaque zone doit avoir au moins 2 enregistrements NS.

> [!example] Utilisation de CNAME
> Les CNAME permettent de créer des alias. Par exemple :
> ```
> www.wildcodeschool.com.    IN    CNAME    wildcodeschool.com.
> ```
> Cela signifie que `www.wildcodeschool.com` pointe vers `wildcodeschool.com`.

> [!warning] Limitation des CNAME
> Un CNAME ne peut pas coexister avec d'autres enregistrements pour le même nom. Un CNAME ne peut pas pointer vers un autre CNAME (pas de chaînage).

**Ressource complémentaire :**
Pour une liste complète des types d'enregistrements DNS, consulter [Liste des enregistrements DNS sur Wikipédia](https://fr.wikipedia.org/wiki/Liste_des_enregistrements_DNS)

---

### La résolution inverse

> [!quote] Dans l'autre sens
> La **résolution inverse** (reverse DNS) permet de récupérer le **nom de domaine** à partir d'une **adresse IP**.

**Principe de la résolution inverse :**
- Utilise un pseudo-domaine spécial : **`.in-addr.arpa`** (IPv4) ou **`.ip6.arpa`** (IPv6)
- Inverse le sens de l'adresse IP (pour correspondre au sens des noms de domaine)
- Stockée dans des enregistrements **PTR** (Pointer)

**Nécessité de la résolution inverse :**
- **Nécessaire pour les adresses publiques** (notamment les serveurs de mail)
- Utilisée pour la vérification et la sécurité
- Certains services refusent les connexions sans PTR valide

**Fonctionnement pour IPv4 :**
- Découpe par **octet** et notation **décimale**
- Inversion de l'ordre des octets
- Ajout du suffixe `.in-addr.arpa`

> [!example] Résolution inverse IPv4
> Pour l'adresse IP `172.67.146.155` :
> 1. Inverser l'ordre : `155.146.67.172`
> 2. Ajouter le suffixe : `155.146.67.172.in-addr.arpa`
> 3. Créer un enregistrement PTR :
> ```
> 155.146.67.172.in-addr.arpa.    IN    PTR    wildcodeschool.com.
> ```

**Fonctionnement pour IPv6 :**
- Découpe par **chiffre hexadécimal**
- Inversion de l'ordre des chiffres
- Ajout du suffixe `.ip6.arpa`

> [!example] Résolution inverse IPv6
> Pour l'adresse IPv6 `2001:db8::1` :
> 1. Écriture complète : `2001:0db8:0000:0000:0000:0000:0000:0001`
> 2. Découper en nibbles (chiffres hexa) et inverser :
> ```
> 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa.
> ```
> 3. Créer un enregistrement PTR vers le nom de domaine

> [!warning] Complexité IPv6
> La notation complète de la résolution inverse IPv6 est très longue (64 caractères + suffixe). C'est normal et prévu par la RFC.

> [!tip] Vérification PTR
> Pour vérifier un enregistrement PTR :
> ```bash
> # IPv4
> dig -x 172.67.146.155
> 
> # IPv6
> dig -x 2001:db8::1
> ```

**Exemple complet de l'énoncé :**
```
155.146.67.172.in-addr.arpa
b.9.2.9.3.4.C.1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.3.0.2.3.0.0.7.4.6.0.6.2.ip6.arpa
```

---

### DNS round-robin

> [!info] Le tourniquet
> Le **DNS round-robin** est une technique simple de **répartition de charge** (load balancing) utilisant plusieurs adresses IP pour un même nom.

**Principe :**
- On associe **plusieurs adresses** à un même nom de domaine
- À chaque requête DNS, la **réponse est différente**
- Le serveur DNS **fait tourner les adresses** (rotation)

**Fonctionnement :**
1. Un nom de domaine a plusieurs enregistrements A (ou AAAA)
2. Le serveur DNS retourne ces adresses dans un ordre différent à chaque requête
3. Les clients utilisent généralement la première adresse de la réponse
4. La charge est ainsi distribuée entre les différents serveurs

> [!example] Configuration round-robin
> ```dns
> www.wildcodeschool.com.    3600    IN    A    192.0.2.1
> www.wildcodeschool.com.    3600    IN    A    192.0.2.2
> www.wildcodeschool.com.    3600    IN    A    192.0.2.3
> ```
> 
> - Requête 1 : réponse `192.0.2.1, 192.0.2.2, 192.0.2.3`
> - Requête 2 : réponse `192.0.2.2, 192.0.2.3, 192.0.2.1`
> - Requête 3 : réponse `192.0.2.3, 192.0.2.1, 192.0.2.2`

**Avantages :**
- **Simplicité** de mise en œuvre
- **Répartition automatique** de la charge
- Pas besoin d'équipement spécialisé (load balancer)

**Limitations :**
- Pas de vérification de l'**état de santé** des serveurs
- Pas de répartition **intelligente** (tous les serveurs reçoivent environ la même charge)
- Le **cache DNS** peut limiter l'efficacité de la rotation
- Sessions non persistantes (sticky sessions) difficiles à gérer

> [!warning] Pas un vrai load balancer
> Le DNS round-robin est une solution simple mais ne remplace pas un vrai load balancer. Il n'y a pas de détection de panne ni de répartition intelligente de la charge.

> [!tip] Solution moderne
> Pour une répartition de charge professionnelle, utiliser un vrai load balancer (HAProxy, nginx, cloud load balancer) ou un système d'anycast.

---

## Enregistrer un nom de domaine

> [!abstract] Section : Enregistrer un nom de domaine
> Cette section explique comment fonctionne l'enregistrement et la gestion des noms de domaine, de la racine jusqu'aux bureaux d'enregistrement.

---

### Racine et TLD

> [!quote] Comment tout ça est géré ?
> L'enregistrement et la gestion des noms de domaine suivent une structure hiérarchique stricte, commençant par la racine DNS.

**Gestion de la racine DNS :**
- La racine DNS est gérée par l'**ICANN** (Internet Corporation for Assigned Names and Numbers)
- Plus précisément via sa composante **IANA** (Internet Assigned Numbers Authority)
- C'est à la **racine** qu'on doit enregistrer les **TLD** (Top Level Domains)

**Historique des TLD :**

Longtemps, la liste des TLD a été **très limitée** :

| Catégorie | Type | Exemples | Description |
|-----------|------|----------|-------------|
| **ccTLD** | Nationaux | `.fr`, `.de`, `.uk`, `.us` | Country Code TLD - un par pays/territoire |
| **gTLD ouverts** | Génériques | `.com`, `.org`, `.net`, `.info` | Ouverts à tous |
| **Commandités** | Spécialisés | `.edu`, `.gov`, `.mil` | Réservés à des activités particulières |
| **Spéciaux** | Techniques | `.arpa`, `.example`, `.localhost`, `.test`, `.invalid` | Usage technique ou réservé |

**Évolution récente :**
- Depuis **2012**, de nombreux autres TLD ont vu le jour
- Nouveaux gTLD : `.paris`, `.restaurant`, `.app`, `.dev`, etc.
- Tous enregistrés à l'**IANA**

> [!important] ICANN/IANA
> - **ICANN** : organisation de gouvernance globale d'Internet
> - **IANA** : département de l'ICANN gérant les ressources Internet (IP, DNS, protocoles)

> [!info] ccTLD (Country Code TLD)
> Chaque pays ou territoire possède un ccTLD de 2 lettres basé sur la norme ISO 3166-1 alpha-2. Exemple : `.fr` pour France, `.jp` pour Japon, `.ca` pour Canada.

> [!example] Nouveaux TLD depuis 2012
> Exemples de nouveaux TLD :
> - Géographiques : `.paris`, `.london`, `.nyc`
> - Thématiques : `.restaurant`, `.music`, `.photo`
> - Entreprises : `.google`, `.amazon`, `.apple`
> - Techniques : `.app`, `.dev`, `.cloud`

> [!note] Domaine .arpa
> Le domaine `.arpa` (Address and Routing Parameter Area) est un domaine technique spécial utilisé principalement pour la résolution inverse (`in-addr.arpa` et `ip6.arpa`).

---

### Registre de noms de domaine

> [!quote] Définition : Registre de noms de domaine
> Un **registre de nom de domaine** (domain name registry) désigne à la fois la base de données des informations sur un domaine et l'organisme en charge de sa gestion.

**Deux significations du terme "registre" :**

1. **La base de données** : domain name registry
   - Contient toutes les informations sur les noms de domaine d'un TLD
   - Enregistrements techniques, propriétaires, dates d'expiration, etc.

2. **L'organisme** : registry operator
   - Organisation en charge de la gestion d'un TLD
   - Parfois aussi appelé **NIC** (Network Information Center)

**Délégation de gestion :**
- L'**IANA** gère la racine
- L'IANA **délègue** la gestion des sous-domaines (TLD) à d'autres organismes
- Chaque TLD a son propre **registre**

> [!example] L'AFNIC - Exemple de registre
> L'**AFNIC** (Association Française pour le Nommage Internet en Coopération) est le registre de :
> - `.fr` (France métropolitaine)
> - `.pm` (Saint-Pierre-et-Miquelon)
> - `.re` (La Réunion)
> - `.tf` (Terres australes et antarctiques françaises)
> - `.wf` (Wallis-et-Futuna)
> - `.yt` (Mayotte)

**Autres exemples de registres :**

| TLD | Registre | Pays/Organisation |
|-----|----------|-------------------|
| `.com`, `.net` | Verisign | États-Unis |
| `.org` | Public Interest Registry | États-Unis |
| `.de` | DENIC | Allemagne |
| `.uk` | Nominet | Royaume-Uni |
| `.eu` | EURid | Union Européenne |

**Rôles du registre :**
- Maintenir la base de données des noms de domaine
- Gérer les serveurs DNS autoritaires du TLD
- Définir les règles d'enregistrement
- Accréditer les bureaux d'enregistrement (registrars)

> [!important] Distinction registre/registrar
> - **Registre (registry)** : gère le TLD en gros
> - **Bureau d'enregistrement (registrar)** : interface avec les clients finaux
> Les deux sont différents mais travaillent ensemble.

---

### Bureau d'enregistrement

> [!quote] À qui m'adresser ?
> Un **bureau d'enregistrement** (registrar) est chargé par un registre de la **relation avec les clients** voulant réserver un nom de domaine.

**Rôle du bureau d'enregistrement :**
- Interface entre le **client final** et le **registre**
- Permet de **réserver un nom de domaine**
- Gère le renouvellement et les modifications
- Collecte les informations du propriétaire (WHOIS)

**Services additionnels :**
- Parfois, le bureau d'enregistrement peut aussi **héberger votre zone DNS** sur ses serveurs (**Hébergeur DNS**)
- Sinon, il doit indiquer l'adresse de vos serveurs (**NS**) pour le domaine que vous réservez

**Exemples de bureaux d'enregistrement :**
- **OVH** (France)
- **Gandi** (France)
- **GoDaddy** (États-Unis)
- **Namecheap** (États-Unis)
- **Ionos** (Allemagne)

> [!info] Accréditation
> Pour le TLD `.fr`, il y a **383 bureaux d'enregistrement accrédités** par l'AFNIC.

**Processus d'enregistrement d'un domaine :**

1. **Choix du nom** : vérifier la disponibilité
2. **Choix du bureau d'enregistrement** : comparer les prix et services
3. **Enregistrement** : fournir les informations nécessaires
4. **Configuration DNS** : soit chez le registrar, soit pointer vers vos propres serveurs NS
5. **Paiement** : annuel ou pluriannuel
6. **Renouvellement** : à effectuer avant expiration

> [!warning] Expiration du domaine
> Si vous ne renouvelez pas votre domaine à temps, il peut être libéré et enregistré par quelqu'un d'autre. Mettez en place des renouvellements automatiques ou des rappels.

**Hébergement DNS :**

| Option | Description | Usage |
|--------|-------------|-------|
| **DNS du registrar** | Le bureau d'enregistrement héberge votre zone DNS | Simple, convient à la plupart des usages |
| **DNS externe** | Vous utilisez vos propres serveurs ou un service tiers | Contrôle total, flexibilité, hébergement mutualisé |

> [!example] Enregistrer wildcodeschool.com
> 1. Vérifier que `wildcodeschool.com` est disponible
> 2. Choisir un registrar (ex : Gandi)
> 3. Fournir les informations du propriétaire
> 4. Configurer les serveurs DNS (ex : `ns1.gandi.net`, `ns2.gandi.net`)
> 5. Payer l'enregistrement (prix variable selon le TLD)
> 6. Attendre la propagation DNS (quelques heures)

> [!tip] Comparaison des registrars
> Comparez les prix, les services inclus (email, WHOIS privé, SSL), et la qualité du support avant de choisir un bureau d'enregistrement.

---

## Outils

> [!abstract] Section : Outils DNS
> Cette section présente les outils en ligne de commande pour interroger et diagnostiquer DNS.

---

### dig - Domain Information Groper

> [!quote] Interroger DNS
> **dig** (Domain Information Groper) est une commande Unix pour interroger des serveurs DNS qui fait partie de la suite **BIND**.

**Caractéristiques de dig :**
- Outil en **ligne de commande** pour interroger DNS
- Fait partie de la suite **BIND** (Berkeley Internet Name Domain)
- Disponible sur **Linux, macOS, BSD**
- **Préféré** à `nslookup` ou `host` car offrant plus de fonctionnalités

**Disponibilité :**
- **Unix/Linux** : `dig` installé par défaut ou via paquet `bind-utils` / `dnsutils`
- **macOS** : `dig` disponible par défaut
- **Windows** : seul `nslookup` est disponible par défaut (dig peut être installé via BIND)

> [!important] Alternative Windows
> Sur Windows, l'outil par défaut est `nslookup`. Pour utiliser `dig`, il faut installer BIND for Windows ou utiliser WSL.

**Syntaxe de base :**

```bash
dig [nom_de_domaine] [type] [@serveur_dns]
```

**Exemples d'utilisation :**

```bash
# Requête A (IPv4) simple
dig wildcodeschool.com

# Requête AAAA (IPv6)
dig wildcodeschool.com AAAA

# Requête MX (serveurs mail)
dig wildcodeschool.com MX

# Requête NS (serveurs DNS)
dig wildcodeschool.com NS

# Requête vers un serveur DNS spécifique
dig @8.8.8.8 wildcodeschool.com

# Requête inverse (PTR)
dig -x 172.67.146.155

# Afficher uniquement la réponse courte
dig +short wildcodeschool.com

# Tracer le chemin de résolution
dig +trace wildcodeschool.com

# Interroger tous les types d'enregistrements
dig wildcodeschool.com ANY
```

> [!example] Lecture de la sortie de dig
> ```bash
> $ dig wildcodeschool.com
> 
> ; <<>> DiG 9.18.28 <<>> wildcodeschool.com
> ;; QUESTION SECTION:
> ;wildcodeschool.com.        IN  A
> 
> ;; ANSWER SECTION:
> wildcodeschool.com.     300 IN  A   172.67.146.155
> 
> ;; Query time: 23 msec
> ;; SERVER: 1.1.1.1#53(1.1.1.1)
> ;; WHEN: Mon Jan 06 10:30:00 CET 2026
> ;; MSG SIZE  rcvd: 64
> ```

**Sections de la sortie :**
- **QUESTION SECTION** : la requête envoyée
- **ANSWER SECTION** : la réponse reçue
- **AUTHORITY SECTION** : serveurs autoritaires (si présents)
- **ADDITIONAL SECTION** : informations supplémentaires (si présentes)

**Options utiles :**

| Option | Description |
|--------|-------------|
| `+short` | Affiche uniquement la réponse (format court) |
| `+trace` | Trace la résolution récursive depuis la racine |
| `+noall +answer` | Affiche uniquement la section réponse |
| `+stats` | Affiche les statistiques de la requête |
| `@serveur` | Interroge un serveur DNS spécifique |
| `-x adresse` | Résolution inverse (PTR) |

> [!tip] Option +trace
> L'option `+trace` est très utile pour comprendre le processus de résolution DNS et identifier où se situe un problème.

> [!example] dig +trace
> ```bash
> $ dig +trace wildcodeschool.com
> 
> .               518400  IN  NS  a.root-servers.net.
> .               518400  IN  NS  b.root-servers.net.
> [...]
> com.            172800  IN  NS  a.gtld-servers.net.
> [...]
> wildcodeschool.com. 300 IN  A   172.67.146.155
> ```
> Cette commande montre le chemin complet : racine → .com → wildcodeschool.com

**Alternatives à dig :**

| Outil | Description | Usage |
|-------|-------------|-------|
| **host** | Outil simple pour requêtes DNS | Sortie plus lisible, moins de détails |
| **nslookup** | Outil historique, disponible sur Windows | Mode interactif possible |
| **drill** | Alternative à dig (LDNS) | Syntaxe similaire à dig |

> [!warning] nslookup vs dig
> Bien que `nslookup` soit plus ancien et disponible sur Windows, `dig` est généralement préféré sur Unix/Linux pour sa richesse fonctionnelle et sa sortie détaillée.

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP
> Voici les éléments essentiels à maîtriser sur le DNS pour ton titre TSSR.

### Concepts fondamentaux

- **DNS** est le système qui permet de convertir des **noms de domaine** en **adresses IP**
- C'est une **base de données distribuée et hiérarchique** garantissant robustesse et passage à l'échelle
- Le DNS est une **infrastructure invisible** mais essentielle au fonctionnement d'Internet
- Le système est né en **1983** pour remplacer le fichier **HOSTS.TXT** devenu obsolète

### Structure des noms de domaine

- Un nom de domaine est **hiérarchique**, lu de **droite à gauche**
- Structure : **sous-domaine.domaine.TLD.racine** (le point final est souvent omis)
- La **racine** est notée `.` (point)
- **TLD** = Top Level Domain (`.com`, `.fr`, `.org`, etc.)
- **FQDN** = Fully Qualified Domain Name (nom complet avec tous les niveaux)
- Un label peut contenir jusqu'à **63 caractères**, le nom complet jusqu'à **255 caractères**

### Architecture DNS

**Les trois types de serveurs :**

1. **Serveurs racines** (13 domaines, > 130 sites réels)
   - Gérés par l'ICANN, connus de tous les résolveurs
   - Nommés de `a.root-servers.net` à `m.root-servers.net`
   - Point de départ de toute résolution

2. **Serveurs autoritaires** (authoritative servers)
   - Contiennent les données officielles d'une zone
   - Serveur **primaire** (master) et **secondaires** (slaves)
   - Synchronisation par **transfert de zone**

3. **Résolveurs** (recursive resolvers)
   - Interrogent la hiérarchie DNS pour le compte des clients
   - Mettent en **cache** les réponses (contrôlé par le **TTL**)
   - Peuvent être privés (FAI) ou publics (Cloudflare, Google, Quad9)

### Processus de résolution

1. Client interroge le **stub resolver** local
2. Stub resolver consulte le **cache** et `/etc/hosts`
3. Si absent, interroge le **résolveur récursif** configuré
4. Le résolveur interroge successivement : **racine** → **TLD** → **domaine autoritaire**
5. Le résolveur met la réponse en **cache** et la retourne au client

### Resource Records (RR)

**Types d'enregistrements essentiels :**

| Type | Usage |
|------|-------|
| **A** | Adresse IPv4 |
| **AAAA** | Adresse IPv6 |
| **NS** | Serveur faisant autorité |
| **CNAME** | Alias (nom canonique) |
| **MX** | Serveur de courrier |
| **PTR** | Résolution inverse (IP → nom) |
| **SOA** | Serveur primaire d'une zone |
| **TXT** | Texte libre (SPF, DKIM, validation) |

- Chaque enregistrement a un **TTL** (Time To Live) contrôlant la durée de cache

### Résolution inverse

- Utilise les pseudo-domaines **`.in-addr.arpa`** (IPv4) et **`.ip6.arpa`** (IPv6)
- L'adresse IP est **inversée** pour correspondre au sens des noms de domaine
- Stockée dans des enregistrements **PTR**
- **Nécessaire** pour les serveurs publics (notamment mail)

### Protocole DNS

- Protocole **client-serveur**, couche **7** (Application)
- Port **53** (UDP en général, TCP pour transferts de zone)
- Port **853** pour DNS over TLS (DoT)
- En général : **une requête → une réponse**

### Enregistrement de domaines

**Hiérarchie de gestion :**

1. **ICANN/IANA** : gère la racine
2. **Registre** (registry) : gère un TLD (ex : AFNIC pour `.fr`)
3. **Bureau d'enregistrement** (registrar) : interface avec les clients (ex : OVH, Gandi)

### Outils DNS

- **dig** : outil principal sur Unix/Linux pour interroger DNS
- Préféré à `nslookup` ou `host` pour ses fonctionnalités étendues
- Options importantes : `+trace`, `+short`, `-x` (résolution inverse)
- Sur Windows : utiliser `nslookup` (par défaut)

### Configuration client

- Configuration via **/etc/resolv.conf** (Linux) ou paramètres réseau (Windows)
- Fournie généralement par **DHCP**
- Recommandé : configurer **au moins 2 serveurs DNS** (redondance)
- Le stub resolver consulte dans l'ordre : **cache** → **/etc/hosts** → **résolveur DNS**

### Concepts avancés

- **DNS round-robin** : répartition de charge simple par rotation d'adresses
- **Transfert de zone** : synchronisation primaire → secondaires
- **Cache DNS** : optimisation des performances, contrôlé par TTL
- **Résolveurs publics** : Cloudflare (1.1.1.1), Google (8.8.8.8), Quad9 (9.9.9.9)

### Points d'attention pour l'examen

- Savoir expliquer le **processus complet de résolution** d'un nom
- Connaître les **principaux types d'enregistrements** et leur usage
- Comprendre la **hiérarchie** : racine → TLD → domaine → sous-domaine
- Maîtriser l'utilisation de **dig** pour le diagnostic
- Connaître la différence entre **serveur autoritaire** et **résolveur**
- Comprendre le rôle du **TTL** dans le cache DNS
- Savoir configurer un client DNS (Linux et Windows)

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR
> Termes techniques importants à maîtriser pour comprendre et travailler avec DNS.

| Terme | Définition |
|-------|------------|
| **DNS** | Domain Name System - Système de noms de domaine permettant de convertir des noms en adresses IP |
| **FQDN** | Fully Qualified Domain Name - Nom de domaine complet incluant tous les niveaux jusqu'à la racine |
| **TLD** | Top Level Domain - Domaine de premier niveau (.com, .fr, .org, etc.) |
| **ccTLD** | Country Code TLD - TLD national de 2 lettres (ex : .fr, .de, .uk) |
| **gTLD** | Generic TLD - TLD générique ouvert (.com, .net, .org, etc.) |
| **IDN** | Internationalized Domain Name - Nom de domaine avec caractères non-ASCII |
| **Label** | Composant d'un nom de domaine séparé par des points (max 63 caractères) |
| **Racine** | Sommet de la hiérarchie DNS, noté `.` (point) |
| **Zone** | Partie de l'arborescence DNS gérée par un serveur autoritaire |
| **Serveur autoritaire** | Serveur contenant les données officielles d'une zone DNS |
| **Résolveur** | Serveur DNS récursif interrogeant la hiérarchie DNS pour les clients |
| **Stub resolver** | Client DNS minimal intégré au système d'exploitation |
| **Résolution récursive** | Processus d'interrogation successive de la hiérarchie DNS |
| **Résolution inverse** | Conversion d'une adresse IP en nom de domaine (via PTR) |
| **Resource Record (RR)** | Enregistrement DNS associant un nom à une donnée |
| **TTL** | Time To Live - Durée de validité d'un enregistrement dans un cache (en secondes) |
| **Cache DNS** | Stockage temporaire des réponses DNS pour optimiser les performances |
| **A** | Type d'enregistrement contenant une adresse IPv4 |
| **AAAA** | Type d'enregistrement contenant une adresse IPv6 (4 × A) |
| **NS** | Name Server - Enregistrement indiquant le serveur autoritaire d'une zone |
| **CNAME** | Canonical Name - Alias pointant vers un autre nom de domaine |
| **PTR** | Pointer - Enregistrement pour la résolution inverse |
| **MX** | Mail Exchanger - Serveur de courrier du domaine |
| **SOA** | Start of Authority - Enregistrement définissant le serveur primaire d'une zone |
| **TXT** | Enregistrement de texte libre (validation, SPF, DKIM, etc.) |
| **SRV** | Service - Emplacement de services spécifiques |
| **Serveur primaire** | Master - Serveur contenant la copie originale d'une zone |
| **Serveur secondaire** | Slave - Réplique du serveur primaire, synchronisé par transfert de zone |
| **Transfert de zone** | Synchronisation des données entre serveur primaire et secondaires |
| **Root servers** | 13 domaines (a-m.root-servers.net) au sommet de la hiérarchie DNS |
| **Anycast** | Technique permettant à plusieurs serveurs de partager une même adresse IP |
| **ICANN** | Internet Corporation for Assigned Names and Numbers - Organisme gérant la racine DNS |
| **IANA** | Internet Assigned Numbers Authority - Département de l'ICANN gérant les ressources Internet |
| **Registre** | Organisation gérant un TLD (registry operator) |
| **Bureau d'enregistrement** | Registrar - Interface entre clients et registre pour enregistrer des domaines |
| **NIC** | Network Information Center - Autre nom pour un registre de domaines |
| **AFNIC** | Association Française pour le Nommage Internet en Coopération - Registre du .fr |
| **WHOIS** | Protocole et base de données des informations sur les propriétaires de domaines |
| **DNS round-robin** | Répartition de charge par rotation de plusieurs adresses IP pour un même nom |
| **UDP** | User Datagram Protocol - Protocole de transport principal pour DNS (port 53) |
| **TCP** | Transmission Control Protocol - Utilisé pour transferts de zone et grandes réponses DNS |
| **DoT** | DNS over TLS - DNS chiffré sur le port 853 |
| **DoH** | DNS over HTTPS - DNS chiffré sur le port 443 |
| **dig** | Domain Information Groper - Outil en ligne de commande pour interroger DNS |
| **nslookup** | Name Server Lookup - Outil historique d'interrogation DNS (Windows) |
| **host** | Outil Unix simple pour requêtes DNS |
| **/etc/hosts** | Fichier local de correspondances nom/IP (Unix/Linux) |
| **/etc/resolv.conf** | Fichier de configuration DNS sur Unix/Linux |
| **systemd-resolved** | Service de résolution DNS sur Linux moderne (systemd) |
| **BIND** | Berkeley Internet Name Domain - Serveur DNS le plus utilisé historiquement |
| **RFC 1034** | RFC définissant les concepts et facilités du DNS (1987) |
| **RFC 1035** | RFC définissant l'implémentation et les spécifications du DNS (1987) |
| **.in-addr.arpa** | Pseudo-domaine pour résolution inverse IPv4 |
| **.ip6.arpa** | Pseudo-domaine pour résolution inverse IPv6 |
| **.arpa** | Domaine technique spécial (Address and Routing Parameter Area) |
| **Punycode** | Encodage des caractères Unicode pour IDN en ASCII compatible |
| **DNSSEC** | DNS Security Extensions - Extensions de sécurité pour DNS (signatures cryptographiques) |

---

**Fin du document de révision DNS - TSSR**

*Document créé pour la préparation du titre RNCP Technicien Supérieur Systèmes et Réseaux*

*Janvier 2026*
