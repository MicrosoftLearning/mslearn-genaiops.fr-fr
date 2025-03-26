---
lab:
  title: Explorer l’ingénierie des invites avec Prompty
---

## Explorer l’ingénierie des invites avec Prompty

Pendant l’idéation, vous souhaitez tester et améliorer rapidement les différentes invites avec votre modèle de langage. Il existe différentes façons d’aborder l’ingénierie d’invite : via le terrain de jeu dans le portail Azure AI Foundry ou en utilisant Prompty pour une approche plus orientée code.

Dans cet exercice, vous allez explorer l’ingénierie d’invite avec Prompty dans Visual Studio Code, à l’aide d’un modèle déployé via Azure AI Foundry.

Cet exercice prend environ **40** minutes.

## Scénario

Imaginez que vous souhaitez créer une application pour aider les étudiants à apprendre à coder en Python. Dans l’application, vous souhaitez un tuteur automatisé qui peut aider les étudiants à écrire et évaluer du code. Toutefois, vous ne souhaitez pas que l’application de conversation fournisse toutes les réponses. Vous voulez que les étudiants reçoivent des conseils personnalisés qui les encouragent à réfléchir à la façon de procéder.

Vous avez sélectionné un modèle GPT-4 pour commencer à expérimenter. Vous souhaitez maintenant appliquer l’ingénierie d’invite pour guider le comportement de la conversation afin qu’elle devienne un tuteur qui génère des conseils personnalisés.

Commençons par déployer les ressources nécessaires pour utiliser ce modèle dans le portail Azure AI Foundry.

## Créer un projet et un hub Azure AI

> **Note** : si vous disposez déjà d’un hub et d’un projet Azure AI, vous pouvez ignorer cette procédure et utiliser votre projet existant.

