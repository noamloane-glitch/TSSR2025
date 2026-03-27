# AD-07 – FSMO, Réplication & Bonnes Pratiques

**Tags** : #AD #FSMO #Replication #Securite #TSSR #CCP2

---

## 1. Les 5 rôles FSMO

| Rôle | Niveau | Qté | Fonction principale | Criticité |
|---|---|---|---|---|
| **Schema Master** | Forêt | 1 | Seul DC à modifier le schéma AD | Critique |
| **Domain Naming Master** | Forêt | 1 | Ajouter/supprimer des domaines | Importante |
| **RID Master** | Domaine | 1/dom | Distribue les pools de RID (blocs 500) → compose les SID | Critique |
| **Infrastructure Master** | Domaine | 1/dom | Gère les références croisées inter-domaines | Moyenne |
| **PDC Emulator** | Domaine | 1/dom | Temps, MDP, verrouillage comptes, GPO | **Très critique** |

> ⚠️ Par défaut, le 1er DC cumule les **5 rôles** → à redistribuer dès que possible.

---

## 2. Identifier les rôles FSMO

```powershell
# Tous les rôles en une commande
netdom query fsmo

# Rôles domaine
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster

# Rôles forêt
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
```

---

## 3. Réplication AD

**Données répliquées** : base NTDS.dit | GPO via SYSVOL | scripts | zones DNS intégrées

| Contexte | Intervalle |
|---|---|
| Intra-site | 5 min |
| Inter-site | 180 min (configurable) |

**Processus (8 étapes)** : DC1 modifié → interroge **KCC** → KCC identifie DC2 → interroge **DNS** → DNS localise DC2 → DC1 notifie DC2 → DC2 demande les modifs → réplication effectuée.

```powershell
# Forcer la réplication
repadmin /syncall /AdeP

# Vérifier l'état de la réplication
repadmin /replsummary
repadmin /showrepl
```

**KCC (Knowledge Consistency Checker)** : service présent sur tous les DC, génère automatiquement la topologie de réplication. Se recalcule si changement détecté.

---

## 4. Bonnes pratiques FSMO

```
DC1 → Schema Master + Domain Naming Master   (rôles forêt)
DC2 → RID Master + Infrastructure Master + PDC Emulator  (rôles domaine)
Idéal → 1 rôle par DC (5 DC)
```

> ⚠️ **Infrastructure Master** : ne jamais placer sur un DC Catalogue Global (sauf si **tous** les DC sont GC).

> ✅ **PDC Emulator** : toujours sur le serveur le plus performant / site principal.

---

## 5. Sécurité & Bonnes pratiques AD

### Tiering Model Microsoft

| Tier | Périmètre | Comptes |
|---|---|---|
| **Tier 0** | DC, Enterprise Admins | Comptes les plus sensibles |
| **Tier 1** | Serveurs applicatifs, SCCM, WSUS | Admins serveurs |
| **Tier 2** | Postes utilisateurs | Comptes standard |

> Isolation stricte entre les tiers — un compte Tier 2 ne doit **jamais** se connecter sur un Tier 0.

### JIT & JEA

| Principe | Définition |
|---|---|
| **JEA** (Just Enough Administration) | Limiter les privilèges aux actions strictement nécessaires |
| **JIT** (Just-In-Time) | Élévation **temporaire** des privilèges, durée limitée |

> Combinés → sécurité optimale contre la compromission de comptes admin.

### LAPS

`LAPS` (Local Administrator Password Solution) : gestion automatique et rotation des mots de passe des comptes admin **locaux** sur chaque machine. Stocké dans AD, lisible uniquement par les admins habilités.

### Sécurité réseau

- DC dans un **VLAN dédié** (voire DMZ)
- Utiliser **LDAPS** (port `636`) au lieu de LDAP (port `389`)
- MAJ régulières (patch management)
- Auditer les Event IDs clés : `4720` (création compte), `4728` (ajout groupe), `4625` (échec auth)

### Sauvegarde

Règle **3-2-1** : 3 copies | 2 supports différents | 1 hors site

---

## 6. Ne pas confondre !

| Type | Exemples | Où les gérer |
|---|---|---|
| **Rôles serveur Windows** | DHCP, DNS, IIS, Hyper-V | Gestionnaire de serveur |
| **Rôles Active Directory** | AD DS, AD CS, AD FS, AD RMS, AD LDS | Gestionnaire de serveur |
| **Rôles FSMO** | Schema Master, RID Master, PDC Emulator… | Outils AD / PowerShell |

---

## 7. GUI

| Outil | Usage |
|---|---|
| `dsa.msc` | Utilisateurs et ordinateurs AD |
| `Active Directory Sites and Services` | Gestion des sites et réplication |
| `ntdsutil` | Saisie autoritaire des rôles FSMO (si DC défaillant) |

```powershell
# Transfert de rôle FSMO (si DC source disponible)
Move-ADDirectoryServerOperationMasterRole -Identity "DC2" -OperationMasterRole PDCEmulator

# Saisie autoritaire (si DC source hors ligne) → via ntdsutil
ntdsutil
# > roles → connections → connect to server DC2 → quit
# > seize PDC
```
