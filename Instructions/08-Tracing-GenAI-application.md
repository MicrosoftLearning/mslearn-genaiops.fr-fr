---
lab:
  title: Analyser et déboguer votre application d’IA générative avec le traçage
  description: 'Découvrez comment déboguer votre application d’IA générative à l’aide du traçage de son flux de travail, de l’entrée utilisateur à la réponse du modèle, en passant par le post-traitement.'
---

# Analyser et déboguer votre application d’IA générative avec le traçage

Cet exercice prend environ **30** minutes.

> **Note** : cet exercice suppose une certaine connaissance d’Azure AI Foundry, c’est pourquoi certaines instructions sont intentionnellement moins détaillées pour encourager une exploration plus active et un apprentissage pratique.

## Introduction

Dans cet exercice, vous allez exécuter un assistant d’IA générative à plusieurs étapes qui recommande des randonnées et suggère des équipements de plein air. Vous allez utiliser les fonctionnalités de traçage du Kit de développement logiciel (SDK) Azure AI Inference pour analyser la façon dont votre application s’exécute et identifier les points de décision clés pris par le modèle et la logique environnante.

Vous allez interagir avec un modèle déployé pour simuler un parcours utilisateur réel, tracer chaque étape de l’application (de l’entrée utilisateur à la réponse du modèle et au post-traitement) et afficher les données de trace dans Azure AI Foundry. Cela vous aidera à comprendre comment le traçage améliore l’observabilité, simplifie le débogage et prend en charge l’optimisation des performances des applications d’IA génératives.

## Configurer l’environnement

Pour effectuer les tâches de cet exercice, vous avez besoin des éléments suivants :

- Un hub Azure AI Foundry
- Un projet Azure AI Foundry
- Un modèle déployé (comme GPT-4o)
- Une ressource Application Insights connectée

### Créer un projet et un hub AI Foundry

Pour configurer rapidement un hub et un projet, des instructions simples pour utiliser l’interface utilisateur du portail Azure AI Foundry sont fournies ci-dessous.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Sur la page d’accueil, sélectionnez **+Créer un projet**.
1. Dans l’assistant **Créer un projet**, saisissez un nom valide et, si un hub existant est suggéré, choisissez l’option permettant d’en créer un. Passez ensuite en revue les ressources Azure qui seront créées automatiquement pour prendre en charge votre hub et votre projet.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Nom du hub** : *un nom valide pour votre hub*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Emplacement** : sélectionnez **Aidez-moi à choisir**, puis sélectionnez **gpt-4o** dans la fenêtre de l’assistant de l’emplacement et utilisez la région recommandée\*.
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une nouvelle ressource AI Services*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

    > \* Les ressources Azure OpenAI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.

### Déployer un modèle

Pour générer des données que vous pouvez surveiller, vous devez d’abord déployer un modèle et interagir avec celui-ci. Dans les instructions, vous êtes invité à déployer un modèle GPT-4o, mais **vous pouvez utiliser n’importe quel modèle** à partir de la collection Azure OpenAI Service disponible.

1. Dans le menu de gauche, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
1. Dans le menu **+ Déployer un modèle**, sélectionnez **Déployer le modèle de base**.
1. Sélectionnez le modèle **gpt-4o** dans la liste et déployez-le avec les paramètres suivants en sélectionnant **Personnaliser** dans les détails du déploiement :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : Standard
    - **Mise à jour automatique de la version** : activée
    - **Version du modèle** : *sélectionnez la version la plus récente disponible.*
    - **Ressource IA connectée** : *sélectionnez votre connexion de ressources Azure OpenAI*
    - **Limite de débit en jetons par minute (en milliers)** : 5 000
    - **Filtre de contenu** : DefaultV2
    - **Enable dynamic quota** : désactivé

    > **Remarque** : La réduction du nombre de jetons par minute permet d’éviter une surutilisation du quota disponible dans l’abonnement que vous utilisez. 5 000 TPM devraient suffire pour les données utilisées dans cet exercice. Si votre quota disponible est inférieur à cette valeur, vous pourrez tout de même terminer l’exercice, mais vous pourriez rencontrer des erreurs en cas de dépassement de la limite.

1. Attendez la fin du déploiement.

### Se connecter à Application Insights

Connectez Application Insights à votre projet dans Azure AI Foundry pour commencer à collecter des données à des fins d’analyse.

