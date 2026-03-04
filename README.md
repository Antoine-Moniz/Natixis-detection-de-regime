# Détection de Régime & Allocation d'Actifs – Projet Natixis 2026

> **M2 Quantitative Finance** — Défi Natixis  
> Outil probabiliste de détection de cycle économique (Markov + Kalman) appliqué à l'allocation stratégique d'actifs.

---

## Objectif

Construire un **indicateur quantitatif de cycle** capable de détecter le régime macro-économique courant (récession, reprise, boom, ralentissement), puis l'utiliser pour :

1. Élaborer un **scénario macro-financier** prospectif (2026).

Les données proviennent d'un export Bloomberg (Excel), sont consolidées avec `pandas`, puis exploitées dans des notebooks interactifs et un rapport Markdown.

---

## Structure du dépôt

```
detection-de-regime/
│
├── data/
│   └── processed/
│       ├── macro_panel.parquet        # Panel consolidé (18 séries macro)
│       ├── macro_panel.csv            # Idem au format CSV
│       ├── macro_metadata.csv         # Métadonnées (pilier, fréquence, source)
│       ├── recession_prob.csv         # Probabilités baseline (Markov/Kalman/blend)
│       └── scenarios/
│           └── scenario_us_hard_landing.csv
│
├── notebooks/
│   ├── macro_dashboard.ipynb          # Exploration & contrôle des séries importées
│   ├── recession_indicator.ipynb      # Visualisation des probabilités de cycle
│   └── scenario_dashboard.ipynb       # Dashboard interactif : chocs & simulations
│
├── reports/
│   └── projet_allocation_2026.md      # Rapport complet (blocs 1-2)
│
├── src/
│   ├── ingest_bloomberg.py            # Ingestion Excel → parquet/CSV
│   ├── recession_indicator.py         # Modèle Markov 4 régimes + Kalman
│   └── run_scenario.py               # Application de chocs prospectifs (YAML)
│
├── requirements.txt
└── README.md
```

---

## Méthodologie

### 1. Modèle de Markov à 4 régimes (HMM)

Un modèle de Markov caché est estimé par **algorithme EM** (forward-backward) sur la croissance trimestrielle du PIB US (`us_gdp_qoq`). Les 4 régimes sont ordonnés par moyenne croissante :

| Régime | Interprétation |
|---|---|
| **Récession** | Contraction (moyenne de croissance la plus basse) |
| **Reprise** | Sortie de crise, croissance en accélération |
| **Boom** | Expansion soutenue |
| **Ralentissement** | Croissance décélère, risque de basculement |

Le modèle produit des **probabilités lissées** d'appartenance à chaque régime à chaque date.
La probabilité de stress Markov est définie comme :

> **P_markov = P(récession) + 0,5 × P(ralentissement)**

### 2. Filtre de Kalman sur indicateurs avancés

Un filtre de Kalman scalaire (marche aléatoire) est appliqué à un **signal composite** construit à partir de 4 indicateurs avancés US, chacun normalisé en z-score :

| Indicateur | Rôle |
|---|---|
| `us_ism_manufacturing` | Activité manufacturière |
| `us_michigan_sentiment` | Confiance des consommateurs |
| `us_initial_claims` | Demandes d'allocations chômage (inversé) |
| `us_nfp_change` | Créations d'emplois non agricoles |

L'état latent lissé est transformé en probabilité via une **fonction logistique** (état négatif → stress élevé).

### 3. Probabilité blendée

Les deux signaux sont fusionnés avec un terme d'interaction :

> **P_blend = clip( 0,6 · P_markov + 0,4 · P_kalman + P_markov · P_kalman , 0, 1 )**

Le terme multiplicatif amplifie le signal lorsque **les deux approches convergent** vers le même diagnostic.

---

## Données

**Source** : Bloomberg via l'export Excel `Indices_Bloomberg_Macro données.xlsx`.

18 séries macro couvrant 5 piliers :

| Pilier | Exemples d'indicateurs |
|---|---|
| **Croissance** | PIB US QoQ, PIB Japon QoQ |
| **Entreprises** | ISM Manufacturing, IFO, ZEW, PMI France, Confiance Italie |
| **Consommateurs / Emploi** | Michigan Sentiment, NFP, Initial Claims, Chômage Japon |
| **Inflation** | CPI US MoM, CPI France YoY, HICP Zone Euro |
| **Politique monétaire** | Fed Funds (borne haute), Taux refi BCE |

Chaque feuille Excel contient des dates en colonne A et des valeurs en colonne B (à partir de la ligne 3).

---

## Rapport d'allocation

Le rapport complet se trouve dans `reports/projet_allocation_2026.md` et couvre :

| Bloc | Contenu |
|---|---|
| **Bloc 1** | Scénario économique narratif (reprise modérée US 2026) |
| **Bloc 2** | Traduction financière : lien macro → prix/taux pour chaque classe d'actifs |

## Risques & limites

1. **Données** : le modèle s'appuie sur des séries historiques US ; la transposition directe à d'autres géographies nécessite un recalibrage.
2. **Markov HMM** : le nombre de régimes (4) est fixé a priori ; un test BIC/AIC pourrait affiner ce choix.
3. **Kalman** : les hyperparamètres (q = 0,02, r = 0,15) sont calibrés manuellement ; une optimisation par maximum de vraisemblance serait envisageable.

---

#