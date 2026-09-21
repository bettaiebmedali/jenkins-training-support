# Agents Jenkins 🤖

## Contexte / Storytelling
La startup GFP Tech voit son nombre de pipelines augmenter et ses builds devenir plus lourds.  
Pour gérer la charge et **optimiser les temps de build**, il est nécessaire d’installer et configurer des **agents Jenkins distants**.  
Ces agents permettent d’isoler les builds, répartir la charge et garantir la stabilité de Jenkins.

---

## Objectifs pédagogiques
- Installer et configurer un agent Jenkins distant.  
- Exécuter des pipelines sur des agents spécifiques.  
- Comprendre la répartition des ressources et la scalabilité.  
- Tester la communication et sécurité entre master et agent.

---

## Prérequis techniques
- Jenkins Master fonctionnel avec pipelines Chapitre 1 à 6.  
- Accès SSH à une machine distante (VM ou container).  
- Java et Docker installés sur l’agent (si besoin).  
- Droits administrateur Jenkins pour configurer les agents.

---

## Étapes détaillées

### **1. Préparer la machine agent**
1. Installer Java sur l’agent :  
   ```bash
   sudo apt update && sudo apt install openjdk-17-jdk -y
   ```
2. Créer un utilisateur dédié `jenkins` sur l’agent :  
   ```bash
   sudo adduser jenkins
   ```
3. Configurer les clés SSH pour l’accès Jenkins Master → Agent.

### **2. Ajouter l’agent dans Jenkins**
1. Aller dans **Manage Jenkins → Manage Nodes and Clouds → New Node**.  
2. Configurer :  
   - Nom : `agent1`  
   - Type : Permanent Agent  
   - Remote root directory : `/home/jenkins`  
   - Launch method : **Launch agents via SSH**  
   - Credentials : utilisateur `jenkins` avec clé SSH

### **3. Configurer l’exécution de pipelines sur l’agent**
1. Dans un `Jenkinsfile`, spécifier l’agent :  
   ```groovy
   pipeline {
       agent { label 'agent1' }
       stages {
           stage('Build') {
               steps {
                   sh 'mvn clean compile'
               }
           }
       }
   }
   ```
2. Déclencher le pipeline et vérifier qu’il s’exécute sur l’agent distant.  

### **4. Tester la scalabilité et isolation**
1. Créer plusieurs jobs ou multibranch pipelines.  
2. Observer la répartition sur l’agent : parallélisation et consommation de ressources.  
3. Installer un deuxième agent et configurer des labels pour répartir certains jobs.

### **5. Intégration avec le projet fil rouge**
1. S’assurer que tous les pipelines existants (build, tests, déploiement) peuvent s’exécuter sur les agents.  
2. Vérifier les logs et les artefacts pour chaque exécution.

---

## Livrable final
- Un ou plusieurs agents Jenkins configurés et connectés au Master.  
- Pipelines exécutés avec succès sur les agents distants.  
- Répartition des builds efficace et isolation des environnements.

---

## Bonus (optionnel)
- Configurer des agents **Docker** pour exécuter des builds dans des containers éphémères.  
- Ajouter un agent temporaire pour tests de charge.  
- Surveiller les agents via plugin **Monitoring** pour détecter surcharge ou erreurs.
~~~

