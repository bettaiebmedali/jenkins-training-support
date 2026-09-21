# Automatisation du déploiement 🚀

## Contexte / Storytelling
La startup GFP Tech a maintenant un pipeline complet pour les builds et tests.  
Il est temps de passer à **l’automatisation du déploiement** afin de livrer rapidement les artefacts sur les environnements de staging et production, tout en assurant des mécanismes de rollback sécurisés.

---

## Objectifs pédagogiques
- Automatiser le déploiement d’un artefact Maven ou Docker.  
- Mettre en place des scripts de déploiement fiables.  
- Intégrer un mécanisme de rollback simple pour revenir à une version stable.  
- Déployer sur différents environnements selon les paramètres.

---

## Prérequis techniques
- Jenkins avec pipelines Chapitre 1 à 4.  
- Projet Maven avec build et tests automatisés.  
- Docker installé sur l’agent ou le serveur cible.  
- Accès SSH à un environnement staging ou serveur de test.

---

## Étapes détaillées

### **1. Préparer le script de déploiement**
Créer un script `deploy.sh` dans le projet :

```bash
#!/bin/bash
ENV=$1
IMAGE_TAG=$2

echo "Déploiement sur l'environnement $ENV avec l'image $IMAGE_TAG"

docker pull gfptech/app:$IMAGE_TAG || { echo "Échec pull"; exit 1; }
docker stop app-$ENV || true
docker rm app-$ENV || true
docker run -d --name app-$ENV -p 8080:8080 gfptech/app:$IMAGE_TAG
```
- Paramètres : environnement (`dev`, `staging`, `prod`) et tag Docker.

### **2. Modifier le Jenkinsfile pour inclure le déploiement**
Ajouter un stage dans le pipeline :

```groovy
stage('Deploy') {
    steps {
        script {
            def envTarget = params.ENV ?: 'staging'
            def imageTag = "${env.BUILD_NUMBER}"
            sh "./deploy.sh ${envTarget} ${imageTag}"
        }
    }
    post {
        success {
            echo "✅ Déploiement réussi"
        }
        failure {
            echo "⚠️ Échec du déploiement, rollback nécessaire"
        }
    }
}
```

### **3. Ajouter un mécanisme de rollback simple**
1. Conserver la dernière image stable Docker.  
2. En cas d’échec, exécuter :

```bash
docker stop app-$ENV
docker rm app-$ENV
docker run -d --name app-$ENV gfptech/app:last-stable
```

3. Optionnel : notifier l’équipe via Slack ou Email.

### **4. Tester le pipeline**
1. Déclencher le pipeline sur l’environnement `staging`.  
2. Vérifier que l’image Docker est déployée correctement.  
3. Simuler un échec (ex: tag incorrect) et observer le rollback.  
4. Tester les notifications et logs.

---

## Livrable final
- Pipeline Jenkins avec :  
  - Build et tests automatisés  
  - Déploiement Docker automatisé sur environnement staging/production  
  - Rollback automatique en cas d’échec  
- Script `deploy.sh` fonctionnel et testé  
- Logs et notifications disponibles pour l’équipe

---

## Bonus (optionnel)
- Déploiement conditionnel selon branche (feature → staging, release → prod).  
- Déploiement multistage avec Canary ou Blue-Green.  
- Ajout d’une étape de migration base de données si nécessaire.
~~~

