# TP — GitLab avec Jenkins Multibranch

## 1. Titre

**TP : Intégration GitLab avec Jenkins Multibranch**

## 2. Durée

**Durée estimée : 3 heures**

- Niveau 1 — Connexion Jenkins ↔ GitLab : 30 min
- Niveau 2 — Création du Pipeline Multibranch : 45 min
- Niveau 3 — Gestion des branches : 45 min
- Niveau 4 — Webhook et automatisation : 30 min
- Challenge final : 30 min

## 3. Objectifs

À la fin du TP, les participants seront capables de :

- Connecter Jenkins à un repository GitLab.
- Configurer les credentials GitLab dans Jenkins.
- Créer un projet Jenkins Multibranch Pipeline.
- Utiliser un `Jenkinsfile` stocké dans GitLab.
- Détecter automatiquement les branches Git.
- Exécuter des étapes différentes selon la branche.
- Utiliser les variables Jenkins.
- Configurer un webhook GitLab vers Jenkins.
- Déclencher automatiquement un pipeline après un `push`.

## 4. Prérequis

- Jenkins fonctionnel.
- GitLab accessible.
- Compte GitLab.
- Compte Jenkins avec droits de configuration.
- Git installé.
- Python 3 installé sur l'agent Jenkins.
- Navigateur Web.
- Connaissances de base de Git et Jenkins.

### Hypothèse

On suppose :

```text
GitLab  : http://gitlab.local
Jenkins : http://localhost:8080
```

Adapter les URL à l'environnement utilisé.

Repository :

```text
gitlab.local/devops/jenkins-multibranch-demo
```

## 5. Architecture ou contexte

```text
                    +----------------+
                    |     GitLab     |
                    |                |
                    | main           |
                    | develop        |
                    | feature/*      |
                    +-------+--------+
                            |
                            | Push / Webhook
                            v
                    +----------------+
                    |     Jenkins    |
                    |  Multibranch   |
                    +-------+--------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
           main          develop       feature/*
              |             |             |
              v             v             v
          Pipeline       Pipeline       Pipeline
```

Structure du repository :

```text
jenkins-multibranch-demo/
├── Jenkinsfile
├── requirements.txt
├── app/
│   └── app.py
├── tests/
│   └── test_app.py
└── README.md
```

## 6. Étapes pratiques détaillées

### Niveau 1 — Préparer le repository GitLab

Créer dans GitLab :

```text
Projects
→ New project
→ Create blank project
```

Nom :

```text
jenkins-multibranch-demo
```

Namespace :

```text
devops
```

Visibilité :

```text
Private
```

### Configurer Git

```bash
git config --global user.name "Jenkins Training"
git config --global user.email "jenkins-training@example.com"
```

### Cloner le repository

```bash
git clone http://gitlab.local/devops/jenkins-multibranch-demo.git
cd jenkins-multibranch-demo
```

### Créer l'application

Créer `app/app.py` :

```python
def hello():
    return "Hello from GitLab + Jenkins"


if __name__ == "__main__":
    print(hello())
```

Créer `tests/test_app.py` :

```python
from app.app import hello


def test_hello():
    assert hello() == "Hello from GitLab + Jenkins"
```

Créer `requirements.txt` :

```text
pytest
```

Créer `README.md` :

```md
# Jenkins Multibranch Demo

Application utilisée pour le TP GitLab + Jenkins Multibranch.

Branches :

- main
- develop
- feature/*
```

### Premier commit

```bash
git add .
git commit -m "Initial application"
git push origin main
```

### Créer develop

```bash
git checkout -b develop
git push -u origin develop
```

### Créer une feature

```bash
git checkout -b feature/add-version
```

Modifier `app/app.py` :

```python
def hello():
    return "Hello from GitLab + Jenkins"


def version():
    return "1.1.0"


if __name__ == "__main__":
    print(hello())
    print(version())
```

Puis :

```bash
git add .
git commit -m "Add application version"
git push -u origin feature/add-version
```

Vérifier :

```bash
git branch -a
```

Branches attendues :

```text
main
develop
feature/add-version
```

# Niveau 2 — Configurer les credentials GitLab dans Jenkins

## Créer un token GitLab

Dans GitLab :

```text
User Settings
→ Access Tokens
```

Créer :

```text
Name: jenkins-training
```

Accorder au minimum :

```text
read_repository
```

Créer le token et conserver sa valeur.

