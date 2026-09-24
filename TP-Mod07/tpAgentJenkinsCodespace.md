# TP — Jenkins Agent + Docker + Nginx

## 1. Créer l'agent Jenkins

Dans Jenkins :

1. Allez dans **Manage Jenkins → Nodes → New Node**
2. Configurer le nœud :
   - **Nom :** `codespace01`
   - **Type :** `Permanent Agent`
3. Renseigner les paramètres suivants :
   - **Remote root directory :** `/home/codespace/jenkins-agent`
   - **Labels :** `codespace01`
   - **Launch method :** `Launch agents via WebSocket`
4. Cliquer sur **Enregistrer**.

---

## 2. Préparer le Codespace

Dans le terminal du Codespace :

1. Vérifier le répertoire utilisateur :
   ```bash
   echo $HOME
   ```

2. Créer le dossier dédié à l'agent :
   ```bash
   mkdir -p ~/jenkins-agent
   ```

3. Télécharger le binaire de l'agent :
   ```bash
   cd ~/jenkins-agent
   curl -sO https://jenkinstraining.duckdns.org/jnlpJars/agent.jar
   ```

4. Lancer l'agent avec la commande fournie par Jenkins :
   ```bash
   java -jar agent.jar \
     -url https://jenkinstraining.duckdns.org/ \
     -secret 'SECRET_FOURNI_PAR_JENKINS' \
     -name codespace01 \
     -webSocket \
     -workDir "$HOME/jenkins-agent"
   ```

5. Dans Jenkins, vérifier l'état :
   - **`codespace01`** : 🟢 `Connected`

---

## 3. Vérifier Docker sur l'agent

Dans le terminal du Codespace :

1. Vérifier que Docker fonctionne :
   ```bash
   docker --version
   docker ps
   ```

2. Lancer un conteneur Nginx de test :
   ```bash
   docker run -d \
     --name nginx-test \
     -p 8080:80 \
     nginx:alpine
   ```

3. Tester le conteneur :
   ```bash
   curl http://localhost:8080
   ```

4. Nettoyer le test :
   ```bash
   docker rm -f nginx-test
   ```

---

## 4. Créer le Pipeline Jenkins

Dans l'interface Jenkins :

1. **New Item → Pipeline**
2. **Nom :** `nginx-agent`
3. Choisir le type **Pipeline** puis valider.
4. Dans la section **Pipeline → Definition**, sélectionner **Pipeline script**.

---

## 5. Script du Pipeline

Coller le code Groovy suivant :

```groovy
pipeline {

    agent {
        label 'codespace01'
    }

    stages {

        stage('Informations') {
            steps {
                sh '''
                    echo "Agent : $(hostname)"
                    echo "Utilisateur : $(whoami)"
                    echo "Workspace : $(pwd)"
                '''
            }
        }

        stage('Docker') {
            steps {
                sh '''
                    docker --version
                    docker ps
                '''
            }
        }

        stage('Build Image Nginx') {
            steps {
                sh '''
                    cat > Dockerfile <<'EOF'
FROM nginx:alpine

RUN echo '<h1>🚀 Nginx déployé par Jenkins</h1>' \
    > /usr/share/nginx/html/index.html

EXPOSE 80
EOF

                    docker build \
                        -t demo-nginx:1.0 \
                        .
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker rm -f demo-nginx 2>/dev/null || true

                    docker run -d \
                        --name demo-nginx \
                        -p 8080:80 \
                        demo-nginx:1.0

                    docker ps
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    sleep 2

                    curl -I http://localhost:8080

                    echo ""
                    echo "Contenu de la page :"

                    curl http://localhost:8080
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Nginx déployé avec succès !"
        }

        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}
```

---

## 6. Résultat attendu

### Flux d'exécution

```text
Jenkins
   │
   │ Pipeline
   ▼
codespace01
   │
   │ docker build
   ▼
demo-nginx:1.0
   │
   │ docker run
   ▼
demo-nginx
   │
   │ :8080 → :80
   ▼
Nginx
```

### Vérification du résultat

La commande :
```bash
curl http://localhost:8080
```

Doit retourner la réponse HTML suivante :
```html
<h1>🚀 Nginx déployé par Jenkins</h1>
```

---

## 📌 Commandes importantes à retenir

| Action | Commande |
| :--- | :--- |
| **Construire l'image** | `docker build -t demo-nginx:1.0 .` |
| **Lancer le conteneur** | `docker run -d --name demo-nginx -p 8080:80 demo-nginx:1.0` |
| **Lister les conteneurs** | `docker ps` |
| **Tester le service** | `curl http://localhost:8080` |
| **Supprimer le conteneur** | `docker rm -f demo-nginx` |