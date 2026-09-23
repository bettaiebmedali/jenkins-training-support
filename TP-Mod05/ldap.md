# TP — Jenkins + LDAP avec Docker Compose

## 1. Objectifs

Dans ce TP, nous allons :

- Déployer Jenkins avec Docker Compose.
- Déployer un serveur OpenLDAP dans un autre conteneur.
- Déployer phpLDAPadmin pour administrer LDAP.
- Créer quelques utilisateurs LDAP simples.
- Configurer Jenkins pour utiliser LDAP.
- Tester la connexion Jenkins avec les utilisateurs LDAP.

---

# 2. Architecture

```text
                    Docker Compose
                         |
              jenkins-network
             /        |        \
            /         |         \
           /          |          \
     Jenkins       OpenLDAP    phpLDAPadmin
     :8080           :389         :8081
       |               |
       |               |
       +------ LDAP ---+
```

Les services sont dans le même réseau Docker.

---

# 3. Prérequis

Vérifier Docker :

```bash
docker --version
docker compose version
```

---

# 4. Créer le projet

```bash
mkdir jenkins-ldap-lab
cd jenkins-ldap-lab
```

Créer le fichier :

```bash
touch compose.yaml
```

---

# 5. Fichier Docker Compose

Mettre le contenu suivant dans `compose.yaml` :

```yaml
services:

  ldap:
    image: osixia/openldap:1.5.0
    container_name: ldap
    restart: unless-stopped

    environment:
      LDAP_ORGANISATION: "Training"
      LDAP_DOMAIN: "example.local"
      LDAP_ADMIN_PASSWORD: "admin123"

    ports:
      - "389:389"

    volumes:
      - ldap_data:/var/lib/ldap
      - ldap_config:/etc/ldap/slapd.d

    networks:
      - jenkins-network


  phpldapadmin:
    image: osixia/phpldapadmin:0.9.0
    container_name: phpldapadmin
    restart: unless-stopped

    environment:
      PHPLDAPADMIN_LDAP_HOSTS: ldap
      PHPLDAPADMIN_HTTPS: "false"

    ports:
      - "8081:80"

    depends_on:
      - ldap

    networks:
      - jenkins-network


  jenkins:
    image: jenkins/jenkins:lts-jdk21
    container_name: jenkins
    restart: unless-stopped

    ports:
      - "8080:8080"
      - "50000:50000"

    volumes:
      - jenkins_home:/var/jenkins_home

    depends_on:
      - ldap

    networks:
      - jenkins-network


networks:
  jenkins-network:


volumes:
  jenkins_home:
  ldap_data:
  ldap_config:
```

---

# 6. Démarrer les conteneurs

```bash
docker compose up -d
```

Vérifier :

```bash
docker compose ps
```

Résultat attendu :

```text
NAME
jenkins
ldap
phpldapadmin
```

Les trois conteneurs doivent être `Up`.

---

# 7. Tester Jenkins

Ouvrir :

```text
http://localhost:8080
```

Ou tester avec :

```bash
curl -I http://localhost:8080
```

Résultat attendu :

```text
HTTP/1.1 200 OK
```

---

# 8. Tester phpLDAPadmin

Ouvrir :

```text
http://localhost:8081
```

Page de connexion LDAP.

Utiliser :

```text
Login DN:
cn=admin,dc=example,dc=local

Password:
admin123
```

---

# 9. Comprendre notre LDAP

Notre domaine LDAP est :

```text
dc=example,dc=local
```

L'utilisateur administrateur est :

```text
cn=admin,dc=example,dc=local
```

Nous allons créer cette structure :

```text
dc=example,dc=local
└── ou=Users
    ├── uid=devops
    ├── uid=developer
    └── uid=manager
```

---

# 10. Créer l'OU Users

Dans phpLDAPadmin :

```text
dc=example,dc=local
        |
        +-- Create a child
```

Choisir :

```text
Generic: Organisational Unit
```

Mettre :

```text
ou = Users
```

Créer l'objet.

La structure devient :

```text
dc=example,dc=local
└── ou=Users
```

---

# 11. Créer l'utilisateur devops

Entrer dans :

```text
ou=Users
```

Puis :

```text
Create a child
```

Choisir :

```text
inetOrgPerson
```

Renseigner :

```text
RDN:
uid=devops

uid:
devops

cn:
DevOps User

sn:
User

userPassword:
Devops123
```

Créer l'utilisateur.

Résultat :

