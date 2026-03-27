## ⚡ L'essentiel en 5 minutes - Services Bureautiques

### 📌 C'est quoi en 2 lignes ?
Ensemble de logiciels et applications fournis par la DSI pour les tâches quotidiennes de bureau : messagerie, stockage/partage de fichiers, suites bureautiques (Word/Excel/PPT) et prise de main à distance. Disponibles en local (on-premises) ou dans le cloud.

---

### 💡 Concepts clés à retenir :

* **Service bureautique** : Logiciel permettant tâches courantes de bureau (communication, édition, stockage, support)
* **Messagerie électronique** : Système échange messages/fichiers asynchrone avec traçabilité et gestion agenda
* **Stockage de fichiers** : Service centralisé de conservation/partage/collaboration sur fichiers numériques
* **Suite bureautique** : Ensemble d'applications pour créer/éditer documents, tableurs, présentations
* **Prise de main à distance** : Technologie d'accès/contrôle ordinateur distant pour support et administration
* **SMB (Server Message Block)** : Protocole Windows partage fichiers/imprimantes en réseau local
* **Samba** : Implémentation open-source SMB pour Linux/Unix (interopérabilité avec Windows)
* **WebDAV** : Extension HTTP pour collaboration/gestion documents sur le Web (cloud storage)

---

### 💻 Commandes et configurations essentielles :

```bash
# 🐧 Linux - Samba (partage fichiers Windows-compatible)
sudo apt install samba                     # Installer Samba
sudo smbpasswd -a user                     # Ajouter utilisateur Samba
sudo systemctl restart smbd                # Redémarrer service

# Configuration /etc/samba/smb.conf
[partage]
   path = /srv/partage
   browseable = yes
   read only = no
   valid users = @groupe

# Tester configuration
testparm                                    # Vérifier syntaxe config

# Monter partage SMB
mount -t cifs //serveur/partage /mnt/smb -o username=user,password=pass
# ou via fstab
//serveur/partage /mnt/smb cifs credentials=/root/.smbcredentials,uid=1000 0 0
```

```bash
# 🐧 Linux - SSH (prise de main à distance)
ssh user@serveur                           # Connexion standard
ssh -X user@serveur                        # X11 forwarding (GUI)
ssh -L 8080:localhost:80 user@serveur     # Tunnel local (port forwarding)
ssh-copy-id user@serveur                   # Copier clé publique

# Transfert fichiers via SSH
scp fichier.txt user@serveur:/chemin/     # Copier fichier
sftp user@serveur                          # FTP sécurisé via SSH
```

```powershell
# 🪟 Windows - SMB partage réseau
New-SmbShare -Name "Partage" -Path "C:\Dossier" -FullAccess "Utilisateur"    # Créer partage
Get-SmbShare                                                                  # Lister partages
Remove-SmbShare -Name "Partage"                                              # Supprimer partage

# Désactiver SMBv1 (faille sécurité)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
Get-SmbServerConfiguration | Select EnableSMB1Protocol                       # Vérifier état

# Mapper lecteur réseau
net use Z: \\serveur\partage /persistent:yes                                 # Ligne commande
New-PSDrive -Name "Z" -PSProvider FileSystem -Root "\\serveur\partage"      # PowerShell
```

```powershell
# 🪟 Windows - RDP (Remote Desktop)
mstsc /v:serveur.domain.com                                  # Ouvrir client RDP
mstsc /v:serveur /admin                                      # Session admin console
mstsc /v:serveur /f                                          # Plein écran

# Activer RDP sur serveur
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"       # Autoriser firewall

# PowerShell remoting
Enter-PSSession -ComputerName serveur                        # Session interactive
Invoke-Command -ComputerName serveur -ScriptBlock {Get-Process}  # Commande unique
```

```bash
# 🌐 Protocoles messagerie
# SMTP (Simple Mail Transfer Protocol) - Port 25, 587 (TLS), 465 (SSL)
# Envoi de mails

# IMAP (Internet Message Access Protocol) - Port 143, 993 (SSL)
# Réception avec synchronisation serveur

# POP3 (Post Office Protocol v3) - Port 110, 995 (SSL)
# Réception avec téléchargement local
```