## Ajouter le credential dans Jenkins

Dans Jenkins :

```text
Manage Jenkins
→ Credentials
→ System
→ Global credentials
→ Add Credentials
```

Type :

```text
Username with password
```

Configurer :

```text
Username    : <utilisateur_gitlab>
Password    : <token_gitlab>
ID          : gitlab-credentials
Description : GitLab credentials for Multibranch
```

Enregistrer.

# Niveau 3 — Créer le Jenkinsfile

Créer `Jenkinsfile` à la racine :

```groovy
pipeline {

    agent any

    stages {

        stage('Information') {
            steps {
                echo "Branch: ${env.BRANCH_NAME}"
                echo "Build: ${env.BUILD_NUMBER}"
            }
        }

        stage('Install dependencies') {
            steps {
                sh '''
                    python3 -m pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m pytest
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    mkdir -p build
                    cp app/app.py build/
                '''
            }
        }

        stage('Develop validation') {
            when {
                branch 'develop'
            }
            steps {
                echo "Validation spécifique à develop"
            }
        }

        stage('Feature validation') {
            when {
                branch pattern: "feature/.*", comparator: "REGEXP"
            }
            steps {
                echo "Validation spécifique aux branches feature"
            }
        }

        stage('Production deployment') {
            when {
                branch 'main'
            }
            steps {
                echo "Déploiement production"
            }
        }
    }

    post {
        success {
            echo "Pipeline successfully completed"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
```

Envoyer le Jenkinsfile :

```bash
git add Jenkinsfile requirements.txt
git commit -m "Add Jenkins pipeline"
git push
```

# Niveau 4 — Créer le Multibranch Pipeline

Dans Jenkins :

```text
New Item
```

Nom :

```text
jenkins-multibranch-demo
```

Type :

```text
Multibranch Pipeline
```

Cliquer sur :

```text
OK
```

## Configurer la source Git

Dans :

```text
Branch Sources
```

Ajouter :

```text
Git
```

Repository URL :

```text
http://gitlab.local/devops/jenkins-multibranch-demo.git
```

Credentials :

```text
gitlab-credentials
```

## Configurer la découverte

Activer :

```text
Discover branches
```

Si disponible avec les plugins installés, activer également la découverte des Merge Requests.

## Configurer le Jenkinsfile

Dans :

```text
Build Configuration
```

Sélectionner :

```text
by Jenkinsfile
```

Script Path :

```text
Jenkinsfile
```

Enregistrer.

## Scanner le repository

Dans le job :

```text
Scan Multibranch Pipeline Now
```

Jenkins doit détecter :

```text
main
develop
feature/add-version
```

Structure Jenkins attendue :

```text
jenkins-multibranch-demo
├── main
├── develop
└── feature-add-version
```

# Niveau 5 — Tester les pipelines

## Tester main

```text
jenkins-multibranch-demo
→ main
→ Build Now
```

Vérifier les étapes :

```text
Information
Install dependencies
Test
Build
Production deployment
```

## Tester develop

```text
jenkins-multibranch-demo
→ develop
→ Build Now
```

Vérifier :

```text
Develop validation
```

## Tester feature/add-version

```text
jenkins-multibranch-demo
→ feature-add-version
→ Build Now
```

Vérifier :

```text
Feature validation
```

# Niveau 6 — Ajouter un Webhook GitLab

Dans GitLab :

```text
Project
→ Settings
→ Webhooks
```

Configurer l'URL Jenkins adaptée à l'installation.

Exemple :

```text
http://jenkins.local/project/jenkins-multibranch-demo
```

Activer :

```text
Push events
```

Si nécessaire :

```text
Merge request events
```

Tester le webhook depuis GitLab.

# Niveau 7 — Tester l'automatisation

Se positionner sur `develop` :

```bash
git checkout develop
```

Modifier `app/app.py` :

```python
def hello():
    return "Hello from GitLab + Jenkins - version 2"


if __name__ == "__main__":
    print(hello())
```

Commit et push :

```bash
git add app/app.py
git commit -m "Update application message"
git push origin develop
```

Workflow attendu :

```text
git push
    |
    v
GitLab
    |
    | Webhook
    v
Jenkins
    |
    | Multibranch
    v
develop
    |
    v
Jenkinsfile
    |
    +--> Install
    |
    +--> Test
    |
    +--> Build
    |
    +--> Develop validation
```

