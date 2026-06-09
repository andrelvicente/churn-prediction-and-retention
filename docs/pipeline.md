# Pipeline e Arquitetura

O projeto é organizado em **4 fases**, cobrindo desde o carregamento dos dados brutos até a geração de recomendações de retenção.

---

## Visão Geral

```
Dados Brutos → EDA → Pré-processamento → Clusterização → Enriquecimento → Predição → Recomendações
```

A integração entre as disciplinas ocorre na etapa de enriquecimento: o cluster gerado de forma não supervisionada é adicionado como atributo ao conjunto de dados antes do treinamento do modelo preditivo.

```python
# Implementado em 02_clustering.ipynb
kmeans = KMeans(n_clusters=2, random_state=42, n_init=10)
df['cluster'] = kmeans.fit_predict(X_cluster)
df['cluster_label'] = df['cluster'].map({0: 'Insatisfeito com serviços', 1: 'Econômico estável'})

# Próxima fase (04_churn_prediction.ipynb)
X_com_cluster = df.drop('Churn', axis=1)   # inclui 'cluster' e 'cluster_label'
X_sem_cluster = df.drop(['Churn', 'cluster', 'cluster_label'], axis=1)
```

---

## Fase 1 — Preparação dos Dados

| Etapa | Descrição | Status | Notebook |
|---|---|---|---|
| 1. Carregamento | Leitura do CSV e inspeção inicial (shape, tipos, nulos) | Concluída | `01_eda` |
| 2. EDA | Distribuições, correlações, análise da variável alvo | Concluída | `01_eda` |
| 3. Tratamento | Conversão de `TotalCharges`, remoção de duplicatas | Concluída | `01_eda` |
| 4. Pré-processamento | Engenharia de atributos, remoção de identificadores | Concluída | `01_eda` |
| 5. Encoding inicial | Binários, ordinais e labels derivados | Concluída | `01_eda` |
| 6. Exportação | `telco_clean.csv` (7.043 × 35 colunas) | Concluída | `01_eda` |

**Saída:** `data/processed/telco_clean.csv`

---

## Fase 2 — Agrupamento de Clientes

| Etapa | Descrição | Status | Notebook |
|---|---|---|---|
| 7. Seleção de atributos | 7 features comportamentais/financeiras | Concluída | `02_clustering` |
| 8. Pré-processamento | Encoding + StandardScaler seletivo → 12 features | Concluída | `02_clustering` |
| 9. Seleção do K | Elbow + Silhouette (k=2..12) → K=2 | Concluída | `02_clustering` |
| 10. K-Means baseline | Treinamento + 3 métricas internas | Concluída | `02_clustering` |
| 11. Visualizações | PCA, dendrograma, heatmap, churn, silhouette | Concluída | `02_clustering` |
| 12. Interpretação | 2 perfis nomeados semanticamente | Concluída | `02_clustering` |
| 13. DBSCAN | Clusterização por densidade | Pendente | `03_clustering_advanced` |
| 14. Agglomerative | Clusterização hierárquica | Pendente | `03_clustering_advanced` |
| 15. Comparação | Seleção do melhor algoritmo | Pendente | `03_clustering_advanced` |

**Saídas:**
- `data/processed/telco_clustering_base.csv` — subconjunto de 7 atributos + alvo
- `data/processed/telco_with_clusters.csv` — base completa + `cluster`, `cluster_label`
- `models/scaler.joblib`, `models/kmeans_model.joblib`
- `reports/figures/04_*.png` a `09_*.png`

---

## Fase 3 — Inteligência Computacional

| Etapa | Descrição | Status |
|---|---|---|
| 16. Enriquecimento | Adição da coluna `cluster` como atributo preditivo | Pendente |
| 17. Treinamento | Random Forest, Decision Tree, Logistic Regression, MLP | Pendente |
| 18. Avaliação | F1-Score, ROC-AUC, Confusion Matrix, validação cruzada | Pendente |
| 19. Comparação | Experimento com vs. sem cluster para medir ganho preditivo | Pendente |

---

## Fase 4 — Recomendações e Insights

| Etapa | Descrição | Status |
|---|---|---|
| 20. Recomendações | Ações de retenção personalizadas por perfil de cluster | Pendente |
| 21. Relatório | Consolidação de insights estratégicos e conclusões | Pendente |

---

## Diagrama de Fluxo de Dados

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Dataset   │───▶│  EDA + Prep  │───▶│  Clusterização  │
│  ~7k linhas │    │  (01_eda)    │    │  K-Means K=2    │
└─────────────┘    └──────────────┘    │  (02_clustering)│
                                       └────────┬────────┘
                                                │ cluster + cluster_label
                                       ┌────────▼────────┐
                                       │ telco_with_     │
                                       │ clusters.csv    │
                                       └────────┬────────┘
                                                │
                                  ┌─────────────▼─────────────┐
                                  │     Modelos Preditivos     │
                                  │  RF / DT / LR / MLP       │
                                  │  (04_churn_prediction)    │
                                  └─────────────┬─────────────┘
                                                │
                                  ┌─────────────▼─────────────┐
                                  │  Predição + Perfil do     │
                                  │  cliente identificado     │
                                  └─────────────┬─────────────┘
                                                │
                                  ┌─────────────▼─────────────┐
                                  │  Recomendação personalizada│
                                  │  de retenção por cluster  │
                                  └───────────────────────────┘
```

---

## Pré-processamento para Clustering (implementado)

```python
# Ordinal
contract_map = {'Month-to-month': 0, 'One year': 1, 'Two year': 2}

# Binário com NA semântico
binary_map = {'Yes': 1, 'No': 0, 'No internet service': -1}

# Nominal — OHE sem drop_first
df = pd.get_dummies(df, columns=['InternetService', 'PaymentMethod'])

# Normalização APENAS nas numéricas contínuas
scaler = StandardScaler()
df[['tenure_scaled', 'MonthlyCharges_scaled']] = scaler.fit_transform(
    df[['tenure', 'MonthlyCharges']]
)
# Resultado: matriz 7043 × 12 features
```

**Distância:** Euclidiana. **Outliers:** mantidos (valores plausíveis, não erros de coleta).

---

## Resultados do Baseline (K-Means, K=2)

| Métrica | Valor |
|---|---|
| Silhouette Score | 0,344 |
| Davies-Bouldin Index | 1,146 |
| Calinski-Harabasz Score | 3.104,6 |

| Cluster | Rótulo | Tamanho | Churn |
|---|---|---|---|
| 0 | Insatisfeito com serviços | 77,9% | 31,8% |
| 1 | Econômico estável | 22,1% | 8,0% |
