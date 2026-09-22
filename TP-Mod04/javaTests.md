````md
# 🧪 TP — Automatisation des tests avec Jenkins

## Java + JUnit + JMeter

---

# 🎯 Objectif du TP

Dans ce TP, nous allons découvrir comment Jenkins peut automatiser les tests d'une application Java.

Nous allons utiliser :

- **Java** → pour développer une petite application
- **Maven** → pour compiler le projet
- **JUnit** → pour réaliser les tests unitaires
- **JMeter** → pour réaliser un test de performance
- **Jenkins** → pour automatiser tout cela

À la fin, notre pipeline Jenkins fera :

```text
Récupération du code
        ↓
Compilation
        ↓
Tests JUnit
        ↓
Rapport de tests
        ↓
Test de performance JMeter
```

---

# 🏗️ 1. Préparer le projet

Nous allons créer une petite application Java très simple.

Elle permettra de faire des calculs.

## Structure finale

Notre projet aura cette structure :

```text
gfptech-tests/
│
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── gfptech/
│   │               └── Calculator.java
│   │
│   └── test/
│       └── java/
│           └── com/
│               └── gfptech/
│                   └── CalculatorTest.java
│
├── performance/
│   └── test-plan.jmx
│
├── pom.xml
│
└── Jenkinsfile
```

---

# ☕ 2. Vérifier Java et Maven

Avant de commencer, vérifier Java :

```bash
java -version
```

Puis Maven :

```bash
mvn -version
```

Vous devez obtenir quelque chose ressemblant à :

```text
Java version: 17
```

et :

```text
Apache Maven 3.x
```

> Si ces commandes fonctionnent, vous pouvez continuer.

---

# 📁 3. Créer le projet

Créer le répertoire :

```bash
mkdir gfptech-tests
cd gfptech-tests
```

Créer les répertoires :

```bash
mkdir -p src/main/java/com/gfptech
mkdir -p src/test/java/com/gfptech
mkdir -p performance
```

Créer les fichiers :

```bash
touch src/main/java/com/gfptech/Calculator.java
touch src/test/java/com/gfptech/CalculatorTest.java
touch performance/test-plan.jmx
touch pom.xml
touch Jenkinsfile
```

---

# ☕ 4. Créer l'application Java

Ouvrir :

```text
src/main/java/com/gfptech/Calculator.java
```

Ajouter :

```java
package com.gfptech;

public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public int divide(int a, int b) {

        if (b == 0) {
            throw new IllegalArgumentException("Division by zero");
        }

        return a / b;
    }
}
```

Notre application sait maintenant faire :

```text
2 + 3
10 - 4
3 × 4
10 / 2
```

---

# 📦 5. Créer le fichier Maven

Ouvrir :

```text
pom.xml
```

Ajouter :

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.gfptech</groupId>

    <artifactId>gfptech-tests</artifactId>

    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>

        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>

        </plugins>

    </build>

</project>
```

---

# 🧪 6. Créer le premier test JUnit

Ouvrir :

```text
src/test/java/com/gfptech/CalculatorTest.java
```

Ajouter :

```java
package com.gfptech;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

public class CalculatorTest {

    @Test
    void testAdd() {

        Calculator calculator = new Calculator();

        assertEquals(5, calculator.add(2, 3));
    }

    @Test
    void testSubtract() {

        Calculator calculator = new Calculator();

        assertEquals(6, calculator.subtract(10, 4));
    }

    @Test
    void testMultiply() {

        Calculator calculator = new Calculator();

        assertEquals(12, calculator.multiply(3, 4));
    }

    @Test
    void testDivide() {

        Calculator calculator = new Calculator();

        assertEquals(5, calculator.divide(10, 2));
    }

    @Test
    void testDivideByZero() {

        Calculator calculator = new Calculator();

        assertThrows(
            IllegalArgumentException.class,
            () -> calculator.divide(10, 0)
        );
    }
}
```

---

# ▶️ 7. Lancer les tests JUnit

Nous allons maintenant vérifier que les tests fonctionnent **avant même d'utiliser Jenkins**.

À la racine du projet :

```bash
mvn test
```

Maven va :

```text
1. Compiler le code
        ↓
2. Compiler les tests
        ↓
3. Exécuter JUnit
        ↓
