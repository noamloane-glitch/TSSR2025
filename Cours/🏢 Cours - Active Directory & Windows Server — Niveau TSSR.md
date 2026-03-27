# 🏢 Cours : Active Directory & Windows Server — Niveau TSSR

> **Objectif** : Comprendre la structure d'Active Directory, savoir gérer les objets, les GPO, l'authentification, et diagnostiquer les problèmes courants en environnement Windows Server.

---

## PARTIE 1 — Les bases d'Active Directory

---

## 1. Qu'est-ce qu'Active Directory ?

**Active Directory (AD)** est un service d'annuaire développé par Microsoft. Il permet de **centraliser la gestion des utilisateurs, des machines et des ressources** d'un réseau d'entreprise.

Sans AD → chaque PC gère ses propres utilisateurs localement. Avec AD → un seul annuaire central gère tout le réseau.

### Ce qu'Active Directory permet

|Fonctionnalité|Exemple concret|
|---|---|
|Authentification centralisée|Un utilisateur se connecte avec le même compte sur n'importe quel PC du domaine|
|Gestion des droits|Jean peut accéder au dossier RH, pas Paul|
|Application de politiques|Tous les PC du service comptabilité ont le fond d'écran de l'entreprise|
|Inventaire du réseau|AD connaît tous les PC, serveurs, imprimantes du domaine|
|Délégation d'administration|Le helpdesk peut réinitialiser les mots de passe sans être admin du domaine|

> 💡 **Analogie** : AD c'est comme le service RH d'une entreprise. Il gère qui est qui, qui a accès à quoi, et s'assure que tout le monde respecte les règles.

---

## 2. Les composants fondamentaux d'AD

### Le domaine

C'est l'unité de base d'AD. Un domaine regroupe des objets (utilisateurs, machines, groupes) sous une même administration.

Exemple : `entreprise.local` ou `entreprise.com`

### La forêt

C'est l'ensemble de tous les domaines AD d'une organisation. La forêt est la frontière de sécurité ultime.

```
Forêt : entreprise.com
├── Domaine racine : entreprise.com
│   ├── Sous-domaine : rh.entreprise.com
│   └── Sous-domaine : it.entreprise.com
```

### L'arbre (Tree)

Ensemble de domaines qui partagent un espace de noms contigu (même suffixe DNS).

### Le contrôleur de domaine (DC)

C'est le serveur qui **héberge et gère Active Directory**. Il s'occupe de l'authentification, de la réplication, et stocke l'annuaire. Il doit y en avoir **au minimum deux** pour la redondance.

---

## 3. La structure logique d'AD — Unités d'Organisation (OU)

Les **Unités d'Organisation (OU)** permettent de **structurer les objets** dans l'annuaire et d'y **appliquer des GPO**.

```
entreprise.local
├── OU : Direction
│   ├── Utilisateur : Jean Dupont (PDG)
├── OU : Informatique
│   ├── Utilisateur : Marie Martin (Admin)
│   └── Ordinateur : PC-ADMIN-01
├── OU : Comptabilité
│   ├── Utilisateur : Pierre Durand
│   └── Utilisateur : Sophie Bernard
│   └── OU : Stagiaires
│       └── Utilisateur : Lucas Martin
```

> 💡 Les OU s'organisent selon la **structure de l'entreprise** (par département, par site, par fonction). C'est au choix de l'administrateur.

---

## 4. Les objets Active Directory

|Objet|Description|Exemple|
|---|---|---|
|**Utilisateur**|Compte d'une personne|jean.dupont|
|**Ordinateur**|Machine membre du domaine|PC-COMPTA-01|
|**Groupe**|Ensemble d'utilisateurs ou d'ordinateurs|GRP_Comptabilité|
|**OU**|Conteneur logique pour organiser les objets|OU=Comptabilité|
|**GPO**|Politique de configuration appliquée à une OU|Fond d'écran entreprise|
|**Contact**|Adresse email externe (pas de connexion)|client@partenaire.com|

