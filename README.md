# fleet-api

Mini-service de supervision d'une flotte de robots d'entrepôt.

Ce dépôt est le **fil rouge** du module *Usine Logicielle et CI/CD* (ESIA A3). Vous
allez le faire grossir séance après séance jusqu'à disposer d'une chaîne
d'intégration et de déploiement complète.

---

## Ce que fait le service

Les robots d'un entrepôt émettent régulièrement leur télémétrie : tension
batterie, position, état de charge. `fleet-api` la reçoit et en calcule des
indicateurs — niveau de charge, autonomie restante, distance parcourue, alertes
batterie, agrégats de flotte.

Le cœur métier vit dans `src/fleet_api/telemetry.py` : une dizaine de
**fonctions pures**, sans état ni entrée-sortie, donc directement testables.

## Démarrage

Prérequis : [uv](https://docs.astral.sh/uv/). Inutile d'installer Python vous-même :
uv télécharge la version voulue (≥ 3.12) au premier `uv sync`.

```bash
uv sync                 # installe les dépendances
uv run pytest -v        # lance la suite de tests
```

Vous devez obtenir **6 tests au vert**. Si ce n'est pas le cas, signalez-le
avant d'aller plus loin.

## Structure

```
src/fleet_api/
├── models.py       Position, Reading, RobotState
├── telemetry.py    les fonctions de calcul  ← l'objet du TD 1
├── store.py        stockage en mémoire ou PostgreSQL  (séance 5)
└── api.py          les endpoints HTTP                 (séance 5)
tests/
└── test_telemetry.py   3 tests d'exemple, le reste est à écrire
```

## Règles du jeu

1. **Les docstrings font foi.** Elles sont la spécification. Quand le code et la
   docstring divergent, c'est le code qui a tort. Ne modifiez jamais une
   docstring pour la faire coller à l'implémentation.
2. **On travaille par pull request.** À partir de la séance 2, `main` est
   protégée : plus de push direct.
3. **Un commit, une intention.** Le message dit *pourquoi*, pas *quoi* — le diff
   dit déjà quoi.

## Backlog

Évolutions possibles pour le rendu final, par ordre de difficulté croissante.
Vous n'avez pas à toutes les traiter : mieux vaut deux fonctionnalités bien
testées et bien intégrées que six bâclées.

- [ ] `GET /robots/{id}/history` — historique de télémétrie d'un robot
- [ ] Alerte sur immobilité prolongée (aucun déplacement depuis N minutes)
- [ ] `GET /fleet/heatmap` — densité de présence par zone de l'entrepôt
- [ ] Estimation du temps de charge restant
- [ ] Détection de dérive de calibration entre robots d'un même modèle
- [ ] Export Prometheus des indicateurs de flotte

## Progression du module

| Séance | Ce que vous ajoutez |
|---|---|
| 1 | Les tests manquants, un premier workflow |
| 2 | Pipeline lint / test / build, protection de `main` |
| 3 | Workflow réutilisable, pre-commit, Dependabot |
| 4 | Dockerfile multi-stage, docker compose |
| 5 | Build et publication d'image sur GHCR, scan de vulnérabilités |
| 6 | Couverture, typage, analyse statique, quality gate |
| 7 | Release versionnée, environnements, bascule et retour arrière |
| 8 | Revue croisée, finalisation |

## TD 2

### Durées

| Run | Durée totale |
|---|---|
| Cache froid | 20 s |
| Cache chaud | 17 s |

Détail du run cache chaud :

| Tâche | Durée | Installer uv | uv sync | Commande |
|---|---|---|---|---|
| lint | 10 s | 4 s | - | 0 s |
| test (3.12) | 8 s | 2 s | 0 s | 2 s |
| test (3.13) | 11 s | 2 s | 2 s | 1 s |
| test (3.14) | 11 s | 2 s | 2 s | 1 s |
| build | 10 s | 3 s | - | 1 s |

### Pourquoi la matrice sur test et pas sur lint ?

ruff analyse le code sans l'exécuter, son résultat ne dépend pas de la version de
Python qui le lance. Le lancer trois fois coûterait trois fois plus pour le même
résultat. Les tests, eux, exécutent le code, donc le comportement peut changer
d'une version à l'autre.

### Où passe le temps ?

Presque pas dans nos commandes : ruff, pytest et uv build prennent 1 à 2 s. Le
reste c'est le prix fixe de chaque tâche : démarrage de la machine, checkout,
installation de uv et des dépendances. Le cache ne gagne que quelques secondes
parce que le projet a peu de dépendances.

### Qu'est-ce qu'on ferait en premier pour accélérer ?

Ne pas découper plus : chaque tâche paie ce prix fixe. On pourrait même regrouper
lint et build dans une seule tâche. Ensuite limiter la matrice aux versions
qu'on supporte vraiment.
