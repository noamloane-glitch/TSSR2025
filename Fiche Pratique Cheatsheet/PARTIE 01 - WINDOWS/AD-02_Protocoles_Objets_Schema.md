# AD-02 – Protocoles Réseau, Objets & Schéma AD

**Tags** : #AD #Protocoles #Kerberos #DNS #SNTP #Schema #Objets #TSSR #CCP2

---

## 1. Protocoles réseau associés à AD

| Protocole | Rôle dans AD | Point clé |
|---|---|---|
| **DNS** | Résolution noms + localisation services (SRV) | **Obligatoire** — sans DNS, AD ne fonctionne pas |
| **SNTP** | Synchronisation temporelle | Écart max **5 min** pour Kerberos |
| **LDAP** | Accès et requêtes annuaire | Port `389` ; LDAPS `636` (chiffré) |
| **Kerberos** | Authentification par défaut depuis Win2000 | Authentification **mutuelle**, tickets TGT |
| **X.509** | Certificats numériques (PKI via AD CS) | Auth, chiffrement, non-répudiation |
| **NTFS** | Permissions fichiers/dossiers liées à AD | Toujours via groupes → méthode AGDLP |

### DNS – double rôle
```
Résolution de noms  : serveur1.lab.lan → 10.10.1.2
Résolution services : _ldap._tcp.lab.lan  (SRV records → localisation DC)
                      _kerberos._tcp.lab.lan
```
> ⚠️ **DNS défaillant = AD défaillant** → toujours au moins 2 serveurs DNS (= les DC)

### SNTP – chaîne de synchronisation
```
Source NTP externe → PDC Emulator → autres DC → clients
```
> ⚠️ Déphasage > 5 min → échec authentification Kerberos, échecs de réplication, logs incohérents

### Kerberos – processus d'authentification
```
1. Client → KDC : demande TGT
2. KDC → Client : TGT chiffré (vérifie identité)
3. Client → KDC : présente TGT, demande ticket de service
4. Client → Serveur ressource : présente ticket de service
5. Serveur : valide → accès accordé
```
Tickets valides **10h** par défaut | Kerberos V5 | Remplace LM/NTLM

### X.509 – usages courants
Carte à puce | **EFS** (chiffrement fichiers) | **IPsec** | **HTTPS/TLS** | LDAPS

---

## 2. Objets AD – Structure

### 3 types d'objets

| Type | Exemples |
|---|---|
| **Ressources** | Postes, serveurs, imprimantes, dossiers partagés |
| **Utilisateurs** | Comptes individuels, groupes sécurité, groupes distribution |
| **Services** | Messagerie, applications, SCP |

### Classes d'objets (ObjectClass)

| Classe | Conteneur ? | Description |
|---|---|---|
| `Container` | ✅ | Conteneur natif AD |
| `OrganizationalUnit` | ✅ | OU — peut contenir objets + GPO |
| `Group` | ✅ | Groupe de sécurité ou distribution |
| `User` | ❌ | Compte utilisateur |
| `Computer` | ❌ | Poste ou serveur |
| `Contact` | ❌ | Externe, sans compte Windows |

### Identifiants uniques

| ID | Portée | Immuable | Attribut AD | Usage |
|---|---|---|---|---|
| **GUID** | Forêt | ✅ Oui | `ObjectGUID` | Référence permanente (128 bits) |
| **SID** | Domaine | ❌ Non | `ObjectSID` | Contrôle d'accès (Domain SID + RID) |
| **DN** | Forêt | ❌ Non | — | Chemin LDAP complet |

```
Exemple DN   : CN=Jean.Dupont,OU=Users,DC=lab,DC=lan
Exemple SID  : S-1-5-21-156063872-1535639461-3779917529-1134
Exemple GUID : 3F2504E0-4F89-11D3-9A0C-0305E82C3301
```

### Attributs clés d'un objet utilisateur

```
Identifiants    : sAMAccountName, userPrincipalName, ObjectGUID, ObjectSID
Infos perso     : givenName, sn, displayName, mail, telephoneNumber
Infos système   : lastLogon, pwdLastSet, accountExpires, badPwdCount
Appartenance    : memberOf
```

---

## 3. Schéma AD

Définit **toutes les classes d'objets** et **tous les attributs** possibles dans la forêt.

> ⚠️ Modification du schéma : nécessite le rôle **Schema Master**, impacte **toute la forêt**, opération **irréversible**. Outil : `schmmgmt.msc`

---

## 4. Niveau fonctionnel

Détermine les **fonctionnalités AD disponibles** selon la version Windows Server la plus ancienne des DC.

> ⚠️ Montée de niveau **irréversible** — vérifier la compatibilité avant toute opération.

```powershell
# Voir le niveau fonctionnel
Get-ADDomain | Select-Object DomainMode
Get-ADForest | Select-Object ForestMode

# Monter le niveau fonctionnel du domaine
Set-ADDomainMode -Identity lab.lan -DomainMode Windows2016Domain

# Monter le niveau fonctionnel de la forêt
Set-ADForestMode -Identity lab.lan -ForestMode Windows2016Forest
```

---

## 5. Commandes PowerShell

```powershell
# Lister tous les attributs d'un utilisateur
Get-ADUser -Identity jdupont -Properties *

# Lister les classes d'objets disponibles
Get-ADObject -SearchBase (Get-ADRootDSE).schemaNamingContext -Filter * | Select Name

# Rechercher par ObjectClass
Get-ADObject -Filter {ObjectClass -eq "computer"}

# Voir le GUID et SID d'un objet
Get-ADUser jdupont | Select-Object ObjectGUID, SID
```

---

## 6. GUI

| Outil | Usage |
|---|---|
| `dsa.msc` | Utilisateurs et ordinateurs AD |
| `schmmgmt.msc` | Éditeur de schéma AD (à activer via `regsvr32 schmmgmt.dll`) |
| `adsiedit.msc` | Éditeur ADSI — attributs bruts LDAP |
| `dnsmgmt.msc` | Console DNS — vérifier enregistrements SRV |
