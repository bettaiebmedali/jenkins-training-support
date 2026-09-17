# 🧩 TP2 — Tous les Types de Projets Jenkins (Items)

> Compatible avec Jenkins v2.528.1+  
> Objectif : maîtriser **tous les types d’items Jenkins** et comprendre quand les utiliser.

---

## 🧱 1. Freestyle Project

### 🎯 Objectif
Créer un **projet simple** sans script, basé sur des actions configurées dans l’interface.

### ⚙️ Étapes

1. Tableau de bord → **Nouveau Item**  
2. Nom : `TP4-Freestyle`  
3. Type : **Projet freestyle**  
4. Configurer :  
   - **Description :** « Projet Freestyle de test »  
   - **Gestion du code source :** Git → `https://github.com/monrepo/demo.git`  
   - **Build Triggers :** “Build périodique” → `H/5 * * * *` (toutes les 5 min)  
   - **Build Steps :**
     - “Exécuter une commande shell” :  
       ```bash
       echo "Bonjour Jenkins !"
       echo "Date du build : $(date)"
       ```

5. Sauvegarder → **Build Now**  

### 🔍 Résultat
Ce job s’exécute périodiquement, sans pipeline scripté.  
> Idéal pour les tests simples, les jobs d’intégration, ou les scripts de maintenance.

---

## 🔁 2. Pipeline Project

### 🎯 Objectif
Créer un pipeline scripté via un **Jenkinsfile**.

### ⚙️ Étapes

1. Nouveau Item → `TP4-Pipeline`  
2. Type : **Pipeline**  
3. Section *Pipeline script* :

```groovy
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo "Hello depuis un pipeline !"
            }
        }
    }
}
```

4. Sauvegarde → **Build Now**

### 🔍 Résultat
Un pipeline codé et versionné.  
> Idéal pour l’automatisation CI/CD, avec gestion des branches et artefacts.

---

## 🧩 3. Multibranch Pipeline

### 🎯 Objectif
Créer un pipeline dynamique qui détecte **chaque branche Git** et exécute le Jenkinsfile correspondant.

### ⚙️ Étapes

1. Nouveau Item → `TP4-Multibranch`  
2. Type : **Multibranch Pipeline**  
3. Dans **Sources** :
   - Type : Git  
   - URL : `https://github.com/monrepo/monapp.git`
4. Sauvegarde. Jenkins scanne le repo.  
5. Chaque branche contenant un `Jenkinsfile` devient un job.

### Jenkinsfile exemple (branche `main`)
```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { echo "Build de la branche ${env.BRANCH_NAME}" } }
    }
}
```

### 🔍 Résultat
Chaque branche est indépendante et détectée automatiquement.  
> Idéal pour les projets Git avec développement parallèle.

---

## 🗂️ 4. Folder

### 🎯 Objectif
Organiser les jobs en **dossiers hiérarchiques**.

### ⚙️ Étapes

1. Nouveau Item → Nom : `TP4-Folder`  
2. Type : **Folder**  
3. Ajouter un sous-élément : “Nouveau Item” → “TP4-Freestyle-Child”  
   - Type : Freestyle  
   - Commande shell : `echo "Job dans un dossier"`

### 🔍 Résultat
Les dossiers permettent de gérer des environnements ou équipes séparées.  
> Idéal pour les gros Jenkins avec plusieurs pipelines.

---

## 🧮 5. Maven Project

### 🎯 Objectif
Utiliser un projet **basé sur Maven**.

### ⚙️ Étapes

1. Installe le plugin “Maven Integration”.  
2. Nouveau Item → Nom : `TP4-Maven-App`  
3. Type : **Projet Maven**  
4. Configurer :  
   - Code source : Git → `https://github.com/monrepo/maven-demo.git`  
   - Objectif Maven : `clean package`
   - JDK : `OpenJDK 17` (ou ton JDK installé)
   - Maven : `Maven 3.x`
5. Sauvegarde → Build

### 🔍 Résultat
Jenkins exécute Maven automatiquement, détecte les tests et archive les artefacts `.jar`.  
> Idéal pour les projets Java standards.


---

## 🧰 6. Multi-configuration (Matrix Project)

### 🎯 Objectif
Tester un projet sur **plusieurs environnements ou versions**.

### ⚙️ Étapes

1. Installe le plugin **Matrix Project** (souvent inclus).  
2. Nouveau Item → Nom : `TP4-Matrix`  
3. Type : **Multi-configuration project**
4. Configurer les **Axes de build** :
   - `OS` → `linux`  `windows`
   - `JDK` → `11`  `17`
5. Étapes de build :  
   ```bash
   echo "Build sur OS=$OS avec JDK=$JDK"
   ```

### 🔍 Résultat
Jenkins exécute **4 builds en parallèle** (Linux+JDK11, Linux+JDK17, Windows+JDK11, Windows+JDK17).  
> Idéal pour les tests de compatibilité et CI multi-plateformes.

---

## 🧾 Résumé général

| Type d’Item | Description | Cas d’usage principal |
|--------------|--------------|------------------------|
| **Freestyle** | Job simple sans script | Tâches manuelles ou tests |
| **Pipeline** | Script codé (Jenkinsfile) | CI/CD complet |
| **Multibranch Pipeline** | Pipeline par branche Git | Dev Git multi-branches |
| **Folder** | Conteneur de jobs | Organisation & permissions |
| **Maven Project** | Intégré à Maven | Build Java classique |
| **External Job** | Suivi externe | Scripts legacy |
| **Matrix (Multi-configuration)** | Exécutions multiples | Tests multi-envs |

---

## 🏁 Conclusion
Tu sais désormais créer et configurer **tous les types d’Items Jenkins**.  
Chacun a un usage précis selon ton projet, ton langage et ta complexité.

**Prochaine étape suggérée :** combiner ces items dans une architecture CI/CD complète (TP5).  
