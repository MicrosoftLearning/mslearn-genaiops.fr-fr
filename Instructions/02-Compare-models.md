---
lab:
  title: Comparer les modèles de langage du catalogue de modèles
  description: Découvrez comment comparer et sélectionner les modèles appropriés pour votre projet IA générative.
---

## Comparer les modèles de langage du catalogue de modèles

Lorsque vous avez défini votre cas d’usage, vous pouvez utiliser le catalogue de modèles pour déterminer si un modèle IA résout votre problème. Vous pouvez utiliser le catalogue de modèles pour sélectionner des modèles à déployer, que vous pouvez ensuite comparer pour explorer le modèle le mieux adapté à vos besoins.

Dans cet exercice, vous comparez deux modèles de langage via le catalogue de modèles dans le portail Azure AI Foundry.

Cet exercice prend environ **30** minutes.

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

## Comparer les modèles

Vous savez qu’il existe trois modèles qui acceptent les images comme entrée dont l’infrastructure d’inférence est entièrement gérée par Azure. Maintenant, vous devez les comparer pour décider lequel est idéal pour notre cas d’usage.

1. Dans un nouvel onglet de navigateur, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
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

## Configurer votre environnement de développement dans Cloud Shell

Pour expérimenter et itérer rapidement, vous utiliserez un ensemble de scripts Python dans Cloud Shell.

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet**.
1. Enregistrez la chaîne dans un bloc-notes. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.
1. Dans l’onglet Portail Azure, ouvrez Cloud Shell si vous l’avez fermé avant et exécutez la commande suivante pour accéder au dossier avec les fichiers de code utilisés dans cet exercice :

     ```powershell
    cd ~/mslearn-genaiops/Files/02/
     ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques dont vous avez besoin :

    ```powershell
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai matplotlib
    ```

1. Saisissez la commande suivante pour ouvrir le fichier de configuration fourni :

    ```powershell
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée à partir de la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry). Notez que le premier et deuxième modèle utilisés dans l’exercice sont respectivement **gpt-4o** et **gpt-4o-mini**.
1. *Une* fois que vous avez remplacé l’espace réservé, dans l’éditeur de code, utilisez la commande **CTRL+S**ou**Faites un clic droit sur > Enregistrer** pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Envoyer des invites à vos modèles déployés

Vous allez maintenant exécuter plusieurs scripts qui envoient différentes invites à vos modèles déployés. Ces interactions génèrent des données que vous pouvez observer ultérieurement dans Azure Monitor.

1. Exécutez la commande suivante pour **afficher le premier script** fourni :

    ```powershell
   code model1.py
    ```

Le script encodera l’image utilisée dans cet exercice dans une URL de données. Cette URL sera utilisée pour incorporer l’image directement dans la requête de saisie semi-automatique de conversation avec la première invite de texte. Le script affichera ensuite la réponse du modèle, l’ajoutera à l’historique des conversations, puis enverra une deuxième invite. La deuxième invite est envoyée et stockée dans le but de rendre les mesures observées ultérieurement plus loin. Néanmoins, vous pouvez supprimer les marques de commentaire de la section facultative du code pour obtenir également la deuxième réponse en sortie.

1. Dans le volet de ligne de commande Cloud Shell sous l’éditeur de code, entrez la commande suivante pour exécuter le **premier** script :

    ```powershell
   python model1.py
    ```

    Le modèle génèrera une réponse, qui sera capturée avec Application Insights pour une analyse plus approfondie. Utilisons le deuxième modèle pour explorer leurs différences.

1. Dans le volet de ligne de commande Cloud Shell sous l’éditeur de code, entrez la commande suivante pour exécuter le **deuxième** script :

    ```powershell
   python model2.py
    ```

    Maintenant que vous disposez des sorties des deux modèles, y a-t-il des différences entre elles ?

    > **Remarque** : Vous pouvez également tester les scripts donnés comme réponses en copiant les blocs de code, en exécutant la commande `code your_filename.py`, en collant le code dans l’éditeur, en enregistrant le fichier, puis en exécutant la commande `python your_filename.py`. Si le script s’est exécuté correctement,une image enregistrée devrait être disponible. Vous pouvez la télécharger sur `download imgs/gpt-4o.jpg` ou `download imgs/gpt-4o-mini.jpg`.

## Comparer l’utilisation des jetons des modèles

Enfin, vous allez exécuter un troisième script qui tracera le nombre de jetons traités au fil du temps pour chaque modèle. Ces données sont obtenues à partir d’Azure Monitor.

1. Avant d’exécuter le dernier script, vous devez copier l’ID de la ressource de votre Azure AI Services à partir du portail Azure. Accédez à la page de présentation de votre ressource Azure AI Services, puis sélectionnez **Affichage JSON**. Copiez l’ID de la ressource et remplacez l’espace réservé `your_resource_id` dans le fichier de code :

    ```powershell
   code plot.py
    ```

1. Enregistrez les changements apportés.

1. Dans le volet de ligne de commande Cloud Shell sous l’éditeur de code, entrez la commande suivante pour exécuter le **troisième** script :

    ```powershell
   python plot.py
    ```

1. Une fois le script terminé, entrez la commande suivante pour télécharger le tracé des mesures :

    ```powershell
   download imgs/plot.png
    ```

## Conclusion

Après examen du tracé et rappel des valeurs de point de référence dans le rapport Exactitude vs. En vous basant sur le graphique des coûts présenté précédemment, pouvez-vous déterminer quel modèle est le mieux adapté à votre cas d’utilisation ? La différence de précision entre les sorties justifie-t-elle l’écart observé dans le nombre de jetons générés et par conséquent dans le coût ?

## Nettoyage

Si vous avez terminé d’explorer Azure AI Services, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com?azure-portal=true) dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