1. Utilisez le menu de gauche, puis sélectionnez la page **Suivi**.
1. **Créez** une ressource Application Insights pour vous connecter à votre application.
1. Entrez un nom de ressource Application Insights, puis sélectionnez **Créer**.

Application Insights est désormais connecté à votre projet et les données commencent à être collectées pour l’analyse.

## Exécuter une application d’IA générative avec Cloud Shell

Vous allez vous connecter à votre projet Azure AI Foundry à partir d’Azure Cloud Shell et interagir par programmation avec un modèle déployé dans le cadre d’une application d’IA générative.

### Interagir avec un modèle déployé

Commencez par récupérer les informations nécessaires pour être authentifié afin d’interagir avec votre modèle déployé. Ensuite, accédez à Azure Cloud Shell et mettez à jour le code de votre application d’IA générative.

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet**.
1. **Enregistrez** la chaîne dans un bloc-notes. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.
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

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-genaiops/Files/08
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
    1. Remplacez l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement GPT-4o (par défaut `gpt-4o`).

1. *Une* fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faites un clic droit sur > Enregistrer** pour **enregistrer vos modifications**, puis utilisez la commande **CTRL+Q** ou **Faites un clic droit > Quitter** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Mettre à jour le code de votre application d’IA générative

Maintenant que votre environnement est configuré et que votre fichier .env est configuré, il est temps de préparer votre script d’assistant IA pour l’exécution. Outre la connexion à un projet IA et l’activation d’Application Insights, vous devez :

- Interagir avec votre modèle déployé
- Définir la fonction pour spécifier votre invite
- Définir le flux principal qui appelle toutes les fonctions

Vous allez ajouter ces trois parties à un script de départ.

1. Exécutez la commande suivante pour **ouvrir le script** fourni :

    ```
   code start-prompt.py
    ```

    Vous verrez que plusieurs lignes clés ont été laissées vides ou marquées avec des commentaires # vides. Votre tâche consiste à terminer le script en copiant et en collant les lignes appropriées ci-dessous dans les emplacements appropriés.

1. Dans le script, recherchez **# Function to call the model and handle tracing**.
1. Sous ce commentaire, collez le code suivant :

    ```
   def call_model(system_prompt, user_prompt, span_name):
        with tracer.start_as_current_span(span_name) as span:
            span.set_attribute("session.id", SESSION_ID)
            span.set_attribute("prompt.user", user_prompt)
            start_time = time.time()
    
            response = chat_client.complete(
                model=model_name,
                messages=[SystemMessage(system_prompt), UserMessage(user_prompt)]
            )
    
            duration = time.time() - start_time
            output = response.choices[0].message.content
            span.set_attribute("response.time", duration)
            span.set_attribute("response.tokens", len(output.split()))
            return output
    ```

1. Dans le script, recherchez **# Function to recommend a hike based on user preferences**.
1. Sous ce commentaire, collez le code suivant :

    ```
   def recommend_hike(preferences):
        with tracer.start_as_current_span("recommend_hike") as span:
            prompt = f"""
            Recommend a named hiking trail based on the following user preferences.
            Provide only the name of the trail and a one-sentence summary.
            Preferences: {preferences}
            """
            response = call_model(
                "You are an expert hiking trail recommender.",
                prompt,
                "recommend_model_call"
            )
            span.set_attribute("hike_recommendation", response.strip())
            return response.strip()
    ```

1. Dans le script, recherchez **# ---- Main Flow ----**.
1. Sous ce commentaire, collez le code suivant :

    ```
   if __name__ == "__main__":
       with tracer.start_as_current_span("trail_guide_session") as session_span:
           session_span.set_attribute("session.id", SESSION_ID)
           print("\n--- Trail Guide AI Assistant ---")
           preferences = input("Tell me what kind of hike you're looking for (location, difficulty, scenery):\n> ")

           hike = recommend_hike(preferences)
           print(f"\n✅ Recommended Hike: {hike}")

           # Run profile function


           # Run match product function


           print(f"\n🔍 Trace ID available in Application Insights for session: {SESSION_ID}")
    ```

1. **Enregistrez les modifications** que vous avez apportées au script.
1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour **exécuter le script** :

    ```
   python start-prompt.py
    ```

1. Donnez une description du type de randonnée que vous recherchez, par exemple :

    ```
   A one-day hike in the mountains
    ```

    Le modèle génèrera une réponse, qui sera capturée avec Application Insights. Vous pouvez visualiser les traces dans le **portail Azure AI Foundry**.

