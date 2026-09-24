# 🚀 Jenkins Administration — Sauvegarde, PRA & Bonnes Pratiques

## 🎯 Objectifs du TP

À la fin de ce TP, vous serez capable de :

- Administrer l'instance Jenkins via la gestion des répertoires clés (`JENKINS_HOME`).
- Mettre en place un système de **sauvegarde automatisé** de votre serveur Jenkins.
- Définir et tester une procédure de **Plan de Reprise d'Activité (PRA / Disaster Recovery)**.
- Gérer la sécurité, le nettoyage du workspace et la rétention de logs.
- Assurer la haute disponibilité et la scalabilité à l'aide d'agents dédiés.

---

## 🧩 Architecture & Périmètre du TP

```text
       ┌────────────────────────────────────────────────────────┐
       │                 Jenkins Primary Node                   │
       │                                                        │
       │   /var/jenkins_home                                    │
       │   ├── config.xml          (Config Globale)             │
       │   ├── plugins/            (Extensions)                 │
       │   ├── secrets/            (Clés de chiffrement)        │
       │   ├── users/              (Utilisateurs & Droits)      │
       │   └── jobs/               (Pipelines & Historique)     │
       └──────────────────────────┬─────────────────────────────┘
                                  │
                   Sauvegarde Automatique (Cron / Script)
                                  │
                                  ▼
       ┌────────────────────────────────────────────────────────┐
       │             Storage Distant / Backup Server            │
       │                                                        │
       │   /backups/jenkins/                                    │
       │   └── jenkins-backup-YYYY-MM-DD.tar.gz                 │
       └──────────────────────────┬─────────────────────────────┘
                                  │
                     Restauration (Test de PRA)
                                  │
                                  ▼
       ┌────────────────────────────────────────────────────────┐
       │                  Jenkins Recovery Node                 │
       │             (Restauration en cas de crash)             │
       └────────────────────────────────────────────────────────┘
```

---

## 📄 Jenkinsfile — Automated Backup Pipeline

Ce pipeline permet d'automatiser la sauvegarde hebdomadaire du `JENKINS_HOME`, d'exclure les éléments non essentiels (artifacts volumineux, builds temporaires) et d'envoyer l'archive vers un stockage sécurisé.

```groovy
pipeline {

    agent any

    environment {
        BACKUP_DIR     = '/var/backups/jenkins'
        JENKINS_HOME   = '/var/jenkins_home'
        BACKUP_DATE    = ''
        ARCHIVE_NAME   = ''
        RETENTION_DAYS = '7'
    }

    stages {

        // ============================================================
        // 1. Initialisation des variables de date et de répertoire
        // ============================================================

        stage('Initialisation') {

            steps {

                script {
                    env.BACKUP_DATE  = sh(script: "date +'%Y-%m-%d_%H-%M'", returnStdout: true).trim()
                    env.ARCHIVE_NAME = "jenkins-backup-${env.BACKUP_DATE}.tar.gz"

                    echo "========================================"
                    echo "Début du processus de sauvegarde"
                    echo "========================================"
                    echo "Date de backup   : ${env.BACKUP_DATE}"
                    echo "Nom de l'archive : ${env.ARCHIVE_NAME}"
                    echo "Dossier cible    : ${env.BACKUP_DIR}"
                    echo "========================================"

                    // Créer le dossier de destination si inexistant
                    sh "mkdir -p ${env.BACKUP_DIR}"
                }
            }
        }

        // ============================================================
        // 2. Création de l'archive compressée
        // ============================================================

        stage('Créer Sauvegarde') {

            steps {

                echo "📦 Archivage et compression de JENKINS_HOME..."

                /* 
                 * Remarque : On exclut les dossiers volumineux / temporaires :
                 * - workspace : Peut être reconstruit via SCM
                 * - caches    : Données temporaires
                 * - builds    : Optionnel (exclu ici pour réduire la taille)
                 */
                sh """
                    tar --exclude='${env.JENKINS_HOME}/workspace*' \
                        --exclude='${env.JENKINS_HOME}/caches' \
                        --exclude='${env.JENKINS_HOME}/plugins/*.bak' \
                        -czf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} \
                        -C ${env.JENKINS_HOME} .
                """

                echo "✅ Archive créée avec succès"
            }
        }

        // ============================================================
        // 3. Vérification de l'intégrité de la sauvegarde
        // ============================================================

        stage('Vérifier Intégrité') {

            steps {

                echo "🔍 Contrôle de l'intégrité du fichier .tar.gz..."

                sh """
                    tar -tzf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} > /dev/null
                    ls -lh ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}
                """

                echo "✅ Archive valide"
            }
        }

        // ============================================================
        // 4. Nettoyage des anciennes sauvegardes (Rétention)
        // ============================================================

        stage('Nettoyage Rétention') {

            steps {

                echo "🧹 Suppressions des sauvegardes de plus de ${env.RETENTION_DAYS} jours..."

                sh """
                    find ${env.BACKUP_DIR} -name "jenkins-backup-*.tar.gz" -mtime +${env.RETENTION_DAYS} -delete
                """

                echo "✅ Nettoyage terminé"
            }
        }

        // ============================================================
        // 5. Test de PRA simulé
        // ============================================================

        stage('Simulation PRA') {

            steps {

                echo "🔄 Simulation de restauration sur environnement de test..."

                sh """
                    TMP_RESTORE="/tmp/jenkins_test_restore_${env.BACKUP_DATE}"
                    mkdir -p \$TMP_RESTORE
                    tar -xzf ${env.BACKUP_DIR}/${env.ARCHIVE_NAME} -C \$TMP_RESTORE
                    
                    test -f \$TMP_RESTORE/config.xml
                    test -d \$TMP_RESTORE/secrets
                    
                    echo "Configuration et secrets vérifiés dans le dump !"
                    rm -rf \$TMP_RESTORE
                """

                echo "✅ Test de Restauration PRA Validé !"
            }
        }
    }

    // ================================================================
    // Actions finales & Notifications
    // ================================================================

    post {

        success {

            echo """
            ========================================
            ✅ SAUVEGARDE ET TEST PRA RÉUSSIS
            ========================================

            Archive    : ${env.BACKUP_DIR}/${env.ARCHIVE_NAME}
            Statut     : Valide & Restauration vérifiée

            ========================================
            """
        }

        failure {

            echo """
            ========================================
            ❌ ÉCHEC DE LA SAUVEGARDE JENKINS
            ========================================

            Une erreur est survenue lors du backup.
            Vérifiez les logs du nœud maître.

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

## 🔑 Points Importants sur l'Administration Jenkins

### 1. Structure critique du `JENKINS_HOME`
Pour réussir une sauvegarde et un PRA, il faut comprendre le rôle des dossiers clés dans `/var/jenkins_home` :

- `config.xml` : Contient la configuration globale du serveur (sécurité, nœuds, vue).
- `secrets/` : Contient les clés d'encodage (notamment `master.key` et `hudson.util.Secret`). **Sans ce dossier, vos identifiants enregistrés dans Jenkins seront illisibles !**
- `credentials.xml` : Les identifiants enregistrés (mots de passe, clés SSH, tokens), chiffrés par les clés dans `secrets/`.
- `plugins/` : La liste des extensions installées (`.jpi` / `.hpi`).
- `jobs/` : Contient la définition des projets et l'historique des exécutions.
- `users/` : Contient la base d'utilisateurs locaux et leurs configurations individuelles.

### 2. Procédure Pas-à-Pas de PRA (Plan de Reprise d'Activité)

En cas de crash total du serveur principal :

```text
Étape 1 : Provisionner une nouvelle instance Jenkins (Même version de préférence).
Étape 2 : Stopper le service Jenkins.
          systemctl stop jenkins  (ou docker stop container_jenkins)
