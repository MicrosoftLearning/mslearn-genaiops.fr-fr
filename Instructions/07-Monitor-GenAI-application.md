---
lab:
  title: Surveillez votre application d'IA générative
  description: Découvrez comment surveiller les interactions avec votre modèle déployé et obtenir des Informations sur l’optimisation de son utilisation avec votre application IA générative.
---

# Surveillez votre application d'IA générative

Cet exercice prend environ **30** minutes.

> **Note** : cet exercice suppose une certaine connaissance d’Azure AI Foundry, c’est pourquoi certaines instructions sont intentionnellement moins détaillées pour encourager une exploration plus active et un apprentissage pratique.

## Introduction

Dans cet exercice, vous allez activer la surveillance d’une application de complétion de conversation et afficher ses performances dans Azure Monitor. Vous interagissez avec votre modèle déployé pour générer des données, afficher les données générées via le tableau de bord Insights pour les applications d’IA génératives et configurer des alertes pour vous aider à optimiser le déploiement du modèle.

## Configurer l’environnement

Pour effectuer les tâches de cet exercice, vous avez besoin des éléments suivants :

- Un projet Azure AI Foundry
- Un modèle déployé (comme GPT-4o)
- Une ressource Application Insights connectée

### Déployer un modèle dans un projet Azure AI Foundry

Pour configurer rapidement un projet Azure AI Foundry, des instructions simples pour utiliser l’interface utilisateur du portail Azure AI Foundry sont fournies ci-dessous.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Dans la page d’accueil, dans la section **Explorer les modèles et les fonctionnalités**, recherchez le modèle `gpt-4o` ; que nous utiliserons dans notre projet.
1. Dans les résultats de la recherche, sélectionnez le modèle **gpt-4o** pour afficher ses détails, puis en haut de la page du modèle, sélectionnez **Utiliser ce modèle**.
1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez **les options avancées**.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre projet :
    - **Ressource Azure AI Foundry** : *un nom valide pour votre ressource Azure AI Foundry.*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *sélectionnez n’importe quel emplacement pris en charge par les services d’IA***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet, y compris le déploiement du modèle gpt-4 que vous avez sélectionné, soit créé.
1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour afficher la page principale de votre projet.
1. Dans la zone **Points de terminaison et clés** , vérifiez que la bibliothèque **Azure AI Foundry** est sélectionnée et affichez le **point de terminaison du projet Azure AI Foundry**.
1. **Enregistrez** le point de terminaison dans un bloc-notes. Vous utiliserez ce point de terminaison pour connecter votre projet à une application cliente.

### Se connecter à Application Insights

Connectez Application Insights à votre projet dans Azure AI Foundry pour commencer à collecter des données pour la surveillance.

1. Utilisez le menu de gauche, puis sélectionnez la page **Suivi**.
1. **Créez** une ressource Application Insights pour vous connecter à votre application.
1. Entrez un nom de ressource Application Insights, puis sélectionnez **Créer**.

Application Insights est désormais connecté à votre projet et les données commencent à être collectées pour l’analyse.

## Interagir avec un modèle déployé

Vous allez interagir avec votre modèle déployé par programmation en configurant une connexion à votre projet Azure AI Foundry à l’aide d’Azure Cloud Shell. Cela vous permet d’envoyer une invite au modèle et de générer des données de surveillance.

### Se connecter à un modèle via Cloud Shell

Commencez par récupérer les informations nécessaires pour être authentifié afin d’interagir avec votre modèle. Ensuite, accédez à Azure Cloud Shell et mettez à jour la configuration pour envoyer les invites fournies à votre propre modèle déployé.

1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant).
1. Dans un nouvel onglet, accédez au [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.
1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique**.

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique de Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, entrez et exécutez les commandes suivantes :

    ```
    rm -r mslearn-genaiops -f
    git clone https://github.com/microsoftlearning/mslearn-genaiops mslearn-genaiops
    ```

    Cette commande clone le référentiel GitHub contenant les fichiers de code pour cet exercice.

    > **Conseil** : lorsque vous collez des commandes dans Cloud Shell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-genaiops/Files/07
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques dont vous avez besoin :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai azure-identity azure-ai-projects opentelemetry-instrumentation-openai-v2 azure-monitor-opentelemetry
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code :

    1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry).
    1. Remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement GPT-4o (par défaut `gpt-4o`).

1. *Une* fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faites un clic droit sur > Enregistrer** pour **enregistrer vos modifications**, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Envoyer des invites à votre modèle déployé

Vous allez maintenant exécuter plusieurs scripts qui envoient différentes invites à votre modèle déployé. Ces interactions génèrent des données que vous pouvez observer ultérieurement dans Azure Monitor.

