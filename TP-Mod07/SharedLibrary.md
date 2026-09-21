# Création et Utilisation d'une Shared Library Jenkins

## 🎯 Objectif
Création d'une Shared Library Jenkins et son utilisation dans un pipeline.

---

## 📌 Prérequis
- Compte GitHub  
- Accès administrateur à Jenkins  
- Plugins Jenkins installés : **Git**, **Pipeline**

---

## Étape 1 : Création de la structure de la Shared Library sur GitHub

1. Créer un nouveau dépôt GitHub nommé **jenkins-shared-library**
2. Cloner le dépôt localement :
   ```bash
   git clone https://github.com/votre-utilisateur/jenkins-shared-library.git
   cd jenkins-shared-library
Créer la structure suivante :

css
Copy code
jenkins-shared-library/
├── vars/
│   └── sayHello.groovy
└── src/
    └── org/
        └── example/
            └── Utils.groovy
Ajouter le contenu suivant :

vars/sayHello.groovy
groovy
Copy code
def call(String name = 'World') {
    echo "Hello, ${name}!"
}
src/org/example/Utils.groovy
groovy
Copy code
package org.example

class Utils {
    static String toUpperCase(String input) {
        return input.toUpperCase()
    }
}
Pousser les changements vers GitHub :

bash
Copy code
git add .
git commit -m "Add shared library structure and files"
git push origin main
Étape 2 : Configuration de la Shared Library dans Jenkins
Aller dans : Manage Jenkins > System Configuration > Configure System

Descendre jusqu'à Global Pipeline Libraries

Ajouter une nouvelle bibliothèque :

Name : jenkins-shared-library

Default version : main

Retrieval method : Modern SCM

Source Code Management : Git

Project URL :

arduino
Copy code
https://github.com/votre-utilisateur/jenkins-shared-library.git
Sauvegarder.

Étape 3 : Création d'un pipeline testant la Shared Library
Créer un pipeline (via un job Pipeline ou un Jenkinsfile) :

groovy
Copy code
@Library('jenkins-shared-library') _

pipeline {
    agent any

    stages {
        stage('Test Shared Library') {
            steps {
                script {
                    // Utilisation de la fonction globale
                    sayHello 'Jenkins'
                    sayHello 'Lab User'

                    // Utilisation de la classe utilitaire
                    def utils = new org.example.Utils()
                    def result = utils.toUpperCase('this should be uppercase')
                    echo "Uppercase result: ${result}"
                }
            }
        }
    }
}
Étape 4 : Exécution et Validation
Lancer le pipeline, puis vérifier dans les logs :

Le message "Hello, Jenkins!" apparaît

Le message "Hello, Lab User!" apparaît

Le message "THIS SHOULD BE UPPERCASE" apparaît