> **Note** : l’affichage des données de surveillance dans Azure Monitor peut prendre quelques minutes.

## Afficher les données de traces dans le portail Azure AI Foundry

Après avoir exécuté le script, vous avez capturé une trace de l’exécution de votre application d’IA. Vous allez maintenant l’explorer à l’aide d’Application Insights dans Azure AI Foundry.

> **Note :** plus tard, vous réexécuterez le code et afficherez à nouveau les traces dans le portail Azure AI Foundry. Commençons par regarder où trouver les traces pour les visualiser.

### Accéder au portail Azure AI Foundry

1. **Gardez Cloud Shell ouvert !** Vous y reviendrez pour mettre à jour le code et le réexécuter.
1. Accédez à l’onglet de votre navigateur avec le **portail Azure AI Foundry** ouvert.
1. Dans le menu de gauche, sélectionnez **Suivi**.
1. *Si* aucune donnée n’est affichée, **actualisez** la vue.
1. Sélectionnez la trace **train_guide_session** pour ouvrir une nouvelle fenêtre qui affiche plus de détails.

### Vérifier votre trace

Cette vue affiche la trace d’une session complète de l’assistant d’IA Trail Guide.

- **Étendue de niveau supérieur** : trail_guide_session. Il s’agit de l’étendue parente. Elle représente l’ensemble de l’exécution de votre assistant du début à la fin.

- **Étendues enfants imbriquées** : chaque ligne mise en retrait représente une opération imbriquée. Vous y trouverez :

    - **recommend_hike** qui capture votre logique pour décider d’une randonnée.
    - **recommend_model_call** qui est l’étendue créée par call_model() à l’intérieur de recommend_hike.
    - **chat gpt-4o** qui est automatiquement instrumenté par le Kit de développement logiciel (SDK) Azure AI Inference pour afficher l’interaction LLM réelle.

1. Vous pouvez cliquer sur n’importe quelle étendue pour afficher :

    1. Sa durée.
    1. Ses attributs tels que l’invite utilisateur, les jetons utilisés, le temps de réponse.
    1. Toutes les erreurs ou données personnalisées attachées à **span.set_attribute(...)**.

## Ajouter d’autres fonctions à votre code

1. Accédez à l’onglet de votre navigateur avec le **portail Azure** ouvert.
1. Exécutez la commande suivante pour **ouvrir de nouveau le script :**

    ```
   code start-prompt.py
    ```

1. Dans le script, recherchez **# Function to generate a trip profile for the recommended hike**.
1. Sous ce commentaire, collez le code suivant :

    ```
   def generate_trip_profile(hike_name):
       with tracer.start_as_current_span("trip_profile_generation") as span:
           prompt = f"""
           Hike: {hike_name}
           Respond ONLY with a valid JSON object and nothing else.
           Do not include any intro text, commentary, or markdown formatting.
           Format: {{ "trailType": ..., "typicalWeather": ..., "recommendedGear": [ ... ] }}
           """
           response = call_model(
               "You are an AI assistant that returns structured hiking trip data in JSON format.",
               prompt,
               "trip_profile_model_call"
           )
           print("🔍 Raw model response:", response)
           try:
               profile = json.loads(response)
               span.set_attribute("profile.success", True)
               return profile
           except json.JSONDecodeError as e:
               print("❌ JSON decode error:", e)
               span.set_attribute("profile.success", False)
               return {}
    ```

1. Dans le script, recherchez **# Function to match recommended gear with products in the catalog**.
1. Sous ce commentaire, collez le code suivant :

    ```
   def match_products(recommended_gear):
       with tracer.start_as_current_span("product_matching") as span:
           matched = []
           for gear_item in recommended_gear:
               for product in mock_product_catalog:
                   if any(word in product.lower() for word in gear_item.lower().split()):
                       matched.append(product)
                       break
           span.set_attribute("matched.count", len(matched))
           return matched
    ```

1. Dans le script, recherchez **# Run profile function**.
1. Ci-dessous et **aligné avec** ce commentaire, collez le code suivant :

    ```
           profile = generate_trip_profile(hike)
           if not profile:
               print("Failed to generate trip profile. Please check Application Insights for trace.")
               exit(1)

           print(f"\n📋 Trip Profile for {hike}:")
           print(json.dumps(profile, indent=2))
    ```