1. Exécutez la commande suivante pour **afficher le premier script** fourni :

    ```
   code start-prompt.py
    ```

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.
1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python start-prompt.py
    ```

    Le modèle génèrera une réponse, qui sera capturée avec Application Insights pour une analyse plus approfondie. Nous allons varier nos invites pour explorer leurs effets.

1. **Ouvrez et passez en revue le script**, où l’invite demande au modèle de **ne répondre uniquement avec une seule phrase et une liste** :

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

## Afficher les données de surveillance dans Azure Monitor

Pour afficher les données collectées à partir de vos interactions de modèle, accédez au tableau de bord lié à un classeur dans Azure Monitor.

### Dans le portail Azure AI Foundry, accédez à Azure Monitor.

1. Accédez à l’onglet de votre navigateur avec le **portail Azure AI Foundry** ouvert.
1. Utilisez le menu sur la gauche, sélectionnez **Surveillance**.
1. Sélectionnez l’**utilisation des ressources** et passez en revue les données résumées des interactions avec votre modèle déployé.

> **Remarque** : Vous pouvez également sélectionner **Azure Monitor Metrics Explorer** en bas de la page Surveillance pour obtenir une vue complète de toutes les métriques disponibles. Le lien ouvre Azure Monitor dans un nouvel onglet.

## Interpréter les métriques de surveillance

Maintenant, il est temps d’explorer les données et de commencer à interpréter les informations qu’elles vous fournissent.

### Passer en revue l’utilisation des jetons

Concentrez-vous d’abord sur la section **Utilisation des jetons** et passez en revue les mesures suivantes :

- **Nombre total de requêtes** : nombre de demandes d’inférence distinctes, qui est le nombre de fois où le modèle a été appelé.

> Utile pour analyser le débit et comprendre le coût moyen par appel.

- **Nombre total de jetons** : total combiné des jetons d’invite et des jetons d’achèvement.

> C’est la mesure la plus importante pour la facturation et les performances, car elle détermine la latence et le coût.

- **Nombre de jetons d’invite** : nombre total de jetons utilisés dans l’entrée (les invites que vous avez envoyées) pour tous les appels de modèle.

> Considérez cela comme le *coût pour poser une question* au modèle.

- **Nombre de jetons d’achèvement** : nombre de jetons retournés en sortie par le modèle, essentiellement la longueur des réponses.

> Les jetons d’achèvement générés représentent souvent la majeure partie de l’utilisation et du coût des jetons, en particulier pour les réponses longues ou détaillées.

### Comparer les invites individuelles

1. Dans le menu de gauche, sélectionnez **Suivi**. Développez chaque étendue d’IA de génération **generate_completion** pour voir leurs étendues enfants. Chaque invite est représentée sous la forme d’une nouvelle ligne de données. Passez en revue et comparez le contenu des colonnes suivantes :

- **Entrée** : affiche le message utilisateur envoyé au modèle.

> Utilisez cette colonne pour évaluer quelles formulations d’invite sont efficaces ou problématiques.

- **Sortie** : contient la réponse du modèle.

> Utilisez cette colonne pour évaluer l’éloquence, la pertinence et la cohérence. En particulier en ce qui concerne les nombres de jetons et la durée.

- **Durée** : indique le délai de réponse du modèle, en millisecondes.

> Comparez les lignes pour explorer les modèles d’invite qui entraînent des temps de traitement plus longs.

- **Réussite** : indique si un appel de modèle a réussi ou échoué.

> Utilisez cette colonne pour identifier les invites problématiques ou les erreurs de configuration. La dernière invite a probablement échoué car l’invite était trop longue.

## (FACULTATIF) Créer une alerte

Si vous avez encore du temps, essayez de configurer une alerte pour vous avertir lorsque la latence du modèle dépasse un certain seuil. Il s’agit d’un exercice conçu pour vous mettre au défi, ce qui signifie que les instructions sont intentionnellement moins détaillées.

- Dans Azure Monitor, créez une **règle d’alerte** pour votre projet et modèle Azure AI Foundry.
- Choisissez une mesure telle que **Durée de la requête (ms)** et définissez un seuil (par exemple, supérieur à 4 000 ms).
- Créez un **groupe d’actions** pour définir la façon dont vous serez averti.

Les alertes vous aident à préparer la production en établissant une surveillance proactive. Les alertes que vous configurez dépendent des priorités de votre projet et de la façon dont votre équipe a décidé de mesurer et d’atténuer les risques.

## Où trouver d’autres labos

Vous pouvez explorer d’autres labos et exercices dans le [portail d’apprentissage d’Azure AI Foundry](https://ai.azure.com) ou consulter la **section de labo** du cours pour obtenir d’autres activités.
