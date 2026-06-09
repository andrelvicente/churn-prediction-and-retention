# Clusterização de Clientes

A clusterização é usada para descobrir perfis comportamentais nos dados de forma não supervisionada. Os clusters resultantes são interpretados semanticamente e incorporados ao pipeline preditivo como atributo adicional.

**Notebook de referência:** [`notebooks/02_clustering.ipynb`](../notebooks/02_clustering.ipynb)

---

## Status da Implementação

| Etapa | Status |
|---|---|
| Seleção e justificativa de atributos | Concluída |
| Pré-processamento e padronização | Concluída |
| K-Means baseline (Semana 2) | Concluída |
| DBSCAN | Pendente (Semana 3) |
| Agglomerative Clustering | Pendente (Semana 3) |
| Comparação entre algoritmos | Pendente (Semana 3) |

---

## Atributos Utilizados

```
tenure, MonthlyCharges, Contract, InternetService,
TechSupport, OnlineSecurity, PaymentMethod
```

**Excluídos:** `TotalCharges` (colinear), `customerID` (identificador), `Churn` (data leakage).

---

## Pré-processamento Aplicado

| Tipo | Atributos | Tratamento |
|---|---|---|
| Numérico contínuo | `tenure`, `MonthlyCharges` | `StandardScaler` |
| Ordinal | `Contract` | Month-to-month=0, One year=1, Two year=2 |
| Binário com NA | `TechSupport`, `OnlineSecurity` | Yes=1, No=0, No internet service=-1 |
| Nominal | `InternetService`, `PaymentMethod` | One-Hot Encoding (sem `drop_first`) |

Matriz final de clustering: **7.043 registros × 12 features** após encoding.

**Distância:** Euclidiana (padrão do K-Means). Limitação reconhecida para dados mistos — alternativa futura: Distância de Gower ou K-Prototypes.

---

## Baseline K-Means (Semana 2)

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

---

## Perfis Identificados

| Cluster | Rótulo | Tamanho | Churn | tenure médio | MonthlyCharges médio | Contrato dominante | Internet dominante |
|---|---|---|---|---|---|---|---|
| 0 | Insatisfeito com serviços | 77,9% | 31,8% | 33,0 meses | R$ 77,10 | Month-to-month | Fiber optic |
| 1 | Econômico estável | 22,1% | 8,0% | 30,1 meses | R$ 21,20 | Two year | No |

### Interpretação

- **Cluster 0 — Insatisfeito com serviços:** concentra a maior parte da base, com gasto elevado, contrato mensal e fibra óptica. Taxa de churn ~4× superior ao Cluster 1. Perfil de alto risco comercial.
- **Cluster 1 — Econômico estável:** clientes com planos básicos (sem internet), contratos longos e baixo gasto mensal. Baixa taxa de churn — segmento fidelizado.

> Os perfis A (Fidelizado de alto valor) e B (Novo em risco) definidos inicialmente **não emergiram** com K=2. Segmentações mais granulares serão exploradas na Semana 3 com DBSCAN e Agglomerative.

---

## Visualizações Geradas

| Arquivo | Descrição |
|---|---|
| `04_elbow_silhouette.png` | Método do cotovelo + curva de Silhouette |
| `05_pca_clusters.png` | PCA 2D com clusters coloridos |
| `06_dendrogram.png` | Dendrograma parcial (Ward, amostra de 500) |
| `07_centroids_heatmap.png` | Médias dos atributos por cluster |
| `08_churn_by_cluster.png` | Taxa de churn (%) por cluster |
| `09_silhouette_samples.png` | Silhouette por amostra (diagnóstico) |

---

## Problemas Identificados no Baseline

1. **Esfericidade assumida** — K-Means força clusters convexos; PCA mostra sobreposição entre grupos.
2. **Dados mistos** — distância Euclidiana trata OHE e codificações -1/0/1 como contínuas.
3. **Elbow difuso** — ausência de cotovelo claro indica separação natural fraca.
4. **Clusters desbalanceados** — 77,9% vs. 22,1% reduz utilidade da segmentação menor.
5. **K=2 insuficiente** — apenas 2 perfis emergiram; os 4 perfis teóricos exigem K maior ou outro algoritmo.
6. **Silhouette moderado** — 0,344 indica estrutura presente, mas com sobreposição significativa.
7. **Multicolinearidade residual** — `MonthlyCharges` correlacionado com quantidade de serviços contratados.

---

## Algoritmos Planejados (Semana 3)

### DBSCAN
Baseado em densidade. Identifica clusters de formato arbitrário e trata outliers nativamente.

- **Parâmetros críticos:** `eps` (raio de vizinhança) e `min_samples`
- **Vantagem:** não exige número de clusters pré-definido
- **Limitação:** sensível à escolha de parâmetros em alta dimensionalidade

### Agglomerative Clustering (Hierárquico)
Abordagem bottom-up que agrupa progressivamente os pontos mais similares.

- **Visualização:** dendrograma completo para análise da estrutura hierárquica
- **Linkage testados:** Ward, Complete, Average
- **Vantagem:** permite análise em múltiplos níveis de granularidade

---

## Artefatos Gerados

| Artefato | Caminho |
|---|---|
| Base para clustering | `data/processed/telco_clustering_base.csv` |
| Dados com rótulos | `data/processed/telco_with_clusters.csv` |
| Scaler treinado | `models/scaler.joblib` |
| Modelo K-Means | `models/kmeans_model.joblib` |

Colunas adicionadas em `telco_with_clusters.csv`: `cluster`, `cluster_label`, `clustering_algorithm`.

---

## Descobertas Comportamentais

Com base no baseline K-Means (K=2):

- Clientes com **fibra óptica + contrato mensal + alto gasto** concentram ~78% da base e ~32% de churn.
- Clientes **sem internet + contrato longo + baixo gasto** formam segmento estável com apenas 8% de churn.
- A principal separação encontrada é **faixa de gasto/churn**, não tenure ou demografia.
- `MonthlyCharges` é o atributo com maior poder discriminativo entre os dois clusters.