# 7. Commandes à exécuter

## Git

```bash
git status
git branch
git branch -a
git remote -v
git log --oneline --decorate --graph --all
```

## Branches

```bash
git checkout -b develop
git push -u origin develop
```

```bash
git checkout -b feature/add-version
git push -u origin feature/add-version
```

## Commit et push

```bash
git add .
git commit -m "Update application"
git push
```

## Python

```bash
python3 --version
python3 -m pip --version
python3 -m pytest
```

## Informations Jenkins

```bash
echo "Branch: ${BRANCH_NAME}"
echo "Build: ${BUILD_NUMBER}"
echo "Workspace: ${WORKSPACE}"
```

# 8. Fichiers de configuration nécessaires

## Jenkinsfile

Utiliser le Jenkinsfile fourni dans le niveau 3.

## requirements.txt

```text
pytest
```

## app/app.py

```python
def hello():
    return "Hello from GitLab + Jenkins"


def version():
    return "1.1.0"


if __name__ == "__main__":
    print(hello())
    print(version())
```

## tests/test_app.py

```python
from app.app import hello


def test_hello():
    assert hello() == "Hello from GitLab + Jenkins"
```

## README.md

```md
# Jenkins Multibranch Demo

Application utilisée pour le TP GitLab + Jenkins Multibranch.

Branches :

- main
- develop
- feature/*
```

# 9. Résultat attendu

À la fin du TP :

```text
jenkins-multibranch-demo
│
├── main
│   └── Pipeline SUCCESS
│
├── develop
│   └── Pipeline SUCCESS
│
└── feature-add-version
    └── Pipeline SUCCESS
```

Comportement :

```text
main
 └── Test → Build → Production deployment

develop
 └── Test → Build → Develop validation

feature/*
 └── Test → Build → Feature validation
```

# 10. Challenge ou exercice final

Créer :

```text
feature/ci-improvement
```

Commandes :

```bash
git checkout develop
git pull
git checkout -b feature/ci-improvement
```

Modifier `app/app.py` :

```python
def hello():
    return "Hello from GitLab + Jenkins"


def version():
    return "2.0.0"


def environment():
    return "training"


if __name__ == "__main__":
    print(hello())
    print(version())
    print(environment())
```

Ajouter un test dans `tests/test_app.py` :

```python
from app.app import environment


def test_environment():
    assert environment() == "training"
```

Envoyer :

```bash
git add .
git commit -m "Add environment information"
git push -u origin feature/ci-improvement
```

Vérifier que Jenkins détecte :

```text
feature-ci-improvement
```

Vérifier :

```text
Checkout
Install dependencies
Test
Build
Feature validation
```

## Challenge supplémentaire

Ajouter au Jenkinsfile :

```groovy
stage('Quality Check') {
    when {
        anyOf {
            branch 'develop'
            branch 'main'
        }
    }

    steps {
        sh '''
            echo "Running quality checks..."
            python3 -m pytest
        '''
    }
}
```

Ajouter un déploiement uniquement sur `main` :

```groovy
stage('Deploy') {
    when {
        branch 'main'
    }

    steps {
        sh '''
            echo "Deploying application to production..."
        '''
    }
}
```

Commit :

```bash
git add Jenkinsfile
git commit -m "Improve CI pipeline"
git push
```

Tester :

```text
main
develop
feature/ci-improvement
```

# 11. Compétences mises en pratique

- Git
- GitLab
- Git branches
- Git push
- Git credentials
- Jenkins credentials
- Jenkins Multibranch Pipeline
- Jenkinsfile
- Declarative Pipeline
- Jenkins stages
- Jenkins `when`
- Variables Jenkins
- `BRANCH_NAME`
- Tests automatisés
- Webhooks
- GitLab → Jenkins
- CI
- Pipeline conditionnel
- Gestion des branches
- Préparation d'un workflow CI/CD

## Workflow final

```text
Developer
    |
    | git checkout -b feature/...
    |
    | git commit
    |
    | git push
    v
  GitLab
    |
    | Webhook
    v
  Jenkins
    |
    | Multibranch Pipeline
    |
    +--------------------+
    |                    |
    v                    v
feature/*            develop/main
    |                    |
    v                    v
Tests                 Tests
    |                    |
    v                    v
Build                 Build
    |                    |
    v                    +---------> Validation
Feature validation         |
                           v
                         main
                           |
                           v
                       Deployment
```
