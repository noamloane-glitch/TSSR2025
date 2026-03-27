# Plan de Cours : Netfilter (Formation TSSR)

📘 **PARTIE 1 - Fondamentaux de Netfilter**  
Dossier Obsidian suggéré : `01-fondamentaux-netfilter/`

**Sujets à couvrir :**

1. Introduction à Netfilter → `01-introduction-netfilter.md`
   - Définition et rôle de Netfilter dans le noyau Linux
   - Historique et évolution
   - Architecture générale
   - Relation avec iptables

2. Architecture et composants → `02-architecture-composants.md`
   - Les tables (filter, nat, mangle, raw)
   - Les chaînes (PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING)
   - Le cheminement des paquets
   - Les hooks du noyau

---

📘 **PARTIE 2 - Filtrage de paquets**  
Dossier Obsidian suggéré : `02-filtrage-paquets/`

**Sujets à couvrir :**

1. Table filter et règles de base → `01-table-filter.md`
   - Structure de la table filter
   - Chaînes INPUT, OUTPUT, FORWARD
   - Syntaxe des règles
   - Critères de correspondance (source, destination, protocole, port)

2. Actions et cibles → `02-actions-cibles.md`
   - ACCEPT, DROP, REJECT
   - LOG et suivi des paquets
   - Cibles personnalisées
   - Politique par défaut

3. Règles avancées de filtrage → `03-regles-avancees.md`
   - Correspondance d'états (state/conntrack)
   - Correspondance par interface
   - Correspondance par plage d'adresses
   - Modules de correspondance courants

---

📘 **PARTIE 3 - Translation d'adresses (NAT)**  
Dossier Obsidian suggéré : `03-nat/`

**Sujets à couvrir :**

1. Principes du NAT → `01-principes-nat.md`
   - Définition et utilité du NAT
   - Types de NAT (SNAT, DNAT, Masquerading)
   - Table nat et chaînes associées
   - Cas d'usage courants

2. Configuration SNAT et Masquerading → `02-snat-masquerading.md`
   - SNAT pour adresses fixes
   - Masquerading pour adresses dynamiques
   - Syntaxe et exemples
   - Scénarios pratiques

3. Configuration DNAT → `03-dnat.md`
   - Redirection de ports
   - Publication de services
   - Syntaxe et exemples
   - Scénarios pratiques

---

📘 **PARTIE 4 - Gestion et administration**  
Dossier Obsidian suggéré : `04-gestion-administration/`

**Sujets à couvrir :**

1. Commandes iptables → `01-commandes-iptables.md`
   - Lister les règles (-L, -S)
   - Ajouter des règles (-A, -I)
   - Supprimer des règles (-D, -F)
   - Sauvegarde et restauration

2. Persistance des règles → `02-persistance-regles.md`
   - iptables-save et iptables-restore
   - Fichiers de configuration système
   - Services et automatisation
   - Différences selon les distributions

3. Sécurisation et bonnes pratiques → `03-securisation-bonnes-pratiques.md`
   - Politique restrictive par défaut
   - Règles anti-spoofing
   - Protection contre les attaques courantes
   - Organisation des règles
   - Documentation et maintenance

---

📘 **PARTIE 5 - Introduction à nftables**  
Dossier Obsidian suggéré : `05-nftables/`

**Sujets à couvrir :**

1. Présentation de nftables → `01-presentation-nftables.md`
   - Qu'est-ce que nftables
   - Historique et motivation
   - Différences architecturales avec iptables
   - Avantages et inconvénients

2. Syntaxe et concepts de base → `02-syntaxe-concepts-base.md`
   - Structure des commandes nft
   - Tables et chaînes dans nftables
   - Familles d'adresses (ip, ip6, inet, arp, bridge)
   - Types de chaînes (filter, nat, route)

3. Règles et correspondances → `03-regles-correspondances.md`
   - Syntaxe des règles
   - Expressions de correspondance
   - Verdicts et actions
   - Sets et maps

4. Configuration NAT avec nftables → `04-nat-nftables.md`
   - SNAT et masquerading
   - DNAT et redirection de ports
   - Syntaxe et exemples pratiques
   - Comparaison avec iptables

5. Migration depuis iptables → `05-migration-iptables.md`
   - Outils de conversion (iptables-translate)
   - Méthodologie de migration
   - Cohabitation iptables/nftables
   - Exemples de conversion

---

📘 **PARTIE 6 - Mise en pratique**  
Dossier Obsidian suggéré : `06-mise-en-pratique/`

**Sujets à couvrir :**

1. Scénarios avec iptables → `01-scenarios-iptables.md`
   - Pare-feu simple pour serveur
   - Pare-feu pour passerelle réseau
   - Pare-feu DMZ
   - Règles pour services courants (SSH, HTTP, DNS)

2. Scénarios avec nftables → `02-scenarios-nftables.md`
   - Configuration équivalente en nftables
   - Pare-feu serveur moderne
   - Routeur avec nftables
   - Scripts de configuration

3. Dépannage et diagnostic → `03-depannage-diagnostic.md`
   - Analyse des règles actives (iptables et nftables)
   - Logs et journalisation
   - Outils de test (ping, telnet, nmap)
   - Méthodologie de résolution de problèmes
   - Débogage spécifique nftables