---

## 5. Les groupes AD — types et étendues

### Types de groupes

|Type|Rôle|
|---|---|
|**Sécurité**|Attribuer des droits d'accès aux ressources|
|**Distribution**|Liste de diffusion email (pas de droits)|

### Étendues des groupes (scope)

|Étendue|Membres possibles|Utilisable dans|
|---|---|---|
|**Local de domaine**|Utilisateurs, groupes globaux de n'importe quel domaine|Ressources du domaine local uniquement|
|**Global**|Utilisateurs et groupes du même domaine|N'importe quel domaine de la forêt|
|**Universel**|Utilisateurs et groupes de n'importe quel domaine|N'importe quel domaine de la forêt|

### La stratégie AGDLP — à retenir absolument

**A**ccount → **G**lobal group → **D**omain **L**ocal group → **P**ermissions

```
Utilisateurs → mis dans un Groupe Global
Groupe Global → ajouté à un Groupe Local de Domaine
Groupe Local de Domaine → reçoit les permissions sur la ressource
```

**Exemple concret** :

- Jean, Pierre, Sophie → Groupe Global `GG_Comptables`
- `GG_Comptables` → ajouté à Groupe Local `GL_Accès_Partage_Compta`
- `GL_Accès_Partage_Compta` → Permission Lecture/Écriture sur `\\Serveur\Compta`

> 💡 Cette méthode facilite la gestion : pour donner l'accès à un nouveau comptable, on l'ajoute juste au groupe global. Les permissions restent inchangées.

---

## PARTIE 2 — L'authentification dans AD

---

## 6. Comment fonctionne l'authentification — Kerberos

AD utilise le protocole **Kerberos** pour authentifier les utilisateurs. DNS est indispensable au fonctionnement de Kerberos.

### Le processus simplifié

```
1. L'utilisateur entre son identifiant + mot de passe
2. Le DC vérifie les credentials → délivre un TGT (Ticket Granting Ticket)
3. Quand l'utilisateur accède à une ressource, il présente son TGT
4. Le DC délivre un ticket de service spécifique à la ressource
5. L'utilisateur accède à la ressource avec ce ticket
```

> 💡 L'utilisateur ne saisit son mot de passe qu'une seule fois → c'est le **SSO (Single Sign-On)**. Ensuite, les tickets Kerberos gèrent tout en arrière-plan.

### Pourquoi DNS est critique pour Kerberos

Kerberos utilise les noms DNS pour identifier les contrôleurs de domaine. Si le DNS est cassé, l'authentification AD échoue — même si le réseau IP fonctionne.

**Symptôme classique** : utilisateurs qui ne peuvent plus se connecter le matin → vérifier DNS en premier.

---

## PARTIE 3 — Les GPO (Group Policy Objects)

---

## 7. Qu'est-ce qu'une GPO ?

Une **GPO (Group Policy Object)** est un ensemble de paramètres de configuration qui s'applique automatiquement aux **utilisateurs** et aux **ordinateurs** d'une OU.

### Ce qu'on peut configurer avec les GPO

|Catégorie|Exemples|
|---|---|
|Sécurité|Longueur minimale du mot de passe, verrouillage après X tentatives|
|Bureau|Fond d'écran imposé, accès au panneau de configuration désactivé|
|Réseau|Lecteur réseau mappé automatiquement, proxy configuré|
|Logiciels|Installation automatique d'un logiciel sur tous les PC|
|Scripts|Script de démarrage/arrêt, script de connexion utilisateur|
|Windows Update|Forcer les mises à jour à une heure précise|

---

## 8. Comment les GPO s'appliquent — LSDOU

Les GPO s'appliquent dans un ordre précis, et en cas de conflit **la dernière appliquée gagne**.

```
L → Local        : GPO locale du PC
S → Site         : GPO du site AD
D → Domain       : GPO liée au domaine
O → OU           : GPO liée à l'OU (puis sous-OU, puis sous-sous-OU...)
```

