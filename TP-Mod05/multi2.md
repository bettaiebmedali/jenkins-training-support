# TP — Jenkins Multibranch Pipeline

## Objectif

À la fin de ce TP, vous serez capables de :

- comprendre le principe d'un Multibranch Pipeline ;
- comprendre la différence entre un Pipeline classique et un Multibranch ;
- connecter Jenkins à un dépôt GitLab ;
- détecter automatiquement les branches Git ;
- exécuter un Jenkinsfile selon la branche ;
- comprendre la variable Jenkins BRANCH_NAME.


# 1. Le problème

Imaginez que notre équipe travaille avec plusieurs branches Git :

    main
    develop
    feature/hello

Avec un Pipeline Jenkins classique, on pourrait avoir :

    Jenkins
       |
       +-- Pipeline

Mais comment Jenkins sait-il quelle branche construire ?

On pourrait créer manuellement :

    Jenkins
       +-- Pipeline-main
       +-- Pipeline-develop
       +-- Pipeline-feature-hello

Cela devient rapidement difficile à maintenir.

Avec un Multibranch Pipeline, Jenkins fait automatiquement le travail :

                    GitLab
                       |
            +----------+----------+
            |          |          |
           main     develop   feature/hello
            |          |          |
            +----------+----------+
                       |
                    Jenkins
                       |
              Multibranch Pipeline
                       |
            +----------+----------+
            |          |          |
           main     develop   feature/hello


# 2. Préparer le projet GitLab

Pour ce TP, nous allons utiliser un projet très simple.

Pas besoin de Java.

Pas besoin de Maven.

Pas besoin de Docker.

Pas besoin de tests.

Le projet contient simplement :

    jenkins-multibranch-demo/
    |
    +-- Jenkinsfile
    +-- README.md


# 3. Créer le Jenkinsfile

Créer un fichier :

    Jenkinsfile

Avec le contenu suivant :

    pipeline {

        agent any

        stages {

            stage('Information') {
                steps {
                    echo "Bonjour depuis Jenkins !"
                    echo "Branche : ${env.BRANCH_NAME}"
                    echo "Build : ${env.BUILD_NUMBER}"
                }
            }

            stage('Build') {
                steps {
                    echo "Build de la branche ${env.BRANCH_NAME}"
                }
            }
        }
    }

La ligne importante est :

    ${env.BRANCH_NAME}

Jenkins fournit automatiquement le nom de la branche.

Par exemple :

    Branche : main

ou :

    Branche : develop

ou :

    Branche : feature/hello


# 4. Créer le README.md

Créer simplement :

    README.md

Avec :

    # Jenkins Multibranch Demo

    Projet de démonstration Jenkins Multibranch Pipeline.


# 5. Créer les branches Git

Nous allons utiliser trois branches :

    main
    develop
    feature/hello

Exemple :

    git checkout -b develop
    git push origin develop

Puis :

    git checkout -b feature/hello
    git push origin feature/hello

Le dépôt GitLab doit donc contenir :

    main
    develop
    feature/hello


# 6. Créer le Multibranch Pipeline dans Jenkins

Dans Jenkins :

    New Item

Choisir :

    Multibranch Pipeline

Nom :

    jenkins-multibranch-demo

Puis :

    Branch Sources
        -> Add source
        -> Git

Indiquer le dépôt GitLab :

    https://gitlab.com/bettaiebmedaly/jenkins-multibranch-demo.git

Si le dépôt est privé, sélectionner les credentials GitLab nécessaires.


# 7. Indiquer le Jenkinsfile

Dans la configuration du Multibranch Pipeline :

    Build Configuration

Choisir :

    by Jenkinsfile

Script Path :

    Jenkinsfile

Jenkins va chercher un fichier nommé :

    Jenkinsfile

dans chaque branche.


# 8. Scanner le dépôt

Sauvegarder le job.

Puis lancer :

    Scan Multibranch Pipeline Now

Jenkins va analyser le dépôt GitLab.

Il va découvrir :

    main
    develop
    feature/hello

Et créer automatiquement les jobs correspondants.

Exemple :

    jenkins-multibranch-demo
    |
    +-- main
    +-- develop
    +-- feature-hello


# 9. Exécuter la branche main

Ouvrir :

    main

Puis :

    Build Now

Dans la console, nous devons retrouver quelque chose comme :

    Bonjour depuis Jenkins !
    Branche : main
    Build : 1
    Build de la branche main


# 10. Exécuter la branche develop

Ouvrir :

    develop

Puis :

    Build Now

La console affiche :

    Bonjour depuis Jenkins !
    Branche : develop
    Build : 1
    Build de la branche develop


# 11. Exécuter la branche feature/hello

Ouvrir :

    feature/hello

Puis :

    Build Now

La console affiche :

    Bonjour depuis Jenkins !
    Branche : feature/hello
    Build : 1
    Build de la branche feature/hello


