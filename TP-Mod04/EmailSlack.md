# TP -- Jenkins : Notifications Email (Gmail) & Slack

## 🎯 Objectifs du TP

-   Configurer Jenkins pour envoyer des notifications email via Gmail\
-   Créer une notification Slack via un Webhook\
-   Ajouter ces notifications dans un Pipeline `Jenkinsfile`\
-   Tester les notifications en cas de succès ou d'échec

## 🧩 1. Configuration des notifications Email (Gmail)

### 1.1 Préparer Gmail

Google a désactivé les *less secure apps*.\
➡️ Vous devez utiliser un **mot de passe d'application**.

### Étapes :

1.  Aller sur : https://myaccount.google.com\
2.  Menu **Security**\
3.  Activer **2-Step Verification**\
4.  Une fois activée → section **App passwords**\
5.  Choisir :
    -   **App** : Mail\
    -   **Device** : Other (Custom) → écrire *Jenkins*
6.  Copier le mot de passe généré (mot de passe SMTP)

### 1.2 Configurer Jenkins

Aller dans :

**Manage Jenkins → Configure System → Extended E-mail Notification**

Configurer :

  Champ         Valeur
  ------------- ----------------------------
  SMTP server   smtp.gmail.com
  SMTP Port     587
  Use TLS       Oui
  Credentials   Gmail + mot de passe d'app

Section **Email Notification** :

  Champ                       Valeur
  --------------------------- ---------------------
  SMTP server                 smtp.gmail.com
  Default user email suffix   @gmail.com
  Reply-To address            votre adresse Gmail

## 🧩 2. Configuration des notifications Slack

### 2.1 Créer un Webhook Slack

1.  Aller sur : https://api.slack.com/messaging/webhooks\
2.  Create a new Slack App\
3.  App Name : JenkinsNotifications\
4.  Choisir workspace\
5.  Activer Incoming Webhooks\
6.  Add New Webhook to Workspace\
7.  Choisir un channel → #jenkins\
8.  Copier le webhook

### 2.2 Ajouter la clé dans Jenkins Credentials

**Manage Jenkins → Credentials → Global → Add Credential**

  Champ    Valeur
  -------- ---------------
  Kind     Secret Text
  Secret   Webhook Slack
  ID       slack-webhook

## 🧩 3. Jenkinsfile -- Email + Slack

``` groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Compilation en cours...'
            }
        }
        stage('Tests') {
            steps {
                echo 'Tests en cours...'
            }
        }
    }

    post {
        success {
            echo "Build success"
            emailext(
                to: "yourMail@gmail.com",
                subject: "✔ SUCCESS - Build Jenkins",
                body: "Le build est passé au vert 👍"
            )
            slackNotification("✔️ Build SUCCESS")
        }
        failure {
            echo "Build failed"
            emailext(
                to: "yourMail@gmail.com",
                subject: "❌ FAILURE - Build Jenkins",
                body: "Le build a échoué ❗"
            )
            slackNotification("❌ Build FAILED")
        }
    }
}

def slackNotification(String message) {
    withCredentials([string(credentialsId: 'slackWebhook', variable: 'SLACK_WEBHOOK')]) {
        sh """
        curl -X POST -H 'Content-type: application/json' \
        --data '{"text": "${message}"}' $SLACK_WEBHOOK
        """
    }
}

```

## 🧪 4. Tester les notifications

### Test échec

-   Modifier un test JUnit pour le faire échouer volontairement

### Test succès

-   Rebuild

Résultats :

-   Succès → 1 email + 1 Slack\
-   Échec → 1 email + 1 Slack
