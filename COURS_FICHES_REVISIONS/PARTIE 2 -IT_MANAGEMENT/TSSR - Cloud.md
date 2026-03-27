## ⚡ L'essentiel en 5 minutes - Cloud Computing

### 📌 C'est quoi en 2 lignes ?
Fourniture de services informatiques (serveurs, stockage, bases de données, réseau, logiciels) via Internet avec paiement à l'usage. Permet flexibilité, évolutivité et réduction des coûts d'infrastructure tout en déléguant maintenance et gestion à un fournisseur cloud.

---

### 💡 Concepts clés à retenir :

* **IaaS (Infrastructure as a Service)** : Location ressources virtualisées (serveurs, stockage, réseau) - Ex: AWS EC2, Azure VM
* **PaaS (Platform as a Service)** : Environnement développement/déploiement d'applications clé en main - Ex: Azure App Service, Heroku
* **SaaS (Software as a Service)** : Logiciels en ligne via abonnement sans installation - Ex: Office 365, Google Workspace, Salesforce
* **Cloud Public** : Ressources partagées multi-tenants via Internet (AWS, Azure, GCP) - évolutif et économique
* **Cloud Privé** : Infrastructure dédiée à une seule organisation - contrôle et sécurité max
* **Cloud Hybride** : Combinaison public + privé avec portabilité données/applications
* **Hyperviseur Type 1 (Bare Metal)** : S'exécute directement sur matériel (ESXi, Hyper-V, Proxmox) - datacenter cloud
* **Hyperviseur Type 2 (Hosted)** : S'exécute sur OS hôte (VirtualBox, VMware Workstation) - tests/dev
* **Scalabilité verticale (Scale Up/Down)** : Augmenter/réduire capacité serveur existant (CPU/RAM)
* **Scalabilité horizontale (Scale Out/In)** : Ajouter/supprimer serveurs pour ajuster ressources
* **Elasticité** : Ajustement automatique ressources selon demande en temps réel

---

### 💻 Commandes et configurations essentielles :

```bash
# ☁️ AWS CLI (Amazon Web Services)
aws configure                              # Configurer credentials AWS
aws ec2 describe-instances                 # Lister instances EC2
aws ec2 start-instances --instance-ids i-xxx  # Démarrer instance
aws ec2 stop-instances --instance-ids i-xxx   # Arrêter instance
aws s3 ls s3://mon-bucket/                 # Lister contenu bucket S3
aws s3 cp fichier.txt s3://bucket/         # Upload fichier S3

# ☁️ Azure CLI
az login                                   # Connexion Azure
az vm list                                 # Lister VMs
az vm start --name vm-prod --resource-group rg-prod  # Démarrer VM
az vm stop --name vm-prod --resource-group rg-prod   # Arrêter VM
az storage blob list --account-name compte --container-name conteneur  # Lister blobs

# ☁️ Google Cloud (gcloud)
gcloud auth login                          # Connexion GCP
gcloud compute instances list              # Lister instances
gcloud compute instances start instance-1  # Démarrer instance
gcloud compute instances stop instance-1   # Arrêter instance
```

```bash
# 🐧 Docker (conteneurisation)
docker pull nginx:latest                   # Télécharger image
docker images                              # Lister images locales
docker run -d -p 80:80 nginx              # Lancer conteneur en background
docker ps                                  # Lister conteneurs actifs
docker ps -a                               # Lister tous conteneurs
docker stop <container_id>                 # Arrêter conteneur
docker rm <container_id>                   # Supprimer conteneur
docker logs <container_id>                 # Voir logs conteneur

# Dockerfile exemple
FROM python:3.11
COPY app.py /app/
RUN pip install flask
CMD ["python", "/app/app.py"]
```

```bash
# 🏗️ Terraform (Infrastructure as Code)
terraform init                             # Initialiser projet
terraform plan                             # Prévisualiser changements
terraform apply                            # Appliquer infrastructure
terraform destroy                          # Détruire infrastructure

# Exemple fichier main.tf (Azure VM)
resource "azurerm_virtual_machine" "vm" {
  name     = "vm-prod-web"
  location = "France Central"
  size     = "Standard_B2s"
}
```

```powershell
# 🪟 PowerShell Azure
Connect-AzAccount                          # Connexion Azure
Get-AzVM                                   # Lister VMs
Start-AzVM -Name "vm-prod" -ResourceGroupName "rg-prod"    # Démarrer VM
Stop-AzVM -Name "vm-prod" -ResourceGroupName "rg-prod"     # Arrêter VM
```

