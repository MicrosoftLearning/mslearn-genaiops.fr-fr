---
lab:
  title: Comparer les modèles de langage du catalogue de modèles
---

## Comparer les modèles de langage du catalogue de modèles

Lorsque vous avez défini votre cas d’usage, vous pouvez utiliser le catalogue de modèles pour déterminer si un modèle IA résout votre problème. Vous pouvez utiliser le catalogue de modèles pour sélectionner des modèles à déployer, que vous pouvez ensuite comparer pour explorer le modèle le mieux adapté à vos besoins.

Dans cet exercice, vous comparez deux modèles de langage via le catalogue de modèles dans le portail Azure AI Foundry.

Cet exercice prend environ **25** minutes.

## Scénario

Imaginez que vous souhaitez créer une application pour aider les étudiants à apprendre à coder en Python. Dans l’application, vous souhaitez un tuteur automatisé qui peut aider les étudiants à écrire et évaluer du code. Dans un exercice, les étudiants doivent trouver le code Python nécessaire pour tracer un graphique en secteurs, en fonction de l’exemple d’image suivant :

![Graphique en secteurs montrant les notes obtenues dans un examen avec des sections pour les mathématiques (34,9 %), la physique (28,6 %), la chimie (20,6 %) et l’anglais (15,9 %)](./images/demo.png)

Vous devez sélectionner un modèle de langage qui accepte des images en tant qu’entrée et peut générer du code précis. Les modèles disponibles qui répondent à ces critères sont GPT-4 Turbo, GPT-4o et GPT-4o mini.

Commençons par déployer les ressources nécessaires pour utiliser ces modèles dans le portail Azure AI Foundry.

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

## Comparer les modèles

Vous savez qu’il existe trois modèles qui acceptent les images comme entrée dont l’infrastructure d’inférence est entièrement gérée par Azure. Maintenant, vous devez les comparer pour décider lequel est idéal pour notre cas d’usage.

1. Dans un navigateur web, ouvrez le [portail Azure Ai Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Si vous y êtes invité, sélectionnez le projet IA créé précédemment.
1. Accédez à la page **Catalogue de modèles** à l’aide du menu de gauche.
1. Sélectionnez **Comparer les modèles** (recherchez le bouton en regard des filtres dans le volet de recherche).
1. Supprimez les modèles sélectionnés.
1. Ajoutez un par un les trois modèles que vous souhaitez comparer : **gpt-4**, **gpt-4o** et **gpt-4o-mini**. Pour **gpt-4**, assurez-vous que la version sélectionnée est **turbo-2024-04-09**, car il s’agit de la seule version qui accepte les images comme entrée.
1. Remplacez l’axe X par **Précision**.
1. Vérifiez que l’axe Y est défini sur **Coût**.

Passez en revue le tracé et essayez de répondre aux questions suivantes :

- *Quel modèle est le plus précis ?*
- *Quel modèle est le moins cher à utiliser ?*

La précision des métriques de benchmark est calculée en fonction des jeux de données génériques disponibles publiquement. À partir du tracé, nous pouvons déjà filtrer l’un des modèles, car il a le coût le plus élevé par jeton, mais pas la précision la plus élevée. Avant de prendre une décision, examinons la qualité des sorties des deux modèles restants spécifiques à votre cas d’usage.

## Configurer votre environnement de développement local

Pour expérimenter et itérer rapidement, vous allez utiliser un notebook avec du code Python dans Visual Studio (VS) Code. Préparons VS Code pour l’idéation locale.

1. Ouvrez VS Code et **clonez** le référentiel Git suivant : [https://github.com/MicrosoftLearning/mslearn-genaiops.git](https://github.com/MicrosoftLearning/mslearn-genaiops.git).
1. Stockez le clone sur un lecteur local et ouvrez le dossier après le clonage.
1. Dans l’Explorateur VS Code (volet gauche), ouvrez le notebook **02-Compare-models.ipynb** dans le dossier **Files/02**.
1. Exécutez toutes les cellules dans le notebook.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