---

### ⚠️ Pièges à éviter :

* ❌ **Utiliser SMBv1** : Failles sécurité critiques (WannaCry exploitait SMBv1), TOUJOURS désactiver
* ❌ **RDP exposé Internet sans VPN** : Cible privilégiée attaques brute-force, ransomware
* ❌ **Partages réseau sans gestion droits** : Accès non autorisé aux données sensibles
* ❌ **Pas de 2FA sur prise de main à distance** : Compromission facile des accès admin
* ❌ **Mots de passe simples partages SMB** : Brute-force rapide sur protocole ancien
* ❌ **Oublier chiffrement stockage cloud** : Données accessibles au prestataire/pirates
* ❌ **Autoriser RDP depuis n'importe quelle IP** : Exposition inutile, limiter par IP source
* ❌ **Ne pas surveiller accès distants** : Connexions suspectes passent inaperçues
* ❌ **Partager données sensibles sans versioning** : Impossibilité récupérer version antérieure

---

### ✅ Bonnes pratiques :

* ✅ **Désactiver SMBv1, utiliser SMBv2/v3** : Sécurité renforcée + meilleures performances
* ✅ **VPN obligatoire pour RDP depuis Internet** : Jamais exposer directement RDP au Web
* ✅ **Activer 2FA sur tous services critiques** : Messagerie, cloud, prise de main à distance
* ✅ **Gestion fine des droits d'accès** : Principe du moindre privilège sur partages fichiers
* ✅ **Chiffrer données sensibles stockage cloud** : Protection confidentialité end-to-end
* ✅ **Logs et monitoring accès distants** : Détection intrusions, audit conformité
* ✅ **Former utilisateurs bonnes pratiques** : Phishing, mots de passe, partage responsable
* ✅ **Versioning automatique fichiers importants** : Récupération après erreur/corruption
* ✅ **Sauvegardes régulières données cloud** : Cloud ≠ sauvegarde, toujours dupliquer
* ✅ **Limiter accès RDP par IP/réseau** : Firewall restrictif, bastion/jump server

---

### 📚 Vocabulaire technique :

| Terme | Définition courte |
|-------|------------------|
| **SMB (Server Message Block)** | Protocole partage fichiers/imprimantes Windows sur réseau local |
| **Samba** | Implémentation SMB open-source pour Linux/Unix |
| **WebDAV** | Extension HTTP pour édition collaborative documents Web |
| **SMTP** | Protocole envoi mails (Simple Mail Transfer Protocol) |
| **IMAP** | Protocole réception mails avec sync serveur (messages restent serveur) |
| **POP3** | Protocole réception mails avec téléchargement local (suppression serveur) |
| **RDP (Remote Desktop Protocol)** | Protocole Microsoft prise de main à distance graphique |
| **RDWeb** | Accès RDP via navigateur web sans client dédié |
| **VNC (Virtual Network Computing)** | Protocole open-source partage écran/contrôle distant |
| **SSH (Secure Shell)** | Protocole chiffré accès distant ligne commande (+ X11 pour GUI) |
| **X11 Forwarding** | Affichage GUI applications Linux distantes via SSH |
| **Guacamole** | Passerelle web clientless pour RDP/VNC/SSH (HTML5) |
| **On-premises** | Hébergement local sur infrastructure entreprise (vs cloud) |
| **Workgroup** | Groupe de travail Windows sans domaine Active Directory |
| **NAS (Network Attached Storage)** | Serveur de stockage réseau dédié |

---

### 🎯 À retenir ABSOLUMENT :

1. 💡 **Théorique** : SMBv1 = DANGEREUX (failles critiques), toujours utiliser SMBv2/v3 minimum
2. 💻 **Pratique** : RDP sécurisé = VPN + 2FA + limitation IP source (jamais direct Internet)
3. ⚠️ **Piège** : Cloud storage ≠ sauvegarde → TOUJOURS dupliquer données critiques ailleurs

---

### 📧 Messagerie - Solutions courantes :

**On-premises (local) :**
- **Serveurs** : Microsoft Exchange, Zimbra, Postfix (Linux), MDaemon
- **Clients** : Outlook, Thunderbird, Apple Mail, Roundcube (webmail)

