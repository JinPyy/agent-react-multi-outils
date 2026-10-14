# Agent IA multi-outils (ReAct) — TER Master 1

Agent basé sur un LLM (**Qwen2.5-7B-Instruct**, quantifié 4-bit) capable d'utiliser des outils externes pour répondre à des questions qui demandent des données réelles ou du calcul. L'agent est structuré en **graphe d'états** avec un **nœud évaluateur** qui vérifie la réponse avant de la renvoyer.

> Travail d'Étude et de Recherche (TER), Master 1 Informatique, Université Paris 8 — 2025-2026.

## Architecture

```
ROUTEUR (LLM, ReAct) ──action──▶ EXECUTION_OUTIL ──observation──▶ ROUTEUR
        │
        └─ réponse candidate ──▶ EVALUATEUR (LLM juge) ──VALIDE──▶ FIN
                                        │
                                        └─INVALIDE (feedback)──▶ ROUTEUR
```

- **ROUTEUR** : le LLM produit `Thought / Action / Action Input` ou une `Final Answer`.
- **EXECUTION_OUTIL** : appelle l'outil et réinjecte une observation normalisée (nettoyée, tronquée) pour limiter la dérive du contexte.
- **EVALUATEUR** : second appel au LLM qui répond `VALIDE` ou `INVALIDE` selon que la réponse est fondée sur les observations (garde-fou contre l'hallucination).
- Le graphe est implémenté à la main en Python (sans LangGraph), pour garder le contrôle du mécanisme ReAct.

## Outils intégrés

| Outil | Fonction | Source |
|---|---|---|
| Meteo | Météo actuelle et prévisions | Open-Meteo |
| RechercheLieu | Lieux touristiques | Photon / OpenStreetMap |
| InfoPays | Capitale et population | Countries Now |
| ConvertisseurDevises | Conversion de devises | Frankfurter |
| CoursCrypto | Cours d'une cryptomonnaie | CoinGecko |
| Wikipedia | Résumé encyclopédique | API Wikipédia (fr) |
| Calculateur | Calcul arithmétique sécurisé (AST) | Local |
| DateRelative | Date courante ou décalée | Local |
| JourSemaine | Jour de la semaine d'une date | Local |

## Évaluation

Benchmark de 20 questions (9 mono-outil, 7 multi-outils, 4 sans outil attendu), agent complet comparé à la même LLM sans outils (baseline).

| Indicateur | Agent | Baseline |
|---|---|---|
| Taux de succès global | 85 % | — |
| Mono-outil / multi-outils | 100 % / 100 % | — |
| Temps de réponse moyen | 15,78 s | 7,18 s |
| Étapes ROUTEUR (moyenne) | 2,2 | 1 |

Les 3 échecs concernent la catégorie « sans outil attendu » et viennent en partie du critère binaire du benchmark (ex. délégation correcte d'un calcul au Calculateur comptée comme échec). Sur la requête chaînée *météo → lieu*, le chaînage a réussi 4 fois sur 5 (petit échantillon).

**Limites** : parsing ReAct fragile (regex), hallucination résiduelle d'un modèle 7B, dépendance à des API tierces, évaluateur et routeur portés par le même modèle.

## Lancer le projet

Le notebook est prévu pour **Google Colab avec un GPU T4** (gratuit).

1. Ouvrir `TER_agent_Yannick_JIN.ipynb` dans Google Colab.
2. Choisir *Exécution > Modifier le type d'exécution > GPU T4*.
3. Exécuter les cellules dans l'ordre (installation, chargement du modèle, outils, graphe, benchmark).

Dépendances : voir `requirements.txt`. Les API utilisées sont publiques et ne demandent pas de clé.

## Rapport

Le rapport complet du TER (état de l'art sur 5 publications, architecture, implémentation, évaluation et limites) est disponible dans [`docs/Rapport_TER.pdf`](docs/Rapport_TER.pdf).

## Auteur

Yannick Jin — Master 2 Informatique & Big Data, Université Paris 8.
