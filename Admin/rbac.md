# Gestion des Utilisateurs, Rôles et Sécurité Jenkins (Détaillé)

## 🎯 Objectifs
- Sécuriser Jenkins avec une authentification
- Créer des utilisateurs internes
- Installer et configurer le plugin RBAC (Role-Based Authorization Strategy)
- Créer plusieurs rôles : admin, dev, viewer
- Donner des permissions granulaire aux utilisateurs
- Tester la configuration en se connectant avec différents comptes

---

# 1️⃣ Activation de la sécurité Jenkins

## Étape 1 : Activer la base utilisateurs Jenkins
Navigation :  
**Manage Jenkins → Configure Global Security**

Cocher :

- [x] *Jenkins’ own user database*  
- [x] *Allow users to sign up* (optionnel)
- [x] *Require authentication to access Jenkins*

## Étape 2 : Définir une stratégie de sécurité
Sélectionner :


Authorization: Matrix-based security


Exemple permissions minimum pour l’admin :


Overall → Administer


⛔ Important : **ne jamais retirer Administer à l’utilisateur admin avant test !**

---

# 2️⃣ Création d’utilisateurs

Chemin :
**Manage Jenkins → Manage Users → Create User**

Créer 3 utilisateurs :

1. **admin01**
2. **dev01**
3. **viewer01**

---

# 3️⃣ Installation du plugin RBAC

Chemin :
**Manage Jenkins → Manage Plugins → Available**

Rechercher :


Role-based Authorization Strategy


Installer + redémarrer Jenkins.

---

# 4️⃣ Configuration RBAC

Chemin :
**Manage Jenkins → Manage and Assign Roles → Manage Roles**

Créer les rôles :

## Rôle admin


Overall: Administer
Job: *
View: *
Credentials: *
Agent: *


## Rôle dev (permissions limitées)


Overall → Read
Job → Build
Job → Configure
Job → Read
Job → Workspace
Credentials → Use


## Rôle viewer (lecture seule)


Overall → Read
Job → Read
View → Read


---

# 5️⃣ Assignation des rôles

Chemin :
**Manage and Assign Roles → Assign Roles**

Assigner :

- admin01 → admin
- dev01 → dev
- viewer01 → viewer

---

# 6️⃣ Tests des comptes

## Test compte viewer01
Doit pouvoir :
- voir les jobs
Ne doit pas pouvoir :
- lancer un build
- modifier un job

## Test compte dev01
Doit pouvoir :
- lancer un job
- accéder aux logs
Ne doit pas pouvoir :
- modifier les credentials
- gérer les agents
- installer plugins

## Test admin01
Doit pouvoir tout faire.

---

# 7️⃣ Sauvegarde de la configuration sécurité

Dans :


/var/lib/jenkins/config.xml


Vérifier :

<authorizationStrategy class="...RoleBasedStrategy"> ```
Conclusion

Vous avez configuré :

la sécurité Jenkins

les rôles

les permissions précises

des utilisateurs avec accès différencié

Ce TP prépare l’administration à grande échelle.


---

# ✅ **TP4 – Gestion des Nodes et Exécution Distribuée (Version Très Détaillée)**

```md
# TP4 – Gestion des Nodes Jenkins (Détaillé)

## 🎯 Objectifs
- Ajouter un agent Jenkins (Linux)
- Utiliser l’agent via SSH
- Installer les outils nécessaires
- Exécuter un pipeline sur un agent spécifique
- Diagnostiquer les erreurs de communication (agent offline)

---

# 1️⃣ Préparation de la machine agent

Sur le serveur agent :

### Installer Java obligatoire
```bash
sudo apt update
sudo apt install -y openjdk-17-jdk

Créer un utilisateur Jenkins
sudo useradd -m -s /bin/bash jenkins
sudo passwd jenkins

Autoriser sudo (optionnel)
sudo usermod -aG sudo jenkins

2️⃣ Ajouter une clé SSH pour l’agent

Sur le MASTER Jenkins :

Manage Jenkins → Credentials → System → Global → Add Credentials


Type :

SSH Username with private key


Username : jenkins

Private Key : (clé privée)

ID : jenkins-agent-key

3️⃣ Ajouter un nouvel agent dans Jenkins

Chemin :

Manage Jenkins → Nodes → New Node


Paramètres :

Nom : linux-agent-01
Type : Permanent Agent

Paramètres recommandés :
# of executors : 2
Remote root directory : /home/jenkins
Labels : linux docker build
Usage : Only build jobs with label restrictions
Launch method : Launch agents via SSH

4️⃣ Tester la connexion SSH depuis le master

Depuis Jenkins master :

ssh -i /path/key jenkins@AGENT_IP


Si OK → continuer
Si NOK → vérifier :

Firewall

SSHD

Droits sur ~/.ssh

5️⃣ Pipeline exécuté sur l’agent
pipeline {
  agent { label 'linux' }

  stages {
    stage('Check Agent') {
      steps {
        sh 'hostname'
        sh 'whoami'
        sh 'java -version'
      }
    }
  }
}

6️⃣ Débogage Agent Offline
Vérifier le log de l’agent :
Manage Jenkins → Nodes → linux-agent-01 → Logs

Causes fréquentes :

Java absent

Mauvaise clé SSH

Port 22 bloqué

Droits sur /home/jenkins incorrects

Conclusion

Agent Jenkins installé, configuré et utilisé dans un pipeline distribué.


---

# ✅ **TP7 – Monitoring Jenkins (Version Très Détaillée)**

```md
# TP7 – Monitoring Jenkins (Détaillé)

## 🎯 Objectifs
- Installer les plugins de monitoring
- Suivre CPU, RAM, Thread pools
- Configurer une politique d’alertes
- Mettre en place un parsing des logs
- Comprendre l'état de santé Jenkins

