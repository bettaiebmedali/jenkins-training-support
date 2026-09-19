# TP Chapitre 3 – Mutualisation des pipelines 🔄

## Contexte / Storytelling
La startup GFP Tech grandit et de nombreux pipelines se répètent dans différents projets.  
Pour gagner du temps et assurer la cohérence, vous allez créer **des librairies de pipelines réutilisables** en Groovy.  
Cela permettra de partager des fonctions communes, comme l’envoi de notifications ou la compilation, entre plusieurs pipelines.

---

## Objectifs pédagogiques
- Créer une **librairie de pipeline partagée** (Shared Library) dans Jenkins.  
- Implémenter des **fonctions réutilisables** en Groovy.  
- Ajouter des **tests unitaires** pour la librairie.  
- Intégrer la librairie dans un pipeline existant.

---

## Prérequis techniques
- Jenkins avec Chapitre 1 et 2 configurés.  
- Projet Maven simple sur GitLab.  
- Accès administrateur Jenkins pour publier la librairie.  

---

## Étapes détaillées

### **1. Créer la structure de la librairie**
Dans un repo Git distinct ou dans le même projet, créer la structure :

```
(gfptech-pipeline-lib)
 ├── vars
 │   └── notifySlack.groovy
 └── src
     └── org/gfptech/utils/
         └── Helper.groovy
```

### **2. Ajouter une fonction réutilisable**
Exemple `vars/notifySlack.groovy` :

```groovy
def call(String channel, String message) {
    echo "📣 Notification Slack pour ${channel}: ${message}"
    // Ici, ajouter un appel réel à l’API Slack si besoin
}
```

Exemple `src/org/gfptech/utils/Helper.groovy` :

```groovy
package org.gfptech.utils

class Helper {
    static String formatMessage(String project, String status) {
        return "${project} - Build status: ${status}"
    }
}
```

### **3. Publier la librairie dans Jenkins**
1. Aller dans **Manage Jenkins → Configure System → Global Pipeline Libraries**  
2. Ajouter la librairie :  
   - Nom : `gfptech-lib`  
   - Default version : `main`  
   - Repository : URL Git de la librairie  
   - Load implicitly : optionnel  

### **4. Intégrer la librairie dans un pipeline**
Modifier un `Jenkinsfile` existant pour utiliser la librairie :

```groovy
@Library('gfptech-lib') _

pipeline {
    agent any
    stages {
        stage('Notify') {
            steps {
                notifySlack('dev-team', 'Pipeline démarré')
            }
        }
    }
}
```

### **5. Tester la librairie**
1. Déclencher le pipeline sur Jenkins.  
2. Vérifier que la fonction `notifySlack` fonctionne et affiche le message.  
3. Ajouter un autre pipeline qui utilise `Helper.formatMessage` pour démontrer la réutilisation.  

### **6. Ajouter des tests unitaires pour la librairie**
1. Créer des tests Groovy avec `JUnit` ou `Spock` dans `src/test/groovy`.  
2. Tester la fonction `Helper.formatMessage` pour différents inputs.  
3. Exécuter les tests via pipeline pour valider la librairie.

---

## Livrable final
- Une librairie Jenkins réutilisable et testée.  
- Pipeline existant utilisant la librairie pour envoyer des notifications et formatter des messages.  
- Tests unitaires validés dans Jenkins.

---

## Bonus (optionnel)
- Ajouter une fonction pour **build Maven** réutilisable.  
- Ajouter une fonction pour **déployer Docker** dans la librairie et la réutiliser dans plusieurs pipelines.
~~~