```bash
# 🌐 Diagnostic cloud
# Test connectivité VM
ping <ip_publique>                         # Test ICMP
ssh user@<ip_publique>                     # Test SSH (Linux)
telnet <ip> 3389                           # Test RDP (Windows)

# Vérifier DNS (messagerie M365)
nslookup -type=MX domaine.fr               # Enregistrement MX
nslookup -type=TXT domaine.fr              # SPF/DKIM
dig domaine.fr MX                          # Alternative Linux
```

---

### 📐 Modèles tarifaires :

**On-Demand (à la demande) :**
- Facturation heure/seconde usage réel
- Prix plein, flexibilité maximale
- Usage : Tests, charges imprévisibles
- Ex: 0,10 €/h ou 0,000027778 €/s

**Reserved Instances (instances réservées) :**
- Engagement 1-3 ans
- Réduction -30% à -70% vs On-Demand
- Usage : Serveurs production permanents
- Ex: Payer 3000€/an au lieu de 876€/mois

**Stockage S3 (AWS) :**
```
Coût stockage : 0,02 €/Go/mois (Standard)
Requêtes GET   : 0,0004 € / 1000 requêtes
Requêtes PUT   : 0,005 € / 1000 requêtes
Transfert IN   : Gratuit
Transfert OUT  : 0,09 €/Go (Egress facturé !)
```

**Exemple calcul mensuel :**
```
VM Standard_B2s (2 vCPU, 4Go RAM) Azure
On-Demand    : 730h × 0,10€ = 73€/mois
Reserved 1an : 43,80€/mois (-40%)
Reserved 3ans: 29,20€/mois (-60%)
```

---

### ⚠️ Pièges à éviter :

* ❌ **Oublier d'éteindre VMs de dev** : Facturation 24/7 même si inutilisées (-60% coûts possibles)
* ❌ **Sous-estimer coûts transfert sortant (Egress)** : Sortie de données cloud très chère (0,09€/Go)
* ❌ **Ne pas dimensionner correctement (Right-Sizing)** : VM surdimensionnée = gaspillage budgétaire
* ❌ **Supprimer ligne IaC sans réfléchir** : Terraform/ARM = destruction ressources réelles !
* ❌ **Cloud = sauvegarde automatique** : FAUX ! Toujours faire sauvegardes séparées même en cloud
* ❌ **Ignorer localisation données RGPD** : Données sensibles doivent rester UE (conformité légale)
* ❌ **Pas de MFA sur comptes admin cloud** : Risque compromission critique de l'infrastructure entière
* ❌ **Dépendance vis-à-vis unique fournisseur (Vendor Lock-in)** : Migration complexe/coûteuse vers autre cloud
* ❌ **Négliger monitoring coûts** : Facture surprise en fin de mois (alertes budgétaires essentielles)
* ❌ **Laisser IP publiques inutilisées** : Facturation même si non attachées à ressources

---

### ✅ Bonnes pratiques :

* ✅ **Activer MFA obligatoire tous comptes admin** : Protection contre vol credentials (2FA app > SMS)
* ✅ **Appliquer principe moindre privilège (RBAC)** : Donner uniquement droits nécessaires par rôle
* ✅ **Éteindre automatiquement VMs dev hors heures** : Scripts automatisation (-60% facture)
* ✅ **Configurer alertes budgétaires** : Notification si dépassement seuil dépenses
* ✅ **Taguer toutes ressources** : Identifier propriétaire, projet, environnement (prod/dev/test)
* ✅ **Nettoyer ressources orphelines** : Disques non attachés, snapshots obsolètes, IPs non utilisées
* ✅ **Utiliser Reserved Instances pour prod** : Économies importantes (-40% à -70%) sur charges stables
* ✅ **Activer accès conditionnel** : Bloquer connexions depuis pays suspects, exiger MFA hors VPN
* ✅ **Répliquer géographiquement données critiques** : Protection sinistre (PRA/PCA)
* ✅ **Versionner infrastructure IaC dans Git** : Traçabilité, rollback, collaboration équipe

---

### 📚 Vocabulaire technique :

