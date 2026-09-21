# Administration Jenkins 🛡️

## Contexte / Storytelling
La startup GFP Tech gère maintenant plusieurs pipelines et agents Jenkins.  
Pour garantir la **sécurité, la fiabilité et la performance** du système, il est essentiel de mettre en place des pratiques d’administration : gestion des utilisateurs, monitoring, sauvegardes et plans de reprise d’activité.

---

## Objectifs pédagogiques
- Configurer la **sécurité Jenkins** et gérer les utilisateurs et rôles.  
- Mettre en place le **monitoring** des ressources (CPU, RAM, disque).  
- Configurer la **sauvegarde automatique** du `$JENKINS_HOME`.  
- Tester un **plan de reprise** en cas d’incident.

---

## Prérequis techniques
- Jenkins installé et fonctionnel avec pipelines existants (Chapitre 1 à 5).  
- Accès administrateur Jenkins.  
- Accès SSH ou console pour la gestion des agents et sauvegardes.  

---

## Étapes détaillées

### **1. Activer la sécurité Jenkins**
1. Aller dans **Manage Jenkins → Configure Global Security**.  
2. Activer :  
   - Jenkins own user database  
   - Matrix-based security (RBAC)  
3. Créer des utilisateurs avec différents rôles :  
   - Admin : accès complet  
   - Developer : accès pipelines  
   - Viewer : lecture seule  

### **2. Configurer les permissions et rôles**
1. Définir les permissions par type d’utilisateur (ex: `Job → Build` pour Developer, `Administer` pour Admin).  
2. Vérifier que les utilisateurs ne peuvent accéder qu’aux pipelines autorisés.

### **3. Monitoring et points de stress**
1. Installer plugin **Monitoring** ou **Metrics**.  
2. Suivre CPU, RAM et espace disque Jenkins.  
3. Simuler un build lourd pour observer la charge.  
4. Ajouter alertes ou notifications si seuils dépassés.

### **4. Sauvegarde automatique**
1. Créer un script `backup_jenkins.sh` :

```bash
#!/bin/bash
tar -czf /backups/jenkins_home_$(date +%F_%H-%M).tar.gz $JENKINS_HOME
```
2. Ajouter un **cron job** sur le serveur Jenkins :

```bash
0 2 * * * /path/to/backup_jenkins.sh
```

### **5. Plan de reprise / rollback**
1. Arrêter Jenkins.  
2. Restaurer le backup le plus récent :  
   ```bash
   tar -xzf /backups/jenkins_home_latest.tar.gz -C $JENKINS_HOME
   systemctl restart jenkins
   ```  
3. Vérifier que les pipelines et agents sont fonctionnels.

### **6. Intégration avec le projet fil rouge**
1. Vérifier que le pipeline multibranch fonctionne même après restauration.  
2. Simuler un incident : supprimer un build ou un agent et restaurer.  
3. Observer les logs et notifications.

---

## Livrable final
- Jenkins sécurisé avec utilisateurs et rôles configurés.  
- Monitoring actif et seuils configurés.  
- Sauvegarde automatique du `$JENKINS_HOME`.  
- Plan de reprise testé avec pipeline multibranch opérationnel.

---

## Bonus (optionnel)
- Mettre en place des **snapshots réguliers des agents** pour accélérer la récupération.  
- Ajouter des **dashboards personnalisés** pour le suivi des ressources et builds.  
- Déployer un **agent temporaire** pour tester la montée en charge.
~~~