1. Dans le script, recherchez **# Run match product function**.
1. Ci-dessous et **aligné avec** ce commentaire, collez le code suivant :

    ```
           matched = match_products(profile.get("recommendedGear", []))
           print("\n🛒 Recommended Products from Lakeshore Retail:")
           print("\n".join(matched))
    ```

1. **Enregistrez les modifications** que vous avez apportées au script.
1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour **exécuter le script** :

    ```
   python start-prompt.py
    ```

1. Donnez une description du type de randonnée que vous recherchez, par exemple :

    ```
   I want to go for a multi-day adventure along the beach
    ```

<br>
<details>
<summary><b>Script de solution</b> : Si votre code ne fonctionne pas.</summary><br>
<p>Si vous inspectez la trace LLM pour la fonction generate_trip_profile, vous remarquerez que la réponse de l’assistant inclut des backticks et le mot json pour mettre en forme la sortie en tant que bloc de code.

Bien que cela soit utile pour l’affichage, cela entraîne des problèmes dans le code, car la sortie n’est plus du JSON valide. Cela entraîne une erreur d’analyse lors du traitement ultérieur.

L’erreur est probablement due à la façon dont le LLM doit adhérer à un format spécifique pour sa sortie. L’inclusion des instructions dans l’invite utilisateur paraît plus efficace que de les placer dans l’invite du système.</p>
</details>


> **Note** : l’affichage des données de surveillance dans Azure Monitor peut prendre quelques minutes.

### Afficher les nouvelles traces dans le portail Azure AI Foundry

1. Revenez au portail Azure AI Foundry.
1. Une nouvelle trace portant le même nom **trail_guide_session** doit apparaître. Actualisez la vue si nécessaire.
1. Sélectionnez la nouvelle trace pour ouvrir la vue plus détaillée.
1. Passez en revue les nouvelles étendues enfants imbriquées, **trip_profile_generation** et **product_matching**.
1. Sélectionnez **product_matching** et passez en revue les métadonnées qui s’affichent.

    Dans la fonction product_matching, vous avez inclus **span.set_attribute("matched.count", len(matched)).** En définissant l’attribut avec la paire clé-valeur **matched.count** et la longueur de la variable mise en correspondance, vous avez ajouté ces informations à la trace **product_matching**. Vous trouverez cette paire clé-valeur sous les **attributs** dans les métadonnées.

## (FACULTATIF) Tracer une erreur

Si vous avez du temps supplémentaire, vous pouvez examiner comment utiliser les traces en cas d’erreur. Un script susceptible de déclencher une erreur vous est fourni. Exécutez-le et passez les traces en revue.

Il s’agit d’un exercice conçu pour vous mettre au défi, ce qui signifie que les instructions sont intentionnellement moins détaillées.

1. Dans Cloud Shell, ouvrez le script **error-prompt.py**. Ce script se trouve dans le même répertoire que le script **start-prompt.py**. Vérifiez son contenu.
1. Exécutez le script **error-prompt.py**. Fournissez une réponse dans la ligne de commande lorsque vous y êtes invité.
1. *Normalement*, le message de sortie devrait inclure **Failed to generate trip profile. Please check Application Insights for trace.**.
1. Accédez à la trace de **trip_profile_generation** et examinez pourquoi une erreur s’est produite.

<br>
<details>
<summary><b>Obtenez une réponse</b> : pourquoi vous avez pu rencontrer une erreur...</summary><br>
<p>Si vous inspectez la trace LLM pour la fonction generate_trip_profile, vous remarquerez que la réponse de l’assistant inclut des backticks et le mot json pour mettre en forme la sortie en tant que bloc de code.

Bien que cela soit utile pour l’affichage, cela entraîne des problèmes dans le code, car la sortie n’est plus du JSON valide. Cela entraîne une erreur d’analyse lors du traitement ultérieur.

L’erreur est probablement due à la façon dont le LLM doit adhérer à un format spécifique pour sa sortie. L’inclusion des instructions dans l’invite utilisateur paraît plus efficace que de les placer dans l’invite du système.</p>
</details>

## Où trouver d’autres labos

Vous pouvez explorer d’autres labos et exercices dans le [portail d’apprentissage d’Azure AI Foundry](https://ai.azure.com) ou consulter la **section de labo** du cours pour obtenir d’autres activités.
