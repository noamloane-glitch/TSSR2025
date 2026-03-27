## ⚡ L'essentiel en 5 minutes - Outils de l'Administrateur Système

### 📌 C'est quoi en 2 lignes ?
Ensemble des logiciels et interfaces permettant à un administrateur système de gérer, configurer et maintenir des serveurs et équipements réseau, principalement via des outils en ligne de commande (CLI) pour plus d'efficacité et de rapidité.

---

### 💡 Concepts clés à retenir :

* **Poste client** : Ordinateur individuel avec interface graphique (clavier/souris/écran), 1 utilisateur, allumé au besoin
* **Serveur** : Machine allumée 24/7, multi-utilisateurs simultanés, sans interface graphique, gérée à distance
* **Shell (CLI)** : Interface en ligne de commande pour communiquer directement avec l'OS sans GUI
* **Terminal** : Application permettant d'accéder au shell (émulateur de console)
* **Console système** : Interface d'accès direct au système (historiquement via terminal physique)
* **Switch KVM** : Dispositif permettant de contrôler plusieurs serveurs avec un seul clavier/écran/souris
* **Éditeur CLI** : Logiciel d'édition de fichiers en ligne de commande (nano, vim, emacs)

---

### 💻 Outils essentiels de l'admin :

```bash
# 🐧 Terminaux et émulateurs
xterm                              # Terminal X11 classique
terminator                         # Terminal avancé multi-onglets
screen                             # Multiplexeur de terminaux (sessions persistantes)
tmux                               # Multiplexeur moderne (alternative à screen)
```

```bash
# 🔐 Clients SSH (accès distant)
ssh user@serveur                   # Connexion SSH standard (OpenSSH)
ssh -p 2222 user@serveur           # SSH sur port custom
```

```powershell
# 🪟 Windows - Outils terminaux
putty.exe                          # Client SSH/Telnet Windows
MobaXterm                          # Suite complète SSH/X11/FTP pour Windows
```

```bash
# 📝 Éditeurs de texte CLI
nano fichier.txt                   # Éditeur simple et intuitif
vim fichier.txt                    # Éditeur puissant mais complexe
emacs fichier.txt                  # Éditeur extensible et complet
```

```bash
# 🌐 Accès console serveur
# Méthodes modernes d'accès :
# - Switch KVM (partage physique clavier/écran/souris)
# - Connexion série directe (RJ45 ou USB vers console)
# - SSH via réseau (le plus courant)
# - Interface web d'administration (IPMI, iLO, iDRAC)
```

---

### ⚠️ Pièges à éviter :

* ❌ **Utiliser GUI sur serveur** : Consomme ressources inutilement, pas toujours disponible, moins efficace que CLI
* ❌ **Oublier de sauvegarder avant édition** : Toujours copier fichier config avant modification (cp config config.bak)
* ❌ **Éditer directement en production sans test** : Tester commandes/scripts sur environnement de dev d'abord
* ❌ **Perdre session SSH critique** : Utiliser screen/tmux pour sessions persistantes (survivent aux déconnexions)
* ❌ **Négliger la documentation** : Noter commandes utilisées, configurations appliquées pour traçabilité
* ❌ **Utiliser nano/vim sans connaître les raccourcis** : Apprendre au moins les bases (sauver/quitter)

---

### ✅ Bonnes pratiques :

* ✅ **Maîtriser le shell** : CLI plus rapide, scriptable, consomme peu de ressources, essentiel pour serveurs
* ✅ **Utiliser SSH systématiquement** : Protocole sécurisé pour administration distante (jamais Telnet)
* ✅ **Documenter ses actions** : Tenir journal de bord des modifications (fichier log ou notes)
* ✅ **Avoir un laptop portable d'admin** : Ordinateur dédié avec tous les outils nécessaires
* ✅ **Apprendre un éditeur CLI avancé** : vim ou emacs pour efficacité maximale (après nano)
* ✅ **Utiliser screen/tmux en production** : Sessions persistantes évitent pertes en cas de déco réseau
* ✅ **Préférer console physique pour boot** : Accès direct via KVM/série pour débogage démarrage

