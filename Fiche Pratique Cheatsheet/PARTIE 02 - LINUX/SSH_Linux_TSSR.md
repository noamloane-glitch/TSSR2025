# SSH — Secure Shell

> Protocole d'administration à distance **chiffré**. Remplace Telnet (non sécurisé).  
> Implémentation : **OpenSSH** — standard Linux/Unix.

---

## Protocole & Port

| Port | Transport | Rôle |
|------|-----------|------|
| `22` | TCP | Shell distant, SCP, SFTP |

---

## Procédure : Installation & Configuration serveur

### 1. Installation

```bash
sudo apt update
sudo apt install openssh-server    # installe sshd
sudo systemctl enable ssh          # démarrage automatique
sudo systemctl start ssh           # démarrage immédiat
sudo systemctl status ssh          # vérifier que le service tourne
```

### 2. Configuration `/etc/ssh/sshd_config`

```bash
sudo nano /etc/ssh/sshd_config
```

| Paramètre | Valeur recommandée | Effet |
|-----------|-------------------|-------|
| `Port` | `22` (ou autre) | Port d'écoute |
| `PermitRootLogin` | `no` | Interdire root en SSH |
| `PasswordAuthentication` | `no` | Forcer clés SSH uniquement |
| `PubkeyAuthentication` | `yes` | Autoriser clés publiques |
| `AllowUsers` | `wilder admin` | Restreindre les utilisateurs |
| `MaxAuthTries` | `3` | Limiter les tentatives |

### 3. Tester & Appliquer

```bash
sudo sshd -t                       # ⚠️ toujours tester avant redémarrage
sudo systemctl restart ssh         # appliquer la config
```

---

## Procédure : Authentification par clé (côté client)

### Étape 1 — Générer une paire de clés

```bash
ssh-keygen                         # RSA 3072 bits par défaut (fonctionnel)
ssh-keygen -t ed25519              # ✅ recommandé (rapide, moderne)
ssh-keygen -t rsa -b 4096          # RSA 4096 (acceptable, plus lent)
```

> ⚠️ `ssh-keygen` seul **fonctionne** mais génère du RSA 3072 bits — pas optimal.  
> Le client **n'a pas de clés par défaut**, il faut les générer manuellement.

**Déroulement interactif :**
```
Enter file: ~/.ssh/id_ed25519      # Entrée = emplacement par défaut
Enter passphrase:                  # ✅ toujours mettre une passphrase
```

### Types de clés

| Type | Commande | Recommandation |
|------|----------|----------------|
| **Ed25519** | `ssh-keygen -t ed25519` | ✅ **Recommandé** — rapide, moderne |
| **ECDSA** | `ssh-keygen -t ecdsa -b 521` | ✅ OK |
| **RSA** | `ssh-keygen -t rsa -b 4096` | ✅ Acceptable |
| **DSA** | — | ❌ Obsolète, ne pas utiliser |

### Étape 2 — Copier la clé publique sur le serveur

```bash
ssh-copy-id user@ip                           # méthode recommandée
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@ip  # clé spécifique
```

> Copie automatiquement dans `~/.ssh/authorized_keys` sur le serveur.

### Étape 3 — Se connecter sans mot de passe

```bash
ssh user@ip                        # ✅ connexion par clé
```

---

## Commandes essentielles

| Commande | Rôle |
|----------|------|
| `ssh user@ip` | Connexion shell distant |
| `ssh -p 2222 user@ip` | Port personnalisé |
| `ssh -i ~/.ssh/cle user@ip` | Clé privée spécifique |
| `scp fichier.txt user@ip:/dest/` | Copie locale → distant |
| `scp user@ip:/src/fichier .` | Copie distant → local |
| `scp -r dossier/ user@ip:/dest/` | Copie récursive (dossier) |
| `scp -P 2222 fichier user@ip:/dest/` | SCP sur port différent (⚠️ majuscule) |
| `sftp user@ip` | Session transfert interactif |

---

## Fichiers importants

| Fichier | Rôle |
|---------|------|
| `/etc/ssh/sshd_config` | Config **serveur** SSH |
| `/etc/ssh/ssh_config` | Config **client** SSH (global) |
| `~/.ssh/config` | Config client utilisateur (prioritaire) |
| `~/.ssh/id_ed25519` | Clé **privée** — ne jamais partager |
| `~/.ssh/id_ed25519.pub` | Clé **publique** |
| `~/.ssh/authorized_keys` | Clés publiques autorisées (côté serveur) |
| `~/.ssh/known_hosts` | Empreintes des serveurs connus |

### Permissions obligatoires

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/authorized_keys
```

---

## ⚠️ À retenir absolument

- `ssh-keygen` seul = RSA 3072 bits — fonctionnel mais **pas optimal**
- Toujours préférer `ssh-keygen -t ed25519`
- `scp -P` (MAJUSCULE) pour le port ≠ `ssh -p` (minuscule)
- Toujours `sudo sshd -t` avant `systemctl restart ssh`
- `known_hosts` = protection contre attaque **MITM**
- Désactiver `PasswordAuthentication` **après** avoir configuré les clés
- Passphrase sur clé privée = protection si vol du fichier