> 💡 **Exemple** : Si la GPO de domaine impose un fond d'écran bleu, et la GPO de l'OU Comptabilité impose un fond d'écran rouge → les comptables auront le fond rouge (OU gagne sur Domain).

### Héritage et blocage

- Par défaut, les GPO des OU parentes s'appliquent aux OU enfants (**héritage**)
- On peut **bloquer l'héritage** sur une OU enfant (elle n'hérite plus des GPO parentes)
- On peut **forcer l'application** d'une GPO (elle s'applique même si l'héritage est bloqué)

---

## 9. Filtrage des GPO

On peut affiner qui reçoit une GPO avec deux mécanismes :

|Mécanisme|Rôle|
|---|---|
|**Filtrage de sécurité**|La GPO ne s'applique qu'aux membres d'un groupe spécifique|
|**Filtrage WMI**|La GPO ne s'applique qu'aux machines répondant à une condition (ex: Windows 11 uniquement)|

---

## PARTIE 4 — Les rôles FSMO

---

## 10. Les 5 rôles FSMO

AD répartit certaines responsabilités critiques sur des contrôleurs de domaine spécifiques via les **rôles FSMO (Flexible Single Master Operations)**. Un seul DC peut avoir chaque rôle à la fois.

|Rôle|Niveau|Rôle|
|---|---|---|
|**Schema Master**|Forêt|Gère les modifications du schéma AD|
|**Domain Naming Master**|Forêt|Gère l'ajout/suppression de domaines dans la forêt|
|**RID Master**|Domaine|Attribue des blocs d'identifiants uniques aux DC|
|**PDC Emulator**|Domaine|Synchronisation de l'heure, gestion des mots de passe, compatibilité|
|**Infrastructure Master**|Domaine|Gère les références entre objets de domaines différents|

> 💡 En pratique, le **PDC Emulator** est le plus critique au quotidien. Si les utilisateurs ont des problèmes de mot de passe ou de synchronisation horaire, c'est souvent lui qu'on regarde en premier.

---

## PARTIE 5 — Scénarios de dépannage

---

### 🔴 Scénario 1 — Un utilisateur ne peut pas se connecter au domaine

**Situation** :

- Jean tente de se connecter sur son PC le matin
- Message : "Le service d'ouverture de session n'est pas disponible"
- Les autres utilisateurs se connectent normalement
- Jean peut se connecter avec son compte local

**Questions à se poser** :

1. Le PC de Jean est-il bien membre du domaine ?
2. Le PC de Jean peut-il joindre le contrôleur de domaine ?
3. Le DNS fonctionne-t-il sur le PC de Jean ?
4. Le compte de Jean est-il verrouillé ou expiré dans AD ?
5. Y a-t-il un problème de canal sécurisé entre le PC et le DC ?

**Analyse** :

|Vérification|Résultat possible|
|---|---|
|Autres utilisateurs OK|Problème lié au compte ou au PC de Jean|
|Connexion compte local OK|Le PC fonctionne, le problème est l'authentification AD|
|DNS KO sur le PC|Jean ne trouve pas le DC → Kerberos échoue|

**Marche à suivre** :

1. Vérifier le compte dans AD : `Utilisateurs et ordinateurs AD` → compte verrouillé ? expiré ? désactivé ?
2. Sur le PC de Jean : `nslookup entreprise.local` → le DNS résout-il le domaine ?
3. `ping dc01.entreprise.local` → le DC est-il joignable ?
4. Tester le canal sécurisé : `nltest /sc_verify:entreprise.local`
5. Si canal cassé : `netdom resetpwd /server:DC01 /userd:admin /passwordd:*` ou rejoindre le domaine

**Commandes utiles** :

