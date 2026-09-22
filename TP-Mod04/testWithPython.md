# Automatisation des tests avec Jenkins 🧪

## Contexte / Storytelling

La startup **GFP Tech** développe une petite application Python permettant de réaliser des calculs simples.

L'équipe souhaite maintenant intégrer les tests directement dans Jenkins afin de :

- vérifier automatiquement le code à chaque modification ;
- détecter rapidement les régressions ;
- publier les résultats des tests dans Jenkins ;
- mesurer les performances de l'application.

Dans ce TP, nous allons utiliser un projet Python volontairement simple afin de nous concentrer sur **Jenkins et l'automatisation des tests**.

---

## 🎯 Objectifs pédagogiques

À la fin du TP, vous serez capables de :

- exécuter des tests unitaires Python avec `pytest` ;
- intégrer `pytest` dans un pipeline Jenkins ;
- publier les résultats des tests dans Jenkins ;
- faire échouer automatiquement le pipeline lorsqu'un test échoue ;
- réaliser un test de performance simple avec **Locust** ;
- intégrer le test de performance dans Jenkins.

---

## 🧰 Prérequis

- Jenkins fonctionnel
- Git
- Python 3
- Projet Git
- Pipeline Jenkins
- Agent Jenkins capable d'exécuter Python

Vérifier :

```bash
python3 --version
git --version