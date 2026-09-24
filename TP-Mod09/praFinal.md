# TP — Sauvegarde et PRA de Jenkins avec Pipeline

## 🎯 Objectifs
À la fin de ce TP, vous serez capables de :
* Identifier le répertoire `JENKINS_HOME` de Jenkins ;
* Comprendre pourquoi Jenkins doit être sauvegardé ;
* Créer une sauvegarde automatisée avec un Pipeline ;
* Compresser `JENKINS_HOME` avec `tar` et `gzip` ;
* Vérifier l'intégrité d'une sauvegarde ;
* Mettre en place une politique simple de rétention ;
* Simuler un scénario de PRA (Plan de Reprise d'Activité) ;
* Comprendre la différence entre sauvegarde et restauration.

---

## Sommaire
- [1. Contexte](#1-contexte)
- [2. Architecture du TP](#2-architecture-du-tp)
- [3. Préparation de l'environnement](#3-préparation-de-lenvironnement)
- [4. Créer le Pipeline](#4-créer-le-pipeline)
- [5. Pipeline de sauvegarde](#5-pipeline-de-sauvegarde)
- [6. Comprendre le Pipeline](#6-comprendre-le-pipeline)
- [7. Les variables d'environnement](#7-les-variables-denvironnement)
- [8. Générer la date](#8-générer-la-date)
- [9. Générer le nom de l'archive](#9-générer-le-nom-de-larchive)
- [10. Comprendre `tar`](#10-comprendre-tar)
- [11. Pourquoi utiliser `-C` ?](#11-pourquoi-utiliser--c-)
- [12. Exclusions](#12-exclusions)
- [13. Vérifier l'intégrité](#13-vérifier-lintégrité)
- [14. Rétention](#14-rétention)
- [15. Simulation PRA](#15-simulation-pra)
- [16. Exécuter le Pipeline](#16-exécuter-le-pipeline)
- [17. Vérification côté serveur](#17-vérification-côté-serveur)
- [18. Explorer l'archive](#18-explorer-larchive)
- [19. Exercice — Ajouter une taille minimale](#19-exercice--ajouter-une-taille-minimale)
- [20. Exercice — Conserver seulement 5 sauvegardes](#20-exercice--conserver-seulement-5-sauvegardes)
- [21. Exercice — Automatiser le backup](#21-exercice--automatiser-le-backup)
- [22. Exercice final — Vraie restauration PRA](#22-exercice-final--vraie-restauration-pra)
- [23. ⚠️ Restauration complète de Jenkins](#23-️-restauration-complète-de-jenkins)
- [24. Questions de synthèse](#24-questions-de-synthèse)
- [🎯 Résultat attendu](#-résultat-attendu)
- [🏁 Défi final](#-défi-final)

---

## 1. Contexte
Jenkins contient de nombreuses informations importantes :
* Les jobs ;
* Les pipelines ;
* Les configurations ;
* Les utilisateurs ;
* Les credentials ;
* Les plugins ;
* Les paramètres Jenkins ;
* Les historiques de builds ;
* Les configurations globales.

Ces données sont stockées principalement dans :
```text
/var/lib/jenkins
```
Ce répertoire correspond au **`JENKINS_HOME`**.

Si ce répertoire est perdu, Jenkins peut être réinstallé, mais les données et configurations peuvent disparaître.  
L'objectif de ce TP est donc de mettre en place une sauvegarde automatisée.

---

## 2. Architecture du TP
```text
                    Jenkins
                       │
                       │
                       ▼
              /var/lib/jenkins
                  JENKINS_HOME
                       │
                       │ tar + gzip
                       ▼
              /var/backups/jenkins
                       │
                       ▼
       jenkins-YYYY-MM-DD_HH-MM.tar.gz
```

Le Pipeline va réaliser les étapes suivantes :
1. **Initialisation**
2. **Création de la sauvegarde**
3. **Vérification de l'intégrité**
4. **Nettoyage des anciennes sauvegardes**
5. **Simulation PRA**

---

## 3. Préparation de l'environnement

### 3.1 Vérifier JENKINS_HOME
Sur le serveur Jenkins :
```bash
ls -ld /var/lib/jenkins
```

Vérifier également le processus Jenkins :
```bash
ps aux | grep jenkins
```
Vous devez constater que Jenkins utilise `/var/lib/jenkins`.

### 3.2 Créer le répertoire de sauvegarde
Créer le répertoire :
```bash
sudo mkdir -p /var/backups/jenkins
```

Donner les droits à l'utilisateur Jenkins :
```bash
sudo chown jenkins:jenkins /var/backups/jenkins
```

Vérifier :
```bash
ls -ld /var/backups/jenkins
```
On doit obtenir un propriétaire similaire à : `jenkins jenkins`.

### 3.3 Tester les permissions
Tester que l'utilisateur Jenkins peut écrire dans le répertoire :
```bash
sudo -u jenkins touch /var/backups/jenkins/test.txt
```

Puis supprimer le fichier :
```bash
sudo -u jenkins rm /var/backups/jenkins/test.txt
```

> **❓ Question**  
> Pourquoi Jenkins doit-il avoir les droits d'écriture sur `/var/backups/jenkins` ?

---

## 4. Créer le Pipeline
1. Créer un nouveau job Jenkins nommé : `pra`
2. Choisir : **Pipeline**
3. Dans **Pipeline** → **Definition**, sélectionner : **Pipeline script**

---

## 5. Pipeline de sauvegarde
Copier le Jenkinsfile suivant :

```groovy
pipeline {
    agent any

    environment {
        BACKUP_DIR = '/var/backups/jenkins'
    }

    stages {

        stage('Initialisation') {
            steps {
                script {
                    env.BACKUP_DATE = sh(
                        script: 'date +%Y-%m-%d_%H-%M',
                        returnStdout: true
                    ).trim()

                    env.ARCHIVE_NAME = "jenkins-${env.BACKUP_DATE}.tar.gz"

                    echo "========================================"
                    echo "Début du processus de sauvegarde"
                    echo "========================================"
                    echo "Date de backup   : ${env.BACKUP_DATE}"
                    echo "Nom de l'archive : ${env.ARCHIVE_NAME}"
                    echo "Dossier cible    : ${env.BACKUP_DIR}"
                    echo "========================================"

                    sh "mkdir -p ${env.BACKUP_DIR}"
                }
            }
        }

        stage('Créer Sauvegarde') {
            steps {
                echo "📦 Archivage et compression de JENKINS_HOME..."

                sh """
                    tar \
                      --exclude=/var/lib/jenkins/workspace/* \
                      --exclude=/var/lib/jenkins/caches \
                      --exclude=/var/lib/jenkins/plugins/*.bak \
                      -czf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} \
                      -C /var/lib/jenkins .
                """
            }
        }

        stage('Vérifier Intégrité') {
            steps {
                echo "🔍 Vérification de l'archive..."

                sh """
                    test -f ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}

                    tar -tzf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} > /dev/null
                """

                echo "✅ Archive valide"
            }
        }

        stage('Nettoyage Rétention') {
            steps {
                echo "🧹 Nettoyage des anciennes sauvegardes..."

                sh """
                    find ${env.BACKUP_DIR} \
                      -name 'jenkins-*.tar.gz' \
                      -type f \
                      -mtime +7 \
                      -delete
                """
            }
        }

        stage('Simulation PRA') {
            steps {
                echo "🚨 Simulation PRA"
                echo "Archive disponible : ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}"

                sh "ls -lh ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}"
            }
        }
    }

    post {
        success {
            echo """
            ========================================
            ✅ SAUVEGARDE JENKINS RÉUSSIE
            ========================================

            Archive : ${env.ARCHIVE_NAME}
            Dossier : ${env.BACKUP_DIR}

            ========================================
            """
        }

        failure {
            echo """
            ========================================
            ❌ ÉCHEC DE LA SAUVEGARDE JENKINS
            ========================================

            Vérifiez les logs du pipeline.

            ========================================
            """
        }

        always {
            echo "🏁 Fin du traitement d'administration"
        }
    }
}
```

---

## 6. Comprendre le Pipeline

### 6.1 `agent any`
```groovy
agent any
```
Cela signifie que Jenkins peut exécuter le Pipeline sur n'importe quel agent disponible correspondant aux contraintes.  
Dans notre cas, les commandes Linux doivent être exécutées sur un nœud disposant de : `sh`, `tar`, `gzip`, `find`.

---

## 7. Les variables d'environnement
Nous définissons :
```groovy
environment {
    BACKUP_DIR = '/var/backups/jenkins'
}
```
Cette variable contient le répertoire où seront stockées les sauvegardes.

---

## 8. Générer la date
Le Pipeline utilise :
```groovy
env.BACKUP_DATE = sh(
    script: 'date +%Y-%m-%d_%H-%M',
    returnStdout: true
).trim()
```
La commande Linux `date +%Y-%m-%d_%H-%M` peut retourner par exemple : `2026-09-24_19-27`.  
Le `.trim()` permet de supprimer le retour à la ligne produit par la commande shell.

---

## 9. Générer le nom de l'archive
Nous construisons ensuite :
```groovy
env.ARCHIVE_NAME = "jenkins-${env.BACKUP_DATE}.tar.gz"
```
Exemple : `jenkins-2026-09-24_19-27.tar.gz`.

**Pourquoi ajouter la date ?**  
Parce que cela permet d'avoir plusieurs sauvegardes horodatées :
* `jenkins-2026-09-22_02-00.tar.gz`
* `jenkins-2026-09-23_02-00.tar.gz`
* `jenkins-2026-09-24_02-00.tar.gz`

---

## 10. Comprendre `tar`
La commande principale est :
```bash
tar -czf archive.tar.gz ...
```

| Option | Signification |
| :--- | :--- |
| **`-c`** | **Create** : créer une archive |
| **`-z`** | Utiliser **gzip** pour la compression |
| **`-f`** | Spécifier le **nom du fichier** archive |

Donc `tar -czf backup.tar.gz` signifie : *Créer une archive compressée gzip appelée `backup.tar.gz`.*

---

## 11. Pourquoi utiliser `-C` ?
Dans le Pipeline :
```bash
-C /var/lib/jenkins .
```
Cela signifie : **Se placer dans `/var/lib/jenkins` puis archiver le contenu courant `.`**.  
Cela évite notamment d'avoir des chemins absolus superflus dans l'arborescence de l'archive.

---

## 12. Exclusions
Nous excluons :
* `--exclude=/var/lib/jenkins/workspace/*` : Les workspaces peuvent être très volumineux et sont régénérables via le SCM (Git, etc.).
* `--exclude=/var/lib/jenkins/caches`
* `--exclude=/var/lib/jenkins/plugins/*.bak`

---

## 13. Vérifier l'intégrité
Le Pipeline utilise :
```bash
test -f archive.tar.gz
```
Cette commande vérifie que le fichier existe.

Ensuite :
```bash
tar -tzf archive.tar.gz > /dev/null
```
Permet de lire la table des matières de l'archive sans la désarchiver. Si l'archive est corrompue, la commande renvoie un code d'erreur et Jenkins stoppe le Pipeline en **FAILURE**.

---

## 14. Rétention
Le Pipeline utilise :
```bash
find /var/backups/jenkins \
  -name 'jenkins-*.tar.gz' \
  -type f \
  -mtime +7 \
  -delete
```
Cette commande supprime les sauvegardes modifiées il y a plus de 7 jours.

> **❓ Question**  
> Que signifie `-mtime +7` ?

---

## 15. Simulation PRA
Le Pipeline affiche : `🚨 Simulation PRA`, puis :
```bash
ls -lh /var/backups/jenkins/jenkins-....tar.gz
```
Exemple de sortie :
```text
-rw-r--r-- 1 jenkins jenkins 547M Sep 24 19:27 /var/backups/jenkins/jenkins-2026-09-24_19-27.tar.gz
```
Cela permet de vérifier l'existence, la taille, le propriétaire et l'emplacement de l'archive.

---

## 16. Exécuter le Pipeline
1. Cliquer sur **Build Now**.
2. Observer la console.

Vous devez obtenir un journal similaire à :
```text
Date de backup   : 2026-09-24_19-27
Nom de l'archive : jenkins-2026-09-24_19-27.tar.gz
Dossier cible    : /var/backups/jenkins
...
📦 Archivage et compression de JENKINS_HOME...
🔍 Vérification de l'archive...
✅ Archive valide
🚨 Simulation PRA
Finished: SUCCESS
```

---

## 17. Vérification côté serveur
Sur le serveur Jenkins, vérifiez le fichier produit :
```bash
ls -lh /var/backups/jenkins/
```
Vous devez voir l'archive `jenkins-2026-09-24_19-27.tar.gz`.

---

## 18. Explorer l'archive
Afficher les 30 premiers fichiers contenus dans l'archive :
```bash
tar -tzf /var/backups/jenkins/jenkins-2026-09-24_19-27.tar.gz | head -30
```

> **❓ Questions**
> 1. Quels fichiers Jenkins retrouvez-vous ?
> 2. Retrouvez-vous les configurations des jobs ?
> 3. Retrouvez-vous les utilisateurs ?
> 4. Retrouvez-vous les plugins ?
> 5. Pourquoi les workspaces ne sont-ils pas sauvegardés ?

---

## 19. Exercice — Ajouter une taille minimale
Modifier le Pipeline pour vérifier que la sauvegarde fait **au moins 100 Mo** (104 857 600 octets).

* **Indice** : Utiliser `stat -c%s fichier` puis une condition Shell (`if ...`).
* **Objectif** :
  * Si `Archive < 100 Mo` ➔ ❌ **Pipeline FAILURE**
  * Si `Archive >= 100 Mo` ➔ ✅ **Pipeline SUCCESS**

---

## 20. Exercice — Conserver seulement 5 sauvegardes
Modifier la stratégie de rétention pour conserver uniquement les **5 fichiers de sauvegarde les plus récents**, indépendamment de leur date de création.

> **⚠️ Attention** : `-mtime +7` se base sur l'âge (7 jours), pas sur le nombre de fichiers.  
> *Indice* : Réfléchissez à une commande combinant `ls -t`, `tail -n +6` et `xargs rm -f`.

---

## 21. Exercice — Automatiser le backup
Configurer le job Jenkins pour qu'il s'exécute automatiquement chaque nuit.

1. Aller dans la configuration du job → **Triggers d'exécution (Build Triggers)**.
2. Cocher **Exécuter périodiquement (Build periodically)**.
3. Renseigner :
```text
H 2 * * *
```
*(Déclenche une exécution quotidienne aux alentours de 2h du matin avec un étalement de charge).*

---

## 22. Exercice final — Vraie restauration PRA
> **⚠️ ATTENTION** : Réaliser cet exercice **uniquement dans un environnement de laboratoire**.

1. **Créer un fichier de test** dans `JENKINS_HOME` :
   ```bash
   sudo -u jenkins touch /var/lib/jenkins/pra-test.txt
   ls -l /var/lib/jenkins/pra-test.txt
   ```
2. **Lancer la sauvegarde** via votre Pipeline.
3. **Simuler la perte du fichier** :
   ```bash
   sudo rm /var/lib/jenkins/pra-test.txt
   ls -l /var/lib/jenkins/pra-test.txt   # Doit retourner "No such file or directory"
   ```
4. **Restaurer uniquement le fichier** à partir de l'archive :
   ```bash
   sudo tar -xzf /var/backups/jenkins/jenkins-YYYY-MM-DD_HH-MM.tar.gz \
     -C /var/lib/jenkins ./pra-test.txt
   ```
5. **Vérifier** :
   ```bash
   ls -l /var/lib/jenkins/pra-test.txt
   ```

> **❓ Question**  
> Le fichier a-t-il été correctement restauré ?

---

## 23. ⚠️ Restauration complète de Jenkins
Une restauration complète de `JENKINS_HOME` requiert d'arrêter le service pour éviter toute corruption d'état.

Procédure globale :
```bash
# 1. Arrêter le service Jenkins
sudo systemctl stop jenkins

# 2. Sauvegarder l'état défectueux actuel par précaution
sudo mv /var/lib/jenkins /var/lib/jenkins.old
sudo mkdir -p /var/lib/jenkins

# 3. Restaurer l'archive validée
sudo tar -xzf /var/backups/jenkins/jenkins-2026-09-24_19-27.tar.gz -C /var/lib/jenkins

# 4. Corriger les permissions
sudo chown -R jenkins:jenkins /var/lib/jenkins

# 5. Redémarrer Jenkins
sudo systemctl start jenkins

# 6. Vérifier l'état du service
sudo systemctl status jenkins
```

---

## 24. Questions de synthèse

* **Question 1** : Quel est le rôle de `JENKINS_HOME` ?
* **Question 2** : Pourquoi sauvegarder Jenkins ?
* **Question 3** : Quelle est la différence entre un **Backup** (sauvegarde) et un **PRA** (Plan de Reprise d'Activité) ?
* **Question 4** : Pourquoi vérifier l'intégrité de l'archive (`tar -tzf`) ?
* **Question 5** : Pourquoi exclure les *workspaces* de la sauvegarde ?
* **Question 6** : Que se passe-t-il si `/var/backups/jenkins` n'est pas accessible en écriture par l'utilisateur `jenkins` ?
* **Question 7** : Quelle est la différence entre une étape `sh` et une étape `bat` dans un Pipeline ?
* **Question 8** : Pourquoi utiliser une date horodatée dans le nom de l'archive ?

---

## 🎯 Résultat attendu
À la fin du TP, votre infrastructure de sauvegarde doit fonctionner selon le schéma suivant :

```text
Jenkins
   │
   │ Pipeline ("pra")
   ▼
/var/lib/jenkins
   │
   │ tar + gzip
   ▼
/var/backups/jenkins
   │
   ├── jenkins-2026-09-22_02-00.tar.gz
   ├── jenkins-2026-09-23_02-00.tar.gz
   └── jenkins-2026-09-24_02-00.tar.gz
             │
             ▼
       Vérification (Intégrité & Taille >= 100 Mo)
             │
             ▼
       Rétention (Rétention des 5 derniers fichiers)
             │
             ▼
       Simulation PRA
```

---

## 🏁 Défi final

Voici une version complète et enrichie du Pipeline répondant à l'ensemble des exercices :

```groovy
pipeline {
    agent any

    environment {
        BACKUP_DIR = '/var/backups/jenkins'
        MIN_SIZE_BYTES = 104857600 // 100 Mo en octets
        NOTIFICATION_EMAIL = 'admin-devops@example.com'
    }

    triggers {
        cron('H 2 * * *')
    }

    stages {

        stage('Initialisation') {
            steps {
                script {
                    env.BACKUP_DATE = sh(
                        script: 'date +%Y-%m-%d_%H-%M',
                        returnStdout: true
                    ).trim()

                    env.ARCHIVE_NAME = "jenkins-${env.BACKUP_DATE}.tar.gz"

                    echo "========================================"
                    echo "Début du processus de sauvegarde"
                    echo "========================================"
                    echo "Date de backup   : ${env.BACKUP_DATE}"
                    echo "Nom de l'archive : ${env.ARCHIVE_NAME}"
                    echo "Dossier cible    : ${env.BACKUP_DIR}"
                    echo "========================================"

                    sh "mkdir -p ${env.BACKUP_DIR}"
                }
            }
        }

        stage('Créer Sauvegarde') {
            steps {
                echo "📦 Archivage et compression de JENKINS_HOME..."

                sh """
                    tar \
                      --exclude=/var/lib/jenkins/workspace/* \
                      --exclude=/var/lib/jenkins/caches \
                      --exclude=/var/lib/jenkins/plugins/*.bak \
                      -czf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} \
                      -C /var/lib/jenkins .
                """
            }
        }

        stage('Vérifier Intégrité et Taille') {
            steps {
                echo "🔍 Vérification de l'archive..."

                sh """
                    # 1. Vérification existence et intégrité tar
                    test -f ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}
                    tar -tzf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} > /dev/null

                    # 2. Vérification taille minimale (>= 100 Mo)
                    FILE_SIZE=\$(stat -c%s ${env.BACKUP_DIR}/${env.ARCHIVE_NAME})
                    echo "Taille de l'archive : \$FILE_SIZE octets"

                    if [ \$FILE_SIZE -lt ${env.MIN_SIZE_BYTES} ]; then
                        echo "❌ ERREUR: L'archive est trop petite (< 100 Mo)!"
                        exit 1
                    fi
                """

                echo "✅ Archive valide et conforme aux exigences de taille."
            }
        }

        stage('Nettoyage Rétention (Top 5)') {
            steps {
                echo "🧹 Conservation uniquement des 5 sauvegardes les plus récentes..."

                sh """
                    cd ${env.BACKUP_DIR}
                    ls -t jenkins-*.tar.gz | tail -n +6 | xargs -r rm -f
                """
            }
        }

        stage('Simulation PRA') {
            steps {
                echo "🚨 Simulation PRA"
                echo "Archive disponible : ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}"

                sh "ls -lh ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}"
            }
        }
    }

    post {
        success {
            echo """
            ========================================
            ✅ SAUVEGARDE JENKINS RÉUSSIE
            ========================================
            Archive : ${env.ARCHIVE_NAME}
            Dossier : ${env.BACKUP_DIR}
            ========================================
            """
            mail to: "${env.NOTIFICATION_EMAIL}",
                 subject: "✅ [SUCCESS] Sauvegarde Jenkins ${env.BACKUP_DATE}",
                 body: "La sauvegarde de Jenkins a réussi.\nArchive: ${env.ARCHIVE_NAME}\nEmplacement: ${env.BACKUP_DIR}"
        }

        failure {
            echo """
            ========================================
            ❌ ÉCHEC DE LA SAUVEGARDE JENKINS
            ========================================
            Vérifiez les logs du pipeline.
            ========================================
            """
            mail to: "${env.NOTIFICATION_EMAIL}",
                 subject: "❌ [FAILURE] Sauvegarde Jenkins ${env.BACKUP_DATE}",
                 body: "La sauvegarde de Jenkins a échoué. Veuillez vérifier les logs de la console Jenkins."
        }

        always {
            echo "🏁 Fin du traitement d'administration"
        }
    }
}
```