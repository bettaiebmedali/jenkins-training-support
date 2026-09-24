# 🚀 Jenkins Expert — CI/CD complet

## 🎯 Objectif

Créer un **Multibranch Pipeline Jenkins** capable de :

- Récupérer automatiquement le code depuis Git ;
- Lire les variables d'un fichier `.env` ;
- Construire une image Docker ;
- Publier l'image sur Docker Hub ;
- Déployer uniquement depuis la branche `main` ;
- Utiliser les credentials Jenkins de manière sécurisée.

---

## 🧩 Structure du projet

```text
my-app/
├── .env
├── Dockerfile
├── app.py
└── Jenkinsfile
```

### Exemple de `.env`

```env
APP_NAME=MicroApp
APP_PORT=8080
APP_VERSION=3.0.0
```

---

## 🔄 Fonctionnement

Le projet est configuré comme **Multibranch Pipeline**. Jenkins récupère automatiquement le dépôt et le `Jenkinsfile`. Il n'est donc pas nécessaire de faire un checkout manuel dans le `Jenkinsfile`.

```text
Git
 │
 ├── main
 ├── develop
 └── feature/*
       │
       ▼
Multibranch Pipeline
       │
       ▼
Jenkins récupère le code
       │
       ▼
Lecture du .env
       │
       ▼
Build Docker
       │
       ▼
Push Docker
       │
       ▼
Deploy uniquement sur main
```

---

## 📄 Jenkinsfile

```groovy
pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'monuser/microapp'
    }

    stages {

        stage('Charger .env') {

            steps {

                script {

                    def envFile = readFile('.env')

                    envFile.split('\n').each { line ->

                        line = line.trim()

                        if (line && !line.startsWith('#')) {

                            def parts = line.split('=', 2)

                            if (parts.size() == 2) {

                                def key = parts[0].trim()
                                def value = parts[1].trim()

                                env[key] = value
                            }
                        }
                    }

                    echo "Application : ${env.APP_NAME}"
                    echo "Port        : ${env.APP_PORT}"
                    echo "Version     : ${env.APP_VERSION}"
                }
            }
        }

        stage('Build Docker') {

            steps {

                script {

                    def image = "${env.DOCKER_IMAGE}:${env.APP_VERSION}"

                    echo "🐳 Construction de l'image : ${image}"

                    docker.build(image)
                }
            }
        }

        stage('Push Docker') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker_hub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker push \
                        "$DOCKER_IMAGE:$APP_VERSION"
                    '''
                }
            }
        }

        stage('Deploy') {

            when {
                branch 'main'
            }

            steps {

                echo """
                🚀 Déploiement en production

                Application : ${env.APP_NAME}
                Version     : ${env.APP_VERSION}
                Port        : ${env.APP_PORT}
                Image       : ${env.DOCKER_IMAGE}:${env.APP_VERSION}
                """
            }
        }
    }

    post {

        success {
            echo "✅ Pipeline terminé avec succès"
        }

        failure {
            echo "❌ Pipeline en échec"
        }
    }
}
```

---

## 🔑 Points importants

### 1. Pas de `stage('Checkout')`
Dans un Multibranch Pipeline, Jenkins effectue automatiquement le checkout du dépôt. On ne fait donc pas :

```groovy
stage('Checkout') {
    steps {
        git branch: "${env.BRANCH_NAME}",
            url: 'https://github.com/monrepo/my-app.git'
    }
}
```
*Jenkins s'en charge automatiquement.*

### 2. Le `.env` est chargé après le checkout
Le fichier `.env` est déjà présent dans le workspace. On peut donc faire :

```groovy
def envFile = readFile('.env')
```

Puis transformer :

```env
APP_NAME=MicroApp
APP_PORT=8080
APP_VERSION=3.0.0
```

en variables Jenkins :

- `env.APP_NAME`
- `env.APP_PORT`
- `env.APP_VERSION`

### 3. Pourquoi `split('=', 2)` ?
On préfère `split('=', 2)` à `tokenize('=')` car une valeur peut elle-même contenir un `=`.

**Exemple :**
```env
APP_URL=https://example.com?a=123
```

Avec `split('=', 2)` :
- `key` = `APP_URL`
- `value` = `https://example.com?a=123`

### 4. Credentials Docker
On ne met jamais le mot de passe Docker dans le `.env`.

❌ **À éviter dans `.env` :**
```env
DOCKER_PASSWORD=monMotDePasse
```

✅ **Le `.env` contient uniquement la configuration applicative :**
```env
APP_NAME=MicroApp
APP_PORT=8080
APP_VERSION=3.0.0
```

Les identifiants Docker sont stockés directement dans Jenkins :

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'docker_hub',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASSWORD'
    )
])
```

### 🌿 5. Condition sur la branche `main`

Cette partie :

```groovy
when {
    branch 'main'
}
```

signifie que seule la branche `main` déclenche le déploiement.

**Exemple :**

```text
feature/login
      │
      ├── Charger .env
      ├── Build Docker
      ├── Push Docker
      └── ❌ Pas de Deploy

main
 │
 ├── Charger .env
 ├── Build Docker
 ├── Push Docker
 └── Deploy
       ✅
```

---

## 🧾 Résumé

| Élément | Rôle |
| :--- | :--- |
| **Multibranch Pipeline** | Détecte et construit les branches |
| **SCM** | Gère automatiquement le checkout |
| **`.env`** | Contient les paramètres de l'application |
| **`readFile()`** | Lit le fichier `.env` |
| **`env.APP_NAME`** | Variable de l'environnement Jenkins |
| **`docker.build()`** | Construit l'image Docker |
| **`withCredentials()`** | Protège les identifiants de manière sécurisée |
| **`docker push`** | Publie l'image sur le registre |
| **`when`** | Contrôle les conditions du déploiement |
| **`post`** | Gère les actions après l'exécution du pipeline |