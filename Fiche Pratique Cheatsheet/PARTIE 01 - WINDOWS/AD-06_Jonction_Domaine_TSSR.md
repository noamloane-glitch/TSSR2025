# AD-06 — Jonction au domaine & gestion des connexions

> Intégrer un poste Windows au domaine AD pour centraliser l'authentification.  
> **CCP2** — compétence évaluée au titre : "Intégrer un poste client au domaine".

---

## Prérequis avant la jonction

| Prérequis | Détail |
|-----------|--------|
| **DNS pointant vers le DC** | Le poste doit résoudre le nom du domaine via le DC |
| **Connectivité réseau** | Joignabilité du DC sur le réseau |
| **Compte avec droits** | Compte Admin du domaine (ou délégué) |
| **Nom d'ordinateur unique** | Pas de doublon dans AD |

> ⚠️ Le DNS est le prérequis n°1 — sans lui, la jonction échoue même si le réseau fonctionne.

---

## GUI — Jonction au domaine (Windows)

### Méthode 1 : via Système (classique)

1. Clic droit sur `Ce PC` → `Propriétés`
2. Cliquer `Modifier les paramètres` → onglet `Nom de l'ordinateur`
3. Cliquer `Modifier`
4. Sélectionner `Domaine` → saisir le nom du domaine (ex: `lab.lan`)
5. Saisir les **identifiants** d'un compte Admin du domaine
6. `OK` → **Redémarrer**

### Méthode 2 : via Paramètres (Windows 10/11)

1. `Paramètres` → `Système` → `Informations système`
2. `Joindre un domaine` → saisir le nom du domaine
3. Saisir les identifiants Admin → `OK` → Redémarrer

### Vérifier la jonction (GUI)

- `Ce PC` → `Propriétés` → vérifier `Domaine : lab.lan`
- Dans ADUC sur le DC : vérifier que l'ordinateur apparaît dans `CN=Computers` ou l'OU cible

---

## PowerShell — Jonction au domaine

```powershell
# Joindre le domaine (redémarrage requis)
Add-Computer -DomainName "lab.lan" `
             -Credential (Get-Credential) `
             -Restart

# Joindre ET placer directement dans une OU spécifique
Add-Computer -DomainName "lab.lan" `
             -OUPath "OU=LabComputers,DC=lab,DC=lan" `
             -Credential (Get-Credential) `
             -Restart

# Joindre sans redémarrage immédiat
Add-Computer -DomainName "lab.lan" `
             -Credential (Get-Credential)

# Vérifier le domaine actuel
(Get-WmiObject Win32_ComputerSystem).Domain

# Quitter le domaine (retour workgroup)
Remove-Computer -UnjoinDomainCredential (Get-Credential) `
                -WorkgroupName "WORKGROUP" -Restart

# Renommer le PC avant jonction
Rename-Computer -NewName "PC-CLIENT01" -Restart
```

---

## Déplacer un ordinateur dans une OU (après jonction)

### GUI

1. Dans ADUC, trouver l'ordinateur dans `CN=Computers`
2. Clic droit → `Déplacer`
3. Sélectionner l'OU cible (ex: `LabComputers`) → `OK`

### PowerShell

```powershell
# Déplacer le compte ordinateur vers une OU
Move-ADObject -Identity "CN=PC-CLIENT01,CN=Computers,DC=lab,DC=lan" `
              -TargetPath "OU=LabComputers,DC=lab,DC=lan"
```

---

## Restrictions de connexion utilisateur

> Éléments fréquemment demandés en checkpoint (Q.1.2 checkpoint 3).

### Restriction des horaires de connexion (GUI)