```powershell
# Vérifier le canal sécurisé
nltest /sc_verify:entreprise.local

# Voir quel DC a authentifié l'utilisateur
nltest /dsgetdc:entreprise.local

# Réinitialiser le canal sécurisé (PowerShell)
Reset-ComputerMachinePassword -Server DC01 -Credential (Get-Credential)

# Vérifier le compte utilisateur (AD)
Get-ADUser jean.dupont -Properties LockedOut, Enabled, PasswordExpired
```

> 💡 **Cause la plus fréquente** : le compte est verrouillé après plusieurs tentatives de mot de passe incorrect. Toujours commencer par vérifier ça dans AD.

---

### 🔴 Scénario 2 — Une GPO ne s'applique pas

**Situation** :

- Tu as créé une GPO pour imposer un fond d'écran sur les PC de la comptabilité
- La GPO est liée à l'OU `Comptabilité`
- Après redémarrage, les PC de comptabilité n'ont toujours pas le fond d'écran

**Questions à se poser** :

1. La GPO est-elle bien liée à la bonne OU ?
2. La GPO est-elle activée ?
3. Le filtrage de sécurité est-il correctement configuré ?
4. Y a-t-il une GPO de priorité plus haute qui écrase celle-ci ?
5. Est-ce un paramètre utilisateur ou ordinateur ? Le bon type est-il configuré ?

**Analyse** :

|Vérification|Ce qu'on cherche|
|---|---|
|Lien GPO|La GPO est-elle liée à l'OU qui contient les PC et non pas les utilisateurs ?|
|Filtrage sécurité|Les PC ou utilisateurs sont-ils membres du groupe filtré ?|
|Priorité|Une autre GPO avec priorité plus haute définit-elle un autre fond d'écran ?|
|Utilisateur vs Ordinateur|Un paramètre "Configuration utilisateur" ne s'applique qu'aux utilisateurs, pas aux PC|

**Marche à suivre** :

1. Sur le PC : `gpresult /r` → voir les GPO appliquées et celles refusées + pourquoi
2. `gpresult /h rapport.html` → rapport complet avec toutes les GPO
3. Forcer l'application : `gpupdate /force` sur le PC
4. Dans la console GPMC, vérifier le lien, le filtrage et l'ordre de priorité
5. Vérifier que les PC sont bien dans l'OU ciblée (et pas dans l'OU `Computers` par défaut)

**Commandes utiles** :

```powershell
# Voir les GPO appliquées
gpresult /r

# Rapport HTML complet
gpresult /h C:\rapport.html

# Forcer l'application des GPO
gpupdate /force

# Forcer + redémarrage si nécessaire
gpupdate /force /boot
```

> ⚠️ **Erreur classique** : les objets ordinateurs sont restés dans le conteneur `Computers` par défaut (pas une OU). Les GPO ne s'appliquent qu'aux **OU**, pas aux conteneurs par défaut.

---

### 🔴 Scénario 3 — Un utilisateur ne peut pas accéder à un partage réseau

**Situation** :

- Marie tente d'accéder à `\\Serveur\Compta`
- Message : "Accès refusé"
- Son collègue Pierre accède normalement au même partage
- Marie et Pierre ont tous les deux le même profil métier

**Questions à se poser** :

1. Marie est-elle membre du groupe qui a accès au partage ?
2. Y a-t-il des permissions NTFS qui bloquent Marie spécifiquement ?
3. Les permissions de partage et les permissions NTFS sont-elles cohérentes ?
4. Le compte de Marie est-il actif et non verrouillé ?

**Analyse** :

En environnement Windows, l'accès à un partage est contrôlé par **deux niveaux de permissions** qui s'appliquent en même temps :

```
Permission effective = Intersection (permissions partage ∩ permissions NTFS)
```

La permission la plus restrictive des deux l'emporte.

|Niveau|Emplacement|Ce qu'il contrôle|
|---|---|---|
|**Permissions de partage**|Sur le dossier partagé|Accès réseau|
|**Permissions NTFS**|Sur le système de fichiers|Accès local ET réseau|

**Marche à suivre** :

