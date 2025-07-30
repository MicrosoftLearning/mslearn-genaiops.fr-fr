---
title: Exercices GenAIOps
permalink: index.html
layout: home
---

# Opérationnaliser les applications IA génératives

Les exercices de démarrage rapide suivants sont conçus pour vous offrir une expérience d’apprentissage pratique qui vous permettra d’explorer les tâches courantes requises pour opérationnaliser une charge de travail d’IA générative sur Microsoft Azure.

> **Note** : pour effectuer les exercices, vous aurez besoin d’un abonnement Azure et de disposer des autorisations et du quota suffisants pour approvisionner les ressources Azure et les modèles d’IA génératives nécessaires. Si vous n’avez pas de compte Azure, vous pouvez en créer un [ici](https://azure.microsoft.com/free). Les nouveaux utilisateurs peuvent profiter d’un essai gratuit qui inclut des crédits pour les 30 premiers jours.

## Exercices de démarrage rapide

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Note** : bien que vous puissiez effectuer ces exercices de façon indépendante, ils sont conçus pour accompagner des modules [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/), dans lesquels vous trouverez une présentation plus approfondie de certains des concepts sous-jacents sur lesquels se basent ces exercices.
