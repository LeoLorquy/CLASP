# 🧠 README — Système de Conversation Multi-Utilisateur « Naturelle »
> Faire d’un LLM un **interlocuteur social** au lieu d’un **répondeur automatique**.

## 1. Le problème
Un grand modèle de langage, tel quel, **ne comprend pas la vie en groupe** :
- Il traite chaque message isolément.
- Il répond **à tout le monde, tout le temps, sans filtre**.
- Il **perd le fil** dès que 3 ou 4 personnes parlent en même temps.
- Il **gaspile des tokens** sur des “salut ça va ?” lancés dans le vide.
- Il **hallucine** quand le bruit devient trop fort.

En résumé : aucune conscience conversationnelle multi-utilisateur.


## 2. La solution — sans fine-tuning, juste des règles

### 2.1 Cycle de vie d’une conversation

| Étape                      | Déclencheur                                  | Action                                                                                                                  |
| -------------------------- | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Démarrage**              | Ping (`@bot`) **ou** réponse au bot          | Créer une session en mémoire :<br>`{threadId, rootUserId, participants<Set>, summary, lastTurnTs}`                      |
| **Premiers échanges**      | 2-3 messages échangés                        | Générer un **résumé incrémental** (`prompt: "Résume en 2 phrases max"`).                                                |
| **Extension**              | Message tiers                                | `embedding(message)` ➜ cosine vs résumé > 0.75 ➜ ajouter `userId` + mettre à jour le résumé.                            |
| **Dérive / essoufflement** | Nouveau message                              | `prompt: "Ce message est-il hors-sujet ? (true/false)"`<br>Si `true` → mettre à jour le résumé ou `return;` la session. |
| **Fin**                    | Inactivité 5 min **ou** token-budget dépassé | TTL expirée → purge de la session.                                                                                      |


### 2.2 Prompts de référence
Résume en 2 phrases la conversation ci-dessus.

### Détection de pertinence
Ce message est-il hors-sujet par rapport au résumé suivant ?
Résumé : <summary>
Message : <text>
Répond uniquement par true ou false.


## 3. Bénéfices immédiats

| Problème classique                      | Résolu par                                    |
| --------------------------------------- | --------------------------------------------- |
| Faux positifs (réponse à tout le monde) | Déclencheur explicite + classifieur           |
| Prompt surchargé                        | Résumé incrémental (compression intelligente) |
| Hors-sujet / spam                       | Dérive détectée → résumé mis à jour ou fin    |
| Hallucinations en chaîne                | Contexte fermé et filtré                      |
| Expérience “robot”                      | Conversation continue, cohérente, sociale     |


## 4. Comparaison « avant / après »

| Scénario                | Bot classique              | Avec ce système                         |
| ----------------------- | -------------------------- | --------------------------------------- |
| 50 personnes parlent    | Répond à chaque “lol”      | Ignore le bruit, suit le sujet          |
| Alice pose une question | Réponse à Alice seule      | Invite Bob si son message est pertinent |
| Sujet dérive            | Continue n’importe comment | Regénère le résumé ou s’arrête          |
| 10 min plus tard        | Toujours “actif”           | Session auto-nettoyée                   |


## 5. Implémentation minimale (MVP)

📁 src/
   conversation.js       # état + TTL
   classifier.js         # embedding + seuil
   summarizer.js         # appel LLM + prompts
   drift.js              # true/false hors-sujet

1. État en mémoire (Map ou Redis).
2. Embedding : all-MiniLM-L6-v2 local ou service gratuit (OpenRouter, etc.).
3. TTL : setTimeout 5 min ou Date.now() - lastTurnTs.


## 6. Améliorations futures

| Idée                      | Description                                                                                    |
| ------------------------- | ---------------------------------------------------------------------------------------------- |
| **Hiérarchie de sujets**  | Plusieurs résumés imbriqués si la discussion bifurque réellement.                              |
| **Contexte global**       | Garder un résumé des 10 derniers messages en plus du résumé principal.                         |
| **Réactions utilisateur** | 👎 sur une réponse → réduction du poids de l’utilisateur dans le résumé.                       |
| **Whisper mode**          | Le bot peut glisser un message éphémère à un participant sans briser la conversation publique. |


## 7. Choix de modèles (gratuit)

| Rang | Modèle               | Utilité ici                              |
| ---- | -------------------- | ---------------------------------------- |
| 🥇   | **Gemini 2.5 Pro**   | Raisonnement long, 1 M token, multimodal |
| 🥈   | **DeepSeek-R1-0528** | Code & explications pointues             |
| 🥉   | **Kimi K2**          | Français naturel, 200 k ctx              |
| —    | **Mistral Medium**   | Endpoint stable, moins profond           |

> Le cerveau vient du modèle.
> La mémoire sociale vient de ton code.

pour une nouvelle version du CLASP je propose de l'embedding a la place d'un resumé pour detecter si les x derniers message on pour meme sujet.
ou meme un algo cosine-similarity pour detecter les reponse de tel ou tel questions il est donc plus simple de savoir si une personne repond au bot ou non SANS explicitement mentionné le bot ( avec comme exemple discord )

# CLASP – Context-Locked Adaptive Social Prompting
> by Léo Eyrard
