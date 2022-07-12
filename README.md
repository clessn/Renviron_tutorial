# Secrets Tutorial
Comment cacher des valeurs secretes comme des mots de passe dans un projet

## Concept
Le problème avec github, c'est qu'il garde tous les derniers changements en mémoire. Donc, si vous pushez votre mot de passe, alors même si vous le supprimez et re-pushez vos changements, il reste dans l'historique. Il existe des solutions, mais elles sont généralement plus complexes que d'utiliser de bonnes méthodes de travail. Hadley Wickham recommande donc d'utiliser un fichier `.Renviron`. 

### Qu'est-ce qu'une variable d'environnement?
Une variable d'environnement est une variable accessible par tous les processus d'une session. Dans notre cas et pour nos besoins, c'est une variable qu'on peut configurer en dehors de R et y accéder par R. C'est une des fonctionnalité les plus utilisée d'un systeme d'exploitation, mais c'est aussi une bonne solution pour ajouter nos configurations secrètes et ne pas les pusher.

Si vous allez dans le terminal (pas la console R, mais le terminal), vous pouvez visualiser quelques variables d'environnement qui existent sur votre system:

Windows: tapez `echo %DATE%` pour voir la env var qui contient la date du jour

Mac et linux: tapez `echo $HOSTNAME` pour voir le nom de votre ordinateur

Ces valeurs sont des variables, et non des fonctions qui retournent des valeurs. Notez qu'une variable d'environnement ne vit que dans une session (R, un terminal, etc.), et donc si vous créez vos variables, vous devrez les recréer si vous fermez votre RStudio. D'où l'utilité du fichier `.Renviron`

### Pour les curieux:
1. Ouvrez un terminal sur votre ordinateur. tapez `export PATATE=jaune` sur mac ou linux, ou `SET PATATE=jaune` sur windows.
2. tapez `echo $PATATE` sur mac et linux, ou `echo %PATATE%` sur windows. Vous devriez voir apparaitre `jaune`.
3. Fermez le terminal, lancez un nouveau, puis recommencez le point 2. `jaune` n'apparait plus.

Les variables d'environnement, à moins d'être configurés à chaque nouvelle session, cessent d'exister!!

## La partie importante: R et .Renviron
Pour R, il est possible de configurer des variables d'environnement à deux
niveaux :

- usager : les variables sont visibles par tous les projets R
- projet : les variables sont visibles uniquement par le projet dans lequel 
on se trouve

Les variables d'environnement configurées au niveau de l'usager sont utiles
pour des valeurs de variables qui demeurent les mêmes à travers les projets. Par exemple, l'url du hub3.0.

### Pour les variable d'environnement au niveau usager
1. Exécuter `usethis::edit_r_environ()` sur la ligne de commande R
2. Ajouter la ligne `MA_VALEUR=patate123` dans le .Renviron
4. Redémarrer R (Session -> Restart R, ou Ctrl+Shift+F10)
5. Taper dans la console R `Sys.getenv("MA_VALEUR")`
6. Confirmer que `patate123` apparait
7. Profit!

Et oui, vous pouvez utiliser les fonctions `Sys.getenv()` et `Sys.setenv()` pour manipuler les variables d'environnement. Mais le plus important est donc `Sys.getenv()`

Supposons maintenant que vous voulez utiliser une table dans le hub3.0. Les fonctions du hub demandent à ce que vous leur fournissiez vos informations de connection (*credentials*). Pour ce faire

1. Ajoutez dans le fichier .Renviron (avec la fonction `usethis::edit_r_environ()`)
```sh
# .Renviron
HUB3_URL      = "https://clhub.clessn.cloud/"
HUB3_USERNAME = "mon.nom.usager"
HUB3_PASSWORD = "mon.mdp"
```
2. Redémarrez la session R et validez que les variables existent.
3. Maintenant, il est possible d'utiliser ces variables d'environnement pour instancier les *credentials* à utiliser dans les fonctions de `hublot`:
```R
credentials <- hublot::get_credentials(
            Sys.getenv("HUB3_URL"), 
            Sys.getenv("HUB3_USERNAME"), 
            Sys.getenv("HUB3_PASSWORD")
            )
```
Le code ainsi produit pourra être utilisé par tout utilisateur ayant configurer ses variables d'environnement pour accéder au hub3.0.


### Pour les variables d'environnement au niveau du projet
1. S'assurer que la ligne .Renviron est présente dans .gitignore (sinon vous pusherez vos variables secrètes. Zut!)
2. Créer un fichier .Renviron
3. Ajouter la ligne `MA_VALEUR=patate123` dans le .Renviron
4. Redémarrer R (Session -> Restart R, ou Ctrl+Shift+F10)
5. taper dans la console R `Sys.getenv("MA_VALEUR")`
6. Confirmer que `patate123` apparait
7. Profit!

Et oui, vous pouvez utiliser les fonction `Sys.getenv()` et `Sys.setenv()` pour manipuler les variables d'environnement. Mais le plus important est donc `Sys.getenv()`

Supposons maintenant que vous voulez vous connecter au clessnhub. Généralement, la fonction `clessnhub::connect()` vous demanderait d'entrer votre identifiant et mot de passe visuellement, ce qui n'est pas efficace si vous voulez automatiser.

Vous pourriez utiliser le code suivant:
```R
clessnhub::login('myusername', 'mypassword')
```
Mais le mot de passe se retrouvera dans github. Pas mieux!

#### La solution pour les variables d'environnement relatives à un projet
1. Assurez-vous d'abord que .Renviron est dans le .gitignore.
2. Ajoutez dans le fichier .Renviron (ou créez-le), des variables pour votre utilisateur et mot de passe
```sh
# .Renviron
CLESSNHUB_USERNAME=olivier
CLESSNHUB_PASSWORD=patate123
```
3. Maintenant, dans réécrivons le code `clessnhub::login` pour utiliser les variables d'environnement:
```R
clessnhub::login(
    Sys.getenv("CLESSNHUB_USERNAME"), 
    Sys.getenv("CLESSNHUB_PASSWORD")
)
```
4. Redémarrez la session R et testez le code.

Voilà! Il vous suffit d'indiquer, dans le readme de votre projet, qu'un nouvel utilisateur devrait lui aussi créer son .Renviron et mettre ses valeurs dans les variables. Quelque chose du genre:
```md
1. Clonez ce repo
2. Créer votre fichier .Renviron
3. Ajoutez-y les variables CLESSNHUB_USERNAME et CLESSNHUB_PASSWORD et vos identifiants
```

À noter que même si .Renviron est dans .gitignore, il restera sur votre poste et vous pourrez donc travailler sur votre projet même si vous pullez ou pushez. La seule situation où il peut disparaitre est si vous changez de branche puis mergez, mais c'est rarement le cas dans vos projets.