1. Vérifier les groupes de Marie : `Get-ADUser marie.martin -Properties MemberOf`
2. Comparer avec Pierre : voir dans quels groupes Pierre est et Marie n'est pas
3. Sur le serveur : vérifier les permissions NTFS du dossier `Compta`
4. Vérifier les permissions de partage (clic droit → Partage → Autorisations)
5. Ajouter Marie au bon groupe → attendre la réplication ou faire `gpupdate`

**Commandes utiles** :

```powershell
# Voir les groupes d'un utilisateur
Get-ADUser marie.martin -Properties MemberOf | Select -ExpandProperty MemberOf

# Comparer avec un autre utilisateur
Get-ADUser pierre.durand -Properties MemberOf | Select -ExpandProperty MemberOf

# Voir les permissions NTFS d'un dossier (PowerShell)
Get-Acl "\\Serveur\Compta" | Format-List
```

---

### 🔴 Scénario 4 — La réplication AD est en échec

**Situation** :

- Le réseau a deux contrôleurs de domaine : DC01 et DC02
- Un admin crée un utilisateur sur DC01
- L'utilisateur n'existe pas sur DC02 30 minutes plus tard
- Des erreurs apparaissent dans l'observateur d'événements

**Questions à se poser** :

1. DC01 et DC02 peuvent-ils communiquer sur le réseau ?
2. Y a-t-il des erreurs de réplication dans l'observateur d'événements ?
3. Les ports nécessaires à la réplication AD sont-ils ouverts ?
4. Y a-t-il un problème de DNS entre les DC ?

**Analyse** :

La réplication AD utilise plusieurs ports. Un firewall mal configuré entre les DC peut bloquer la réplication silencieusement.

|Port|Protocole|Usage|
|---|---|---|
|389|TCP/UDP|LDAP|
|636|TCP|LDAP sécurisé|
|3268|TCP|Global Catalog|
|49152-65535|TCP|RPC dynamique (réplication AD)|
|88|TCP/UDP|Kerberos|
|53|TCP/UDP|DNS|

**Marche à suivre** :

1. `repadmin /showrepl` → voir l'état de la réplication entre tous les DC
2. `repadmin /replsummary` → résumé des erreurs de réplication
3. `dcdiag /test:replications` → diagnostic complet de la réplication
4. Vérifier la connectivité réseau entre DC01 et DC02
5. Forcer la réplication : `repadmin /syncall /AdeP`

**Commandes utiles** :

```powershell
# État de la réplication
repadmin /showrepl

# Résumé des erreurs
repadmin /replsummary

# Diagnostic complet du DC
dcdiag

# Forcer la réplication
repadmin /syncall /AdeP

# Tester la santé générale d'AD
dcdiag /test:dns
dcdiag /test:netlogons
```

---

### 🔴 Scénario 5 — Utilisateurs verrouillés en masse

**Situation** :

- Le helpdesk reçoit de nombreux appels : des utilisateurs sont verrouillés
- Ça touche des utilisateurs de différents services
- Les comptes se re-verrouillent peu après avoir été déverrouillés

**Questions à se poser** :

1. Y a-t-il une attaque par brute force sur le domaine ?
2. Un service ou une tâche planifiée utilise-t-il des credentials obsolètes ?
3. Un utilisateur a-t-il changé son mot de passe et des sessions anciennes tentent encore l'ancien ?
4. Quel DC enregistre les verrouillages ? (PDC Emulator en premier)

**Analyse** :

Les verrouillages en masse qui se répètent indiquent généralement une source automatique qui tente des connexions avec un mauvais mot de passe : un service Windows, une tâche planifiée, une session ouverte sur un autre PC, une application métier avec credentials stockés.

**Marche à suivre** :

1. Identifier la source des verrouillages avec **Microsoft Account Lockout Tool**
2. Sur le PDC Emulator : chercher les événements `4740` dans l'observateur d'événements
3. L'événement 4740 indique le compte verrouillé ET la machine source
4. Se connecter sur la machine source et chercher les credentials stockés
5. Mettre à jour ou supprimer les credentials obsolètes

