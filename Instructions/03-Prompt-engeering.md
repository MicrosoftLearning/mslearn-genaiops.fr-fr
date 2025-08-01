---
lab:
  title: Explorer l’ingénierie des invites avec Prompty
  description: Découvrez comment utiliser Prompty pour tester et améliorer rapidement les différentes invites avec votre modèle de langage et vous assurer qu’elles sont construites et orchestrées pour obtenir de meilleurs résultats.
---

## Explorer l’ingénierie des invites avec Prompty

Cet exercice prend environ **45** minutes.

> **Note** : cet exercice suppose une certaine connaissance d’Azure AI Foundry, c’est pourquoi certaines instructions sont intentionnellement moins détaillées pour encourager une exploration plus active et un apprentissage pratique.

## Introduction

Pendant l’idéation, vous souhaitez tester et améliorer rapidement les différentes invites avec votre modèle de langage. Il existe différentes façons d’aborder l’ingénierie d’invite : via le terrain de jeu dans le portail Azure AI Foundry ou en utilisant Prompty pour une approche plus orientée code.

Dans cet exercice, vous allez explorer l’ingénierie des invites avec Prompty dans Azure Cloud Shell, à l’aide d’un modèle déployé via Azure AI Foundry.

## Configurer l’environnement

Pour effectuer les tâches de cet exercice, vous avez besoin des éléments suivants :

- Un hub Azure AI Foundry
- Un projet Azure AI Foundry
- Un modèle déployé (comme GPT-4o).

### Créer un projet et un hub Azure AI

> **Remarque** : Si vous avez déjà un projet Azure AI, vous pouvez ignorer cette procédure et utiliser votre projet existant.

Vous pouvez créer un projet Azure AI manuellement via le portail Azure AI Foundry, ainsi que déployer le modèle utilisé dans l’exercice. Toutefois, vous pouvez également automatiser ce processus via l’utilisation d’une application modèle avec [Azure Developer CLI (azd).](https://aka.ms/azd)

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

### Configurer votre environnement virtuel dans Cloud Shell

Pour expérimenter et itérer rapidement, vous utiliserez un ensemble de scripts Python dans Cloud Shell.

1. Dans le volet de ligne de commande Cloud Shell, entrez la commande suivante pour accéder au dossier contenant des fichiers de code utilisés dans cet exercice :

     ```powershell
    cd ~/mslearn-genaiops/Files/03/
     ```

1. Entrez les commandes suivantes pour activer un environnement virtuel et installer les bibliothèques dont vous avez besoin :

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv openai tiktoken azure-ai-projects prompty[azure]
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```powershell
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **ENDPOINTNAME** et **APIKEY** par les valeurs de point de terminaison et de clé que vous avez copiées précédemment.
1. *Une* fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faites un clic droit sur > Enregistrer** pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Optimiser l’invite système

La réduction de la longueur des invites système tout en conservant les fonctionnalités de l’IA générée est essentielle pour les déploiements à grande échelle. Des invites plus courtes peuvent accélérer les temps de réponse, car le modèle IA traite moins de jetons et utilise également moins de ressources de calcul.

1. Entrez la commande suivante pour ouvrir le fichier d’application fourni :

    ```powershell
   code optimize-prompt.py
    ```

    Passez en revue le code et notez que le script exécute le `start.prompty` fichier de modèle qui a déjà une invite système prédéfinie.

1. Exécutez `code start.prompty` cette commande pour passer en revue l’invite système. Réfléchissez à la façon dont vous pouvez la raccourcir tout en conservant son intention claire et efficace. Par exemple :

   ```python
   original_prompt = "You are a helpful assistant. Your job is to answer questions and provide information to users in a concise and accurate manner."
   optimized_prompt = "You are a helpful assistant. Answer questions concisely and accurately."
   ```

   Supprimez les mots redondants et concentrez-vous sur les instructions essentielles. Enregistrez votre invite optimisée dans le fichier.

### Tester et valider votre optimisation

Il est important de tester les modifications apportées à l’invite pour vous assurer de réduire l’utilisation des jetons sans perdre la qualité.

1. Exécutez `code token-count.py` pour ouvrir et examiner l’application de compteur de jetons fournie dans l’exercice. Si vous avez utilisé une invite optimisée différente de celle fournie dans l’exemple ci-dessus, vous pouvez également l’utiliser dans cette application.

1. Exécutez le script avec `python token-count.py` et observez la différence dans le nombre de jetons. Assurez-vous que l’invite optimisée produit toujours des réponses de haute qualité.

## Analyser les interactions utilisateur

Comprendre comment les utilisateurs interagissent avec votre application permet d’identifier les modèles qui augmentent l’utilisation des jetons.

1. Passez en revue un exemple de jeu de données d’invites utilisateur :

    - **« Résumez le tracé de la guerre et de *la tranquillité* ».**
    - **« Quels sont les faits amusants sur les chats ? »**
    - **« Écrivez un plan d’entreprise détaillé pour une start-up qui utilise l’IA pour optimiser les chaînes d’approvisionnement. »**
    - **« Traduisez « Hello, how are you ? » en français. »**
    - **« Expliquez l’enchevêtrement quantique à une personne de 10 ans. »**
    - **« Donnez-moi 10 idées créatives pour une histoire courte de science-fi. »**

    Pour chacune d’elles, déterminez si elle est susceptible d’entraîner une **réponse courte**, **moyenne** ou **longue/complexe** de l’IA.

1. Passez en revue vos catégorisations. Quels modèles remarquez-vous ? Vous devez :

    - **Le niveau d’abstraction** (par exemple, créatif ou non) affecte-t-il la longueur ?
    - Les **invites ouvertes** ont-ils tendance à être plus longues ?
    - Comment la **complexité des instructions** (par exemple, « expliquer comme si j’avais 10 ans » influence-t-elle la réponse ?

1. Entrez la commande suivante pour exécuter l’application à **invite d’optimisation ** :

    ```
   python optimize-prompt.py
    ```

1. Utilisez certains des exemples fournis ci-dessus pour vérifier votre analyse.
1. Utilisez maintenant l’invite de formulaire long suivante et passez en revue sa sortie :

    ```
   Write a comprehensive overview of the history of artificial intelligence, including key milestones, major contributors, and the evolution of machine learning techniques from the 1950s to today.
    ```

1. Réécrivez cette invite pour :

    - Limiter l’étendue
    - Définir les attentes en termes de concision
    - Utiliser la mise en forme ou la structure pour guider la réponse

1. Comparez les réponses pour vérifier que vous avez obtenu une réponse plus concise.

> **REMARQUE** : Vous pouvez utiliser `token-count.py` pour comparer l’utilisation des jetons dans les deux réponses.
<br>
<details>
<summary><b>Exemple d’invite réécrite :</b></summary><br>
<p>« Donnez un résumé de 5 jalons clés dans l’historique de l’IA. »</p>
</details>

## [**FACULTATIF**] Appliquer vos optimisations dans un scénario réel

1. Imaginez que vous créez un chatbot de support client qui doit fournir des réponses rapides et précises.
1. Intégrez votre invite système optimisée et votre modèle dans le code du chatbot (*vous pouvez l’utiliser `optimize-prompt.py` comme point de départ*).
1. Testez le chatbot avec différentes requêtes utilisateur pour vous assurer qu’il répond efficacement et efficacement.

## Conclusion

L’optimisation des invites est une compétence clé pour réduire les coûts et améliorer les performances des applications d’intelligence artificielle générées. En raccourcissant les invites, en utilisant des modèles et en analysant les interactions utilisateur, vous pouvez créer des solutions plus efficaces et scalables.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