---

# 1️⃣ Installer les plugins de monitoring

Plugins nécessaires :
- **Monitoring**
- **Metrics**
- **Prometheus Metrics**
- **Health Advisor by CloudBees**
- **Log Parser Plugin**

Installation :  
Manage Jenkins → Manage Plugins → Available

---

# 2️⃣ Monitoring Plugin

Une fois installé :


Dashboard → Monitoring


Indicateurs affichés :
- CPU usage (master + agents)
- Heap memory
- Non-heap memory
- Garbage Collector
- Thread usage
- File descriptor usage
- Load average

---

# 3️⃣ Log Parser – Analyse des logs

Créer un fichier de règles :


ERROR\s.*
WARN\s.*
Exception
Traceback


Uploader :


Manage Jenkins → Log Parser → Create a new rule


Puis dans un job :


Post-build Action → Console output parsing


---

# 4️⃣ Health Advisor by CloudBees

Fonctionnalités :
- analyse automatique Jenkins
- recommandation de sécurité
- détection de jobs obsolètes
- détection d’agents saturés

Chemin :


Manage Jenkins → Health Advisor by CloudBees


---

# 5️⃣ Exposition des métriques Prometheus

Endpoint :


http://JENKINS:8080/prometheus/


Ajouter dans Prometheus :


scrape_configs:

job_name: 'jenkins'
static_configs:

targets: ['IP:8080']


---

# 6️⃣ Alertes recommandées

- CPU > 80%
- Heap Memory > 70%
- Queue Size > 50 jobs
- Executor Saturation > 90%
- Agent offline > 5 minutes

---

# Conclusion
Un Jenkins bien monitoré évite la plupart des pannes (OOM, saturation, agents offline).

✅ TP11 – Shared Library Avancée (Très Détaillé)
# TP11 – Shared Library (Détaillé)

## 🎯 Objectifs
- Créer une Shared Library complète
- Ajouter plusieurs fonctions globales
- Ajouter une classe utilitaire avancée
- Importer la librairie dans un pipeline
- Tester l’intégration

---

# 1️⃣ Structure complète du repo Shared Library



jenkins-shared-library/
├── vars/
│ ├── sayHello.groovy
│ ├── buildApp.groovy
│ ├── sendNotification.groovy
└── src/org/example/
├── Utils.groovy
└── DockerHelper.groovy


---

# 2️⃣ Fichier vars/sayHello.groovy

```groovy
def call(String name = 'World') {
  echo "Hello, ${name}!"
}

3️⃣ Fichier vars/buildApp.groovy
def call() {
  echo "Building app..."
  sh 'mvn clean package'
}

4️⃣ src/org/example/Utils.groovy
package org.example

class Utils {
  static String formatDate() {
    return new Date().format("yyyy-MM-dd HH:mm:ss")
  }

  static void printBanner(String msg) {
    println "========================"
    println " ${msg}"
    println "========================"
  }
}

5️⃣ Pipeline utilisant la Shared Library
@Library('jenkins-shared-library') _

pipeline {
  agent any

  stages {
    stage("Test Library") {
      steps {
        script {
          sayHello("DevOps Student")

          def utils = new org.example.Utils()
          utils.printBanner("Starting build...")

          buildApp()

          echo "Time: ${utils.formatDate()}"
        }
      }
    }
  }
}

Conclusion

Shared Library avancée = code réutilisable, pipelines plus courts, meilleur versionning.


---

# ✅ **TP14 – Gestion des Secrets avec Vault / SOPS / Credentials (Très Détaillé)**

```md
# TP14 – Gestion des Secrets (Vault / SOPS / Credentials)

## 🎯 Objectifs
- Comprendre les stratégies de gestion des secrets
- Intégrer Vault avec Jenkins
- Utiliser des secrets dynamiques
- Intégrer les secrets Jenkins dans un pipeline

---

# 1️⃣ Installation HashiCorp Vault (Dev Mode)

```bash
vault server -dev
export VAULT_ADDR=http://127.0.0.1:8200

2️⃣ Créer un secret Vault
vault kv put secret/app api_key="123456789" password="s3cr3t"

3️⃣ Configuration du plugin Vault dans Jenkins

Chemin :

Manage Jenkins → Configure System → HashiCorp Vault


Configurer :

Vault URL

Token

Engine Version : KV2

4️⃣ Pipeline utilisant Vault
pipeline {
  agent any

  environment {
    VAULT_ADDR = "http://127.0.0.1:8200"
  }

  stages {
    stage("Read secrets") {
      steps {
        withVault(
          configuration: [
            vaultUrl: "http://127.0.0.1:8200",
            credentialsId: "vault-token"
          ],
          vaultSecrets: [
            [path: "secret/data/app", secretValues: [
              [envVar: 'API_KEY', vaultKey: 'api_key'],
              [envVar: 'PASSWORD', vaultKey: 'password']
            ]]
          ]
        ) {
          sh 'echo "Key: $API_KEY"'
          sh 'echo "Password: $PASSWORD"'
        }
      }
    }
  }
}

5️⃣ Secrets dynamiques (exemple MySQL)

Activer le backend DB :

vault secrets enable database


Vault va générer :

un login unique

un mot de passe unique

valides 1 heure

6️⃣ SOPS (si Kubernetes / GitOps)

Exemple fichier chiffré :

api_key: ENC[AES256_GCM,...]

Conclusion

Ce TP introduit les secrets sécurisés et l’intégration Vault dans les pipelines.


---

Si tu veux, je peux aussi créer :

✅ **un dossier GitHub complet** contenant tous ces TP  
ou  
✅ **un PDF par TP**  
ou  
✅ **une version corrigée + un TP d’évaluation**  

Souhaites-tu la suite ?