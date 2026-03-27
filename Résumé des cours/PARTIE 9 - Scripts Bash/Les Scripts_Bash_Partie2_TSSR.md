# Les scripts Bash - Partie 2
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Logique de scripting - Structures conditionnelles et itératives  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Les structures conditionnelles|Les structures conditionnelles]]
   - [[#Les tests|Les tests]]
   - [[#Codes de retour|Codes de retour]]
   - [[#Comparaison chaînes|Comparaison de chaînes]]
   - [[#Comparaison nombres|Comparaison de nombres]]
   - [[#Opérateurs logiques|Opérateurs logiques]]
   - [[#Opérateurs fichiers|Opérateurs sur fichiers/dossiers]]
   - [[#Structure if|Structure if...then...else]]
   - [[#Structure case|Structure case]]
3. [[#Les structures itératives|Les structures itératives]]
   - [[#Définition boucles|Définition]]
   - [[#Boucle for|Boucle for]]
   - [[#Boucle for arithmétique|Boucle for arithmétique]]
   - [[#Boucle while|Boucle while]]
4. [[#Ressources|Ressources et références]]
5. [[#Points clés à retenir|Points clés]]
6. [[#Glossaire technique|Glossaire]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Cette **partie 2** du cours sur les scripts Bash se concentre sur la **logique de scripting** : structures conditionnelles (if, case) et structures itératives (boucles for, while). Ces éléments permettent de créer des scripts **dynamiques** et **intelligents**.

### Pourquoi maîtriser la logique de scripting ?

En tant que **TSSR**, vous devez :

- Créer des scripts capables de **prendre des décisions**
- **Automatiser** des tâches répétitives
- **Traiter** des données en masse (fichiers, utilisateurs, logs)
- Gérer des **conditions** et des **erreurs**
- Créer des **menus** et interfaces interactives

**Objectifs de cette partie** :
- ✅ Maîtriser les tests et conditions
- ✅ Utiliser les structures if/elif/else et case
- ✅ Créer des boucles for et while
- ✅ Combiner logique et automatisation

---

## Les structures conditionnelles

### Les tests

> [!important] Vrai ou faux en Bash

**Principe fondamental** :
Pour Bash, un test est :
- **Vrai** s'il vaut **0**
- **Faux** s'il vaut **n'importe quelle autre valeur** (généralement 1)

> [!warning] Attention : Convention inverse !
> C'est **contre-intuitif** pour ceux qui connaissent d'autres langages :
> - **0 = Succès = Vrai** ✅
> - **Non-zéro = Erreur = Faux** ❌
> 
> C'est une convention **Unix/Linux** héritée des codes de sortie.

---

#### Codes de sortie et tests

**Relation** :
- Le **code de sortie** (status code) d'une commande qui a **réussi** équivaut à **vrai** (0)
- Le code de sortie d'une commande qui a **échoué** équivaut à **faux** (non-zéro)

**Variable spéciale** : `$?`
- Récupère le **code de sortie** de la **dernière commande** exécutée

---

#### Syntaxes de tests

On peut construire des tests de **trois façons** :

**1. Avec la commande `test`** :
```bash
test <expression>
```

**2. Avec les crochets simples `[ ]`** (équivalent à `test`) :
```bash
[ <expression> ]
```

**3. Avec les doubles crochets `[[ ]]`** (extension Bash, recommandé) :
```bash
[[ <expression> ]]
```

> [!tip] Recommandation
> Préférer `[[ ]]` car :
> - Plus robuste (meilleure gestion des espaces)
> - Supporte les expressions régulières
> - Moins d'erreurs de syntaxe
> - Extension Bash moderne

---

### Codes de retour

> [!example] Exemple concret

**Commande réussie (retourne 0 = vrai)** :
```bash
wilder@host:~$ mkdir newDir
wilder@host:~$ echo $?
0
```

**Commande échouée (retourne 1 = faux)** :
```bash
wilder@host:~$ mkdir newDir
mkdir: impossible de créer le répertoire «newDir»: Le fichier existe
wilder@host:~$ echo $?
1
```

**Analyse** :
- Première création : succès → code `0`
- Deuxième création : échec (existe déjà) → code `1`

---

#### Utilisation pratique

> [!example] Tester une commande dans un script

```bash
#!/bin/bash

mkdir /tmp/test_dir

if [ $? -eq 0 ]
then
    echo "Répertoire créé avec succès"
else
    echo "Erreur lors de la création du répertoire"
fi
```

**Ou plus simplement (recommandé)** :
```bash
#!/bin/bash

if mkdir /tmp/test_dir
then
    echo "Répertoire créé avec succès"
else
    echo "Erreur lors de la création"
fi
```

> [!tip] Bonne pratique
> Tester **directement la commande** dans le `if` plutôt que de tester `$?`.

---

### Comparaison chaînes

> [!important] Tester des chaînes de caractères

Supposons deux chaînes `s1` et `s2` :

| Opérateur | Signification | Vrai si... |
|-----------|---------------|------------|
| `s1 = s2` ou `s1 == s2` | Égalité | Les chaînes sont identiques |
| `s1 != s2` | Différence | Les chaînes sont différentes |
| `-z s1` | Zero (vide) | `s1` est vide |
| `-n s1` | Non-zero (non vide) | `s1` n'est pas vide |

> [!warning] Attention aux espaces !
> Toujours mettre les variables entre **guillemets** : `"$var"`

---

#### Exemples

> [!example] Tests de chaînes

**Test d'égalité** :
```bash
wilder@host:~$ test 'identique' = 'identique'
wilder@host:~$ echo $?
0    # Vrai (succès)
```

**Test de différence** :
```bash
wilder@host:~$ test 'identique' = 'différent'
wilder@host:~$ echo $?
1    # Faux (échec)
```

**Avec crochets** :
```bash
wilder@host:~$ [ 'identique' = 'identique' ]
wilder@host:~$ echo $?
0    # Vrai

wilder@host:~$ [ 'identique' != 'différent' ]
wilder@host:~$ echo $?
0    # Vrai
```

**Test de chaîne vide** :
```bash
wilder@host:~$ [ -z '' ]
wilder@host:~$ echo $?
0    # Vrai (chaîne vide)
```

---

#### Exemples dans un script

> [!example] Vérification de saisie

```bash
#!/bin/bash

read -p "Entrez votre nom : " nom

if [ -z "$nom" ]
then
    echo "Erreur : Le nom ne peut pas être vide"
    exit 1
fi

if [ "$nom" = "admin" ]
then
    echo "Bienvenue administrateur !"
else
    echo "Bonjour $nom"
fi
```

---

### Comparaison nombres

> [!important] Opérateurs de comparaison numérique

Supposons deux nombres `n1` et `n2` :

| Opérateur | Signification | Description |
|-----------|---------------|-------------|
| `n1 -eq n2` | **EQ**ual | `n1` == `n2` |
| `n1 -ne n2` | **N**ot **E**qual | `n1` != `n2` |
| `n1 -lt n2` | **L**ess **T**han | `n1` < `n2` |
| `n1 -le n2` | **L**ess or **E**qual | `n1` <= `n2` |
| `n1 -gt n2` | **G**reater **T**han | `n1` > `n2` |
| `n1 -ge n2` | **G**reater or **E**qual | `n1` >= `n2` |

> [!tip] Mnémotechnique
> - **eq** = equal (égal)
> - **ne** = not equal (différent)
> - **lt** = less than (plus petit)
> - **le** = less or equal (plus petit ou égal)
> - **gt** = greater than (plus grand)
> - **ge** = greater or equal (plus grand ou égal)

---

#### Exemples

> [!example] Tests numériques

```bash
wilder@host:~$ trois=3
wilder@host:~$ [ $trois -eq 3 ]
wilder@host:~$ echo $?
0    # Vrai (3 == 3)

wilder@host:~$ [ 2 -ne $trois ]
wilder@host:~$ echo $?
0    # Vrai (2 != 3)

wilder@host:~$ deux=2
wilder@host:~$ [ $deux -lt $trois ]
wilder@host:~$ echo $?
0    # Vrai (2 < 3)
```

---

#### Exemple dans un script

> [!example] Vérification d'âge

```bash
#!/bin/bash

read -p "Entrez votre âge : " age

if [ $age -lt 18 ]
then
    echo "Vous êtes mineur"
elif [ $age -ge 18 -a $age -lt 65 ]
then
    echo "Vous êtes adulte"
else
    echo "Vous êtes senior"
fi
```

---

### Opérateurs logiques

> [!important] Combiner des conditions

Supposons `c1` et `c2` des conditions :

| Opérateur | Nom | Description | Vrai si... |
|-----------|-----|-------------|------------|
| `! c1` | **NOT** (NON) | Négation | `c1` est **faux** |
| `c1 -a c2` | **AND** (ET) | Conjonction | `c1` **et** `c2` sont vrais |
| `c1 -o c2` | **OR** (OU) | Disjonction | `c1` **ou** `c2` est vrai |

> [!tip] Alternative avec `[[ ]]`
> Avec les doubles crochets `[[ ]]`, on peut utiliser :
> - `!` pour NOT
> - `&&` pour AND
> - `||` pour OR
> 
> Exemple : `[[ $a -eq 1 && $b -eq 2 ]]`

---

#### Exemples

> [!example] Opérateurs logiques

**Négation (NOT)** :
```bash
wilder@host:~$ trois=3
wilder@host:~$ [ ! $trois -eq 3 ]
wilder@host:~$ echo $?
1    # Faux (NON 3==3 → NON vrai → faux)
```

**Conjonction (AND)** :
```bash
wilder@host:~$ [ 2 -lt $trois -a $trois -lt 4 ]
wilder@host:~$ echo $?
0    # Vrai (2 < 3 ET 3 < 4)
```

**Disjonction (OR)** :
```bash
wilder@host:~$ [ $trois -eq 2 -o $trois -eq 3 ]
wilder@host:~$ echo $?
0    # Vrai (3 == 2 OU 3 == 3 → faux OU vrai → vrai)
```

---

#### Table de vérité

> [!example] Rappel logique booléenne

**Opérateur AND (ET)** :

| c1 | c2 | c1 AND c2 |
|----|-------|-----------|
| V  | V     | **V** |
| V  | F     | F |
| F  | V     | F |
| F  | F     | F |

**Opérateur OR (OU)** :

| c1 | c2 | c1 OR c2 |
|----|-------|----------|
| V  | V     | **V** |
| V  | F     | **V** |
| F  | V     | **V** |
| F  | F     | F |

---

### Opérateurs fichiers

> [!important] Tests sur les fichiers et dossiers

Supposons `p` un chemin (path) :

| Opérateur | Signification | Vrai si... |
|-----------|---------------|------------|
| `-e p` | **E**xists | `p` existe |
| `-s p` | **S**ize > 0 | `p` existe et taille > 0 |
| `-f p` | **F**ile | `p` est un fichier |
| `-d p` | **D**irectory | `p` est un dossier |
| `-r p` | **R**eadable | Je peux lire `p` |
| `-w p` | **W**ritable | Je peux écrire dans `p` |
| `-x p` | e**X**ecutable | Je peux exécuter `p` |
| `-L p` | **L**ink | `p` est un lien symbolique |

---

#### Exemples

> [!example] Tests sur fichiers

**Existence** :
```bash
wilder@host:~$ [ -e /tmp ]
wilder@host:~$ echo $?
0    # Vrai (/tmp existe)
```

**Fichier régulier** :
```bash
wilder@host:~$ [ -f /etc/passwd ]
wilder@host:~$ echo $?
0    # Vrai (/etc/passwd est un fichier)
```

**Permissions** :
```bash
wilder@host:~$ [ -r /etc/passwd ]
wilder@host:~$ echo $?
0    # Vrai (je peux lire /etc/passwd)

wilder@host:~$ [ -w /etc/passwd ]
wilder@host:~$ echo $?
1    # Faux (je ne peux pas écrire /etc/passwd sans sudo)
```

---

#### Exemple dans un script

> [!example] Vérification d'un fichier de configuration

```bash
#!/bin/bash

config_file="/etc/mon_app/config.conf"

# Vérifier que le fichier existe
if [ ! -e "$config_file" ]
then
    echo "Erreur : Le fichier de configuration n'existe pas" >&2
    exit 1
fi

# Vérifier que c'est bien un fichier
if [ ! -f "$config_file" ]
then
    echo "Erreur : $config_file n'est pas un fichier régulier" >&2
    exit 1
fi

# Vérifier les permissions de lecture
if [ ! -r "$config_file" ]
then
    echo "Erreur : Pas de permission de lecture sur $config_file" >&2
    exit 1
fi

echo "Fichier de configuration OK"
```

---

### Structure if

> [!important] Structure conditionnelle if...then...else

**Syntaxe complète** :
```bash
if condition
then
    instructions
elif condition2    # Optionnel
then
    instructions
else              # Optionnel
    instructions
fi
```

> [!note] Algorithme équivalent
> ```
> SI condition
> ALORS
>     instructions
> SINON SI condition2
> ALORS
>     instructions
> SINON
>     instructions
> FIN SI
> ```

---

#### Exemples

> [!example] if simple

```bash
wilder@host:~$ if mkdir newDir
> then
>     echo "Création dossier succès"
> else
>     echo "Création dossier échec"
> fi
Création dossier succès
```

---

> [!example] if avec test

```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Erreur : Aucun argument fourni"
    exit 1
fi

echo "Premier argument : $1"
```

---

> [!example] if...elif...else complet

```bash
#!/bin/bash

read -p "Entrez un nombre : " nombre

if [ $nombre -lt 0 ]
then
    echo "Le nombre est négatif"
elif [ $nombre -eq 0 ]
then
    echo "Le nombre est zéro"
else
    echo "Le nombre est positif"
fi
```

---

#### Syntaxe condensée (une ligne)

```bash
if [ ! -e newDir ]; then mkdir newDir; else echo "newDir existe déjà"; fi
```

**Décomposition** :
- `;` sépare les commandes sur une même ligne
- Équivalent à la version sur plusieurs lignes
- Pratique pour de courtes conditions

---

#### Combinaison avec opérateurs

> [!example] if avec opérateurs logiques

```bash
#!/bin/bash

age=$1
permis=$2

if [ $age -ge 18 -a "$permis" = "oui" ]
then
    echo "Vous pouvez conduire"
else
    echo "Vous ne pouvez pas conduire"
fi
```

---

### Structure case

> [!important] Énumérer les cas

**Syntaxe** :
```bash
case valeur in
    valeur1)
        instructions
        ;;
    valeur2 | valeur3)
        instructions
        ;;
    *)
        instructions par défaut
        ;;
esac
```

**Caractéristiques** :
- `|` permet de grouper plusieurs valeurs
- `*)` cas par défaut (comme `default` ou `else`)
- `;;` termine chaque cas
- `esac` (case à l'envers) ferme la structure

---

#### Exemples

> [!example] Menu simple

```bash
wilder@host:~$ choice=1
wilder@host:~$ case $choice in
> 1) 
>     echo "choix 1"
>     echo "Merci"
>     ;;
> 2)
>     echo "choix 2"
>     echo "Bon choix"
>     ;;
> esac
choix 1
Merci
```

---

> [!example] Script avec menu

```bash
#!/bin/bash

echo "=== Menu ==="
echo "1. Afficher la date"
echo "2. Afficher l'utilisateur"
echo "3. Afficher le répertoire"
echo "4. Quitter"
read -p "Votre choix : " choix

case $choix in
    1)
        date
        ;;
    2)
        whoami
        ;;
    3)
        pwd
        ;;
    4)
        echo "Au revoir !"
        exit 0
        ;;
    *)
        echo "Choix invalide"
        exit 1
        ;;
esac
```

---

#### Syntaxe condensée

```bash
case $choice in
    1) echo "choix 1" ;;
    2) echo "choix 2" ;;
    *) echo "autre" ;;
esac
```

---

#### Patterns (motifs)

> [!tip] Utilisation de patterns

```bash
#!/bin/bash

case "$1" in
    start)
        echo "Démarrage du service..."
        ;;
    stop)
        echo "Arrêt du service..."
        ;;
    restart|reload)
        echo "Redémarrage du service..."
        ;;
    status)
        echo "Statut du service..."
        ;;
    -h|--help)
        echo "Usage: $0 {start|stop|restart|status}"
        ;;
    *)
        echo "Commande inconnue : $1" >&2
        exit 1
        ;;
esac
```

---

## Les structures itératives

### Définition boucles

> [!quote] Définition
> En algorithmique, on appelle **structure itérative**, une construction d'un langage qui permet la **répétition d'instructions**.

**C'est-à-dire** :
- Portions de code dont l'exécution va être effectuée un **nombre de fois donné**
- OU **tant qu'une condition est remplie**

**Vocabulaire** :
- On les qualifie couramment de **boucles**
- Chaque exécution = une **itération**

---

### Boucle for

> [!important] Boucler sur une liste

**Syntaxe** :
```bash
for variable in liste
do
    instructions
done
```

**La liste peut être** :
- Une **suite de mots** : `"mot1" "mot2" "mot3"`
- Le résultat d'une **substitution** : `{1..10}`
- Le résultat d'une **commande** : `$(ls)`

---

#### Exemples

> [!example] Boucle sur des mots

```bash
wilder@host:~$ for word in "One" "Two" "Three"
> do
>     echo $word
> done
One
Two
Three
```

---

> [!example] Boucle avec substitution de commande

```bash
wilder@host:~$ for number in $(seq 3 -1 0)
> do
>     echo $number
> done
3
2
1
0
```

**Explication** :
- `seq 3 -1 0` génère la séquence : 3, 2, 1, 0
- `-1` : décrément de 1

---

> [!example] Script : Mon propre ls

```bash
#!/bin/bash
# My own ls

for path in *
do
    echo $path
done
```

**Explication** :
- `*` = tous les fichiers et dossiers du répertoire courant
- La boucle parcourt chaque élément

---

#### Boucle sur les arguments

> [!example] Afficher tous les arguments

```bash
#!/bin/bash

echo "Arguments du script :"

for arg in $*
do
    echo "- $arg"
done
```

**Exécution** :
```bash
./script.sh alpha beta gamma
Arguments du script :
- alpha
- beta
- gamma
```

---

#### Exercice : Arguments numérotés

> [!example] À vous de jouer !

**Objectif** : Créer un script qui affiche la liste de ses arguments :
- Un argument par ligne
- Numéroté de la forme :
  ```
  1 - Argument1
  2 - Argument2
  ...
  ```

---

**Solution** :

```bash
#!/bin/bash

# Echo the numbered list of the script's arguments

number=1
for param in $*
do
    echo "$number - $param"
    number=$(( $number + 1 ))
done

exit 0
```

**Exécution** :
```bash
./script.sh Alice Bob Charlie
1 - Alice
2 - Bob
3 - Charlie
```

---

#### Expansion de séquences

> [!tip] Générer des séquences facilement

**Avec accolades `{}`** :
```bash
# Nombres de 1 à 5
for i in {1..5}
do
    echo $i
done

# Lettres de a à e
for lettre in {a..e}
do
    echo $lettre
done

# Avec un pas (step)
for i in {0..10..2}
do
    echo $i    # 0, 2, 4, 6, 8, 10
done
```

---

### Boucle for arithmétique

> [!important] L'autre syntaxe for (style C)

**Syntaxe** :
```bash
for (( e1 ; e2 ; e3 ))
do
    instructions
done
```

**Où** :
- `e1`, `e2`, `e3` sont des **expressions arithmétiques**
- `e1` : Effectuée **une fois au début** (initialisation)
- `e2` : **Condition** de continuation (tant que vraie)
- `e3` : Effectuée **après chaque tour** (incrémentation)

---

#### Exemples

> [!example] Boucle classique de 1 à 3

```bash
wilder@host:~$ for (( i=1 ; i < 4 ; i++ ))
> do 
>     echo $i
> done
1
2
3
```

**Décomposition** :
- `i=1` : Initialisation
- `i < 4` : Condition (continue tant que i < 4)
- `i++` : Incrémentation (i = i + 1)

---

> [!example] Compte à rebours

```bash
#!/bin/bash

echo "Compte à rebours :"

for (( i=10 ; i >= 0 ; i-- ))
do
    echo $i
    sleep 1
done

echo "Décollage !"
```

---

> [!example] Table de multiplication

```bash
#!/bin/bash

read -p "Table de multiplication de : " nombre

for (( i=1 ; i <= 10 ; i++ ))
do
    resultat=$(( nombre * i ))
    echo "$nombre x $i = $resultat"
done
```

**Exécution** :
```bash
./table.sh
Table de multiplication de : 7
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
...
7 x 10 = 70
```

---

### Boucle while

> [!important] Boucler tant qu'une condition est vraie

**Syntaxe** :
```bash
while condition
do
    instructions
done
```

**Principe** :
- Tant que la **condition est vraie**, les instructions sont exécutées
- La condition est testée **avant** chaque itération

---

#### Exemples

> [!example] Compte à rebours avec while

```bash
wilder@host:~$ number=3
wilder@host:~$ while [ $number -ge 0 ]
> do
>     echo $number
>     number=$(( $number - 1 ))
> done
3
2
1
0
```

**Analyse** :
1. `number=3` : Initialisation
2. `while [ $number -ge 0 ]` : Tant que number >= 0
3. `echo $number` : Afficher
4. `number=$(( $number - 1 ))` : Décrémenter
5. Retour à l'étape 2

---

> [!example] Lecture de fichier ligne par ligne

```bash
#!/bin/bash

fichier="/etc/passwd"

while read ligne
do
    echo "Ligne : $ligne"
done < "$fichier"
```

**Explication** :
- `read ligne` : Lit une ligne dans la variable `ligne`
- `< "$fichier"` : Redirige le contenu du fichier vers la boucle
- Continue tant qu'il y a des lignes

---

> [!example] Menu interactif

```bash
#!/bin/bash

choix=""

while [ "$choix" != "q" ]
do
    echo ""
    echo "=== Menu ==="
    echo "1. Date"
    echo "2. Utilisateur"
    echo "3. Répertoire"
    echo "q. Quitter"
    read -p "Votre choix : " choix
    
    case $choix in
        1) date ;;
        2) whoami ;;
        3) pwd ;;
        q) echo "Au revoir !" ;;
        *) echo "Choix invalide" ;;
    esac
done
```

---

#### Boucle infinie

> [!warning] Attention aux boucles infinies !

**Volontaire (daemon, serveur)** :
```bash
#!/bin/bash

while true
do
    echo "Serveur en écoute..."
    sleep 5
done
```

**Sortie** : Ctrl+C ou `kill`

**Involontaire (bug)** :
```bash
#!/bin/bash

i=1
while [ $i -le 10 ]
do
    echo $i
    # Oubli de l'incrémentation → boucle infinie !
done
```

> [!tip] Déboguer
> Toujours vérifier que :
> - La condition **finira par être fausse**
> - Les variables sont **modifiées** dans la boucle

---

#### Commandes de contrôle

> [!tip] break et continue

**`break`** : Sort de la boucle
```bash
#!/bin/bash

for i in {1..10}
do
    if [ $i -eq 5 ]
    then
        break    # Sort de la boucle
    fi
    echo $i
done
# Affiche : 1 2 3 4
```

**`continue`** : Passe à l'itération suivante
```bash
#!/bin/bash

for i in {1..10}
do
    if [ $i -eq 5 ]
    then
        continue    # Passe à l'itération suivante
    fi
    echo $i
done
# Affiche : 1 2 3 4 6 7 8 9 10 (5 est sauté)
```

---

## Ressources

> [!note] Documentation et références

### Documentation officielle

**Documentation Bash** :
- **GNU Bash Manual** : https://www.gnu.org/software/bash/manual/
- **Bash Reference Manual** : https://www.gnu.org/software/bash/manual/bash.html

### Tutoriels et guides

**Français** :
- **Wikibooks - Programmation Bash** : https://fr.wikibooks.org/wiki/Programmation_Bash
  - Section Tests et conditions
  - Section Boucles

**Anglais** :
- **Bash Hackers Wiki** : https://wiki.bash-hackers.org/
  - Tests and conditionals
  - Loops
- **The Bash Guide (Greg)** : https://mywiki.wooledge.org/BashGuide
- **Advanced Bash-Scripting Guide** : https://tldp.org/LDP/abs/html/

### Outils en ligne

**ExplainShell** :
- URL : https://explainshell.com/
- Explique les commandes Bash ligne par ligne

**ShellCheck** :
- URL : https://www.shellcheck.net/
- Vérification en ligne de scripts

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Tests et conditions

**Convention Bash** :
- **0 = Vrai = Succès** ✅
- **Non-zéro = Faux = Échec** ❌

**Variable `$?`** :
- Code de sortie de la dernière commande

**Syntaxes de test** :
```bash
test expression
[ expression ]       # Équivalent à test
[[ expression ]]     # Recommandé (Bash)
```

---

### Comparaison de chaînes

| Opérateur | Vrai si... |
|-----------|------------|
| `s1 = s2` | Identiques |
| `s1 != s2` | Différentes |
| `-z s1` | Vide |
| `-n s1` | Non vide |

> **Toujours mettre entre guillemets** : `"$var"`

---

### Comparaison de nombres

| Opérateur | Signification |
|-----------|---------------|
| `-eq` | == (égal) |
| `-ne` | != (différent) |
| `-lt` | < (plus petit) |
| `-le` | <= (plus petit ou égal) |
| `-gt` | > (plus grand) |
| `-ge` | >= (plus grand ou égal) |

---

### Opérateurs logiques

| Opérateur | Nom |
|-----------|-----|
| `! c` | NOT (négation) |
| `c1 -a c2` | AND (et) |
| `c1 -o c2` | OR (ou) |

**Avec `[[ ]]`** : `&&`, `||`, `!`

---

### Tests sur fichiers

| Opérateur | Vrai si... |
|-----------|------------|
| `-e p` | Existe |
| `-f p` | Fichier régulier |
| `-d p` | Dossier |
| `-r p` | Lisible |
| `-w p` | Modifiable |
| `-x p` | Exécutable |
| `-s p` | Taille > 0 |

---

### Structure if

```bash
if condition
then
    instructions
elif condition2
then
    instructions
else
    instructions
fi
```

---

### Structure case

```bash
case $variable in
    valeur1)
        instructions
        ;;
    valeur2|valeur3)
        instructions
        ;;
    *)
        instructions par défaut
        ;;
esac
```

---

### Boucle for

**Syntaxe 1** (liste) :
```bash
for variable in liste
do
    instructions
done
```

**Syntaxe 2** (arithmétique) :
```bash
for (( i=0 ; i < 10 ; i++ ))
do
    instructions
done
```

---

### Boucle while

```bash
while condition
do
    instructions
done
```

**Boucle infinie** :
```bash
while true
do
    instructions
done
```

---

### Commandes de contrôle

| Commande | Action |
|----------|--------|
| `break` | Sort de la boucle |
| `continue` | Passe à l'itération suivante |
| `exit N` | Termine le script avec code N |

---

### Exemples types

**Vérifier arguments** :
```bash
if [ $# -eq 0 ]; then
    echo "Usage: $0 <argument>" >&2
    exit 1
fi
```

**Vérifier fichier** :
```bash
if [ ! -f "$fichier" ]; then
    echo "Erreur: Fichier inexistant" >&2
    exit 1
fi
```

**Menu avec case** :
```bash
case $choix in
    1) commande1 ;;
    2) commande2 ;;
    *) echo "Invalide" ;;
esac
```

**Parcourir arguments** :
```bash
for arg in $*
do
    echo "Argument: $arg"
done
```

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Structure conditionnelle** | Construction permettant d'exécuter du code selon une condition |
| **Structure itérative** | Construction permettant de répéter des instructions (boucle) |
| **Test** | Expression évaluée à vrai (0) ou faux (non-zéro) |
| **Code de sortie** | Valeur numérique retournée par une commande (0 = succès) |
| **`$?`** | Variable contenant le code de sortie de la dernière commande |
| **`if`** | Structure conditionnelle (si...alors...sinon) |
| **`elif`** | Else if (sinon si) |
| **`case`** | Structure de choix multiple (switch) |
| **`for`** | Boucle itérant sur une liste |
| **`while`** | Boucle tant qu'une condition est vraie |
| **Itération** | Une exécution d'une boucle |
| **`break`** | Sort de la boucle |
| **`continue`** | Passe à l'itération suivante |
| **`-eq`** | Equal (égal) pour nombres |
| **`-ne`** | Not equal (différent) pour nombres |
| **`-lt`** | Less than (plus petit) |
| **`-gt`** | Greater than (plus grand) |
| **`-le`** | Less or equal (plus petit ou égal) |
| **`-ge`** | Greater or equal (plus grand ou égal) |
| **`-z`** | Zero (chaîne vide) |
| **`-n`** | Non-zero (chaîne non vide) |
| **`-e`** | Exists (chemin existe) |
| **`-f`** | File (est un fichier) |
| **`-d`** | Directory (est un dossier) |
| **`-r`** | Readable (lecture autorisée) |
| **`-w`** | Writable (écriture autorisée) |
| **`-x`** | eXecutable (exécution autorisée) |
| **`!`** | Opérateur NOT (négation) |
| **`-a`** | Opérateur AND (et) |
| **`-o`** | Opérateur OR (ou) |
| **`[[ ]]`** | Test amélioré (extension Bash) |
| **`[ ]`** | Test standard (équivalent à `test`) |
| **`seq`** | Génère une séquence de nombres |
| **`{a..b}`** | Expansion de séquence (a à b) |

---

**Document créé le** : 21 novembre 2025  
**Version** : 1.0  
**Source** : Cours "Les scripts Bash - Partie 2" - Formation TSSR

> [!success] ✅ BON COURAGE POUR VOTRE TITRE RNCP TSSR !

> [!quote] Citation finale
> "La logique de scripting est au cœur de l'automatisation.
> 
> **Maîtrisez les tests et les boucles pour devenir efficace !**"

---