| Terme | Définition courte |
|-------|------------------|
| **IaaS** | Infrastructure as a Service - Location ressources virtualisées (VM, stockage, réseau) |
| **PaaS** | Platform as a Service - Environnement dev/déploiement clé en main |
| **SaaS** | Software as a Service - Logiciels en ligne par abonnement |
| **On-premises** | Infrastructure locale gérée en interne (vs cloud) |
| **Multi-tenant** | Architecture où ressources partagées entre plusieurs clients (cloud public) |
| **Single-tenant** | Infrastructure dédiée à un seul client (cloud privé) |
| **VPS (Virtual Private Server)** | Serveur virtuel dédié sur serveur physique partagé |
| **Bare Metal** | Serveur physique dédié complet (pas virtualisé) |
| **Block Storage** | Stockage bloc attaché à VM (disque virtuel pour OS/BDD) - haute performance |
| **Object Storage** | Stockage fichiers non structuré (S3, Blob) - photos, vidéos, backups |
| **Egress** | Transfert données sortant du cloud (facturé) vs Ingress (gratuit) |
| **Instance** | Machine virtuelle en cours d'exécution dans cloud |
| **Snapshot** | Image instantanée état VM/disque à instant T |
| **Auto-scaling** | Ajustement automatique nombre d'instances selon charge |
| **Load Balancer** | Répartiteur de charge entre plusieurs serveurs |
| **Availability Zone** | Datacenter physiquement séparé dans même région (redondance) |
| **Region** | Zone géographique contenant plusieurs datacenters |
| **RTO (Recovery Time Objective)** | Temps max acceptable pour restaurer service après incident |
| **RPO (Recovery Point Objective)** | Perte données max acceptable (intervalle entre sauvegardes) |
| **Failover** | Bascule automatique vers infrastructure secondaire en cas panne |
| **RBAC (Role-Based Access Control)** | Gestion droits basée sur rôles prédéfinis |
| **MFA (Multi-Factor Authentication)** | Authentification multi-facteurs (mot de passe + code/token) |
| **SSO (Single Sign-On)** | Authentification unique pour accéder à multiples services |
| **IaC (Infrastructure as Code)** | Description infrastructure via code (Terraform, ARM, CloudFormation) |
| **Kubernetes (K8s)** | Orchestrateur conteneurs multi-machines (déploiement, scaling automatique) |
| **Container** | Instance exécution d'une image Docker (application isolée) |
| **Image** | Template figé pour créer conteneur (ex: nginx:latest) |
| **Dockerfile** | Fichier instructions pour construire image Docker |

---

### 🎯 À retenir ABSOLUMENT :

1. 💡 **Théorique** : IaaS = Infra virtualisée | PaaS = Plateforme dev | SaaS = Logiciel en ligne (responsabilités croissantes fournisseur)
2. 💻 **Pratique** : Éteindre VMs dev inutilisées + Reserved Instances prod + alertes budgétaires = économies massives
3. ⚠️ **Piège** : Cloud ≠ sauvegarde automatique → TOUJOURS sauvegarder séparément même données cloud !

---

### ☁️ Principaux fournisseurs cloud :

**AWS (Amazon Web Services) :**
- Leader marché (~32%)
- Services : EC2 (VM), S3 (stockage), RDS (BDD), Lambda (serverless)
- Régions : 30+ régions mondiales
- CLI : `aws` command

**Microsoft Azure :**
- 2ème marché (~23%)
- Services : Virtual Machines, Blob Storage, SQL Database, Functions
- Intégration parfaite Microsoft (AD, Office 365, Windows)
- CLI : `az` command

**Google Cloud Platform (GCP) :**
- 3ème marché (~10%)
- Services : Compute Engine, Cloud Storage, BigQuery (analytics)
- Excellence IA/ML et analytics
- CLI : `gcloud` command

**Autres :**
- **OVHcloud** : Français, RGPD-friendly, tarifs compétitifs
- **Oracle Cloud** : Spécialisé bases de données
- **IBM Cloud** : Enterprise, IA Watson
- **DigitalOcean** : Simple, développeurs/startups

---

### 🔐 Partage responsabilités (Shared Responsibility Model) :

```
                    On-premises    IaaS      PaaS      SaaS
                    -----------    ----      ----      ----
Applications        Client         Client    Client    Fournisseur
Data               Client         Client    Client    Partagé
Runtime            Client         Client    Fournisseur  Fournisseur
Middleware         Client         Client    Fournisseur  Fournisseur
OS                 Client         Client    Fournisseur  Fournisseur
Virtualisation     Client         Fournisseur  Fournisseur  Fournisseur
Serveurs           Client         Fournisseur  Fournisseur  Fournisseur
Stockage           Client         Fournisseur  Fournisseur  Fournisseur
Réseau             Client         Fournisseur  Fournisseur  Fournisseur
```

