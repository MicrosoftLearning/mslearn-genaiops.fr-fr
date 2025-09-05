---
lab:
  title: Orchestrer un système RAG
  description: Découvrez comment mettre en œuvre des systèmes de génération augmentée par la récupération (RAG) dans vos applications afin d’améliorer l’exactitude et la pertinence des réponses générées.
---

## Orchestrer un système RAG

Les systèmes de génération augmentée de récupération (RAG) combinent la puissance des grands modèles de langage avec des mécanismes de récupération efficaces pour améliorer la précision et la pertinence des réponses générées. En tirant parti de LangChain pour l’orchestration et Azure AI Foundry pour les fonctionnalités d’IA, nous pouvons créer un pipeline robuste qui récupère des informations pertinentes à partir d’un jeu de données et génère des réponses cohérentes. Dans cet exercice, vous allez suivre les étapes de configuration de votre environnement, de prétraitement des données, de création d’incorporations et de création d’un index, vous permettant ainsi d’implémenter efficacement un système RAG.

Cet exercice prend environ **30** minutes.

## Scénario

Supposons que vous souhaitiez créer une application qui formule des recommandations sur les hôtels à Londres. Dans l’application, vous souhaitez un agent qui peut non seulement recommander des hôtels, mais aussi répondre aux questions que les utilisateurs peuvent avoir à leur sujet.

Vous avez sélectionné un modèle GPT-4 pour fournir des réponses génératives. Vous souhaitez maintenant mettre en place un système RAG qui fournira des données d’ancrage au modèle en fonction des avis d’autres utilisateurs, guidant le comportement de la conversation en donnant des recommandations personnalisées.

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
    cd ~/mslearn-genaiops/Files/04/
     ```

1. Entrez les commandes suivantes pour activer un environnement virtuel et installer les bibliothèques dont vous avez besoin :

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv langchain-text-splitters langchain-community langchain-openai
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```powershell
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **your_azure_openai_service_endpoint** et **your_azure_openai_service_api_key** par les valeurs de point de terminaison et de clé que vous avez copiées précédemment.
1. *Une* fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faites un clic droit sur > Enregistrer** pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Mettre en œuvre RAG

Vous allez maintenant exécuter un script qui ingère et prétraite les données, crée des intégrations, et génère un magasin de vecteurs et un index, vous permettant ainsi de mettre en œuvre un système RAG de manière efficace.

1. Exécutez la commande suivante pour **modifier le script** fourni :

    ```powershell
   code RAG.py
    ```

1. Dans le script, recherchez **# Initialiser les composants qui seront utilisés à partir de la suite d’intégrations de LangChain**. Sous ce commentaire, collez le code suivant :

    ```python
   # Initialize the components that will be used from LangChain's suite of integrations
   llm = AzureChatOpenAI(azure_deployment=llm_name)
   embeddings = AzureOpenAIEmbeddings(azure_deployment=embeddings_name)
   vector_store = InMemoryVectorStore(embeddings)
    ```

1. Vérifiez le script et notez qu’il utilise bien un fichier .csv contenant les avis sur les hôtels comme données d’ancrage. Vous pouvez voir le contenu de ce fichier en exécutant la commande `download app_hotel_reviews.csv` dans le volet de ligne de commande et en ouvrant le fichier.
1. Ensuite, recherchez **# Fractionner les documents en blocs pour l’incorporation et le stockage vectoriel**. Sous ce commentaire, collez le code suivant :

    ```python
   # Split the documents into chunks for embedding and vector storage
   text_splitter = RecursiveCharacterTextSplitter(
       chunk_size=200,
       chunk_overlap=20,
       add_start_index=True,
   )
   all_splits = text_splitter.split_documents(docs)
    
   print(f"Split documents into {len(all_splits)} sub-documents.")
    ```

    Le code ci-dessus fractionne un ensemble de documents volumineux en blocs plus petits. C’est important, car de nombreux modèles d’incorporation (comme ceux utilisés pour la recherche sémantique ou les bases de données vectorielles) ont une limite de jetons et fonctionnent mieux sur les textes plus courts.

1. Ensuite, recherchez **# Incorporer le contenu de chaque bloc de texte et insérer ces incorporations dans un magasin vectoriel**. Sous ce commentaire, collez le code suivant :

    ```python
   # Embed the contents of each text chunk and insert these embeddings into a vector store
   document_ids = vector_store.add_documents(documents=all_splits)
    ```

1. Ensuite, recherchez **# Récupérer les documents pertinents à partir du magasin vectoriel en fonction de l’entrée utilisateur**. Sous ce commentaire, collez le code suivant, en respectant l’alinéa approprié :

    ```python
   # Retrieve relevant documents from the vector store based on user input
   retrieved_docs = vector_store.similarity_search(question, k=10)
   docs_content = "\n\n".join(doc.page_content for doc in retrieved_docs)
    ```

    Le code ci-dessus recherche dans le magasin vectoriel les documents les plus similaires à la question d’entrée de l’utilisateur. La question est convertie en vecteur à l’aide du même modèle d’incorporation que celui utilisé pour les documents. Le système compare ensuite ce vecteur à tous les vecteurs stockés et récupère les vecteurs les plus similaires.

1. Enregistrez les changements apportés.
1. Dans la ligne de commande, **exécutez le script** en entrant la commande suivante :

    ```powershell
   python RAG.py
    ```

1. Une fois l’application en cours d’exécution, vous pouvez commencer à poser des questions de type `Where can I stay in London?`, puis à suivre des demandes plus spécifiques.

## Conclusion

Dans cet exercice, vous avez créé un système RAG classique avec ses principaux composants. Lorsque vous utilisez vos propres documents pour alimenter les réponses d’un modèle, vous fournissez des données d’ancrage utilisées par le LLM pour formuler une réponse. Pour une solution d’entreprise, cela permet de contraindre l’IA générative à votre contenu d’entreprise.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
