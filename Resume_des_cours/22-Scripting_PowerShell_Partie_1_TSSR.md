# Scripting PowerShell - Partie 1
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Scripting PowerShell - Partie 1 (Windows)  
**Date** : Novembre 2024  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Définition|Définition]]
2. [[#La base|La base]]
3. [[#Les variables - Utilisation standard|Les variables - Utilisation standard]]
4. [[#Les variables - Utilisation avancée|Les variables - Utilisation avancée]]
5. [[#Points clés à retenir|Points clés à retenir]]
6. [[#Glossaire technique|Glossaire technique]]
7. [[#📖 Références externes|Références externes]]

---

## Définition

> [!abstract] Vue d'ensemble
> **PowerShell** est un interpréteur de commandes et un langage de script développé par Microsoft pour l'**automatisation de tâches** et la **gestion de configuration** des systèmes Windows. À la différence des shells classiques (Bash, CMD), PowerShell manipule des **objets .NET** et non du texte.

### Script vs Programme

> [!quote] Quelle est la différence ?
> La différence théorique entre les deux réside dans le fait que les **langages de script ne nécessitent pas l'étape de compilation** et sont plutôt **interprétés**. Par exemple, un programme C doit être compilé avant d'être exécuté, alors qu'un langage de script tel que JavaScript ou PHP n'a pas besoin d'être compilé.

| Critère | Programme compilé | Script interprété |
|---------|-------------------|-------------------|
| **Compilation** | Oui (avant exécution) | Non (interprété à la volée) |
| **Exemples** | C, C++, Java | PowerShell, Python, Bash, JavaScript |
| **Exécution** | Fichier binaire | Fichier texte + interpréteur |
| **Vitesse** | Très rapide | Plus lent (mais suffisant) |
| **Modification** | Recompilation nécessaire | Édition directe du fichier |

---

### Le PowerShell

> [!important] Ecrire un script

**Caractéristiques de PowerShell** :

| Élément | Description |
|---------|-------------|
| **Fichiers** | Extension **.PS1** (texte) |
| **Interpréteur** | **PowerShell.exe** |
| **Terminologie** | "Console" au lieu de "shell" |
| **Particularité** | Accepte et retourne des **objets .NET** (pas du texte) |

> [!success] Avantage majeur
> À la différence des autres interpréteurs de commandes qui n'acceptent et ne retournent que du texte, PowerShell accepte et retourne des **objets .NET**.

---

### Objets .NET

> [!info] Le PowerShell - objet .NET

**.NET** :
- **Framework** développé par Microsoft
- Fournit un environnement pour construire et exécuter des applications
- Les données sont représentées sous forme d'**objets**

**Objet** :
- **Instance d'une classe**
- Combine :
  - **Propriétés** (données, attributs)
  - **Méthodes** (actions, fonctions)

**Exemple concret** :
```powershell
# Récupérer un fichier
$file = Get-ChildItem fichier.txt

# Accéder aux propriétés (attributs)
$file.Name          # Nom du fichier
$file.Length        # Taille en octets
$file.CreationTime  # Date de création

# Appeler des méthodes (actions)
$file.CopyTo("copie.txt")  # Copier le fichier
$file.Delete()             # Supprimer le fichier
```

---

### Pour quoi faire ?

> [!success] Objectifs de PowerShell

**PowerShell sert à** :
- **Automatisation de tâches** (ne pas répéter les mêmes commandes)
- **Gestion de configuration** de systèmes Microsoft

**Au quotidien** :
- ✅ Ne pas répéter les mêmes lignes de commandes
- ✅ **Gagner du temps**
- ✅ Décomposer des tâches **complexes** en tâches **simples**
- ✅ Administrer des serveurs à distance
- ✅ Automatiser des déploiements
- ✅ Gérer Active Directory, Exchange, Azure...

---

### Comment ?

> [!info] Les outils nécessaires

**PowerShell = 2 composants** :

1. **Un shell en ligne de commandes** (la console)
2. **Un langage de script** associé (un script)

**En résumé, il faut** :
- ✅ Avoir une **connaissance du langage** de script
- ✅ Avoir les **logiciels adaptés** :
  - **PowerShell ISE** (Integrated Scripting Environment) - Inclus Windows
  - **Visual Studio Code** (avec extension PowerShell)
  - Notepad++, etc.
- ✅ Avoir les **droits d'accès** pour l'écriture et l'exécution de scripts

---

### Mon premier script

> [!success] Hello World !

**Dans une console PowerShell** :

```powershell
Write-Host "Hello World !"
```

**Résultat** :
```
PS C:\Lab> Write-Host "Hello World !"
Hello World !
```

**Créer un fichier script** :
```powershell
# Créer un fichier hello.ps1
"Write-Host 'Hello World !'" | Out-File hello.ps1

# Exécuter le script
.\hello.ps1
```

---

### Bonnes pratiques

> [!tip] Recommandations

**Conventions de nommage** :
- ✅ Les noms des scripts se terminent par **.PS1**
- ✅ Utiliser le symbole **#** pour mettre des commentaires
- ✅ **Nommage clair** des variables (PascalCase recommandé)

**Exemple de script bien commenté** :
```powershell
# Script de sauvegarde automatique
# Auteur : Franck
# Date : 2024-11-01

# Définir le répertoire source
$Source = "C:\Data"

# Définir le répertoire de destination
$Destination = "D:\Backup"

# Copier les fichiers
Copy-Item -Path $Source -Destination $Destination -Recurse
```

---

## La base

### Introduction aux Classes (POO)

> [!important] la classe !

**Classe** :
- **Modèle** ou **plan** pour créer des objets
- Définit les **propriétés** (données, attributs) et les **méthodes** (actions, fonctions)

**Exemple : Classe FileInfo**

| Type | Nom | Description |
|------|-----|-------------|
| **Attributs** | Name | Nom du fichier |
| | FullName | Chemin complet |
| | Length | Taille en octets |
| | CreationTime | Date et heure de création |
| | LastWriteTime | Date de dernière modification |
| | Extension | Extension du fichier |
| **Méthodes** | CopyTo() | Copier le fichier |
| | MoveTo() | Déplacer le fichier |
| | Delete() | Supprimer le fichier |
| | OpenText() | Ouvrir le fichier pour lire le texte |

---

### Exemple de Classe

> [!success] en action !

**Get-ChildItem** retourne des objets :
- **FileInfo** (pour les fichiers)
- **DirectoryInfo** (pour les dossiers)

**Au lieu de manipuler du texte, on accède directement aux propriétés** :

```powershell
# Récupérer un fichier
$file = Get-ChildItem monFichier.txt

# Accéder aux propriétés
$file.Length        # Taille
$file.CreationTime  # Date de création

# Utiliser une méthode
$file.CopyTo("nouveauFichier.txt")
```

---

### Les caractères d'échappement

> [!info] Caractères spéciaux

**Caractère d'échappement** : **`** (backtick, accent grave)

**Les séquences d'échappement** ne sont interprétées que dans des chaînes avec **"** (double quotes).

**Caractères d'échappement courants** :

| Séquence | Description |
|----------|-------------|
| **`n** | Nouvelle ligne (retour à la ligne) |
| **`t** | Tabulation |
| **`r** | Retour chariot |
| **`"** | Guillemet double (échapper ") |
| **``** | Backtick littéral |

---

#### Exemples

```powershell
Write-Output "`nCeci est un saut de ligne`nEt ceci est une tabulation `tentre les mots"
```

**Résultat** :
```
Ceci est un saut de ligne
Et ceci est une tabulation      entre les mots
```

---

### La lecture du flux

> [!note] Sensibilité

**PowerShell n'est PAS sensible à** :
- ❌ La **casse** (majuscules/minuscules)
- ❌ Les **espaces** ou **tabulations**

**Exemples valides** :
```powershell
# Ces 3 lignes sont équivalentes
Write-Output "Hello"
wRite-ouTput "Hello"
WRITE-OUTPUT "Hello"

# Espaces et tabulations ignorés
Write-Output       "Hello"
Write-Output"Hello"
```

**Exemple complet** :
```powershell
wRite-ouTput "Ceci est executé correctement`nDe même que la commande suivante"; geT-chIldITeM -paTH *
```

---

### Les commandes

> [!important] Alias et cmdlets

**PowerShell utilise un système d'alias** prédéfini qui permet l'utilisation de commandes d'autres langages :

**Commandes batch (DOS)** :
- `cd`, `dir`, `copy`, `move`, `del`, `cls`, etc.

**Commandes Linux** :
- `ls`, `cp`, `mv`, `rm`, `cat`, `pwd`, etc.

**En réalité** :
- Les alias prédéfinis pointent vers des **cmdlets PowerShell**

**Exemples** :

| Alias | Cmdlet PowerShell |
|-------|-------------------|
| `dir`, `ls` | `Get-ChildItem` |
| `cd` | `Set-Location` |
| `copy`, `cp` | `Copy-Item` |
| `move`, `mv` | `Move-Item` |
| `del`, `rm` | `Remove-Item` |
| `cat` | `Get-Content` |
| `cls`, `clear` | `Clear-Host` |

---

### Les cmdlets

> [!important] Les commandes (suite)

**Un cmdlet** est sous la forme :
```
<Verbe>-<Nom> -<nom_option> <valeur>
```

**Format** : `Verbe-Nom`

**Verbes courants** :

| Verbe | Action |
|-------|--------|
| **Get** | Récupérer, obtenir |
| **Set** | Définir, modifier |
| **New** | Créer |
| **Remove** | Supprimer |
| **Start** | Démarrer |
| **Stop** | Arrêter |
| **Test** | Tester |
| **Write** | Écrire |

**Exemples** :
```powershell
Get-ChildItem       # Lister les fichiers
Get-Process         # Lister les processus
Set-Location        # Changer de répertoire
New-Item            # Créer un élément
Remove-Item         # Supprimer un élément
Start-Service       # Démarrer un service
Stop-Service        # Arrêter un service
Write-Host          # Afficher du texte
Where-Object        # Filtrer des objets
```

---

### Quotes & Double quotes

> [!important] Apostrophes et guillemets

**PowerShell permet d'encapsuler des caractères** :

**Single quotes (apostrophes) `'`** :
- ❌ Aucun métacaractère sauf `'` (fin de chaîne)
- Contenu interprété **littéralement**

**Double quotes (guillemets doubles) `"`** :
- ✅ Métacaractères : `$`, `` ` ``, `"`, `\`
- Contenu **interprété** (variables substituées)

> [!warning] Attention
> Ne pas confondre `'` (apostrophe) et `` ` `` (backtick, caractère d'échappement).

---

#### Exemples avec single quotes

```powershell
$i = 5
Write-Output 'The value of $i is $i'
# Résultat : The value of $i is $i

Write-Output 'The value of $(2+3) is 5'
# Résultat : The value of $(2+3) is 5
```

**Avec single quotes** : Tout est littéral, pas d'interprétation.

---

#### Exemples avec double quotes

```powershell
$j = 3
Write-Output "The value of $j is $j"
# Résultat : The value of 3 is 3

Write-Output "The value of `$j is $j"
# Résultat : The value of $j is 3

Write-Output "The value of $(2+3) is 5"
# Résultat : The value of 5 is 5
```

**Avec double quotes** : Variables et expressions substituées.

---

## Les variables - Utilisation standard

### Identifiant de variable

> [!important] Les variables

**Une variable** :
- ✅ Commence **toujours** par un **$**
- ✅ Est constituée de lettres, chiffres, caractères spéciaux
- ✅ Le nom (ou une partie) peut être le contenu d'une autre variable
- ✅ Doit être **unique** et ne pas être un mot-clé du langage
- ✅ N'est **pas sensible à la casse**

**Exemples valides** :
```powershell
$Var
$MyVariable
$my_variable
$Variable123
$_temp
```

**Exemples invalides** :
```powershell
Var        # Manque le $
$for       # Mot-clé réservé
$123Var    # Commence par un chiffre
```

---

### Convention de nom

> [!tip] Bonnes pratiques

**En plus des règles imposées par le langage** :

- ✅ Le **PascalCase** est souvent utilisé : `$MaVariable`, `$NombreFichiers`
- ✅ Prendre des noms de variables **qui ont un sens**
- ✅ En **anglais** ou en **français** (mais cohérent)

**Exemples** :
```powershell
# Bons noms (PascalCase, significatifs)
$UserName
$FilePath
$TotalCount
$ServiceStatus

# Mauvais noms (peu clairs)
$x
$var1
$temp
$data
```

---

### Utiliser des variables

> [!success] Comment ça marche ?

**Syntaxe** : `$NomVariable = valeur`

**Détruire une variable** : `Remove-Variable`

```powershell
# Créer une variable
$Var = "Coucou"

# Afficher
Write-Host $Var
# Résultat : Coucou

# Ou simplement
$Var
# Résultat : Coucou
```

---

#### Copie et modification

```powershell
# Copier une variable
$Var1 = $Var

# Modifier l'originale
$Var = "Hello"

# Afficher
$Var
# Résultat : Hello

# Concaténation
$Var + " World !"
# Résultat : Hello World !

# Concaténation complexe avec tabulations
$Var1 + ":`t `t" + $Var + " World !"
# Résultat : Coucou:		Hello World !
```

---

#### Supprimer des variables

```powershell
# Supprimer une ou plusieurs variables
Remove-Variable Var, Var1

# Essayer d'afficher après suppression
$Var1 + ":`t `t" + $Var + " World !"
# Résultat : :		 World !
# (variables vides)
```

---

### Utiliser des variables (suite)

> [!success] Exemple pratique

```powershell
# Définir un nom de dossier
$MyDirectory = "MonDossier"

# Créer le dossier
New-Item -ItemType Directory -Path $MyDirectory

# Lister
Get-ChildItem
```

**Résultat** :
```
    Répertoire : C:\Lab

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        30/05/2022     16:24                MonDossier
```

---

### Interpolation de variable

> [!info] Inclure une variable dans une chaîne

**Interpolation** : Permet d'inclure la valeur d'une variable directement dans une chaîne de caractères en utilisant la syntaxe **$()**.

**Syntaxe** : `"Texte $($Variable.Attribut) texte"`

---

### Affichage dans une chaîne

> [!success] A vous de jouer !

**On utilise $($Variable.Attribut)** dans une chaîne pour afficher un attribut.

```powershell
# Récupérer l'hôte
$Host = Get-Host

# Afficher un attribut dans une chaîne
Write-Host "La version est $($Host.Version)"
# Résultat : La version est 5.1.19041.4170
```

**Sans interpolation** :
```powershell
Write-Host "La version est $Host.Version"
# Résultat : La version est System.Management.Automation.Internal.Host.InternalHost.Version
# (Ne fonctionne pas correctement)
```

---

## Les variables - Utilisation avancée

### Invoke-Expression

> [!important] Exécution dynamique

**Invoke-Expression** : Cmdlet qui permet d'**exécuter une chaîne de caractères en tant que commande** PowerShell.

**Usage** : Exécution d'une construction dynamique de chaîne de caractères.

> [!warning] Attention à la source !
> N'utilisez Invoke-Expression qu'avec du code de **confiance**. Risque de sécurité si la source est externe.

---

### Exemple Invoke-Expression

> [!success] A vous de jouer !

**Exercice** :
1. Déclarer une variable `$Commande` ayant pour valeur `"whoami"`
2. Afficher la valeur de la variable
3. Exécuter `whoami` en utilisant la variable avec `Invoke-Expression`

```powershell
# Déclarer
$Commande = "whoami"

# Afficher
Write-Host $Commande
# Résultat : whoami

$Commande
# Résultat : whoami

# Exécuter
Invoke-Expression -Command $Commande
# Résultat : Computer1\Wilder
```

---

### Les méthodes

> [!important] Fonctions sur objets

**Méthodes** : Fonctions associées à un objet qui effectuent une action spécifique.

**Syntaxe** : `<donnée>.<méthode(paramètre)>`

**On peut les appliquer aux variables.**

---

#### Exemples de méthodes

**Sur une chaîne directe** :
```powershell
"hello"
# Résultat : hello

"hello".ToUpper()
# Résultat : HELLO

"                hello        hello     "
# Résultat :                 hello        hello

"                hello        hello     ".Trim()
# Résultat : hello        hello
```

**Sur une variable** :
```powershell
$String = "hello"
$String.ToUpper()
# Résultat : HELLO
```

---

#### Méthodes courantes sur chaînes

| Méthode | Description | Exemple |
|---------|-------------|---------|
| `.ToUpper()` | Convertir en majuscules | `"hello".ToUpper()` → `HELLO` |
| `.ToLower()` | Convertir en minuscules | `"HELLO".ToLower()` → `hello` |
| `.Trim()` | Supprimer espaces début/fin | `"  hello  ".Trim()` → `hello` |
| `.Replace(old, new)` | Remplacer texte | `"hello".Replace("l", "r")` → `herro` |
| `.Split(char)` | Découper en tableau | `"a,b,c".Split(",")` → `@("a","b","c")` |
| `.Length` | Longueur (propriété) | `"hello".Length` → `5` |
| `.Substring(start, length)` | Extraire sous-chaîne | `"hello".Substring(0,3)` → `hel` |

---

### Typage

> [!important] Type des variables

**Le typage** : Désigne la nature du type de données qu'une variable peut contenir.

**Quelques exemples de types** :

| Type | Description | Exemple |
|------|-------------|---------|
| `[int]` | Entier 32 bits | `42` |
| `[string]` | Chaîne de caractères | `"Hello"` |
| `[bool]` | Booléen | `$true`, `$false` |
| `[double]` | Nombre à virgule flottante | `3.14` |
| `[array]` | Tableau | `@(1, 2, 3)` |

**La méthode `GetType()`** permet de connaître le type.

---

#### Exemples GetType()

```powershell
# Type d'une chaîne directe
"Bonjour".GetType()

# Résultat :
IsPublic IsSerial Name         BaseType
-------- -------- ----         --------
True     True     String       System.Object
```

```powershell
# Type d'une variable chaîne
$Var1 = "hello"
$Var1.GetType()

# Résultat :
IsPublic IsSerial Name         BaseType
-------- -------- ----         --------
True     True     String       System.Object
```

```powershell
# Type d'une variable entier
$Var2 = 10
$Var2.GetType()

# Résultat :
IsPublic IsSerial Name         BaseType
-------- -------- ----         --------
True     True     Int32        System.ValueType
```

---

### Transtypage ou casting

> [!important] Conversion de type

**Transtypage (casting)** : Mécanisme de conversion **explicite** d'une valeur d'un type à un autre.

**Syntaxe** : `[<Type>] <donnée>`

```powershell
# Convertir une chaîne en entier
[Int]$Var = "10"

# Vérifier le type
$Var.GetType()

# Résultat :
IsPublic IsSerial Name         BaseType
-------- -------- ----         --------
True     True     Int32        System.ValueType

# Opération arithmétique (maintenant possible)
$Var + 5
# Résultat : 15
```

**Sans transtypage** :
```powershell
$Var = "10"
$Var + 5
# Résultat : 105 (concaténation de chaînes, pas addition)
```

---

### Substitution de commandes

> [!important] Récupérer le résultat d'une commande

**Substitution** : Récupérer le résultat d'une commande (au lieu de l'afficher).

**Syntaxe** : `$(commande)`

**Utilisation** :
- Stocker dans une variable
- Utiliser dans une autre commande

---

#### Exemples de substitution

**Afficher directement** :
```powershell
Get-Host | Select-Object Version

# Résultat :
Version
-------
5.1.19041.1237
```

**Stocker dans une variable** :
```powershell
$HostVersion = $(Get-Host | Select-Object Version)
$HostVersion

# Résultat :
Version
-------
5.1.19041.1237
```

---

#### Substitution avancée

**Utiliser le contenu d'une variable comme nom** :
```powershell
$Name = "Var"
$Var = "Hello"

Write-Host (Get-Variable -Name $Name -ValueOnly)
# Résultat : Hello
```

---

### Substitution arithmétique

> [!success] Faire des calculs

**Effectuer un calcul** :

**Syntaxe** :
- `<operation>`
- ou `$(<operation>)`

---

#### Exemples de calculs

**Calcul simple** :
```powershell
12 * 6
# Résultat : 72
```

**Attention avec Write-Host** :
```powershell
Write-Host 12 * 6
# Résultat : 12 * 6 (pas de calcul !)

Write-Host $(12 * 6)
# Résultat : 72
```

---

#### Calculs avec variables

```powershell
# Stockage direct
$Total1 = 10 + 2
$Total1
# Résultat : 12

# Stockage avec $()
$Total2 = $(7 + 3)
$Total2
# Résultat : 10

# Calcul complexe
Write-Host $($Total2 * 2 + 1)
# Résultat : 21
```

---

### Portée des variables

> [!important] Scope (portée)

**La portée** protège les variables. Les niveaux de portée protègent les éléments qui ne doivent pas être modifiés.

**4 niveaux de portée** :

| Portée | Description |
|--------|-------------|
| **Global** | Dans une console PowerShell, instance d'exécution, session |
| **Script** | Lors de l'exécution d'un script (variables du script uniquement) |
| **Local** | Lors de l'exécution d'une commande ou script (portée actuelle) |
| **Private** | Empêche la visibilité en dehors de la portée définie |

---

#### Portée Global

**Global** :
- Variables présentes et disponibles dans la **portée globale**
- Ensemble des variables, alias et fonctions définis dans le **profil PowerShell**
- Disponibles dans la console

---

#### Portée Script

**Script** :
- Variables définies dans le script
- Uniquement disponibles pour la **portée du script**
- **Non disponibles** pour la portée globale ou parente

---

#### Portée Local

**Local** :
- Variables définies dans la portée actuelle
- Les variables de la portée script sont considérées comme **portée locale** pour le script

---

#### Portée Private

**Private** :
- Empêche la **visibilité** d'une variable en dehors de la portée définie

---

### Portée locale - Exemple

> [!info] Variables locales au script

**Script** :
```powershell
# script.ps1
$Greeting = "Hello, World!"
$Greeting
```

**Exécution** :
```powershell
# Vérifier avant
$Greeting
# Résultat : (vide)

# Exécuter le script
.\script.ps1
# Résultat : Hello, World !

# Vérifier après
$Greeting
# Résultat : (vide)
```

> [!note] Conclusion
> `$Greeting` a une portée **locale** (au niveau du script). Elle n'est **pas disponible** pour la portée parente (la console).

---

### Portée globale - Exemple

> [!info] Héritage de la console

**Script** :
```powershell
# script.ps1
$Greeting
```

**Exécution** :
```powershell
# Exécuter le script (vide)
.\script.ps1
# Résultat : (rien)

# Définir dans la console
$Greeting = "Hello from the global scope !"

# Exécuter le script
.\script.ps1
# Résultat : Hello from the global scope !

# Vérifier dans la console
$Greeting
# Résultat : Hello from the global scope !
```

> [!note] Conclusion
> Le contenu de `$Greeting` a été **hérité** de la console (portée parente) par le script (portée enfant).

---

### Modificateur de portée

> [!important] Forcer une portée

**Modificateurs** :

| Modificateur | Description |
|--------------|-------------|
| `$global:Var` | Variable existe dans la portée **globale** |
| `$local:Var` | Variable existe dans la portée **locale** |
| `$private:Var` | Variable seulement visible dans la portée **actuelle** |
| `$script:Var` | Variable existe dans la portée **script** (ou globale si pas de script) |

---

### Modifier la portée d'une variable

> [!success] Exemple pratique

**Script** :
```powershell
# script.ps1
$global:Greeting = "Hello, World !"
```

**Exécution** :
```powershell
# Définir dans la console
$Greeting = "Hello, Toto !"
$Greeting
# Résultat : Hello, Toto !

# Exécuter le script
.\script.ps1

# Vérifier dans la console
$Greeting
# Résultat : "Hello, World !"
```

> [!note] Conclusion
> `$Greeting` est définie dans la **portée globale** de la console. En exécutant le script, `$Greeting` prend une nouvelle valeur grâce au **modificateur `$global:`**. La valeur a été modifiée dans la portée globale.

---

### Variables spéciales

> [!important] Variables prédéfinies

**Variables automatiques** :

| Variable | Description |
|----------|-------------|
| **$?** | État d'exécution de la dernière opération (`$true` ou `$false`) |
| **$_** | Objet actuel dans le pipeline |
| **$ARGS** | Tableau des paramètres non déclarés passés à une fonction/script |
| **$Null** | Valeur NULL ou vide |
| **$True** | Valeur booléenne True |
| **$False** | Valeur booléenne False |
| **$Home** | Répertoire personnel de l'utilisateur |
| **$Profile** | Chemin du profil PowerShell |
| **$PSCulture** | Langue/culture actuelle |

[Documentation complète](https://learn.microsoft.com/fr-fr/powershell/module/microsoft.powershell.core/about/about_automatic_variables)

---

### Un exemple avec $_

> [!success] Comment ça marche ?

**$_** : Variable spéciale qui contient l'**objet actuel** dans le pipeline.

**Utilisation** :
- S'utilise derrière un **|** (pipe)
- Ou dans une boucle **foreach**
- On peut y ajouter des attributs : `$_.Attribut`

**Get-Member** permet de connaître les attributs possibles.

---

#### Exemple pratique

```powershell
# Créer un dossier
$MyDirectory = "MonDossier2"
New-Item -ItemType Directory -Path $MyDirectory

# Résultat :
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        30/05/2022     16:24                MonDossier2

# Lister les attributs disponibles
Get-ChildItem | Get-Member -MemberType Property

# Résultat :
Name              MemberType       Definition
----              ----------       ----------
[...]
Name              Property         string Name {get;}
[...]

# Filtrer avec $_
Get-ChildItem | Where-Object {$_.Name -like "*mon*"}

# Résultat :
Mode             LastWriteTime         Length Name
----             -------------         ------ ----
d-----           30/05/2022     16:24         MonDossier2
```

---

### Variables d'environnement

> [!info] Une parenthèse

**Variables d'environnement** :
- Variables **dynamiques** et **globales** au sein d'un système d'exploitation
- Les différents processus peuvent y accéder
- Obtenir des informations sur la **configuration actuelle** du système
- Toutes sous la forme **$env:xxx**

**Lister toutes les variables** :
```powershell
Get-ChildItem env:
```

[Documentation Microsoft](https://learn.microsoft.com/fr-fr/powershell/module/microsoft.powershell.core/about/about_environment_variables)

---

### Exemple avec variables d'environnement

> [!success] A vous de jouer

**Exercice** : Créer un script qui affiche :
- Le chemin complet du répertoire personnel de l'utilisateur courant
- Le chemin complet du profil PowerShell
- La langue utilisée
- Le type d'architecture processeur

```powershell
# script.ps1

Write-Host "Chemin du répertoire personnel : $Home"
Write-Host "Chemin du profil PS : $Profile"
Write-Host "Langue du système : $PSCulture"
Write-Host "Type d'architecture : $($env:PROCESSOR_ARCHITECTURE)"
```

---

### Shell et variables

> [!warning] Isolation des consoles

**Chaque console PowerShell est indépendante.**

**Les variables déclarées dans une console ne sont PAS accessibles dans les autres.**

**Exemple** :
```powershell
# Console 1
$Var = "Hello"

# Console 2
$Var
# Résultat : (vide)
```

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définition PowerShell

**PowerShell** :
- Interpréteur de commandes et langage de script Microsoft
- Extension **.PS1**
- Interpréteur : **PowerShell.exe**
- Manipule des **objets .NET** (pas du texte)
- Automatisation de tâches et gestion de configuration

**Script vs Programme** :
- Script = **interprété** (pas de compilation)
- Programme = **compilé** (avant exécution)

**Avantages** :
- Gagner du temps
- Automatiser les tâches répétitives
- Décomposer des tâches complexes

---

### Objets .NET

**Objet** = Instance d'une classe :
- **Propriétés** (attributs, données)
- **Méthodes** (actions, fonctions)

**Exemple** :
```powershell
$file = Get-ChildItem fichier.txt
$file.Name          # Propriété
$file.Length        # Propriété
$file.CopyTo()      # Méthode
```

---

### La base

**Classes (POO)** :
- Modèle pour créer des objets
- Définit propriétés et méthodes
- Exemple : `FileInfo`, `DirectoryInfo`

**Caractères d'échappement** :
- **`** (backtick)
- `n (nouvelle ligne), `t (tabulation)
- Seulement dans **" "** (double quotes)

**Sensibilité** :
- ❌ Pas sensible à la casse
- ❌ Pas sensible aux espaces/tabulations

**Commandes** :
- **Alias** : `dir`, `ls`, `cd`, `cp`, `mv`...
- **Cmdlets** : `<Verbe>-<Nom>`
- Exemples : `Get-ChildItem`, `Set-Location`, `New-Item`

**Quotes** :
- **Single quotes `' '`** : Littéral, pas d'interprétation
- **Double quotes `" "`** : Variables substituées

---

### Variables - Standard

**Identifiant** :
- Commence par **$**
- Lettres, chiffres, caractères spéciaux
- Pas sensible à la casse
- Unique, pas un mot-clé

**Convention** : **PascalCase**

**Syntaxe** :
```powershell
$NomVariable = valeur
Remove-Variable NomVariable  # Supprimer
```

**Interpolation** :
```powershell
"La version est $($Host.Version)"
```

---

### Variables - Avancé

**Invoke-Expression** :
- Exécuter une chaîne comme commande
- ⚠️ Attention à la source (sécurité)

**Méthodes** :
- `<donnée>.<méthode()>`
- Exemples : `.ToUpper()`, `.ToLower()`, `.Trim()`, `.Replace()`

**Typage** :
- `.GetType()` : Connaître le type
- Types : `[int]`, `[string]`, `[bool]`, `[double]`, `[array]`

**Transtypage (casting)** :
```powershell
[Int]$Var = "10"
```

**Substitution de commandes** :
```powershell
$Result = $(Get-ChildItem)
```

**Substitution arithmétique** :
```powershell
$Total = $(10 + 5)
Write-Host $(12 * 6)
```

---

### Portée des variables

**4 niveaux** :

| Portée | Description |
|--------|-------------|
| **Global** | Console, session |
| **Script** | Variables du script uniquement |
| **Local** | Portée actuelle |
| **Private** | Invisible en dehors |

**Modificateurs** :
- `$global:Var`
- `$script:Var`
- `$local:Var`
- `$private:Var`

**Héritage** :
- Variables globales accessibles dans les scripts
- Variables de script non accessibles dans la console (sauf `$global:`)

---

### Variables spéciales

| Variable | Usage |
|----------|-------|
| `$?` | État dernière opération |
| `$_` | Objet actuel pipeline |
| `$ARGS` | Paramètres fonction/script |
| `$Null` | Valeur NULL |
| `$True` / `$False` | Booléens |
| `$Home` | Répertoire personnel |
| `$Profile` | Profil PowerShell |
| `$PSCulture` | Langue système |

**Variables d'environnement** : `$env:xxx`

---

### Bonnes pratiques

**Nommage** :
- ✅ Scripts : extension **.PS1**
- ✅ Commentaires avec **#**
- ✅ Variables : **PascalCase**, noms significatifs

**Sécurité** :
- ⚠️ `Invoke-Expression` : Source de confiance uniquement
- ⚠️ Ne jamais exécuter de code non vérifié

**Organisation** :
- ✅ Commenter le code
- ✅ Noms de variables clairs
- ✅ Documenter les scripts (auteur, date, objectif)

---

### Pièges à éviter

> [!warning] Erreurs courantes

**Quotes** :
1. ❌ Confondre `'` (apostrophe) et `` ` `` (backtick)
2. ❌ Utiliser single quotes pour interpoler des variables
3. ❌ Oublier `$()` pour les expressions dans double quotes

**Variables** :
1. ❌ Oublier le `$` devant une variable
2. ❌ Utiliser des noms de variables peu clairs (`$x`, `$temp`)
3. ❌ Ne pas vérifier le type avant une opération
4. ❌ Confondre concaténation (`"10" + 5` = `"105"`) et addition (`10 + 5` = `15`)

**Portée** :
1. ❌ Compter sur une variable de script dans la console
2. ❌ Ne pas comprendre l'héritage des portées
3. ❌ Modifier involontairement une variable globale

**Commandes** :
1. ❌ Utiliser `Write-Host` pour les calculs sans `$()`
2. ❌ Ne pas tester le résultat d'une commande (`$?`)

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **PowerShell** | Interpréteur de commandes et langage de script Microsoft |
| **Script** | Fichier texte (.PS1) contenant des commandes PowerShell |
| **Console** | Shell en ligne de commandes (PowerShell.exe) |
| **.NET** | Framework Microsoft pour construire et exécuter des applications |
| **Objet** | Instance d'une classe (propriétés + méthodes) |
| **Classe** | Modèle pour créer des objets (POO) |
| **Propriété** | Attribut d'un objet (données) |
| **Méthode** | Fonction d'un objet (actions) |
| **Cmdlet** | Commande PowerShell format Verbe-Nom |
| **Alias** | Nom alternatif pour une cmdlet |
| **Variable** | Conteneur pour stocker une valeur (commence par $) |
| **Typage** | Nature du type de données d'une variable |
| **Transtypage** | Conversion explicite d'un type à un autre |
| **Casting** | Synonyme de transtypage |
| **Interpolation** | Inclusion d'une variable dans une chaîne |
| **Substitution** | Récupérer le résultat d'une commande |
| **Portée** | Scope, visibilité d'une variable |
| **Global** | Portée console/session |
| **Script** | Portée d'un script |
| **Local** | Portée actuelle |
| **Private** | Portée invisible en dehors |
| **Pipeline** | Passage d'objets entre commandes (pipe \|) |
| **$_** | Variable spéciale, objet actuel dans pipeline |
| **Backtick** | Caractère d'échappement `` ` `` |
| **Single quote** | Apostrophe `'` (littéral) |
| **Double quote** | Guillemet `"` (interprétation) |
| **PascalCase** | Convention de nommage (MaVariable) |
| **GetType()** | Méthode pour connaître le type |
| **Invoke-Expression** | Exécuter une chaîne comme commande |
| **Get-Member** | Lister propriétés et méthodes d'un objet |
| **Remove-Variable** | Supprimer une variable |
| **Variables d'environnement** | Variables système ($env:xxx) |
| **POO** | Programmation Orientée Objet |

---

## 📖 Références externes

> [!note] Documentation officielle Microsoft
> Ressources pour approfondir PowerShell et devenir autonome.

| Thème | Ressource | Description | Lien |
|-------|-----------|-------------|------|
| **Documentation PowerShell** | Microsoft Learn | Documentation officielle complète | [learn.microsoft.com/powershell](https://learn.microsoft.com/en-us/powershell/) |
| **About Topics** | About Articles | Articles détaillés sur concepts PowerShell | [learn.microsoft.com/powershell/about](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/) |
| **Variables automatiques** | About Automatic Variables | Liste complète des variables spéciales | [learn.microsoft.com/powershell/about_automatic_variables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables) |
| **Variables d'environnement** | About Environment Variables | Documentation variables $env: | [learn.microsoft.com/powershell/about_environment_variables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables) |
| **Portée des variables** | About Scopes | Documentation complète des portées | [learn.microsoft.com/powershell/about_scopes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scopes) |
| **Cmdlets** | Cmdlet Overview | Vue d'ensemble des cmdlets | [learn.microsoft.com/powershell/cmdlet-overview](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/cmdlet-overview) |
| **PowerShell Gallery** | PSGallery | Dépôt de modules et scripts | [powershellgallery.com](https://www.powershellgallery.com) |
| **PowerShell ISE** | ISE Guide | Guide PowerShell ISE | [learn.microsoft.com/powershell/ise](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise) |
| **Visual Studio Code** | VS Code PowerShell Extension | Extension VS Code pour PowerShell | [marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell) |

> [!tip] Comment utiliser ces ressources
> Ces liens te permettront de :
> - **Approfondir** les concepts PowerShell (objets, classes, méthodes)
> - **Consulter** la documentation officielle des cmdlets
> - **Apprendre** les bonnes pratiques Microsoft
> - **Découvrir** les variables automatiques et d'environnement
> - **Maîtriser** la portée des variables (scopes)
> - **Installer** des modules depuis PowerShell Gallery
> - **Optimiser** ton environnement de développement (ISE, VS Code)

---

### Ressources complémentaires recommandées

> [!info] Pour aller plus loin

| Thème | Ressource | Description | Lien |
|-------|-----------|-------------|------|
| **PowerShell.org** | Communauté PowerShell | Forums, articles, ressources | [powershell.org](https://powershell.org) |
| **SS64** | Référence de commandes | Référence rapide cmdlets | [ss64.com/ps](https://ss64.com/ps/) |
| **PowerShell Magazine** | Articles et tutoriels | Magazine en ligne PowerShell | [powershellmagazine.com](https://www.powershellmagazine.com) |
| **GitHub PowerShell** | Code source PowerShell | Repository officiel open source | [github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell) |

---

> [!success] Document de révision complet
> Ce document couvre l'intégralité de **PowerShell Partie 1**. Tu as maintenant tous les éléments pour :
> - Comprendre le **langage PowerShell** et les **objets .NET**
> - Maîtriser les **variables** (standard et avancé)
> - Utiliser les **cmdlets** et comprendre les **alias**
> - Manipuler les **types** et le **transtypage**
> - Gérer la **portée des variables** (global, script, local, private)
> - Utiliser les **variables spéciales** ($_, $?, $ARGS...)
> - Appliquer les **bonnes pratiques** de scripting
> - Créer tes **premiers scripts PowerShell**
> 
> **Suite** : PowerShell Partie 2 (structures de contrôle, boucles, fonctions...)
> 
> **Bon courage pour la préparation de ton titre RNCP TSSR !** 🎓💻✨

---

**Fin du document de révision**
