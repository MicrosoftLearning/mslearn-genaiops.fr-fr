---
lab:
  title: Orchestrer un système RAG
---

## Orchestrer un système RAG

Les systèmes de génération augmentée de récupération (RAG) combinent la puissance des grands modèles de langage avec des mécanismes de récupération efficaces pour améliorer la précision et la pertinence des réponses générées. En tirant parti de LangChain pour l’orchestration et Azure AI Foundry pour les fonctionnalités d’IA, nous pouvons créer un pipeline robuste qui récupère des informations pertinentes à partir d’un jeu de données et génère des réponses cohérentes. Dans cet exercice, vous allez suivre les étapes de configuration de votre environnement, de prétraitement des données, de création d’incorporations et de création d’un index, vous permettant ainsi d’implémenter efficacement un système RAG.

## Scénario

Imaginez que vous souhaitez créer une application qui fournit des recommandations sur les hôtels. Dans l’application, vous souhaitez un agent qui peut non seulement recommander des hôtels, mais aussi répondre aux questions que les utilisateurs peuvent avoir à leur sujet.

Vous avez sélectionné un modèle GPT-4 pour fournir des réponses génératives. Vous souhaitez maintenant mettre en place un système RAG qui fournira des données d’ancrage au modèle en fonction des avis d’autres utilisateurs, guidant le comportement de la conversation en donnant des recommandations personnalisées.

Commençons par déployer les ressources nécessaires pour créer cette application.

## Créer un projet et un hub Azure AI

Vous pouvez créer un hub Azure AI et un projet manuellement via le portail Azure AI Foundry, ainsi que déployer les modèles utilisés dans l’exercice. Toutefois, vous pouvez également automatiser ce processus via l’utilisation d’une application modèle avec [Azure Developer CLI (azd).](https://aka.ms/azd)

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
    Get-AzCognitiveServicesAccountKey -ResourceGroupName <rg-env_name> -Name <aoai-xxxxxxxxxx> | Select-Object -Property Key1
     ```

1. Copiez ces valeurs, car elles seront utilisées ultérieurement.

## Configurer votre environnement de développement local

Pour expérimenter et itérer rapidement, vous allez utiliser un notebook avec du code Python dans Visual Studio (VS) Code. Préparons VS Code pour l’idéation locale.

1. Ouvrez VS Code et **clonez** le référentiel Git suivant : [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git).
1. Stockez le clone sur un lecteur local et ouvrez le dossier après le clonage.
1. Dans l’Explorateur VS Code (volet gauche), ouvrez le notebook **04-RAG.ipynb** dans le dossier **Files/04**.
1. Exécutez toutes les cellules dans le notebook.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
