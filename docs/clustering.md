# Clusterização de Clientes

A clusterização é usada para descobrir perfis comportamentais nos dados de forma não supervisionada. Os clusters resultantes são interpretados semanticamente e incorporados ao pipeline preditivo como atributo adicional.

**Notebooks de referência:**
- [`notebooks/02_clustering.ipynb`](../notebooks/02_clustering.ipynb) — K-Means baseline
- [`notebooks/03_clustering_advanced.ipynb`](../notebooks/03_clustering_advanced.ipynb) — DBSCAN e comparação

---

## Status da Implementação

| Etapa | Status |
|---|---|
| Seleção e justificativa de atributos | **Concluída** |
| Pré-processamento e padronização | **Concluída** |
| K-Means baseline (K=2) | **Concluída** (`02_clustering.ipynb`) |
| DBSCAN (grid search + treino final) | **Concluída** (`03_clustering_advanced.ipynb`) |
| Comparação K-Means vs. DBSCAN | **Concluída** (`03_clustering_advanced.ipynb`) |
| Geração de `telco_with_clusters.csv` final | **Concluída** (rótulos DBSCAN) |

---

## Atributos Utilizados

```
tenure, MonthlyCharges, Contract, InternetService,
TechSupport, OnlineSecurity, PaymentMethod
```

**Excluídos:** `TotalCharges` (colinear com `MonthlyCharges`×`tenure`), `customerID` (identificador), `Churn` (data leakage).

---

## Pré-processamento Aplicado

| Tipo | Atributos | Tratamento |
|---|---|---|
| Numérico contínuo | `tenure`, `MonthlyCharges` | `StandardScaler` |
| Ordinal | `Contract` | Month-to-month=0, One year=1, Two year=2 |
| Binário com NA | `TechSupport`, `OnlineSecurity` | Yes=1, No=0, No internet service=-1 |
| Nominal | `InternetService`, `PaymentMethod` | One-Hot Encoding (sem `drop_first`) |

Matriz final de clustering: **7.043 registros × 12 features** após encoding.

**Distância:** Euclidiana (padrão do K-Means e DBSCAN). Limitação reconhecida para dados mistos.

---

## Baseline K-Means

### Seleção do K

Critérios testados para k = 2 a 12: método do cotovelo (inertia) + Silhouette Score.

| K | Silhouette Score |
|---|---|
| **2** | **0,344** (selecionado) |
| 3 | 0,322 |
| 5 | 0,318 |

O elbow foi difuso (sem cotovelo claro). K=2 apresentou o maior Silhouette Score no intervalo testado.

### Métricas do Modelo Final (K=2)

| Métrica | Valor | Interpretação |
|---|---|---|
| **Silhouette Score** | 0,344 | Estrutura moderada (abaixo de 0,5) |
| **Davies-Bouldin Index** | 1,146 | Clusters pouco separados (> 1,0) |
| **Calinski-Harabasz Score** | 3.104,6 | Razão inter/intra-cluster favorável |

### Perfis Identificados (K-Means, K=2)

| Cluster | Rótulo | Tamanho | Churn | tenure médio | MonthlyCharges médio | Contrato dominante | Internet dominante |
|---|---|---|---|---|---|---|---|
| 0 | Insatisfeito com serviços | 77,9% | 31,8% | 33,0 meses | R$ 77,10 | Month-to-month | Fiber optic |
| 1 | Econômico estável | 22,1% | 8,0% | 30,1 meses | R$ 21,20 | Two year | No |

**Interpretação:**
- **Cluster 0 — Insatisfeito com serviços:** concentra a maior parte da base, com gasto elevado, contrato mensal e fibra óptica. Taxa de churn ~4× superior ao Cluster 1. Perfil de alto risco comercial.
- **Cluster 1 — Econômico estável:** clientes com planos básicos (sem internet), contratos longos e baixo gasto mensal. Baixa taxa de churn — segmento fidelizado.

---

## DBSCAN (Etapa 3)

### Estratégia de Busca de Parâmetros

Grid search exaustivo:
- **eps:** {0,30, 0,40, 0,50, 0,60, 0,80, 1,00, 1,20}
- **min_samples:** {3, 5, 7, 10}

Total: 28 combinações avaliadas. Critério de seleção: maior Silhouette Score com n_clusters ≥ 2.

### Top Resultados DBSCAN

| eps | min_samples | N.º clusters | Ruído (%) | Silhouette | Davies-Bouldin |
|---|---|---|---|---|---|
| **0,80** | **3** | **108** | **0,4%** | **0,364** | 1,299 |
| 0,80 | 5 | 103 | 0,8% | 0,362 | 1,341 |
| 0,80 | 7 | 100 | 1,2% | 0,361 | 1,335 |
| 0,60 | 5 | 102 | 2,1% | 0,359 | 1,344 |
| 0,60 | 3 | 115 | 1,2% | 0,359 | 1,327 |