---

### 📚 Vocabulaire technique :

| Terme | Définition courte |
|-------|------------------|
| **CLI** | Command-Line Interface - Interface en ligne de commande |
| **GUI** | Graphical User Interface - Interface graphique utilisateur |
| **Shell** | Interpréteur de commandes (bash, zsh, sh) |
| **Terminal** | Émulateur logiciel d'un terminal physique |
| **Console système** | Interface d'accès direct au système d'exploitation |
| **KVM** | Keyboard-Video-Mouse - Switch matériel pour contrôler plusieurs machines |
| **SSH** | Secure Shell - Protocole d'accès distant sécurisé |
| **Multiplexeur** | Logiciel permettant plusieurs sessions dans un terminal (screen, tmux) |
| **Émulateur** | Logiciel reproduisant comportement d'un terminal physique |
| **Hot-plug** | Connexion/déconnexion à chaud sans arrêt système |
| **RJ45** | Connecteur réseau Ethernet standard |
| **Série** | Port de communication série (RS-232/USB console) |
| **IPMI** | Intelligent Platform Management Interface - Gestion matériel à distance |
| **iLO/iDRAC** | Interfaces de gestion serveur HP/Dell (web + console virtuelle) |

---

### 🎯 À retenir ABSOLUMENT :

1. 💡 **Théorique** : Les serveurs n'ont généralement PAS d'interface graphique et sont administrés à distance via SSH/CLI
2. 💻 **Pratique** : `ssh user@serveur` pour se connecter, nano/vim pour éditer, screen pour session persistante
3. ⚠️ **Piège** : Perdre connexion SSH pendant modification critique = utiliser screen/tmux TOUJOURS

---

### 🖥️ Types d'ordinateurs - Comparaison :

| Critère | Poste client | Serveur |
|---------|--------------|---------|
| **Allumage** | Au besoin | 24/7/365 |
| **Utilisateurs** | 1 seul | Plusieurs simultanés |
| **Interface** | GUI (clavier/souris/écran) | CLI (sans périphériques) |
| **Emplacement** | Bureau utilisateur | Local technique (datacenter) |
| **Administration** | Locale | Distante (SSH/Web) |
| **Système** | Windows/MacOS/Linux | Principalement Linux/Windows Server |

---

### 🔧 Systèmes d'exploitation courants :

**Linux (serveurs) :**
- Debian / Ubuntu Server
- Red Hat / CentOS / Rocky Linux
- SUSE Linux Enterprise
- Arch Linux

**BSD (serveurs spécialisés) :**
- FreeBSD
- OpenBSD
- NetBSD

**Windows :**
- Windows 10/11 (postes clients)
- Windows Server 2019/2022 (serveurs)

**Autres :**
- macOS (postes clients)
- VMware ESXi / Proxmox (hyperviseurs)

---

### 🛠️ La boîte à outils complète de l'admin :

**Accès et connexion :**
- Terminaux : xterm, terminator, puTTY, MobaXterm
- SSH clients : OpenSSH, puTTY
- Multiplexeurs : screen, tmux

**Édition de fichiers :**
- Débutant : nano
- Avancé : vim, emacs
- Graphique distant : gedit, kate (via X11 forwarding)

**Réseau :**
- Diagnostic : ping, traceroute, mtr, netstat
- Analyse : tcpdump, wireshark, nmap
- Transfert : scp, rsync, sftp

**Navigation web :**
- Firefox, Chromium (pour interfaces web admin)

**Monitoring :**
- htop, top (processus)
- df, du (disques)
- iftop, nethogs (réseau)

---

### 📖 Méthodes d'accès serveur (historique → moderne) :

**Historique : Terminal physique**
- Connecté directement au serveur
- Clavier + écran dédié par machine

**Évolution 1 : Switch KVM**
- 1 clavier/écran/souris → N serveurs
- Commutation manuelle entre machines

