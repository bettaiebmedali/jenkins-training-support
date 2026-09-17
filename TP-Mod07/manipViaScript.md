# TP — Manipulation des Jobs Jenkins via Script Console (Move, Scan, Folder)

Ce TP vous permet d’apprendre à :

- Exécuter un script Groovy dans **Jenkins Script Console**
- Déplacer automatiquement des jobs
- Créer des dossiers dynamiquement
- Faire un audit des jobs / folders
- Ajouter des use cases avancés (copie, renommage, verrouillage, suppression)

Compatible : Jenkins 2.528+

---

## 1. Pré-requis

- Accès à **Manage Jenkins → Script Console**
- Rôle administrateur Jenkins
- Des jobs existants à la racine
- Plugin **Folders** installé

---

## 2. Exécution du script (rappel)

Ouvrez :

Manage Jenkins → Script Console

kotlin
Copy code

Collez un script Groovy puis exécutez.

⚠️ Les scripts agissent **en direct** sur l’instance.

---

# 3. TP — Déplacer un job dans un folder (script utilisé)

## 🟦 Script : déplacer un job dans un folder

```groovy
import jenkins.model.*
import com.cloudbees.hudson.plugins.folder.*

def jenkins = Jenkins.instance
def jobName = "alljobs"
def folderName = "PROJETS-Formation-17112025"

// chercher le job
def job = jenkins.getItem(jobName)
if (!job) {
    println "❌ Job '${jobName}' introuvable."
    return
}

// chercher le folder
def folder = jenkins.getItem(folderName)
if (!folder) {
    println "❌ Folder '${folderName}' introuvable."
    return
}

// déplacer
println "Déplacement du job '${jobName}' vers '${folderName}'..."
jenkins.move(job, folder)
println "✔ Déplacement terminé."
4. Use Case #1 — Déplacer TOUS les jobs dont le nom commence par « poste »
🟦 Script
groovy
Copy code
import jenkins.model.*
import com.cloudbees.hudson.plugins.folder.*

def jenkins = Jenkins.instance
def folder = jenkins.getItem("POSTE-JOBS") ?: jenkins.createProject(com.cloudbees.hudson.plugins.folder.Folder, "POSTE-JOBS")

jenkins.items.each { item ->
    if (item.name =~ /^poste/) {
        println "Déplacement : ${item.name}"
        jenkins.move(item, folder)
    }
}
println "✔ Tous les jobs 'poste*' ont été déplacés."
5. Use Case #2 — Créer automatiquement un folder par équipe (Team1, Team2, …)
groovy
Copy code
import jenkins.model.*
import com.cloudbees.hudson.plugins.folder.*

def jenkins = Jenkins.instance
def teams = ["TEAM1", "TEAM2", "TEAM3"]

teams.each { t ->
    if (!jenkins.getItem(t)) {
        jenkins.createProject(Folder, t)
        println "✔ Folder créé : ${t}"
    }
}
6. Use Case #3 — Copier un job dans un folder (clone job)
groovy
Copy code
import jenkins.model.*
import hudson.model.*

def jenkins = Jenkins.instance
def origin = jenkins.getItem("job-template")
def folder = jenkins.getItem("TEAM1")

def newJob = Items.copy(origin, "job-template-copy")
folder.add(newJob, "job-template-copy")

println "✔ Job cloné dans TEAM1"
7. Use Case #4 — Renommer automatiquement des jobs
groovy
Copy code
import jenkins.model.*

def jenkins = Jenkins.instance

jenkins.items.each { job ->
    if (job.name.contains("dev-")) {
        def newName = job.name.replace("dev-", "develop-")
        println "Renommage : ${job.name} → ${newName}"
        job.renameTo(newName)
    }
}
println "✔ Opération terminée."
8. Use Case #5 — Auditer tous les jobs + leur emplacement
groovy
Copy code
import jenkins.model.*
import com.cloudbees.hudson.plugins.folder.*

def printTree
printTree = { item, level ->
    println ("  " * level) + "• " + item.name
    if (item instanceof com.cloudbees.hudson.plugins.folder.Folder) {
        item.items.each { sub ->
            printTree(sub, level + 1)
        }
    }
}

println "# Arborescence complète Jenkins :"
Jenkins.instance.items.each { rootItem ->
    printTree(rootItem, 0)
}
9. Use Case #6 — Supprimer tous les jobs marqués “deprecated”
groovy
Copy code
import jenkins.model.*

Jenkins.instance.items.each { job ->
    if (job.name.contains("deprecated")) {
        println "Suppression : ${job.name}"
        job.delete()
    }
}
println "✔ Tous les jobs 'deprecated' ont été supprimés."
10. Use Case #7 — Vérifier quels jobs n’ont pas été lancés depuis plus de 60 jours
groovy
Copy code
import jenkins.model.*

println "Jobs inactifs depuis +60 jours :"

Jenkins.instance.items.each { job ->
    def last = job.getLastBuild()?.getTimestamp()
    if (last) {
        long days = (System.currentTimeMillis() - last.timeInMillis) / (1000*60*60*24)
        if (days > 60) {
            println "• ${job.name} — ${days} jours"
        }
    }
}
11. Use Case #8 — Désactiver tous les jobs sauf production
groovy
Copy code
import jenkins.model.*

Jenkins.instance.items.each { job ->
    if (!job.name.contains("prod")) {
        println "Désactivation : ${job.name}"
        job.disable()
    }
}
println "✔ Tous les jobs non-prod sont désactivés."
12. Fin du TP
Vous savez désormais :

Exécuter un script dans Jenkins

Réorganiser automatiquement vos jobs

Gérer folders, copie, suppression, renommage

Faire des audits automatisés



deplacement des job par prefix 
def PREFIX = "poste00"


✅ SCRIPT GROOVY FINAL (fonctionne, sans Markdown, compatible Jenkins)
import jenkins.model.*
import hudson.model.*
import com.cloudbees.hudson.plugins.folder.*

// =====================
// PARAMETRE A MODIFIER
// =====================
def PREFIX = "poste00"

// Fonction récursive pour collecter les jobs dans tous les dossiers
def collectJobsRecursively(item, prefix, result) {
    if (item instanceof Folder) {
        item.getItems().each { subItem ->
            collectJobsRecursively(subItem, prefix, result)
        }
    } else if (item instanceof Job) {
        if (item.name.startsWith(prefix)) {
            result << item
        }
    }
}

def j = Jenkins.instance
def folderName = PREFIX

// 1. Création du folder si pas existant
def targetFolder = j.getItem(folderName)
if (targetFolder == null) {
    println "📁 Folder '${folderName}' introuvable, création..."
    targetFolder = new Folder(j, folderName)
    j.putItem(folderName, targetFolder)
    targetFolder.save()
    println "✅ Folder créé : ${folderName}"
} else {
    println "📁 Folder '${folderName}' déjà existant"
}

// 2. Recherche de tous les jobs partout
println "\n🔎 Recherche des jobs commençant par '${PREFIX}'..."
def jobsToMove = []
j.getItems().each { item ->
    collectJobsRecursively(item, PREFIX, jobsToMove)
}

if (jobsToMove.isEmpty()) {
    println "⚠️ Aucun job trouvé avec le préfixe '${PREFIX}'"
} else {
    println "📌 Jobs trouvés :"
    jobsToMove.each { println "   - ${it.fullName}" }
}

// 3. Déplacement des jobs
println "\n🚚 Déplacement vers '${folderName}'...\n"
jobsToMove.each { job ->
    try {
        Items.move(job, targetFolder)
        println "➡️ Déplacé : ${job.fullName}"
    } catch (Exception e) {
        println "❌ Erreur : ${job.fullName} — ${e.message}"
    }
}

println "\n🎉 Terminé !"

🔥 Maintenant ça va marcher sans erreur.


en cas de probleme 

import com.cloudbees.hudson.plugins.folder.Folder
import jenkins.model.Jenkins

Jenkins j = Jenkins.instance

String rootFolderName = "poste01"

// --- Fonction utilitaire : retourner ou créer un folder ---
Folder getOrCreateFolder(ItemGroup parent, String name) {
    def f = parent.getItem(name)
    if (f == null) {
        println "📁 Folder '${name}' introuvable, création..."
        f = new Folder(parent, name)
        parent.add(f, name)
    }
    return f
}

// --- Création du dossier principal ---
Folder root = getOrCreateFolder(j, rootFolderName)

println "📂 Dossier racine : ${root.fullName}"

// --- Fonction qui scanne récursivement tous les sous-dossiers ---
void scanFolder(Folder folder) {
    println "🔍 Scan dossier : ${folder.fullName}"

    folder.getItems().each { item ->
        if (item instanceof Folder) {
            println "➡️ Sous-folder trouvé : ${item.fullName}"
            scanFolder(item) // récursion
        } else {
            println "📄 Job trouvé : ${item.fullName}"
        }
    }
}

// --- Lancer le scan ---
scanFolder(root)

println "✔ Scan terminé."


une autre version à valider 

/**
 * Script Groovy — Regrouper les jobs provenant d’un dossier PROJETS-Formation-17112025
 * dans un dossier racine FORMATION-<SUFFIX>
 *
 * Modes :
 *   A → Regroupe par prefix utilisateur (poste09 → poste09/)
 *   B → Regroupe par type (TP, build, deploy)
 *   C → Regroupe tous les jobs ensemble dans un seul dossier
 *
 * Paramètres :
 */
def ROOT_FOLDER_NAME = "PROJETS-Formation-17112025"
def TARGET_SUFFIX    = "00"     // devient FORMATION-00
def GROUP_MODE       = "A"      // A, B ou C

println "📂 Scan du dossier : ${ROOT_FOLDER_NAME}"
println "🎯 Mode : ${GROUP_MODE}"
println "🎯 Folder cible (racine) : FORMATION-${TARGET_SUFFIX}"
println "───────────────────────────────────────────────"

// -----------------------------------------------------------
// 1. Récupération du dossier source contenant les jobs
// -----------------------------------------------------------
def jenkins = Jenkins.instance
def root = jenkins

def source = root.getItem(ROOT_FOLDER_NAME)
if (!source) {
    println "❌ Le dossier '${ROOT_FOLDER_NAME}' n'existe pas."
    return
}

println "📁 Dossier source trouvé : ${source.name}"

// -----------------------------------------------------------
// 2. Création du folder FORMATION-XX sous la racine
// -----------------------------------------------------------
def targetRootName = "FORMATION-${TARGET_SUFFIX}"
def targetRoot = root.getItem(targetRootName)

if (!targetRoot) {
    println "📁 Folder '${targetRootName}' introuvable, création…"
    targetRoot = new com.cloudbees.hudson.plugins.folder.Folder(root, targetRootName)
    root.add(targetRoot, targetRootName)
    targetRoot.save()
    root.save()
}

println "📁 Folder racine de destination : ${targetRootName}"
println "───────────────────────────────────────────────"


// Fonction utilitaire : trouver ou créer un folder dans un parent
def getOrCreateFolder(parent, name) {
    def f = parent.getItem(name)
    if (!f) {
        println "📁 Création du folder : ${name}"
        f = new com.cloudbees.hudson.plugins.folder.Folder(parent, name)
        parent.add(f, name)
        f.save()
    }
    return f
}

// Fonction : détecter type (pour mode B)
def detectType(jobName) {
    if (jobName =~ /(?i)TP/) return "TP"
    if (jobName =~ /(?i)deploy/) return "deploy"
    if (jobName =~ /(?i)build/) return "build"
    return "autre"
}

println "🔍 Recherche des jobs dans ${ROOT_FOLDER_NAME}"
println "───────────────────────────────────────────────"

def movedCount = 0

source.items.each { job ->

    println "\n→ Job trouvé : ${job.name}"

    // Déterminer cible selon mode
    def destinationFolder = null

    switch (GROUP_MODE.toUpperCase()) {

        case "A":
            // Mode A : prefix utilisateur → poste09-xxx → poste09
            def prefix = job.name.split("-")[0]
            destinationFolder = getOrCreateFolder(targetRoot, prefix)
            break

        case "B":
            // Mode B : regrouper par type (TP, deploy, build…)
            def t = detectType(job.name)
            destinationFolder = getOrCreateFolder(targetRoot, t)
            break

        case "C":
            // Mode C : tout dans un seul dossier
            destinationFolder = getOrCreateFolder(targetRoot, "ALL")
            break
    }

    println "🚚 Déplacement → ${destinationFolder.name}"

    try {
        hudson.model.Items.move(job, destinationFolder)
        movedCount++
    } catch (Exception e) {
        println "❌ Erreur lors du déplacement : ${e.message}"
    }
}

println "\n🎉 Terminé ! Jobs déplacés : ${movedCount}"
println "📦 Destination racine : ${targetRootName}"
