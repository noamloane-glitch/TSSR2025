# SSH — Windows (OpenSSH)

> Même protocole SSH que Linux, disponible nativement sur **Windows 10/11 et Server 2019+**.  
> Implémentation : **OpenSSH for Windows** (intégré à Windows).

---

## Protocole & Port

| Port | Transport | Couche OSI | Rôle |
|------|-----------|------------|------|
| `22` | TCP | 7 — Application | Shell distant, SCP, SFTP |

---

## Disponibilité Windows

| Composant | Disponibilité | Remarque |
|-----------|--------------|----------|
| **Client SSH** (`ssh.exe`) | ✅ Intégré Win10 1809+ | Activé par défaut |
| **Serveur SSH** (`sshd`) | ⚠️ Intégré mais à installer | Fonctionnalité optionnelle |
| **ssh-keygen.exe** | ✅ Inclus avec le client | Même syntaxe que Linux |
| **scp.exe / sftp.exe** | ✅ Inclus avec le client | Même syntaxe que Linux |

---

## Procédure : Installation serveur SSH (PowerShell)

### 1. Vérifier la disponibilité

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
# Affiche l'état : Installed / NotPresent
```

### 2. Installer le serveur

```powershell
# Installer le serveur SSH
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# (Client, si absent)
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

### 3. Démarrer et activer le service

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic    # démarrage automatique
Get-Service sshd                                  # vérifier l'état
```

### 4. Autoriser dans le pare-feu (normalement automatique)

```powershell
# Vérifier la règle pare-feu
Get-NetFirewallRule -Name *ssh*

# Si absente, l'ajouter manuellement
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' `
  -Enabled True -Direction Inbound -Protocol TCP `
  -Action Allow -LocalPort 22
```

---

## Procédure : Authentification par clé (PowerShell)

### Étape 1 — Générer une paire de clés

```powershell
ssh-keygen                          # RSA 3072 bits (fonctionnel)
ssh-keygen -t ed25519               # ✅ recommandé
```

> Clés stockées dans `C:\Users\<user>\.ssh\`

### Étape 2 — Copier la clé publique sur le serveur

**Vers un serveur Linux :**
```powershell
# Depuis PowerShell Windows (ssh-copy-id n'existe pas nativement)
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh user@ip "cat >> ~/.ssh/authorized_keys"
```

**Vers un serveur Windows :**
```powershell
# Pour un utilisateur standard
$pubkey = Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
Add-Content "\\serveur\C$\Users\user\.ssh\authorized_keys" $pubkey

# Pour un administrateur → fichier spécial !
Add-Content "C:\ProgramData\ssh\administrators_authorized_keys" $pubkey
```

> ⚠️ Sur Windows, les **admins** utilisent `C:\ProgramData\ssh\administrators_authorized_keys`  
> et non `~\.ssh\authorized_keys` — c'est une différence majeure avec Linux !

---

## Configuration serveur `sshd_config`

```powershell
# Emplacement du fichier de config
C:\ProgramData\ssh\sshd_config

# Éditer
notepad C:\ProgramData\ssh\sshd_config
```

| Paramètre | Valeur recommandée | Effet |
|-----------|-------------------|-------|
| `Port` | `22` | Port d'écoute |
| `PermitRootLogin` | `no` | Pas de root (Administrator) direct |
| `PasswordAuthentication` | `no` | Forcer clés SSH |
| `PubkeyAuthentication` | `yes` | Autoriser clés publiques |

```powershell
# Appliquer après modification
Restart-Service sshd
```

---

## Commandes essentielles (identiques Linux)

| Commande | Rôle |
|----------|------|
| `ssh user@ip` | Connexion shell distant |
| `ssh -p 2222 user@ip` | Port personnalisé |
| `scp fichier.txt user@ip:/dest/` | Copie vers serveur Linux |
| `scp user@ip:C:/chemin/fic .` | Copie depuis serveur Windows |
| `sftp user@ip` | Session transfert interactif |
| `ssh-keygen -t ed25519` | Générer une paire de clés |
| `Get-Service sshd` | Vérifier l'état du service |
| `Restart-Service sshd` | Redémarrer le service |

---

## Clients SSH alternatifs sur Windows

| Outil | Type | Remarque |
|-------|------|----------|
| **OpenSSH natif** (`ssh.exe`) | CLI | ✅ Recommandé, intégré |
| **PuTTY** | GUI | Historique, très répandu |
| **MobaXterm** | GUI | Tout-en-un (SSH + SFTP + X11) |
| **Windows Terminal** | CLI | Meilleur terminal pour ssh.exe |

---

## Fichiers importants

| Fichier | Rôle |
|---------|------|
| `C:\ProgramData\ssh\sshd_config` | Config **serveur** SSH |
| `C:\ProgramData\ssh\administrators_authorized_keys` | Clés autorisées pour **admins** |
| `C:\Users\<user>\.ssh\authorized_keys` | Clés autorisées (utilisateurs standard) |
| `C:\Users\<user>\.ssh\id_ed25519` | Clé **privée** client |
| `C:\Users\<user>\.ssh\id_ed25519.pub` | Clé **publique** client |
| `C:\Users\<user>\.ssh\known_hosts` | Empreintes serveurs connus |

---

## ⚠️ À retenir absolument

- Client SSH intégré depuis **Windows 10 version 1809**
- Serveur SSH = **fonctionnalité optionnelle** à installer via PowerShell
- Admins → `administrators_authorized_keys` ≠ `~\.ssh\authorized_keys`
- `ssh-copy-id` **n'existe pas** nativement sur Windows → utiliser `type | ssh`
- Même port `22/TCP`, même protocole, mêmes commandes que Linux
- Après modif `sshd_config` → toujours `Restart-Service sshd`
- Couche OSI : **7 — Application**