Vous pouvez créer un hub Azure AI et un projet manuellement via le portail Azure AI Foundry, ainsi que déployer le modèle utilisé dans l’exercice. Toutefois, vous pouvez également automatiser ce processus via l’utilisation d’une application modèle avec [Azure Developer CLI (azd).](https://aka.ms/azd)

1. Dans un navigateur web, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

1. Cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel de cet exercice :

     ```powershell
    rm -r mslearn-genaiops -f
    git clone https://github.com/MicrosoftLearning/mslearn-genaiops
     ```

1. Une fois le référentiel cloné, entrez les commandes suivantes pour initialiser le modèle de démarrage. 
   
     ```powershell
    cd ./mslearn-genaiops/Starter
    azd init
     ```

1. Une fois invité, donnez au nouvel environnement un nom, car il sera utilisé comme base pour donner des noms uniques à toutes les ressources approvisionnées.
        
1. Ensuite, entrez la commande suivante pour exécuter le modèle de démarrage. Il approvisionne un hub IA avec des ressources dépendantes, un projet IA, des services IA et un point de terminaison en ligne.

     ```powershell
    azd up
     ```

1. Lorsque vous y êtes invité, choisissez l’abonnement que vous souhaitez utiliser, puis choisissez l’un des emplacements suivants pour l’approvisionnement des ressources :
   - USA Est
   - USA Est 2
   - Centre-Nord des États-Unis
   - États-Unis - partie centrale méridionale
   - Suède Centre
   - USA Ouest
   - USA Ouest 3
    
1. Attendez que le script se termine. Cela prend généralement environ 10 minutes, mais dans certains cas, cela peut prendre plus de temps.

    > **Note** : les ressources Azure OpenAI sont limitées au niveau du locataire par des quotas régionaux. Les régions répertoriées ci-dessus incluent le quota par défaut pour les types de modèle utilisés dans cet exercice. Le choix aléatoire d’une région réduit le risque qu’une seule région atteigne sa limite de quota. Si une limite de quota est atteinte, vous devrez peut-être créer un autre groupe de ressources dans une autre région. En savoir plus sur la [disponibilité du modèle par région](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=standard%2Cstandard-chat-completions#global-standard-model-availability)

    <details>
      <summary><b>Conseil de résolution des problèmes</b> : aucun quota disponible dans une région donnée</summary>
        <p>Si vous recevez une erreur de déploiement pour l’un des modèles en raison d’une indisponibilité de quota dans la région choisie, essayez d’exécuter les commandes suivantes :</p>
        <ul>
          <pre><code>azd env set AZURE_ENV_NAME new_env_name
   azd env set AZURE_RESOURCE_GROUP new_rg_name
   azd env set AZURE_LOCATION new_location
   azd up</code></pre>
        Remplacement de <code>new_env_name</code>, <code>new_rg_name</code> et <code>new_location</code> par les nouvelles valeurs. Le nouvel emplacement doit être l’une des régions répertoriées au début de l’exercice, par exemple <code>eastus2</code>, <code>northcentralus</code>, etc.
        </ul>
    </details>

1. Une fois que toutes les ressources ont été approvisionnées, utilisez les commandes suivantes pour récupérer le point de terminaison et la clé d’accès à votre ressource AI Services. Notez que vous devez remplacer `<rg-env_name>` et `<aoai-xxxxxxxxxx>` par les noms de votre groupe de ressources et de votre ressource AI Services. Les deux sont imprimés dans la sortie du déploiement.

     ```powershell
    Get-AzCognitiveServicesAccount -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property endpoint
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiez ces valeurs, car elles seront utilisées ultérieurement.
   
## Configurer votre environnement de développement local

Pour expérimenter et itérer rapidement, vous allez utiliser Prompty dans Visual Studio (VS) Code. Préparons VS Code pour l’idéation locale.

1. Ouvrez VS Code et **clonez** le référentiel Git suivant : [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git).
1. Stockez le clone sur un lecteur local et ouvrez le dossier après le clonage.
1. Dans le volet des extensions de VS Code, recherchez et installez l’extension **Prompty**.
1. Dans l’Explorateur VS Code (volet gauche), cliquez avec le bouton droit sur le dossier **Files/03**.
1. Sélectionnez **Nouveau Prompty** dans le menu déroulant.
1. Ouvrez le fichier nouvellement créé nommé **basic.prompty**.
1. Exécutez le fichier Prompty en sélectionnant le bouton de **lecture** en haut à droite (ou appuyez sur F5).
1. Lorsque vous êtes invité à vous connecter, sélectionnez **Autoriser**.
1. Sélectionnez votre compte Azure et connectez-vous.
1. Revenez sur VS Code. Un volet **Sortie** s’ouvrira avec un message d’erreur. Le message d’erreur doit vous indiquer que le modèle déployé n’est pas spécifié ou est introuvable.

Pour corriger l’erreur, vous devez configurer un modèle que Prompty doit utiliser.

## Mettre à jour les métadonnées d’invite

Pour exécuter le fichier Prompty, vous devez spécifier le modèle de langage à utiliser pour générer la réponse. Les métadonnées sont définies dans le *frontmatter* du fichier Prompty. Nous allons mettre à jour les métadonnées avec la configuration du modèle et d’autres informations.

1. Ouvrez le volet Terminal de Visual Studio Code.
1. Copiez le fichier **basic.prompty** (dans le même dossier) et renommez la copie en `chat-1.prompty`.
1. Ouvrez **chat-1.prompty** et mettez à jour les champs suivants pour modifier certaines informations de base :

    - **Nom :**

        ```yaml
        name: Python Tutor Prompt
        ```

    - **Description** :

        ```yaml
        description: A teaching assistant for students wanting to learn how to write and edit Python code.
        ```

    - **Modèle déployé** :

        ```yaml
        azure_deployment: ${env:AZURE_OPENAI_CHAT_DEPLOYMENT}
        ```

1. Ensuite, ajoutez l’espace réservé suivant pour la clé API sous le paramètre **azure_deployment**.

    - **Clé de point de terminaison** :

        ```yaml
        api_key: ${env:AZURE_OPENAI_API_KEY}
        ```

1. Enregistrez le fichier Prompty mis à jour.

Le fichier Prompty possède désormais tous les paramètres nécessaires, mais certains paramètres utilisent des espaces réservés pour obtenir les valeurs requises. Les espaces réservés sont stockés dans le fichier **.env** dans le même dossier.

## Mettre à jour la configuration du modèle

Pour spécifier le modèle utilisé par Prompty, vous devez fournir les informations de votre modèle dans le fichier .env.

1. Ouvrez le fichier **.env** dans le dossier **Files/03**.
1. Mettez à jour chacun des espaces réservés avec les valeurs que vous avez copiées précédemment à partir de la sortie du déploiement de modèle dans le portail Azure :

    ```yaml
    - AZURE_OPENAI_CHAT_DEPLOYMENT="gpt-4"
    - AZURE_OPENAI_ENDPOINT="<Your endpoint target URI>"
    - AZURE_OPENAI_API_KEY="<Your endpoint key>"
    ```

1. Enregistrez le fichier .env.
1. Réexécutez le fichier **chat-1.prompty**.

Vous devez maintenant obtenir une réponse générée par l’IA, même si elle n’est pas liée à votre scénario, car elle utilise simplement l’exemple d’entrée. Nous allons mettre à jour le modèle pour en faire un assistant d’enseignement IA.

## Modifier l’exemple de section

L’exemple de section spécifie les entrées à Prompty et fournit les valeurs par défaut à utiliser si aucune entrée n’est fournie.

1. Modifiez les champs des paramètres suivants :

    - **firstName** : choisissez un autre nom.
    - **context** : supprimez l’intégralité de cette section.
    - **question** : remplacez le texte fourni par :

    ```yaml
    What is the difference between 'for' loops and 'while' loops?
    ```

    Votre **exemple de section** doit maintenant se présenter comme ceci :
    
    ```yaml
    sample:
    firstName: Daniel
    question: What is the difference between 'for' loops and 'while' loops?
    ```

    1. Exécutez le fichier Prompty mis à jour et passez en revue la sortie.

