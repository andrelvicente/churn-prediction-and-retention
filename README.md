<div align="center">

# Churn Prediction & Retention

**Integrantes**

[André Luiz Vicente Silva](https://github.com/andrelvicente) · [Fernando Santos de Almeida](https://github.com/Fernando-alme) · [João Henrique Batista Junior](https://github.com/whoiamrootuser)

**Predição de cancelamento e recomendação de retenção de clientes com clustering + ML**

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License](https://img.shields.io/badge/Licença-MIT-green?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow?style=flat-square)]()

</div>

---

Projeto integrador das disciplinas de **Agrupamento de Dados** e **Inteligência Computacional**.

**Dataset:** [Telco Customer Churn 📡](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data) (~7k clientes, Kaggle)

---

## Pipeline

```
Dados → EDA → Pré-processamento → Clusterização → Predição de Churn → Recomendações
```

1. Análise exploratória e tratamento dos dados
2. Clusterização de clientes (K-Means, DBSCAN, Agglomerative)
3. Cluster como atributo adicional no modelo preditivo
4. Treinamento e comparação dos modelos (Random Forest, Decision Tree, Logistic Regression, MLP)
5. Recomendações de retenção personalizadas por perfil

---

## Instalação

```bash
git clone https://github.com/<seu-usuario>/churn-prediction-and-retention.git
cd churn-prediction-and-retention
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Baixe o dataset em [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) e salve em `data/raw/`.

## Execução

Execute os notebooks na ordem numérica em `notebooks/`:

```
01_eda → 02_preprocessing → 03_clustering → 04_churn_prediction → 05_recommendations
```

---

## Estrutura

```
data/          # Dataset bruto e processado
notebooks/     # Pipeline interativo (Jupyter)
src/           # Módulos Python reutilizáveis
models/        # Modelos treinados serializados
reports/       # Gráficos e relatório final
docs/          # Documentação detalhada do projeto
```

---

## Documentação

| Documento | Descrição |
|---|---|
| [Pipeline e Arquitetura](docs/pipeline.md) | Detalhamento das 14 etapas do projeto |
| [Dataset](docs/dataset.md) | Dicionário de dados e análise dos atributos |
| [Clusterização](docs/clustering.md) | Técnicas, métricas e interpretação dos clusters |
| [Modelos Preditivos](docs/models.md) | Algoritmos, métricas e experimento comparativo |
| [Retenção](docs/retention.md) | Lógica de recomendação por perfil de cluster |
| [Roadmap](docs/roadmap.md) | Status das fases, desafios e melhorias futuras |

---

## Tecnologias

Python · Pandas · NumPy · Scikit-learn · Matplotlib · Seaborn · Jupyter

---

## Licença

[MIT](LICENSE) — Projeto acadêmico, 2026.
