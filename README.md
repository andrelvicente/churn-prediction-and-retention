<div align="center">

# Churn Prediction & Retention

**Integrantes**

[André Luiz Vicente Silva](https://github.com/andrelvicente) · [Fernando Santos de Almeida](https://github.com/Fernando-alme) · [João Henrique Batista Junior](https://github.com/whoiamrootuser)

**Professores Orientadores**

[Gustavo Prado Oliveira](https://github.com/gpradooliv) · [Clarimundo](https://github.com/gpradooliv)

**Predição de cancelamento e recomendação de retenção de clientes com clustering + ML**

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License](https://img.shields.io/badge/Licença-MIT-green?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow?style=flat-square)]()

</div>

---

Projeto integrador das disciplinas de **Agrupamento de Dados** e **Inteligência Computacional**.

**Dataset:** [Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data) (~7k clientes, Kaggle)

---

## Pipeline

```
Dados → EDA → Clusterização (baseline) → Predição de Churn → Recomendações
```

1. Análise exploratória e tratamento dos dados
2. Clusterização baseline de clientes (K-Means; DBSCAN e Agglomerative na Semana 3)
3. Cluster como atributo adicional no modelo preditivo
4. Treinamento e comparação dos modelos (Random Forest, Decision Tree, Logistic Regression, MLP)
5. Recomendações de retenção personalizadas por perfil

---

## Status atual

| Semana | Entrega | Status |
|---|---|---|
| 1 | EDA e base limpa | Concluída (`01_eda.ipynb`) |
| 2 | Agrupamento baseline (K-Means) | Concluída (`02_clustering.ipynb`) |
| 3 | Comparação de algoritmos de cluster | Pendente |
| 4 | Predição de churn | Pendente |
| 5 | Recomendações de retenção | Pendente |

**Descoberta principal (Semana 2):** o K-Means com K=2 separou dois perfis distintos — clientes de **alto gasto e alto churn** (77,9%) vs. clientes **econômicos e estáveis** (22,1%). Silhouette Score = 0,344.

---

## Instalação

```bash
git clone https://github.com/andrelvicente/churn-prediction-and-retention.git
cd churn-prediction-and-retention
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Baixe o dataset em [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) e salve em `data/raw/`.

## Execução

Execute os notebooks na ordem numérica em `notebooks/`:

```
01_eda → 02_clustering → 03_clustering_advanced → 04_churn_prediction → 05_recommendations
```

Notebooks implementados até o momento: `01_eda`, `02_clustering`.

---

## Estrutura

```
data/
  raw/              # Dataset original (Telco-Customer-Churn.csv)
  processed/        # telco_clean.csv, telco_clustering_base.csv, telco_with_clusters.csv
notebooks/          # Pipeline interativo (Jupyter)
models/             # scaler.joblib, kmeans_model.joblib
reports/figures/    # Gráficos EDA (01–03) e clustering (04–09)
docs/               # Documentação detalhada do projeto
src/                # Módulos Python reutilizáveis (futuro)
```

---

## Documentação

| Documento | Descrição |
|---|---|
| [Pipeline e Arquitetura](docs/pipeline.md) | Detalhamento das etapas do projeto |
| [Dataset](docs/dataset.md) | Dicionário de dados e atributos de clusterização |
| [Clusterização](docs/clustering.md) | Baseline K-Means, métricas e perfis identificados |
| [Modelos Preditivos](docs/models.md) | Algoritmos, métricas e experimento comparativo |
| [Retenção](docs/retention.md) | Lógica de recomendação por perfil de cluster |
| [Roadmap](docs/roadmap.md) | Status das fases, desafios e melhorias futuras |

---

## Tecnologias

Python · Pandas · NumPy · Scikit-learn · Matplotlib · Seaborn · Jupyter · SciPy

---

## Licença

[MIT](LICENSE) — Projeto acadêmico, 2026. 
