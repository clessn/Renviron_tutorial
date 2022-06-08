# Secrets Tutorial
Comment cacher des valeurs secretes comme des mots de passe dans un projet

1. S'assurer que la ligne .Renviron est présente dans .gitignore
2. Créer un fichier .Renviron
3. Ajouter la ligne `MA_VALEUR=patate123` dans le .Renviron
4. Redémarrer R (Session -> Restart R, ou Ctrl+Shift+F10)
5. taper dans la console `Sys.getenv("MA_VALEUR")`
6. Confirmer que `patate123` apparait
