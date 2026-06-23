# Roadmap

## Status das Fases

| Fase | Descrição | Status |
|---|---|---|
| **Fase 1** | Estruturação do repositório e documentação | Concluída |
| **Fase 2** | Análise exploratória e pré-processamento | Concluída (`01_eda.ipynb`) |
| **Fase 3** | Clusterização baseline (K-Means) | Concluída (`02_clustering.ipynb`) |
| **Fase 3b** | Comparação de algoritmos (DBSCAN) | Concluída (`03_clustering_advanced.ipynb`) |
| **Fase 4** | Predição de churn e comparação de modelos | Concluída (`04_churn_prediction.ipynb`) |
| **Fase 5** | Recomendações de retenção e relatório final | **Pendente** |

## Checklist Detalhado

```
[✅] Definição do escopo e documentação inicial
[✅] Estrutura do repositório
[✅] EDA — distribuições, correlações, análise da variável alvo (01_eda.ipynb)
[✅] Tratamento de nulos e inconsistências (TotalCharges, duplicatas)
[✅] Encoding inicial no EDA (binários, ordinais, labels)
[✅] Seleção e justificativa de atributos para clustering
[✅] Pré-processamento para clustering (encoding + StandardScaler seletivo)
[✅] K-Means baseline + seleção empírica do K (Elbow + Silhouette)
[✅] Métricas internas (Silhouette, Davies-Bouldin, Calinski-Harabasz)
[✅] Visualizações (PCA, dendrograma, heatmap, churn, silhouette/amostra)
[✅] Interpretação semântica dos clusters (2 perfis identificados)
[✅] Identificação formal de problemas do baseline
[✅] Implementação DBSCAN (grid search eps × min_samples)
[✅] Comparação K-Means vs. DBSCAN com métricas internas
[✅] Seleção do algoritmo vencedor e geração de telco_with_clusters.csv
[✅] Treinamento dos modelos preditivos (LR, DT, RF, MLP)
[✅] Experimento comparativo com/sem cluster (K-Means como feature)
[✅] Validação cruzada 5-fold com SMOTE dentro de cada fold
[✅] Persistência do modelo baseline (churn_lr_baseline.joblib)
[⬜] Sistema de recomendação de retenção (05_recommendations.ipynb)
[⬜] Relatório final e apresentação
```

---

## Resultados da Semana 2 — K-Means Baseline

| Item | Resultado |
|---|---|
| Algoritmo | K-Means (baseline) |
| K selecionado | 2 (empírico, k=2..12) |
| Silhouette Score | 0,344 |
| Davies-Bouldin | 1,146 |
| Calinski-Harabasz | 3.104,6 |
| Clusters identificados | 2 (Insatisfeito com serviços / Econômico estável) |
| Notebook | `02_clustering.ipynb` |

---

## Resultados da Semana 3 — DBSCAN Avançado

| Item | Resultado |
|---|---|
| Algoritmo aplicado | DBSCAN |
| Grid de busca | eps ∈ {0,30..1,20} × min_samples ∈ {3,5,7,10} |
| Melhor configuração | eps=0,8, min_samples=3 |
| N.º de clusters | 108 micro-clusters |
| Ruído (noise) | 0,4% da base (≈ 28 pontos) |
| Silhouette Score | 0,364 (> K-Means: 0,344) |
| Davies-Bouldin | 1,299 |
| Calinski-Harabasz | 544,0 (< K-Means: 3.104,6) |
| Algoritmo vencedor | **DBSCAN** (Silhouette superior) |
| Saída gerada | `telco_with_clusters.csv` (38 colunas, rótulos DBSCAN) |
| Notebook | `03_clustering_advanced.ipynb` |

> **Observação:** Embora o DBSCAN tenha vencido pelo critério Silhouette, os 108 clusters resultantes são impraticáveis como feature categórica isolada. A Etapa 04 optou por reconstruir os clusters K-Means (2 perfis) para o experimento comparativo de predição.

---

## Resultados da Semana 4 — Predição de Churn

### Validação Cruzada (5-fold, SMOTE, SEM cluster)

| Modelo | F1 Médio | F1 Desvio | AUC Médio | AUC Desvio |
|---|---|---|---|---|
| **Logistic Regression** | **0,629** | 0,010 | **0,844** | 0,013 |
| Decision Tree | 0,577 | 0,014 | 0,813 | 0,007 |
| Random Forest | 0,558 | 0,033 | 0,824 | 0,013 |
| MLP | 0,569 | 0,021 | 0,808 | 0,010 |

### Comparativo Hold-out (80/20): SEM vs. COM cluster K-Means

| Modelo | F1 sem cluster | F1 com cluster | ΔF1 | AUC sem | AUC com | ΔAUC |
|---|---|---|---|---|---|---|
| Logistic Regression | 0,613 | 0,618 | +0,005 | 0,840 | 0,840 | +0,000 |
| Decision Tree | 0,583 | 0,583 | 0,000 | 0,809 | 0,809 | 0,000 |
| Random Forest | 0,566 | 0,572 | +0,006 | 0,824 | 0,825 | +0,001 |
| MLP | 0,600 | 0,594 | −0,007 | 0,821 | 0,821 | 0,000 |

**Modelo persistido:** `churn_lr_baseline.joblib` (Logistic Regression, F1 CV = 0,629)

---

## Desafios Técnicos

| Desafio | Estratégia | Status |
|---|---|---|
| Desbalanceamento de classes (~73/27) | SMOTE dentro de cada fold de CV | Resolvido (Fase 4) |
| Escolha do K ideal | Elbow + Silhouette + análise interpretativa | Resolvido (K=2) |
| Elbow difuso no Telco | Documentado como limitação; DBSCAN aplicado | Resolvido |
| Dados mistos (numérico + categórico) | Distância Euclidiana com OHE | Mitigado (limitação conhecida) |
| Colinearidade entre atributos numéricos | Exclusão de `TotalCharges` do clustering | Resolvido |
| DBSCAN com muitos micro-clusters | Documentado; K-Means usado na predição | Identificado |
| Cluster como feature com ganho marginal | Redundância com features originais documentada | Identificado |
| Overfitting em modelos complexos | Validação cruzada estratificada, `class_weight='balanced'` | Resolvido (Fase 4) |

---

## Limitações do Projeto

- Dataset estático — sem captura de mudanças temporais no comportamento do cliente.
- O Telco Dataset é público e pode não refletir fielmente um cenário de produção real.
- K-Means baseline com K=2 produziu apenas 2 perfis; DBSCAN gerou micro-clusters impraticáveis como feature.
- A adição do cluster K-Means não melhorou significativamente a predição (ΔF1 ≤ +0,006).
- Recomendações ainda baseadas em regras estáticas, sem feedback loop.

---

## Melhorias Futuras

| Melhoria | Complexidade | Prioridade |
|---|---|---|
| Sistema de recomendação de retenção (Fase 5) | Baixa | **Alta** (próximo passo) |
| Tuning de hiperparâmetros (GridSearchCV/RandomizedSearchCV) | Média | Alta |
| Ajuste de threshold de decisão (Recall > Precision) | Baixa | Alta |
| SHAP/LIME para explicabilidade das predições | Média | Média |
| K-Prototypes / Gower distance para dados mistos | Média | Média |
| Dashboard interativo com Streamlit | Média | Média |
| XGBoost / LightGBM como modelos adicionais | Baixa | Baixa |
| MLflow para rastreamento de experimentos | Média | Baixa |
| API REST para predição em tempo real | Média | Baixa |
| Feedback loop com resultado das ações de retenção | Muito Alta | Baixa |