**Cloud (SaaS) :**
- **Google Workspace** (Gmail, Drive, Docs) - Intégration complète
- **Microsoft 365** (Exchange Online, OneDrive, Office) - Standard entreprise
- **Zoho Mail** - Alternative économique
- **ProtonMail** - Chiffrement end-to-end (confidentialité max)

**Protocoles messagerie :**
```
Envoi     : SMTP port 25/587(TLS)/465(SSL)
Réception : IMAP port 143/993(SSL) - synchronisation serveur
            POP3 port 110/995(SSL) - téléchargement local
```

---

### 💾 Stockage de fichiers - Solutions :

**Protocoles locaux :**
- **SMB/CIFS** : Standard Windows, aussi Samba sur Linux
- **NFS** : Standard Unix/Linux (Network File System)
- **AFP** : Apple Filing Protocol (macOS, obsolète)

**Cloud storage :**
- **Google Drive** : 15 Go gratuit, intégration Workspace
- **Microsoft OneDrive** : Intégration Office 365
- **Dropbox** : Synchronisation multi-device
- **Nextcloud** : Auto-hébergeable, open-source, RGPD-friendly

**Protocole cloud :** WebDAV (HTTP étendu)

---

### 📄 Suites bureautiques - Solutions :

**Locales (on-premises) :**
- **Microsoft Office** : Word, Excel, PowerPoint, Access (standard industrie)
- **LibreOffice** : Suite libre complète (Writer, Calc, Impress)
- **Apache OpenOffice** : Alternative libre
- **WPS Office** : Compatible Office, gratuit

**Cloud (collaboration temps réel) :**
- **Google Workspace** : Docs, Sheets, Slides (gratuit + versions payantes)
- **Microsoft 365** : Office Online (Word/Excel/PPT en ligne)
- **Zoho Workplace** : Alternative économique
- **OnlyOffice** : Auto-hébergeable + cloud

**Fonctionnalités clés cloud :**
- Édition collaborative temps réel
- Historique versions automatique
- Commentaires et suggestions
- Partage granulaire par lien
- Intégration stockage cloud

---

### 🖥️ Prise de main à distance - Solutions :

**Protocoles/Standards :**
- **SSH** : Ligne commande sécurisée (+ X11 forwarding pour GUI Linux)
- **RDP** : Standard Windows (graphique)
- **VNC** : Multi-plateforme open-source (VNC Connect, UltraVNC, TigerVNC)
- **SPICE** : Visualisation machines virtuelles (KVM/QEMU)

**Solutions cloud :**
- **TeamViewer** : Multi-plateforme, simple (protocole propriétaire)
- **AnyDesk** : Rapide, léger
- **Chrome Remote Desktop** : Via navigateur Chrome
- **LogMeIn** : Professionnel

**Solutions web (clientless) :**
- **Apache Guacamole** : Passerelle HTML5 pour RDP/VNC/SSH
- **RDWeb** : RDP via navigateur (Windows Server)

---

### 🔒 Sécurité services bureautiques :

**Messagerie :**
```bash
# SPF (Sender Policy Framework) - DNS
# Anti-spoofing, autorise serveurs envoi légitimes
v=spf1 ip4:203.0.113.0/24 include:_spf.google.com ~all

# DKIM (DomainKeys Identified Mail)
# Signature cryptographique mails sortants

# DMARC (Domain-based Message Authentication)
# Politique traitement mails SPF/DKIM échoués
```

**Filtrage spam/malware :**
- SpamAssassin, Amavis (on-premises)
- Solutions cloud intégrées (Microsoft Defender, Google Workspace)

**Stockage/Partage :**
- Droits NTFS/ACL granulaires
- Chiffrement au repos (BitLocker, LUKS)
- Chiffrement transit (SMB3, HTTPS)
- DLP (Data Loss Prevention)
- Audit trail (qui accède à quoi)

**Prise de main à distance :**
- VPN obligatoire (jamais exposition directe)
- 2FA/MFA systématique
- Limitation IP source (whitelist)
- Logging toutes connexions
- Sessions timeout automatique
- Chiffrement fort (AES-256)

