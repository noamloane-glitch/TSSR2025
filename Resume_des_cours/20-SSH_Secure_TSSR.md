# Secure Shell (SSH)
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : SSH - Connexion sécurisée à distance  
**Date** : Novembre 2024  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Authentification SSH|Authentification SSH]]
3. [[#Usages et configuration|Usages et configuration]]
4. [[#Fail2Ban|Fail2Ban]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]
7. [[#📖 Références externes|Références externes]]

---

## Introduction

> [!abstract] Vue d'ensemble
> **SSH (Secure Shell)** est un protocole de communication sécurisé permettant d'établir une connexion chiffrée entre deux machines. Il remplace les anciens protocoles non sécurisés (Telnet, RSH, FTP) et assure **confidentialité, authentification bidirectionnelle et intégrité** des échanges.

### Historique - Avant SSH

> [!warning] Protocoles non sécurisés

**Premiers systèmes d'information** : Émergence de piles protocolaires visant l'échange de données entre machines.

**Protocoles historiques** :

| Protocole | Usage | Problèmes de sécurité |
|-----------|-------|----------------------|
| **Telnet** | Terminal à distance | ❌ Authentification en clair |
| **RSH** | Remote Shell | ❌ Pas d'authentification robuste |
| **FTP** | Transfert de fichiers | ❌ Identifiants en clair |

**Problèmes communs** :
- ❌ **Authentification** non sécurisée (mots de passe en clair)
- ❌ **Intégrité** non garantie (modification possible)
- ❌ **Confidentialité** absente (écoute réseau = lecture des données)

**Solution** : **SSH** (Secure Shell)

---

### Définition de SSH

> [!quote] Secure Shell (SSH)
> Protocole **client-serveur** de communication sécurisé initié par **Tatu Ylönen** (RFC 4251, 4252, 4253 et 4254).

**Caractéristiques** :
- ✅ Établissement d'une **session de communication sécurisée** entre 2 machines/terminaux
- ✅ Assure **confidentialité, authentification bidirectionnelle et intégrité**
- ✅ Basé sur le modèle **client / serveur**
- ✅ Encapsulé dans **TCP port 22** (par défaut)
- ✅ Utilisé sur **Linux, Unix, Windows**

---

### Le protocole SSH

> [!info] Détails techniques

**Couche OSI** :
- Protocole de **couche 7** (Application)

**Versions** :

| Version | Statut | Description |
|---------|--------|-------------|
| **SSH-1** | ❌ Obsolète | Vulnérabilités connues, ne plus utiliser |
| **SSH-2** | ✅ **Recommandé** | Version actuelle et sécurisée |
| **SSH3** (2023) | 🆕 Alternative | Basé sur HTTP/3 et QUIC |

> [!warning] Toujours utiliser SSH-2
> SSH-1 contient des failles de sécurité. SSH-2 est le standard actuel.

---

### Utilisations de SSH

> [!success] À quoi sert SSH ?

**1. Terminal et commandes à distance** :
- Ouverture de **terminal distant**
- Lancement de **commandes à distance**
- Remplacement de **Telnet** et **RSH**

**2. Transfert de fichiers** :
- **SFTP** : SSH File Transfer Protocol (transfert sécurisé de fichiers)
- **SCP** : Secure Copy (copie de fichiers)
- **SSHFS** : SSH File System (montage de dossier distant)

**3. Tunnels et redirections** :
- **Transfert de port** (port forwarding)
- **Proxy SOCKS**
- **Tunnels VPN-like**

**4. Session graphique** :
- **X11 forwarding** : Ouverture de session graphique à distance

---

### OpenSSH - L'implémentation

> [!important] OpenSSH
> **Principale implémentation** du protocole SSH.

**Composants** :

| Composant | Paquet | Description |
|-----------|--------|-------------|
| **Client** | `openssh-client` | Programme `ssh` |
| **Serveur** | `openssh-server` | Daemon `sshd` |

**Informations** :
- **Version actuelle** : 9.9p2 (juillet 2024)
- **Mainteneur** : Équipe **OpenBSD**
- **Licence** : Open Source (BSD)

**Installation** :
```bash
# Installer le client SSH (souvent pré-installé)
sudo apt install openssh-client

# Installer le serveur SSH
sudo apt install openssh-server

# Vérifier la version
ssh -V
```

---

### Connexion SSH - Fonctionnement

> [!info] Comment fonctionne une connexion SSH ?

#### Phase 1 : Négociation

**Début de connexion** :
```
1. Le client se connecte au serveur (port 22)
2. Le client indique sa version SSH et implémentation
3. Le serveur indique sa version SSH et implémentation
4. Négociation des algorithmes cryptographiques utilisés
```

**Algorithmes négociés** :
- Chiffrement symétrique
- Échange de clés
- Fonction de hachage (MAC)
- Compression

---

#### Phase 2 : Échange de clés et sécurisation

> [!important] Sécurisation de la connexion

**Mécanismes cryptographiques** :

| Mécanisme | Usage | Garantie |
|-----------|-------|----------|
| **Chiffrement symétrique + HMAC** | Communication chiffrée | Confidentialité |
| **Chiffrement asymétrique** | Échange de clés, authentification | Sécurité de l'échange |
| **Hachage (MAC)** | Vérification des messages | Intégrité |

**Échange de clés** :
- **Diffie-Hellman** (en pratique **ECDH** - Elliptic Curve Diffie-Hellman)
- Permet la **Confidentialité Persistante** (PFS - Perfect Forward Secrecy)
- → Compromission d'une clé n'affecte pas les sessions passées

> [!success] Après l'échange de clés
> À ce stade, la connexion devient **confidentielle et intègre**.

---

#### Phase 3 : Authentification

**Authentification bidirectionnelle** :
1. **Client authentifie le serveur** (vérifier que c'est le bon serveur)
2. **Serveur authentifie le client** (vérifier l'identité de l'utilisateur)

---

### Cryptosystèmes SSH

> [!info] Algorithmes supportés par SSH

**SSH distingue plusieurs catégories** :

| Catégorie | Description | Exemples |
|-----------|-------------|----------|
| **cipher** | Chiffrements symétriques | AES-128, AES-256 |
| **cipher-auth** | Chiffrements symétriques authentifiés | AES-GCM, ChaCha20-Poly1305 |
| **kex** | Échange de clés | ECDH, DH |
| **key** | Types de clés | Ed25519, RSA, ECDSA |
| **mac** | Contrôles d'intégrité | HMAC-SHA2 |

**Lister les algorithmes supportés** :
```bash
# Afficher la version
ssh -V

# Lister les chiffrements authentifiés
ssh -Q cipher-auth

# Exemples de sortie :
# aes128-gcm@openssh.com
# aes256-gcm@openssh.com
# chacha20-poly1305@openssh.com

# Lister les types de clés
ssh -Q key-plain

# Exemples de sortie :
# ssh-ed25519
# ssh-rsa
# ecdsa-sha2-nistp256
# ecdsa-sha2-nistp384
# ecdsa-sha2-nistp521
```

> [!note] ECDSA
> **ECDSA** : Elliptic Curve Digital Signature Algorithm (algorithme de signature basé sur les courbes elliptiques).

---

## Authentification SSH

> [!abstract] T'es qui toi ?
> SSH propose une **authentification bidirectionnelle** : le client authentifie le serveur ET le serveur authentifie le client.

### Principe de l'authentification

**Ordre d'authentification** :

```
1. Authentification du SERVEUR par le client
   ↓ (lors de l'initialisation de la connexion)
   
2. Authentification du CLIENT par le serveur
   ↓ (sur le canal établi avec le bon serveur)
   
3. Communication sécurisée
   → Confidentialité et intégrité des échanges
```

> [!important] Pourquoi cet ordre ?
> Authentifier d'abord le serveur garantit que l'utilisateur ne va pas **envoyer son mot de passe à un serveur malveillant** (attaque MITM - Man In The Middle).

---

### Authentification du serveur

> [!important] Où suis-je connecté ?

#### Approche classique : TOFU (Trust On First Use)

**Modèle TOFU** : "Faire confiance à la première utilisation"

**Fonctionnement** :
```
1. Le serveur dispose d'une paire de clés asymétriques
2. À la première connexion :
   - Le serveur envoie sa clé publique
   - Le client affiche l'empreinte (fingerprint) de la clé
   - Le client demande validation manuelle
3. Après validation :
   - Clé stockée dans ~/.ssh/known_hosts
   - Connexions futures vérifiées automatiquement
```

> [!note] Approche alternative : Certificats
> **Authentification par certificats** (moins courant) est aussi possible. SSH permet une gestion de certificats avec une autorité interne et des certificats auto-signés.

---

### Clés du serveur

> [!info] L'identité d'un serveur

**Emplacement** : `/etc/ssh/` (par défaut)

**Format des fichiers** :
- `ssh_host_<algo>_key` : Clé privée
- `ssh_host_<algo>_key.pub` : Clé publique

**Lister les clés serveur** :
```bash
ls /etc/ssh/ssh_host_*

# Sortie typique :
# /etc/ssh/ssh_host_ecdsa_key
# /etc/ssh/ssh_host_ecdsa_key.pub
# /etc/ssh/ssh_host_ed25519_key
# /etc/ssh/ssh_host_ed25519_key.pub
# /etc/ssh/ssh_host_rsa_key
# /etc/ssh/ssh_host_rsa_key.pub
```

**Afficher l'empreinte d'une clé** :
```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub

# Sortie :
# 256 SHA256:kPFlNQn+PHJ+PMOcHe490TpKDjAIv7qmLM9XiZn3ahs root@server (ED25519)
```

> [!note] À l'installation
> Lors de l'installation du serveur SSH :
> - Génération automatique de clés pour les algorithmes recommandés
> - Clés créées pour Ed25519, ECDSA, RSA

---

### Première connexion au serveur

> [!important] La première connexion

**Processus** :

```bash
ssh server
```

**Affichage** :
```
The authenticity of host 'server (fd26:ba41:c8d6:0:a00:27ff:fea8:3fdf)' can't be established.
ED25519 key fingerprint is SHA256:kPFlNQn+PHJ+PMOcHe490TpKDjAIv7qmLM9XiZn3ahs.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'server' (ED25519) to the list of known hosts.
wilder@server's password:
```

**Étapes** :
1. **Affichage de l'empreinte** (fingerprint)
2. **Vérification manuelle** (en théorie, comparer avec l'empreinte du serveur)
3. **Acceptation** : Taper `yes`
4. **Stockage** : Clé copiée dans `~/.ssh/known_hosts`
5. **Challenge** : Client envoie un challenge au serveur
   - Le serveur prouve qu'il possède la clé privée associée
   - Échec au challenge = fin de connexion

> [!tip] Bonne pratique
> **Vérifier l'empreinte** avant d'accepter :
> - Sur le serveur : `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`
> - Comparer avec l'empreinte affichée lors de la connexion

---

### Connexions suivantes

> [!success] Les connexions suivantes

**Processus** :
```
1. Vérification : Correspondance entre clé reçue et clé connue
2. Challenge serveur
3. Résultat :
   ✅ Correspondance → Suite de la connexion
   ❌ Différence → ALERTE ! Possible usurpation
```

**En cas de correspondance** : Connexion établie sans avertissement.

---

### Alerte d'usurpation

> [!warning] Attention : Clé serveur modifiée !

**Situation** : La clé du serveur a changé depuis la dernière connexion.

**Affichage** :
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@  WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!  @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:wAlhyiiioksvb9WnJWon2sT7yCvL88llyest1wyWYz8.
Please contact your system administrator.
Add correct host key in /home/wilder/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/wilder/.ssh/known_hosts:13
  remove with:
  ssh-keygen -f "/home/wilder/.ssh/known_hosts" -R "server"
Host key for server has changed and you have requested strict checking.
Host key verification failed.
```

**Causes possibles** :
- ✅ **Légitime** : Réinstallation du serveur, nouvelles clés générées
- ⚠️ **Attaque MITM** : Un attaquant intercepte la connexion

**Action** :
```bash
# Si changement légitime, supprimer l'ancienne clé
ssh-keygen -f "/home/wilder/.ssh/known_hosts" -R "server"

# Puis se reconnecter
ssh server
```

---

### Authentification du client

> [!important] Qui veut se connecter ?

**À la suite de la connexion** : Authentification du **client** par le **serveur**.

#### Méthodes d'authentification du client

| Méthode | Description | Usage |
|---------|-------------|-------|
| **password** | Par mot de passe | Basée sur les comptes système |
| **publickey** | Par clé asymétrique | Paire de clés client + clé publique sur serveur |
| **hostbased** | Basée sur l'hôte | Par clés, pour tous les utilisateurs d'un hôte |
| **GSSAPI** | Outils tiers | Kerberos, PAM |

> [!note] Méthodes multiples
> Ces différentes méthodes peuvent **s'additionner** (ex: clé + mot de passe).

---

### Génération de clés client

> [!important] Générer ses clés

**Emplacement** : `~/.ssh/`

**Format des fichiers** :
- `id_<algo>` : Clé privée
- `id_<algo>.pub` : Clé publique

> [!warning] Pas de clés par défaut
> Contrairement au serveur, le client n'a **pas de clés par défaut**. Il faut les générer.

**Génération avec ssh-keygen** :
```bash
# Générer une paire de clés ECDSA
ssh-keygen -t ecdsa -b 256

# Processus interactif :
Generating public/private ecdsa key pair.
Enter file in which to save the key (/home/wilder/.ssh/id_ecdsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/wilder/.ssh/id_ecdsa
Your public key has been saved in /home/wilder/.ssh/id_ecdsa.pub
The key fingerprint is:
SHA256:fi1V+zFtqOar2bMIiFG8KogqycMEBpzvV4N+kBT7z0E wilder@host
```

**Types de clés recommandés** :

| Type | Commande | Longueur | Recommandation |
|------|----------|----------|----------------|
| **Ed25519** | `ssh-keygen -t ed25519` | 256 bits | ✅ **Très recommandé** (rapide, sûr) |
| **ECDSA** | `ssh-keygen -t ecdsa -b 521` | 256-521 bits | ✅ Recommandé |
| **RSA** | `ssh-keygen -t rsa -b 4096` | 2048-4096 bits | ✅ Acceptable (préférer ≥ 3072) |
| **DSA** | - | - | ❌ Obsolète, ne pas utiliser |

---

#### Passphrase

> [!important] Passphrase : Chiffrement de la clé privée

**Passphrase** : Mot de passe pour chiffrer la clé privée.

**Avantages** :
- ✅ Protège la clé privée si le fichier est volé
- ✅ Sécurité renforcée

**Inconvénient** :
- ⚠️ Demandée à chaque utilisation de la clé (atténué avec `ssh-agent`)

**Bonnes pratiques** :
- ✅ Toujours utiliser une passphrase
- ✅ Passphrase longue et robuste
- ✅ Utiliser `ssh-agent` pour éviter de la retaper constamment

---

### Copier sa clé publique sur le serveur

> [!success] Faire connaissance

**Configuration par défaut de sshd** : `publickey` OU `password`

**Objectif** : Copier la clé publique sur le serveur pour pouvoir s'authentifier par clé.

#### Méthode 1 : ssh-copy-id (Recommandé)

```bash
ssh-copy-id server

# Processus :
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s)...
wilder@server's password:

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'server'"
and check to make sure that only the key(s) you wanted were added.

# Test de connexion
ssh server
# ✅ Plus besoin de mot de passe !
```

**Fonctionnement** :
- Copie la clé publique via SSH
- Utilise une autre méthode d'authentification (password)
- Ajoute la clé dans `~/.ssh/authorized_keys` sur le serveur

---

#### Méthode 2 : Copie manuelle

```bash
# Sur le client : afficher la clé publique
cat ~/.ssh/id_ed25519.pub

# Sur le serveur : ajouter dans authorized_keys
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-ed25519 AAAA..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

---

#### Désactiver l'authentification par mot de passe

> [!tip] Sécurité renforcée

**Après avoir configuré l'authentification par clé**, on peut désactiver le mot de passe :

```bash
# Sur le serveur : éditer la config SSH
sudo nano /etc/ssh/sshd_config

# Modifier
PasswordAuthentication no

# Redémarrer le service
sudo systemctl restart sshd
```

> [!warning] Attention
> S'assurer d'avoir une clé fonctionnelle AVANT de désactiver le mot de passe, sinon impossible de se connecter !

---

## Usages et configuration

### Configuration du serveur SSH

> [!info] Un peu de conf

**Daemon** : `sshd` (serveur SSH)

**Gestion** : Daemon classique (systemd ou init)
```bash
# Démarrer
sudo systemctl start ssh

# Arrêter
sudo systemctl stop ssh

# Redémarrer
sudo systemctl restart ssh

# Statut
sudo systemctl status ssh

# Activer au démarrage
sudo systemctl enable ssh
```

**Fichier de configuration** : `/etc/ssh/sshd_config`

**Documentation** : `man sshd_config`

---

#### Quelques idées de configuration

**1. Changer le port SSH** :

> [!tip] Ne pas ouvrir le port 22 sur Internet
> Le port 22 est **très scanné** par les bots. Changer le port réduit les tentatives automatiques.

```bash
# /etc/ssh/sshd_config
Port 2222
```

---

**2. Authentification par clé obligatoire** :

```bash
# Seulement clé publique
AuthenticationMethods publickey

# Clé + mot de passe (multi-facteur)
AuthenticationMethods publickey,password
```

---

**3. Désactiver root SSH** :

```bash
PermitRootLogin no
```

---

**4. Limiter les utilisateurs autorisés** :

```bash
# Autoriser uniquement certains utilisateurs
AllowUsers wilder admin

# Ou autoriser un groupe
AllowGroups ssh-users
```

---

**5. Tester la configuration** :

```bash
# Vérifier la syntaxe (ne démarre pas le serveur)
sudo sshd -t

# Tester et afficher la configuration
sudo sshd -T
```

> [!warning] Toujours tester avant de redémarrer
> Une erreur de configuration peut **bloquer l'accès SSH** !

---

### Configuration du client SSH

> [!info] Un peu de conf 2

**Programme** : `ssh` (client SSH)

**Fichiers de configuration** :

| Fichier | Portée | Priorité |
|---------|--------|----------|
| `/etc/ssh/ssh_config` | Système (global) | Basse |
| `~/.ssh/config` | Utilisateur | **Haute** (prioritaire) |

**Documentation** : `man ssh_config`

---

#### Quelques idées de configuration

**1. Vérifier IP et nom de domaine** :

```bash
# ~/.ssh/config
CheckHostIP yes
```

> [!tip] Éviter le spoofing DNS
> Vérifie que le nom de domaine et l'IP correspondent.

---

**2. Préciser le port par défaut** :

```bash
Port 2222
```

---

**3. Configuration spécifique à un hôte** :

```bash
# ~/.ssh/config

# Configuration pour un serveur spécifique
Host monserveur
    HostName 192.168.1.100
    User wilder
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_special

# Configuration pour tous les serveurs d'un domaine
Host *.exemple.com
    User admin
    Port 22
```

**Utilisation** :
```bash
# Au lieu de :
ssh -p 2222 wilder@192.168.1.100

# On peut simplement taper :
ssh monserveur
```

---

**4. Afficher la configuration** :

```bash
# Configuration pour un hôte spécifique
ssh -G host
```

---

### Shell distant

> [!success] Remplacer Telnet et RSH

**Programme** : `ssh` (client)

**Syntaxe** :
```bash
ssh [options] [user@]host
# Ou format URI :
ssh://[user@]host[:port]
```

**Paramètres** :
- **host** : Nom d'hôte ou adresse IP (obligatoire)
- **user** : Nom d'utilisateur (défaut : login local)
- **port** : Port SSH (défaut : 22)

---

#### Options courantes

| Option | Description |
|--------|-------------|
| `-p <port>` | Spécifier le port |
| `-l <login>` | Spécifier l'utilisateur |
| `-v`, `-vv`, `-vvv` | Mode verbeux (debug) |
| `-i <fichier>` | Spécifier la clé privée |
| `-X` | Activer X11 forwarding |
| `-L` | Port forwarding local |
| `-R` | Port forwarding distant |
| `-D` | Proxy SOCKS |

---

#### Exemples de connexion

```bash
# Connexion simple
ssh server

# Spécifier l'utilisateur
ssh wilder@server

# Format URI
ssh ssh://wilder@server:22

# Avec options
ssh -l wilder -p 22 server

# Connexion verbeux (debug)
ssh -v server

# Avec clé spécifique
ssh -i ~/.ssh/id_rsa_special server
```

---

### Copie de fichiers

> [!success] Remplacer RCP

**Programme** : `scp` (Secure Copy)

**Syntaxe** :
```bash
scp [options] source destination
```

**Format des chemins** :
- **Local** : `/chemin/vers/fichier`
- **Distant** : `[user@]host:/chemin/vers/fichier`

---

#### Exemples

**Copier vers le serveur** :
```bash
# Fichier local → serveur distant
scp local_file server:distant_name
scp fichier.txt wilder@server:/home/wilder/

# Dossier entier (récursif)
scp -r /local/dossier server:/remote/path/
```

**Copier depuis le serveur** :
```bash
# Serveur distant → local
scp server:/dir/distant_name ~/Downloads/
scp wildo@server:/var/log/syslog /tmp/
```

**Copier entre deux serveurs** :
```bash
# Serveur1 → Serveur2 (via le client)
scp server1:/fichier server2:/destination/
```

---

#### Options SCP

| Option | Description |
|--------|-------------|
| `-r` | Copie récursive (dossiers) |
| `-P <port>` | Spécifier le port (attention : **majuscule**) |
| `-p` | Préserver permissions et dates |
| `-C` | Activer la compression |
| `-v` | Mode verbeux |

**Exemples** :
```bash
# Copier un dossier avec compression
scp -r -C /local/dossier server:/remote/

# Avec port spécifique
scp -P 2222 fichier.txt server:/destination/
```

> [!note] Destination doit exister
> Le répertoire de destination doit déjà exister sur le serveur.

---

### Cas d'usage des clés SSH

> [!info] Pour quels usages ?

#### Clé privée chez moi (client)

**Génération sur l'hôte local** → Clé privée reste sur le client

**Usages** :

1. **Accès à un serveur distant** :
   ```
   Client (clé privée) → Serveur (clé publique dans authorized_keys)
   ```

2. **Gestion d'un dépôt Git distant (GitHub, GitLab...)** :
   ```
   Client (clé privée) → GitHub (clé publique ajoutée au compte)
   ```
   - `git push` et `git pull` utilisent la clé privée
   - Optionnel : Créer `~/.ssh/config` pour spécifier la clé

**Exemple de config GitHub** :
```bash
# ~/.ssh/config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
```

---

#### Clé privée sur un serveur distant (bastion)

**Accès au parc d'un client via serveur bastion** :

```
Poste local → Serveur bastion (clé privée + publique) → Serveurs du parc
```

**Processus** :
1. Connexion au serveur bastion (avec clé du poste local)
2. Depuis le bastion, connexion aux autres serveurs (avec clé du bastion)

**Avantage** : Un seul point d'entrée sécurisé (bastion)

---

### Transfert de fichiers interactif

> [!success] Remplacer FTP

**Programme** : `sftp` (SSH File Transfer Protocol)

**Syntaxe** :
```bash
sftp [user@]host[:chemin]
# Ou format URI :
sftp://[user@]host[:port][/chemin]
```

**Ouverture d'une invite de commande** : Similaire à FTP

---

#### Commandes SFTP

```bash
# Se connecter
sftp server

# Invite de commande :
sftp> help
Available commands:
bye                             Quit sftp
cd path                         Change remote directory
get [-afpR] remote [local]      Download file
lcd path                        Change local directory
ls [-1afhlnrSt] [path]          Display remote directory
put [-afpR] local [remote]      Upload file
mkdir path                      Create remote directory
pwd                             Display remote working directory
rm path                         Delete remote file
rmdir path                      Remove remote directory
```

---

#### Exemples d'utilisation

```bash
# Connexion
sftp server

# Navigation
sftp> pwd                    # Afficher répertoire distant
sftp> lpwd                   # Afficher répertoire local
sftp> ls                     # Lister fichiers distants
sftp> cd /var/www            # Changer répertoire distant
sftp> lcd ~/Downloads        # Changer répertoire local

# Téléchargement (serveur → client)
sftp> get fichier.txt
sftp> get -r dossier/        # Récursif

# Upload (client → serveur)
sftp> put fichier.txt
sftp> put -r dossier/        # Récursif

# Quitter
sftp> bye
```

---

#### Clients graphiques

**Clients SFTP avec interface graphique** :
- **FileZilla** : Multi-plateforme
- **WinSCP** : Windows
- **Cyberduck** : macOS, Windows

---

### Tunnels SSH

> [!important] Encapsuler dans SSH

**SSH sert aussi de tunnel** pour encapsuler du trafic et le transmettre de l'autre côté.

#### 1. Redirection de port local (Local Port Forwarding)

**Syntaxe** : `ssh -L port:host:hostport server`

**Principe** :
```
Client (port local) → Tunnel SSH → Serveur → host:hostport
```

**Exemple** :
```bash
# Accéder à un service MySQL sur le serveur via localhost:3306
ssh -L 3306:localhost:3306 server

# Depuis le client, se connecter à localhost:3306
mysql -h 127.0.0.1 -P 3306 -u user -p
```

**Usage** : Accéder à un service distant comme s'il était local.

---

#### 2. Redirection de port distant (Remote Port Forwarding)

**Syntaxe** : `ssh -R port:host:hostport server`

**Principe** :
```
Serveur (port distant) → Tunnel SSH → Client → host:hostport
```

**Exemple** :
```bash
# Exposer un service local (port 8080) sur le serveur (port 9090)
ssh -R 9090:localhost:8080 server

# Depuis le serveur, accéder à localhost:9090 pour atteindre le client:8080
```

**Usage** : Exposer un service local à travers le serveur.

---

#### 3. Proxy SOCKS (Dynamic Port Forwarding)

**Syntaxe** : `ssh -D port server`

**Principe** :
```
Applications → Proxy SOCKS (localhost:port) → Tunnel SSH → Serveur → Internet
```

**Exemple** :
```bash
# Créer un proxy SOCKS sur le port 1080
ssh -D 1080 server

# Configurer le navigateur pour utiliser SOCKS5 : localhost:1080
```

**Usage** :
- Anonymiser la navigation (IP du serveur)
- Contourner des restrictions réseau

---

#### 4. Tunnels VPN-like (TUN/TAP)

**Syntaxe** : `ssh -w local_tun_num:remote_tun_num server`

**Principe** : Création d'interfaces réseau virtuelles (tun) de chaque côté.

**Usage** : VPN DIY (niveau IP, pas juste port).

---

#### 5. X11 Forwarding

**Syntaxe** : `ssh -X server`

**Principe** : Permet le lancement d'applications graphiques à distance.

**Exemple** :
```bash
# Se connecter avec X11 forwarding
ssh -X server

# Lancer une application graphique
firefox &
gedit &
```

**Affichage** : L'application s'affiche sur le client, mais s'exécute sur le serveur.

**Usage** : Administration graphique à distance (sans VNC/RDP).

---

### Outils additionnels

> [!info] Et le reste

**L'installation de SSH apporte aussi des outils additionnels** :

| Outil | Usage |
|-------|-------|
| **ssh-keygen** | Gestion et génération de clés |
| **ssh-agent** | Stockage de clés privées en mémoire (évite de retaper la passphrase) |
| **ssh-add** | Ajout de clés à ssh-agent |
| **ssh-keyscan** | Récupération de clés publiques sur des serveurs |
| **sftp-server** | Sous-système de gestion des connexions SFTP (utilisé par sshd) |
| **ssh-keysign** | Gestionnaire pour l'authentification host-based |

> [!note] sftp-server et ssh-keysign
> Ces outils ne sont **pas utilisés directement** par l'utilisateur.

---

#### ssh-agent et ssh-add

**Problème** : Clé privée avec passphrase → Demandée à chaque connexion

**Solution** : `ssh-agent`

**Fonctionnement** :
1. Démarrer `ssh-agent`
2. Ajouter la clé avec `ssh-add`
3. La passphrase est demandée une seule fois
4. Les connexions suivantes utilisent la clé en mémoire

**Exemple** :
```bash
# Démarrer ssh-agent (souvent automatique sur les systèmes modernes)
eval $(ssh-agent)

# Ajouter une clé
ssh-add ~/.ssh/id_ed25519

# Lister les clés chargées
ssh-add -l

# Supprimer toutes les clés
ssh-add -D
```

---

## Fail2Ban

> [!abstract] Protection anti-bruteforce

### Le problème

**Attaques par force brute** sur SSH :
- ❌ Très courantes sur les serveurs exposés sur Internet
- ❌ Tentatives automatiques de connexion avec des mots de passe courants
- ❌ Des milliers de tentatives par jour sur le port 22

---

### La solution : Fail2Ban

> [!success] Fail2Ban
> Outil qui **surveille les journaux système** et **bloque les adresses IP** qui ont trop d'échecs de connexion.

**Fonctionnement** :
```
1. Fail2Ban surveille les logs (/var/log/auth.log)
2. Détecte les tentatives échouées (mot de passe incorrect)
3. Après N échecs, bannit l'IP (avec iptables/nftables)
4. Débannit automatiquement après un délai
```

---

### Installation et configuration

**Installation** :
```bash
# Debian/Ubuntu
sudo apt install fail2ban

# Démarrer et activer
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

**Configuration** :
```bash
# Fichier principal (ne pas modifier directement)
/etc/fail2ban/jail.conf

# Créer une config locale (prioritaire)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Éditer
sudo nano /etc/fail2ban/jail.local
```

---

### Configuration SSH

**Exemple de configuration** :
```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

**Paramètres** :

| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| `enabled` | Activer la jail | `true` |
| `port` | Port à surveiller | `22` |
| `maxretry` | Nombre d'échecs avant ban | `3` |
| `bantime` | Durée du ban (secondes) | `3600` (1h) |
| `findtime` | Fenêtre de temps pour compter les échecs | `600` (10 min) |

**Redémarrer après modification** :
```bash
sudo systemctl restart fail2ban
```

---

### Vérification

**Statut de Fail2Ban** :
```bash
# Statut général
sudo systemctl status fail2ban

# Statut d'une jail spécifique
sudo fail2ban-client status sshd

# Sortie :
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     127
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 3
   |- Total banned:     42
   `- Banned IP list:   192.168.1.50 203.0.113.10 198.51.100.5
```

**Débannir une IP manuellement** :
```bash
sudo fail2ban-client set sshd unbanip 192.168.1.50
```

---

### Configuration avancée

**Augmenter la sévérité** :
```ini
[sshd]
enabled = true
maxretry = 2        # Seulement 2 tentatives
bantime = 86400     # Ban 24 heures
findtime = 3600     # Sur 1 heure
```

**Ban permanent après X bans** :
```ini
[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log
bantime = -1        # Ban permanent
findtime = 604800   # 1 semaine
maxretry = 3
```

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Concepts fondamentaux

**SSH (Secure Shell)** :
- Protocole **client-serveur** de communication sécurisée
- Remplace Telnet, RSH, FTP (non sécurisés)
- Port par défaut : **22** (TCP)
- Garantit : **Confidentialité, Authentification bidirectionnelle, Intégrité**

**Versions** :
- SSH-1 : ❌ Obsolète
- SSH-2 : ✅ Standard actuel
- SSH3 : 🆕 Alternative (HTTP/3 + QUIC)

**OpenSSH** :
- Implémentation principale du protocole SSH
- Client : `ssh`, Serveur : `sshd`
- Version actuelle : 9.9p2

---

### Connexion SSH

**Phases de connexion** :
1. **Négociation** : Versions et algorithmes
2. **Échange de clés** : Diffie-Hellman (ECDH), PFS
3. **Authentification** : Serveur puis client

**Cryptographie** :
- **Chiffrement symétrique** : AES-GCM, ChaCha20-Poly1305
- **Chiffrement asymétrique** : Ed25519, ECDSA, RSA
- **MAC** : HMAC-SHA2
- **PFS** (Perfect Forward Secrecy) : Compromission d'une clé n'affecte pas les sessions passées

---

### Authentification

**Authentification bidirectionnelle** :
1. **Client authentifie serveur** (TOFU - Trust On First Use)
2. **Serveur authentifie client** (password ou publickey)

**Serveur** :
- Clés dans `/etc/ssh/ssh_host_*`
- Première connexion : Validation manuelle de l'empreinte
- Connexions suivantes : Vérification automatique (`~/.ssh/known_hosts`)

**Client** :
- Clés dans `~/.ssh/id_*`
- Génération : `ssh-keygen -t ed25519`
- Copie sur serveur : `ssh-copy-id server`
- Authentification : password OU publickey (ou les deux)

**Recommandations** :
- ✅ Toujours utiliser une **passphrase** pour les clés privées
- ✅ Préférer **Ed25519** (rapide, sûr)
- ✅ Désactiver **PasswordAuthentication** après config par clé
- ✅ Utiliser **ssh-agent** pour éviter de retaper la passphrase

---

### Usages SSH

**Terminal distant** :
- `ssh [user@]host` : Connexion shell
- Options : `-p` (port), `-l` (login), `-v` (verbeux)

**Transfert de fichiers** :
- **SCP** : `scp source destination` (copie simple)
- **SFTP** : `sftp server` (transfert interactif)
- **SSHFS** : Montage de dossier distant

**Tunnels** :
- **Port forwarding local** : `ssh -L port:host:hostport`
- **Port forwarding distant** : `ssh -R port:host:hostport`
- **Proxy SOCKS** : `ssh -D port`
- **X11 forwarding** : `ssh -X` (applications graphiques)

---

### Configuration

**Serveur (`/etc/ssh/sshd_config`)** :
- Changer le port : `Port 2222`
- Désactiver root : `PermitRootLogin no`
- Authentification : `AuthenticationMethods publickey,password`
- Tester : `sudo sshd -t`

**Client (`~/.ssh/config`)** :
- Config par hôte : `Host monserveur`
- Port, utilisateur, clé spécifique
- Afficher config : `ssh -G host`

---

### Fail2Ban

**Protection anti-bruteforce** :
- Surveille les logs d'authentification
- Bannit les IP après N échecs
- Configuration : `/etc/fail2ban/jail.local`
- Vérifier : `sudo fail2ban-client status sshd`

**Paramètres clés** :
- `maxretry` : Nombre d'échecs avant ban (3)
- `bantime` : Durée du ban en secondes (3600)
- `findtime` : Fenêtre de temps (600)

---

### Bonnes pratiques TSSR

> [!tip] Recommandations professionnelles

**Sécurité** :
1. ✅ Utiliser **SSH-2** uniquement
2. ✅ Authentification par **clé publique** (+ désactiver password)
3. ✅ Clés **Ed25519** ou ECDSA-521
4. ✅ **Passphrase** sur toutes les clés privées
5. ✅ **Changer le port** (≠ 22) si exposition Internet
6. ✅ Désactiver **PermitRootLogin**
7. ✅ Installer **Fail2Ban**
8. ✅ Limiter les utilisateurs autorisés (`AllowUsers`)
9. ✅ Vérifier empreintes serveur à la première connexion
10. ✅ Utiliser **ssh-agent** pour la gestion des clés

**Configuration** :
1. ✅ Toujours tester avec `sshd -t` avant redémarrage
2. ✅ Créer `~/.ssh/config` pour simplifier les connexions
3. ✅ Documenter les configurations spécifiques
4. ✅ Sauvegarder les clés privées (dans un endroit sécurisé)

**Monitoring** :
1. ✅ Surveiller `/var/log/auth.log` régulièrement
2. ✅ Vérifier les connexions actives : `who`, `w`
3. ✅ Auditer les tentatives échouées
4. ✅ Vérifier le statut Fail2Ban régulièrement

---

### Pièges à éviter

> [!warning] Erreurs courantes

1. ❌ Utiliser SSH-1 (obsolète et vulnérable)
2. ❌ Laisser le port 22 ouvert sur Internet sans Fail2Ban
3. ❌ Autoriser `PermitRootLogin yes`
4. ❌ Clés sans passphrase
5. ❌ Accepter l'empreinte serveur sans vérification
6. ❌ Partager sa clé privée
7. ❌ Stocker la clé privée sur un serveur distant (sauf bastion)
8. ❌ Désactiver PasswordAuthentication sans avoir testé les clés
9. ❌ Ne pas sauvegarder les clés privées
10. ❌ Utiliser DSA (obsolète)

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **SSH** | Secure Shell, protocole de communication sécurisé |
| **OpenSSH** | Implémentation open source du protocole SSH |
| **sshd** | SSH Daemon, serveur SSH |
| **ssh** | Client SSH |
| **Port 22** | Port TCP par défaut de SSH |
| **SSH-2** | Version 2 du protocole SSH (recommandée) |
| **Telnet** | Ancien protocole de terminal distant (non sécurisé) |
| **RSH** | Remote Shell (non sécurisé) |
| **PFS** | Perfect Forward Secrecy, confidentialité persistante |
| **TOFU** | Trust On First Use, modèle d'authentification serveur |
| **Fingerprint** | Empreinte d'une clé publique (hash) |
| **known_hosts** | Fichier contenant les clés publiques des serveurs connus |
| **authorized_keys** | Fichier contenant les clés publiques autorisées sur le serveur |
| **Passphrase** | Mot de passe pour chiffrer une clé privée |
| **ssh-keygen** | Outil de génération de clés SSH |
| **ssh-copy-id** | Outil pour copier une clé publique sur un serveur |
| **ssh-agent** | Agent de gestion des clés en mémoire |
| **ssh-add** | Ajouter une clé à ssh-agent |
| **SCP** | Secure Copy, copie de fichiers sécurisée |
| **SFTP** | SSH File Transfer Protocol |
| **SSHFS** | SSH File System, montage distant |
| **Port forwarding** | Redirection de port via tunnel SSH |
| **Local forwarding** | Tunnel SSH pour accéder à un service distant |
| **Remote forwarding** | Tunnel SSH pour exposer un service local |
| **SOCKS** | Proxy générique niveau application |
| **X11 forwarding** | Affichage d'applications graphiques distantes |
| **Tunnel SSH** | Encapsulation de trafic dans SSH |
| **MITM** | Man In The Middle, attaque d'interception |
| **Fail2Ban** | Outil de protection anti-bruteforce |
| **Bruteforce** | Attaque par essais successifs de mots de passe |
| **Jail** | Prison Fail2Ban surveillant un service |
| **Ed25519** | Algorithme de signature moderne (recommandé) |
| **ECDSA** | Elliptic Curve Digital Signature Algorithm |
| **RSA** | Algorithme asymétrique classique |
| **AES-GCM** | Chiffrement symétrique authentifié |
| **ChaCha20-Poly1305** | Chiffrement de flux moderne |
| **ECDH** | Elliptic Curve Diffie-Hellman |
| **HMAC** | Hash-based Message Authentication Code |
| **Bastion** | Serveur intermédiaire sécurisé (point d'entrée unique) |

---

## 📖 Références externes

> [!note] Liens mentionnés dans le cours
> Ces ressources ont été référencées dans le PowerPoint original pour approfondir certains sujets.

| Section du cours | Ressource | Description | Lien |
|------------------|-----------|-------------|------|
| **Introduction SSH** | Wikipédia - Secure Shell | Article complet sur SSH, historique et concepts | [Wikipédia - Secure Shell](https://fr.wikipedia.org/wiki/Secure_Shell) |
| **Spécifications SSH** | RFC 4251 | Architecture du protocole SSH | [RFC 4251 - SSH Protocol Architecture](https://tools.ietf.org/html/rfc4251) |
| **Spécifications SSH** | RFC 4252 | Protocole d'authentification SSH | [RFC 4252 - SSH Authentication Protocol](https://tools.ietf.org/html/rfc4252) |
| **Spécifications SSH** | RFC 4253 | Protocole de transport SSH | [RFC 4253 - SSH Transport Layer Protocol](https://tools.ietf.org/html/rfc4253) |
| **Spécifications SSH** | RFC 4254 | Protocole de connexion SSH | [RFC 4254 - SSH Connection Protocol](https://tools.ietf.org/html/rfc4254) |
| **OpenSSH officiel** | Site OpenSSH | Site officiel du projet OpenSSH | [openssh.com](https://www.openssh.com) |
| **Formation SSH** | SSH Academy | Tutoriels et formations sur SSH | [ssh.com/academy](https://www.ssh.com/academy/ssh) |
| **Sécurisation OpenSSH** | ANSSI | Recommandations de sécurité pour OpenSSH | [ANSSI - Recommandations OpenSSH](https://www.ssi.gouv.fr/guide/recommandations-pour-un-usage-securise-dopenssh/) |
| **Fail2Ban** | Ubuntu-FR | Documentation Fail2Ban en français | [doc.ubuntu-fr.org/fail2ban](https://doc.ubuntu-fr.org/fail2ban) |

> [!tip] Comment utiliser ces ressources
> Ces liens te permettront de :
> - **Approfondir** les concepts SSH (Wikipédia, SSH Academy)
> - **Consulter les standards officiels** (RFC 4251-4254)
> - **Sécuriser** tes configurations SSH (ANSSI)
> - **Installer** et configurer OpenSSH (site officiel)
> - **Protéger** contre les attaques bruteforce (Fail2Ban)
> - Comprendre les **détails techniques** du protocole

---

### Ressources complémentaires recommandées

> [!info] Pour aller plus loin

| Thème | Ressource | Description | Lien |
|-------|-----------|-------------|------|
| **Documentation OpenSSH** | Manuel OpenSSH | Pages man en ligne | [man.openbsd.org/ssh](https://man.openbsd.org/ssh) |
| **Configuration sshd** | man sshd_config | Documentation configuration serveur | [man.openbsd.org/sshd_config](https://man.openbsd.org/sshd_config) |
| **Configuration ssh** | man ssh_config | Documentation configuration client | [man.openbsd.org/ssh_config](https://man.openbsd.org/ssh_config) |
| **Génération de clés** | man ssh-keygen | Documentation ssh-keygen | [man.openbsd.org/ssh-keygen](https://man.openbsd.org/ssh-keygen) |
| **Fail2Ban officiel** | Site Fail2Ban | Documentation officielle Fail2Ban | [fail2ban.org](https://www.fail2ban.org) |
| **Sécurité SSH** | Mozilla SSH Guidelines | Guide de configuration sécurisée | [Mozilla Infosec - OpenSSH](https://infosec.mozilla.org/guidelines/openssh) |
| **Test configuration SSH** | SSH Audit | Outil d'audit de configuration SSH | [github.com/jtesta/ssh-audit](https://github.com/jtesta/ssh-audit) |

---

> [!success] Document de révision complet
> Ce document couvre l'intégralité du cours sur **SSH (Secure Shell)**. Tu as maintenant tous les éléments pour :
> - Comprendre le **protocole SSH** et son fonctionnement
> - Configurer un **serveur SSH** sécurisé
> - Utiliser le **client SSH** pour les connexions distantes
> - Gérer l'**authentification** (clés publiques/privées)
> - Transférer des fichiers avec **SCP** et **SFTP**
> - Créer des **tunnels SSH** (port forwarding, SOCKS)
> - Protéger avec **Fail2Ban** contre les attaques bruteforce
> - Appliquer les **bonnes pratiques de sécurité**
> 
> **Bon courage pour la préparation de ton titre RNCP TSSR !** 🎓🔐✨

---

**Fin du document de révision**