4. Générer le rapport
```

Vous devriez obtenir :

```text
Tests run: 5
Failures: 0
Errors: 0
Skipped: 0
```

Et à la fin :

```text
BUILD SUCCESS
```

🎯 **Important :**

Avant de mettre quelque chose dans Jenkins, on vérifie toujours que cela fonctionne localement.

---

# 📊 8. Où sont les résultats JUnit ?

Maven génère automatiquement les résultats dans :

```text
target/surefire-reports/
```

Afficher le contenu :

```bash
ls target/surefire-reports
```

Vous devriez voir des fichiers comme :

```text
TEST-com.gfptech.CalculatorTest.xml
com.gfptech.CalculatorTest.txt
```

Le fichier XML sera utilisé par Jenkins pour afficher les résultats.

---

# 💥 9. Tester volontairement une erreur

Nous allons maintenant simuler un problème.

Dans :

```text
Calculator.java
```

modifier temporairement :

```java
return a + b;
```

en :

```java
return a - b;
```

Lancer :

```bash
mvn test
```

Le test doit maintenant échouer.

Vous devriez obtenir quelque chose ressemblant à :

```text
Tests run: 5
Failures: 1
Errors: 0
```

et :

```text
BUILD FAILURE
```

🎯 Jenkins pourra donc détecter automatiquement ce problème.

---

# 🔄 10. Corriger le problème

Remettre :

```java
return a + b;
```

Puis :

```bash
mvn test
```

Le résultat doit redevenir :

```text
Tests run: 5
Failures: 0
Errors: 0
```

et :

```text
BUILD SUCCESS
```

---

# 🚀 11. Préparer Jenkins

Maintenant que les tests fonctionnent localement, nous allons les exécuter avec Jenkins.

Créer le fichier :

```text
Jenkinsfile
```

Ajouter une première version très simple :

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                sh 'mvn clean package'
            }
        }

        stage('Tests') {

            steps {

                sh 'mvn test'
            }
        }
    }
}
```

---

# 🔍 12. Comprendre le Jenkinsfile

Notre pipeline contient deux étapes.

## Étape 1 — Build

```groovy
stage('Build') {
    steps {
        sh 'mvn clean package'
    }
}
```

Jenkins exécute :

```bash
mvn clean package
```

Cela permet de compiler l'application.

---

## Étape 2 — Tests

```groovy
stage('Tests') {
    steps {
        sh 'mvn test'
    }
}
```

Jenkins exécute :

```bash
mvn test
```

Les tests JUnit sont donc exécutés automatiquement.

---

# 📊 13. Publier les résultats JUnit dans Jenkins

Nous allons améliorer le pipeline.

