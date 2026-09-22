````md
# 🧪 TP — Automatiser les tests avec Jenkins

## Projet : Spring PetClinic

### Java 21 + Maven + JUnit + JMeter

---

# 🎯 Objectif

Dans ce TP, nous allons partir d'un **projet Java existant**.

Nous allons utiliser :

- Java 21
- Maven
- Spring Boot
- JUnit
- JMeter
- Jenkins

L'objectif est de construire progressivement ce pipeline :

```text
        Git
         │
         ▼
   Récupération du projet
         │
         ▼
       Maven
         │
         ▼
     Tests JUnit
         │
         ▼
   Rapport JUnit
         │
         ▼
      JMeter
         │
         ▼
Test de performance
```

---

# 🏗️ 1. Le projet utilisé

Nous allons utiliser le projet **Spring PetClinic**.

Repository :

```text
https://github.com/spring-projects/spring-petclinic
```

Le projet est une application Spring Boot existante avec Maven et des tests déjà présents.

L'application peut être démarrée avec Maven et accessible sur :

```text
http://localhost:8080
```

---

# ☕ 2. Vérifier Java 21

Avant de commencer, vérifier Java :

```bash
java -version
```

Nous voulons Java 21.

Exemple :

```text
openjdk version "21.x.x"
```

Vérifier également :

```bash
javac -version
```

Résultat attendu :

```text
javac 21.x.x
```

---

# 📦 3. Vérifier Git

```bash
git --version
```

Exemple :

```text
git version 2.x.x
```

---

# 🧰 4. Vérifier Maven

```bash
mvn -version
```

Vérifier surtout la ligne Java :

```text
Java version: 21
```

Si Maven utilise une autre version de Java, il faudra corriger `JAVA_HOME`.

Vérifier :

```bash
echo $JAVA_HOME
```

Puis :

```bash
which java
```

---

# 📥 5. Télécharger le projet

Créer un répertoire de travail :

```bash
mkdir -p ~/formation-jenkins
cd ~/formation-jenkins
```

Cloner le projet :

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
```

Entrer dans le projet :

```bash
cd spring-petclinic
```

Vérifier :

```bash
ls
```

Vous devez voir notamment :

```text
pom.xml
src
mvnw
mvnw.cmd
```

---

# 🔎 6. Comprendre le projet

Avant de lancer quoi que ce soit, regarder le projet :

```bash
ls src
```

Vous devez avoir notamment :

```text
main
test
```

Les sources de l'application sont dans :

```text
src/main
```

Les tests sont dans :

```text
src/test
```

Le fichier Maven est :

```text
pom.xml
```

---

# 🧪 7. Première étape : exécuter les tests localement

Nous allons d'abord vérifier que les tests fonctionnent **sans Jenkins**.

Depuis la racine du projet :

```bash
./mvnw test
```

Si `mvnw` n'est pas exécutable :

```bash
chmod +x mvnw
```

Puis :

```bash
./mvnw test
```

---

# ⏳ 8. Observer Maven

Maven va automatiquement :

```text
1. Télécharger les dépendances
        ↓
2. Compiler le projet
        ↓
3. Compiler les tests
        ↓
4. Exécuter les tests
        ↓
