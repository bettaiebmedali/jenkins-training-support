# Automatisation des tests 🧪

## Contexte / Storytelling
La startup GFP Tech souhaite garantir la **qualité du code** sur tous les pipelines.  
Après avoir mis en place les pipelines et la mutualisation, il est temps d’automatiser **les tests unitaires, d’intégration et de performance** pour détecter les erreurs rapidement et améliorer la fiabilité du projet.

---

## Objectifs pédagogiques
- Ajouter des tests unitaires et d’intégration automatisés dans le pipeline Jenkins.  
- Générer et visualiser des **rapports de tests**.  
- Mesurer la **couverture de code**.  
- Ajouter des tests de performance JMeter et suivre leur résultat.  
- Optimiser l’exécution des tests pour un pipeline efficace.

---

## Prérequis techniques
- Jenkins avec pipelines précédents configurés.  
- Projet Maven Java avec tests unitaires existants.  
- Jenkins Plugin **JUnit** et **Performance** installé.  
- JMeter installé (local ou sur agent Jenkins).

---

## Étapes détaillées

### **1. Ajouter les tests unitaires et d’intégration**
1. Créer ou vérifier les tests JUnit dans `src/test/java`.  
2. Pour les tests d’intégration, ajouter des tests dans `src/integration-test/java`.  
3. Ajouter un profil Maven pour différencier tests unitaires et d’intégration :
   ```xml
   <profiles>
       <profile>
           <id>integration-test</id>
           <build>
               <plugins>
                   <plugin>
                       <groupId>org.apache.maven.plugins</groupId>
                       <artifactId>maven-surefire-plugin</artifactId>
                   </plugin>
               </plugins>
           </build>
       </profile>
   </profiles>
   ```

### **2. Modifier le pipeline pour exécuter les tests**
Dans le `Jenkinsfile` :

```groovy
stage('Unit Tests') {
    steps {
        sh 'mvn test'
    }
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}

stage('Integration Tests') {
    steps {
        sh 'mvn verify -Pintegration-test'
    }
    post {
        always {
            junit '**/target/failsafe-reports/*.xml'
        }
    }
}
```

### **3. Ajouter analyse de couverture**
1. Ajouter le plugin **JaCoCo** dans le `pom.xml`.  
2. Ajouter un stage Jenkins pour générer le rapport de couverture :

```groovy
stage('Code Coverage') {
    steps {
        sh 'mvn jacoco:report'
        publishHTML(target: [
            reportName: 'Coverage Report',
            reportDir: 'target/site/jacoco',
            reportFiles: 'index.html',
            keepAll: true
        ])
    }
}
```

### **4. Ajouter tests de performance avec JMeter**
1. Créer un test JMeter (`.jmx`) pour le projet.  
2. Ajouter un stage Jenkins pour exécuter le test :

```groovy
stage('Performance Test') {
    steps {
        sh 'jmeter -n -t tests/performance/test.jmx -l results.jtl'
        perfReport sourceDataFiles: 'results.jtl'
    }
}
```

### **5. Exécuter et vérifier**
1. Déclencher le pipeline et vérifier que toutes les étapes passent.  
2. Consulter les rapports : JUnit, JaCoCo, JMeter.  
3. Observer les résultats dans Jenkins et identifier les problèmes éventuels.

---

## Livrable final
- Pipeline Jenkins exécutant :  
  - Tests unitaires et d’intégration  
  - Génération de rapports JUnit  
  - Analyse de couverture avec JaCoCo  
  - Tests de performance JMeter  
- Rapport complet visible dans Jenkins.

---

## Bonus (optionnel)
- Ajouter parallélisation des tests unitaires et d’intégration.  
- Déclencher tests de performance uniquement sur certaines branches (ex: release/main).  
- Ajouter notifications Slack pour les tests échoués.
~~~