Modifier le `Jenkinsfile` :

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                sh 'mvn clean package'
            }
        }

        stage('Tests') {

            steps {

                sh 'mvn test'
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

La partie :

```groovy
junit 'target/surefire-reports/*.xml'
```

indique à Jenkins :

> "Va chercher les résultats JUnit générés par Maven et affiche-les dans Jenkins."

---

# 🔗 14. Mettre le projet dans Git

Initialiser Git :

```bash
git init
```

Ajouter les fichiers :

```bash
git add .
```

Créer le premier commit :

```bash
git commit -m "initial java testing project"
```

Ajouter le repository distant :

```bash
git remote add origin <URL_DU_REPOSITORY>
```

Envoyer :

```bash
git push -u origin main
```

---

# ⚙️ 15. Créer le Pipeline Jenkins

Dans Jenkins :

```text
New Item
```

Créer :

```text
gfptech-java-tests
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

Mettre l'URL du repository.

Puis :

```text
Script Path

Jenkinsfile
```

Sauvegarder.

---

# ▶️ 16. Premier lancement Jenkins

Cliquer :

```text
Build Now
```

Jenkins doit exécuter :

```text
Checkout
    ↓
Build
    ↓
Tests
```

Résultat attendu :

```text
Build       ✅
Tests       ✅
```

---

# 📊 17. Voir les résultats JUnit

Ouvrir le build Jenkins.

Vous devriez avoir un lien :

```text
Test Result
```

Cliquer dessus.

Vous devez retrouver :

```text
Tests run: 5
Failures: 0
Errors: 0
```

🎯 Jenkins est maintenant capable de :

```text
Compiler
   +
Tester
   +
Afficher les résultats
```

---

# ⚡ 18. Installer JMeter

Nous allons maintenant ajouter un test de performance.

Vérifier si JMeter est disponible :

```bash
jmeter --version
```

Si JMeter est installé, vous devez obtenir sa version.

Pour Jenkins, il faut également que JMeter soit disponible sur l'agent Jenkins qui exécutera le pipeline.

---

# 📈 19. Comprendre JMeter

JUnit répond à la question :

```text
Est-ce que mon code fonctionne ?
```

JMeter répond plutôt à la question :

```text
Comment mon application se comporte-t-elle sous charge ?
```

Exemple :

```text
10 utilisateurs
     ↓
Application
     ↓
Mesure des performances
```

---

# 🧪 20. Créer un test JMeter très simple

Nous allons créer :

```text
performance/test-plan.jmx
```

Le plus simple est de créer le test avec l'interface graphique JMeter.

Lancer :

```bash
jmeter
```

---

# 🖥️ 21. Créer le Thread Group

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
chacun exécute le test 5 fois
```

---

# 🧮 22. Ajouter un élément simple

Pour garder le TP simple, nous allons utiliser une requête HTTP.

Dans :

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

Configurer par exemple :

```text
Protocol : http
Server  : localhost
Port    : 8080
Path    : /
```

⚠️ Cette partie suppose qu'une application HTTP est disponible sur `localhost:8080`.

Si votre application n'est pas une application HTTP, vous pouvez utiliser une application de démonstration ou adapter le serveur cible.

---

# 📊 23. Ajouter un rapport JMeter

Dans :

```text
Thread Group
```

ajouter :

```text
Add
    ↓
Listener
    ↓
View Results Tree
```

Ajouter également :

```text
Add
    ↓
Listener
    ↓
Summary Report
```

---

# 💾 24. Sauvegarder le test JMeter

Sauvegarder le fichier :

```text
performance/test-plan.jmx
```

Vérifier :

```bash
ls performance
```

Vous devez avoir :

```text
test-plan.jmx
```

---

# ▶️ 25. Exécuter JMeter en ligne de commande

Jenkins ne doit pas utiliser l'interface graphique de JMeter.

Nous allons utiliser le mode non graphique :

```bash
jmeter -n
```

Exemple :

```bash
jmeter \
    -n \
    -t performance/test-plan.jmx \
    -l performance/results.jtl
```

Signification :

```text
-n
mode non graphique

-t
fichier JMeter

-l
fichier contenant les résultats
```

---

# 📄 26. Vérifier le résultat

Après l'exécution :

```bash
ls performance
```

Vous devriez avoir :

```text
test-plan.jmx
results.jtl
```

Le fichier :

```text
results.jtl
```

contient les résultats du test de performance.

---

# 🤖 27. Ajouter JMeter dans Jenkins

Modifier le `Jenkinsfile` :

```groovy
pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                sh 'mvn clean package'
            }
        }

        stage('JUnit Tests') {

            steps {

                sh 'mvn test'
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
                    -t performance/test-plan.jmx \
                    -l performance/results.jtl
                '''
            }
        }
    }
}
```

---

# 🚀 28. Pipeline final

Notre pipeline Jenkins est maintenant :

```text
                    Jenkins
                       │
                       ▼
                 Checkout Git
                       │
                       ▼
                    Build
                       │
                       ▼
                  Tests JUnit
                       │
                       ▼
                Rapport JUnit
                       │
                       ▼
                Test JMeter
                       │
                       ▼
             Résultats performance
```

---

# 🧪 29. Tester le pipeline

Faire un commit :

```bash
git add .
git commit -m "add jmeter performance test"
git push
```

Puis dans Jenkins :

```text
Build Now
```

Observer les étapes :

```text
Build
  ✅

JUnit Tests
  ✅

Performance Test
  ✅
```

---

# 💥 30. Challenge — Faire échouer JUnit

Modifier volontairement le code :

```java
public int add(int a, int b) {
    return a - b;
}
```

Faire :

```bash
git add .
git commit -m "test pipeline failure"
git push
```

Lancer Jenkins.

Que doit-il se passer ?

```text
Build
  ✅

JUnit Tests
  ❌

Performance Test
  ⛔
```

Le pipeline s'arrête car les tests unitaires ont échoué.

---

# 🔧 31. Challenge — Corriger le problème

Remettre :

```java
public int add(int a, int b) {
    return a + b;
}
```

Puis :

```bash
git add .
git commit -m "fix calculator"
git push
```

Relancer Jenkins.

Résultat attendu :

```text
Build
  ✅

JUnit Tests
  ✅

Performance Test
  ✅
```

---

# 🎯 32. Challenge final

Modifier le pipeline afin que le test JMeter soit exécuté uniquement sur la branche :

```text
main
```

Utiliser :

```groovy
when {
    branch 'main'
}
```

Exemple :

```groovy
stage('Performance Test') {

    when {
        branch 'main'
    }

    steps {

        sh '''
            jmeter \
            -n \
            -t performance/test-plan.jmx \
            -l performance/results.jtl
        '''
    }
}
```

---

# 📦 33. Livrable final

Le repository doit contenir :

```text
gfptech-tests/
│
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── gfptech/
│   │               └── Calculator.java
│   │
│   └── test/
│       └── java/
│           └── com/
│               └── gfptech/
│                   └── CalculatorTest.java
│
├── performance/
│   └── test-plan.jmx
│
├── pom.xml
│
└── Jenkinsfile
```

---

# 💡 34. À retenir

## JUnit

JUnit permet de tester le code Java :

```text
Calculator
     ↓
JUnit
     ↓
OK / KO
```

---

## Maven

Maven permet notamment de :

```text
Compiler
   ↓
Tester
   ↓
Générer les résultats
```

Commande principale :

```bash
mvn test
```

---

## JMeter

JMeter permet de réaliser des tests de performance :

```text
Utilisateurs
      ↓
   JMeter
      ↓
Application
      ↓
Résultats
```

---

## Jenkins

Jenkins automatise tout :

```text
Git
 ↓
Build
 ↓
JUnit
 ↓
Rapport
 ↓
JMeter
 ↓
Validation
```

---

# 🏁 Conclusion

Nous avons construit un pipeline Jenkins simple permettant de :

```text
✅ Compiler une application Java

✅ Exécuter des tests JUnit

✅ Publier les résultats JUnit

✅ Exécuter un test de performance JMeter

✅ Automatiser les tests avec Jenkins

✅ Faire échouer le pipeline lorsqu'un test échoue
```

Le principe important à retenir est :

```text
                 CI
                  │
                  ▼
              Jenkins
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      JUnit               JMeter
        │                   │
        ▼                   ▼
  Qualité du code      Performance
```
````