1. Dans ADUC, double-clic sur l'utilisateur → onglet `Compte`
2. Cliquer `Logon Hours` (Heures d'ouverture de session)
3. Sélectionner les plages horaires autorisées (bleu = autorisé)
4. `OK`

> Cas checkpoint : "Faire en sorte que l'utilisateur ne puisse se connecter que du lundi au vendredi, de 7h à 17h"

### Restriction des postes de connexion (GUI)

1. Dans ADUC, double-clic sur l'utilisateur → onglet `Compte`
2. Cliquer `Log On To` (Se connecter à)
3. Sélectionner `The following computers` → ajouter les noms de machines autorisées
4. `OK`

### PowerShell — Restrictions

```powershell
# Voir les heures de connexion autorisées
Get-ADUser -Identity "jean.dupont" -Properties LogonHours |
    Select Name, LogonHours

# Restreindre les postes de connexion autorisés
Set-ADUser -Identity "jean.dupont" `
           -LogonWorkstations "PC-CLIENT01,PC-CLIENT02"

# Supprimer les restrictions de postes
Set-ADUser -Identity "jean.dupont" -LogonWorkstations $null

# Définir une expiration de compte
Set-ADAccountExpiration -Identity "jean.dupont" `
                        -DateTime "2025-12-31"

# Supprimer l'expiration
Clear-ADAccountExpiration -Identity "jean.dupont"
```

---

## Profil utilisateur de domaine

### Types de profils

| Type | Stockage | Persistance | Usage |
|------|----------|-------------|-------|
| **Local** | `C:\Users\<user>` sur le poste | Reste sur la machine | Standard |
| **Itinérant** | `\\serveur\Profils\<user>` | Synchronisé sur tous les postes | Mobilité |
| **Obligatoire** | Serveur (lecture seule) | Jamais sauvegardé | Kiosque |

### Configurer un profil itinérant (GUI)

1. Double-clic sur l'utilisateur dans ADUC → onglet `Profil`
2. Dans `Chemin du profil` : saisir `\\serveur\Profils\%username%`
3. `OK`

> `%username%` = variable automatiquement remplacée par le login de l'utilisateur.

### Configurer via PowerShell

```powershell
# Définir un profil itinérant
Set-ADUser -Identity "jean.dupont" `
           -ProfilePath "\\SRVWIN01\Profils\jean.dupont"

# Définir un script de logon
Set-ADUser -Identity "jean.dupont" `
           -ScriptPath "logon.bat"

# Définir un dossier de base (home)
Set-ADUser -Identity "jean.dupont" `
           -HomeDirectory "\\SRVWIN01\DossiersIndividuels\jean.dupont" `
           -HomeDrive "H:"
```

---

## Workgroup vs Domaine — Rappel

| Critère | Workgroup | Domaine |
|---------|-----------|---------|
| **Comptes** | Locaux sur chaque machine | Centralisés sur le DC |
| **Connexion** | Uniquement sur la machine locale | Sur n'importe quel poste du domaine |
| **GPO** | Stratégies locales uniquement | GPO centralisées |
| **Limite** | Quelques dizaines de postes | Illimité |
| **Administration** | Machine par machine | Centralisée depuis le DC |
| **DNS requis** | Non | ✅ Oui (pointe vers le DC) |

---

## Commandes de diagnostic post-jonction

| Commande | Rôle |
|----------|------|
| `whoami /fqdn` | Afficher le nom complet de l'utilisateur connecté |
| `whoami /groups` | Lister les groupes de l'utilisateur connecté |
| `nltest /dsgetdc:lab.lan` | Trouver un DC du domaine |
| `net config workstation` | Voir le domaine actuel du poste |
| `klist` | Voir les tickets Kerberos actifs |
| `gpresult /r` | GPO appliquées à l'utilisateur/ordinateur |
| `gpupdate /force` | Forcer l'application des GPO |
| `Test-ComputerSecureChannel` | Vérifier le lien sécurisé poste-DC (PowerShell) |
| `Test-ComputerSecureChannel -Repair` | Réparer le lien sécurisé si cassé |

---

## ⚠️ À retenir absolument (checkpoints + REAC)

- DNS vers le DC = **prérequis absolu** avant la jonction
- Après jonction : l'ordinateur apparaît dans `CN=Computers` → le **déplacer dans l'OU** appropriée
- `Add-Computer -OUPath` = jonction + placement OU en une seule commande
- Restriction horaires = onglet `Compte` → `Logon Hours` dans ADUC
- Restriction postes = onglet `Compte` → `Log On To` dans ADUC
- Profil itinérant = onglet `Profil` → `\\serveur\Profils\%username%`
- `%username%` en GUI = `%LogonUser%` dans GPO Drive Maps
- `Test-ComputerSecureChannel -Repair` = réparer la relation de confiance poste/domaine (canal sécurisé cassé)
- `CCP2` : "Intégrer un poste client au domaine" — compétence évaluée au titre TSSR
