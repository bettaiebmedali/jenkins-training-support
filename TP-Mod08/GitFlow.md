# Gestion des branches GitLab et GitFlow 🌿

## Contexte / Storytelling
La startup GFP Tech grandit et plusieurs développeurs travaillent simultanément.  
Pour organiser les développements et automatiser les pipelines Jenkins, il est crucial d’adopter **GitFlow** et de gérer correctement les branches dans GitLab.  
Ce TP vous permet de mettre en place un pipeline **multibranch** qui détecte les commits et les Merge Requests pour chaque type de branche.

---

## Objectifs pédagogiques
- Comprendre la **stratégie GitFlow** : feature, develop, release, hotfix, main.  
- Configurer Jenkins pour un **pipeline multibranch**.  
- Automatiser le build et tests selon le type de branche.  
- Déclencher des pipelines sur commits et Merge Requests (MR).  
- Gérer les artefacts et déploiements conditionnels selon la branche.

---

## Prérequis techniques
- Jenkins avec pipelines Chapitre 1 à 7.  
- Projet Maven dans GitLab.  
- Plugin GitLab Branch Source installé sur Jenkins.  
- Droits pour configurer Webhooks GitLab → Jenkins.

---

## Étapes détaillées

### **1. Préparer le repository GitLab**
1. Créer les branches principales selon GitFlow :
   ```bash
   git checkout -b develop
   git checkout -b release/1.0
   git checkout -b hotfix/1.0.1
   git push origin develop release/1.0 hotfix/1.0.1
   ```
2. Créer une feature branch :
   ```bash
   git checkout -b feature/login
   git push origin feature/login
   ```

### **2. Configurer le pipeline multibranch**
1. Dans Jenkins : **New Item → Multibranch Pipeline**.  
2. Configurer le **GitLab Branch Source** :  
   - URL du repo  
   - Credentials GitLab  
   - Discovery : Branches et Merge Requests
3. Jenkins détecte automatiquement toutes les branches et MR et crée des jobs pour chacune.

### **3. Adapter le Jenkinsfile pour GitFlow**
Exemple :

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "🛠️ Build de la branche ${env.BRANCH_NAME}"
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            when {
                branch 'release/*'
            }
            steps {
                echo "🚀 Déploiement pour la branche ${env.BRANCH_NAME}"
                sh "./deploy.sh staging ${env.BUILD_NUMBER}"
            }
        }
    }
}
```

### **4. Déclenchement automatique**
1. Configurer **Webhooks GitLab → Jenkins** pour que chaque commit ou MR déclenche un build.  
2. Vérifier que les pipelines s’exécutent pour :  
   - Feature branches : build + tests  
   - Release branches : build + tests + déploiement staging  
   - Main : build + tests + déploiement prod

### **5. Tester et valider**
1. Créer un commit sur une feature branch et ouvrir un MR.  
2. Observer la création automatique du job Jenkins pour cette MR.  
3. Fusionner le MR dans develop et vérifier l’exécution du pipeline.  
4. Créer un release branch et observer le déploiement automatique sur staging.

---

## Livrable final
- Pipeline Jenkins multibranch fonctionnel.  
- Intégration complète avec GitLab et Webhooks.  
- Pipelines adaptés selon le type de branche GitFlow.  
- Déploiement conditionnel sur release/main.

---

## Bonus (optionnel)
- Ajouter un **stage de notification Slack** pour chaque MR.  
- Ajouter un **test de sécurité ou SonarQube** pour release branches.  
- Déclencher le déploiement Blue-Green pour la branche main.
~~~

