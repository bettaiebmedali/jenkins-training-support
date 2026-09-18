# TP Pipeline simple 🚀

## Contexte / Storytelling
Bienvenue dans la startup GFP Tech !  
Votre équipe commence tout juste à travailler sur un projet Java/Maven. L'objectif est de mettre en place **le premier pipeline CI/CD** pour automatiser le build et les tests unitaires.  
Cette première étape vous permettra de comprendre les bases de Jenkins, de créer un `Jenkinsfile` et d’exécuter votre pipeline.

---

## Objectifs pédagogiques
- Découvrir Jenkins et l’interface utilisateur.  
- Comprendre les concepts de **pipeline déclaratif**.  
- Créer et exécuter un pipeline simple.  
- Visualiser les logs et les résultats des builds.  
- Vérifier le succès des tests unitaires Maven.

---

## Prérequis techniques
- Jenkins installé et accessible (Master).  
- GitLab ou autre repo Git avec un projet Maven simple.  
- Maven installé sur le Master ou dans l’agent Jenkins.  
- Accès au terminal ou interface Jenkins.

---

## Étapes détaillées

### **1. Préparer le projet**
1. Créer un nouveau projet Maven Java sur GitLab :
   ```bash
   mvn archetype:generate -DgroupId=com.gfptech -DartifactId=demo-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```
2. Initialiser le dépôt Git :
   ```bash
   cd demo-app
   git init
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```

### **2. Créer un Jenkinsfile**
1. À la racine du projet, créer un fichier `Jenkinsfile` :
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Build') {
               steps {
                   echo '🛠️ Build en cours...'
                   sh 'mvn clean compile'
               }
           }
           stage('Test') {
               steps {
                   echo '✅ Tests unitaires...'
                   sh 'mvn test'
               }
           }
       }
   }
   ```
2. Commiter et pousser le Jenkinsfile dans GitLab :
   ```bash
   git add Jenkinsfile
   git commit -m "Add Jenkinsfile"
   git push
   ```

### **3. Créer un pipeline Jenkins**
1. Dans Jenkins, créer un **nouveau pipeline multibranch** ou simple lié au repo GitLab.  
2. Configurer l’URL du repo et les identifiants si nécessaire.  
3. Détecter automatiquement le `Jenkinsfile` et sauvegarder la configuration.

### **4. Exécuter le pipeline**
1. Cliquer sur **Build Now** pour déclencher le pipeline.  
2. Observer les étapes : Build → Test.  
3. Vérifier les logs pour chaque étape :
   - Compilation Maven  
   - Résultat des tests unitaires  

### **5. Modifier et rejouer**
1. Ajouter un nouveau test unitaire simple dans `src/test/java`.  
2. Commit et push les modifications.  
3. Observer que le pipeline se déclenche automatiquement (si Webhooks activés).  

---

## Livrable final
- Pipeline Jenkins fonctionnel qui :  
  - Compile le projet Maven  
  - Exécute les tests unitaires  
  - Affiche les logs correctement  
- Projet GitLab avec `Jenkinsfile` à jour.

---

## Bonus (optionnel)
- Ajouter un **stage “Reports”** pour générer un rapport JUnit :
   ```groovy
   stage('Reports') {
       steps {
           junit '**/target/surefire-reports/*.xml'
       }
   }
   ```
- Observer les résultats directement dans Jenkins.
~~~

