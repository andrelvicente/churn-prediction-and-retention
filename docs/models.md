# Modelos Preditivos

> **Status:** Fase 4 pendente. A coluna `cluster` já está disponível em `telco_with_clusters.csv` para o experimento comparativo.

---

## Algoritmos

| Modelo | Papel no Projeto |
|---|---|
| **Logistic Regression** | Baseline interpretável; coeficientes revelam peso de cada atributo |
| **Decision Tree** | Regras explícitas de decisão; útil para explicabilidade |
| **Random Forest** | Modelo principal; robusto a overfitting, estima importância de atributos |
| **MLP (Rede Neural)** | Captura relações não lineares complexas |

---

## Input Disponível para Treinamento

| Dataset | Colunas | Uso |
|---|---|---|
| `telco_clean.csv` | 35 colunas (sem cluster) | Experimento baseline |
| `telco_with_clusters.csv` | 38 colunas (+ `cluster`, `cluster_label`, `clustering_algorithm`) | Experimento + cluster |

Atributos de cluster disponíveis:

| Coluna | Valores | Descrição |
|---|---|---|
| `cluster` | 0, 1 | ID numérico do cluster K-Means |
| `cluster_label` | Insatisfeito com serviços, Econômico estável | Rótulo semântico |
| `clustering_algorithm` | kmeans | Algoritmo que gerou o rótulo |

**Hipótese:** incluir `cluster` como feature categórica deve melhorar Recall e F1, especialmente para identificar clientes do Cluster 0 (31,8% de churn).

---

## Estratégia de Treinamento

```
telco_with_clusters.csv
       │
       ├── Divisão Treino/Teste (80/20 estratificada)
       ├── Validação Cruzada (Stratified K-Fold, k=5)
       ├── Ajuste de Hiperparâmetros (GridSearchCV / RandomizedSearchCV)
       └── Avaliação Final no conjunto de teste
```

**Tratamento do desbalanceamento**: `class_weight='balanced'` nos modelos suportados e/ou SMOTE aplicado apenas ao conjunto de treino.

---

## Métricas de Avaliação

| Métrica | Fórmula | Por que importa |
|---|---|---|
| **Precision** | TP / (TP + FP) | Evitar contato desnecessário com clientes que não cancelariam |
| **Recall** | TP / (TP + FN) | Não deixar passar clientes que vão cancelar |
| **F1-Score** | 2 × (P × R) / (P + R) | Equilíbrio entre os dois |
| **ROC-AUC** | Área sob a curva ROC | Capacidade discriminativa geral do modelo |
| **Confusion Matrix** | — | Distribuição completa dos erros |

> Em problemas de churn, o **Recall** costuma ser mais crítico que a Accuracy: o custo de não identificar um cliente que vai cancelar é maior que o custo de um falso alarme.

---

## Experimento Comparativo

O objetivo central é quantificar o ganho obtido pela incorporação dos clusters:

| Experimento | Atributos de Entrada | Hipótese |
|---|---|---|
| **Baseline** | Atributos originais (sem `cluster`) | Performance de referência |
| **+ Cluster** | Atributos originais + `cluster` | F1 e AUC superiores |

Com base nos perfis identificados, espera-se que o modelo com cluster capture melhor a separação entre:
- Cluster 0 (31,8% churn) — clientes de alto risco
- Cluster 1 (8,0% churn) — clientes estáveis

Os resultados serão apresentados com intervalos de confiança (validação cruzada).

---

## Metas de Performance

| Modelo | F1-Score Alvo | ROC-AUC Alvo |
|---|---|---|
| Baseline (sem cluster) | ≥ 0,70 | ≥ 0,78 |
| Com cluster incorporado | ≥ 0,75 | ≥ 0,82 |

---

## Contexto do Clustering (Semana 2)

O baseline K-Means (K=2) produziu Silhouette Score = 0,344 — estrutura moderada. Mesmo com separação imperfeita, a diferença de churn entre clusters (31,8% vs. 8,0%) sugere valor preditivo na feature `cluster`.

Se algoritmos alternativos (DBSCAN, Agglomerative) produzirem segmentações mais granulares na Semana 3, o experimento comparativo será refeito com o melhor modelo de cluster.
