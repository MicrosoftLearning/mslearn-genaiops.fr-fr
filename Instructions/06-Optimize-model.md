---
lab:
  title: Optimiser votre modèle à l’aide d’un jeu de données synthétique
  description: Découvrez comment créer des jeux de données synthétiques et les utiliser pour améliorer les performances et la fiabilité de votre modèle.
---

## Optimiser votre modèle à l’aide d’un jeu de données synthétique

L’optimisation d’une application d’IA générative implique l’utilisation de jeux de données pour améliorer les performances et la fiabilité du modèle. En utilisant des données synthétiques, les développeurs peuvent simuler un large éventail de scénarios et de cas complexes qui peuvent ne pas être présents dans des données réelles. En outre, l’évaluation des sorties du modèle est cruciale pour obtenir des applications IA de haute qualité et fiables. L’ensemble du processus d’optimisation et d’évaluation peut être géré efficacement à l’aide du Kit de développement logiciel (SDK) Azure AI Evaluation, qui fournit des outils et des infrastructures robustes pour simplifier ces tâches.

Cet exercice prendra environ **30** minutes\*

> \* Cette durée estimée n’inclut pas la tâche facultative à la fin de l’exercice.
## Scénario

Imaginez que vous souhaitez créer une application de guide intelligente basée sur l’IA pour améliorer les expériences des visiteurs dans un musée. L’application vise à répondre à des questions sur les figures historiques. Pour évaluer les réponses de l’application, vous devez créer un jeu de données synthétiques complet de questions-réponses couvrant différents aspects de ces personnalités et de leur œuvre.

Vous avez sélectionné un modèle GPT-4 pour fournir des réponses génératives. Vous souhaitez maintenant mettre en place un simulateur qui génère des interactions contextuellement pertinentes, en évaluant les performances de l’IA dans différents scénarios.

Commençons par déployer les ressources nécessaires pour créer cette application.

## Créer un projet et un hub Azure AI