```text
uid=devops,ou=Users,dc=example,dc=local
```

---

# 12. Créer l'utilisateur developer

Même procédure.

```text
RDN:
uid=developer

uid:
developer

cn:
Developer User

sn:
User

userPassword:
Dev12345
```

Résultat :

```text
uid=developer,ou=Users,dc=example,dc=local
```

---

# 13. Créer l'utilisateur manager

Même procédure.

```text
RDN:
uid=manager

uid:
manager

cn:
Manager User

sn:
User

userPassword:
Manager123
```

Résultat :

```text
uid=manager,ou=Users,dc=example,dc=local
```

---

# 14. Vérifier les utilisateurs LDAP

Depuis le terminal :

```bash
docker exec ldap ldapsearch \
  -x \
  -H ldap://localhost:389 \
  -D "cn=admin,dc=example,dc=local" \
  -w admin123 \
  -b "ou=Users,dc=example,dc=local" \
  "(objectClass=inetOrgPerson)" \
  uid cn
```

On doit retrouver :

```text
uid: devops
cn: DevOps User

uid: developer
cn: Developer User

uid: manager
cn: Manager User
```

---

# 15. Tester l'authentification LDAP

Tester directement l'utilisateur `devops` :

```bash
docker exec ldap ldapwhoami \
  -x \
  -H ldap://localhost:389 \
  -D "uid=devops,ou=Users,dc=example,dc=local" \
  -w Devops123
```

Résultat attendu :

```text
dn:uid=devops,ou=Users,dc=example,dc=local
```

Tester `developer` :

```bash
docker exec ldap ldapwhoami \
  -x \
  -H ldap://localhost:389 \
  -D "uid=developer,ou=Users,dc=example,dc=local" \
  -w Dev12345
```

Tester `manager` :

```bash
docker exec ldap ldapwhoami \
  -x \
  -H ldap://localhost:389 \
  -D "uid=manager,ou=Users,dc=example,dc=local" \
  -w Manager123
```

Si les trois commandes retournent un DN, LDAP fonctionne correctement.

---

# 16. Installer le plugin LDAP dans Jenkins

Se connecter à Jenkins :

```text
http://localhost:8080
```

Aller dans :

```text
Manage Jenkins
```

Puis :

```text
Plugins
```

Rechercher :

```text
LDAP
```

Installer :

```text
LDAP Plugin
```

Redémarrer Jenkins si nécessaire.

---

# 17. Configurer le Security Realm

Dans Jenkins :

```text
Manage Jenkins
```

Puis :

```text
Security
```

Dans :

```text
Security Realm
```

Sélectionner :

```text
LDAP
```

---

# 18. Configuration LDAP Jenkins

### Server

Mettre :

```text
ldap://ldap:389
```

Attention :

```text
ldap
```

est le nom du conteneur Docker.

Il ne faut pas mettre :

```text
localhost
```

Pourquoi ?

Depuis le conteneur Jenkins :

```text
localhost
```

désigne Jenkins lui-même.

Le serveur LDAP est accessible avec :

```text
ldap
```

---

# 19. Root DN

Mettre :

```text
dc=example,dc=local
```

---

# 20. User Search Base

Mettre :

```text
ou=Users
```

---

# 21. User Search Filter

Mettre :

```text
uid={0}
```

Le `{0}` sera remplacé par le login fourni par l'utilisateur.

Par exemple :

```text
devops
```

devient :

```text
(uid=devops)
```

---

# 22. Manager DN

Pour ce TP, utiliser le compte LDAP administrateur :

```text
cn=admin,dc=example,dc=local
```

Password :

```text
admin123
```

---

# 23. Attribut Display Name

Mettre :

```text
cn
```

---

# 24. Configuration complète

La configuration Jenkins doit donc être :

```text
LDAP Server:
ldap://ldap:389

Root DN:
dc=example,dc=local

User Search Base:
ou=Users

User Search Filter:
uid={0}

Manager DN:
cn=admin,dc=example,dc=local

Manager Password:
admin123

Display Name Attribute:
cn
```

---

# 25. Tester la connexion LDAP dans Jenkins

Dans la configuration LDAP de Jenkins, utiliser la fonction :

```text
Test LDAP configuration
```

Tester avec :

```text
Username:
devops

Password:
Devops123
```

Le test doit être réussi.

---

# 26. Activer LDAP comme authentification Jenkins

Sauvegarder la configuration.

