# Timeline Formation Jenkins – 2 jours

## Jour 1 – Pipelines, Tests et Déploiement

**Horaire : 09h15 – 17h00**

| Heure | Durée | Chapitre / Activité | Contenu / Concepts | Type |
|---|---:|---|---|---|
| **09:15 - 09:35** | 20 min | **Icebreaker – Le Pipeline humain** | Présentation rapide, CI/CD, remise en ordre des étapes d’un Pipeline, identification d’erreurs | Icebreaker |
| **09:35 - 10:00** | 25 min | **1. Pipelines : mise en place** | Jenkins, CI/CD, architecture Jenkins, Controller/Agent, principe d’un Pipeline | Théorie |
| **10:00 - 10:20** | 20 min | **1. Pipelines : mise en place** | Groovy, DSL, Jenkinsfile, Declarative vs Scripted Pipeline | Théorie |
| **10:20 - 10:30** | 10 min | **1. Pipelines : mise en place** | `pipeline`, `agent`, `stages`, `stage`, `steps` | Démo |
| **10:30 - 10:45** | 15 min | **TP 1 – Premier Pipeline** | Création d’un Jenkinsfile, exécution, logs | Pratique |
| **10:45 - 11:00** | 15 min | **Pause** | | |
| **11:00 - 11:20** | 20 min | **2. Pipelines : configuration avancée** | Plugins, configuration globale, paramètres, Credentials | Théorie |
| **11:20 - 11:40** | 20 min | **2. Pipelines : configuration avancée** | `post`, erreurs, `retry`, `timeout`, `when`, artefacts, fingerprints | Théorie |
| **11:40 - 12:00** | 20 min | **TP 2 – Pipeline avancé** | `post`, artefacts, Credentials, gestion d’erreur, timeout | Pratique |
| **12:00 - 13:00** | 60 min | **Pause déjeuner** | | |
| **13:00 - 13:20** | 20 min | **2. Pipelines : exécution avancée** | Parallélisation, Matrix Pipeline, Docker comme agent | Théorie / Démo |
| **13:20 - 13:50** | 30 min | **TP 3 – Exécution avancée** | `parallel`, Matrix Pipeline, agent Docker | Pratique |
| **13:50 - 14:10** | 20 min | **3. Automatisation des tests** | Tests automatisés, Build → Test → Report, intégration JUnit | Théorie |
| **14:10 - 14:30** | 20 min | **TP 4 – Tests automatisés** | Maven test, résultats JUnit, publication des rapports | Pratique |
| **14:30 - 14:45** | 15 min | **Pause** | | |
| **14:45 - 15:05** | 20 min | **3. Automatisation des tests** | Couverture de code, qualité, introduction JMeter | Théorie |
| **15:05 - 15:25** | 20 min | **TP 5 – Analyse des résultats** | Build en échec si tests KO, `post`, analyse des rapports | Pratique |
| **15:25 - 15:40** | 15 min | **4. Automatisation du déploiement** | Maven lifecycle, package, artefacts, stratégie de déploiement | Théorie |
| **15:40 - 16:00** | 20 min | **TP 6 – Build & Package** | Maven build, génération du JAR, archivage de l’artefact | Pratique |
| **16:00 - 16:15** | 15 min | **4. Automatisation du déploiement** | Dockerfile, Docker Build, tags, Registry | Théorie / Démo |
| **16:15 - 16:40** | 25 min | **TP 7 – Docker** | Build image Docker, tagging et publication dans un Registry | Pratique |
| **16:40 - 17:00** | 20 min | **TP de synthèse Jour 1** | Checkout → Build → Test → Package → Docker Build → Docker Push → Artefacts | Pratique |

---

## Jour 2 – GitLab, Shared Library, Administration, Agents et Sauvegarde

**Horaire : 09h00 – 17h00**

| Heure | Durée | Chapitre / Activité | Contenu / Concepts | Type |
|---|---:|---|---|---|
| **09:00 - 09:20** | 20 min | **5. Gestion de branches avec GitLab** | GitFlow, branches, Feature Branch, Develop, Main, Merge Request | Théorie |
| **09:20 - 09:40** | 20 min | **5. Gestion de branches avec GitLab** | Jenkins/GitLab, Webhook, Multibranch Pipeline, découverte automatique des branches | Théorie |
| **09:40 - 10:15** | 35 min | **TP 8 – GitLab + Multibranch** | Création d’un Multibranch Pipeline, connexion GitLab, branche, build | Pratique |
| **10:15 - 10:30** | 15 min | **Pause** | | |
| **10:30 - 10:50** | 20 min | **6. Mutualisation des Pipelines** | Shared Library, architecture d’une librairie Jenkins | Théorie |
| **10:50 - 11:05** | 15 min | **6. Mutualisation des Pipelines** | `vars/`, `src/`, `resources/`, fonctions Groovy, versionnement | Théorie |
| **11:05 - 11:30** | 25 min | **TP 9 – Shared Library** | Fonction réutilisable, appel depuis Jenkinsfile, factorisation | Pratique |
| **11:30 - 12:00** | 30 min | **TP 10 – Notifications** | Notification succès/échec, Slack ou Email | Pratique |
| **12:00 - 13:00** | 60 min | **Pause déjeuner** | | |
| **13:00 - 13:15** | 15 min | **7. Administration Jenkins** | Architecture Jenkins, `$JENKINS_HOME`, jobs, plugins | Théorie |
| **13:15 - 13:30** | 15 min | **7. Administration Jenkins** | Sécurité, utilisateurs, rôles, Credentials, logs, monitoring, espace disque | Théorie |
| **13:30 - 13:45** | 15 min | **TP 11 – Administration** | Utilisateur, permissions, logs, utilisation disque | Pratique |
| **13:45 - 14:00** | 15 min | **8. Agents Jenkins** | Controller vs Agent, labels, executors, exécution distribuée | Théorie |
| **14:00 - 14:30** | 30 min | **TP 12 – Agents Jenkins** | Configuration d’un agent, label, executor, exécution d’un Pipeline | Pratique |
| **14:30 - 14:45** | 15 min | **Pause** | | |
| **14:45 - 15:00** | 15 min | **9. Stratégie de sauvegarde** | `$JENKINS_HOME`, jobs, Credentials, plugins, configurations | Théorie |
| **15:00 - 15:15** | 15 min | **9. Stratégie de sauvegarde** | Backup/Restore, fréquence, rétention, restauration, plan de reprise | Théorie |
| **15:15 - 15:30** | 15 min | **TP 13 – Backup / Restore** | Sauvegarde Jenkins, éléments critiques, simulation de restauration | Pratique |
| **15:30 - 16:30** | 60 min | **Fil rouge – Projet final** | GitLab → Multibranch → Jenkins → Maven → JUnit → SonarQube → Nexus → Docker → Déploiement | Pratique |
| **16:30 - 17:00** | 30 min | **Validation finale** | Exécution complète, analyse des logs, tests, artefacts, Docker, résultats et Q/R | Pratique / Conclusion |