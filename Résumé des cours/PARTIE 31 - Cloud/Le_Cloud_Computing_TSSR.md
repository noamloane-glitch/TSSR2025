# Le Cloud Computing

## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)

**Sujet** : Cloud Computing - Concepts, technologies et mise en œuvre

**Date** : Février 2026

**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
   - [[#Le Cloud Computing|Le Cloud Computing]]
   - [[#Pourquoi le cloud ?|Pourquoi le cloud ?]]
   - [[#Cloud dans la vie quotidienne|Cloud dans la vie quotidienne]]
   - [[#Termes clés|Termes clés]]
   - [[#La fondation technique|La fondation technique]]
2. [[#L'externalisation|L'externalisation]]
   - [[#Qu'est-ce que l'externalisation ?|Qu'est-ce que l'externalisation ?]]
   - [[#Défis de l'externalisation dans le Cloud|Défis de l'externalisation]]
   - [[#Sécurité et conformité|Sécurité et conformité]]
3. [[#Modèles de déploiement et services|Modèles de déploiement et services]]
   - [[#Modèles de déploiement Cloud|Modèles de déploiement]]
   - [[#Diversité des services cloud|Diversité des services]]
   - [[#Le stockage|Le stockage]]
   - [[#Conteneurisation|Conteneurisation]]
4. [[#Les modèles économiques|Les modèles économiques]]
   - [[#Forfait|Forfait]]
   - [[#Paiement à la consommation|Paiement à la consommation]]
   - [[#Optimisation des coûts|Optimisation des coûts]]
   - [[#Cloud bureautiques|Cloud bureautiques]]
5. [[#Interfaces d'accès au Cloud|Interfaces d'accès]]
   - [[#Shell SSH|Shell SSH]]
   - [[#Interface Web|Interface Web]]
   - [[#API|API]]
   - [[#Diagnostic et troubleshooting|Diagnostic et troubleshooting]]
6. [[#Usages, avantages et inconvénients|Usages, avantages et inconvénients]]
   - [[#Utilisation typique|Utilisation typique]]
   - [[#Avantages du Cloud|Avantages]]
   - [[#Inconvénients du Cloud|Inconvénients]]
   - [[#Sécuriser l'accès aux services|Sécuriser l'accès]]
7. [[#Briques techniques du Cloud|Briques techniques]]
   - [[#Hyperviseurs|Hyperviseurs]]
   - [[#Orchestration|Orchestration]]
   - [[#Infrastructure as Code (IaC)|Infrastructure as Code]]
   - [[#Architecture réseau|Architecture réseau]]
   - [[#Gestion de ressources|Gestion de ressources]]
8. [[#Points clés à retenir|Points clés à retenir]]
9. [[#Glossaire technique|Glossaire technique]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Le cloud computing révolutionne la façon dont les entreprises gèrent leurs infrastructures informatiques. En tant que TSSR, tu seras amené à déployer, gérer et sécuriser des ressources cloud. Ce cours couvre les concepts fondamentaux, les modèles de services (IaaS, PaaS, SaaS), les architectures de déploiement, et les bonnes pratiques de gestion du cloud.

### Pourquoi étudier le cloud computing ?

En tant que **TSSR**, tu dois maîtriser le cloud computing car il est devenu **incontournable** dans le paysage informatique moderne. Les compétences cloud sont essentielles pour :
- Déployer et maintenir des infrastructures modernes
- Optimiser les coûts informatiques
- Assurer la scalabilité et la disponibilité des services
- Gérer la sécurité dans des environnements hybrides

---

## Le Cloud Computing

> [!quote] Définition officielle
> Le **cloud computing** consiste à fournir des services informatiques sur le réseau. Les Systèmes d'Information (SI) comprennent l'infrastructure informatique courante, telle que les machines virtuelles, le stockage, les bases de données et le réseau.

---

## Pourquoi le cloud ?

> [!important] Les motivations du cloud
> Le cloud computing apporte des avantages stratégiques majeurs qui expliquent son adoption massive.

### Bénéfices principaux

| Avantage | Explication |
|----------|-------------|
| **Flexibilité et évolutivité** | Ajustement rapide des ressources selon les besoins |
| **Réduction des coûts d'infrastructure** | Pas d'investissement initial en matériel, paiement à l'usage |
| **Accès amélioré aux ressources** | Disponibilité mondiale 24/7 |
| **Collaboration et accessibilité accrues** | Travail collaboratif facilité, accès de n'importe où |

> [!example] Exemple concret
> Une startup peut démarrer avec un serveur à 10€/mois et scaler jusqu'à des centaines de serveurs en quelques clics quand son activité explose, sans jamais acheter de matériel physique.

---

## Cloud dans la vie quotidienne

> [!info] Le cloud est partout
> Nous utilisons quotidiennement des services cloud, souvent sans nous en rendre compte.

**Exemples d'utilisation courante :**
- **Stockage** de photos et documents : Google Drive, iCloud, OneDrive, Dropbox
- **Services de streaming** : Netflix, Spotify, Disney+, YouTube
- **Solutions de travail collaboratif** : Office 365, Google Workspace, Notion, Slack

> [!tip] Réflexion
> Tout service accessible via Internet sans installation locale est probablement un service cloud !

---

## Termes clés

> [!note] Vocabulaire essentiel du Cloud Computing
> La compréhension de ces termes est fondamentale pour travailler dans le cloud.

### Les modèles de services

| Acronyme | Signification | Description |
|----------|--------------|-------------|
| **IaaS** | Infrastructure as a Service | Fournit des ressources informatiques virtualisées |
| **PaaS** | Platform as a Service | Environnement de développement et déploiement |
| **SaaS** | Software as a Service | Logiciels disponibles via abonnement en ligne |

### Les modèles de déploiement

| Type | Caractéristique |
|------|-----------------|
| **On-premises** | Infrastructure hébergée en interne (local) |
| **Cloud Privé** | Ressources cloud dédiées à une organisation |
| **Cloud Public** | Ressources partagées via Internet |
| **Cloud Hybride** | Combinaison de cloud public et privé |

---

## IaaS (Infrastructure as a Service)

> [!quote] Définition
> **Infrastructure as a Service** : Fournit des ressources informatiques virtualisées sur Internet.

**Exemples de ressources IaaS :**
- **Serveurs virtuels** (VMs - Virtual Machines)
- **Stockage** (Block storage, Object storage)
- **Réseaux virtuels** (VPC, VNet)
- **Équilibreurs de charge** (Load Balancers)

> [!example] Exemples de fournisseurs IaaS
> - **Amazon Web Services (AWS)** : EC2 (serveurs), S3 (stockage), VPC (réseau)
> - **Microsoft Azure** : Virtual Machines, Azure Storage, Virtual Network
> - **Google Cloud Platform (GCP)** : Compute Engine, Cloud Storage
> - **OVHcloud** : Public Cloud, VPS

> [!tip] Cas d'usage typique
> Héberger un serveur web Linux avec base de données MySQL sans acheter de matériel physique.

---

## PaaS (Platform as a Service)

> [!quote] Définition
> **Platform as a Service** : Offre un environnement de développement et de déploiement d'applications clé en main.

**Composants fournis :**
- **Outils de développement** et IDE en ligne
- **Bases de données** managées (SQL, NoSQL)
- **Runtime** (environnements d'exécution : Python, Node.js, Java, .NET)
- **Middleware** et services d'intégration

> [!example] Exemples de PaaS
> - **Heroku** : Déploiement d'applications web simplifié
> - **Google App Engine** : Plateforme pour applications web et mobiles
> - **Azure App Service** : Hébergement d'applications .NET, PHP, Node.js
> - **AWS Elastic Beanstalk** : Déploiement et gestion d'applications

> [!success] Avantage principal
> Les développeurs se concentrent sur le code applicatif sans gérer l'infrastructure sous-jacente (OS, patches, mise à l'échelle).

---

## SaaS (Software as a Service)

> [!quote] Définition
> **Software as a Service** : Logiciels disponibles via un abonnement en ligne, accessibles par navigateur ou application.

**Caractéristiques :**
- Aucune installation locale nécessaire
- Mises à jour automatiques
- Accessibilité multiplateforme
- Paiement par abonnement (mensuel/annuel)

> [!example] Exemples de SaaS
> - **Suite Adobe Creative Cloud** : Photoshop, Illustrator en ligne
> - **Microsoft 365** : Word, Excel, PowerPoint, Teams
> - **Google Workspace** : Gmail, Docs, Sheets, Meet
> - **Salesforce** : CRM (Customer Relationship Management)
> - **Slack, Trello, Asana** : Outils de collaboration

---

## Partage des responsabilités

> [!important] Modèle de responsabilité partagée
> La sécurité et la gestion dans le cloud sont partagées entre le fournisseur et le client. Le niveau de responsabilité varie selon le modèle de service.

### Tableau de partage des responsabilités

| Couche | On-Premises | IaaS | PaaS | SaaS |
|--------|-------------|------|------|------|
| **Application** | Client | Client | Client | **Fournisseur** |
| **Données** | Client | Client | Client | **Client/Fournisseur** |
| **Runtime/Middleware** | Client | Client | **Fournisseur** | **Fournisseur** |
| **OS** | Client | Client | **Fournisseur** | **Fournisseur** |
| **Virtualisation** | Client | **Fournisseur** | **Fournisseur** | **Fournisseur** |
| **Serveurs** | Client | **Fournisseur** | **Fournisseur** | **Fournisseur** |
| **Stockage** | Client | **Fournisseur** | **Fournisseur** | **Fournisseur** |
| **Réseau** | Client | **Fournisseur** | **Fournisseur** | **Fournisseur** |
| **Datacenter physique** | Client | **Fournisseur** | **Fournisseur** | **Fournisseur** |

> [!warning] Attention critique !
> Les **données en cloud requièrent TOUJOURS des sauvegardes** par le client, quel que soit le modèle (IaaS, PaaS, SaaS). Le fournisseur assure la disponibilité de l'infrastructure, mais pas la récupération de vos données en cas d'erreur humaine ou de suppression accidentelle.

> [!example] Exemples pratiques
> - **IaaS** : Le fournisseur gère le matériel et la virtualisation. Vous gérez l'OS, les applications et les données.
> - **PaaS** : Le fournisseur gère tout jusqu'au runtime. Vous gérez uniquement votre code et vos données.
> - **SaaS** : Le fournisseur gère tout. Vous gérez uniquement vos données et la configuration de l'application.

---

## La fondation technique

> [!info] La base technologique du cloud
> Le cloud computing repose principalement sur la **virtualisation**, l'**automatisation** et le **réseau**.

### Les hyperviseurs

> [!quote] Définition
> Les **hyperviseurs** constituent le socle de la virtualisation. Ils permettent de faire tourner plusieurs machines virtuelles sur un seul serveur physique.

**Types d'hyperviseurs :**

| Type | Nom | Caractéristiques | Cas d'usage |
|------|-----|------------------|-------------|
| **Type 1 (Bare Metal)** | ESXi, Hyper-V, Proxmox, KVM | S'exécute **directement sur le matériel** physique | Datacenters cloud (AWS, Azure, GCP) |
| **Type 2 (Hosted)** | VirtualBox, VMware Workstation | S'exécute **sur un OS hôte** | Tests, développement, démonstrations |

> [!success] Équation du Cloud
> **Le Cloud = Virtualisation + Automatisation + Réseau**

### Les instances cloud

> [!note] Qu'est-ce qu'une instance cloud ?
> Les **instances cloud** sont des **machines virtuelles** gérées automatiquement par le fournisseur cloud.

> [!example] Exemple concret
> **EC2 (AWS)** = Machine virtuelle Linux/Windows fonctionnant sur un hyperviseur propriétaire (Xen/Nitro d'Amazon).

> [!tip] Mnémotechnique
> **VAN** : **V**irtualisation + **A**utomatisation + **N**etwork = Cloud

---

## L'externalisation

### Qu'est-ce que l'externalisation ?

> [!quote] Définition
> L'**externalisation** est le processus par lequel les entreprises **délèguent** des processus métier ou des fonctions informatiques à des **prestataires de services tiers**.

> [!example] Exemple dans le contexte du cloud
> Externalisation du stockage de données et des serveurs vers AWS, Azure ou Google Cloud au lieu de maintenir un datacenter en interne.

**Avantages de l'externalisation :**
- Réduction des coûts d'infrastructure
- Concentration sur le cœur de métier
- Accès à des expertises spécialisées
- Scalabilité et flexibilité

---

## Défis de l'externalisation dans le Cloud

> [!warning] Points de vigilance critiques
> L'externalisation apporte des avantages mais aussi des défis spécifiques qu'il faut anticiper.

### Défis principaux

| Défi | Description | Solution |
|------|-------------|----------|
| **Gestion et surveillance** | Nécessité de surveiller les prestataires | SLA (Service Level Agreement), tableaux de bord |
| **Sécurité et conformité** | Protection des données externalisées | Chiffrement, audits réguliers, certifications |
| **Dépendance fournisseur** | Vendor lock-in (enfermement propriétaire) | Architecture multi-cloud, portabilité des données |
| **Perte de contrôle** | Moins de maîtrise sur l'infrastructure | Clauses contractuelles précises, monitoring |

> [!tip] Conseil TSSR
> Toujours négocier des **SLA (Service Level Agreements)** clairs avec des pénalités en cas de non-respect des engagements de disponibilité.

---

## Sécurité et conformité

> [!important] Défis sécuritaires du Cloud
> La sécurité dans le cloud nécessite des mesures spécifiques adaptées à l'environnement réseau.

### Protection des données en transit

> [!warning] Données en transit sur Internet
> Les données transitent par Internet, ce qui expose à des risques d'interception.

**Solution obligatoire : Chiffrement TLS**
- Utilisation systématique de **HTTPS** (TLS 1.2 minimum, idéalement TLS 1.3)
- Protocoles sécurisés : **SSH**, **SFTP**, **VPN**
- Chiffrement de bout en bout pour les données sensibles

### Gestion des identités

> [!info] Centralisation de l'authentification
> La gestion centralisée des identités simplifie la sécurité et l'administration.

**SSO (Single Sign-On) :**
- **Un seul identifiant** pour accéder à tous les services
- Réduction du nombre de mots de passe à mémoriser
- Simplification de la gestion des accès
- Exemples : **Azure AD**, **Okta**, **Google Identity**

### Conformité RGPD

> [!important] Localisation des données
> Le RGPD impose des contraintes sur la localisation géographique des données personnelles.

**Principe :**
- Les données personnelles de citoyens européens doivent rester dans l'**Espace Économique Européen (EEE)**
- Choix de la **région géographique du datacenter** critique
- Vérification des clauses contractuelles du fournisseur

> [!example] Exemple de conformité
> Pour une application traitant des données de clients français :
> - **Azure** : Région "France Central" (Paris) ou "France South" (Marseille)
> - **AWS** : Région "eu-west-3" (Paris)
> - **GCP** : Région "europe-west9" (Paris)

---

## Certifications Cloud

> [!success] Labels de confiance
> Les certifications attestent du niveau de sécurité et de conformité d'un fournisseur cloud.

### Principales certifications

| Certification | Domaine | Description |
|---------------|---------|-------------|
| **ISO 27001** | Sécurité de l'information | Norme internationale pour le management de la sécurité |
| **SOC 2** | Contrôles organisationnels | Contrôles sur la disponibilité, confidentialité, intégrité |
| **HDS** | Hébergement Données de Santé | **Obligatoire en France** pour héberger des données médicales |
| **PCI DSS** | Paiements par carte | Sécurité des données de cartes bancaires |
| **TISAX** | Automobile | Sécurité de l'information dans l'industrie automobile |

> [!warning] HDS obligatoire en santé
> Pour héberger des **données de santé** en France, la certification **HDS (Hébergement de Données de Santé)** est **obligatoire** légalement.

> [!tip] Vérification des certifications
> Avant de choisir un fournisseur cloud, vérifier sur son site web la liste de ses certifications et audits de sécurité.

---

## Modèles de déploiement et services

## Modèles de déploiement Cloud

> [!abstract] Trois modèles principaux
> Le choix du modèle de déploiement dépend des besoins de sécurité, de contrôle et de budget de l'organisation.

### Cloud Public

> [!quote] Définition
> Ressources **partagées** mises à disposition via Internet par un fournisseur tiers.

**Caractéristiques :**
- Infrastructure mutualisée entre plusieurs clients
- Évolutif et économique (paiement à l'usage)
- Maintenance et mises à jour gérées par le fournisseur
- Accès rapide aux dernières technologies

> [!example] Exemples de fournisseurs
> - **AWS** (Amazon Web Services)
> - **Microsoft Azure**
> - **Google Cloud Platform (GCP)**
> - **OVHcloud**
> - **IBM Cloud**
> - **Oracle Cloud**

> [!tip] Cas d'usage
> Startups, applications web, environnements de test, services non critiques.

---

### Cloud Privé

> [!quote] Définition
> Ressources **dédiées** à une seule organisation, avec un contrôle et une sécurité renforcés.

**Caractéristiques :**
- Infrastructure réservée exclusivement à une organisation
- Contrôle total sur l'infrastructure
- Sécurité et confidentialité maximales
- Peut être géré en interne ou par un prestataire
- Coûts généralement plus élevés

> [!example] Solutions de cloud privé
> - **VMware vSphere** avec **vCloud**
> - **OpenStack**
> - **Microsoft Azure Stack**
> - **Proxmox VE**

> [!tip] Cas d'usage
> Secteurs réglementés (banque, santé, défense), données ultra-sensibles, exigences de conformité strictes.

---

### Cloud Hybride

> [!quote] Définition
> **Combinaison** de cloud public et privé, offrant flexibilité et optimisation des ressources.

**Caractéristiques :**
- Données sensibles dans le cloud privé
- Applications moins critiques dans le cloud public
- Portabilité des données et applications entre environnements
- Optimisation des coûts selon les besoins

> [!success] Avantages du cloud hybride
> - **Flexibilité** : utiliser le meilleur des deux mondes
> - **Optimisation des coûts** : répartir les charges selon la criticité
> - **Conformité** : garder les données sensibles on-premises
> - **Bursting** : déborder vers le cloud public en cas de pics de charge

> [!example] Scénario typique
> Une banque héberge :
> - Ses **données clients** et **transactions** dans son cloud privé (conformité, sécurité)
> - Son **site web marketing** et **applications mobiles** dans le cloud public (scalabilité, coûts)

> [!tip] Technologies d'intégration
> - **Azure Arc** : Gestion unifiée des ressources hybrides Microsoft
> - **AWS Outposts** : Infrastructure AWS déployée on-premises
> - **Google Anthos** : Plateforme multi-cloud et hybride

---

## Diversité des services cloud

> [!info] Multitude de services
> Au-delà de IaaS, PaaS et SaaS, les fournisseurs cloud proposent une gamme étendue de services adaptés à différents besoins.

### Hébergement Web mutualisé

> [!quote] Définition
> **Partage** d'un serveur web entre plusieurs clients, chacun ayant son propre site.

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Coût très réduit (quelques €/mois) | ❌ Performances limitées |
| ✅ Idéal pour petits sites web | ❌ Peu de personnalisation possible |
| ✅ Gestion simplifiée (pas d'admin système) | ❌ Ressources partagées (impact des voisins) |

> [!example] Cas d'usage
> Blog personnel, site vitrine de PME, portfolio, site associatif.

> [!tip] Fournisseurs populaires
> OVH, Ionos, Hostinger, o2switch, Gandi.

---

### Serveurs Privés Virtuels (VPS)

> [!quote] Définition
> **Serveur virtuel dédié** pour chaque client sur un même serveur physique.

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Plus de contrôle qu'un hébergement mutualisé | ❌ Nécessite des compétences techniques |
| ✅ Ressources garanties (CPU, RAM) | ❌ Plus cher que le mutualisé |
| ✅ Accès root/administrateur complet | ❌ Gestion de la sécurité à votre charge |
| ✅ Bon équilibre coût/performance | |

> [!success] VPS = Compromis idéal
> Le VPS offre un excellent **équilibre entre coût et performance** pour de nombreux projets.

> [!example] Cas d'usage
> Applications web moyennes, serveurs de jeux, environnements de développement, serveurs mail.

> [!tip] Fournisseurs VPS
> OVH, Scaleway, Contabo, DigitalOcean, Linode, Vultr.

---

### Serveurs Dédiés (Bare Metal)

> [!quote] Définition
> **Serveur physique entièrement dédié** à un seul client, sans virtualisation intermédiaire.

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Performances maximales | ❌ Plus coûteux |
| ✅ Contrôle total sur le matériel | ❌ Engagement souvent minimum (1 mois à 1 an) |
| ✅ Pas de "noisy neighbor" (voisins bruyants) | ❌ Moins flexible qu'un VPS |
| ✅ Idéal pour charges intensives | ❌ Provisioning plus lent (heures vs minutes) |

> [!example] Cas d'usage
> Bases de données volumineuses, calcul intensif, big data, applications critiques nécessitant des performances constantes.

> [!tip] Quand choisir du Bare Metal ?
> Lorsque la **performance** et la **prévisibilité** sont critiques et justifient le surcoût.

---

## Le stockage

> [!important] Deux types de stockage principaux
> Comprendre la différence entre stockage bloc et stockage objet est essentiel pour bien architecturer dans le cloud.

### Stockage Bloc (Block Storage)

> [!quote] Définition
> Un **disque virtuel** attaché à une VM, comme un disque dur traditionnel.

**Caractéristiques :**
- Fonctionnement comme un disque classique (HDD/SSD)
- Formaté avec un système de fichiers (ext4, NTFS, XFS, etc.)
- **Performance élevée** (faible latence)
- **1 disque = 1 VM** (pas de partage)

**Usage typique :**
- Système d'exploitation (OS)
- Bases de données (MySQL, PostgreSQL, SQL Server)
- Applications nécessitant des I/O rapides

> [!example] Exemples de services Block Storage
> - **AWS** : EBS (Elastic Block Store)
> - **Azure** : Managed Disks
> - **GCP** : Persistent Disks
> - **OVH** : Block Storage

---

### Stockage Objet (Object Storage)

> [!quote] Définition
> Stockage de **fichiers** sans système de fichiers hiérarchique traditionnel. Chaque fichier est un "objet" avec des métadonnées.

**Caractéristiques :**
- **Pas de système de fichiers hiérarchique** classique (pas de dossiers/sous-dossiers au sens traditionnel)
- Accès via **API HTTP/HTTPS**
- **Scalabilité quasi-illimitée**
- Redondance et disponibilité élevées
- Coût très compétitif

**Types de fichiers stockés :**
- Photos, vidéos
- Logs, archives
- Sauvegardes
- Assets de sites web (CSS, JS, images)

> [!example] Format d'accès
> `s3://monBucket/backup/2026/janvier/fichier.zip`

> [!example] Exemples de services Object Storage
> - **AWS** : S3 (Simple Storage Service)
> - **Azure** : Blob Storage
> - **GCP** : Cloud Storage
> - **OVH** : Object Storage Swift/S3

> [!tip] Quand utiliser Object Storage ?
> Pour tout ce qui est **fichier statique**, **archive**, ou **sauvegarde**. Moins cher et plus scalable que le Block Storage.

---

## Conteneurisation

> [!quote] Définition
> La **conteneurisation** permet d'**isoler des applications** dans des environnements légers et portables appelés conteneurs.

**Avantages :**
- **Portabilité** : fonctionne partout de manière identique
- **Efficacité** : plus léger qu'une VM complète
- **Isolation** : applications séparées les unes des autres
- **Déploiement rapide** : secondes vs minutes pour une VM

> [!success] Popularité croissante
> La conteneurisation est devenue **incontournable** pour le déploiement d'applications dans des environnements cloud variés.

---

## Conteneurisation avec Docker

> [!info] Docker : Leader de la conteneurisation
> Docker est la technologie de conteneurisation la plus répandue.

### Concepts clés Docker

| Concept | Définition |
|---------|------------|
| **Image** | Template **figé** et **immuable** (ex : `nginx:latest`, `ubuntu:22.04`) |
| **Conteneur** | **Instance en cours d'exécution** d'une image |
| **Dockerfile** | Fichier de configuration pour la **création d'une image** |
| **Registry** | Dépôt d'images (Docker Hub, registries privés) |

### Exemple de Dockerfile

```dockerfile
FROM python:3.11

# Copier le code de l'application
COPY app.py /app/

# Installer les dépendances
RUN pip install flask

# Commande de démarrage
CMD ["python", "/app/app.py"]
```

> [!note] Explication du Dockerfile
> 1. **FROM** : Image de base (Python 3.11)
> 2. **COPY** : Copie du fichier `app.py` dans le conteneur
> 3. **RUN** : Installation de Flask
> 4. **CMD** : Commande exécutée au démarrage du conteneur

> [!example] Commandes Docker essentielles
> ```bash
> # Construire une image
> docker build -t monapp:v1 .
> 
> # Lancer un conteneur
> docker run -d -p 8080:80 nginx
> 
> # Lister les conteneurs en cours
> docker ps
> 
> # Arrêter un conteneur
> docker stop <container_id>
> ```

---

## Services Cloud de conteneurs

> [!info] Conteneurs managés dans le cloud
> Les fournisseurs cloud proposent des services pour exécuter des conteneurs **sans gérer l'infrastructure sous-jacente**.

### Principaux services

| Fournisseur | Service | Caractéristiques |
|-------------|---------|------------------|
| **Azure** | **Azure Container Instances (ACI)** | Conteneurs **sans gestion de serveur**, facturation à la seconde |
| **AWS** | **AWS Fargate** | Exécution de conteneurs **sans EC2**, serverless |
| **Google** | **Cloud Run** | Déploiement d'**API en conteneur en 1 clic**, auto-scaling |

> [!success] Serverless containers
> Ces services permettent de déployer des conteneurs **sans se soucier des machines virtuelles** sous-jacentes.

> [!example] Cas d'usage Cloud Run
> Déployer une API REST en Python :
> 1. Créer un Dockerfile
> 2. Pusher sur Google Container Registry
> 3. Déployer sur Cloud Run en 1 commande
> 4. Obtenir une URL HTTPS automatiquement

---

## Les modèles économiques

> [!abstract] Flexibilité tarifaire
> Le cloud propose différents modèles de tarification pour s'adapter aux besoins variés des entreprises.

### Forfait

> [!quote] Définition
> Paiement d'un **montant fixe** mensuel ou annuel pour un ensemble défini de ressources et services.

**Caractéristiques :**
- Ressources et services définis à l'avance
- **Prévisibilité budgétaire** totale
- Idéal pour les besoins **constants et prévisibles**
- Simplicité de gestion

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Coûts fixes et prévisibles | ❌ Moins de flexibilité |
| ✅ Pas de surprise sur la facture | ❌ Paiement même si sous-utilisation |
| ✅ Simplifie la comptabilité | ❌ Peut être plus cher si utilisation faible |

> [!example] Exemples de forfait
> - **Microsoft 365** : X€/utilisateur/mois
> - **Google Workspace** : Plans fixes par utilisateur
> - **VPS** : 9,99€/mois pour 2 vCPU, 4GB RAM

---

### Paiement à la consommation

> [!quote] Définition
> Facturation basée sur l'**utilisation réelle** des ressources (Pay-as-you-go).

**Caractéristiques :**
- Facturation **à l'heure**, **à la minute** ou **à la seconde**
- Flexibilité totale pour s'adapter à des besoins **fluctuants**
- Permet d'éviter le **surdimensionnement** et les coûts inutiles
- Optimisation fine des dépenses

| Avantages | Inconvénients |
|-----------|---------------|
| ✅ Ne payez que ce que vous utilisez | ❌ Facture variable et imprévisible |
| ✅ Flexibilité maximale | ❌ Risque de dépassement budgétaire |
| ✅ Évite le gaspillage | ❌ Nécessite un monitoring constant |
| ✅ Idéal pour charges variables | ❌ Peut être plus cher si utilisation constante |

> [!success] Pay-as-you-go : Flexibilité et optimisation des coûts
> Ce modèle est idéal pour les **charges de travail variables** et permet une **optimisation fine** des dépenses.

---

## Choix du modèle

> [!important] Critères de décision
> Le choix entre forfait et paiement à la consommation dépend de votre contexte.

**Facteurs à considérer :**
- **Besoins spécifiques** de l'entreprise
- **Prévisibilité** de la charge de travail
- **Expertise** dans la gestion cloud
- **Tolérance** à la variabilité budgétaire

> [!tip] Stratégie hybride
> Beaucoup d'entreprises combinent les deux modèles :
> - **Forfait** pour les ressources de base (prévisibles)
> - **Pay-as-you-go** pour les pics et besoins ponctuels

---

## Optimisation des coûts

> [!warning] Pay-as-you-go ≠ Gratuit !
> Une **maîtrise budgétaire** est **essentielle** pour éviter les mauvaises surprises.

### Tarification des instances (VMs)

> [!info] Différents modèles de tarification pour les VMs
> Les fournisseurs cloud proposent plusieurs options de tarification pour optimiser les coûts.

#### On-Demand (À la demande)

**Caractéristiques :**
- Facturation **à l'heure** ou **à la seconde**
- **Prix plein**, **flexibilité maximale**
- Aucun engagement
- Démarrage/arrêt instantané

**Usage typique :**
- Environnements de **test** et **développement**
- Charges de travail **imprévisibles**
- Applications avec des pics sporadiques

> [!example] Exemple de tarif
> - 0,10 €/heure
> - ou 0,000027778 € /seconde

---

#### Reserved Instances (Instances réservées)

**Caractéristiques :**
- **Engagement contractuel** de **1 à 3 ans**
- Réduction de **30% à 70%** par rapport au On-Demand
- Paiement partiel ou total à l'avance possible (réduction supplémentaire)

**Usage typique :**
- **Serveurs de production** à charge stable
- Applications fonctionnant **24/7**
- Environnements avec besoins prévisibles

> [!success] Économies substantielles
> Pour des serveurs fonctionnant en continu, les Reserved Instances peuvent générer des **économies de 50% ou plus**.

> [!example] Calcul d'économies
> - VM On-Demand : 0,10 €/h × 24h × 365j = 876 €/an
> - VM Reserved (3 ans, -60%) : 350 €/an
> - **Économie : 526 €/an par VM**

---

### Facturation du stockage

> [!info] Coûts multiples du stockage
> Le stockage cloud ne se facture pas uniquement au volume, mais aussi aux opérations et transferts.

**Composantes de coût :**

| Type de coût | Description | Exemple de tarif |
|--------------|-------------|------------------|
| **Stockage** | Coût par Go/mois | 0,02 €/Go/mois pour S3 Standard |
| **Opérations** | GET, PUT, LIST, DELETE requests | 0,0004 € pour 1000 requêtes GET |
| **Transfert sortant (Egress)** | Données sortant du cloud | 0,09 €/Go |
| **Transfert entrant (Ingress)** | Données entrant dans le cloud | **Gratuit** généralement |

> [!warning] Piège du transfert de données
> Le **transfert de données sortant** (Egress) est **facturé**, alors que l'**entrée** est souvent **gratuite**. Cela peut représenter un coût significatif pour des applications avec beaucoup de trafic sortant.

> [!example] Scénario de coût
> Site web hébergeant 100 Go de vidéos :
> - Stockage : 100 Go × 0,02 €/Go = 2 €/mois
> - Transfert sortant : 1000 Go de vidéos vues × 0,09 €/Go = 90 €/mois
> - **Total : 92 €/mois** (dont 98% de transfert !)

---

### Bonnes pratiques d'optimisation

> [!success] Stratégies d'économie
> Quelques actions simples permettent de réduire significativement les coûts cloud.

**1. Éteindre les VMs de dev/test hors heures de travail**
- Économie : **-60% sur les ressources** de développement
- Automatisation avec des scripts ou Azure Automation

**2. Bon dimensionnement (Right-Sizing)**
- Analyser l'utilisation réelle (CPU, RAM)
- Ajuster la taille des instances
- Éviter le surdimensionnement "par précaution"

**3. Suppression des ressources inutilisées**
- Disques non attachés
- Snapshots anciens
- Adresses IP publiques non utilisées
- Images obsolètes

**4. Utiliser des tiers de stockage adaptés**
- Stockage "chaud" (fréquent) : S3 Standard
- Stockage "tiède" (occasionnel) : S3 Infrequent Access (-50%)
- Stockage "froid" (archive) : S3 Glacier (-80%)

> [!tip] Outils d'optimisation
> - **AWS Cost Explorer**, **Azure Cost Management**, **GCP Cost Management**
> - Alertes budgétaires
> - Recommandations automatiques de dimensionnement

---

## Cloud bureautiques

> [!info] Suites collaboratives cloud
> Les suites bureautiques cloud ont remplacé les installations locales dans de nombreuses entreprises.

### Microsoft 365 (Ex-Office 365)

**Services inclus :**
- **Exchange Online** : Messagerie (email, calendrier, contacts)
- **SharePoint** : Gestion documentaire et intranet
- **Teams** : Communication et collaboration (chat, visio, partage)
- **OneDrive** : Stockage cloud personnel

**Administration : Portail M365 Admin Center**

**Tâches d'administration courantes :**
- Gestion des utilisateurs et des licences
- Configuration des domaines personnalisés (MX, SPF, DKIM)
- Politiques de sécurité (MFA, accès conditionnel)
- Gestion des groupes de distribution
- Configuration DLP (Data Loss Prevention)

> [!example] Configuration DNS pour M365
> ```
> MX    : mail.protection.outlook.com (priorité 0)
> SPF   : v=spf1 include:spf.protection.outlook.com -all
> DKIM  : selector1._domainkey et selector2._domainkey
> ```

---

### Google Workspace (Ex-G Suite)

**Services inclus :**
- **Gmail** : Messagerie professionnelle
- **Drive** : Stockage et partage de fichiers
- **Meet** : Visioconférence
- **Calendar** : Gestion d'agenda partagé
- **Docs, Sheets, Slides** : Suite bureautique en ligne

**Administration : Console d'administration Google**

**Tâches d'administration courantes :**
- Création d'**unités organisationnelles** (OU)
- Gestion des **groupes de distribution**
- Configuration des **règles de messagerie**
- Gestion des **appareils mobiles** (MDM)
- Politiques de **partage externe**

> [!tip] Unités organisationnelles
> Utiliser les OU pour appliquer des **politiques différentes** selon les départements (ex : RH avec chiffrement obligatoire, Marketing avec partage externe autorisé).

---

## Rôle du TSSR

> [!important] Responsabilités du TSSR dans les environnements cloud bureautiques
> En tant que TSSR, tu seras amené à gérer au quotidien les tâches d'administration courantes.

**Tâches courantes (niveau N1/N2) :**

- **Gestion des comptes utilisateurs**
  - Création/modification/suppression de comptes
  - Réinitialisation de mots de passe
  - Attribution de licences

- **Gestion des boîtes mail partagées**
  - Création de boîtes partagées (support@, contact@)
  - Gestion des permissions d'accès

- **Configuration de synchronisation AD ↔ Azure AD (environnement hybride)**
  - Azure AD Connect
  - Troubleshooting des problèmes de sync

- **Diagnostic des problèmes de connexion (N2)**
  - Problèmes d'authentification
  - MFA bloqué
  - Accès conditionnel refusé

> [!tip] Compétences clés TSSR
> Maîtriser **PowerShell** pour Microsoft 365 et **gcloud CLI** pour Google Workspace permet d'automatiser les tâches répétitives.

---

## Interfaces d'accès au Cloud

> [!abstract] Multiples moyens d'accès
> Les services cloud sont accessibles via différentes interfaces selon les besoins et le niveau d'expertise.

### Shell SSH

> [!quote] Définition
> Accès **sécurisé en ligne de commande** pour la gestion complète des serveurs et applications.

**Caractéristiques :**
- **Contrôle total** sur le système
- Nécessite des **compétences techniques** avancées
- Idéal pour l'**automatisation** et les scripts
- **Protocole sécurisé** (chiffrement)

> [!example] Connexion SSH à une VM
> ```bash
> ssh -i ~/.ssh/ma-cle.pem ubuntu@51.210.123.45
> ```

> [!tip] Cas d'usage
> Administration système, déploiement d'applications, troubleshooting avancé, automatisation via scripts Bash/Python.

---

### Interface Web

> [!quote] Définition
> Accès via un **navigateur web**, interface graphique **facile et intuitive**.

**Caractéristiques :**
- **Accessible partout** (navigateur web)
- **Convivial** pour les débutants
- Convient pour la **gestion de base**
- **Suivi visuel** des ressources (dashboards, graphiques)

> [!example] Consoles web cloud
> - **AWS** : Console AWS (console.aws.amazon.com)
> - **Azure** : Portail Azure (portal.azure.com)
> - **GCP** : Console Google Cloud (console.cloud.google.com)

> [!success] Idéal pour commencer
> L'interface web est le point d'entrée recommandé pour découvrir et comprendre les services cloud.

---

### API (Application Programming Interface)

> [!quote] Définition
> Permet une **intégration** et une **automatisation programmables** des services cloud.

**Caractéristiques :**
- **Intégration** avec des applications tierces
- **Automatisation** de tâches complexes
- Opérations **personnalisées** et **avancées**
- Accès via **HTTP/HTTPS** (REST, GraphQL)

> [!example] Exemple d'appel API AWS (création de VM)
> ```bash
> aws ec2 run-instances \
>   --image-id ami-0c55b159cbfafe1f0 \
>   --instance-type t2.micro \
>   --key-name ma-cle \
>   --region eu-west-3
> ```

> [!tip] Outils CLI (Command Line Interface)
> - **AWS** : `aws cli`
> - **Azure** : `az cli`
> - **GCP** : `gcloud`
> - **Terraform** : Infrastructure as Code multi-cloud

---

## Diagnostic et troubleshooting

> [!warning] Problèmes courants en environnement cloud
> Savoir diagnostiquer rapidement les problèmes les plus fréquents est essentiel pour un TSSR.

### Problème 1 : "Je ne peux pas me connecter à ma VM"

> [!example] Diagnostic systématique

**1. Vérifier les groupes de sécurité / Firewall**
- Azure : Network Security Groups (NSG)
- AWS : Security Groups
- Règles d'entrée (Inbound) pour SSH (22) ou RDP (3389) ?

**2. Vérifier l'assignation d'@IP publique**
- La VM a-t-elle une IP publique ?
- L'IP publique est-elle statique ou dynamique ?

**3. Tester la connectivité réseau**
```bash
# Test ping (si ICMP autorisé)
ping 51.210.123.45

# Test SSH (Linux)
ssh -v ubuntu@51.210.123.45

# Test RDP (Windows) - port 3389
telnet 51.210.123.45 3389
```

**4. Vérifier le statut de la VM**
- VM démarrée ?
- Ressources disponibles (CPU, RAM) ?

> [!tip] Checklist de connexion
> **NSG/SG → IP publique → Connectivité réseau → État VM**

---

### Problème 2 : "Mon application est lente"

> [!example] Diagnostic de performance

**1. Consulter les métriques CPU/RAM de l'hyperviseur**
- Azure Monitor, CloudWatch, Cloud Monitoring
- CPU saturé (>80%) ?
- RAM insuffisante (swap) ?

**2. Vérifier la bande passante réseau**
- **Throttling** (limitation) réseau actif ?
- Latence élevée ?
- Paquets perdus ?

**3. Analyser les logs applicatifs**
- Logs dans **Azure Monitor**, **CloudWatch Logs**, **Stackdriver**
- Erreurs applicatives ?
- Requêtes lentes vers la BDD ?

> [!tip] Métriques clés à surveiller
> - **CPU** : Utilisation moyenne et pics
> - **RAM** : Utilisation et swap
> - **Disque** : IOPS et latence
> - **Réseau** : Bande passante et latence

---

### Problème 3 : "L'email ne part pas depuis M365"

> [!example] Diagnostic email

**1. Vérifier les enregistrements DNS**
- **MX** : Serveur de messagerie
- **SPF** : Serveurs autorisés à envoyer
- **DKIM** : Signature des emails
- **DMARC** : Politique d'authentification

**2. Vérifier dans le Centre de messages M365**
- Incident en cours chez Microsoft ?
- Maintenance planifiée ?

**3. Tester avec nslookup**
```bash
# Vérifier l'enregistrement MX
nslookup -type=MX domaine.fr

# Vérifier SPF
nslookup -type=TXT domaine.fr
```

**4. Consulter les logs de transport Exchange**
- Messages bloqués ?
- Erreurs de relais ?

> [!warning] SPF mal configuré = emails en spam
> Un enregistrement SPF incorrect ou absent envoie souvent les emails dans les spams des destinataires.

---

## Usages, avantages et inconvénients

## Utilisation typique des services Cloud

> [!info] Applications courantes du cloud
> Le cloud computing s'est imposé dans de nombreux domaines d'activité.

**Cas d'usage principaux :**
- **Stockage et traitement de données** pour les entreprises (data lakes, analytics)
- **Hébergement de sites web et d'applications** (e-commerce, SaaS)
- **Solutions de backup et récupération** en cas de sinistre (PRA/PCA)
- **Environnements de développement et test** (CI/CD, DevOps)
- **Big Data et Machine Learning** (entraînement de modèles IA)
- **Streaming vidéo et CDN** (distribution de contenu)

---

## Avantages du Cloud

> [!success] Bénéfices majeurs du cloud computing
> Le cloud apporte des avantages stratégiques et opérationnels significatifs.

### Avantages principaux

| Avantage | Description | Impact business |
|----------|-------------|-----------------|
| **Flexibilité et évolutivité** | Ajustement rapide des ressources à la demande | Réactivité face aux variations de charge |
| **Réduction des coûts d'infrastructure** | Pas d'investissement matériel initial (CAPEX → OPEX) | Trésorerie préservée, coûts variables |
| **Collaboration améliorée** | Accès partagé aux données et applications | Productivité des équipes distribuées |
| **Accès à distance** | Disponible partout, tout le temps | Travail hybride, mobilité |
| **Mises à jour automatiques** | Nouvelles fonctionnalités sans intervention | Toujours à jour technologiquement |
| **Haute disponibilité** | Redondance et résilience intégrées | Continuité de service |

> [!example] Transformation CAPEX → OPEX
> **Avant** : Acheter 50 serveurs à 5000€ = 250 000€ d'investissement initial
> **Avec le cloud** : 0€ initial, paiement mensuel selon utilisation réelle

---

## Inconvénients du Cloud

> [!warning] Limites et défis du cloud computing
> Malgré ses avantages, le cloud présente aussi des inconvénients qu'il faut anticiper.

### Inconvénients principaux

| Inconvénient | Description | Mitigation |
|--------------|-------------|------------|
| **Sécurité et confidentialité** | Données confiées à un tiers | Chiffrement, certifications, audits |
| **Dépendance fournisseur** | Vendor lock-in, difficultés de migration | Architecture multi-cloud, standards ouverts |
| **Complexité de gestion** | Multitude de services et options | Formation, bonnes pratiques, governance |
| **Coûts imprévisibles** | Risque de dépassement budgétaire | Monitoring, alertes, optimisation continue |
| **Performances réseau** | Dépendance à la connexion Internet | Liens redondants, cloud hybride |
| **Conformité réglementaire** | Complexité RGPD, souveraineté données | Choix de régions appropriées, contrats DPA |

> [!tip] Stratégie de mitigation
> La plupart des inconvénients peuvent être atténués par une **bonne gouvernance** et une **architecture réfléchie**.

---

## Sécuriser l'accès aux services

> [!important] Sécurité de l'accès : priorité absolue
> La sécurisation des accès aux ressources cloud est critique pour protéger les données et les systèmes.

### Authentification Multi-Facteurs (MFA)

> [!quote] Définition
> L'**Authentification Multi-Facteurs** combine **plusieurs facteurs d'authentification** pour vérifier l'identité d'un utilisateur.

**Principe des facteurs :**
1. **Quelque chose que vous savez** : mot de passe, code PIN
2. **Quelque chose que vous avez** : téléphone, token physique, carte
3. **Quelque chose que vous êtes** : biométrie (empreinte, reconnaissance faciale)

> [!warning] MFA obligatoire pour les admins
> La MFA doit être **obligatoire** pour **tous les comptes administrateurs** sans exception.

**Méthodes de MFA courantes :**
- **Code SMS** (moins sécurisé, à éviter si possible)
- **Application d'authentification** (Google Authenticator, Microsoft Authenticator, Authy)
- **Token physique** (YubiKey, RSA SecurID)
- **Notification push** (validation sur smartphone)
- **Biométrie** (Windows Hello, Touch ID)

> [!success] Protection contre le vol de credentials
> Même si un attaquant obtient votre mot de passe, il ne pourra pas se connecter sans le second facteur.

---

### RBAC (Role-Based Access Control)

> [!quote] Définition
> Le **contrôle d'accès basé sur les rôles** attribue des **permissions prédéfinies** selon la fonction de l'utilisateur.

**Rôles typiques dans Azure :**
- **Lecteur** (Reader) : Lecture seule, aucune modification possible
- **Contributeur** (Contributor) : Gestion des ressources, sauf gestion des accès
- **Propriétaire** (Owner) : Contrôle total, y compris gestion des accès

**Rôles typiques dans AWS :**
- **ReadOnlyAccess** : Consultation uniquement
- **PowerUserAccess** : Gestion des ressources, sauf IAM
- **AdministratorAccess** : Accès complet

> [!important] Principe du moindre privilège
> Donner **uniquement les droits nécessaires** pour accomplir les tâches requises, pas plus.

> [!example] Application du moindre privilège
> - **Développeur** : Contributeur sur le groupe de ressources "dev" uniquement
> - **Opérateur** : Lecteur en prod + redémarrage de VMs
> - **Administrateur** : Propriétaire global (limité à 2-3 personnes)

> [!warning] Éviter les comptes "Dieu"
> Ne pas utiliser un compte **Propriétaire/Admin** pour les tâches quotidiennes. Créer des comptes avec privilèges limités.

---

### Accès Conditionnel (Conditional Access)

> [!quote] Définition
> L'**accès conditionnel** applique des **politiques de sécurité dynamiques** en fonction du contexte de connexion.

**Conditions évaluées :**
- **Géolocalisation** : Pays, région, IP
- **Type d'appareil** : Managed/Unmanaged, OS, compliance
- **Réseau** : Réseau d'entreprise vs externe
- **Application** : Sensibilité de l'application accédée
- **Risque utilisateur** : Comportement anormal détecté
- **Heure** : Plage horaire de travail

**Actions possibles :**
- **Autoriser** l'accès
- **Bloquer** l'accès
- **Exiger MFA**
- **Exiger un appareil compliant**
- **Exiger un mot de passe fort**

> [!example] Exemple de politique d'accès conditionnel
> **SI** géolocalisation = pays étranger
> **ET** utilisateur = Administrateur
> **ALORS** Blocage complet de l'accès
> 
> **Ou bien :**
> 
> **SI** connexion hors réseau d'entreprise
> **ET** application = Azure Portal
> **ALORS** Exiger MFA

> [!tip] Politiques graduelles
> Commencer par des politiques en **mode Report-Only** (audit) avant de passer en mode **Enforcement** (application).

---

## Briques techniques du Cloud

## Hyperviseurs

> [!quote] Rappel : Définition
> Les **hyperviseurs** sont des logiciels permettant la **création et la gestion de machines virtuelles**.

**Rôle crucial :**
- Partage des ressources physiques entre plusieurs VMs
- Isolation entre les VMs
- Gestion de l'allocation dynamique des ressources

> [!example] Hyperviseurs courants
> - **VMware** : ESXi (Type 1)
> - **Microsoft** : Hyper-V (Type 1)
> - **Linux** : KVM (Kernel-based Virtual Machine)
> - **Citrix** : XenServer
> - **Proxmox** : Proxmox VE (open source)

> [!info] Dans le cloud public
> Les fournisseurs cloud utilisent des hyperviseurs propriétaires optimisés :
> - **AWS** : Xen / Nitro
> - **Azure** : Hyper-V customisé
> - **GCP** : KVM customisé

---

## Orchestration

> [!important] Outils de cluster et orchestration
> L'orchestration automatise le déploiement, la mise à l'échelle et la gestion d'applications conteneurisées.

### Kubernetes (K8s)

> [!quote] Définition
> **Kubernetes** est un orchestrateur open-source qui gère automatiquement le déploiement d'applications conteneurisées sur plusieurs machines.

**Fonctionnalités clés :**
- **Déploiement automatisé** de conteneurs
- **Scaling** (mise à l'échelle) horizontal automatique
- **Self-healing** : redémarrage automatique des conteneurs défaillants
- **Load balancing** intégré
- **Rolling updates** sans interruption de service

**Services Cloud Kubernetes :**
- **Azure** : **AKS** (Azure Kubernetes Service)
- **AWS** : **EKS** (Elastic Kubernetes Service)
- **GCP** : **GKE** (Google Kubernetes Engine)

> [!success] Kubernetes = Standard de facto
> Kubernetes est devenu le **standard de l'industrie** pour l'orchestration de conteneurs.

---

### Docker Swarm

> [!info] Alternative à Kubernetes
> **Docker Swarm** est le système de gestion de cluster natif de Docker.

**Caractéristiques :**
- Plus **simple** que Kubernetes
- Intégré nativement à Docker
- Courbe d'apprentissage plus douce
- Moins de fonctionnalités avancées

> [!tip] Quand choisir Docker Swarm ?
> Pour des **clusters simples** ou des équipes débutantes. Pour des besoins complexes, Kubernetes est préférable.

---

## Infrastructure as Code (IaC)

> [!quote] Définition
> L'**Infrastructure as Code** consiste à **décrire l'infrastructure** avec des **fichiers de configuration** versionnés, plutôt que manuellement.

**Avantages :**
- **Reproductibilité** : infrastructure identique à chaque déploiement
- **Versionnement** : historique des changements dans Git
- **Automatisation** : déploiement via CI/CD
- **Documentation** : le code documente l'infrastructure
- **Rapidité** : déploiement de dizaines de ressources en minutes

### Terraform

> [!info] Terraform : Outil IaC multi-cloud
> **Terraform** de HashiCorp permet de gérer l'infrastructure sur plusieurs clouds avec un langage unifié.

**Exemple de fichier `.tf` :**

```hcl
resource "azurerm_virtual_machine" "vm" {
  name                = "vm-prod-web"
  location            = "France Central"
  resource_group_name = "rg-production"
  vm_size             = "Standard_B2s"
  
  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

> [!success] Infrastructure versionnée
> Avec Terraform, l'infrastructure est **versionnée dans Git** comme du code applicatif.

---

### Autres outils IaC

| Outil | Fournisseur | Format | Spécificité |
|-------|-------------|--------|-------------|
| **ARM** | Azure | JSON | Templates Azure Resource Manager |
| **CloudFormation** | AWS | YAML/JSON | Service natif AWS |
| **Pulumi** | Multi-cloud | TypeScript, Python, Go | IaC avec langages de programmation |
| **Ansible** | Red Hat | YAML | Configuration management + IaC |

---

### Avantages de l'IaC

> [!success] Bénéfices concrets de l'Infrastructure as Code

**1. Déploiement rapide et à grande échelle**
- Déploiement de **50 VMs identiques en 10 minutes**
- Configuration cohérente garantie

**2. Versionnement dans Git**
- Historique complet des changements
- Revue de code (code review) avant déploiement
- Rollback facile en cas de problème

**3. Reproduction d'environnements**
- Reproduction d'un environnement de **test à l'identique** en production
- Environnements éphémères pour les démos

**4. Documentation automatique**
- Le code IaC **documente** l'infrastructure
- Pas de "configuration fantôme" oubliée

> [!warning] Danger : Suppression de ressources
> La **suppression d'une ligne** dans le code Terraform a pour conséquence la **destruction de ressources réelles** !
> 
> Toujours :
> - Utiliser `terraform plan` avant `terraform apply`
> - Activer les protections contre la suppression
> - Faire des sauvegardes du state file

> [!example] Workflow IaC typique
> ```bash
> # 1. Écrire le code Terraform
> vim main.tf
> 
> # 2. Initialiser Terraform
> terraform init
> 
> # 3. Prévisualiser les changements
> terraform plan
> 
> # 4. Appliquer les changements
> terraform apply
> 
> # 5. Versionner dans Git
> git add main.tf
> git commit -m "Ajout VM production"
> git push
> ```

---

## Plateformes de Cloud

> [!info] Environnements intégrés de gestion cloud
> Les plateformes cloud fournissent des outils complets pour déployer et gérer des infrastructures cloud privées ou publiques.

### OpenStack

> [!quote] Définition
> **OpenStack** est une plateforme **open-source** pour créer et gérer des clouds **privés et publics**.

**Composants principaux :**
- **Nova** : Gestion des instances (VMs)
- **Neutron** : Réseau virtuel
- **Cinder** : Block storage
- **Swift** : Object storage
- **Glance** : Gestion d'images
- **Keystone** : Authentification

> [!example] Utilisateurs d'OpenStack
> - OVHcloud
> - Rackspace
> - De nombreuses entreprises pour leurs clouds privés

---

## Architecture réseau

> [!important] Réseau dans le cloud
> L'architecture réseau cloud repose sur des concepts de réseaux virtuels et de segmentation.

### Plages d'adresses et sous-réseaux

**Organisation typique :**
- **Plages d'adresses privées** découpées en sous-réseaux
  - RFC 1918 : 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16

**Types de sous-réseaux :**

| Type | Fonction | Accès Internet |
|------|----------|----------------|
| **Sous-réseau public** | Ressources accessibles depuis Internet | Oui (via NAT Gateway/Load Balancer) |
| **Sous-réseau privé** | BDD, serveurs internes, backend | Non (sauf via NAT pour sorties) |

> [!example] Exemple d'architecture
> ```
> VNet Azure : 10.0.0.0/16
> ├── Subnet-Public : 10.0.1.0/24 (Web servers)
> ├── Subnet-Private : 10.0.2.0/24 (Databases)
> └── Subnet-Management : 10.0.3.0/24 (Bastion, outils admin)
> ```

---

### Règles de pare-feu

> [!warning] Sécurité réseau
> Les règles de pare-feu (NSG/Security Groups) contrôlent le trafic entrant et sortant.

> [!example] Exemple de règle NSG
> **Accès RDP (TCP 3389) uniquement depuis le VPN d'entreprise**
> - **Source** : 203.0.113.0/24 (IP VPN entreprise)
> - **Destination** : 10.0.2.0/24 (subnet privé)
> - **Port** : 3389
> - **Protocole** : TCP
> - **Action** : Allow

> [!tip] Bonnes pratiques réseau
> - **Principe du moindre privilège** : bloquer par défaut, autoriser explicitement
> - **Segmentation** : séparer les environnements (dev, test, prod)
> - **Bastion host** : point d'entrée unique pour l'administration
> - **Logging** : activer les logs de flux réseau

---

## Gestion de ressources

## Scalabilité Verticale (Scaling Up/Down)

> [!quote] Définition
> **Augmentation ou réduction** des capacités d'un serveur **existant** (CPU, RAM, disque).

| Avantages | Limites |
|-----------|---------|
| ✅ Simplicité de mise en œuvre | ❌ Plafond de capacité matériel |
| ✅ Pas de modification d'architecture | ❌ Interruption de service (redémarrage) |
| ✅ Application compatible sans changement | ❌ Single Point of Failure (SPOF) |

> [!example] Exemple de scaling vertical
> VM avec 2 vCPU, 4GB RAM → 4 vCPU, 8GB RAM (nécessite redémarrage)

> [!tip] Quand utiliser le scaling vertical ?
> Pour des applications **monolithiques** qui ne peuvent pas être distribuées, ou pour des ajustements ponctuels de capacité.

---

## Scalabilité Horizontale (Scaling Out/In)

> [!quote] Définition
> **Ajout ou suppression de serveurs** pour augmenter ou diminuer les ressources totales.

| Avantages | Limites |
|-----------|---------|
| ✅ Flexibilité accrue | ❌ Complexité de gestion |
| ✅ Meilleure tolérance aux pannes | ❌ Application doit supporter la distribution |
| ✅ Pas de limite théorique de capacité | ❌ Nécessite un load balancer |
| ✅ Pas d'interruption de service | ❌ Gestion de l'état (sessions) |

> [!example] Exemple de scaling horizontal
> 2 serveurs web → 10 serveurs web (sans interruption, via load balancer)

> [!success] Approche moderne privilégiée
> Le **scaling horizontal** est l'approche **privilégiée dans le cloud** pour les applications modernes (microservices, stateless).

---

## Élasticité

> [!quote] Définition
> L'**élasticité** est la capacité à ajuster **automatiquement** les ressources en fonction de la demande.

**Principe :**
- **Auto-scaling** : ajout/suppression automatique de ressources
- **Basé sur des métriques** : CPU, RAM, nombre de requêtes, etc.
- **Optimisation coûts/performance** : ressources adaptées en temps réel

> [!example] Exemple d'élasticité
> Un site e-commerce :
> - **Nuit** : 2 serveurs web (charge faible)
> - **Journée** : 5 serveurs web (charge normale)
> - **Black Friday** : 20 serveurs web (charge intense)
> - **Automatique** : scaling basé sur CPU > 70%

> [!success] Élasticité = Automatisation du scaling
> L'élasticité permet une **gestion des ressources dynamique et efficace** sans intervention manuelle.

---

## Haute disponibilité (HA) et continuité d'activité

> [!important] Garantir la disponibilité des services
> La haute disponibilité repose sur la redondance géographique et technique.

### Zones de Disponibilité (Availability Zones)

> [!quote] Définition
> **Datacenters physiquement séparés** dans une même région géographique.

**Caractéristiques :**
- **Séparation physique** : bâtiments différents, alimentation électrique séparée
- **Protection contre pannes locales** : incendie, coupure électrique, inondation
- **Latence faible** entre zones : < 2ms généralement

> [!example] Zones de disponibilité
> - **Azure** : 3 zones à Paris (France Central)
> - **AWS** : Généralement 3 zones par région
> - **GCP** : Multiple zones par région

**Stratégie de déploiement :**
- Déployer une **VM dans chaque zone** pour la redondance
- Si une zone tombe, les autres continuent

---

### Régions Géographiques

> [!info] Régions cloud
> Chaque fournisseur cloud dispose de multiples régions dans le monde.

**Exemples de régions :**

| Fournisseur | Nom de région | Localisation |
|-------------|---------------|--------------|
| **AWS** | eu-west-3 | Paris, France |
| **AWS** | us-east-1 | Virginie, USA |
| **Azure** | France Central | Paris, France |
| **Azure** | West Europe | Pays-Bas |
| **GCP** | europe-west9 | Paris, France |

> [!warning] Attention au RGPD
> Pour les données **sensibles** ou **personnelles**, privilégier des régions **en Europe** pour respecter le RGPD.

---

## PRA/PCA dans le Cloud

> [!important] Reprise et continuité d'activité
> Le cloud facilite la mise en place de PRA et PCA grâce à ses fonctionnalités natives.

### RTO et RPO

> [!quote] Définitions
> - **RTO (Recovery Time Objective)** : Temps **maximum acceptable** avant restauration du service
> - **RPO (Recovery Point Objective)** : Perte de données **acceptable**

> [!example] Exemple de RTO/RPO
> **RTO de 4h** = Le service doit être rétabli en **maximum 4 heures**
> **RPO de 1h** = Perte de données **maximum 1 heure**

**Impact sur l'architecture :**
- **RPO faible** (15 min) → Réplication synchrone, coût élevé
- **RPO élevé** (24h) → Sauvegardes quotidiennes, coût réduit

---

### Solutions Cloud pour PRA/PCA

| Solution | Description | RTO | RPO |
|----------|-------------|-----|-----|
| **Sauvegarde automatique** | Snapshots, backups planifiés | Heures | Minutes à heures |
| **Réplication géographique** | BDD répliquée entre 2 régions | Minutes | Secondes |
| **Bascule automatique (Failover)** | Basculement auto si région HS | Secondes à minutes | Quasi-nul |

> [!example] Scénario de disaster recovery
> **SI** datacenter Paris hors service (panne majeure)
> **ALORS** Bascule automatique vers datacenter Amsterdam
> **RÉSULTAT** : Service maintenu avec RTO < 5 minutes

> [!success] Cloud = PRA/PCA simplifié
> Le cloud facilite grandement la mise en place de **plans de reprise et de continuité d'activité** grâce à ses fonctionnalités natives de réplication et de haute disponibilité.

---

## Load Balancing (Équilibrage de Charge)

> [!quote] Définition
> Le **load balancing** distribue le trafic ou les demandes de travail entre **plusieurs serveurs**.

**Objectifs :**
- **Utilisation optimale** des ressources
- **Éviter la surcharge** d'un seul serveur
- **Haute disponibilité** : si un serveur tombe, les autres prennent le relais
- **Performance** : distribution intelligente du trafic

**Types de load balancers :**
- **Layer 4 (Transport)** : Basé sur IP/Port (TCP/UDP)
- **Layer 7 (Application)** : Basé sur le contenu HTTP (URL, headers)

> [!example] Load balancer Layer 7
> ```
> myapp.com/api → Serveur API (pool backend-api)
> myapp.com/images → Serveur statique (pool backend-static)
> ```

**Services cloud :**
- **Azure** : Azure Load Balancer (L4), Application Gateway (L7)
- **AWS** : Classic Load Balancer, Application Load Balancer (L7), Network Load Balancer (L4)
- **GCP** : Cloud Load Balancing

> [!tip] Health checks
> Les load balancers effectuent des **health checks** (vérifications de santé) réguliers pour retirer automatiquement les serveurs défaillants du pool.

---

## Conclusion

> [!success] Le cloud est incontournable
> Le cloud est devenu **incontournable et essentiel** en entreprise moderne.

### Points clés finaux

**Le cloud apporte :**
- **Flexibilité** et capacité d'adaptation rapide
- **Économies** via le passage de CAPEX à OPEX
- **Innovation** accélérée grâce à l'accès aux dernières technologies

**Défis à relever :**
- **Sécurité des données** reste un défi permanent
- **Bonne analyse des coûts** indispensable pour éviter les dérives budgétaires
- **Gestion rigoureuse** nécessaire pour optimiser l'utilisation

> [!important] Compétences TSSR essentielles
> En tant que TSSR, tu dois maîtriser :
> - Les **modèles de services** (IaaS, PaaS, SaaS)
> - Les **bonnes pratiques de sécurité** (MFA, RBAC, chiffrement)
> - L'**optimisation des coûts** (right-sizing, reserved instances)
> - Le **diagnostic et troubleshooting** des problèmes courants
> - Les **outils d'administration** (CLI, IaC, portails web)

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Modèles de services

- **IaaS** : Infrastructure virtualisée (VMs, stockage, réseau) - Vous gérez OS et applications
- **PaaS** : Plateforme de développement - Vous gérez uniquement le code
- **SaaS** : Logiciel en ligne - Vous gérez uniquement les données et la configuration

### Modèles de déploiement

- **Cloud Public** : Ressources partagées, économique, scalable (AWS, Azure, GCP)
- **Cloud Privé** : Ressources dédiées, contrôle maximal, coûteux
- **Cloud Hybride** : Meilleur des deux mondes, flexibilité optimale

### Hyperviseurs et virtualisation

- **Type 1 (Bare Metal)** : ESXi, Hyper-V, KVM - Directement sur le matériel, pour production
- **Type 2 (Hosted)** : VirtualBox, VMware Workstation - Sur un OS hôte, pour tests
- **Cloud = Virtualisation + Automatisation + Réseau**

### Stockage cloud

- **Block Storage** : Disque virtuel attaché à une VM (OS, BDD) - Haute performance, 1:1
- **Object Storage** : Fichiers sans hiérarchie (photos, archives) - Scalable, économique

### Sécurité

- **MFA obligatoire** pour tous les comptes administrateurs
- **RBAC** : Principe du moindre privilège, rôles prédéfinis
- **Accès conditionnel** : Politiques basées sur contexte (géolocalisation, appareil, réseau)
- **Chiffrement TLS** pour toutes les données en transit
- **Certifications** : ISO 27001, SOC 2, HDS (santé en France)

### Modèles économiques

- **Forfait** : Coûts fixes, prévisibilité budgétaire
- **Pay-as-you-go** : Paiement à l'usage, flexibilité maximale
- **On-Demand** : Prix plein, aucun engagement
- **Reserved Instances** : -30% à -70%, engagement 1-3 ans
- **Optimisation** : Éteindre VMs inutilisées, right-sizing, supprimer ressources orphelines

### Infrastructure as Code (IaC)

- **Terraform** : Multi-cloud, fichiers .tf en HCL
- **ARM** : Azure Resource Manager (JSON)
- **CloudFormation** : AWS (YAML/JSON)
- **Avantages** : Reproductibilité, versionnement Git, automatisation, documentation

### Haute disponibilité

- **Zones de disponibilité** : Datacenters séparés dans une région (protection pannes locales)
- **Régions géographiques** : Datacenters dans différents pays (disaster recovery)
- **RTO** : Temps max de restauration (ex : 4h)
- **RPO** : Perte de données acceptable (ex : 1h)
- **Load Balancing** : Distribution du trafic entre serveurs

### Scalabilité

- **Verticale** : Augmenter CPU/RAM d'un serveur existant (limité, interruption)
- **Horizontale** : Ajouter des serveurs (illimité, sans interruption, complexe)
- **Élasticité** : Ajustement automatique basé sur métriques

### Troubleshooting courant

- **Connexion impossible** : NSG/Security Groups → IP publique → Connectivité → État VM
- **Application lente** : Métriques CPU/RAM → Bande passante → Logs applicatifs
- **Email bloqué** : DNS (MX, SPF, DKIM) → Centre de messages M365 → Logs de transport

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Cloud Computing** | Fourniture de services informatiques (serveurs, stockage, BDD, réseau, logiciels) via Internet |
| **IaaS** | Infrastructure as a Service - Ressources informatiques virtualisées (VMs, stockage, réseau) |
| **PaaS** | Platform as a Service - Environnement de développement et déploiement clé en main |
| **SaaS** | Software as a Service - Logiciels accessibles via abonnement en ligne |
| **Cloud Public** | Infrastructure partagée via Internet (AWS, Azure, GCP) |
| **Cloud Privé** | Infrastructure dédiée à une seule organisation |
| **Cloud Hybride** | Combinaison de cloud public et privé |
| **Hyperviseur** | Logiciel permettant de créer et gérer des machines virtuelles |
| **VM (Virtual Machine)** | Machine virtuelle - Serveur virtualisé fonctionnant sur un hyperviseur |
| **Instance** | Machine virtuelle dans le cloud (terme générique) |
| **Container** | Environnement isolé léger pour exécuter une application (Docker) |
| **Kubernetes (K8s)** | Orchestrateur de conteneurs open-source |
| **VPC/VNet** | Virtual Private Cloud/Network - Réseau virtuel isolé dans le cloud |
| **NSG** | Network Security Group - Pare-feu virtuel (Azure) |
| **Security Group** | Pare-feu virtuel (AWS) |
| **Load Balancer** | Équilibreur de charge - Distribue le trafic entre plusieurs serveurs |
| **Auto-scaling** | Ajustement automatique du nombre de ressources selon la charge |
| **Scaling Vertical** | Augmenter CPU/RAM d'un serveur existant (Scale Up/Down) |
| **Scaling Horizontal** | Ajouter/supprimer des serveurs (Scale Out/In) |
| **Élasticité** | Capacité à ajuster automatiquement les ressources selon la demande |
| **Block Storage** | Stockage bloc - Disque virtuel attaché à une VM (EBS, Azure Disk) |
| **Object Storage** | Stockage objet - Fichiers sans hiérarchie (S3, Azure Blob) |
| **Snapshot** | Capture instantanée de l'état d'un disque ou VM |
| **Availability Zone** | Zone de disponibilité - Datacenter isolé dans une région |
| **Region** | Région géographique contenant plusieurs datacenters |
| **RTO** | Recovery Time Objective - Temps max acceptable de restauration |
| **RPO** | Recovery Point Objective - Perte de données max acceptable |
| **Failover** | Basculement automatique vers ressources de secours |
| **MFA** | Multi-Factor Authentication - Authentification multi-facteurs |
| **SSO** | Single Sign-On - Authentification unique pour plusieurs services |
| **RBAC** | Role-Based Access Control - Contrôle d'accès basé sur les rôles |
| **IAM** | Identity and Access Management - Gestion des identités et accès |
| **IaC** | Infrastructure as Code - Infrastructure décrite par du code |
| **Terraform** | Outil IaC multi-cloud (HashiCorp) |
| **ARM** | Azure Resource Manager - Outil IaC natif Azure |
| **CloudFormation** | Outil IaC natif AWS |
| **API** | Application Programming Interface - Interface de programmation |
| **CLI** | Command Line Interface - Interface en ligne de commande |
| **SDK** | Software Development Kit - Kit de développement logiciel |
| **SLA** | Service Level Agreement - Accord de niveau de service |
| **CAPEX** | Capital Expenditure - Investissement en capital (achat matériel) |
| **OPEX** | Operational Expenditure - Dépenses opérationnelles (abonnements) |
| **On-Demand** | À la demande - Tarification à l'usage sans engagement |
| **Reserved Instance** | Instance réservée - Engagement 1-3 ans avec réduction tarifaire |
| **Spot Instance** | Instance spot - VMs à prix réduit sur capacité disponible (AWS) |
| **Egress** | Trafic sortant du cloud (souvent facturé) |
| **Ingress** | Trafic entrant dans le cloud (généralement gratuit) |
| **Throttling** | Limitation volontaire de débit/performance |
| **RGPD** | Règlement Général sur la Protection des Données (GDPR en anglais) |
| **TLS** | Transport Layer Security - Protocole de chiffrement (ex-SSL) |
| **VPN** | Virtual Private Network - Réseau privé virtuel sécurisé |
| **CDN** | Content Delivery Network - Réseau de distribution de contenu |
| **DNS** | Domain Name System - Système de noms de domaine |
| **MX** | Mail eXchange - Enregistrement DNS pour serveurs mail |
| **SPF** | Sender Policy Framework - Validation des expéditeurs d'emails |
| **DKIM** | DomainKeys Identified Mail - Signature cryptographique des emails |
| **DMARC** | Domain-based Message Authentication - Politique d'authentification email |
| **Bare Metal** | Serveur physique dédié sans virtualisation |
| **VPS** | Virtual Private Server - Serveur privé virtuel |
| **Serverless** | Architecture sans gestion de serveurs (Functions as a Service) |
| **Docker** | Plateforme de conteneurisation d'applications |
| **Image Docker** | Template figé pour créer des conteneurs |
| **Dockerfile** | Fichier de configuration pour construire une image Docker |
| **OpenStack** | Plateforme open-source pour gérer des clouds privés/publics |

---

> [!quote] Citation finale
> "Le cloud n'est pas une destination, c'est un voyage continu d'optimisation et d'innovation." - Proverbe IT moderne

**Fin du document de révision**

---

**📚 Pour aller plus loin :**
- AWS Documentation : https://docs.aws.amazon.com/
- Microsoft Learn (Azure) : https://learn.microsoft.com/azure/
- Google Cloud Documentation : https://cloud.google.com/docs
- Kubernetes Documentation : https://kubernetes.io/docs/
- Terraform Documentation : https://www.terraform.io/docs