# 12. Démonstration importante

Maintenant, nous allons modifier uniquement le Jenkinsfile de :

    feature/hello

Par exemple :

    pipeline {

        agent any

        stages {

            stage('Information') {
                steps {
                    echo "Pipeline de la branche feature/hello"
                    echo "Branche : ${env.BRANCH_NAME}"
                    echo "Build : ${env.BUILD_NUMBER}"
                }
            }

            stage('Build') {
                steps {
                    echo "Construction de la nouvelle fonctionnalité..."
                }
            }
        }
    }

Commit :

    git add Jenkinsfile
    git commit -m "Update feature pipeline"
    git push origin feature/hello

Relancer le job :

    feature/hello

On obtient :

    Pipeline de la branche feature/hello
    Branche : feature/hello
    Build : 2
    Construction de la nouvelle fonctionnalité...

Mais le pipeline main n'a pas changé.


# 13. Point important

Chaque branche possède potentiellement son propre Jenkinsfile.

Donc :

    main
        +-- Jenkinsfile

    develop
        +-- Jenkinsfile

    feature/hello
        +-- Jenkinsfile

Jenkins exécute le Jenkinsfile présent dans la branche concernée.


# 14. Mini-défi

Créer une nouvelle branche :

    feature/test

Commande :

    git checkout -b feature/test
    git push origin feature/test

Puis dans Jenkins :

    Scan Multibranch Pipeline Now

Question :

    Que va faire Jenkins ?

Réponse attendue :

    Jenkins détecte la nouvelle branche
            |
            v
    Jenkins trouve le Jenkinsfile
            |
            v
    Jenkins crée automatiquement le job
            |
            v
    Le pipeline peut être exécuté


# 15. Modifier le Jenkinsfile de feature/test

Dans la branche :

    feature/test

Modifier le message :

    echo "Je suis dans la branche feature/test"

Puis :

    git add Jenkinsfile
    git commit -m "Update test feature"
    git push origin feature/test

Relancer le pipeline.

Résultat attendu :

    Je suis dans la branche feature/test
    Branche : feature/test


# 16. Pipeline classique vs Multibranch

## Pipeline classique

    GitLab
       |
       +-- Pipeline Jenkins


## Multibranch Pipeline

                    GitLab
                       |
            +----------+----------+
            |          |          |
           main     develop   feature/hello
            |          |          |
            +----------+----------+
                       |
                  Jenkins
                       |
             Multibranch Pipeline
                       |
            +----------+----------+
            |          |          |
           main     develop   feature/hello


# 17. Ce qu'il faut retenir

## 1. Jenkins surveille le dépôt Git

    GitLab
       |
       v
    Branches


## 2. Jenkins détecte les branches

    main
    develop
    feature/hello


## 3. Jenkins cherche le Jenkinsfile

    branche
       |
       v
    Jenkinsfile


## 4. Jenkins crée les jobs correspondants

    main
    develop
    feature/hello


## 5. Jenkins fournit automatiquement le nom de branche

    env.BRANCH_NAME


# 18. Organisation possible sur 3 heures

## Partie 1 — Comprendre le concept

Durée : 30 min

Présenter :

- problème des jobs multiples ;
- principe du Multibranch ;
- rôle des branches Git ;
- rôle du Jenkinsfile.


## Partie 2 — Démonstration

Durée : 45 min

Faire en direct :

    Créer le projet GitLab
            |
            v
    Créer les branches
            |
            v
    Créer le Multibranch Jenkins
            |
            v
    Scanner le dépôt
            |
            v
    Observer les jobs
            |
            v
    Lancer les pipelines


## Partie 3 — Manipulation

Durée : 45 min

Les participants :

1. créent leur Multibranch ;
2. configurent GitLab ;
3. lancent le scan ;
4. exécutent main ;
5. exécutent develop ;
6. exécutent feature/hello.


## Partie 4 — Démonstration dynamique

Durée : 30 min

Créer :

    feature/test

Puis :

    git push

Scanner le dépôt.

Observer que Jenkins découvre automatiquement la nouvelle branche.


## Partie 5 — Mini-défi et questions

Durée : 30 min

Défi :

    Créer une nouvelle branche feature/demo,
    modifier le message du Jenkinsfile
    et vérifier que Jenkins crée le pipeline correspondant.


# 19. Conclusion

Le message principal du TP est simple :

    Un Multibranch Pipeline permet à Jenkins de détecter
    automatiquement les branches Git et d'exécuter
    le Jenkinsfile présent dans chaque branche.

À retenir :

    Git Branch
        |
        v
    Jenkinsfile
        |
        v
    Multibranch Pipeline
        |
        v
    Job Jenkins par branche

C'est la base avant d'aborder ensuite :

    Pull Request
    Webhooks
    Tests
    Build
    Docker
    Déploiement
    CI/CD

Ces éléments pourront être ajoutés dans des TPs suivants.