**⚠️ Important :** Même en SaaS, client TOUJOURS responsable :
- Sauvegardes données
- Contrôle d'accès utilisateurs
- Configuration sécurité (MFA, accès conditionnel)

---

### 📊 Services cloud conteneurs :

**Orchestration conteneurs managée :**
- **AKS** (Azure Kubernetes Service) - Azure
- **EKS** (Elastic Kubernetes Service) - AWS
- **GKE** (Google Kubernetes Engine) - GCP

**Conteneurs serverless (sans gestion serveur) :**
- **Azure Container Instances (ACI)** - Démarrage rapide conteneur isolé
- **AWS Fargate** - Exécution conteneurs sans gérer EC2
- **Google Cloud Run** - Déploiement API conteneur en 1 clic

**Comparaison Docker Swarm vs Kubernetes :**
| Critère | Docker Swarm | Kubernetes |
|---------|--------------|------------|
| **Simplicité** | ✅ Simple | ⚠️ Complexe |
| **Scalabilité** | ⚠️ Limitée | ✅ Massive |
| **Écosystème** | ⚠️ Restreint | ✅ Énorme |
| **Adoption** | ⚠️ Déclin | ✅ Standard industrie |

---

### 🛡️ Sécurité et conformité cloud :

**Certifications essentielles :**
- **ISO 27001** : Management sécurité information (international)
- **SOC 2 Type II** : Contrôles disponibilité, confidentialité, intégrité
- **HDS** (Hébergement Données Santé) - Obligatoire données médicales France
- **PCI-DSS** : Obligatoire si traitement cartes bancaires
- **FedRAMP** : Certif gouvernement US

**RGPD (Cloud) :**
```bash
Obligations :
✅ Localisation données UE pour données sensibles
✅ Clauses contractuelles types avec fournisseur
✅ Analyse d'impact (DPIA) si risque élevé
✅ Notification CNIL si transfert hors UE
✅ Droit accès, rectification, effacement garantis

Régions conformes RGPD :
- Azure : France Central, West Europe, North Europe
- AWS : eu-west-3 (Paris), eu-central-1 (Francfort)
- GCP : europe-west1 (Belgique), europe-west9 (Paris)
```

**Chiffrement cloud :**
- **Chiffrement au repos** : Données stockées (AES-256)
- **Chiffrement en transit** : TLS 1.2+ obligatoire
- **Chiffrement client-side** : Données chiffrées AVANT envoi cloud (max sécurité)
- **Key Management** : Azure Key Vault, AWS KMS, GCP KMS

---

### 🔧 Diagnostic problèmes cloud courants :

**Problème 1 : "Je ne peux pas me connecter à ma VM"**
```bash
# Checklist diagnostic :
1. Vérifier IP publique assignée
   az vm show --name vm-prod --resource-group rg-prod --show-details

2. Tester connectivité réseau
   ping <ip_publique>
   telnet <ip> 22    # SSH Linux
   telnet <ip> 3389  # RDP Windows

3. Vérifier groupe sécurité (firewall cloud)
   - Port SSH 22 ouvert ? (Linux)
   - Port RDP 3389 ouvert ? (Windows)
   - IP source autorisée ?

4. Vérifier VM démarrée
   az vm get-instance-view --name vm-prod --resource-group rg-prod

5. Vérifier credentials
   - Clé SSH correcte ? (Linux)
   - Mot de passe correct ? (Windows)
```

**Problème 2 : "Mon application est lente"**
```bash
# Diagnostic performance :
1. Consulter métriques VM
   - CPU > 80% ? → Scale Up ou Scale Out
   - RAM saturée ? → Augmenter taille instance
   - Disk IOPS limités ? → Passer à SSD Premium

2. Vérifier bande passante réseau
   - Throttling (limitation) fournisseur ?
   - Latence élevée ? (ping, traceroute)

3. Analyser logs applicatifs
   - Azure Monitor / CloudWatch / Stackdriver
   - Erreurs SQL ? Requêtes lentes ?

4. Vérifier connexions BDD
   - Pool connexions saturé ?
   - BDD sous-dimensionnée ?
```

