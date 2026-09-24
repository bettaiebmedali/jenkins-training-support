# 🚀 Jenkins Expert — CI/CD complet

## 🎯 Objectif

Créer un **Multibranch Pipeline Jenkins** capable de :

* Récupérer automatiquement le code depuis Git ;
* Lire les variables d'un fichier `.env` ;
* Construire une image Docker (avec tags versionné et `latest`) ;
* Publier l'image sur Docker Hub de manière sécurisée ;
* Déployer uniquement depuis la branche `main` ;
* Utiliser les credentials Jenkins de manière sécurisée.

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

Le projet est configuré comme **Multibranch Pipeline**. 
Jenkins récupère automatiquement le dépôt et le `Jenkinsfile`. Il n'est donc pas nécessaire de faire un checkout manuel dans le `Jenkinsfile`.

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
Build Docker (version + latest)
       │
       ▼
Login & Push Docker
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

        // Utilisateur Docker Hub
        DOCKER_USER = 'dockerID'

        // Repository Docker en minuscules
        DOCKER_IMAGE = 'dockerID/microapp'

        // Variables applicatives
        APP_NAME = ''
        APP_PORT = ''
        APP_VERSION = ''
    }

    stages {

        // ============================================================
        // 1. Lecture du fichier .env
        // ============================================================

        stage('Charger .env') {

            steps {

                script {

                    def config = [:]

                    def envFile = readFile('.env')

                    envFile.split('\n').each { line ->

                        line = line.trim()

                        // Ignore les lignes vides et les commentaires
                        if (line && !line.startsWith('#')) {

                            def parts = line.split('=', 2)

                            if (parts.size() == 2) {

                                def key = parts[0].trim()
                                def value = parts[1].trim()

                                config[key] = value
                            }
                        }
                    }

                    // Docker exige un repository en minuscules
                    env.APP_NAME = config['APP_NAME']?.toLowerCase()
                    env.APP_PORT = config['APP_PORT']
                    env.APP_VERSION = config['APP_VERSION']

                    echo "========================================"
                    echo "Configuration de l'application"
                    echo "========================================"
                    echo "Application : ${env.APP_NAME}"
                    echo "Port        : ${env.APP_PORT}"
                    echo "Version     : ${env.APP_VERSION}"
                    echo "Image       : ${env.DOCKER_IMAGE}:${env.APP_VERSION}"
                    echo "========================================"
                }
            }
        }

        // ============================================================
        // 2. Build Docker
        // ============================================================

        stage('Build Docker') {

            steps {

                echo "🐳 Construction de l'image Docker..."

                sh """
                    docker build \
                        -t ${env.DOCKER_IMAGE}:${env.APP_VERSION} \
                        -t ${env.DOCKER_IMAGE}:latest \
                        .
                """

                echo "✅ Image Docker construite"

                sh """
                    docker images ${env.DOCKER_IMAGE}
                """
            }
        }

        // ============================================================
        // 3. Login Docker Hub
        // ============================================================

        stage('Login Docker Hub') {

            steps {

                withCredentials([
                    string(
                        credentialsId: 'DOCKER_PASSWORD',
                        variable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin
                    '''
                }

                echo "✅ Connexion à Docker Hub réussie"
            }
        }

        // ============================================================
        // 4. Push Docker
        // ============================================================

        stage('Push Docker') {

            steps {

                echo "📦 Publication sur Docker Hub..."

                sh """
                    docker push ${env.DOCKER_IMAGE}:${env.APP_VERSION}
                    docker push ${env.DOCKER_IMAGE}:latest
                """

                echo "✅ Image publiée sur Docker Hub"
            }
        }

        // ============================================================
        // 5. Deploy uniquement sur main
        // ============================================================

        stage('Deploy') {

            when {
                branch 'main'
            }

            steps {

                echo """
                ========================================
                🚀 DÉPLOIEMENT EN PRODUCTION
                ========================================

                Application : ${env.APP_NAME}
                Version     : ${env.APP_VERSION}
                Port        : ${env.APP_PORT}
                Image       : ${env.DOCKER_IMAGE}:${env.APP_VERSION}

                ========================================
                """

                // Déploiement simulé pour le TP
                sh '''
                    echo "Déploiement en cours..."
                    sleep 3
                    echo "✅ Déploiement terminé"
                '''
            }
        }
    }

    // ================================================================
    // Actions finales
    // ================================================================

    post {

        success {

            echo """
            ========================================
            ✅ PIPELINE RÉUSSIE
            ========================================

            Application : ${env.APP_NAME}
            Version     : ${env.APP_VERSION}
            Image       : ${env.DOCKER_IMAGE}:${env.APP_VERSION}

            ========================================
            """
        }

        failure {

            echo """
            ========================================
            ❌ PIPELINE EN ÉCHEC
            ========================================

            Application : ${env.APP_NAME ?: 'inconnue'}
            Version     : ${env.APP_VERSION ?: 'inconnue'}

            ========================================
            """
        }

        always {

            echo "🏁 Fin du pipeline"
        }
    }
}
```

---

## 🔑 Points importants

### 1. Pas de `stage('Checkout')`
Dans un **Multibranch Pipeline**, Jenkins effectue automatiquement le checkout du dépôt. On ne fait donc pas :

```groovy
stage('Checkout') {
    steps {
        git branch: "${env.BRANCH_NAME}",
            url: 'https://github.com/monrepo/my-app.git'
    }
}
```

### 2. Le `.env` est chargé après le checkout
Le fichier `.env` est disponible dans le workspace. On le lit avec `readFile('.env')`, puis on injecte ses clés dans les variables `env.APP_*` de Jenkins.

### 3. Pourquoi `split('=', 2)` ?
On préfère `split('=', 2)` à `tokenize('=')` car une valeur peut elle-même contenir un signe `=`.

**Exemple :**
`APP_URL=https://example.com?a=123`
* `key` = `APP_URL`
* `value` = `https://example.com?a=123`

### 4. Credentials Docker
Les mots de passe ne doivent **jamais** figurer dans le fichier `.env`.
* `.env` contient la configuration de l'application (`APP_NAME`, `APP_PORT`, `APP_VERSION`).
* Les identifiants secrets sont gérés par le store de secrets de Jenkins via `withCredentials`.

---

## 🧾 Résumé

| Élément | Rôle |
| :--- | :--- |
| **Multibranch Pipeline** | Détecte et construit automatiquement les branches Git. |
| **SCM** | Gère le checkout sans action manuelle. |
| **`.env`** | Stocke les paramètres spécifiques de l'application. |
| **`readFile()`** | Charge le fichier `.env` depuis le workspace. |
| **`withCredentials()`** | Sécurise les identifiants pour l'authentification Docker. |
| **`when`** | Restreint le déploiement à la branche `main`. |
| **`post`** | Exécute des étapes de conclusion (succès/échec/toujours). |