```powershell
# Trouver les comptes verrouillés
Search-ADAccount -LockedOut | Select Name, LockedOut, LastLogonDate

# Déverrouiller un compte
Unlock-ADAccount -Identity jean.dupont

# Voir la politique de verrouillage
Get-ADDefaultDomainPasswordPolicy

# Trouver l'événement 4740 sur le PDC
Get-WinEvent -ComputerName DC01 -FilterHashtable @{LogName='Security'; Id=4740} | Select -First 20
```

> 💡 **À retenir** : Les verrouillages qui se répètent ne sont presque jamais des attaques. Dans 90% des cas, c'est un credential obsolète quelque part (service, session RDP ouverte, téléphone pro avec l'ancien mot de passe).

---

### 🔴 Scénario 6 — Un PC ne peut pas rejoindre le domaine

**Situation** :

- Un technicien tente d'ajouter un nouveau PC au domaine `entreprise.local`
- Message d'erreur : "Le nom de domaine `entreprise.local` est introuvable"
- Le PC a accès à Internet
- Le PC est sur le bon réseau local

**Questions à se poser** :

1. Le DNS du PC pointe-t-il vers le contrôleur de domaine ?
2. Le PC peut-il résoudre `entreprise.local` ?
3. Le compte utilisé pour rejoindre le domaine a-t-il les droits suffisants ?
4. Y a-t-il un quota de machines rejoignant le domaine (par défaut, 10 par utilisateur) ?

**Analyse** :

Le message "nom de domaine introuvable" est presque toujours un problème **DNS**. Pour rejoindre un domaine AD, le PC doit absolument pouvoir résoudre le nom du domaine via un DC qui fait DNS.

|Cause|Vérification|
|---|---|
|DNS pointant vers 8.8.8.8 au lieu du DC|`ipconfig /all` → voir le DNS|
|DNS interne qui ne résout pas la zone|`nslookup entreprise.local`|
|DC injoignable|`ping dc01.entreprise.local`|

**Marche à suivre** :

1. `ipconfig /all` → vérifier que le DNS pointe vers l'IP du DC (ex: `192.168.1.10`)
2. Corriger le DNS si nécessaire (DHCP ou config manuelle)
3. `nslookup entreprise.local` → doit retourner l'IP du DC
4. Rejoindre le domaine avec un compte admin du domaine
5. Placer le PC dans la bonne OU après jonction

```powershell
# Rejoindre le domaine (PowerShell)
Add-Computer -DomainName "entreprise.local" -Credential (Get-Credential) -Restart

# Vérifier la jonction au domaine
(Get-WmiObject Win32_ComputerSystem).PartOfDomain
```

> ⚠️ **Règle d'or** : Avant de rejoindre un domaine, toujours configurer le DNS pour pointer vers un DC. C'est la première chose à vérifier.

---

## 11. Exercices d'entraînement

---

