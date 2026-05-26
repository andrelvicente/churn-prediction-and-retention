# Pipeline e Arquitetura

O projeto é organizado em **4 fases** e **14 etapas**, cobrindo desde o carregamento dos dados brutos até a geração de recomendações de retenção.

---

## Visão Geral

```
Dados Brutos → EDA → Pré-processamento → Clusterização → Enriquecimento → Predição → Recomendações
```

A integração entre as disciplinas ocorre na etapa de enriquecimento: o cluster gerado de forma não supervisionada é adicionado como atributo ao conjunto de dados antes do treinamento do modelo preditivo.

```python
# A coluna 'cluster' passa a compor o dataset de treino
kmeans = KMeans(n_clusters=k_otimo, random_state=42)
df['cluster'] = kmeans.fit_predict(X_cluster)

X_com_cluster = df.drop('Churn', axis=1)   # inclui 'cluster'
X_sem_cluster = df.drop(['Churn', 'cluster'], axis=1)
```

---

## Fase 1 — Preparação dos Dados

| Etapa | Descrição |
|---|---|
| 1. Carregamento | Leitura do CSV e inspeção inicial (shape, tipos, nulos) |
| 2. EDA | Distribuições, correlações, análise da variável alvo |
| 3. Tratamento | Remoção de inconsistências, conversão de `TotalCharges` para numérico |
| 4. Pré-processamento | Engenharia de atributos, remoção de identificadores |
| 5. Encoding | One-Hot Encoding para variáveis nominais, Label Encoding para ordinais |
| 6. Normalização | StandardScaler / MinMaxScaler para atributos numéricos |

## Fase 2 — Agrupamento de Clientes

| Etapa | Descrição |
|---|---|
| 7. Clusterização | Aplicação de K-Means, DBSCAN e Agglomerative Clustering |
| 8. Avaliação | Silhouette Score, Davies-Bouldin, Calinski-Harabasz para seleção do melhor modelo |
| 9. Interpretação | Análise dos centroides, nomeação semântica dos perfis identificados |

## Fase 3 — Inteligência Computacional

| Etapa | Descrição |
|---|---|
| 10. Enriquecimento | Adição da coluna `cluster` como atributo preditivo |
| 11. Treinamento | Random Forest, Decision Tree, Logistic Regression, MLP |
| 12. Avaliação | F1-Score, ROC-AUC, Confusion Matrix, validação cruzada estratificada |
| 13. Comparação | Experimento com vs. sem cluster para medir ganho preditivo |

## Fase 4 — Recomendações e Insights

| Etapa | Descrição |
|---|---|
| 14. Recomendações | Ações de retenção personalizadas por perfil de cluster |
| 15. Relatório | Consolidação de insights estratégicos e conclusões |

---

## Diagrama de Fluxo de Dados

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Dataset   │───▶│  EDA + Prep  │───▶│  Clusterização  │
│  ~7k linhas │    │              │    │  K-Means/DBSCAN │
└─────────────┘    └──────────────┘    └────────┬────────┘
                                                │ cluster_id
                                       ┌────────▼────────┐
                                       │  Dataset + col  │
                                       │    'cluster'    │
                                       └────────┬────────┘
                                                │
                                  ┌─────────────▼─────────────┐
                                  │     Modelos Preditivos     │
                                  │  RF / DT / LR / MLP       │
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
