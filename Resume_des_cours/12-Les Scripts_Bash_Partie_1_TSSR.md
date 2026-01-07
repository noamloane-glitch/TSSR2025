# Les scripts Bash - Partie 1
## Document de révision TSSR - Titre RNCP

---

**Formation** : Technicien Supérieur Systèmes et Réseaux (TSSR)  
**Sujet** : Les scripts Bash - Les fondations  
**Date** : Novembre 2025  
**Type** : Synthèse de cours complète

---

## 📋 Sommaire

1. [[#Introduction|Introduction]]
2. [[#Définition et concepts de base|Définition et concepts de base]]
   - [[#Qu'est-ce qu'un script ?|Qu'est-ce qu'un script ?]]
   - [[#Script vs Programme|Script vs Programme]]
   - [[#Objectifs des scripts|Objectifs des scripts]]
   - [[#Les shells UNIX|Les shells UNIX]]
3. [[#Créer son premier script|Créer son premier script]]
   - [[#Script basique|Script basique]]
   - [[#Bonnes pratiques|Bonnes pratiques]]
   - [[#Le shebang|Le shebang]]
   - [[#Code de sortie|Code de sortie]]
4. [[#Les fondamentaux du shell|Les fondamentaux du shell]]
   - [[#Parser des lignes|Parser des lignes]]
   - [[#Les métacaractères|Les métacaractères]]
   - [[#Le quoting|Le quoting]]
5. [[#Les variables|Les variables]]
   - [[#Utilisation des variables|Utilisation des variables]]
   - [[#Arguments et paramètres|Arguments et paramètres]]
   - [[#Substitution de commandes|Substitution de commandes]]
   - [[#Substitution arithmétique|Substitution arithmétique]]
   - [[#Variables spéciales|Variables spéciales]]
   - [[#Environnement et portée|Environnement et portée]]
6. [[#Points clés à retenir|Points clés à retenir]]
7. [[#Glossaire technique|Glossaire technique]]
8. [[#Références|Références]]

---

## Introduction

> [!abstract] Vue d'ensemble
> Les scripts Bash sont des outils essentiels pour tout administrateur système. Cette première partie couvre les fondations nécessaires pour comprendre et créer des scripts shell efficaces et professionnels.

### Pourquoi étudier les scripts Bash ?

En tant que **TSSR**, tu dois :
- **Automatiser** les tâches répétitives
- **Gagner en efficacité** dans l'administration système
- **Fiabiliser** les opérations par des contrôles systématiques
- **Documenter** tes procédures pour d'autres administrateurs
- **Réagir rapidement** grâce à des scripts préparés à l'avance

Les scripts Bash sont au cœur de l'administration Linux/UNIX et constituent une compétence fondamentale pour le titre RNCP TSSR.

---

## Définition et concepts de base

### Qu'est-ce qu'un script ?

> [!quote] Définition
> Un script est un **fichier texte** contenant du code écrit dans un langage donné, qui peut être **exécuté via un interprète** sans compilation préalable.

> [!info] Caractéristiques d'un script shell
> Pour les scripts shell, l'interpréteur utilisé est un **interpréteur de commande** (CLI - Command Line Interface). L'interpréteur lit et exécute le script **ligne par ligne** dans un environnement non-compilé.

**Caractéristiques principales** :
- Fichier texte simple (éditable avec n'importe quel éditeur)
- Contient des commandes interprétables par le shell
- Exécution séquentielle (ligne après ligne)
- Pas de compilation nécessaire
- Modification immédiate sans recompilation

### Script vs Programme

> [!important] Distinction fondamentale
> On distingue généralement deux types d'exécutables via des commandes interprétées par le shell :

| Type | Description | Interaction |
|------|-------------|-------------|
| **Programme** | Code compilé qui indique au **processeur** ce qu'il doit faire | Interaction directe avec le matériel |
| **Script** | Code interprété qui indique à un **OS ou une application** ce qu'ils doivent faire | Interaction via des commandes système |

**En pratique** :
- Un programme est compilé en langage machine
- Un script reste en texte clair et est interprété à chaque exécution
- Les scripts utilisent des commandes et programmes existants

### Objectifs des scripts

> [!success] Pourquoi scripter ?
> Les scripts simplifient la vie de l'administrateur système !

**Avantages des scripts** :

1. **Reproduction rapide et simple** de suites d'actions répétitives
2. **Anticipation** d'actions futures pour une meilleure réactivité (et être plus serein !)
3. **Fiabilité** : tous les contrôles nécessaires sont effectués systématiquement
4. **Documentation** : un autre admin peut consulter et comprendre les scripts
5. **Automatisation** : déclenchements automatiques via cron, systemd, etc.

> [!example] Exemples d'utilisation
> - Sauvegarde automatique quotidienne de bases de données
> - Création d'utilisateurs avec configuration standardisée
> - Surveillance de services et redémarrage automatique
> - Génération de rapports système périodiques
> - Déploiement d'applications ou de configurations

### Les shells UNIX

> [!info] The UNIX way
> Le shell standard d'UNIX est **sh**. Bash (et les autres shells) sont **compatibles avec sh**.

**Implications pratiques** :
- On peut écrire des scripts `sh` et les faire exécuter par `bash` (ou un autre shell)
- Deux approches possibles :
  1. **Scripts standard** : fonctionnent sur n'importe quel shell compatible POSIX
  2. **Scripts spécifiques** : utilisent les fonctionnalités avancées d'un shell particulier (comme bash)

> [!tip] Conseil pour la compatibilité
> Pour une portabilité maximale, privilégie les fonctionnalités standard `sh`. Pour des fonctionnalités avancées (tableaux, substitutions complexes), utilise explicitement `bash` via le shebang.

**Shells courants** :
- `sh` : Shell standard POSIX
- `bash` : Bourne Again Shell (le plus courant sur Linux)
- `zsh` : Z Shell (shell moderne avec de nombreuses fonctionnalités)
- `dash` : Debian Almquist Shell (rapide, souvent utilisé pour `/bin/sh`)
- `ksh` : Korn Shell

---

## Créer son premier script

### Script basique

> [!example] Hello World - Méthode 1 : avec éditeur
> Créer un fichier et y insérer du code.

```bash
# Création du fichier
wilder@host:~$ touch coucou
wilder@host:~$ nano coucou

# Contenu du fichier :
echo "Hello World !"

# Enregistrer et quitter l'éditeur

# Exécution du script
wilder@host:~$ bash coucou
Hello World !

# Vérification du contenu
wilder@host:~$ cat coucou
echo "Hello World !"
```

> [!example] Hello World - Méthode 2 : tout en CLI
> Création et édition en une seule commande.

```bash
wilder@host:~$ echo 'echo "Hello World !"' > coucou
wilder@host:~$ bash coucou
Hello World !
wilder@host:~$ cat coucou
echo "Hello World !"
```

### Bonnes pratiques

> [!tip] Conventions et recommandations

**Conventions de nommage** :
- Suffixer les noms de scripts par `.sh` (fréquent mais pas essentiel)
- Choisir des noms explicites et descriptifs

**Permissions** :
- Accorder les droits d'exécution aux scripts : `chmod u+x script.sh`
- Vérifier les permissions avec `ls -l`

**Documentation** :
- Utiliser les **commentaires** pour expliquer le code
- Bash ignore les lignes débutant par `#`
- Commenter les parties complexes et les choix techniques

> [!note] Ressources recommandées
> Quelques références pour aller plus loin :
> - **Greg's wiki** : guide de référence pour le scripting bash
> - **Bash Hackers wiki** : documentation technique approfondie

### Le shebang

> [!quote] Définition du shebang
> Sur les systèmes d'exploitation de type Unix, le **shebang** est une convention pour les scripts (fichiers texte "exécutables"). C'est la **première ligne** du script qui indique l'interpréteur à utiliser.

**Syntaxe** : `#!<chemin de l'interpréteur>`

**Exemples courants** :

```bash
#!/bin/bash
# Spécifie bash de manière absolue (Debian/Ubuntu)

#!/usr/bin/env bash
# Approche plus généraliste : cherche bash dans le PATH
# Recommandé pour la portabilité entre différents systèmes
```

> [!info] Pourquoi "shebang" ?
> Le nom vient de la contraction de "**sh**arp" (#) et "**bang**" (!) - voir l'article Wikipedia pour l'historique complet.

> [!important] Différence entre les deux approches
> - `#!/bin/bash` : chemin absolu, fonctionnera si bash est à cet emplacement exact
> - `#!/usr/bin/env bash` : recherche bash dans le PATH, plus portable entre systèmes

### Code de sortie

> [!quote] Terminer en beauté
> Toute commande Unix est censée se terminer en fournissant un **code de sortie**. Ce code est une valeur numérique entre **0 et 255** qui indique au processus exécuteur la raison de sa fin.

> [!important] Concept clé
> Un script shell est une "**commande composite**" : il doit donc fournir un code de sortie au shell qui l'a invoqué. La commande `exit` permet de préciser cette valeur.

**Syntaxe** : `exit <code>`

#### Codes de sortie courants

| N° de code | Signification |
|------------|---------------|
| **0** | Sortie normale, tout va bien ✅ |
| **1** | Erreur générale |
| **2** | Mauvaise syntaxe de commande |
| **126** | Pas de droit d'exécution sur un fichier existant |
| **127** | Commande introuvable |
| **130** | Script terminé par Ctrl+C |
| **255** | Code de sortie hors limites |

> [!tip] Bonne pratique
> Termine toujours tes scripts avec `exit 0` si tout s'est bien passé, ou un code d'erreur approprié si un problème est survenu.

> [!note] Vérifier le code de sortie
> La variable spéciale `$?` contient le code de sortie de la dernière commande exécutée.

```bash
wilder@host:~$ ls /tmp
# ... affichage du contenu ...
wilder@host:~$ echo $?
0

wilder@host:~$ ls /dossier_inexistant
ls: cannot access '/dossier_inexistant': No such file or directory
wilder@host:~$ echo $?
2
```

### Script complet et conforme

> [!success] Hello World dans les règles de l'art

**Objectif** : Créer un script professionnel avec toutes les bonnes pratiques.

**Étapes** :
1. Créer un fichier `coucou.sh`
2. Ajouter le shebang en première ligne
3. Faire afficher "Hello World !"
4. Terminer en renvoyant le code 0
5. Rendre le script exécutable
6. L'exécuter directement

> [!example] Méthode avec éditeur

```bash
wilder@host:~$ touch coucou.sh
wilder@host:~$ nano coucou.sh

# Contenu du fichier :
#!/bin/bash
echo "Hello World !"
exit 0

# Enregistrer et quitter l'éditeur

wilder@host:~$ chmod u+x coucou.sh
wilder@host:~$ ./coucou.sh
Hello World !
```

> [!example] Méthode tout en CLI

```bash
wilder@host:~$ echo '#!/bin/bash' > coucou.sh
wilder@host:~$ echo 'echo "Hello World !"' >> coucou.sh
wilder@host:~$ echo 'exit 0' >> coucou.sh
wilder@host:~$ chmod u+x coucou.sh
wilder@host:~$ ./coucou.sh
Hello World !

# Vérification des permissions et du contenu
wilder@host:~$ ls -l coucou.sh
-rwxrw-r-- 1 wilder wilder 65 mai 17 12:00 coucou.sh

wilder@host:~$ cat coucou.sh
#!/bin/bash
echo "Hello World !"
exit 0
```

> [!note] Différence d'exécution
> - `bash coucou.sh` : lance explicitement bash pour exécuter le script
> - `./coucou.sh` : utilise le shebang pour déterminer l'interpréteur, nécessite les droits d'exécution

---

## Les fondamentaux du shell

### Parser des lignes

> [!info] Back to basics - Comment fonctionne le shell ?

Le shell lit un **flux de caractères** provenant de deux sources :
- **Mode interactif** : caractères entrés au clavier
- **Mode script** : caractères lus dans un fichier

**Processus d'analyse pour chaque ligne** :

1. **Analyse lexicale** : reconnaître les mots
2. **Analyse syntaxique** : reconnaître les phrases (structure des commandes)
3. **Exécution** : exécuter la ou les commandes

> [!important] Fin de ligne
> Une ligne se termine par le caractère **newline** (obtenu via la touche ⏎ Entrée).

**Exemple de parsing** :

```bash
ls -l /home
```

Le shell identifie :
- **Commande** : `ls`
- **Options** : `-l`
- **Arguments** : `/home`

### Les métacaractères

> [!quote] Définition
> Un **métacaractère** est un caractère qui a une signification particulière pour le shell. Il sert de **séparateur de mots** ou définit des opérations spéciales.

#### Types de métacaractères

**Les "blancs" (whitespace)** :
- Espace : ` `
- Tabulation : `⇆`

**Les délimiteurs** :
- Fin de ligne : newline `⏎`

**Les opérateurs de contrôle** :
- `|` : pipe (tube)
- `||` : OU logique
- `&` : exécution en arrière-plan
- `&&` : ET logique
- `;` : séparateur de commandes
- `()` : sous-shell
- `<` `>` : redirections
- `#` : commentaire
- `"` `'` : délimiteurs de chaînes

> [!info] Fonction des opérateurs
> Ces métacaractères permettent de créer :
> - Des **séquences de commandes**
> - Des **redirections** d'entrée/sortie
> - Des **pipelines** (chaînes de commandes)
> - Des **structures de contrôle**

**Le caractère d'échappement** : `\`
- Supprime la fonction particulière du caractère suivant
- Permet de traiter un métacaractère comme un caractère ordinaire

> [!example] Exemples de métacaractères

```bash
# Séparateur de commandes avec ;
wilder@host:~$ echo "Première commande" ; echo "Deuxième commande"
Première commande
Deuxième commande

# Pipeline avec |
wilder@host:~$ ls -l | grep ".sh"
-rwxrw-r-- 1 wilder wilder 65 mai 17 12:00 coucou.sh

# ET logique avec &&
wilder@host:~$ mkdir test && cd test
# cd s'exécute seulement si mkdir réussit

# Échappement avec \
wilder@host:~$ echo "Prix : 10\$"
Prix : 10$
```

### Le quoting

> [!important] Protéger les caractères spéciaux
> Le **quoting** (mise entre guillemets) permet de contrôler l'interprétation des métacaractères par le shell.

#### Types de quotes

**1. Les guillemets simples `'...'` (single quotes)**

> [!quote] Guillemets simples
> Tous les caractères sont interprétés **littéralement**. Aucune expansion n'est effectuée.

```bash
wilder@host:~$ var="monde"
wilder@host:~$ echo 'Hello $var !'
Hello $var !
# $var n'est PAS substitué
```

**2. Les guillemets doubles `"..."` (double quotes)**

> [!quote] Guillemets doubles
> Permettent l'**expansion des variables** et la **substitution de commandes**, mais protègent des autres métacaractères.

```bash
wilder@host:~$ var="monde"
wilder@host:~$ echo "Hello $var !"
Hello monde !
# $var est substitué par sa valeur
```

**3. Le backslash `\` (échappement)**

> [!quote] Échappement
> Annule la signification spéciale du caractère **suivant uniquement**.

```bash
wilder@host:~$ echo "Prix : 15\$"
Prix : 15$

wilder@host:~$ echo "Ligne 1\nLigne 2"
Ligne 1
Ligne 2
```

#### Tableau récapitulatif du quoting

| Type | Syntaxe | Expansion variables | Expansion commandes | Métacaractères protégés |
|------|---------|---------------------|---------------------|-------------------------|
| **Simple quotes** | `'texte'` | ❌ Non | ❌ Non | ✅ Tous |
| **Double quotes** | `"texte"` | ✅ Oui | ✅ Oui | ✅ Partiels (espace, *, etc.) |
| **Backslash** | `\c` | ➖ N/A | ➖ N/A | ✅ Caractère suivant uniquement |

> [!warning] Piège courant
> Ne pas confondre les guillemets simples et doubles ! Les simples quotes bloquent TOUTE expansion, y compris les variables.

> [!example] Cas d'usage pratiques

```bash
# Nom de fichier avec espaces
wilder@host:~$ touch "mon fichier.txt"
# ou
wilder@host:~$ touch mon\ fichier.txt

# Variable contenant des espaces
wilder@host:~$ message="Bonjour tout le monde"
wilder@host:~$ echo "$message"
Bonjour tout le monde

# Commande avec caractères spéciaux
wilder@host:~$ echo 'Prix: $10 & taxes'
Prix: $10 & taxes
```

> [!tip] Mnémotechnique
> - **Simple quotes** `'` : **S**trict - tout est littéral
> - **Double quotes** `"` : **D**ynamique - permet les expansions
> - **Backslash** `\` : échappe le caractère **suivant**

---

## Les variables

### Utilisation des variables

> [!quote] Définition
> Une **variable** est un conteneur qui stocke une valeur (données) que l'on peut réutiliser et modifier dans le script.

#### Syntaxe de base

**Déclaration et affectation** :

```bash
nomVariable=valeur
```

> [!warning] Attention aux espaces !
> Il ne doit y avoir **AUCUN ESPACE** autour du signe `=`. C'est une erreur très courante !

```bash
# ✅ CORRECT
variable="valeur"

# ❌ INCORRECT
variable = "valeur"    # Sera interprété comme une commande !
```

**Accès à la valeur** :

```bash
$nomVariable
# ou avec accolades (recommandé pour plus de clarté)
${nomVariable}
```

**Type par défaut** :
- Par défaut, toutes les variables sont des **chaînes de caractères** (strings)
- Même les nombres sont traités comme du texte (sauf dans les calculs arithmétiques)

**Destruction d'une variable** :

```bash
unset nomVariable
```

> [!example] Exemples pratiques

```bash
# Déclaration et utilisation simple
wilder@host:~$ greetings="Coucou"
wilder@host:~$ echo "$greetings"
Coucou

# Modification de valeur
wilder@host:~$ greetings="Bonjour"
wilder@host:~$ echo "$greetings"
Bonjour

# Utilisation dans une commande
wilder@host:~$ myDirectory="MonDossier"
wilder@host:~$ mkdir $myDirectory
wilder@host:~$ ls | grep "Mon"
MonDossier

# Suppression de variable
wilder@host:~$ unset greetings
wilder@host:~$ echo "$greetings"
# Affiche une ligne vide (variable vide)
```

> [!tip] Bonnes pratiques de nommage
> - Utilise des noms **descriptifs** et en **camelCase** ou **snake_case**
> - Évite les noms trop courts (sauf pour les compteurs : `i`, `j`)
> - Les variables d'environnement sont généralement en **MAJUSCULES**
> - Les variables locales en **minuscules** ou **mixtes**

> [!example] Exercice : variable contenant une commande

**Objectif** :
1. Déclarer une variable `commande` ayant pour valeur `'whoami'`
2. Afficher la valeur de la variable
3. Exécuter `whoami` en utilisant la variable

```bash
wilder@host:~$ commande='whoami'
wilder@host:~$ echo $commande
whoami
wilder@host:~$ $commande
wilder
```

> [!note] Explication
> Lorsqu'on écrit `$commande`, le shell remplace la variable par sa valeur (`whoami`), puis exécute cette commande.

### Arguments et paramètres

> [!quote] Définition
> Les **arguments** sont des valeurs passées à un script lors de son exécution. Ils permettent de donner des données au script sans modifier son code source.

#### Concept fondamental

**Avantage** : Cela permet d'exécuter plusieurs fois un script avec des **données différentes** sans avoir à le modifier.

**Dans le script** : Les arguments deviennent des **paramètres** récupérables via des variables spéciales : `$1`, `$2`, `$3`, etc.

#### Distinction argument vs paramètre

> [!important] Différence subtile mais importante

| Terme | Localisation | Définition |
|-------|--------------|------------|
| **Argument** | En dehors du script (ligne de commande) | Valeur **réelle** passée au script lors de son exécution |
| **Paramètre** | À l'intérieur du script | Nom (variable) utilisé dans le script pour **recevoir** une valeur |

**Analogie** : C'est comme une fonction en programmation - les arguments sont les valeurs passées, les paramètres sont les variables qui les reçoivent.

#### Accès aux paramètres

```bash
$0  # Nom du script lui-même
$1  # Premier argument
$2  # Deuxième argument
$3  # Troisième argument
# ... et ainsi de suite
```

> [!example] Script avec paramètres

**Contenu du script** :

```bash
#!/bin/bash

echo "Bonjour $1 !"
echo "Je cherche $2, l'as-tu vu ?"
exit 0
```

**Exécution sans arguments** :

```bash
wilder@host:~$ ./script.sh
Bonjour  !
Je cherche , l'as-tu vu ?
```

**Exécution avec arguments** :

```bash
wilder@host:~$ ./script.sh Bob Alice
Bonjour Bob !
Je cherche Alice, l'as-tu vu ?
```

> [!note] Analyse
> - `Bob` est passé en premier argument et devient `$1`
> - `Alice` est passée en deuxième argument et devient `$2`
> - Sans arguments, les variables `$1` et `$2` sont vides

> [!example] Exercice : notify-send

**Contexte** : `notify-send` sert à envoyer une notification desktop.

**Objectif** :
1. Consulter l'aide avec `notify-send --help`
2. Déclarer une variable `notify` avec la valeur `notify-send`
3. Afficher une notification avec résumé 'Plop' et texte 'Message envoyé via notify-send'

```bash
wilder@host:~$ notify='notify-send'
wilder@host:~$ $notify Plop "Message envoyé via $notify"
# Une notification apparaît sur le bureau
```

> [!tip] Utilité des variables pour les commandes
> Stocker des commandes dans des variables permet de :
> - Centraliser les chemins d'outils
> - Faciliter les modifications (un seul endroit à changer)
> - Améliorer la lisibilité du code

### Substitution de commandes

> [!quote] Définition
> La **substitution de commandes** permet de **récupérer le résultat** (sortie standard) d'une commande au lieu de simplement l'afficher.

#### Syntaxe

```bash
$(commande)
```

> [!note] Ancienne syntaxe
> L'ancienne syntaxe avec backticks `` `commande` `` existe mais est **déconseillée**. Utilise toujours `$(commande)` pour plus de clarté et de facilité d'imbrication.

#### Utilisations courantes

1. **Stocker le résultat dans une variable**

```bash
wilder@host:~$ id -u
1000
wilder@host:~$ myUID=$(id -u)
wilder@host:~$ echo $myUID
1000
```

2. **Utiliser directement dans une commande**

```bash
wilder@host:~$ echo "Mon UID est : $(id -u)"
Mon UID est : 1000

wilder@host:~$ echo "Nous sommes le $(date +%d/%m/%Y)"
Nous sommes le 21/11/2025
```

3. **Imbrication de substitutions**

```bash
wilder@host:~$ echo "Il y a $(ls $(pwd) | wc -l) fichiers ici"
```

> [!example] Cas pratiques

```bash
# Sauvegarder avec horodatage
wilder@host:~$ backup_name="backup_$(date +%Y%m%d_%H%M%S).tar.gz"
wilder@host:~$ echo $backup_name
backup_20251121_143022.tar.gz

# Compter les utilisateurs connectés
wilder@host:~$ nb_users=$(who | wc -l)
wilder@host:~$ echo "Il y a $nb_users utilisateur(s) connecté(s)"

# Récupérer le hostname
wilder@host:~$ machine=$(hostname)
wilder@host:~$ echo "Vous êtes sur la machine : $machine"
```

> [!tip] Bonne pratique
> Toujours mettre les substitutions de commandes entre guillemets doubles quand tu les utilises dans d'autres commandes, pour gérer correctement les espaces : `"$(commande)"`

### Substitution arithmétique

> [!quote] Définition
> La **substitution arithmétique** permet d'effectuer des **calculs mathématiques** directement dans le shell.

#### Syntaxe

```bash
$(( expression ))
```

#### Opérateurs supportés

| Opérateur | Opération |
|-----------|-----------|
| `+` | Addition |
| `-` | Soustraction |
| `*` | Multiplication |
| `/` | Division entière |
| `%` | Modulo (reste de la division) |
| `**` | Puissance |

> [!warning] Division entière uniquement
> Bash ne gère que les **entiers** dans les calculs arithmétiques. Pour des calculs avec décimales, il faut utiliser `bc` ou `awk`.

> [!example] Exemples de calculs

```bash
# Calcul simple
wilder@host:~$ echo $(( 12 * 6 ))
72

# Avec variables
wilder@host:~$ total=$(( 7 + 3 ))
wilder@host:~$ echo $(( $total * 2 + 1 ))
21

# Ordre des opérations respecté
wilder@host:~$ echo $(( 10 + 5 * 2 ))
20
# 5*2 = 10, puis 10+10 = 20

# Avec parenthèses
wilder@host:~$ echo $(( (10 + 5) * 2 ))
30
# (10+5) = 15, puis 15*2 = 30
```

> [!example] Cas pratiques

```bash
# Incrémenter un compteur
wilder@host:~$ counter=5
wilder@host:~$ counter=$(( counter + 1 ))
wilder@host:~$ echo $counter
6

# Calculer un pourcentage (attention : entiers uniquement)
wilder@host:~$ total=100
wilder@host:~$ reussi=75
wilder@host:~$ pourcentage=$(( reussi * 100 / total ))
wilder@host:~$ echo "$pourcentage%"
75%

# Calcul de capacité disque
wilder@host:~$ ko=1024
wilder@host:~$ mo=$(( ko * 1024 ))
wilder@host:~$ go=$(( mo * 1024 ))
wilder@host:~$ echo "1 Go = $go octets"
1 Go = 1073741824 octets
```

> [!tip] Calculs avec décimales
> Pour des calculs avec virgule flottante, utilise `bc` :

```bash
wilder@host:~$ echo "scale=2; 10 / 3" | bc
3.33
```

### Variables spéciales

> [!important] Variables prédéfinies par le shell
> Le shell Bash fournit un ensemble de **variables spéciales** automatiquement définies, très utiles pour gérer les scripts.

#### Liste des variables spéciales essentielles

| Variable | Description |
|----------|-------------|
| `$0` | Nom du script invoqué |
| `$1`, `$2`, `$3`... | Arguments positionnels (premier, deuxième, troisième argument, etc.) |
| `$#` | Nombre total d'arguments passés au script |
| `$*` | Tous les arguments du script en **un seul mot** |
| `$@` | Tous les arguments du script en **mots séparés** |
| `$?` | Code de sortie de la **dernière commande** exécutée |
| `$$` | **PID** (Process ID) du shell courant |
| `$!` | PID du dernier job lancé en **arrière-plan** |

> [!example] Utilisation de `$0` - Nom du script

```bash
#!/bin/bash
echo "Ce script s'appelle : $0"
exit 0
```

```bash
wilder@host:~$ ./monscript.sh
Ce script s'appelle : ./monscript.sh
```

> [!example] Utilisation de `$#` - Nombre d'arguments

```bash
#!/bin/bash
echo "Nombre d'arguments reçus : $#"
echo "Arguments : $@"
exit 0
```

```bash
wilder@host:~$ ./script.sh un deux trois
Nombre d'arguments reçus : 3
Arguments : un deux trois
```

> [!example] Utilisation de `$?` - Code de sortie

```bash
wilder@host:~$ ls /tmp
# ... liste des fichiers ...
wilder@host:~$ echo $?
0
# Commande réussie

wilder@host:~$ ls /dossier_inexistant
ls: cannot access '/dossier_inexistant': No such file or directory
wilder@host:~$ echo $?
2
# Commande échouée
```

> [!example] Utilisation de `$$` et `$!` - Process IDs

```bash
#!/bin/bash
echo "PID de ce script : $$"

# Lancer une commande en arrière-plan
sleep 60 &
echo "PID du sleep en arrière-plan : $!"

exit 0
```

```bash
wilder@host:~$ ./script.sh
PID de ce script : 12345
PID du sleep en arrière-plan : 12346
```

#### Différence entre `$*` et `$@`

> [!important] Subtilité importante entre `$*` et `$@`

**Sans guillemets** : `$*` et `$@` se comportent de la même manière.

**Avec guillemets** :
- `"$*"` : tous les arguments en **UN SEUL mot** (séparés par le premier caractère de `IFS`, généralement un espace)
- `"$@"` : chaque argument reste un **mot séparé**

```bash
#!/bin/bash
echo "Avec \$* :"
for arg in "$*"; do
    echo "  - $arg"
done

echo "Avec \$@ :"
for arg in "$@"; do
    echo "  - $arg"
done
exit 0
```

```bash
wilder@host:~$ ./script.sh un deux trois
Avec $* :
  - un deux trois

Avec $@ :
  - un
  - deux
  - trois
```

> [!tip] Recommandation
> Utilise `"$@"` quand tu veux préserver les arguments individuels (cas le plus fréquent), et `"$*"` quand tu veux les combiner en une seule chaîne.

### Environnement et portée

#### Shell et variables - Hiérarchie

> [!info] Concept de shell parent/fils
> Lorsqu'on exécute un script via un shell :
> - Le script est exécuté dans un **shell fils** du shell courant
> - Ce shell fils se **termine** à la fin du script
> - Les variables du shell parent ne sont **pas automatiquement** accessibles dans le shell fils

> [!warning] Isolation des variables
> Par défaut, les variables déclarées dans un shell **ne sont pas accessibles** dans les autres shells (même les shells fils).

**Exemple** : Variables locales au shell

```bash
wilder@host:~$ maVariable="valeur"
wilder@host:~$ echo $maVariable
valeur
wilder@host:~$ bash              # Lancement d'un sous-shell
wilder@host:~$ echo $maVariable
                                 # Variable vide !
wilder@host:~$ exit              # Retour au shell parent
wilder@host:~$ echo $maVariable
valeur                           # Variable toujours présente
```

#### Exécution dans le shell courant - source

> [!important] La commande `source`
> Pour exécuter un script dans le **shell courant** (sans créer de shell fils), utilise : `source <script.sh>` ou `. <script.sh>`

**Conséquence** : Les modifications de variables dans le script **affectent** le shell courant.

```bash
# Contenu de script.sh :
#!/bin/bash
variable=modified

# Dans le shell :
wilder@host:~$ variable=initial
wilder@host:~$ echo $variable
initial

wilder@host:~$ source ./script.sh
wilder@host:~$ echo $variable
modified    # La variable a été modifiée !
```

#### Variables d'environnement - export

> [!quote] Définition
> Les **variables d'environnement** sont des variables spéciales qui sont **copiées** dans chaque shell fils.

**Commande** : `export <variable>`

**Effet** : La variable devient disponible pour tous les processus fils (sous-shells, scripts).

> [!example] Utilisation de `export`

```bash
# Contenu de script.sh :
#!/bin/bash
echo $variable
variable=modified
echo $variable
exit 0

# Test sans export
wilder@host:~$ variable=initial
wilder@host:~$ echo $variable
initial
wilder@host:~$ ./script.sh
                # Variable vide dans le script
modified
wilder@host:~$ echo $variable
initial         # Variable inchangée dans le shell parent
```

**Maintenant avec `export`** :

```bash
wilder@host:~$ variable=initial
wilder@host:~$ export variable
wilder@host:~$ ./script.sh
initial         # Variable accessible dans le script !
modified
wilder@host:~$ echo $variable
initial         # Mais toujours inchangée dans le parent
```

> [!note] Variables d'environnement courantes
> Exemples de variables d'environnement importantes :
> - `PATH` : chemins de recherche des exécutables
> - `HOME` : répertoire personnel de l'utilisateur
> - `USER` : nom de l'utilisateur
> - `PWD` : répertoire de travail actuel
> - `SHELL` : shell par défaut

**Lister les variables d'environnement** :

```bash
wilder@host:~$ env
# Affiche toutes les variables d'environnement
```

> [!example] Exercices pratiques sur les variables et l'environnement

**Exercice 1** : Comportement par défaut

```bash
# Contenu de script.sh :
#!/bin/bash
echo $variable
variable=modified
echo $variable
exit 0

# Exécution
wilder@host:~$ variable=initial
wilder@host:~$ echo $variable
initial
wilder@host:~$ ./script.sh
                # Ligne vide (variable non accessible)
modified
wilder@host:~$ echo $variable
initial         # Variable inchangée
```

**Exercice 2** : Avec export

```bash
wilder@host:~$ variable=initial
wilder@host:~$ export variable
wilder@host:~$ ./script.sh
initial         # Variable héritée
modified
wilder@host:~$ echo $variable
initial         # Toujours inchangée dans le parent

# Dans un autre shell distinct :
wilder@host:~$ echo $variable
                # Vide (pas hérité par les shells non-fils)
```

**Exercice 3** : Avec source (sans exit)

```bash
# Contenu de scriptnoexit.sh (même script sans exit 0)
#!/bin/bash
echo $variable
variable=modified
echo $variable

# Exécution avec source
wilder@host:~$ variable=initial
wilder@host:~$ echo $variable
initial
wilder@host:~$ source ./scriptnoexit.sh
initial
modified
wilder@host:~$ echo $variable
modified        # Variable modifiée dans le shell courant !
```

#### Synthèse sur les variables

> [!success] Récapitulatif des comportements

| Méthode d'exécution | Effet sur les variables | Quand l'utiliser ? |
|---------------------|-------------------------|-------------------|
| `./script.sh` | Exécution dans un shell fils - isolation complète | Exécution normale d'un script |
| `source script.sh` | Exécution dans le shell courant - modifie les variables du shell | Configuration d'environnement (`.bashrc`, `.profile`) |
| `export variable` | Rend la variable accessible aux shells fils | Partager des variables avec les processus enfants |

**Règles à retenir** :

1. **`source <script.sh>`** :
   - Exécution du script dans le **shell courant**
   - La modification des variables dans le script **affecte** le shell courant
   - Le script **agit sur** le shell courant

2. **`export <variable>`** :
   - La variable est disponible dans l'**environnement** du shell courant
   - Un script exécuté dans ce shell **hérite** de cette variable
   - Le shell parent **prépare** les variables pour les shells enfants

> [!tip] Mnémotechnique
> - **source** : le script **modifie** mon shell
> - **export** : je **partage** mes variables avec mes enfants
> - **normal** : chacun travaille dans son **coin**

---

## Points clés à retenir

> [!success] Synthèse pour le titre RNCP

### Définition et objectifs

- Un **script** est un fichier texte contenant des commandes interprétées ligne par ligne
- Différence : **Programme** (processeur) vs **Script** (OS/application)
- **Objectifs** : Automatisation, fiabilité, documentation, gain de temps
- **Bash** est compatible avec le shell standard `sh`

### Structure d'un script professionnel

- **Shebang** : `#!/bin/bash` ou `#!/usr/bin/env bash` en première ligne
- **Commentaires** : lignes commençant par `#` pour documenter
- **Permissions** : `chmod u+x script.sh` pour rendre exécutable
- **Code de sortie** : `exit 0` pour succès, autre valeur pour erreur
- **Extension** : `.sh` recommandée mais non obligatoire

### Fondamentaux du shell

- Le shell **parse** les lignes : analyse lexicale, syntaxique, puis exécution
- **Métacaractères** : caractères spéciaux (|, &, ;, <, >, etc.) avec significations particulières
- **Quoting** :
  - `'...'` : protection totale (aucune expansion)
  - `"..."` : permet expansion des variables et substitutions
  - `\` : échappe le caractère suivant uniquement

### Variables

- **Déclaration** : `variable=valeur` (SANS ESPACES !)
- **Utilisation** : `$variable` ou `${variable}`
- **Type** : par défaut string
- **Suppression** : `unset variable`

### Paramètres et arguments

- **Arguments** : valeurs passées en ligne de commande
- **Paramètres** : `$1`, `$2`, `$3`, etc. dans le script
- `$0` : nom du script
- `$#` : nombre d'arguments

### Substitutions

- **Commandes** : `$(commande)` pour récupérer le résultat
- **Arithmétique** : `$(( expression ))` pour les calculs (entiers uniquement)
- Permet de stocker des résultats dans des variables

### Variables spéciales

- `$?` : code de sortie de la dernière commande
- `$$` : PID du shell courant
- `$!` : PID du dernier job en arrière-plan
- `$@` : tous les arguments (séparés)
- `$*` : tous les arguments (en un mot)

### Environnement et portée

- **Par défaut** : variables isolées dans chaque shell
- **`export`** : rend une variable accessible aux shells fils
- **`source`** : exécute un script dans le shell courant (modifie les variables)
- **`env`** : liste les variables d'environnement

### Codes de sortie essentiels

- `0` : Succès ✅
- `1` : Erreur générale
- `2` : Erreur de syntaxe
- `126` : Pas de droits d'exécution

---

## Glossaire technique

> [!note] Définitions essentielles pour le TSSR

| Terme | Définition |
|-------|------------|
| **Script** | Fichier texte contenant des commandes exécutables par un interprète |
| **Shell** | Interpréteur de commandes (interface entre utilisateur et système) |
| **Bash** | Bourne Again Shell - shell le plus répandu sur Linux |
| **Shebang** | Ligne `#!/chemin/interpréteur` indiquant quel interpréteur utiliser |
| **Métacaractère** | Caractère ayant une signification spéciale pour le shell (|, &, ;, etc.) |
| **Quoting** | Méthode pour protéger les caractères spéciaux de l'interprétation du shell |
| **Variable** | Conteneur nommé stockant une valeur réutilisable |
| **Argument** | Valeur passée à un script en ligne de commande |
| **Paramètre** | Variable (`$1`, `$2`, etc.) recevant un argument dans le script |
| **Substitution de commandes** | Syntaxe `$(commande)` pour récupérer le résultat d'une commande |
| **Substitution arithmétique** | Syntaxe `$(( calcul ))` pour effectuer des opérations mathématiques |
| **Code de sortie** | Valeur numérique (0-255) indiquant le statut de fin d'une commande |
| **Variable d'environnement** | Variable accessible par les processus fils (via `export`) |
| **Shell fils** | Nouveau shell créé par un shell parent (sous-shell) |
| **CLI** | Command Line Interface - interface en ligne de commande |
| **PID** | Process ID - identifiant unique d'un processus |
| **Parsing** | Analyse et interprétation d'une ligne de commande par le shell |
| **Whitespace** | Caractère "blanc" (espace ou tabulation) servant de séparateur |
| **Pipeline** | Enchaînement de commandes avec `|` (sortie de l'une → entrée de l'autre) |
| **Redirection** | Modification de l'entrée/sortie standard avec `<`, `>`, `>>` |
| **source** | Commande pour exécuter un script dans le shell courant (`. script.sh`) |
| **export** | Commande pour rendre une variable accessible aux processus fils |
| **unset** | Commande pour supprimer une variable |

---

## Références

> [!info] Ressources pour approfondir

### Documentation officielle
- **Manuel Bash** : `man bash` (documentation complète en ligne de commande)
- **GNU Bash Reference Manual** : documentation officielle en ligne

### Guides et tutoriels recommandés
- **Wikibooks : Programmation Bash** - tutoriel complet en français
- **Bash Hackers Wiki** - documentation technique approfondie
- **Greg's Bash Guide** - guide de référence pour les bonnes pratiques
- **ExplainShell** (explainshell.com) - outil pour comprendre les commandes

### Pour aller plus loin
- **Advanced Bash-Scripting Guide** - guide avancé
- **ShellCheck** (shellcheck.net) - outil de validation de scripts
- **Bash Pitfalls** - erreurs courantes à éviter

> [!tip] Conseil pour l'apprentissage
> Pratique régulièrement en créant de petits scripts utiles pour ton quotidien d'administrateur système. L'expérience pratique est la meilleure façon de maîtriser Bash !

---

> [!success] Fin de la Partie 1 - Les fondations
> Tu maîtrises maintenant les bases essentielles du scripting Bash : structure d'un script, variables, paramètres, substitutions et gestion de l'environnement. Ces concepts sont la fondation pour créer des scripts plus complexes et efficaces !

**Prochaine étape** : Partie 2 - Structures de contrôle (conditions, boucles) et gestion avancée des scripts.

---

*Document créé pour la préparation au titre RNCP Technicien Supérieur Systèmes et Réseaux (TSSR)*  
*Compatible Obsidian avec callouts natifs*