**Exercice 1** — Tu dois donner accès au dossier `\\Serveur\Projets` en lecture à tous les membres du service IT, en lecture/écriture aux chefs de projet, et sans accès aux stagiaires (même s'ils sont dans le service IT).

Décris ta stratégie avec la méthode AGDLP.

<details> <summary>👁️ Voir la réponse</summary>

**Groupes globaux à créer** :

- `GG_IT` → tous les membres du service IT
- `GG_ChefsProjet` → les chefs de projet
- `GG_Stagiaires` → les stagiaires

**Groupes locaux de domaine** :

- `GL_Projets_Lecture` → reçoit permission Lecture sur `\\Serveur\Projets`
- `GL_Projets_Ecriture` → reçoit permission Lecture+Écriture sur `\\Serveur\Projets`

**Assignation** :

- `GG_IT` → `GL_Projets_Lecture`
- `GG_ChefsProjet` → `GL_Projets_Ecriture`
- `GG_Stagiaires` → aucun groupe → **refus explicite** ou simplement pas de permission

> Note : Ne pas oublier de gérer les permissions NTFS ET les permissions de partage.

</details>

---

**Exercice 2** — Un utilisateur appelle : "Ma GPO de restriction USB ne s'applique pas sur mon PC mais s'applique sur celui de mon collègue, on est dans la même OU."

Quelles sont les 4 vérifications à faire dans l'ordre ?

<details> <summary>👁️ Voir la réponse</summary>

1. `gpresult /r` sur le PC problématique → voir si la GPO apparaît dans les GPO appliquées ou refusées
2. Vérifier le **filtrage de sécurité** de la GPO → le PC (ou l'utilisateur) est-il dans le groupe filtré ?
3. Vérifier qu'il n'y a pas une **GPO de priorité plus haute** qui annule la restriction USB
4. `gpupdate /force` → forcer l'application et retester

</details>

---

**Exercice 3** — Quel rôle FSMO contacter si :

- Des utilisateurs ne peuvent pas changer leur mot de passe ?
- Tu veux ajouter un nouveau domaine à la forêt ?
- Un DC manque d'identifiants uniques (SID) pour créer de nouveaux objets ?

<details> <summary>👁️ Voir la réponse</summary>

- Problème de mot de passe → **PDC Emulator** (gère la synchronisation des mots de passe)
- Ajouter un domaine → **Domain Naming Master** (gère la structure de la forêt)
- Manque d'identifiants → **RID Master** (distribue les blocs de RID aux DC)

</details>

---

## 12. Aide-mémoire rapide

```
STRUCTURE AD
Forêt > Arbre > Domaine > OU > Objets (utilisateurs, PC, groupes)

AUTHENTIFICATION
Protocole : Kerberos
Prérequis : DNS fonctionnel (sans DNS, pas d'authentification AD)
SSO : l'utilisateur saisit son mot de passe une seule fois

GROUPES — STRATÉGIE AGDLP
A  → Account (Utilisateur)
G  → Global Group (par service/fonction)
DL → Domain Local Group (par ressource)
P  → Permission (sur la ressource)

GPO — ORDRE D'APPLICATION (LSDOU)
Local → Site → Domaine → OU (la dernière appliquée gagne)
Commande clé : gpresult /r   ou   gpresult /h rapport.html
Forcer l'application : gpupdate /force

FSMO — 5 RÔLES
Schema Master          → Forêt  → Schéma AD
Domain Naming Master   → Forêt  → Ajouter/supprimer domaines
RID Master             → Domaine → SID des objets
PDC Emulator           → Domaine → Mots de passe, heure, compatibilité
Infrastructure Master  → Domaine → Références inter-domaines

RÉPLICATION AD
repadmin /showrepl        → État de la réplication
repadmin /replsummary     → Résumé des erreurs
dcdiag                    → Diagnostic complet du DC
repadmin /syncall /AdeP   → Forcer la réplication

COMMANDES CLÉS
gpresult /r                         → GPO appliquées sur le PC courant
gpupdate /force                     → Forcer l'application des GPO
nltest /sc_verify:domaine.local     → Tester le canal sécurisé
nltest /dsgetdc:domaine.local       → Trouver le DC actif
Get-ADUser <user> -Properties *     → Infos complètes sur un utilisateur
Unlock-ADAccount -Identity <user>   → Déverrouiller un compte
Search-ADAccount -LockedOut         → Lister les comptes verrouillés
```

---

> ✅ **À retenir** : Active Directory repose sur **3 piliers** :
> 
> 1. **DNS** — sans lui, rien ne fonctionne (authentification, réplication, jonction au domaine)
> 2. **Réplication** — les modifications doivent se propager entre tous les DC
> 3. **Permissions** — toujours penser aux deux niveaux : partage ET NTFS
> 
> Quand quelque chose ne fonctionne pas en AD, commence toujours par vérifier le DNS.