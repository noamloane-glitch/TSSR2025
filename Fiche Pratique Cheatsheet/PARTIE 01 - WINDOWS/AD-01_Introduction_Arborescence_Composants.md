# AD-01 – Introduction, Arborescence & Composants

**Tags** : #AD #LDAP #Arborescence #Domaine #Foret #DC #TSSR #CCP2

---

## 1. Définition

**Active Directory (AD)** = mise en œuvre Microsoft des services d'annuaire **LDAP** pour Windows. Base de données **hiérarchique** (≠ relationnelle) gérant de façon centralisée utilisateurs, ordinateurs, groupes et GPO.

| Caractéristique | Valeur |
|---|---|
| Apparu avec | Windows 2000 Server (1999) |
| Nom initial (1996) | NTDS (NT Directory Services) |
| Avant AD | Base **SAM** (locale, non hiérarchique, non évolutive) |
| Moteur de stockage | **ESENT** (remplace Jet) |
| Fichier base | `NTDS.dit` |
| 25% du marché gestion d'identité, max **2 milliards** d'objets |

---

## 2. Les 5 rôles Active Directory

| Rôle | Fonction |
|---|---|
| **AD DS** | Rôle principal — domaine, utilisateurs, GPO |
| **AD CS** | Certificats numériques, PKI |
| **AD FS** | SSO (Single Sign-On) inter-organisations |
| **AD RMS** | Droits numériques sur fichiers (Office, etc.) |
| **AD LDS** | Annuaire LDAP léger **sans domaine** |

---

## 3. Arborescence AD (du plus petit au plus grand)

```
Forêt
└── Arbre (espace de noms contigu)
    └── Domaine (unité admin + sécurité)
        └── OU (Unité d'Organisation)
            └── Objet (utilisateur, ordi, groupe…)
```

| Niveau | Description clé |
|---|---|
| **Objet** | Élément de base — instance d'une classe du schéma |
| **OU** | Conteneur logique pour organiser et appliquer des GPO |
| **Domaine** | Périmètre de gestion centralisée, unité de réplication |
| **Arbre** | Domaines partageant un espace de noms contigu + schéma commun |
| **Forêt** | Niveau le plus haut — arbres avec schéma + Catalogue Global communs |

### Exemple DN (Distinguished Name)
```
CN=Jean.Dupont,OU=Users,DC=lab,DC=lan
```

---

## 4. Workgroup vs Domaine

| Critère | Workgroup | Domaine |
|---|---|---|
| Centralisation | Aucune | Totale (via DC) |
| Nb de machines | Quelques dizaines | Illimité |
| Connexion | Locale uniquement | N'importe quel poste |
| Réseau requis | Même LAN | Réseaux différents OK |

---

## 5. Composants techniques

| Composant | Rôle |
|---|---|
| **DC** (Domain Controller) | Héberge NTDS.dit, gère authentification + GPO + réplication |
| **RODC** (Read Only DC) | DC en lecture seule → sites distants ou peu sécurisés |
| **Catalogue Global** | DC étendu avec réplica partiel de toute la forêt → recherches inter-domaines |

> ⚠️ DC = **SPOF** potentiel → toujours déployer **au minimum 2 DC** par domaine.

> ℹ️ Le **1er DC** d'une forêt cumule automatiquement les 5 rôles FSMO **et** le rôle Catalogue Global.

---

## 6. LDAP – Bases

**Ports** : `389` (LDAP) | `636` (LDAPS — chiffré TLS) → toujours privilégier LDAPS

**5 opérations fondamentales** : `bind` (auth) | `search` | `add` | `delete` | `modify`

**LDIF** (LDAP Data Interchange Format) : fichier texte pour import/export/modification en masse de l'annuaire.

```ldif
# Exemple LDIF — ajout d'un utilisateur
dn: CN=Jean.Dupont,OU=Users,DC=lab,DC=lan
changetype: add
objectClass: user
sAMAccountName: jdupont
userPrincipalName: jdupont@lab.lan
```

---

## 7. Commandes PowerShell essentielles

```powershell
# Infos domaine
Get-ADDomain

# Infos forêt
Get-ADForest

# Infos DC
Get-ADDomainController

# Recherche d'objets
Get-ADObject -Filter {Name -like "*dupont*"}

# Lister les OU
Get-ADOrganizationalUnit -Filter *
```

---

## 8. GUI

| Outil | Usage |
|---|---|
| `dsa.msc` | Utilisateurs et ordinateurs AD |
| `domain.msc` | Domaines et approbations AD |
| `dssite.msc` | Sites et services AD |
| `adsiedit.msc` | Éditeur ADSI (attributs bruts LDAP) |
