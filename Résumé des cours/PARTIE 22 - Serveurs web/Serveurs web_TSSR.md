# Les serveurs Web

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Les serveurs Web - Publier sites et applications web  
**Date** : Janvier 2026  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Le Web|Le Web]]
   - [[#Définition et origines|Définition et origines]]
   - [[#Les trois composants essentiels|Les trois composants essentiels]]
3. [[#Le protocole HTTP|Le protocole HTTP]]
   - [[#HTTP - Fonctionnement|HTTP - Fonctionnement]]
   - [[#HTTPS - Sécurisation|HTTPS - Sécurisation]]
   - [[#Les URL|Les URL]]
   - [[#Requêtes HTTP|Requêtes HTTP]]
   - [[#Réponses HTTP|Réponses HTTP]]
4. [[#Clients HTTP|Clients HTTP]]
   - [[#Les navigateurs web|Les navigateurs web]]
   - [[#Autres clients HTTP|Autres clients HTTP]]
5. [[#Serveurs Web|Serveurs Web]]
   - [[#Rôle d'un serveur HTTP|Rôle d'un serveur HTTP]]
   - [[#Backend web|Backend web]]
   - [[#Virtualisation - Virtual Hosts|Virtualisation - Virtual Hosts]]
   - [[#Exemples de serveurs HTTP|Exemples de serveurs HTTP]]
6. [[#Serveurs Proxy|Serveurs Proxy]]
   - [[#Proxy - Définition|Proxy - Définition]]
   - [[#Proxy HTTP|Proxy HTTP]]
   - [[#Reverse Proxy|Reverse Proxy]]
7. [[#Points clés à retenir|Points clés à retenir]]
8. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Les serveurs web constituent un élément fondamental de l'infrastructure Internet moderne. Ils permettent de publier et de servir des sites web ainsi que des applications web accessibles via le protocole HTTP/HTTPS. Ce document couvre les concepts essentiels du web, le fonctionnement des serveurs HTTP, et le rôle des serveurs proxy dans l'architecture web.

### Pourquoi étudier les serveurs web ?

En tant que **TSSR**, tu dois :
- Comprendre l'architecture client-serveur du web
- Savoir installer et configurer des serveurs web (Apache, Nginx)
- Maîtriser les concepts de sécurité web (HTTPS, certificats)
- Connaître le rôle des proxy et reverse proxy dans l'infrastructure

---

## Le Web

### Définition et origines

> [!quote] Définition officielle
> Le **Web** (World Wide Web) est un système de publication hypertexte fonctionnant sur les réseaux IP.

> [!info] Historique
> - **Création** : Début des années 1990
> - **Inventeurs** : Tim Berners-Lee et Robert Cailliau
> - **Évolution** : D'un simple système de publication à une plateforme majeure de développement d'applications

### Les trois composants essentiels

Le web repose sur **trois piliers fondamentaux** :

1. **HTTP** (Hypertext Transfer Protocol) - Le protocole de communication
2. **URL** (Uniform Resource Locator) - Le système d'adressage
3. **HTML** (Hypertext Markup Language) - Le langage de structuration

> [!important] Concept clé
> Ces trois composants travaillent ensemble pour permettre l'accès et l'affichage de ressources web. Sans l'un d'eux, le web tel qu'on le connaît ne pourrait pas fonctionner.

---

## Le protocole HTTP

### HTTP - Fonctionnement

> [!quote] Définition
> **HTTP** (Hypertext Transfer Protocol) est un protocole de communication client-serveur utilisé pour transférer des ressources web.

#### Caractéristiques principales

| Caractéristique | Description |
|-----------------|-------------|
| **Architecture** | Client-Serveur |
| **Versions** | HTTP/1.1, HTTP/2, HTTP/3 (rare) |
| **Transport** | Basé sur TCP (HTTP/3 basé sur QUIC) |
| **Port standard** | 80 (non sécurisé) |
| **Nature** | Sans état (stateless) - chaque requête est indépendante |
| **Mode** | Requête → Réponse |

#### Structure des messages HTTP

> [!note] Format des messages
> Les messages HTTP (requêtes et réponses) sont des **messages texte** structurés en trois parties :
> 1. **Une ligne de démarrage** - Identifie le type et la nature du message
> 2. **Des entêtes** (headers) - Métadonnées sur la requête/réponse
> 3. **Un corps** (body) - Contenu optionnel (données, ressource)

> [!tip] Ressource complémentaire
> Pour approfondir HTTP : [HTTP sur MDN](https://developer.mozilla.org/fr/docs/Web/HTTP)

---

### HTTPS - Sécurisation

> [!quote] Définition
> **HTTPS** = HTTP over TLS (Transport Layer Security)  
> Version sécurisée de HTTP utilisant le chiffrement TLS sur le port TCP 443

#### Avantages de HTTPS

> [!success] Sécurité apportée
> - **Confidentialité** : Chiffrement des données échangées
> - **Authentification** : Vérification de l'identité du serveur par certificat X.509
> - **Intégrité** : Protection contre la modification des données en transit

#### Certificats et autorités de certification

> [!info] Certificats X.509
> - Nécessitent généralement une **Autorité de Certification (AC) publique**
> - Exemple populaire : **Let's Encrypt** (certificats gratuits)
> - Permettent au navigateur de vérifier l'identité du serveur

#### Mécanismes de sécurité complémentaires

> [!note] Extensions de sécurité HTTPS
> - **HSTS** (HTTP Strict Transport Security) - Force l'utilisation de HTTPS
> - **HPKP** (HTTP Public Key Pinning) - Épinglage des clés publiques
> - **CSP** (Content Security Policy) - Politique de sécurité du contenu

> [!important] Indispensable
> HTTPS est aujourd'hui **indispensable à tout serveur web** en production pour garantir la sécurité et la confiance des utilisateurs.

---

### Les URL

> [!quote] Définition
> **URL** (Uniform Resource Locator) est un identifiant unique permettant de localiser une ressource sur le web.

#### Structure d'une URL

```
schéma://autorité/chemin?requête#fragment
```

> [!example] Exemple d'URL complète
> ```
> https://utilisateur:motdepasse@www.exemple.fr:8080/dossier/page.html?id=123&lang=fr#section2
> ```

#### Décomposition des composants

| Composant | Description | Exemple |
|-----------|-------------|---------|
| **Schéma** | Protocole utilisé | `https`, `http`, `ftp` |
| **Autorité** | `[user:pass@]` nom de domaine ou IP `[:port]` | `www.exemple.fr:8080` |
| **Chemin** | Arborescence de fichiers sur le serveur | `/dossier/page.html` |
| **Requête** | Paramètres sous forme `clé=valeur` (séparés par `&`) | `?id=123&lang=fr` |
| **Fragment** | Ancre vers une partie spécifique de la page | `#section2` |

> [!note] Détails techniques
> - L'**autorité** peut contenir des identifiants utilisateur (rare aujourd'hui)
> - Le **port** est optionnel (80 par défaut pour HTTP, 443 pour HTTPS)
> - La **requête** permet de passer des paramètres au serveur
> - Le **fragment** est traité côté client (navigateur) uniquement

> [!tip] Ressource complémentaire
> Pour approfondir les URL : [Les URL sur MDN](https://developer.mozilla.org/fr/docs/Learn/Common_questions/What_is_a_URL)

---

### Requêtes HTTP

> [!quote] Définition
> Une **requête HTTP** est un message envoyé par un client vers un serveur pour demander une ressource ou effectuer une action.

#### Structure d'une requête HTTP

> [!info] Composition
> Une requête HTTP contient obligatoirement :

**1. Ligne de requête** (première ligne)
- **Méthode HTTP** : Nature de la demande (GET, POST, PUT, DELETE...)
- **URL** : Chemin de la ressource (peut être partielle)
- **Version HTTP** : HTTP/1.1, HTTP/2...

**2. Entêtes (headers)** - Un par ligne
- `Host` : Nom de domaine destinataire (obligatoire en HTTP/1.1)
- `Cookie` : Cookies envoyés au serveur
- `User-Agent` : Identification du client
- Autres paramètres...

**3. Corps (body)** - Optionnel
- Données envoyées au serveur (formulaires, fichiers...)

> [!example] Exemple de requête HTTP
> ```http
> GET /index.html HTTP/1.1
> Host: www.exemple.fr
> User-Agent: Mozilla/5.0
> Accept: text/html
> Cookie: session=abc123
> 
> ```

#### Principales méthodes HTTP

| Méthode | Usage | Corps |
|---------|-------|-------|
| **GET** | Récupérer une ressource | Non |
| **POST** | Envoyer des données | Oui |
| **PUT** | Mettre à jour une ressource | Oui |
| **DELETE** | Supprimer une ressource | Non |
| **HEAD** | Récupérer uniquement les entêtes | Non |
| **PATCH** | Modification partielle | Oui |

> [!warning] Piège courant
> L'entête `Host` est **obligatoire** en HTTP/1.1, car elle permet au serveur de savoir quel site virtuel (VirtualHost) doit traiter la requête.

---

### Réponses HTTP

> [!quote] Définition
> Une **réponse HTTP** est le message renvoyé par le serveur en réaction à une requête client.

#### Structure d'une réponse HTTP

> [!info] Composition
> Une réponse HTTP contient :

**1. Ligne de statut**
- **Version HTTP** : HTTP/1.1, HTTP/2...
- **Code de statut** : Code numérique (200, 404, 500...)
- **Description textuelle** : Explication du code

**2. Entêtes (headers)**
- `Content-Type` : Type de contenu (text/html, application/json...)
- `Content-Encoding` : Encodage du contenu (gzip, deflate...)
- `Cache-Control` : Directives de mise en cache
- `Server` : Signature du serveur web
- Autres métadonnées...

**3. Corps (body)**
- La ressource demandée (page HTML, image, données JSON...)

> [!example] Exemple de réponse HTTP
> ```http
> HTTP/1.1 200 OK
> Content-Type: text/html; charset=UTF-8
> Content-Length: 1234
> Server: Apache/2.4.41
> Cache-Control: max-age=3600
> 
> <!DOCTYPE html>
> <html>
> ...
> </html>
> ```

#### Codes de statut HTTP

> [!note] Catégories de codes de statut

| Catégorie | Code | Signification | Exemples |
|-----------|------|---------------|----------|
| **1xx** | Informatif | Requête reçue, traitement en cours | 100 Continue |
| **2xx** | Succès | Requête réussie | 200 OK, 201 Created |
| **3xx** | Redirection | Action supplémentaire nécessaire | 301 Moved, 302 Found, 304 Not Modified |
| **4xx** | Erreur client | Problème avec la requête | 400 Bad Request, 403 Forbidden, 404 Not Found |
| **5xx** | Erreur serveur | Le serveur a échoué | 500 Internal Error, 503 Service Unavailable |

> [!tip] Codes les plus courants
> - **200 OK** : Tout va bien
> - **301 Moved Permanently** : Redirection permanente
> - **302 Found** : Redirection temporaire
> - **304 Not Modified** : Ressource en cache toujours valide
> - **403 Forbidden** : Accès interdit
> - **404 Not Found** : Ressource introuvable
> - **500 Internal Server Error** : Erreur serveur
> - **503 Service Unavailable** : Service temporairement indisponible

---

## Clients HTTP

### Les navigateurs web

> [!quote] Définition
> Un **navigateur web** est un logiciel client HTTP capable d'envoyer des requêtes, interpréter les réponses et afficher les contenus web.

#### Fonctionnalités d'un navigateur

> [!info] Rôles multiples
> Un navigateur web moderne est bien plus qu'un simple client HTTP :
> 
> 1. **Client HTTP** - Émet des requêtes et reçoit des réponses
> 2. **Interprète HTML/CSS** - Affiche et met en forme le contenu
> 3. **Moteur JavaScript** - Exécute du code côté client
> 4. **Environnement d'exécution** - Fournit des API web modernes

#### APIs web disponibles

> [!note] Principales API navigateur
> - **DOM** (Document Object Model) - Manipulation de la structure HTML
> - **Fetch API** - Requêtes HTTP asynchrones
> - **Canvas** - Dessin 2D
> - **WebGL** - Rendu 3D
> - **Web Storage** - Stockage local (localStorage, sessionStorage)
> - **WebSocket** - Communication bidirectionnelle en temps réel

#### Déclenchement d'une requête HTTP

> [!example] Sources de requêtes HTTP
> Le navigateur émet une requête HTTP sur la base d'une URL provenant de :
> - La **barre d'adresse** (saisie utilisateur)
> - Un **lien hypertexte** (`<a href="...">`)
> - Un **formulaire** (`<form>`)
> - Une **ressource externe** (image, script, CSS)
> - Code **JavaScript** (fetch, XMLHttpRequest)

---

### Autres clients HTTP

> [!info] Clients HTTP en ligne de commande et programmatiques
> Les navigateurs ne sont pas les seuls clients HTTP. Il existe de nombreux outils pour interagir avec des serveurs web.

#### Clients en ligne de commande

| Outil | Usage principal | Caractéristiques |
|-------|----------------|------------------|
| **curl** | Transfert de données via URL | Très versatile, scripting, debugging |
| **wget** | Téléchargement de fichiers | Récursif, reprise de téléchargement |
| **HTTPie** | Client HTTP convivial | Syntaxe simple, sortie colorée |

> [!example] Exemple avec curl
> ```bash
> # Requête GET simple
> curl https://www.exemple.fr
> 
> # Voir les entêtes HTTP
> curl -I https://www.exemple.fr
> 
> # Requête POST avec données
> curl -X POST -d "param=valeur" https://api.exemple.fr
> ```

#### Clients programmatiques

> [!note] Bibliothèques HTTP
> - **Python** : requests, urllib, httpx
> - **JavaScript/Node.js** : axios, node-fetch, got
> - **Java** : HttpClient, OkHttp
> - **PHP** : cURL, Guzzle

> [!tip] Cas d'usage
> Ces clients sont utilisés pour :
> - Automatiser des tests
> - Consommer des API
> - Scraper du contenu web
> - Monitoring de serveurs
> - Intégrations applicatives

---

## Serveurs Web

### Rôle d'un serveur HTTP

> [!quote] Définition
> Un **serveur HTTP** est un logiciel qui écoute sur les ports 80 (HTTP) et 443 (HTTPS) et répond aux requêtes HTTP des clients.

#### Fonctionnement de base

> [!info] Cycle de traitement
> 1. **Écoute** sur les ports TCP 80 et/ou 443
> 2. **Réception** d'une requête HTTP d'un client
> 3. **Traitement** de la requête (analyse, vérification)
> 4. **Génération** d'une réponse HTTP appropriée
> 5. **Envoi** de la réponse au client

#### Cas classique : serveur de fichiers statiques

> [!example] Fonctionnement simple
> Dans sa forme la plus basique, un serveur web :
> - Possède une **racine documentaire** (ex: `/var/www/html`)
> - Reçoit une requête avec un chemin (ex: `/images/logo.png`)
> - **Compose le chemin complet** : racine + chemin demandé
> - **Envoie le fichier** correspondant dans la réponse

```
Requête : GET /images/logo.png
Racine : /var/www/html
Fichier servi : /var/www/html/images/logo.png
```

> [!note] Fichier par défaut
> Si le chemin pointe vers un répertoire, le serveur cherche généralement un fichier par défaut :
> - `index.html`
> - `index.htm`
> - `default.html`

---

### Backend web

> [!quote] Définition
> Un **backend web** est un programme qui génère dynamiquement le contenu des réponses HTTP, au lieu de simplement servir des fichiers statiques.

#### Principe de fonctionnement

> [!info] Génération dynamique
> Au lieu de lire un fichier existant, le serveur :
> 1. **Exécute** un programme/script
> 2. Le programme **traite** la requête (accès BDD, calculs, logique métier...)
> 3. Le programme **génère** le HTML/JSON/XML à renvoyer
> 4. La réponse est **envoyée** au client en temps réel

#### Principales technologies backend

| Langage/Framework | Serveur/Runtime | Cas d'usage typique |
|-------------------|-----------------|---------------------|
| **PHP** | Apache avec mod_php / PHP-FPM | Sites web traditionnels, WordPress, CMS |
| **Java** | Tomcat, JBoss, Spring Boot | Applications d'entreprise (Spring, JEE) |
| **JavaScript** | Node.js | Applications web modernes, API REST |
| **Python** | Gunicorn, uWSGI | Django, Flask, FastAPI |
| **Ruby** | Puma, Unicorn | Rails, applications web |
| **C# / .NET** | IIS, Kestrel | Applications Microsoft |
| **Go** | Serveur intégré | Microservices, APIs performantes |

> [!important] Évolution vers les applications web
> La tendance moderne est au développement d'**applications web complètes** plutôt que de simples sites statiques :
> - **SPAs** (Single Page Applications) avec React, Vue, Angular
> - **APIs RESTful** ou **GraphQL**
> - Architecture **microservices**
> - Applications **temps réel** avec WebSocket

> [!tip] Combo classique TSSR
> Configuration typique :
> - **Nginx ou Apache** : Serveur web frontal (reverse proxy, fichiers statiques)
> - **PHP-FPM / Node.js / Python** : Backend applicatif
> - **MySQL / PostgreSQL** : Base de données
> - **Redis / Memcached** : Cache

---

### Virtualisation - Virtual Hosts

> [!quote] Définition
> La **virtualisation de serveurs web** permet d'héberger plusieurs sites web sur un même serveur physique, chacun avec sa propre configuration.

#### Concepts clés

> [!info] Terminologie selon le serveur
> - **Apache** : Virtual Host (VirtualHost)
> - **Nginx** : Server Block (virtual server)
> - **Principe** : Identique, seule la terminologie change

#### Mécanismes de virtualisation

> [!note] Critères de différenciation des sites
> Un serveur web peut router les requêtes vers différents sites selon :

**1. Le nom d'hôte** (méthode la plus courante)
- Basé sur l'entête `Host:` de la requête HTTP
- Permet d'héberger plusieurs domaines sur une seule IP
- Exemple : `www.site1.fr` et `www.site2.fr` sur 192.168.1.100

**2. L'adresse IP du serveur**
- Chaque site a sa propre adresse IP
- Nécessite plusieurs IPs configurées sur le serveur
- Utilisé pour des raisons de sécurité ou SSL (ancien)

**3. Le port**
- Moins courant
- Exemple : site1 sur port 80, site2 sur port 8080

> [!example] Configuration Apache VirtualHost
> ```apache
> <VirtualHost *:80>
>     ServerName www.site1.fr
>     DocumentRoot /var/www/site1
>     ErrorLog /var/log/apache2/site1-error.log
> </VirtualHost>
> 
> <VirtualHost *:80>
>     ServerName www.site2.fr
>     DocumentRoot /var/www/site2
>     ErrorLog /var/log/apache2/site2-error.log
> </VirtualHost>
> ```

> [!example] Configuration Nginx Server Block
> ```nginx
> server {
>     listen 80;
>     server_name www.site1.fr;
>     root /var/www/site1;
>     error_log /var/log/nginx/site1-error.log;
> }
> 
> server {
>     listen 80;
>     server_name www.site2.fr;
>     root /var/www/site2;
>     error_log /var/log/nginx/site2-error.log;
> }
> ```

> [!important] Avantages de la virtualisation
> - **Économie** : Un seul serveur physique pour plusieurs sites
> - **Flexibilité** : Configurations indépendantes par site
> - **Isolation** : Séparation des logs, des erreurs, des ressources
> - **Facilité** : Gestion simplifiée de multiples projets

> [!warning] Attention
> Pour HTTPS, chaque VirtualHost nécessite son propre certificat SSL/TLS. SNI (Server Name Indication) permet d'avoir plusieurs certificats sur une même IP.

---

### Exemples de serveurs HTTP

> [!info] Serveurs web populaires
> Il existe de nombreux serveurs HTTP, chacun avec ses spécificités.

#### Serveurs principaux

| Serveur | Type | Usage principal | Parts de marché |
|---------|------|----------------|-----------------|
| **Apache HTTP Server** | Modulaire | Historique, très flexible | ~30% |
| **Nginx** | Événementiel | Haute performance, reverse proxy | ~35% |
| **Microsoft IIS** | Windows | Environnements .NET/Windows | ~10% |
| **LiteSpeed** | Commercial | Performance, cache intégré | ~5% |
| **Caddy** | Moderne | HTTPS automatique, config simple | Croissant |

#### Serveurs d'application / Runtime

| Serveur | Langage | Caractéristiques |
|---------|---------|------------------|
| **Apache Tomcat** | Java | Serveur de servlets Java |
| **Node.js** | JavaScript | Runtime JavaScript backend |
| **Gunicorn** | Python | Serveur WSGI pour Django/Flask |
| **Puma** | Ruby | Serveur pour Ruby on Rails |
| **Kestrel** | .NET | Serveur web ASP.NET Core |

> [!success] Choix selon le contexte TSSR
> - **Apache** : Polyvalent, grande compatibilité, bonne documentation française
> - **Nginx** : Meilleure performance, idéal comme reverse proxy, configuration plus simple
> - **Combo** : Nginx en frontal + Apache ou backend applicatif

> [!tip] Points de comparaison Apache vs Nginx
> 
> **Apache** :
> - Architecture multi-processus/threads
> - Configuration par répertoire (.htaccess)
> - Modules dynamiques nombreux
> - Consommation RAM plus élevée
> 
> **Nginx** :
> - Architecture événementielle asynchrone
> - Meilleure gestion de la concurrence
> - Excellent pour servir fichiers statiques
> - Reverse proxy et load balancer natif
> - Configuration centralisée

---

## Serveurs Proxy

### Proxy - Définition

> [!quote] Définition
> Un **serveur proxy** (ou serveur mandataire) est un intermédiaire placé entre le client et le serveur, agissant comme passerelle au niveau de la couche 7 (application) du modèle OSI.

#### Rôle général

> [!info] Fonction d'intermédiaire
> Le proxy :
> - **Reçoit** les requêtes du client
> - **Transmet** ces requêtes au serveur (éventuellement modifiées)
> - **Reçoit** les réponses du serveur
> - **Transmet** ces réponses au client (éventuellement modifiées)

#### Types de proxy

> [!note] Classification des proxy

| Type | Description | Cas d'usage |
|------|-------------|-------------|
| **Proxy forward** | Mandataire pour le client | Navigation anonyme, filtrage sortant |
| **Proxy inverse** | Mandataire pour le serveur | Répartition de charge, cache |
| **Proxy transparent** | Invisible pour le client | Portail captif, filtrage réseau |
| **Proxy générique (SOCKS)** | Protocole agnostique | Tunneling, contournement |
| **Proxy spécifique** | Pour un protocole donné | HTTP, FTP, SMTP... |

#### Services fournis par un proxy

> [!success] Fonctionnalités multiples
> - **Anonymat** : Masquer l'adresse IP du client
> - **Filtrage** : Bloquer l'accès à certains contenus
> - **Cache** : Accélérer l'accès aux ressources fréquentes
> - **Journalisation** : Tracer les connexions (logging)
> - **Sécurité** : Protection contre certaines attaques
> - **Modification** : Transformation à la volée (compression, traduction)
> - **Contrôle d'accès** : Authentification centralisée

> [!tip] Ressource complémentaire
> Pour approfondir : [Proxy sur Wikipedia](https://fr.wikipedia.org/wiki/Proxy)

---

### Proxy HTTP

> [!quote] Définition
> Un **proxy HTTP** est un proxy spécialisé dans le protocole HTTP/HTTPS, offrant des fonctionnalités spécifiques au web.

#### Fonctionnalités principales

> [!info] Services d'un proxy HTTP

**1. Cache (mise en mémoire)**
- Conserve les contenus fréquemment demandés
- Réduit la bande passante et améliore les temps de réponse
- Particulièrement efficace pour les fichiers statiques (images, CSS, JS)

**2. Filtrage de contenu**
- Blocage de sites web (listes noires/blanches)
- Filtrage par catégorie (réseaux sociaux, streaming...)
- Protection contre les malwares et sites malveillants
- Contrôle parental ou d'entreprise

**3. Journalisation et surveillance**
- Enregistrement des sites visités (logging)
- Statistiques d'utilisation
- Détection d'anomalies
- Écoute clandestine (eavesdropping) - usage délicat

**4. Modification à la volée**
- Compression du trafic
- Traduction automatique
- Injection de publicités
- Optimisation mobile

#### Problématique avec HTTPS

> [!warning] Incompatibilité majeure avec HTTPS
> **Le problème** :
> - HTTPS crée un **tunnel chiffré** entre client et serveur
> - Le proxy ne peut pas lire le contenu chiffré
> - Impossible de filtrer, mettre en cache ou modifier sans déchiffrer
> 
> **La "solution"** :
> - Le proxy doit **casser le tunnel TLS** (TLS termination)
> - Équivaut à une **attaque de l'intercepteur** (Man-in-the-Middle - MitM)
> - Nécessite d'installer un **certificat racine** du proxy sur les clients
> - **Pose de graves problèmes de sécurité et de confidentialité**

> [!important] Implications
> - Déchiffrement/rechiffrement du trafic par le proxy
> - Le proxy voit **tout le trafic en clair**
> - Nécessite la confiance totale envers l'organisation gérant le proxy
> - Peut être détecté et bloqué par certains sites (certificate pinning)

#### Exemple de proxy HTTP

> [!example] Squid
> **Squid** est l'un des proxies HTTP open source les plus populaires :
> - Cache HTTP performant
> - Filtrage par ACL (Access Control Lists)
> - Authentification (LDAP, AD, NTLM...)
> - Support HTTP/HTTPS/FTP
> - Très utilisé en entreprise et chez les FAI

```bash
# Installation sur Debian/Ubuntu
sudo apt install squid

# Configuration de base
sudo nano /etc/squid/squid.conf
```

> [!tip] Cas d'usage proxy HTTP
> - **Entreprise** : Contrôler et surveiller la navigation des employés
> - **Éducation** : Filtrer le contenu inapproprié
> - **FAI** : Réduire la bande passante par mise en cache
> - **Tests** : Analyser le trafic HTTP d'une application

---

### Reverse Proxy

> [!quote] Définition
> Un **reverse proxy** (proxy inverse) est un serveur intermédiaire placé devant un ou plusieurs serveurs web, agissant comme frontal pour gérer les connexions entrantes.

#### Fonctionnement

> [!info] Principe
> Contrairement au proxy classique (qui agit pour le client), le reverse proxy :
> - Est mandataire **pour le serveur** (pas pour le client)
> - **Reçoit** les requêtes des clients Internet
> - **Redirige** le trafic vers les serveurs backend appropriés
> - **Retourne** la réponse au client

```
Client → Internet → Reverse Proxy → Serveur Backend
                         ↓
                    Serveurs multiples
```

#### Fonctionnalités principales

> [!success] Services d'un reverse proxy

**1. Mémoire cache**
- Cache les réponses des serveurs backend
- Soulage les serveurs applicatifs
- Améliore les performances pour les clients

**2. Répartition de charge (Load Balancing)**
- Distribue les requêtes entre plusieurs serveurs
- Assure la haute disponibilité
- Permet la scalabilité horizontale
- Algorithmes : round-robin, least connections, IP hash...

**3. Sécurité**
- **Protection DDoS** : Absorbe les attaques
- **WAF** (Web Application Firewall) : Filtre les requêtes malveillantes
- **Masquage** : Cache la topologie interne du réseau
- **Terminaison SSL/TLS** : Gère le chiffrement pour les backends

**4. Autres services**
- Compression du trafic (gzip, brotli)
- Réécriture d'URL
- Authentification centralisée
- Monitoring et logging centralisé

#### Solutions de reverse proxy

> [!note] Principales solutions

| Solution | Type | Spécialités | Usage |
|----------|------|-------------|-------|
| **Apache** | Serveur web | mod_proxy, mod_proxy_balancer | Polyvalent, PHP |
| **Nginx** | Serveur web | Haute performance, événementiel | Très populaire, efficace |
| **Varnish** | Cache HTTP | Cache ultra-rapide | Sites à fort trafic |
| **HAProxy** | Load balancer | TCP/HTTP, très performant | Répartition de charge |
| **Traefik** | Cloud-native | Auto-configuration, Docker/K8s | Microservices, conteneurs |
| **Caddy** | Moderne | HTTPS auto, config simple | Projets modernes |
| **Squid** | Proxy | Cache, aussi forward proxy | Cache, CDN |

> [!example] Configuration Nginx en reverse proxy
> ```nginx
> upstream backend {
>     server backend1.example.com:8080;
>     server backend2.example.com:8080;
>     server backend3.example.com:8080;
> }
> 
> server {
>     listen 80;
>     server_name www.example.com;
> 
>     location / {
>         proxy_pass http://backend;
>         proxy_set_header Host $host;
>         proxy_set_header X-Real-IP $remote_addr;
>         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
>     }
> }
> ```

> [!important] Architecture typique moderne
> ```
> Internet
>    ↓
> Reverse Proxy (Nginx/HAProxy)
>    ├─→ Serveur Web 1 (Node.js)
>    ├─→ Serveur Web 2 (Node.js)
>    └─→ Serveur Web 3 (Node.js)
>         ↓
>    Base de données
> ```

> [!tip] Pourquoi utiliser un reverse proxy ?
> - **Performance** : Cache et compression
> - **Scalabilité** : Ajout facile de serveurs backend
> - **Sécurité** : Point d'entrée unique à sécuriser
> - **Maintenance** : Mise à jour des backends sans interruption
> - **SSL/TLS** : Gestion centralisée des certificats
> - **Logs** : Centralisation de la journalisation

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Le Web et HTTP

- Le **Web** repose sur 3 piliers : HTTP (protocole), URL (adressage), HTML (structuration)
- **HTTP** est un protocole client-serveur, sans état, basé sur TCP port 80
- **HTTPS** (port 443) sécurise HTTP avec TLS : confidentialité, authentification, intégrité
- **Indispensable** : Tout serveur web en production doit utiliser HTTPS

### URL et Messages HTTP

- Structure URL : `schéma://autorité/chemin?requête#fragment`
- **Requête HTTP** : Méthode + URL + Version + Entêtes + Corps optionnel
- **Réponse HTTP** : Version + Code statut + Description + Entêtes + Corps
- Codes de statut : 2xx (succès), 3xx (redirection), 4xx (erreur client), 5xx (erreur serveur)

### Clients HTTP

- **Navigateurs** : Client HTTP + Interprète HTML/CSS + Moteur JS + APIs web
- **Clients CLI** : curl, wget, HTTPie (tests, automatisation, debugging)
- Multiples sources de requêtes : barre d'adresse, liens, formulaires, JavaScript

### Serveurs Web

- **Rôle** : Écouter sur ports 80/443, recevoir requêtes, envoyer réponses
- **Fichiers statiques** : Servir des fichiers depuis une racine documentaire
- **Backend** : Génération dynamique de contenu (PHP, Node.js, Python, Java...)
- **Virtual Hosts** : Héberger plusieurs sites sur un serveur (critère : nom d'hôte, IP, port)
- **Serveurs populaires** : Apache (modulaire), Nginx (performant), IIS (.NET), Tomcat (Java)

### Proxy

- **Proxy forward** : Intermédiaire pour le client (anonymat, filtrage, cache)
- **Proxy HTTP** : Cache, filtrage, journalisation, modification
- **Problème HTTPS** : Nécessite de casser le tunnel TLS (MitM)
- **Exemple** : Squid (proxy HTTP open source)

### Reverse Proxy

- **Rôle** : Frontal devant les serveurs backend (mandataire serveur)
- **Fonctions** : Cache, répartition de charge, sécurité, terminaison SSL
- **Solutions** : Nginx, HAProxy, Varnish, Traefik, Apache
- **Architecture moderne** : Reverse proxy → multiples backends → haute disponibilité

### Compétences TSSR essentielles

- Installer et configurer **Apache et Nginx**
- Créer des **Virtual Hosts** / Server Blocks
- Configurer **HTTPS avec certificats** (Let's Encrypt)
- Mettre en place un **reverse proxy** avec répartition de charge
- Comprendre les **codes de statut HTTP** et savoir les diagnostiquer
- Configurer un **proxy HTTP** (Squid) pour filtrage/cache
- Analyser le trafic HTTP avec **curl** et les outils de debugging

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **HTTP** | Hypertext Transfer Protocol - Protocole de communication client-serveur pour le transfert de ressources web |
| **HTTPS** | HTTP Secure - Version sécurisée de HTTP utilisant TLS/SSL pour le chiffrement |
| **TLS** | Transport Layer Security - Protocole de sécurisation des communications réseau |
| **URL** | Uniform Resource Locator - Identifiant unique d'une ressource web |
| **Stateless** | Sans état - Chaque requête HTTP est indépendante, le serveur ne conserve pas d'information entre les requêtes |
| **Virtual Host** | Configuration permettant d'héberger plusieurs sites web sur un même serveur physique |
| **Server Block** | Équivalent Nginx du Virtual Host Apache |
| **Backend** | Partie serveur d'une application web qui génère dynamiquement le contenu |
| **Frontend** | Partie client d'une application web (HTML/CSS/JS) exécutée dans le navigateur |
| **Proxy** | Serveur intermédiaire entre client et serveur agissant comme mandataire |
| **Reverse Proxy** | Proxy placé devant les serveurs web pour gérer les connexions entrantes |
| **Load Balancing** | Répartition de charge - Distribution des requêtes entre plusieurs serveurs |
| **Cache HTTP** | Stockage temporaire de ressources web pour améliorer les performances |
| **Certificat X.509** | Standard de certificat numérique utilisé pour HTTPS |
| **AC** | Autorité de Certification - Organisme émettant des certificats numériques de confiance |
| **SNI** | Server Name Indication - Extension TLS permettant plusieurs certificats sur une même IP |
| **MitM** | Man-in-the-Middle - Attaque où un tiers intercepte les communications |
| **HSTS** | HTTP Strict Transport Security - Force l'utilisation de HTTPS |
| **CSP** | Content Security Policy - Politique de sécurité limitant les sources de contenu |
| **WAF** | Web Application Firewall - Pare-feu applicatif web |
| **DOM** | Document Object Model - Représentation en mémoire de la structure HTML |
| **API** | Application Programming Interface - Interface de programmation |
| **REST** | Representational State Transfer - Style d'architecture pour APIs web |
| **SPA** | Single Page Application - Application web monopage avec JavaScript |
| **WebSocket** | Protocole de communication bidirectionnelle en temps réel sur HTTP |
| **CDN** | Content Delivery Network - Réseau de distribution de contenu géographiquement distribué |
| **DDoS** | Distributed Denial of Service - Attaque par déni de service distribué |

---

> [!success] Document de révision complet
> Ce document couvre l'ensemble des concepts essentiels sur les serveurs web pour la préparation du titre RNCP TSSR. N'hésite pas à approfondir avec les ressources MDN et la documentation officielle des serveurs (Apache, Nginx).

> [!tip] Pour aller plus loin
> - Pratique sur VM : Installer Apache et Nginx
> - Configurer plusieurs Virtual Hosts
> - Mettre en place HTTPS avec Let's Encrypt
> - Tester un reverse proxy avec Nginx
> - Analyser le trafic HTTP avec les DevTools navigateur et curl

---

**Bon courage pour tes révisions ! 📚✨**