5. Générer les rapports
```

À la fin, nous voulons :

```text
BUILD SUCCESS
```

---

# 📊 9. Vérifier les résultats des tests

Maven génère les résultats dans :

```text
target/surefire-reports/
```

Afficher le répertoire :

```bash
ls target/surefire-reports
```

Vous devez retrouver des fichiers XML de tests.

Par exemple :

```text
TEST-....xml
```

Ces fichiers seront très importants pour Jenkins.

---

# ▶️ 10. Démarrer l'application

Maintenant nous allons démarrer PetClinic.

Exécuter :

```bash
./mvnw spring-boot:run
```

Attendre que Spring Boot démarre.

Dans les logs, rechercher quelque chose ressemblant à :

```text
Tomcat started on port 8080
```

L'application est maintenant disponible sur :

```text
http://localhost:8080
```

---

# 🌐 11. Tester l'application

Ouvrir dans le navigateur :

```text
http://localhost:8080
```

Vous devez voir l'application PetClinic.

Vous pouvez également tester avec :

```bash
curl http://localhost:8080
```

Vous devez obtenir une réponse HTTP.

---

# 🛑 12. Arrêter l'application

Pour l'instant, arrêter Spring Boot avec :

```text
CTRL + C
```

Nous allons plus tard demander à Jenkins de démarrer l'application automatiquement.

---

# 🧪 13. Comprendre JUnit

Le projet contient déjà des tests.

Chercher les tests :

```bash
find src/test -type f | head
```

Vous allez trouver plusieurs classes de tests.

JUnit permet de vérifier automatiquement que le comportement de l'application est correct.

Le principe est :

```text
Code Java
    ↓
JUnit
    ↓
Tests
    ↓
PASS / FAIL
```

---

# 💥 14. Faire volontairement échouer un test

Cette étape permet de comprendre ce que Jenkins devra détecter.

Choisir un test existant dans :

```text
src/test
```

Observer son contenu.

L'objectif n'est pas de modifier définitivement le projet.

Nous allons simplement comprendre le principe :

```text
Test réussi
    ↓
PASS

Test échoué
    ↓
FAIL
```

---

# 🔄 15. Remettre le projet dans son état initial

Comme nous utilisons un projet existant, nous pouvons toujours revenir à l'état Git initial :

```bash
git status
```

Si nous avons modifié des fichiers :

```bash
git restore .
```

Puis :

```bash
git status
```

Le projet doit être propre.

---

# 🚀 16. Première étape Jenkins

Maintenant nous allons demander à Jenkins d'exécuter exactement ce que nous avons testé manuellement.

Notre première commande sera :

```bash
./mvnw test
```

Pourquoi ?

Parce que nous savons déjà qu'elle fonctionne.

---

# 📄 17. Créer le Jenkinsfile

À la racine du projet :

```bash
touch Jenkinsfile
```

Ouvrir :

```text
Jenkinsfile
```

Ajouter :

```groovy
pipeline {

    agent any

    stages {

        stage('Tests JUnit') {

            steps {

                sh './mvnw test'
            }
        }
    }
}
```

---

# 🔎 18. Comprendre ce pipeline

Le pipeline contient une seule étape :

```text
Tests JUnit
```

Jenkins exécute :

```bash
./mvnw test
```

Donc :

```text
Jenkins
   ↓
Maven
   ↓
JUnit
   ↓
