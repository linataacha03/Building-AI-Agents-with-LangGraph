# Building-AI-Agents-with-LangGraph
Assistant universitaire multi-agents construit avec LangGraph : classification, routage conditionnel, outils (calculatrice, date/heure), mémoire par thread et gestion d'erreurs.
# Installation
Python 3.10+ (testé sur Kaggle, Python 3.13).
pip install -U langgraph langchain langchain-google-genai langchain-community pandas
# Clé API
Le LLM est Google Gemini (gemini-3.5-flash-lite). Clé gratuite sur https://aistudio.google.com (Get API key).
# Exécution
Ouvrir langgraph_lab.ipynb et choisir Run All (Internet activé sur Kaggle).
# Choix de conception
- State typé (TypedDict) ; messages utilise le reducer add_messages pour accumuler l'historique au lieu de l'écraser.
- Classifieur LLM + conditional edges : un nœud classe la question, puis une fonction de routage choisit l'agent. Toute étiquette inattendue retombe sur GENERAL.
- Un agent par catégorie, différencié par son prompt système (même LLM, comportements différents). Le prompt ADMINISTRATION interdit d'inventer des règles officielles.
- Boucle outils : agent → tools → agent. Après un appel d'outil, on revient à l'agent qui l'a demandé (route_after_tools).
- Mémoire : MemorySaver + thread_id. Un thread = une conversation ; deux threads sont isolés. La mémoire est en RAM (perdue au redémarrage du kernel).
- Gestion d'erreurs : question vide → EMPTY ; erreur API dans le classifieur → ERROR ; les deux vont au nœud fallback. Une erreur API dans un agent produit un message d'excuse au lieu de faire planter le graphe. ToolNode(handle_tool_errors=True) renvoie l'erreur d'outil au modèle.
- Sécurité : la calculatrice du lab utilisait eval (exécution de code arbitraire). Elle est remplacée par un évaluateur AST limité à l'arithmétique (__import__('os') → Invalid expression).
- to_text() : Gemini renvoie .content sous forme de liste de blocs, OpenAI sous forme de chaîne. Cette fonction normalise le texte (et corrige l'erreur 'list' object has no attribute 'strip').
