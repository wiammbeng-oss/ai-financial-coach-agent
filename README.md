# Coach financier IA multi-agents (Google ADK + Gemini)

Version adaptée d'un projet open source : un coach financier composé de **trois agents IA spécialisés** (budget, épargne, dettes), enchaînés les uns après les autres, avec une interface **Streamlit**.

## Projet d'origine et crédits

Ce dépôt est basé sur le projet **ai_financial_coach_agent** de Shubham Saboo, issu du dépôt open source [awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) (`advanced_ai_agents/multi_agent_apps/ai_financial_coach_agent`).

- Tutoriel d'origine : [Build a Multi-Agent Personal Finance Coach](https://www.theunwindai.com/p/build-a-multi-agent-personal-finance-coach) (Unwind AI, avril 2025).
- Le code de l'application appartient à ses auteurs. Veuillez consulter la licence du dépôt d'origine.
- Mon travail porte sur l'**installation, le dépannage, l'adaptation de la configuration et l'analyse des résultats**, détaillés ci-dessous.

## Ce que fait l'application

1. L'utilisateur saisit son revenu, son nombre de personnes à charge, ses dépenses (CSV ou saisie manuelle) et ses dettes.
2. Trois agents s'exécutent à la suite et se passent un état partagé :
   - **Agent Budget** : analyse des dépenses par catégorie et recommandations d'économies.
   - **Agent Épargne** : fonds d'urgence et répartition de l'épargne.
   - **Agent Dettes** : comparaison des méthodes *avalanche* et *boule de neige*.
3. Les résultats sont affichés sous forme de graphiques et de recommandations.

Chaque agent produit une sortie **structurée** (schémas Pydantic). Si un agent échoue, des résultats par défaut sont prévus dans le code.

## Mes modifications

| # | Modification | Raison |
|---|---|---|
| 1 | Modèle `gemini-2.5-flash` remplacé par `gemini-3.1-flash-lite` dans les trois agents | Google a retiré `gemini-2.5-flash` pour les nouveaux utilisateurs (erreur 404) ; `gemini-3.8-flash` renvoyait des erreurs 503 (forte demande) |
| 2 | Ajout de la dépendance `Deprecated` dans `requirements.txt` | Bibliothèque manquante, erreur `No module named 'deprecated'` au démarrage |
| 3 | `numpy==1.26.4` remplacé par `numpy>=2.0` | Conflit avec `pyarrow` (qui exige NumPy 2.0 ou plus) et impossibilité d'installer NumPy 1.26.4 sans compilateur avec Python 3.14 |
| 4 | Ajout de `transactions_test.csv` | Fichier de transactions fictives pour tester l'application |
| 5 | Ajout de ce README, de `.env.example` et de `.gitignore` | Documentation, et protection de la clé API |

## Problèmes rencontrés et solutions

| Problème | Cause | Solution |
|---|---|---|
| `pip install` échoue sur NumPy (Python 3.14) | Pas de version précompilée de NumPy 1.26.4 pour Python 3.14 | Utiliser **Python 3.12** |
| `No module named 'deprecated'` | Dépendance absente de `requirements.txt` | `pip install Deprecated` |
| `pyarrow nécessite NumPy 2.0 ou plus, trouvé 1.26.4` | Conflit de versions | `pip install --upgrade numpy` |
| `404 NOT_FOUND` sur `gemini-2.5-flash` | Modèle retiré pour les nouveaux utilisateurs | Changer le nom du modèle |
| `503 UNAVAILABLE` | Modèle surchargé côté Google | Réessayer plus tard ou choisir un autre modèle |

## Installation

Prérequis : **Python 3.12** et une clé API Gemini gratuite ([Google AI Studio](https://aistudio.google.com/apikey)).

```bash
# 1. Créer et activer l'environnement virtuel (Windows)
py -3.12 -m venv venv
venv\Scripts\activate

# 2. Installer les dépendances
pip install -r requirements.txt

# 3. Configurer la clé API
copy .env.example .env
# puis ouvrir .env et remplacer your_api_key_here par votre clé

# 4. Lancer l'application
streamlit run ai_financial_coach_agent.py
```

> **Sécurité :** ne publiez jamais votre fichier `.env`. Il est exclu du dépôt par `.gitignore`.

## Choisir un modèle Gemini

Les modèles disponibles changent régulièrement. Pour lister ceux auxquels votre clé a accès :

```bash
python -c "from dotenv import load_dotenv; load_dotenv(); from google import genai; c=genai.Client(); [print(m.name) for m in c.models.list()]" | findstr flash
```

Le nom du modèle se trouve dans `ai_financial_coach_agent.py` (`model="..."`, trois occurrences, une par agent).

## Format du fichier CSV

Colonnes obligatoires : `Date` (AAAA-MM-JJ), `Category`, `Amount`.

```csv
Date,Category,Amount
2026-09-01,Housing,2500.00
2026-09-03,Food,320.50
```

Un exemple complet est fourni dans `transactions_test.csv` (21 transactions fictives).

## Limites observées

- Lors d'un test, les totaux affichés par l'agent semblaient s'écarter de ceux du fichier CSV : les chiffres produits par un modèle doivent être vérifiés. *(À confirmer par des tests plus poussés.)*
- La devise `$` est écrite en dur dans l'interface.
- Les erreurs de surcharge (503) ne sont pas gérées par des tentatives automatiques.

## Pistes d'amélioration (non implémentées)

- Calculer les totaux (dépenses, excédent) **en code Python** plutôt que par le modèle, et ne laisser à l'IA que les recommandations.
- Placer le nom du modèle dans le fichier `.env` au lieu de le répéter trois fois dans le code.
- Ajouter des **tentatives automatiques** (retry avec attente croissante) en cas d'erreur 503.
- Permettre le choix de la devise.

## Auteure des modifications

Wiam Mahboub, ingénieure d'État en Génie Industriel.