Vous pouvez créer un hub Azure AI et un projet manuellement via le portail Azure AI Foundry, ainsi que déployer les modèles utilisés dans l’exercice. Toutefois, vous pouvez également automatiser ce processus via l’utilisation d’une application modèle avec [Azure Developer CLI (azd).](https://aka.ms/azd)

1. Dans un navigateur web, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

1. Cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure. Pour plus d’informations sur l’utilisation d’Azure Cloud Shell, consultez la [documentation Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique**.

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique de Cloud Shell avant de continuer.</font>**

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
        
1. Ensuite, entrez la commande suivante pour exécuter le modèle de démarrage. Il approvisionne un hub IA avec des ressources dépendantes, un projet IA, des services IA et un point de terminaison en ligne. Il déploiera également les modèles GPT-4 Turbo, GPT-4o et GPT-4o mini.

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
     ```

     ```powershell
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiez ces valeurs, car elles seront utilisées ultérieurement.

## Configurer votre environnement de développement dans Cloud Shell

Pour expérimenter et itérer rapidement, vous utiliserez un ensemble de scripts Python dans Cloud Shell.

1. Dans le volet de ligne de commande Cloud Shell, entrez la commande suivante pour accéder au dossier contenant des fichiers de code utilisés dans cet exercice :

     ```powershell
    cd ~/mslearn-genaiops/Files/06/
     ```

1. Entrez les commandes suivantes pour activer un environnement virtuel et installer les bibliothèques dont vous avez besoin :

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-evaluation azure-ai-projects promptflow wikipedia aiohttp openai==1.77.0
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```powershell
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **your_azure_openai_service_endpoint** et **your_azure_openai_service_api_key** par les valeurs de point de terminaison et de clé que vous avez copiées précédemment.
1. *Une* fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faites un clic droit sur > Enregistrer** pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Générer des données synthétiques

Vous allez à présent exécuter un script qui génère un jeu de données synthétiques et l’utilise pour évaluer la qualité de votre modèle pré-entraîné.

1. Exécutez la commande suivante pour **modifier le script** fourni :

    ```powershell
   code generate_synth_data.py
    ```

1. Dans le script, recherchez la **fonction de rappel # Définir**.
1. Sous ce commentaire, collez le code suivant :

    ```
    async def callback(
        messages: List[Dict],
        stream: bool = False,
        session_state: Any = None,  # noqa: ANN401
        context: Optional[Dict[str, Any]] = None,
    ) -> dict:
        messages_list = messages["messages"]
        # Get the last message
        latest_message = messages_list[-1]
        query = latest_message["content"]
        context = text
        # Call your endpoint or AI application here
        current_dir = os.getcwd()
        prompty_path = os.path.join(current_dir, "application.prompty")
        _flow = load_flow(source=prompty_path)
        response = _flow(query=query, context=context, conversation_history=messages_list)
        # Format the response to follow the OpenAI chat protocol
        formatted_response = {
            "content": response,
            "role": "assistant",
            "context": context,
        }
        messages["messages"].append(formatted_response)
        return {
            "messages": messages["messages"],
            "stream": stream,
            "session_state": session_state,
            "context": context
        }
    ```

    Vous pouvez faire correspondre n’importe quel point de terminaison d’application en précisant une fonction de rappel cible. vous utiliserez une application qui est un LLM avec un fichier Prompty `application.prompty`. La fonction de rappel ci-dessus traite chaque message généré par le simulateur en effectuant les tâches suivantes :
    * Récupère le dernier message de l’utilisateur.
    * Charge un flux de requête à partir d’application.prompty.
    * Génère une réponse en utilisant le flux d’invite.
    * Met en forme la réponse pour respecter le protocole de conversation OpenAI.
    * Ajoute la réponse de l’assistant à la liste de messages.

    >**Remarque** : Pour plus d’informations sur l’utilisation de Prompty, consultez [Documentation de Prompty](https://www.prompty.ai/docs).

1. Ensuite, recherchez **# Exécutez le simulateur**.
1. Sous ce commentaire, collez le code suivant :

    ```
    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
    }
    
    simulator = Simulator(model_config=model_config)
    
    outputs = asyncio.run(simulator(
        target=callback,
        text=text,
        num_queries=1,  # Minimal number of queries
    ))
    
    output_file = "simulation_output.jsonl"
    with open(output_file, "w") as file:
        for output in outputs:
            file.write(output.to_eval_qr_json_lines())
    ```

   Le code ci-dessus initialise le simulateur et l’exécute pour générer des conversations synthétiques basées sur un texte précédemment extrait de Wikipédia.

1. Ensuite, recherchez **# Évaluer le modèle**.
1. Sous ce commentaire, collez le code suivant :

    ```
    groundedness_evaluator = GroundednessEvaluator(model_config=model_config)
    eval_output = evaluate(
        data=output_file,
        evaluators={
            "groundedness": groundedness_evaluator
        },
        output_path="groundedness_eval_output.json"
    )
    ```

    Maintenant que vous disposez d’un jeu de données, vous pouvez évaluer la qualité et l’efficacité de votre application IA générative. Dans le code ci-dessus, vous allez utiliser le degré d’ancrage comme mesure de qualité.

1. Enregistrez les changements apportés.
1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour **exécuter le script** :

    ```
   python generate_synth_data.py
    ```

    Une fois le script terminé, vous pouvez télécharger les fichiers de sortie en exécutant `download simulation_output.jsonl` et `download groundedness_eval_output.json`, puis examinez leur contenu. Si la mesure d’ancrage n’est pas proche de la version 3.0, vous pouvez modifier les paramètres LLM comme `temperature`, `top_p` `presence_penalty` ou `frequency_penalty` dans le fichier `application.prompty` et réexécuter le script pour générer un nouveau jeu de données à des fins d’évaluation. Vous pouvez également modifier le `wiki_search_term` pour obtenir un jeu de données synthétiques basé sur un contexte différent.

## (FACULTATIF) Réglez votre modèle

Si vous disposez de temps supplémentaire, vous pouvez utiliser le jeu de données généré pour régler votre modèle dans Azure AI Foundry. L’ajustement dépend des ressources de l’infrastructure cloud, celles-ci pouvant nécessiter un certain temps d’approvisionnement en fonction de la capacité du centre de données et de la demande simultanée.

1. Ouvrez un nouvel onglet de navigateur, accédez au [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com`, puis connectez-vous en utilisant vos informations d’identification Azure.
1. Dans la page d’accueil AI Foundry, sélectionnez le projet que vous avez créé au début de l’exercice.
1. Accédez à la page **Ajustement** dans la section **Créer et personnaliser**, à l’aide du menu de gauche.
1. Sélectionnez le bouton permettant d’ajouter un nouveau modèle ajusté, le modèle **gpt-4o**, puis **Suivant**.
1. **Ajustez** le modèle à l’aide de la configuration suivante :
    - **Version du modèle** : *Sélectionnez la version par défaut*
    - **Méthode de personnalisation** : supervisée
    - **Suffixe de modèle** : `ft-travel`
    - **Ressource IA connectée** : *sélectionnez la connexion qui a été créée lors de la création de votre hub. Elle devrait être sélectionnée par défaut.*
    - **Données d’apprentissage** : charger des fichiers

    <details>  
    <summary><b>Conseil de résolution des problèmes</b> : erreur d’autorisations</summary>
    <p>Si vous recevez une erreur d’autorisation, essayez ce qui suit pour résoudre le problème :</p>
    <ul>
        <li>Dans le Portail Azure, sélectionnez la ressource AI Services.</li>
        <li>Dans Gestion des ressources, dans l’onglet Identité, confirmez qu’il s’agit d’une identité managée attribuée par le système.</li>
        <li>Accédez au compte de stockage associé. Sur la page IAM, ajoutez l’attribution de rôle <em>Propriétaire de données de stockage Blob</em>.</li>
        <li>Sous <strong>Attribuer l’accès à</strong>, choisissez <strong>Identité managée</strong>, <strong>+Sélectionner des membres</strong>, <strong>Toutes les identités managées attribuées par le système</strong>, puis sélectionnez votre ressource Azure AI Services.</li>
        <li>À l’aide de Passer en revue et attribuer, enregistrez les nouveaux paramètres et procédez à nouveau à l’étape précédente.</li>
    </ul>
    </details>

    - **Charger fichier** : sélectionnez le fichier JSONL que vous avez téléchargé lors d’une étape précédente.
    - **Données de validation** : aucune
    - **Paramètres de tâche** : *conserver les paramètres par défaut*
1. L’ajustement commence et peut prendre un certain temps.

    > **Remarque** : l’ajustement et le déploiement peuvent prendre un certain temps (30 minutes ou plus). Vous devrez donc peut-être vérifier l’avancement régulièrement. Vous pouvez consulter plus de détails sur l’avancement en sélectionnant la tâche d’ajustement du modèle et en affichant son onglet **Journaux**.

## (FACULTATIF) Déployer le modèle affiné

Une fois l’ajustement terminé, vous pouvez déployer le modèle ajusté.

1. Sélectionnez le lien de la tâche d’ajustement pour ouvrir sa page des détails. Puis, sélectionnez l’onglet **Mesures** et explorez les mesures d’ajustement.
1. Déployez le modèle ajusté avec les paramètres suivants :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : Standard
    - **Limite de débit en jetons par minute (en milliers)** : 5 K * (ou le maximum disponible dans votre abonnement si inférieur à 5 K)
    - **Filtre de contenu** : valeur par défaut
1. Attendez que le déploiement soit terminé avant de le tester. Cela peut prendre un certain temps. Vérifiez **l’état d’approvisionnement** jusqu’à ce qu’il soit terminé (vous devrez peut-être actualiser le navigateur pour afficher l’état mis à jour).
1. Une fois le déploiement terminé, accédez au modèle ajusté et sélectionnez **Ouvrir dans le terrain de jeu**.

    Maintenant que vous avez déployé votre modèle affiné, vous pouvez le tester sur le terrain de jeu de conversation comme vous le feriez avec n’importe quel modèle de base.

## Conclusion

Dans cet exercice, vous avez créé un jeu de données synthétique qui simule une conversation entre un utilisateur et une application de saisie semi-automatique de conversation. À l’aide de ce jeu de données, vous pouvez évaluer la qualité des réponses de votre application et le régler pour obtenir les résultats souhaités.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
