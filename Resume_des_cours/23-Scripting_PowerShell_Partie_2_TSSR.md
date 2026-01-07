# Scripting PowerShell - Partie 2
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Scripting PowerShell - Partie 2 (Windows)  
**Date** : Novembre 2024  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Wildcards et regex|Wildcards et regex]]
2. [[#Les structures conditionnelles|Les structures conditionnelles]]
3. [[#Les structures itératives|Les structures itératives]]
4. [[#Les tableaux|Les tableaux]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]
7. [[#📖 Références externes|Références externes]]

---

## Wildcards et regex

> [!abstract] Vue d'ensemble
> Les **wildcards** (caractères génériques) et les **regex** (expressions régulières) permettent de manipuler et rechercher des motifs dans des chaînes de caractères. Essentiels pour le filtrage de fichiers, la recherche de patterns et la manipulation de texte.

### Les wildcards

> [!quote] Définition
> Les **wildcards** (ou caractères génériques) sont des symboles qui permettent de remplacer un ou plusieurs caractères dans une chaîne de caractères.

**Caractères génériques PowerShell** :

| Wildcard | Description | Remplace |
|----------|-------------|----------|
| **\*** | Astérisque | Zéro ou plusieurs caractères |
| **?** | Point d'interrogation | Exactement un caractère |
| **[abc]** | Ensemble de caractères | Un caractère parmi ceux spécifiés |
| **[a-z]** | Plage de caractères | Un caractère dans la plage |

---

### Exemples de wildcards

> [!success] Cas d'usage

**Astérisque \*** : Remplace **zéro ou plusieurs** caractères

```powershell
# Rechercher tous les fichiers .txt dans C:\
Get-ChildItem C:\*.txt

# Rechercher tous les fichiers commençant par "test"
Get-ChildItem C:\test*

# Rechercher tous les fichiers
Get-ChildItem C:\*
```

**Point d'interrogation ?** : Remplace **exactement un** caractère

```powershell
# Rechercher file1.txt, file2.txt, filea.txt, etc.
Get-ChildItem C:\file?.txt

# Rechercher tous les fichiers avec 4 caractères + .txt
Get-ChildItem C:\????.txt

# Combiner ? et *
Get-ChildItem C:\file?.tx*
```

**Exemples pratiques** :

```powershell
# Tous les fichiers .log et .txt
Get-ChildItem C:\*.log, C:\*.txt

# Fichiers commençant par "log" suivi d'un chiffre
Get-ChildItem C:\log?.txt

# Fichiers avec 3 caractères minimum
Get-ChildItem C:\???*
```

---

### Les regex

> [!quote] Définition
> Les **regex** (ou expressions régulières) fournissent un moyen **puissant** de manipuler des chaînes de caractères. On peut les utiliser avec des opérateurs comme **-match**, **-replace**, et des cmdlets comme **Select-String**.

**Avantages des regex** :
- ✅ Plus puissantes que les wildcards
- ✅ Recherches complexes
- ✅ Validation de formats (email, IP, téléphone...)
- ✅ Extraction de données
- ✅ Remplacement avancé

---

### Syntaxe regex de base

**Caractères spéciaux** :

| Regex | Description | Exemple |
|-------|-------------|---------|
| **\\d** | Chiffre (0-9) | `\d` → `3` |
| **\\w** | Caractère alphanumérique | `\w` → `a`, `Z`, `5` |
| **\\s** | Espace blanc | `\s` → espace, tab |
| **.** | N'importe quel caractère | `.` → `a`, `3`, `@` |
| **^** | Début de chaîne | `^H` → commence par H |
| **$** | Fin de chaîne | `t$` → finit par t |
| **\*** | 0 ou plusieurs fois | `a*` → `""`, `a`, `aaa` |
| **+** | 1 ou plusieurs fois | `a+` → `a`, `aaa` |
| **?** | 0 ou 1 fois | `a?` → `""`, `a` |
| **[abc]** | Ensemble | `[abc]` → `a`, `b` ou `c` |
| **(a\|b)** | Alternative | `(boy\|girl)` → `boy` ou `girl` |

---

### Exemples de regex

> [!success] Applications pratiques

**Exemple 1 : Rechercher des chiffres**

```powershell
"123" -match "\d"
# Résultat : True (\d correspond à tous les chiffres)

"abc" -match "\d"
# Résultat : False (pas de chiffre)
```

---

**Exemple 2 : Début de chaîne**

```powershell
"Hello" -match "^H"
# Résultat : True (^ correspond à un début de chaîne)

"hello" -match "^H"
# Résultat : False (sensible à la casse par défaut)
```

---

**Exemple 3 : Remplacement avec alternatives**

```powershell
"Hello boy" -replace "(boy|girl)", "everybody"
# Résultat : Hello everybody

"Hello girl" -replace "(boy|girl)", "everybody"
# Résultat : Hello everybody
```

---

**Exemple 4 : Validation d'email**

```powershell
$email = "test@example.com"
$email -match "^\w+@\w+\.\w+$"
# Résultat : True

$email = "invalid.email"
$email -match "^\w+@\w+\.\w+$"
# Résultat : False
```

---

**Exemple 5 : Extraction de chiffres**

```powershell
$text = "Il y a 42 réponses"
$text -match "\d+"
# Résultat : True

# Accéder à la correspondance
$matches[0]
# Résultat : 42
```

---

**Exemple 6 : Validation d'adresse IP**

```powershell
$ip = "192.168.1.100"
$ip -match "^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$"
# Résultat : True
```

---

## Les structures conditionnelles

> [!abstract] Vue d'ensemble
> Les **structures conditionnelles** permettent d'exécuter du code de manière **optionnelle**, en fonction d'une **condition** (test). Si la condition est vraie, le code est exécuté, sinon il est ignoré.

### Définition

> [!quote] Rappel du cours
> En algorithmique, on appelle **structure conditionnelle**, une construction d'un langage qui permet la création d'**instructions optionnelles**.
> 
> C'est-à-dire de portions de code dont l'exécution va dépendre d'une **condition** (on dit aussi **test**).

---

### Les tests

> [!important] Vrai ou faux ?

**En PowerShell, un test est** :
- **Vrai** s'il vaut **True**
- **Faux** s'il vaut **False**

**Code de sortie (status code)** :
- Commande **réussie** → **Vrai** (True)
- Commande **échouée** → **Faux** (False)

**Les tests se font avec des opérateurs de comparaison.**

---

### Codes de sortie

> [!info] Un exemple

**Rappel** : `$?` permet de récupérer le code de sortie de la dernière commande.

```powershell
# Créer un nouveau dossier (succès)
New-Item -Name NewDir -ItemType Directory

# Résultat :
# Mode    LastWriteTime    Length Name
# ----    -------------    ------ ----
# d----   16/06/2022 10:20        NewDir

# Vérifier le statut
$?
# Résultat : True
```

```powershell
# Essayer de créer le même dossier (échec)
New-Item -Name NewDir -ItemType Directory

# Résultat :
# New-Item : Il existe déjà un élément avec le nom spécifié C:\Lab\NewDir

# Vérifier le statut
$?
# Résultat : False
```

---

### Les opérateurs de comparaison

> [!important] Opérateurs disponibles

**Égalité** :
- **-eq** : Equal to (égal à)
- **-ne** : Not equal to (différent de)

**Comparaison numérique** :
- **-gt** : Greater than (supérieur à)
- **-lt** : Less than (inférieur à)
- **-ge** : Greater than or equal to (supérieur ou égal)
- **-le** : Less than or equal to (inférieur ou égal)

**Correspondance de motifs** :
- **-like** : Correspond (avec wildcards)
- **-notLike** : Ne correspond pas (avec wildcards)
- **-match** : Correspond (avec regex)
- **-notMatch** : Ne correspond pas (avec regex)

**Négation** :
- **-not** ou **!** : Inverse le code de sortie (NON logique)

---

### Comparaison de chaînes

> [!success] Un exemple

**Supposons 2 chaînes s1 et s2** :

| Expression | Description | Résultat |
|------------|-------------|----------|
| `s1 -eq s2` | Vrai si les chaînes sont identiques | True/False |
| `s1 -ne s2` | Vrai si les chaînes sont différentes | True/False |
| `[String]::IsNullOrEmpty(s1)` | Vrai si s1 est vide | True/False |
| `![String]::IsNullOrEmpty(s1)` | Vrai si s1 n'est pas vide | True/False |

> [!warning] Attention aux espaces !
> Les espaces sont pris en compte dans les comparaisons.

**Exemples** :

```powershell
'identique' -eq 'identique'
# Résultat : True

'identique' -eq 'différent'
# Résultat : False

'identique' -ne 'différent'
# Résultat : True

[String]::IsNullOrEmpty("")
# Résultat : True

"" -eq $Null
# Résultat : False
```

---

### Comparaison de nombres

> [!success] Un exemple

**Supposons 2 nombres n1 et n2** :

| Expression | Description | Équivalent mathématique |
|------------|-------------|-------------------------|
| `n1 -eq n2` | Égal | n1 = n2 |
| `n1 -ne n2` | Différent | n1 ≠ n2 |
| `n1 -lt n2` | Inférieur strict | n1 < n2 |
| `n1 -le n2` | Inférieur ou égal | n1 ≤ n2 |
| `n1 -gt n2` | Supérieur strict | n1 > n2 |
| `n1 -ge n2` | Supérieur ou égal | n1 ≥ n2 |

**Exemples** :

```powershell
$trois = 3
$trois -eq 3
# Résultat : True

2 -ne $trois
# Résultat : True

$deux = 2
$deux -lt $trois
# Résultat : True
```

---

### Opérateurs logiques booléens

> [!important] Un exemple

**Supposons c1 et c2 des conditions** :

| Opérateur | Description | Vrai si... |
|-----------|-------------|------------|
| **! c1** | NON logique | c1 est faux |
| **c1 -and c2** | ET logique | c1 ET c2 sont vrais |
| **c1 -or c2** | OU logique | c1 OU c2 (ou les deux) est vrai |
| **c1 -xor c2** | OU exclusif | Uniquement l'un des deux est vrai |

**Exemples** :

```powershell
$trois = 3

! $trois -eq 3
# Résultat : False (inverse de True)

2 -lt $trois -and $trois -lt 4
# Résultat : True (2 < 3 ET 3 < 4)

$trois -eq 3 -or $trois -eq 4
# Résultat : True (3 = 3 OU 3 = 4 → premier vrai)

$trois -eq 3 -or $trois -lt 4
# Résultat : True (les deux conditions vraies)

$trois -eq 3 -xor $trois -lt 4
# Résultat : False (les deux conditions vraies, pas exclusif)
```

---

### Opérateurs sur les chemins

> [!info] Un exemple

**Test d'existence** :

**Supposons p un chemin/fichier/dossier** :

| Cmdlet | Description |
|--------|-------------|
| **Test-Path p** | Vrai si p existe |

**Exemple** :

```powershell
Test-Path -Path C:\Windows
# Résultat : True

Test-Path -Path C:\DossierInexistant
# Résultat : False
```

**Autres utilisations** :

```powershell
# Tester un fichier
Test-Path -Path C:\Windows\System32\notepad.exe
# Résultat : True

# Tester avec variable
$CheminFichier = "C:\temp\test.txt"
if (Test-Path -Path $CheminFichier) {
    Write-Host "Le fichier existe"
} else {
    Write-Host "Le fichier n'existe pas"
}
```

---

### Si … Sinon

> [!success] Et si ?

**Structure conditionnelle If** :

```powershell
If (condition)
{
    # Instructions si condition vraie
}
Else
{
    # Instructions si condition fausse
}
```

**Exemple pratique** :

```powershell
If (New-Item -ItemType Directory -Name NewDir -ErrorAction SilentlyContinue)
{
    Write-Host "Création dossier succès"
}
else
{
    Write-Host "Création dossier échec" -ForegroundColor Red
}
```

---

#### Structure If complète

**Avec ElseIf** :

```powershell
If (condition1)
{
    # Instructions si condition1 vraie
}
ElseIf (condition2)
{
    # Instructions si condition2 vraie
}
Else
{
    # Instructions si aucune condition vraie
}
```

**Exemple** :

```powershell
$Note = 15

If ($Note -ge 16)
{
    Write-Host "Très bien" -ForegroundColor Green
}
ElseIf ($Note -ge 14)
{
    Write-Host "Bien" -ForegroundColor Cyan
}
ElseIf ($Note -ge 10)
{
    Write-Host "Passable" -ForegroundColor Yellow
}
Else
{
    Write-Host "Insuffisant" -ForegroundColor Red
}
```

---

### Switch

> [!important] Structure Switch

**Switch** : Alternative à des If/ElseIf multiples.

**Syntaxe** :

```powershell
Switch (condition)
{
    valeur1 {ScriptBlock1}
    valeur2 {ScriptBlock2}
    ...
    default {ScriptBlock par défaut}
}
```

**Exemple simple** :

```powershell
$Condition = 5

Switch ($Condition)
{
    1 {Write-Host "hello"}
    2 {Write-Host "2"}
    5 {Write-Host "5"}
    default {Write-Host "default"}
}

# Résultat : 5
```

---

#### Switch avancé

**Avec plusieurs valeurs** :

```powershell
$Jour = "Lundi"

Switch ($Jour)
{
    "Lundi" {Write-Host "Début de semaine" -ForegroundColor Cyan}
    "Mardi" {Write-Host "2ème jour" -ForegroundColor Cyan}
    "Mercredi" {Write-Host "Milieu de semaine" -ForegroundColor Yellow}
    "Jeudi" {Write-Host "4ème jour" -ForegroundColor Cyan}
    "Vendredi" {Write-Host "Fin de semaine" -ForegroundColor Green}
    {$_ -in "Samedi","Dimanche"} {Write-Host "Weekend !" -ForegroundColor Magenta}
    default {Write-Host "Jour inconnu" -ForegroundColor Red}
}
```

**Avec conditions** :

```powershell
$Nombre = 42

Switch ($Nombre)
{
    {$_ -lt 0} {Write-Host "Négatif"}
    {$_ -eq 0} {Write-Host "Zéro"}
    {$_ -gt 0 -and $_ -lt 10} {Write-Host "Entre 1 et 9"}
    {$_ -ge 10} {Write-Host "Supérieur ou égal à 10"}
}
```

---

## Les structures itératives

> [!abstract] Vue d'ensemble
> Les **structures itératives** (boucles) permettent de **répéter** l'exécution d'instructions un nombre de fois donné ou tant qu'une condition est remplie.

### Définition

> [!quote] Rappel du cours
> En algorithmique, on appelle **structure itérative**, une construction d'un langage qui permet la **répétition d'instructions**.
> 
> C'est-à-dire de portions de code dont l'exécution va être effectuée un **nombre de fois donné** ou **tant qu'une condition est remplie**.
> 
> Il est courant de les qualifier de **boucles**.

---

### La notion d'unaire

> [!important] Opérateurs d'incrémentation

**Les opérateurs unaires** sont souvent utilisés pour **incrémenter** des variables dans les boucles.

**Opérateurs** :
- **++** : Incrémenter de 1
- **--** : Décrémenter de 1

**Exemple** :

```powershell
$i
# Résultat : (vide)

$i = $i + 1
$i
# Résultat : 1

$i++
$i
# Résultat : 2

$i--
$i
# Résultat : 1
```

---

### Sens d'incrémentation unaire

> [!warning] Le sens a son importance

**Position de l'opérateur** :

| Syntaxe | Comportement | Résultat |
|---------|--------------|----------|
| **$i++** | Afficher puis incrémenter | Valeur avant incrémentation |
| **++$i** | Incrémenter puis afficher | Valeur après incrémentation |

**Exemple pratique** :

```powershell
Clear-Host
$i = 1

"`$i vaut $(($i++))"
"`$i maintenant vaut $i`n"

$j = 1

"`$j vaut $((++$j))"
"`$j maintenant vaut $j"
```

**Résultat** :
```
$i vaut 1
$i maintenant vaut 2

$j vaut 2
$j maintenant vaut 2
```

**Explication** :
- **$i++** : $i est affiché (1), **puis** incrémenté (2)
- **++$j** : $j est incrémenté (2), **puis** affiché (2)

---

### Boucle For

> [!success] Boucler sur une liste

**La boucle for** est une boucle que l'on **initialise** et qui a une **fin définie**.

**Syntaxe** :

```powershell
For (initialisation; condition; mise-à-jour)
{
    # Bloc d'instructions
}
```

**Exemple simple** :

```powershell
For ($i = 0; $i -le 10; $i++)
{
   Write-Host "Valeur: $i"
}
```

**Résultat** :
```
Valeur: 0
Valeur: 1
Valeur: 2
...
Valeur: 10
```

---

#### Boucle For à conditions multiples

> [!info] Conditions multiples

**On peut avoir plusieurs variables et conditions** :

```powershell
For (($i=0), ($j=0); $i -le 20 -and ($i+$j) -le 15; $i++, $j++)
{
   Write-Host "Valeur de `$i: $i`nValeur de `$j: $j`nValeur de `$i+`$j: $($i+$j)`n"
}
```

**Explication** :
- **Initialisation** : `$i = 0` ET `$j = 0`
- **Condition** : `$i ≤ 20` ET `($i + $j) ≤ 15`
- **Mise à jour** : `$i++` ET `$j++`

---

### Boucle Foreach

> [!success] Itérer sur une collection

**La boucle foreach** est utilisée pour la manipulation de **collections de données** ou **tableaux**. Elle va lire chaque élément à chaque itération.

**Syntaxe** :

```powershell
Foreach (element in collection)
{
    # Bloc d'instructions
}
```

**Exemple avec services** :

```powershell
$Services = Get-Service

foreach ($Service in $Services)
{
   Write-Host "$($Service.Name) --> $($Service.Status)"
}
```

**Résultat** :
```
AdobeARMservice --> Running
Appinfo --> Stopped
...
```

---

#### Foreach derrière un pipe

> [!info] ForEach-Object

**Foreach peut être utilisé dans un pipeline** :

```powershell
Get-Service | ForEach {Write-Host "$($_.Name) --> $($_.Status)"}
```

> [!note] Alias
> - **Foreach** est l'alias de **ForEach-Object**
> - On peut également le remplacer par **%**

**Équivalent** :

```powershell
# Forme complète
Get-Service | ForEach-Object {Write-Host "$($_.Name) --> $($_.Status)"}

# Alias Foreach
Get-Service | Foreach {Write-Host "$($_.Name) --> $($_.Status)"}

# Alias %
Get-Service | % {Write-Host "$($_.Name) --> $($_.Status)"}
```

---

#### Boucle Foreach avec Switch

> [!success] Exemple avancé

**Combiner Foreach et Switch** :

```powershell
$Services = Get-Service
$Count = 1

Foreach ($Service in $Services)
{
   Switch ($Service.Status)
   {
       "Stopped" {
           Write-Host "$Count - Service: $($Service.Name) ($($Service.DisplayName)) --> $($Service.Status)" -ForegroundColor Red
       }
       "Running" {
           Write-Host "$Count - Service: $($Service.Name) ($($Service.DisplayName)) --> $($Service.Status)" -ForegroundColor Green
       }
       default {
           Write-Host "$Count - Service: $($Service.Name) ($($Service.DisplayName)) --> $($Service.Status)" -ForegroundColor Blue
       }
   }
   $Count++
}
```

**Résultat** : Liste numérotée des services avec couleurs selon le statut.

---

### Boucle While

> [!success] Tant que...

**La boucle while** exécute le bloc d'instructions **tant que** la condition est vérifiée.

**Syntaxe** :

```powershell
While (condition)
{
    # Bloc d'instructions
}
```

**Exemple** :

```powershell
$Count = 0

While ($Count -le 10)
{
   Write-Host "Compteur égal à $Count"
   $Count++
}
```

**Résultat** :
```
Compteur égal à 0
Compteur égal à 1
...
Compteur égal à 10
```

> [!warning] Attention
> Si la condition n'est jamais fausse, c'est une **boucle infinie** !

---

### Boucle Do While

> [!info] Faire... Tant que

**La boucle do while** est comme la boucle while, sauf que la condition est vérifiée **à la fin**, donc il y a **au moins 1 passage** dans la boucle.

**Syntaxe** :

```powershell
Do
{
    # Bloc d'instructions
}
While (condition)
```

**Exemple** :

```powershell
$Count = 0

do
{
   Write-Host "Compteur égal à $Count"
   $Count++
}
While ($Count -le 10)
```

**Différence avec While** :
- **While** : Condition vérifiée **avant** → 0 ou plusieurs passages
- **Do While** : Condition vérifiée **après** → **Au moins 1 passage**

---

### Boucle Do Until

> [!info] Faire... Jusqu'à

**La boucle do until** exécute le bloc de script **jusqu'à ce que** la condition soit réalisée.

**Syntaxe** :

```powershell
Do
{
    # Bloc d'instructions
}
Until (condition)
```

**Exemple** :

```powershell
$Count = 0

do
{
   Write-Host "Compteur égal à $Count"
   $Count++
}
Until ($Count -eq 10)
```

**Différence avec Do While** :
- **Do While** : Continue **tant que** condition **vraie**
- **Do Until** : Continue **jusqu'à ce que** condition **vraie** (inverse)

---

### Comparaison des boucles

> [!note] Récapitulatif

| Boucle | Condition | Passage minimum | Usage principal |
|--------|-----------|-----------------|-----------------|
| **For** | Compteur + condition | 0 | Nombre d'itérations connu |
| **Foreach** | Collection | 0 | Itérer sur tableau/collection |
| **While** | Avant le bloc | 0 | Tant que condition vraie |
| **Do While** | Après le bloc | 1 | Au moins 1 passage garanti |
| **Do Until** | Après le bloc | 1 | Jusqu'à condition vraie |

---

## Les tableaux

> [!abstract] Vue d'ensemble
> Les **tableaux** (ou collections) sont des structures de données qui contiennent **plusieurs éléments**. Essentiels pour stocker et manipuler des ensembles de valeurs.

### Définition

> [!quote] Qu'est-ce qu'un tableau ?

**En programmation** :
- Les **tableaux** (ou collections) sont des structures de données qui contiennent **plusieurs éléments**
- Le tableau est créé sous la forme d'un **bloc séquentiel de mémoire**
- Chaque valeur est stockée juste à côté de l'autre

**Pour accéder aux éléments, 3 méthodes** :
1. Avec un **index** (position)
2. Avec une **boucle** (itération)
3. Avec une **clé** (hashtable)

---

### Initialisation de tableau

> [!important] Init

**L'initialisation d'une variable en type tableau** change sa structure de données.

**Syntaxes** :

```powershell
# Tableau vide
$Tab = @()

# Tableau avec valeurs
$Tab = @(valeur1, valeur2, ...)

# Syntaxe courte
$Tab = valeur1, valeur2, ...

# Plage de valeurs
$Tab = ValeurInit..ValeurFinale
```

**Exemples** :

```powershell
$Tab
# Résultat : (vide)

$Tab -eq $Null
# Résultat : True

# Initialiser tableau vide
$Tab = @()

$Tab -eq $Null
# Résultat : (vide, mais False - c'est un tableau)

# Vérifier le type
$Tab.GetType()
# Résultat :
# IsPublic IsSerial Name      BaseType
# -------- -------- ----      --------
# True     True     Object[]  System.Array

# Vérifier si vide
$Tab.Count -gt 0
# Résultat : False

# Plage de nombres
1..5
# Résultat :
# 1
# 2
# 3
# 4
# 5
```

---

### Mon premier tableau

> [!success] Le tout premier

**Créer un tableau de jours** :

```powershell
$Tab = @("Lundi","Mardi","Mercredi","Jeudi","Vendredi","Samedi","Dimanche")
```

**Propriété `.count`** : Nombre d'éléments

```powershell
$Tab.Count
# Résultat : 7

# Afficher tout le tableau
$Tab
# Résultat :
# Lundi
# Mardi
# Mercredi
# Jeudi
# Vendredi
# Samedi
# Dimanche
```

---

### Tableaux : recherche par index

> [!important] Par indice

**Pour un tableau $Tab** :
- **$Tab[n]** : n-ième élément
- ⚠️ **n commence à 0** (premier élément = index 0)

**Exemples** :

```powershell
$Tab = @("Lundi","Mardi","Mercredi","Jeudi","Vendredi","Samedi","Dimanche")

# Premier élément (index 0)
$Tab[0]
# Résultat : Lundi

# Plusieurs éléments
$Tab[1,3,5]
# Résultat :
# Mardi
# Jeudi
# Samedi

# Dernier élément (index -1)
$Tab[-1]
# Résultat : Dimanche

# Ajouter un élément
$Tab += "JourEnPlus"

$Tab[-1]
# Résultat : JourEnPlus
```

**Index négatifs** :

| Index | Élément |
|-------|---------|
| -1 | Dernier |
| -2 | Avant-dernier |
| -3 | Troisième en partant de la fin |

---

### Tableaux : recherche par boucle

> [!success] On mixe boucle et tableau

**La boucle Foreach** est souvent utilisée dans l'exploitation des tableaux.

```powershell
$Tab = @("Lundi","Mardi","Mercredi","Jeudi","Vendredi","Samedi","Dimanche")
$Count = 1

Foreach ($Date in $Tab)
{
   Write-Host "Jour N°$Count de la semaine : $Date"
   $Count++
}
```

**Résultat** :
```
Jour N°1 de la semaine : Lundi
Jour N°2 de la semaine : Mardi
...
Jour N°7 de la semaine : Dimanche
```

**Autre exemple avec For** :

```powershell
For ($i = 0; $i -lt $Tab.Count; $i++)
{
    Write-Host "Jour N°$($i+1) : $($Tab[$i])"
}
```

---

### Tableaux : recherche par clé

> [!important] Les hashtables

**Les tableaux avec des clés** sont appelés **table de hachage** (ou **dictionnaire** ou **tableau associatif**).

**Ce sont des structures de données qui stockent** une ou plusieurs **paires clé/valeur**.

**Syntaxe** :

```powershell
@{clé1=valeur1; clé2=valeur2; ...}
```

**Exemple** :

```powershell
$HashTable = @{
    1 = "Lundi"
    2 = "Mardi"
    3 = "Mercredi"
    4 = "Jeudi"
    5 = "Vendredi"
    6 = "Samedi"
    7 = "Dimanche"
}
```

---

#### Manipulation de hashtables

**Propriétés et méthodes** :

```powershell
# Nombre d'éléments
$HashTable.Count
# Résultat : 7

# Accéder à un élément par clé
$HashTable[2]
# Résultat : Mardi

# Lister les clés
$HashTable.Keys
# Résultat : 1, 2, 3, 4, 5, 6, 7

# Lister les valeurs
$HashTable.Values
# Résultat : Lundi, Mardi, Mercredi, ...

# Ajouter un élément
$HashTable.Add("8", "Jour d'après")

# Énumérer les paires clé/valeur
$HashTable.GetEnumerator()
# Résultat : Affiche toutes les paires

# Supprimer un élément
$HashTable.Remove("8")
```

---

#### Exemples pratiques de hashtables

**Configuration serveur** :

```powershell
$Config = @{
    ServerName = "SRV01"
    IP = "192.168.1.100"
    Port = 80
    SSL = $true
}

# Accès
Write-Host "Serveur : $($Config.ServerName) sur $($Config.IP):$($Config.Port)"
```

**Informations utilisateur** :

```powershell
$User = @{
    Nom = "Dupont"
    Prenom = "Jean"
    Age = 30
    Ville = "Paris"
}

# Modification
$User.Age = 31
$User["Email"] = "jean.dupont@example.com"
```

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Wildcards et regex

**Wildcards** :
- **\*** : Zéro ou plusieurs caractères
- **?** : Exactement un caractère
- Usage : `Get-ChildItem C:\*.txt`, `Get-ChildItem file?.txt`

**Regex** :
- Plus puissantes que wildcards
- Opérateurs : `-match`, `-replace`, `Select-String`
- Caractères : `\d` (chiffre), `\w` (alphanum), `\s` (espace), `.` (n'importe quoi)
- Ancres : `^` (début), `$` (fin)
- Quantificateurs : `*` (0+), `+` (1+), `?` (0 ou 1)
- Groupes : `(boy|girl)` (alternative)

**Exemples** :
```powershell
"123" -match "\d"                              # True
"Hello" -match "^H"                            # True
"Hello boy" -replace "(boy|girl)", "everybody" # Hello everybody
```

---

### Structures conditionnelles

**Tests** :
- Vrai = True, Faux = False
- `$?` : Code de sortie dernière commande

**Opérateurs de comparaison** :

| Catégorie | Opérateurs |
|-----------|------------|
| **Égalité** | `-eq`, `-ne` |
| **Numérique** | `-gt`, `-lt`, `-ge`, `-le` |
| **Motifs** | `-like`, `-notLike`, `-match`, `-notMatch` |
| **Logiques** | `-and`, `-or`, `-xor`, `-not` (ou `!`) |
| **Chemins** | `Test-Path` |

**Structures** :

**If/ElseIf/Else** :
```powershell
If (condition1) {
    # Instructions
}
ElseIf (condition2) {
    # Instructions
}
Else {
    # Instructions
}
```

**Switch** :
```powershell
Switch (valeur) {
    valeur1 {action1}
    valeur2 {action2}
    default {action_defaut}
}
```

---

### Structures itératives

**Opérateurs unaires** :
- `$i++` : Incrémenter (après)
- `++$i` : Incrémenter (avant)
- `$i--` : Décrémenter (après)
- `--$i` : Décrémenter (avant)

**Boucles disponibles** :

**For** (nombre d'itérations connu) :
```powershell
For ($i=0; $i -le 10; $i++) {
    # Instructions
}
```

**Foreach** (collections) :
```powershell
Foreach ($item in $collection) {
    # Instructions
}

# Pipeline
$collection | ForEach {action}
$collection | % {action}
```

**While** (tant que) :
```powershell
While (condition) {
    # Instructions
}
```

**Do While** (au moins 1 passage) :
```powershell
Do {
    # Instructions
} While (condition)
```

**Do Until** (jusqu'à) :
```powershell
Do {
    # Instructions
} Until (condition)
```

---

### Tableaux

**Initialisation** :
```powershell
$Tab = @()                          # Vide
$Tab = @("val1", "val2")           # Avec valeurs
$Tab = "val1", "val2", "val3"      # Syntaxe courte
$Tab = 1..10                       # Plage
```

**Propriétés** :
- `.Count` : Nombre d'éléments

**Accès par index** :
```powershell
$Tab[0]      # Premier
$Tab[-1]     # Dernier
$Tab[1,3,5]  # Plusieurs
```

**Accès par boucle** :
```powershell
Foreach ($item in $Tab) {
    # Traiter $item
}
```

**Hashtables** (tableaux associatifs) :
```powershell
$Hash = @{
    cle1 = "valeur1"
    cle2 = "valeur2"
}

$Hash.Keys        # Clés
$Hash.Values      # Valeurs
$Hash[cle1]       # Accès
$Hash.Add()       # Ajouter
$Hash.Remove()    # Supprimer
$Hash.GetEnumerator()  # Énumérer
```

---

### Bonnes pratiques

**Wildcards et regex** :
- ✅ Utiliser wildcards pour recherches simples
- ✅ Utiliser regex pour recherches complexes
- ✅ Tester les regex avec `-match` avant `-replace`

**Conditions** :
- ✅ Utiliser `Test-Path` pour vérifier existence
- ✅ Préférer Switch à plusieurs If/ElseIf
- ✅ Grouper les conditions logiques avec parenthèses
- ✅ Utiliser `-ErrorAction SilentlyContinue` pour éviter erreurs dans conditions

**Boucles** :
- ✅ For : Nombre d'itérations connu
- ✅ Foreach : Itérer sur collections
- ✅ While : Condition d'arrêt
- ✅ Toujours prévoir une sortie (éviter boucles infinies)
- ✅ Incrémenter/décrémenter correctement

**Tableaux** :
- ✅ Initialiser avec `@()` pour tableau vide
- ✅ Utiliser `.Count` pour connaître la taille
- ✅ Index commence à 0
- ✅ Index négatifs pour partir de la fin
- ✅ Hashtables pour données structurées (clé/valeur)

---

### Pièges à éviter

> [!warning] Erreurs courantes

**Wildcards et regex** :
1. ❌ Confondre wildcards (`*`, `?`) et regex (`.*`, `.`)
2. ❌ Oublier d'échapper les caractères spéciaux en regex
3. ❌ Utiliser `-like` au lieu de `-match` pour regex

**Conditions** :
1. ❌ Confondre `-eq` (égalité) et `=` (assignation)
2. ❌ Oublier les espaces dans comparaisons de chaînes
3. ❌ Utiliser `$var -eq $Null` au lieu de `[String]::IsNullOrEmpty($var)`
4. ❌ Ne pas tester l'existence avec `Test-Path` avant accès fichier

**Boucles** :
1. ❌ Oublier d'incrémenter le compteur (boucle infinie)
2. ❌ Confondre `$i++` et `++$i`
3. ❌ Modifier une collection pendant l'itération
4. ❌ Utiliser While sans condition de sortie

**Tableaux** :
1. ❌ Oublier que l'index commence à 0
2. ❌ Accéder à un index hors limites
3. ❌ Confondre tableaux et hashtables
4. ❌ Ne pas initialiser avec `@()` (erreurs si vide)
5. ❌ Oublier `.Count` pour vérifier la taille

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Wildcard** | Caractère générique (* ou ?) pour remplacer caractères |
| **Regex** | Expression régulière, motif de recherche puissant |
| **Structure conditionnelle** | Construction permettant instructions optionnelles (If, Switch) |
| **Test** | Condition évaluée à True ou False |
| **Opérateur de comparaison** | Opérateur pour comparer valeurs (-eq, -lt, -match...) |
| **-eq** | Equal to, égal à |
| **-ne** | Not equal to, différent de |
| **-gt** | Greater than, supérieur à |
| **-lt** | Less than, inférieur à |
| **-ge** | Greater or equal, supérieur ou égal |
| **-le** | Less or equal, inférieur ou égal |
| **-like** | Correspond (avec wildcards) |
| **-match** | Correspond (avec regex) |
| **-and** | ET logique |
| **-or** | OU logique |
| **-xor** | OU exclusif |
| **-not** ou **!** | NON logique, négation |
| **If** | Structure conditionnelle si...sinon |
| **ElseIf** | Sinon si |
| **Else** | Sinon |
| **Switch** | Structure conditionnelle à choix multiples |
| **Structure itérative** | Boucle, construction pour répéter instructions |
| **Opérateur unaire** | ++, -- pour incrémenter/décrémenter |
| **For** | Boucle avec compteur |
| **Foreach** | Boucle sur collection |
| **While** | Boucle tant que condition vraie |
| **Do While** | Boucle faire...tant que (au moins 1 passage) |
| **Do Until** | Boucle faire...jusqu'à |
| **Tableau** | Collection d'éléments (array) |
| **Index** | Position d'un élément dans tableau (commence à 0) |
| **Hashtable** | Tableau associatif (paires clé/valeur) |
| **Dictionnaire** | Synonyme de hashtable |
| **.Count** | Propriété, nombre d'éléments |
| **.Keys** | Propriété hashtable, liste des clés |
| **.Values** | Propriété hashtable, liste des valeurs |
| **GetEnumerator()** | Méthode pour énumérer hashtable |
| **$?** | Variable, statut dernière commande |
| **$_** | Variable, objet actuel dans pipeline |
| **Test-Path** | Cmdlet pour tester existence chemin/fichier |
| **Pipeline** | Passage d'objets entre cmdlets avec \| |

---

## 📖 Références externes

> [!note] Documentation officielle Microsoft
> Ressources pour approfondir PowerShell Partie 2.

| Thème | Ressource | Description | Lien |
|-------|-----------|-------------|------|
| **Opérateurs de comparaison** | About Comparison Operators | Documentation complète opérateurs | [learn.microsoft.com/powershell/about_comparison_operators](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators) |
| **Expressions régulières** | About Regular Expressions | Guide complet regex PowerShell | [learn.microsoft.com/powershell/about_regular_expressions](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_regular_expressions) |
| **If, ElseIf, Else** | About If | Documentation structures conditionnelles | [learn.microsoft.com/powershell/about_if](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_if) |
| **Switch** | About Switch | Documentation structure Switch | [learn.microsoft.com/powershell/about_switch](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_switch) |
| **Boucles** | About Do, While, For | Documentation structures itératives | [learn.microsoft.com/powershell/about_do](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_do) |
| **Foreach** | About Foreach | Documentation boucle Foreach | [learn.microsoft.com/powershell/about_foreach](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_foreach) |
| **Tableaux** | About Arrays | Documentation complète tableaux | [learn.microsoft.com/powershell/about_arrays](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays) |
| **Hashtables** | About Hash Tables | Documentation hashtables | [learn.microsoft.com/powershell/about_hash_tables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables) |
| **Opérateurs** | About Operators | Tous les opérateurs PowerShell | [learn.microsoft.com/powershell/about_operators](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators) |

> [!tip] Comment utiliser ces ressources
> Ces liens te permettront de :
> - **Maîtriser** les opérateurs de comparaison et logiques
> - **Approfondir** les expressions régulières
> - **Comprendre** les structures conditionnelles (If, Switch)
> - **Pratiquer** toutes les boucles (For, Foreach, While, Do)
> - **Manipuler** les tableaux et hashtables
> - Consulter la **syntaxe complète** de chaque construction

---

### Ressources complémentaires recommandées

> [!info] Pour aller plus loin

| Thème | Ressource | Description | Lien |
|-------|-----------|-------------|------|
| **Regex101** | Testeur regex en ligne | Tester et déboguer expressions régulières | [regex101.com](https://regex101.com) |
| **SS64 PowerShell** | Référence rapide | Syntaxe et exemples | [ss64.com/ps](https://ss64.com/ps/) |
| **PowerShell Gallery** | Scripts et modules | Exemples pratiques | [powershellgallery.com](https://www.powershellgallery.com) |
| **GitHub PowerShell** | Code source | Apprendre des exemples | [github.com/PowerShell](https://github.com/PowerShell/PowerShell) |

---

> [!success] Document de révision complet
> Ce document couvre l'intégralité de **PowerShell Partie 2**. Tu as maintenant tous les éléments pour :
> - Utiliser les **wildcards** et **regex** pour recherches avancées
> - Maîtriser les **structures conditionnelles** (If, ElseIf, Else, Switch)
> - Créer toutes les **boucles** (For, Foreach, While, Do While, Do Until)
> - Manipuler les **tableaux** (arrays) et **hashtables**
> - Appliquer les **opérateurs de comparaison** et **logiques**
> - Créer des **scripts PowerShell complets** et fonctionnels
> - Combiner conditions, boucles et tableaux pour automatiser des tâches complexes
> 
> **Combiné avec PowerShell Partie 1**, tu as maintenant une base solide pour :
> - Créer des scripts d'automatisation professionnels
> - Gérer des systèmes Windows
> - Préparer ton titre RNCP TSSR
> 
> **Bon courage pour la préparation de ton titre RNCP TSSR !** 🎓💻✨

---

**Fin du document de révision**
