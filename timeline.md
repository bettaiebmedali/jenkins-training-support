# Timeline Formation Jenkins –  (2 jours)

## Jour 1 – Pipelines, Tests et Déploiement

| Heure | Chapitre / Contenu | Type |
|---|---|---|
| **09:00 - 10:00** | **1. Pipelines : mise en place** – Présentation de Jenkins, CI/CD, bénéfices, cas d’usage, Groovy, DSL, Jenkinsfile, pipeline déclaratif vs scripté | Théorie |
| **10:00 - 10:30** | **1. Pipelines : mise en place – TP** – Création et exécution d’un pipeline déclaratif, logs, modification/rejeu, paramètres, variables d’environnement et strings | Pratique |
| **10:30 - 10:45** | **Pause** | – |
| **10:45 - 12:00** | **2. Pipelines : configuration avancée** – Plugins, propriétés globales, artefacts, fingerprints, credentials, gestion des erreurs, parallélisation, Matrix Pipeline, Docker comme agent | Théorie + Pratique |
| **12:00 - 13:00** | **Pause déjeuner** | – |
| **13:00 - 14:30** | **3. Automatisation des tests** – Intégration JUnit, publication des rapports, couverture de code, introduction aux tests de performance avec JMeter | Théorie + Pratique |
| **14:30 - 14:45** | **Pause** | – |
| **14:45 - 16:15** | **4. Automatisation du déploiement** – Maven, génération d’images Docker, publication d’artefacts, scripts de déploiement, rollback | Théorie + Pratique |
| **16:15 - 17:00** | **TP de synthèse Jour 1** – Pipeline CI : Build → Tests → Package → Docker → Publication des artefacts | Pratique |

---

## Jour 2 – GitLab, Shared Library, Administration, Agents et Sauvegarde

| Heure | Chapitre / Contenu | Type |
|---|---|---|
| **09:00 - 10:15** | **5. Gestion de branches avec GitLab** – GitFlow, intégration Jenkins/GitLab, pipeline Multibranch, Merge Requests, builds conditionnels | Théorie + Pratique |
| **10:15 - 10:30** | **Pause** | – |
| **10:30 - 11:30** | **6. Mutualisation des pipelines** – Shared Library, fonctions Groovy réutilisables, notifications Slack/Email | Théorie + Pratique |
| **11:30 - 12:00** | **7. Administration Jenkins** – Sécurité, utilisateurs et rôles, journalisation, monitoring, gestion de l’espace disque | Théorie |
| **12:00 - 13:00** | **Pause déjeuner** | – |
| **13:00 - 13:45** | **8. Agents Jenkins** – Installation et configuration des agents, exécution distribuée, labels, executors, stratégie de répartition des jobs | Théorie + Pratique |
| **13:45 - 14:30** | **9. Stratégie de sauvegarde** – Anatomie de `$JENKINS_HOME`, sauvegarde, restauration, éléments à sauvegarder, plan de reprise | Théorie + Pratique |
| **14:30 - 14:45** | **Pause** | – |
| **14:45 - 16:30** | **Fil rouge – Projet final** – GitLab (GitFlow + Multibranch) → Jenkins Pipeline → Maven → JUnit → SonarQube → Nexus → Docker → Déploiement automatisé | Pratique |
| **16:30 - 17:00** | **Fil rouge – Validation finale** – Tests, démonstration du pipeline complet, analyse des résultats, bonnes pratiques et questions/réponses | Pratique / Conclusion |

---

## Organisation de la formation

- Formation condensée sur **2 jours**.
- Horaires : **09h00 – 17h00**.
- Pause matin : **10h30 – 10h45**.
- Pause déjeuner : **12h00 – 13h00**.
- Pause après-midi : **14h30 – 14h45**.
- Chaque partie théorique est suivie, lorsque pertinent, d'une mise en pratique.
- **Jour 1 :** construction progressive d'un pipeline CI/CD, des bases jusqu'au déploiement.
- **Jour 2 matin :** GitLab, mutualisation des pipelines et administration Jenkins.
- **Jour 2 début d'après-midi :** agents Jenkins et stratégie de sauvegarde.
- **Dernier après-midi :** fil rouge permettant de mettre en œuvre l'ensemble des compétences acquises.