**Problème 3 : "Email ne part pas depuis M365"**
```bash
# Diagnostic messagerie :
1. Vérifier configuration DNS
   nslookup -type=MX domaine.fr
   nslookup -type=TXT domaine.fr  # SPF, DKIM

2. Vérifier enregistrements corrects
   SPF : "v=spf1 include:spf.protection.outlook.com ~all"
   MX  : domaine-fr.mail.protection.outlook.com

3. Vérifier Centre de messages M365
   - Incident en cours ?
   - Maintenance planifiée ?

4. Tester envoi depuis portail web
   - Fonctionne dans Outlook Web ? → Problème client local
   - Ne fonctionne pas ? → Problème Microsoft 365

5. Vérifier boîte bloquée/quotas
   - Boîte pleine ?
   - Compte suspendu ?
```

---

### 💰 Optimisation coûts cloud :

**Stratégies économies :**

1. **Auto-shutdown VMs dev/test**
```bash
# Azure - Arrêt automatique 19h, démarrage 8h
az vm auto-shutdown --name vm-dev --time 1900

# AWS - Lambda function scheduled (EventBridge)
# Script arrêt instances tagged "Environment=Dev" à 19h
```

2. **Right-Sizing (dimensionnement correct)**
```bash
# Analyser utilisation réelle 30 derniers jours
# Si CPU moyen < 20% → downsizing
# Standard_D4s_v3 (4vCPU, 16Go) → Standard_D2s_v3 (2vCPU, 8Go)
# Économie : ~120€/mois
```

3. **Nettoyer ressources orphelines**
```bash
# Disques non attachés
az disk list --query "[?managedBy==null]"

# Snapshots > 30 jours
az snapshot list --query "[?timeCreated<'2025-01-01']"

# IP publiques non assignées (facturées !)
az network public-ip list --query "[?ipConfiguration==null]"
```

4. **Utiliser tiers de stockage adapté**
```
Standard (accès fréquent)   : 0,02 €/Go/mois
Cool (accès rare)           : 0,01 €/Go/mois
Archive (archivage long terme) : 0,002 €/Go/mois

Migration auto via lifecycle policies
```

5. **Reserved Instances production**
```
Serveur production 24/7 :
On-Demand     : 876 €/mois × 12 = 10 512 €/an
Reserved 1an  : 525 €/mois × 12 = 6 300 €/an (-40%)
Reserved 3ans : 350 €/mois × 12 = 4 200 €/an (-60%)

Économie : 6 312 € sur 3 ans !
```

---

### 📋 Checklist migration vers cloud :

**Phase préparation :**
- [ ] Inventaire complet infrastructure actuelle
- [ ] Identification dépendances applications
- [ ] Évaluation besoins performance (CPU, RAM, stockage, réseau)
- [ ] Estimation coûts cloud vs on-premises (TCO)
- [ ] Choix fournisseur cloud (Azure, AWS, GCP, OVH)
- [ ] Définition stratégie migration (Lift&Shift, Refactoring, Hybrid)

**Phase sécurité/conformité :**
- [ ] Analyse conformité RGPD (localisation données)
- [ ] Vérification certifications fournisseur (ISO 27001, SOC 2, HDS)
- [ ] Définition politique sécurité (MFA, RBAC, chiffrement)
- [ ] Configuration VPN site-to-site (cloud hybride)
- [ ] Mise en place monitoring et alertes sécurité

**Phase technique :**
- [ ] Création architecture réseau cloud (VNet, subnets, NSG)
- [ ] Configuration accès (VPN, ExpressRoute/Direct Connect)
- [ ] Déploiement environnement test (POC)
- [ ] Migration données (Azure Data Box, AWS Snowball, rsync)
- [ ] Tests performance et connectivité
- [ ] Configuration sauvegarde/disaster recovery

**Phase déploiement :**
- [ ] Migration applications non-critiques (pilote)
- [ ] Validation fonctionnelle utilisateurs
- [ ] Migration progressive applications critiques
- [ ] Formation équipes IT et utilisateurs
- [ ] Documentation architecture et procédures
- [ ] Décommissionnement infrastructure on-premises

**Phase opérationnelle :**
- [ ] Monitoring 24/7 (métriques, alertes, logs)
- [ ] Optimisation coûts continue (right-sizing, reserved instances)
- [ ] Sauvegardes régulières testées
- [ ] Mises à jour sécurité automatisées
- [ ] Revue trimestrielle architecture et coûts