Étape 3 : Vider ou écraser le contenu du nouveau JENKINS_HOME.
Étape 4 : Extraire la sauvegarde :
          tar -xzf jenkins-backup-YYYY-MM-DD.tar.gz -C /var/jenkins_home/
Étape 5 : Réappliquer les permissions système appropriées (ex: chown -R jenkins:jenkins /var/jenkins_home).
Étape 6 : Redémarrer le service Jenkins.
Étape 7 : Vérifier le bon chargement des credentials et des jobs.
```

### 3. Gestion de la mémoire et des ressources (JVM & GC)
Un aspect fondamental de l'administration Jenkins concerne l'optimisation Java (JVM) :

- **Garbage Collection** : Préférer l'utilisation du Garbage Collector **G1GC**.
- **Arguments JVM recommandés** :
  ```text
  -Xms2g -Xmx4g -XX:+UseG1GC -XX:+ExplicitGCInvokesConcurrent
  ```
- **Politique de Rétention des Builds** : Toujours activer l'option *Discard Old Builds* (Conserver uniquement les N derniers builds) pour éviter d'importer des millions de petits fichiers sur le disque.

### 4. Bonnes Pratiques de Sécurité
- **Principe du Moindre Privilège** : Utiliser la matrice de sécurité (Role-Based Authorization Strategy).
- **Agents Dédiés** : Ne **JAMAIS** exécuter de builds lourds ou non sécurisés sur l'agent `Built-in` (Master). Le Master doit uniquement orchestrer.
- **CSRF Protection** : S'assurer que le *Crumb Issuer* est activé.

---

## 🧾 Résumé des Composants d'Administration

| Élément / Concept | Rôle / Description |
| :--- | :--- |
| **`JENKINS_HOME`** | Répertoire maître contenant toutes les données critiques de Jenkins. |
| **`secrets/`** | Dossier indispensable contenant les clés de déchiffrement des credentials. |
| **ThinBackup / Backup Plugin** | Plugins Jenkins permettant de planifier des sauvegardes sans script externe. |
| **Rétention des Builds** | Mécanisme évitant la saturation de l'espace disque du nœud master. |
| **G1GC** | Garbage Collector recommandé pour éviter les freezes lors des sauvegardes. |
| **Slave / Agent** | Machine distante exécutant les tâches pour protéger le serveur Master. |
| **RTO (Recovery Time Objective)** | Durée maximale acceptable d'interruption du service Jenkins. |
| **RPO (Recovery Point Objective)** | Perte maximale de données acceptable (déterminée par la fréquence du backup). |