Se déconnecter de Jenkins.

Puis essayer :

```text
Username:
devops

Password:
Devops123
```

Jenkins doit maintenant authentifier l'utilisateur via LDAP.

---

# 27. Tester les autres utilisateurs

## Developer

```text
Username:
developer

Password:
Dev12345
```

## Manager

```text
Username:
manager

Password:
Manager123
```

Les trois utilisateurs doivent pouvoir se connecter à Jenkins.

---

# 28. Vérifier le fonctionnement

Architecture finale :

```text
                  Jenkins
                localhost:8080
                     |
                     |
              LDAP Authentication
                     |
                     v
              ldap:389
                     |
                     |
          +----------+----------+
          |          |          |
       devops    developer    manager
```

---

# 29. Test final

Effectuer les tests suivants.

### Test 1 — Jenkins

```bash
curl -I http://localhost:8080
```

Résultat :

```text
HTTP/1.1 200 OK
```

### Test 2 — phpLDAPadmin

```bash
curl -I http://localhost:8081
```

Résultat :

```text
HTTP/1.1 200 OK
```

### Test 3 — LDAP

```bash
docker exec ldap ldapsearch \
  -x \
  -H ldap://localhost:389 \
  -D "cn=admin,dc=example,dc=local" \
  -w admin123 \
  -b "dc=example,dc=local"
```

Résultat :

```text
result: 0 Success
```

### Test 4 — authentification devops

```bash
docker exec ldap ldapwhoami \
  -x \
  -H ldap://localhost:389 \
  -D "uid=devops,ou=Users,dc=example,dc=local" \
  -w Devops123
```

Résultat :

```text
dn:uid=devops,ou=Users,dc=example,dc=local
```

### Test 5 — Jenkins

Se connecter avec :

```text
devops / Devops123
```

---

# 30. Dépannage

## Jenkins ne démarre pas

```bash
docker compose ps
```

Puis :

```bash
docker compose logs jenkins
```

---

## LDAP ne démarre pas

```bash
docker compose logs ldap
```

Tester :

```bash
docker exec ldap ldapsearch \
  -x \
  -H ldap://localhost:389 \
  -D "cn=admin,dc=example,dc=local" \
  -w admin123 \
  -b "dc=example,dc=local"
```

---

## Jenkins ne trouve pas LDAP

Depuis Jenkins :

```bash
docker exec -it jenkins bash
```

Tester la résolution DNS :

```bash
getent hosts ldap
```

Résultat attendu :

```text
<IP_DOCKER> ldap
```

Tester le port LDAP :

```bash
apt-get update
apt-get install -y netcat-openbsd
```

Puis :

```bash
nc -zv ldap 389
```

Résultat attendu :

```text
Connection to ldap 389 port [tcp/ldap] succeeded!
```

---

# 31. Nettoyage du TP

Pour arrêter les conteneurs :

```bash
docker compose down
```

Pour supprimer également les données LDAP et Jenkins :

```bash
docker compose down -v
```

Attention :

```bash
docker compose down -v
```

supprime les volumes et donc :

- les données Jenkins ;
- les utilisateurs LDAP ;
- la configuration LDAP.

---

# 32. Résultat attendu

À la fin du TP, nous avons :

```text
Docker Compose
      |
      +-------------------+
      |                   |
   Jenkins              LDAP
   :8080                 :389
      |                   |
      |              ou=Users
      |                   |
      |          +--------+--------+
      |          |        |        |
      |       devops  developer  manager
      |          |
      +----------+
       Authentification
```

Jenkins utilise maintenant **OpenLDAP comme source d'authentification**.

Les comptes Jenkins ne sont plus créés localement dans Jenkins :

```text
Utilisateur
     |
     v
   Jenkins
     |
     v
   OpenLDAP
     |
     v
Validation login/password
```

---

# 33. Challenge supplémentaire

Une fois le TP terminé, ajouter :

- un groupe LDAP `jenkins-admins` ;
- un groupe LDAP `developers` ;
- des droits Jenkins différents selon les groupes ;
- un utilisateur qui appartient à `developers` ;
- un utilisateur qui appartient à `jenkins-admins`.

Objectif :

```text
LDAP Group
     |
     v
Jenkins Authorization
     |
     +-- jenkins-admins --> Administration
     |
     +-- developers -----> Build / Read
```

Ce challenge permet de passer de la simple **authentification LDAP** à la **gestion des autorisations Jenkins avec LDAP**.