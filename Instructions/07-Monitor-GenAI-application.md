---
lab:
  title: Surveillez votre application d'IA générative
---

# Surveillez votre application d'IA générative

Cet exercice prend environ **30** minutes.

> **Note** : cet exercice suppose une certaine connaissance d’Azure AI Foundry, c’est pourquoi certaines instructions sont intentionnellement moins détaillées pour encourager une exploration plus active et un apprentissage pratique.

## Introduction

Dans cet exercice, vous allez activer la surveillance d’une application de complétion de conversation et afficher ses performances dans Azure Monitor. Vous interagissez avec votre modèle déployé pour générer des données, afficher les données générées via le tableau de bord Insights pour les applications d’IA génératives et configurer des alertes pour vous aider à optimiser le déploiement du modèle.

## 1. Configurez l’environnement.

Pour effectuer les tâches de cet exercice, vous avez besoin des éléments suivants :

- Hub Azure AI Foundry
- Un projet Azure AI Foundry
- Un modèle déployé (comme GPT-4o)
- Une ressource Application Insights connectée

### R : Créer un projet et un hub AI Foundry

Pour configurer rapidement un hub et un projet, des instructions simples pour utiliser l’IU du portail Azure AI Foundry sont fournies ci-dessous.

1. Accédez au portail Azure AI Foundry : ouvrez [https://ai.azure.com](https://ai.azure.com).
1. Connectez-vous à l'aide de vos informations d'identification Azure.
1. Créez un projet :
    1. Accédez à **Tous les hubs + projets**.
    1. Sélectionnez **+ Nouveau projet**.
    1. Entrez un **nom de projet**.
    1. Lorsque vous y êtes invité, **créez un hub**.
    1. Personnalisez le hub :

        1. Sélectionnez un **abonnement**, un **groupe de ressources**, un **emplacement**, etc.
        1. Connectez une **nouvelle ressource Azure AI Services** (ignorez la recherche IA).

    1. Passez en revue les informations, puis sélectionnez **Créer**.

1. **Attendez la fin du déploiement** (environ 1 ou 2 minutes).

### B. Déployer un modèle

Pour générer des données que vous pouvez surveiller, vous devez d’abord déployer un modèle et interagir avec celui-ci. Dans les instructions, vous êtes invité à déployer un modèle GPT-4o, mais **vous pouvez utiliser n’importe quel modèle** à partir de la collection Azure OpenAI Service disponible.

1. Dans le menu de gauche, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
1. Déployez un **modèle de base** et choisissez **gpt-4o**.
1. **Personnalisez les détails du déploiement**.
1. Définissez la **capacité** sur **5 000 jetons par minute (TPM).**

Le hub et le projet sont prêts, avec toutes les ressources Azure requises provisionnées automatiquement.

### C. Se connecter à Application Insights

Connectez Application Insights à votre projet dans Azure AI Foundry pour commencer à collecter des données pour la surveillance.

1. Ouvrez votre projet dans le portail Azure AI Foundry.
1. Utilisez le menu de gauche, puis sélectionnez la page **Suivi**.
1. **Créez une** ressource Application Insights pour vous connecter à votre application.
1. Entrez le **nom d’une ressource Application Insights**.

Application Insights est désormais connecté à votre projet et les données commencent à être collectées pour l’analyse.

## 2. Interagissez avec un modèle déployé.

Vous allez interagir avec votre modèle déployé par programmation en configurant une connexion à votre projet Azure AI Foundry à l’aide d’Azure Cloud Shell. Cela vous permet d’envoyer une invite au modèle et de générer des données de surveillance.

### R : Se connecter à un modèle via Cloud Shell

Commencez par récupérer les informations nécessaires pour être authentifié afin d’interagir avec votre modèle. Ensuite, vous accédez à Azure Cloud Shell et mettez à jour la configuration pour envoyer les invites fournies à votre propre modèle déployé.

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet**.
1. **Enregistrez** la chaîne dans un bloc-notes. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.
1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant).
1. Dans un nouvel onglet, accédez au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.
1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique**.

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, entrez et exécutez les commandes suivantes :

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Cette commande clone le référentiel GitHub contenant les fichiers de code pour cet exercice.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques dont vous avez besoin :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference azure-monitor-opentelemetry
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code :

    1. Remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry).
    1. Remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement gpt-4o (par défaut `gpt-4o`).

1. *Une fois* que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **Ctrl+S** ou **Clic droit > Enregistrer** pour **enregistrer vos modifications**.

### B. Envoyer des invites à votre modèle déployé

Vous allez maintenant exécuter plusieurs scripts qui envoient différentes invites à votre modèle déployé. Ces interactions génèrent des données que vous pouvez observer ultérieurement dans Azure Monitor.

1. Exécutez la commande suivante pour **afficher le premier script** fourni :

    ```
   code start-prompt.py
    ```

1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour **exécuter le script** :

    ```
   python start-prompt.py
    ```

    Le modèle génèrera une réponse, qui sera capturée avec Application Insights pour une analyse plus approfondie. Nous allons varier nos invites pour explorer leurs effets.

1. **Ouvrez et passez en revue le script**, où l’invite demande au modèle de **répondre uniquement avec une seule phrase et une liste** :

    ```
   code short-prompt.py
    ```