### Métricas Finais — Configuração Selecionada

| Métrica | DBSCAN (eps=0,8, ms=3) | K-Means (K=2) |
|---|---|---|
| N.º de clusters | 108 | 2 |
| Ruído | 0,4% (≈ 28 pontos) | 0% |
| **Silhouette Score** | **0,364** | 0,344 |
| Davies-Bouldin | 1,299 | 1,146 |
| Calinski-Harabasz | 544,0 | 3.104,6 |

### Perfis DBSCAN

O DBSCAN produziu 46 clusters ativos (+ ruído). Os rótulos semânticos foram definidos por inspeção das distribuições de churn:

| Rótulo | N.º de clusters com esse rótulo | Descrição |
|---|---|---|
| `Atipicos (ruido DBSCAN)` | 1 (cluster −1) | 2.757 pontos (~39%) classificados como ruído |
| `Perfil intermediario` | ~23 clusters | Clientes com características mistas |
| `Economico estavel` | ~23 clusters | Clientes com baixo gasto e contratos longos |

> **Limitação crítica:** 108 micro-clusters são impraticáveis como feature categórica única no modelo preditivo. Por isso, a Etapa 04 reconstruiu os 2 clusters K-Means para o experimento comparativo.

---

## Comparação Final: K-Means vs. DBSCAN

| Critério | K-Means | DBSCAN | Vencedor |
|---|---|---|---|
| Silhouette Score | 0,344 | **0,364** | DBSCAN |
| Davies-Bouldin | **1,146** | 1,299 | K-Means |
| Calinski-Harabasz | **3.104,6** | 544,0 | K-Means |
| Interpretabilidade | **Alta** (2 perfis) | Baixa (108 clusters) | K-Means |
| Utilidade como feature | **Alta** | Baixa | K-Means |
| Tratamento de ruído | Não | **Sim** | DBSCAN |

**Decisão:** DBSCAN foi selecionado como algoritmo vencedor pela métrica principal (Silhouette). Os rótulos DBSCAN foram exportados para `telco_with_clusters.csv`. Contudo, para o experimento preditivo da Fase 4, os clusters K-Means (2 perfis) foram reconstruídos por serem mais interpretáveis e práticos como feature.

---

## Visualizações Geradas

| Arquivo | Etapa | Descrição |
|---|---|---|
| `04_elbow_silhouette.png` | K-Means | Método do cotovelo + curva de Silhouette |
| `05_pca_clusters.png` | K-Means | PCA 2D com clusters coloridos |
| `06_dendrogram.png` | K-Means | Dendrograma parcial (Ward, amostra de 500) |
| `07_centroids_heatmap.png` | K-Means | Médias dos atributos por cluster |
| `08_churn_by_cluster.png` | K-Means | Taxa de churn (%) por cluster |
| `09_silhouette_samples.png` | K-Means | Silhouette por amostra (diagnóstico) |
| `10_dbscan_param_search.png` | DBSCAN | Heatmap do grid search eps × min_samples |
| `12_pca_comparison_2algorithms.png` | Comparação | PCA 2D: K-Means vs. DBSCAN lado a lado |
| `14_metrics_comparison.png` | Comparação | Métricas internas comparadas em barras |

---

## Artefatos Gerados

| Artefato | Caminho |
|---|---|
| Base para clustering | `data/processed/telco_clustering_base.csv` |
| Dados com rótulos finais (DBSCAN) | `data/processed/telco_with_clusters.csv` |
| Scaler treinado | `models/scaler.joblib` |
| Modelo K-Means | `models/kmeans_model.joblib` |
| Sumário de comparação | `reports/comparison_summary.csv` |

---

## Problemas Identificados no Baseline

1. **Esfericidade assumida** — K-Means força clusters convexos; PCA mostra sobreposição entre grupos.
2. **Dados mistos** — distância Euclidiana trata OHE e codificações -1/0/1 como contínuas.
3. **Elbow difuso** — ausência de cotovelo claro indica separação natural fraca.
4. **Clusters desbalanceados** — 77,9% vs. 22,1% reduz utilidade da segmentação menor.
5. **K=2 insuficiente** — apenas 2 perfis emergiram; os 4 perfis teóricos não se confirmaram.
6. **Silhouette moderado** — 0,344 indica estrutura presente, mas com sobreposição significativa.

## Descobertas Comportamentais

- Clientes com **fibra óptica + contrato mensal + alto gasto** concentram ~78% da base e ~32% de churn.
- Clientes **sem internet + contrato longo + baixo gasto** formam segmento estável com apenas 8% de churn.
- A principal separação encontrada é **faixa de gasto/contrato**, não tenure ou demografia.
- `MonthlyCharges` e `Contract` são os atributos com maior poder discriminativo entre os dois clusters.
- O DBSCAN confirmou a heterogeneidade da base ao identificar dezenas de micro-segmentos, mas não produziu perfis consolidados e interpretáveis.