Tests
```

---

# 📊 19. Publier les résultats JUnit

Nous allons maintenant demander à Jenkins d'afficher les résultats.

Modifier le `Jenkinsfile` :

```groovy
pipeline {

    agent any

    stages {

        stage('Tests JUnit') {

            steps {

                sh './mvnw test'
            }

            post {

                always {

                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

La ligne :

```groovy
junit 'target/surefire-reports/*.xml'
```

dit à Jenkins :

> Cherche les rapports XML générés par Maven et affiche-les comme résultats de tests.

---

# 📤 20. Mettre le Jenkinsfile dans Git

Vérifier :

```bash
git status
```

Ajouter le fichier :

```bash
git add Jenkinsfile
```

Créer un commit :

```bash
git commit -m "add Jenkins pipeline"
```

Envoyer le changement :

```bash
git push
```

---

# ⚙️ 21. Créer le Pipeline dans Jenkins

Dans Jenkins :

```text
New Item
```

Nom :

```text
spring-petclinic-tests
```

Choisir :

```text
Pipeline
```

Puis :

```text
Pipeline
    ↓
Definition
    ↓
Pipeline script from SCM
```

Choisir :

```text
Git
```

Indiquer l'URL du repository :

```text
https://github.com/spring-projects/spring-petclinic.git
```

Puis :

```text
Script Path
```

mettre :

```text
Jenkinsfile
```

Sauvegarder.

---

# ▶️ 22. Lancer Jenkins

Cliquer :

```text
Build Now
```

Jenkins doit :

```text
1. Cloner le projet
        ↓
2. Lire Jenkinsfile
        ↓
3. Exécuter ./mvnw test
        ↓
4. Récupérer les résultats JUnit
```

---

# 📊 23. Voir les résultats JUnit

Ouvrir le build Jenkins.

Chercher :

```text
Test Result
```

Cliquer dessus.

Vous devez retrouver les tests exécutés par le projet.

Vous pouvez voir :

```text
Tests
Failures
Errors
Skipped
```

🎯 Nous avons maintenant notre première partie :

```text
Jenkins + Maven + JUnit
```

---

# ⚡ 24. Ajouter JMeter

Maintenant nous allons tester les performances de l'application.

Vérifier JMeter :

```bash
jmeter --version
```

Si la commande fonctionne, nous pouvons continuer.

---

# 🚀 25. Démarrer PetClinic

Pour JMeter, l'application doit être démarrée.

Dans un terminal :

```bash
./mvnw spring-boot:run
```

Attendre :

```text
Tomcat started on port 8080
```

Tester :

```bash
curl http://localhost:8080
```

Si vous obtenez une réponse, l'application fonctionne.

---

# 📈 26. Créer le test JMeter

Lancer JMeter :

```bash
jmeter
```

Dans JMeter :

```text
Test Plan
   ↓
Add
   ↓
Threads (Users)
   ↓
Thread Group
```

Configurer :

```text
Number of Threads : 10
Ramp-Up Period    : 5
Loop Count        : 5
```

Cela signifie :

```text
10 utilisateurs
    ↓
arrivent progressivement
    ↓
chaque utilisateur réalise 5 appels
```

---

# 🌐 27. Ajouter une requête HTTP

Sur :

```text
Thread Group
```

faire :

```text
Add
   ↓
Sampler
   ↓
HTTP Request
```

Configurer :

```text
Protocol : http
Server  : localhost
Port    : 8080
Path    : /
```

Nous demandons donc à JMeter :

```text
GET http://localhost:8080/
```

---

# 📊 28. Ajouter un rapport JMeter

Sur :

```text
Thread Group
```

faire :

```text
Add
   ↓
Listener
   ↓
Summary Report
```

Vous pouvez également ajouter :

```text
View Results Tree
```

pour observer les requêtes.

---

# 💾 29. Sauvegarder le test JMeter

Créer dans le projet :

```bash
mkdir -p performance
```

Dans JMeter :

```text
File
 ↓
Save Test Plan As
```

Enregistrer sous :

```text
performance/petclinic-test.jmx
```

---

# ▶️ 30. Tester JMeter manuellement

Avant Jenkins, testons JMeter localement.

Arrêter l'interface graphique JMeter si nécessaire.

Puis exécuter :

```bash
jmeter \
    -n \
    -t performance/petclinic-test.jmx \
    -l performance/results.jtl
```

Explication :

```text
-n
mode non graphique

-t
fichier de test JMeter

-l
fichier de résultats
```

---

# 📄 31. Vérifier le résultat

Exécuter :

```bash
ls performance
```

Vous devez avoir :

```text
petclinic-test.jmx
results.jtl
```

Le fichier :

```text
results.jtl
```

contient les résultats du test.

---

# 🤖 32. Ajouter JMeter au Jenkinsfile

Nous allons maintenant automatiser le test JMeter.

Modifier le `Jenkinsfile` :

```groovy
pipeline {

    agent any

    stages {

        stage('Tests JUnit') {

            steps {

                sh './mvnw test'
            }

            post {

                always {

                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Performance Test') {

            steps {

                sh '''
                    jmeter \
                    -n \
                    -t performance/petclinic-test.jmx \
                    -l performance/results.jtl
                '''
            }
        }
    }
}
```

---

# ⚠️ 33. Attention : l'application doit être démarrée

Notre test JMeter appelle :

```text
http://localhost:8080
```

Donc l'application doit être démarrée.

Pour comprendre simplement le principe :

```text
PetClinic
    ↓
localhost:8080
    ↑
    │
JMeter
```

Si PetClinic n'est pas démarrée :

```text
JMeter
   ↓
localhost:8080
   ↓
❌ connexion impossible
```

---

# 🛠️ 34. Faire démarrer PetClinic par Jenkins

Nous allons demander à Jenkins de démarrer l'application avant JMeter.

Modifier le pipeline :

```groovy
pipeline {

    agent any

    stages {

        stage('Tests JUnit') {

            steps {

                sh './mvnw test'
            }

            post {

                always {

                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Start Application') {

            steps {

                sh '''
                    nohup ./mvnw spring-boot:run > petclinic.log 2>&1 &
                '''

                sh '''
                    for i in {1..30}
                    do
                        if curl -s http://localhost:8080 > /dev/null
                        then
                            echo "PetClinic is ready"
                            exit 0
                        fi

                        echo "Waiting for PetClinic..."
                        sleep 2
                    done

                    echo "PetClinic did not start"
                    exit 1
                '''
            }
        }

        stage('Performance Test') {

            steps {

                sh '''
                    jmeter \
                    -n \
                    -t performance/petclinic-test.jmx \
                    -l performance/results.jtl
                '''
            }
        }
    }
}
```

---

# 🔎 35. Comprendre Start Application

Cette commande :

```bash
nohup ./mvnw spring-boot:run > petclinic.log 2>&1 &
```

démarre l'application en arrière-plan.

Les logs sont écrits dans :

```text
petclinic.log
```

Puis Jenkins attend que :

```text
http://localhost:8080
```

réponde.

---

# 🧪 36. Comprendre le pipeline complet

Notre pipeline fait maintenant :

```text
┌──────────────────┐
│     Checkout     │
└────────┬─────────┘
         ↓
┌──────────────────┐
│    Tests JUnit   │
└────────┬─────────┘
         ↓
┌──────────────────┐
│   Rapport JUnit  │
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Start PetClinic  │
└────────┬─────────┘
         ↓
┌──────────────────┐
│   JMeter Test    │
└──────────────────┘
```

---

# 📊 37. Ajouter les résultats JMeter comme artefact

Nous pouvons demander à Jenkins de conserver :

```text
performance/results.jtl
```

Modifier le pipeline :

```groovy
post {

    always {

        junit 'target/surefire-reports/*.xml'

        archiveArtifacts artifacts: 'performance/results.jtl',
                           allowEmptyArchive: true
    }
}
```

Le fichier sera alors disponible dans Jenkins.

---

# 🏁 38. Pipeline final

Le `Jenkinsfile` final peut être :

```groovy
pipeline {

    agent any

    stages {

        stage('Tests JUnit') {

            steps {

                sh './mvnw test'
            }
        }

        stage('Start Application') {

            steps {

                sh '''
                    nohup ./mvnw spring-boot:run > petclinic.log 2>&1 &
                '''

                sh '''
                    for i in {1..30}
                    do
                        if curl -s http://localhost:8080 > /dev/null
                        then
                            echo "PetClinic is ready"
                            exit 0
                        fi

                        echo "Waiting for PetClinic..."
                        sleep 2
                    done

                    echo "PetClinic did not start"
                    exit 1
                '''
            }
        }

        stage('Performance Test') {

            steps {

                sh '''
                    mkdir -p performance

                    jmeter \
                    -n \
                    -t performance/petclinic-test.jmx \
                    -l performance/results.jtl
                '''
            }
        }
    }

    post {

        always {

            junit 'target/surefire-reports/*.xml'

            archiveArtifacts artifacts: 'performance/results.jtl',
                             allowEmptyArchive: true

            archiveArtifacts artifacts: 'petclinic.log',
                             allowEmptyArchive: true
        }
    }
}
```

---

# 🧠 39. Question 1 — Pourquoi JUnit ?

JUnit permet de vérifier :

```text
Le code fonctionne-t-il correctement ?
```

Exemple :

```text
Test
 ↓
PASS
```

ou :

```text
Test
 ↓
FAIL
```

---

# 🧠 40. Question 2 — Pourquoi JMeter ?

JMeter permet de vérifier :

```text
Comment l'application se comporte-t-elle
lorsqu'elle reçoit beaucoup de requêtes ?
```

Par exemple :

```text
10 utilisateurs
20 utilisateurs
50 utilisateurs
100 utilisateurs
```

---

# 🧠 41. Question 3 — Pourquoi Jenkins ?

Sans Jenkins :

```text
Développeur
   ↓
Lance les tests manuellement
   ↓
Regarde les résultats
```

Avec Jenkins :

```text
Git
 ↓
Jenkins
 ↓
Tests
 ↓
Rapports
 ↓
Validation
```

---

# 💥 42. Challenge — Faire échouer JUnit

Modifier volontairement un test existant afin qu'il échoue.

Committer :

```bash
git add .
git commit -m "test jenkins failure"
git push
```

Lancer Jenkins.

Observer :

```text
Tests JUnit
     ↓
    FAIL
```

Question :

> Que fait Jenkins lorsqu'un test JUnit échoue ?

---

# 🔧 43. Challenge — Corriger le test

Corriger le test.

Puis :

```bash
git add .
git commit -m "fix test"
git push
```

Relancer Jenkins.

Résultat attendu :

```text
Tests JUnit
     ↓
     ✅

Start Application
     ↓
     ✅

Performance Test
     ↓
     ✅
```

---

# ⭐ 44. Challenge final

Modifier JMeter pour utiliser :

```text
20 utilisateurs
```

et :

```text
10 secondes
```

Puis comparer les résultats avec :

```text
10 utilisateurs
```

Questions :

```text
1. Combien de requêtes ont été exécutées ?

2. Quel est le temps de réponse moyen ?

3. Y a-t-il eu des erreurs ?

4. Que se passe-t-il lorsque le nombre d'utilisateurs augmente ?
```

---

# 📦 45. Livrable

Le repository doit finalement contenir :

```text
spring-petclinic/
│
├── src/
│
├── performance/
│   ├── petclinic-test.jmx
│   └── results.jtl
│
├── pom.xml
│
└── Jenkinsfile
```

---

# 🎯 46. Ce que vous avez appris

À la fin du TP, vous savez :

```text
✅ Utiliser un projet Java existant

✅ Vérifier Java 21

✅ Utiliser Maven

✅ Exécuter des tests JUnit

✅ Publier les résultats JUnit dans Jenkins

✅ Démarrer une application Spring Boot

✅ Créer un test JMeter

✅ Exécuter JMeter en mode non graphique

✅ Automatiser JMeter avec Jenkins

✅ Archiver les résultats
```

---

# 🏁 Pipeline final

```text
                  Git
                   │
                   ▼
              Jenkins
                   │
                   ▼
             Maven / Java 21
                   │
                   ▼
               JUnit
                   │
                   ▼
            Rapport de tests
                   │
                   ▼
            Démarrage App
                   │
                   ▼
               JMeter
                   │
                   ▼
          Test de performance
                   │
                   ▼
              Résultats
```

# 💡 Message clé

```text
JUnit  → Est-ce que mon code fonctionne ?

JMeter → Est-ce que mon application tient la charge ?

Jenkins → Est-ce que je peux automatiser tout cela ?
```
````