**Évolution 2 : Console série**
- Connexion RJ45 ou USB → port console serveur
- Via émulateur terminal (minicom, screen, puTTY)

**Moderne : SSH (réseau)**
- Accès distant sécurisé
- Standard actuel pour administration

**Moderne : Interface web**
- IPMI, iLO, iDRAC
- Accès console virtuelle + gestion matériel

---

### 💾 Workflow typique admin serveur :

```bash
# 1. Se connecter au serveur
ssh admin@srv-prod-01

# 2. Utiliser screen pour session persistante
screen -S maintenance

# 3. Éditer fichier de configuration
sudo nano /etc/nginx/nginx.conf

# 4. Tester la configuration
sudo nginx -t

# 5. Recharger le service
sudo systemctl reload nginx

# 6. Vérifier les logs
tail -f /var/log/nginx/access.log

# 7. Détacher screen (Ctrl+A puis D)
# Session continue même si SSH déconnecté

# 8. Se reconnecter plus tard
screen -r maintenance
```

---

### 🎓 Courbe d'apprentissage éditeurs CLI :

**nano** → ⭐ Facile
- Raccourcis affichés en bas
- Idéal débutants
- Ctrl+O sauver, Ctrl+X quitter

**vim** → ⭐⭐⭐⭐ Difficile mais puissant
- Modes (normal, insertion, visuel)
- Très efficace une fois maîtrisé
- `:wq` sauver+quitter, `:q!` quitter sans sauver

**emacs** → ⭐⭐⭐⭐⭐ Très complexe mais extensible
- Écosystème complet (éditeur + IDE + plus)
- Courbe d'apprentissage abrupte
- Ctrl+X Ctrl+S sauver, Ctrl+X Ctrl+C quitter

---

### 🚀 Avantages CLI vs GUI :

| Critère | CLI | GUI |
|---------|-----|-----|
| **Vitesse d'exécution** | ⚡⚡⚡⚡⚡ Instantané | ⚡⚡ Plus lent |
| **Ressources** | 💾 Minimal (quelques Mo RAM) | 💾💾💾 Important (Go RAM) |
| **Efficacité** | 🎯 Maximum (scriptable) | 🎯 Limité (clics répétitifs) |
| **Courbe apprentissage** | 📈 Difficile au début | 📉 Facile intuitivement |
| **Accès distant** | 🌐 Léger (SSH = Ko/s) | 🌐🌐 Lourd (VNC/RDP = Mo/s) |
| **Automatisation** | ✅ Scripts, cron | ❌ Difficile |

---

### ⚙️ Configuration initiale poste admin :

```bash
# Installation outils essentiels Debian/Ubuntu
sudo apt update
sudo apt install -y openssh-client screen vim curl wget \
                     htop net-tools dnsutils tcpdump

# Installation MobaXterm Windows
# Télécharger depuis : https://mobaxterm.mobatek.net/

# Configuration SSH (clés plutôt que mots de passe)
ssh-keygen -t ed25519 -C "admin@entreprise.com"
ssh-copy-id user@serveur
```

---

### 🔐 Sécurité - Bonnes pratiques SSH :

```bash
# Ne JAMAIS utiliser root directement
# ❌ ssh root@serveur
# ✅ ssh admin@serveur, puis sudo

# Désactiver mot de passe, utiliser clés SSH
# Dans /etc/ssh/sshd_config :
# PasswordAuthentication no
# PubkeyAuthentication yes

# Changer port par défaut (optionnel)
# Port 2222

# Utiliser fail2ban contre brute force
sudo apt install fail2ban
```

---

### 📊 Checklist maintenance serveur :

- [ ] Connexion SSH établie avec clés
- [ ] Session screen/tmux lancée
- [ ] Sauvegarde config avant modif
- [ ] Test configuration après changement
- [ ] Vérification logs après action
- [ ] Documentation changements effectués
- [ ] Surveillance monitoring (CPU/RAM/disque)
- [ ] Validation accès services après reboot
