---
title: Contribution
nav_exclude: true
permalink: contribution
---

# Contribution

Cette page fournit quelques renseignements pour contribuer au contenu du cours, ou proposer des corrections.

## Initialisation de l'environnement de travail

Le contributeur devrait suivre les instructions suivantes la première fois qu'il ou elle désire contribuer
au projet:

1. Faites une _fork_ du repo sur GitHub: [fork](https://github.com/uquebec-ets/IND500/fork).

2. Clonez le repo et ajouter la fork commune:

    ```bash
    GH_USERNAME=<entrez_votre_compte_github>
    git clone git@github.com:$GH_USERNAME/IND500.git
    cd IND500
    git remote add upstream git@github.com:uquebec-ets/IND500.git
    git remote -v

    # Vous devriez voir apparaitre votre fork, ainsi que la fork commune
    ```

3. Ouvrez le répertoire `IND500` dans l'IDE de votre choix (par example VS Code).

4. Dans GitHub, sur la page de votre fork personnelle, allez dans `Settings`, `Pages` puis, dans la section 
   `Source`, choisissez `Deploy from a branch`. Ensuite, dans la section `Branch`, choisissez `main`, et `/docs`.
   Notez que si vous travaillez sur plusieurs branches, vous pouvez en choisir une autre que `main`, à cette
   étape.

## Effectuer un changement

Le contributeur devrait suivre les instructions suivantes pour chaque modifications:

1. Faites n'importe quelle modification souhaitée dans votre IDE. Consultez la documentation de 
   [Just the Docs](https://just-the-docs.com/docs/navigation/main/exclude/) au besoin.

2. Poussez vos changement à votre propre fork:

    ```bash
    git commit -a # Entrez un message de commit pertinent
    git push origin main

    # N.B.: Par la suite, si vous désirez écraser vos changements précédents, 
    #       plutôt que de créer un commit de plus, vous pouvez à la place faire:

    git commit -a --amend && git push -f origin main
    ```

3. Visionnez les changements sur votre propre fork:

    ```bash
    open https://$GH_USERNAME.github.io/IND500/

    # N.B.: Le déploiment de vos changements à la page ci-dessus peut prendre 
    #       environ 1 minute. Voyez le progrès via:

    open https://github.com/$GH_USERNAME/IND500/actions
    ```

4. Recommencez les étapes 1 à 3 jusqu'à temps que vous atteignez le résultat escompté.

5. Ouvrez une Pull Request de votre fork vers la fork commune.