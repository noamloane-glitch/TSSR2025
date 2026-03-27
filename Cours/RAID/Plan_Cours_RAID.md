📘 PARTIE 1 - Introduction et concepts fondamentaux
Dossier Obsidian suggéré : `01-introduction-concepts-raid/`

**Sujets à couvrir :**

1. Définition et historique du RAID → `01-definition-historique.md`
   - Qu'est-ce que le RAID
   - RAID vs SLED (Single Large Expensive Disk)
   - Objectifs du RAID (performance, fiabilité, capacité)
   - Évolution depuis 1987 (Berkeley)

2. Terminologie essentielle → `02-terminologie.md`
   - Grappe RAID (array)
   - Volume/cluster
   - Disque de parité
   - Disque spare (réserve)
   - Hot-plug et hot-swap
   - Striping (entrelaçage)
   - Mirroring (mise en miroir)
   - Parity (parité)

3. Besoins et analyse → `03-analyse-besoins.md`
   - Volume de stockage nécessaire
   - Besoins en performance (débit, temps d'accès)
   - Niveau de sûreté requis
   - Compromis volume/performance/fiabilité

---

📘 PARTIE 2 - Types d'implémentation RAID
Dossier Obsidian suggéré : `02-implementations-raid/`

**Sujets à couvrir :**

1. RAID matériel → `01-raid-materiel.md`
   - Contrôleur RAID dédié (carte)
   - Avantages (performance, pas de CPU consommé, boot possible)
   - Inconvénients (coût, compatibilité)
   - Configuration via BIOS/firmware
   - Usage en entreprise

2. RAID logiciel → `02-raid-logiciel.md`
   - Fonctionnalité de l'OS
   - Outil mdadm sous Linux
   - Gestionnaire de disques Windows
   - Avantages (souplesse, compatibilité, gratuit)
   - Inconvénients (consommation CPU, boot complexe)

3. RAID hybride → `03-raid-hybride.md`
   - Définition et fonctionnement
   - Contrôleur physique + pilote logiciel
   - Avantages et inconvénients
   - Cas d'usage (cartes mères grand public)

---

📘 PARTIE 3 - Niveaux de RAID standards
Dossier Obsidian suggéré : `03-niveaux-raid-standards/`

**Sujets à couvrir :**

1. JBOD et RAID 0 → `01-jbod-raid0.md`
   - JBOD : Just a Bunch Of Disks
   - RAID 0 : Striping (entrelaçage)
   - Principe de fonctionnement
   - Capacité totale
   - Performance
   - Tolérance aux pannes
   - Cas d'usage

2. RAID 1 → `02-raid1.md`
   - Mirroring (disques miroirs)
   - Principe de fonctionnement
   - Capacité totale
   - Performance
   - Tolérance aux pannes
   - Nombre de disques minimum
   - Cas d'usage

3. RAID 4 → `03-raid4.md`
   - Agrégation par bande avec parité
   - Principe de fonctionnement (parité XOR)
   - Disque de parité dédié
   - Capacité totale
   - Performance (goulot d'étranglement parité)
   - Tolérance aux pannes
   - Nombre de disques minimum

4. RAID 5 → `04-raid5.md`
   - Agrégation avec parité répartie
   - Principe de fonctionnement (round-robin)
   - Répartition de la parité
   - Capacité totale
   - Performance
   - Tolérance aux pannes
   - Nombre de disques minimum
   - Cas d'usage (compromis idéal)

5. RAID 6 → `05-raid6.md`
   - Double parité
   - Principe de fonctionnement
   - Capacité totale
   - Performance
   - Tolérance aux pannes (2 disques)
   - Nombre de disques minimum
   - Reconstruction plus longue
   - Cas d'usage (haute disponibilité)

---

📘 PARTIE 4 - Niveaux de RAID combinés
Dossier Obsidian suggéré : `04-raid-combines/`

**Sujets à couvrir :**

1. RAID 0+1 (RAID 01) → `01-raid01.md`
   - Principe : RAID 1 de grappes RAID 0
   - Schéma architecture
   - Capacité totale
   - Performance
   - Tolérance aux pannes
   - Avantages et inconvénients

2. RAID 1+0 (RAID 10) → `02-raid10.md`
   - Principe : RAID 0 de grappes RAID 1
   - Schéma architecture
   - Capacité totale
   - Performance
   - Tolérance aux pannes (meilleure que RAID 01)
   - Nombre de disques minimum
   - Cas d'usage (performance + fiabilité)

3. Autres combinaisons → `03-autres-combinaisons.md`
   - RAID 50 (RAID 5+0)
   - RAID 60 (RAID 6+0)
   - Cas d'usage spécifiques

---

📘 PARTIE 5 - Gestion et administration
Dossier Obsidian suggéré : `05-gestion-administration/`

**Sujets à couvrir :**

1. Configuration et création → `01-configuration-creation.md`
   - Prérequis (disques identiques recommandés)
   - Création d'une grappe RAID matérielle
   - Création d'une grappe RAID logicielle (mdadm)
   - Vérification de l'état

2. Surveillance et monitoring → `02-surveillance-monitoring.md`
   - Vérification état RAID (/proc/mdstat)
   - Commandes de diagnostic
   - Surveillance des disques (SMART)
   - Alertes et notifications

3. Gestion des pannes → `03-gestion-pannes.md`
   - Détection d'une panne disque
   - Procédure de remplacement
   - Hot-plug et hot-swap
   - Reconstruction automatique
   - Temps de reconstruction
   - Risques durant reconstruction

4. Disques de spare → `04-disques-spare.md`
   - Définition et rôle
   - Configuration d'un spare
   - Reconstruction automatique
   - Hot spare vs cold spare

---

📘 PARTIE 6 - Calculs et dimensionnement
Dossier Obsidian suggéré : `06-calculs-dimensionnement/`

**Sujets à couvrir :**

1. Calculs de capacité → `01-calculs-capacite.md`
   - Formule RAID 0 : n × taille
   - Formule RAID 1 : taille du plus petit
   - Formule RAID 5 : (n-1) × taille
   - Formule RAID 6 : (n-2) × taille
   - Formule RAID 10 : (n/2) × taille
   - Exemples concrets

2. Calculs de performance → `02-calculs-performance.md`
   - Débit théorique en lecture
   - Débit théorique en écriture
   - Impact du niveau de RAID
   - Exemples de calculs

3. Dimensionnement optimal → `03-dimensionnement-optimal.md`
   - Choix du nombre de disques
   - Choix de la taille des disques
   - Impact sur les performances
   - Impact sur les coûts
   - Recommandations par cas d'usage

---

📘 PARTIE 7 - Bonnes pratiques et recommandations
Dossier Obsidian suggéré : `07-bonnes-pratiques/`

**Sujets à couvrir :**

1. Choix du matériel → `01-choix-materiel.md`
   - Utiliser disques identiques (modèle, capacité)
   - Éviter mélange HDD/SSD
   - Importance de la qualité des disques
   - Contrôleur RAID avec cache batterie (BBU)

2. Choix du niveau RAID → `02-choix-niveau-raid.md`
   - RAID 0 : performance pure (données temporaires)
   - RAID 1 : données critiques (faible volumétrie)
   - RAID 5 : compromis idéal (serveurs)
   - RAID 6 : haute disponibilité (gros volumes)
   - RAID 10 : performance + fiabilité (bases de données)

3. Sécurité et sauvegardes → `03-securite-sauvegardes.md`
   - RAID ≠ sauvegarde
   - Nécessité de sauvegardes externes
   - Protection contre corruption données
   - Protection contre erreurs humaines
   - Stratégie 3-2-1

4. Maintenance préventive → `04-maintenance-preventive.md`
   - Vérification régulière état RAID
   - Tests de reconstruction
   - Remplacement disques préventif
   - Mise à jour firmware contrôleur
   - Documentation configuration

---

📘 PARTIE 8 - Cas pratiques et troubleshooting
Dossier Obsidian suggéré : `08-pratique-troubleshooting/`

**Sujets à couvrir :**

1. Commandes mdadm essentielles → `01-commandes-mdadm.md`
   - Créer une grappe RAID
   - Afficher l'état
   - Ajouter un disque
   - Retirer un disque
   - Marquer un disque défaillant
   - Arrêter/démarrer une grappe

2. Scénarios de panne → `02-scenarios-panne.md`
   - Panne 1 disque RAID 5 (récupération)
   - Panne 2 disques RAID 5 (perte données)
   - Panne 1 disque RAID 1 (récupération)
   - Panne durant reconstruction
   - Procédures de récupération

3. Problèmes courants → `03-problemes-courants.md`
   - Grappe dégradée après reboot
   - Disque marqué défaillant à tort
   - Reconstruction bloquée
   - Performances dégradées
   - Solutions et diagnostic

---

📘 PARTIE 9 - RAID et virtualisation
Dossier Obsidian suggéré : `09-raid-virtualisation/`

**Sujets à couvrir :**

1. RAID pour hyperviseurs → `01-raid-hyperviseurs.md`
   - Recommandations VMware (RAID 5/6/10)
   - Recommandations Proxmox
   - Recommandations Hyper-V
   - Stockage datastore

2. RAID vs stockage logiciel → `02-raid-vs-stockage-logiciel.md`
   - RAID matériel vs ZFS
   - RAID matériel vs Ceph
   - RAID matériel vs LVM
   - Avantages et inconvénients
   - Quand utiliser quoi

3. Machines virtuelles et RAID → `03-vm-raid.md`
   - Impact sur performances VM
   - Configuration optimale
   - Séparation OS/données
   - Snapshots et RAID
