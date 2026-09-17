# TP Jenkins --- Gestion des Vues (Views) et Organisation des Jobs

Compatible : **Jenkins 2.528.2**

Ce TP vous apprendra à créer, organiser et personnaliser des **views**
dans Jenkins afin de structurer vos jobs par équipes, par projets ou par
pipelines.

------------------------------------------------------------------------

## 🎯 Objectifs du TP

-   Créer plusieurs types de *Views* Jenkins\
-   Organiser les jobs dans des vues séparées\
-   Filtrer dynamiquement les jobs avec des regex\
-   Créer un tableau de bord clair par département (Dev, QA, Ops)\
-   Bonus : créer une *Nested View* hiérarchique

------------------------------------------------------------------------

# 🧩 1. Préparation de l'environnement

### ✔ Prérequis

-   Jenkins installé\
-   Plugins recommandés :
    -   **Nested View** (optionnel)\
    -   **CloudBees Folders** (optionnel)

### ✔ Jobs utilisés dans le TP

Créez les jobs suivants (Pipeline ou Freestyle, au choix) : -
`dev-build-backend` - `dev-build-frontend` - `qa-test-backend` -
`qa-test-frontend` - `ops-deploy-prod` - `ops-maintenance`

------------------------------------------------------------------------

# 🧩 2. Créer une Vue simple

### 🎯 Objectif : créer une vue « Développeurs »

1.  Aller dans **Dashboard**
2.  Cliquer sur **+ New View**
3.  Nom : `DEV`
4.  Type : **List View**
5.  Cliquer sur **OK**

### Configuration :

-   Section **Job Filters**\

-   Cocher : **Use a regular expression**

-   Regex :

        ^dev-.*$

### Résultat attendu :

La vue affiche uniquement : - `dev-build-backend` - `dev-build-frontend`

------------------------------------------------------------------------

# 🧩 3. Créer une Vue QA

### 🎯 Objectif : filtrer les jobs QA uniquement

1.  Dashboard → **+ New View**
2.  Nom : `QA`
3.  Type : **List View** → OK

### Configuration :

-   **Use a regular expression**

-   Regex :

        ^qa-.*$

Résultat attendu : - `qa-test-backend` - `qa-test-frontend`

------------------------------------------------------------------------

# 🧩 4. Créer une Vue pour les Ops

### 🎯 Objectif : organiser les jobs de production

1.  Dashboard → **+ New View**
2.  Nom : `OPS`
3.  Type : **List View**

### Configuration :

-   Regex :

        ^ops-.*$

Résultat attendu : - `ops-deploy-prod` - `ops-maintenance`

------------------------------------------------------------------------

# 🧩 5. Vue personnalisée avec filtres avancés

### 🎯 Objectif : créer une vue pour tout ce qui touche au backend

1.  Dashboard → **+ New View**

2.  Nom : `Backend`

3.  Type : **List View**

4.  Filtres → Regex :

        .*backend.*

Résultat attendu : - `dev-build-backend` - `qa-test-backend`

------------------------------------------------------------------------

# 🧩 6. Créer une Vue « Tous les Pipelines »

### 🎯 Objectif : isoler uniquement les jobs Pipeline

Configurer une regex pour matcher votre type de jobs Pipeline (exemple :
ils contiennent "pipeline")

    pipeline

------------------------------------------------------------------------

# 🧩 7. Créer une *Nested View* (Vue hiérarchique)

### 🎯 Objectif : un dashboard organisé par département

1.  Dashboard → **+ New View**
2.  Type : **Nested View**
3.  Nom : `ENTREPRISE`

À l'intérieur : - Créer sous-vue : `DEV` - Créer sous-vue : `QA` - Créer
sous-vue : `OPS`

Chaque sous-vue utilise les regex créées dans les étapes précédentes.

### Arborescence finale :

    ENTREPRISE
     ├── DEV
     ├── QA
     └── OPS

------------------------------------------------------------------------




# 🧩 8. BONUS --- Ajouter des colonnes personnalisées

Dans chaque List View : - Build description\
- Last failure\
- Build parameters\
- Pipeline Step summary\
- Git branch

Dashboard → Configure → **Columns**

------------------------------------------------------------------------

# 🎉 Fin du TP

Vous avez maintenant un Jenkins organisé, propre et structuré avec
plusieurs vues efficaces.