1. Dans la ligne de commande, **exécutez le script** en entrant la commande suivante :

    ```
   python short-prompt.py
    ```

1. Le script suivant a un objectif similaire, mais inclut les instructions pour la sortie dans le **message système** au lieu du message utilisateur :

    ```
   code system-prompt.py
    ```

1. Dans la ligne de commande, **exécutez le script** en entrant la commande suivante :

    ```
   python system-prompt.py
    ```

1. Enfin, essayons de déclencher une erreur en exécutant une invite avec **trop de jetons** :

    ```
   code error-prompt.py
    ```

1. Dans la ligne de commande, **exécutez le script** en entrant la commande suivante : Notez qu’il est très **vraisemblable qu’une erreur se produise**.

    ```
   python error-prompt.py
    ```

Maintenant que vous avez interagi avec le modèle, vous pouvez passer en revue les données dans Azure Monitor.

> **Note** : l’affichage des données de surveillance dans Azure Monitor peut prendre quelques minutes.

## 4. Affichez les données de surveillance dans Azure Monitor

Pour afficher les données collectées à partir de vos interactions de modèle, vous accédez au tableau de bord qui lie à un classeur dans Azure Monitor.

### R : Dans le portail Azure AI Foundry, accédez à Azure Monitor.

1. Accédez à l’onglet de votre navigateur avec le **portail Azure AI Foundry** ouvert.
1. Utilisez le menu de gauche, sélectionnez **Suivi**.
1. Sélectionnez un lien en haut, qui indique **Consulter votre tableau de bord Insights pour les applications d’IA générative.** Le lien ouvre Azure Monitor dans un nouvel onglet.
1. Passez en revue la **vue d’ensemble** fournissant des données résumées des interactions avec votre modèle déployé.

## 5. Interprétez les mesures de surveillance dans Azure Monitor

Maintenant, il est temps d’explorer les données et de commencer à interpréter ce qu’elles vous disent.

### R : Passer en revue l’utilisation des jetons

Concentrez-vous d’abord sur la section **Utilisation des jetons** et passez en revue les mesures suivantes :

- **Jetons d’invite** : nombre total de jetons utilisés dans l’entrée (les invites que vous avez envoyées) sur tous les appels de modèle.

> Considérez cela comme le *coût de poser* une question au modèle.

- **Jetons d’achèvement** : nombre de jetons retournés comme sortie par le modèle, essentiellement la longueur des réponses.

> Les jetons d’achèvement générés représentent souvent la majeure partie de l’utilisation et du coût des jetons, en particulier pour les réponses longues ou détaillées.

- **Nombre total de jetons** : total combiné des jetons d’invite totaux et des jetons d’achèvement.

> Mesure la plus importante pour la facturation et les performances, car elle influence la latence et le coût.

- **Nombre total d’appels** : nombre de demandes d’inférence distinctes, qui est le nombre de fois où le modèle a été appelé.

> Utile pour analyser le débit et comprendre le coût moyen par appel.

### B. Comparer les invites individuelles

Faites défiler vers le bas pour rechercher les **étendues d’IA générative**, qui sont visualisées sous la forme d’une table où chaque invite est représentée sous la forme d’une nouvelle ligne de données. Passez en revue et comparez le contenu des colonnes suivantes :

- **Statut** : indique si un appel de modèle a réussi ou échoué.

> Utilisez-le pour identifier les invites problématiques ou les erreurs de configuration. La dernière invite a probablement échoué, car l’invite était trop longue.

- **Durée** : indique la durée de réponse du modèle, en millisecondes.

> Comparez les lignes pour explorer les modèles d’invite qui entraînent des temps de traitement plus longs.

- **Entrée** : affiche le message utilisateur envoyé au modèle.

> Utilisez cette colonne pour évaluer quelles formulations d’invite sont efficaces ou problématiques.

- **Système** : affiche le message système utilisé dans l’invite (le cas échéant).

> Comparez les entrées pour évaluer l’impact de l’utilisation ou de la modification des messages système.

- **Sortie** : contient la réponse du modèle.

> Utilisez-la pour évaluer la verbosité, la pertinence et la cohérence. En particulier en ce qui concerne les nombres de jetons et la durée.

## 6. (FACULTATIF) Créez une alerte

Si vous avez encore du temps, essayez de configurer une alerte pour vous avertir lorsque la latence du modèle dépasse un certain seuil. Il s’agit d’un exercice conçu pour vous mettre au défi, ce qui signifie que les instructions sont intentionnellement moins détaillées.

- Dans Azure Monitor, créez une **règle d’alerte **pour votre projet et modèle Azure AI Foundry.
- Choisissez une mesure telle que **Durée de la requête (ms)** et définissez un seuil (par exemple, supérieur à 4 000 ms).
- Créez un **groupe d’actions **pour définir la façon dont vous serez averti.

Les alertes vous aident à préparer la production en établissant une surveillance proactive. Les alertes que vous configurez dépendent des priorités de votre projet et de la façon dont votre équipe a décidé de mesurer et d’atténuer les risques.

## Où trouver d’autres labos

Vous pouvez explorer d’autres labos et exercices dans le [portail Learning d’Azure AI Foundry](https://ai.azure.com) ou consulter la **section de labo** du cours pour obtenir d’autres activités disponibles.