---

### 📊 Comparaison on-premises vs cloud :

| Critère | On-premises | Cloud (SaaS) |
|---------|-------------|--------------|
| **Coût initial** | ⚠️ Élevé (licences, infra) | ✅ Faible (abonnement) |
| **Coût récurrent** | ✅ Maintenance seule | ⚠️ Abonnement permanent |
| **Contrôle données** | ✅ Total | ⚠️ Chez prestataire |
| **Scalabilité** | ⚠️ Limitée par infra | ✅ Quasi illimitée |
| **Maintenance** | ⚠️ Équipe interne | ✅ Prestataire |
| **Disponibilité** | ⚠️ Dépend de notre infra | ✅ SLA 99.9%+ |
| **Accès distant** | ⚠️ VPN nécessaire | ✅ Natif partout |
| **RGPD** | ✅ Maîtrisé | ⚠️ Dépend contrat |
| **Personnalisation** | ✅ Complète | ⚠️ Limitée |

---

### 🔧 Configuration rapide partage SMB sécurisé :

```bash
# Debian/Ubuntu - Samba
sudo apt install samba

# Créer répertoire partage
sudo mkdir -p /srv/partages/commun
sudo chown nobody:nogroup /srv/partages/commun
sudo chmod 0775 /srv/partages/commun

# Configuration /etc/samba/smb.conf
[global]
   workgroup = ENTREPRISE
   server string = Serveur Fichiers
   security = user
   map to guest = Bad User
   # Désactiver SMBv1
   min protocol = SMB2

[commun]
   path = /srv/partages/commun
   browseable = yes
   writable = yes
   guest ok = no
   valid users = @equipe
   create mask = 0664
   directory mask = 0775

# Ajouter utilisateurs
sudo smbpasswd -a franck
sudo usermod -aG equipe franck

# Redémarrer Samba
sudo systemctl restart smbd nmbd

# Test configuration
testparm

# Depuis Windows : \\serveur\commun
# Depuis Linux : smb://serveur/commun
```

---

### 🌐 Configuration RDP sécurisé :

```powershell
# Windows Server - Sécurisation RDP

# 1. Activer RDP (si nécessaire)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0

# 2. Autoriser uniquement groupe spécifique
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "DOMAINE\Admins-IT"

# 3. Niveau authentification réseau (NLA) obligatoire
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "UserAuthentication" -Value 1

# 4. Changer port par défaut (3389 → custom)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "PortNumber" -Value 13389

# 5. Firewall - limiter IP sources
New-NetFirewallRule -DisplayName "RDP Admin VPN" -Direction Inbound -LocalPort 13389 -Protocol TCP -Action Allow -RemoteAddress 10.0.0.0/24

# 6. Timeout session idle
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name "MaxIdleTime" -Value 900000  # 15 min

# 7. Activer logging connexions
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

---

### 📝 Checklist déploiement service bureautique :

**Phase analyse :**
- [ ] Identifier besoins métier (qui, quoi, où, quand)
- [ ] Évaluer criticité données/services
- [ ] Définir volumétrie (utilisateurs, stockage, bande passante)
- [ ] Choix on-premises vs cloud (coût, contrôle, conformité)

**Phase technique :**
- [ ] Dimensionner infrastructure (serveurs, stockage, réseau)
- [ ] Choisir solutions (messagerie, stockage, suite, remote)
- [ ] Planifier architecture (redondance, sauvegarde)
- [ ] Définir sécurité (chiffrement, 2FA, firewall)
- [ ] Configurer monitoring/alertes

**Phase déploiement :**
- [ ] Pilote sur groupe restreint
- [ ] Tests acceptation utilisateurs (UAT)
- [ ] Documentation procédures (admin + utilisateurs)
- [ ] Formation équipes (DSI + utilisateurs finaux)
- [ ] Migration données existantes
- [ ] Déploiement progressif

**Phase opérationnelle :**
- [ ] Support utilisateurs (helpdesk)
- [ ] Monitoring proactif
- [ ] Sauvegardes vérifiées
- [ ] Mises à jour sécurité
- [ ] Audit régulier accès/droits
- [ ] Révision annuelle besoins
