# Pipeline e Arquitetura

O projeto é organizado em **5 fases**, cobrindo desde o carregamento dos dados brutos até a geração de recomendações de retenção.

---

## Visão Geral

```
Dados Brutos → EDA → Pré-processamento → Clusterização → Enriquecimento → Predição → Recomendações
```

A integração entre as disciplinas ocorre na etapa de enriquecimento: o cluster gerado de forma não supervisionada é adicionado como atributo ao conjunto de dados antes do treinamento do modelo preditivo.

---

## Fase 1 — Preparação dos Dados

| Etapa | Descrição | Status | Notebook |
|---|---|---|---|
| 1. Carregamento | Leitura do CSV e inspeção inicial (shape, tipos, nulos) | **Concluída** | `01_eda` |
| 2. EDA | Distribuições, correlações, análise da variável alvo | **Concluída** | `01_eda` |
| 3. Tratamento | Conversão de `TotalCharges`, remoção de duplicatas | **Concluída** | `01_eda` |
| 4. Pré-processamento | Engenharia de atributos, remoção de identificadores | **Concluída** | `01_eda` |
| 5. Encoding inicial | Binários, ordinais e labels derivados | **Concluída** | `01_eda` |
| 6. Exportação | `telco_clean.csv` (7.043 × 35 colunas) | **Concluída** | `01_eda` |

**Saída:** `data/processed/telco_clean.csv`

---

## Fase 2 — Agrupamento de Clientes

| Etapa | Descrição | Status | Notebook |
|---|---|---|---|
| 7. Seleção de atributos | 7 features comportamentais/financeiras | **Concluída** | `02_clustering` |
| 8. Pré-processamento | Encoding + StandardScaler seletivo → 12 features | **Concluída** | `02_clustering` |
| 9. Seleção do K | Elbow + Silhouette (k=2..12) → K=2 | **Concluída** | `02_clustering` |
| 10. K-Means baseline | Treinamento + 3 métricas internas | **Concluída** | `02_clustering` |
| 11. Visualizações | PCA, dendrograma, heatmap, churn, silhouette | **Concluída** | `02_clustering` |
| 12. Interpretação | 2 perfis nomeados semanticamente | **Concluída** | `02_clustering` |
| 13. DBSCAN | Grid search eps × min_samples; melhor: eps=0,8, min_samples=3 | **Concluída** | `03_clustering_advanced` |
| 14. Comparação | K-Means vs. DBSCAN → DBSCAN vencedor por Silhouette | **Concluída** | `03_clustering_advanced` |
| 15. Exportação final | `telco_with_clusters.csv` com rótulos DBSCAN | **Concluída** | `03_clustering_advanced` |

**Saídas:**
- `data/processed/telco_clustering_base.csv` — subconjunto de 7 atributos + alvo
- `data/processed/telco_with_clusters.csv` — base completa + `cluster`, `cluster_label`, `clustering_algorithm` (DBSCAN)
- `models/scaler.joblib`, `models/kmeans_model.joblib`
- `reports/figures/04_*.png` a `09_*.png` (K-Means), `10_dbscan_param_search.png`, `12_pca_comparison_2algorithms.png`, `14_metrics_comparison.png` (DBSCAN)

---

## Fase 3 — Inteligência Computacional

| Etapa | Descrição | Status | Notebook |
|---|---|---|---|
| 16. Enriquecimento | Reconstrução dos clusters K-Means como feature categórica `kmeans_cluster` | **Concluída** | `04_churn_prediction` |
| 17. Treinamento | Logistic Regression, Decision Tree, Random Forest, MLP | **Concluída** | `04_churn_prediction` |
| 18. Avaliação | F1-Score, ROC-AUC, Confusion Matrix, CV 5-fold com SMOTE | **Concluída** | `04_churn_prediction` |
| 19. Comparação | Experimento SEM vs. COM cluster → ganho marginal (ΔF1 ≤ +0,006) | **Concluída** | `04_churn_prediction` |
| 20. Persistência | `churn_lr_baseline.joblib`, `scaler_pred.joblib` | **Concluída** | `04_churn_prediction` |

**Saídas:**
- `models/churn_lr_baseline.joblib` — Logistic Regression (F1 CV = 0,629)
- `models/scaler_pred.joblib` — StandardScaler ajustado no conjunto de treino
- `reports/comparison_summary.csv` — tabela comparativa F1/AUC por modelo
- `reports/figures/10_confusion_matrices_sem_cluster.png` a `15_feature_importance_com_cluster.png`

---

## Fase 4 — Recomendações e Insights

| Etapa | Descrição | Status |
|---|---|---|
| 21. Recomendações | Ações de retenção personalizadas por perfil de cluster | **Pendente** (`05_recommendations.ipynb`) |
| 22. Relatório | Consolidação de insights estratégicos e conclusões | **Pendente** |

---

## Diagrama de Fluxo de Dados

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────────────────────┐
│   Dataset   │───▶│  EDA + Prep  │───▶│  Clusterização                  │
│  ~7k linhas │    │  (01_eda)    │    │  K-Means K=2 (02_clustering)    │
└─────────────┘    └──────────────┘    │  DBSCAN eps=0,8 (03_clustering) │
                                       └────────────────┬────────────────┘
                                                        │ telco_with_clusters.csv
                                                        │ (rótulos DBSCAN)
                                       ┌────────────────▼────────────────┐
                                       │ Fase 4: reconstrução K-Means    │
                                       │ → kmeans_cluster como feature   │
                                       └────────────────┬────────────────┘
                                                        │
                                       ┌────────────────▼────────────────┐
                                       │ Modelos Preditivos               │
                                       │ LR / DT / RF / MLP              │
                                       │ (04_churn_prediction)           │
                                       │ Melhor: Logistic Regression     │
                                       │ F1 CV=0,629 | AUC CV=0,844      │
                                       └────────────────┬────────────────┘
                                                        │
                                       ┌────────────────▼────────────────┐
                                       │  Recomendação personalizada     │
                                       │  de retenção por cluster        │
                                       │  (05_recommendations — pendente)│
                                       └─────────────────────────────────┘
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

---

## Resultados DBSCAN (Etapa 3)

| Métrica | K-Means | DBSCAN (eps=0,8, min_samples=3) |
|---|---|---|
| N.º de clusters | 2 | 108 |
| Ruído (%) | 0% | 0,4% |
| Silhouette Score | 0,344 | **0,364** |
| Davies-Bouldin | 1,146 | 1,299 |
| Calinski-Harabasz | 3.104,6 | 544,0 |

> O DBSCAN superou o K-Means em Silhouette, mas produziu 108 micro-clusters — estrutura granular demais para uso como feature categórica no modelo preditivo. A Fase 4 reconstruiu os 2 clusters K-Means para o experimento comparativo.

---

## Resultados Preditivos (Etapa 4)

| Modelo | F1 CV (5-fold) | AUC CV | F1 Teste | AUC Teste |
|---|---|---|---|---|
| **Logistic Regression** | **0,629** | **0,844** | **0,613** | **0,840** |
| Decision Tree | 0,577 | 0,813 | 0,583 | 0,809 |
| Random Forest | 0,558 | 0,824 | 0,566 | 0,824 |
| MLP | 0,569 | 0,808 | 0,600 | 0,821 |

Adição do cluster K-Means como feature: ganho marginal (ΔF1 máximo = +0,006 no Random Forest).
