# Roadmap

## Status das Fases

| Fase | Descrição | Status |
|---|---|---|
| **Fase 1** | Estruturação do repositório e documentação | Concluída |
| **Fase 2** | Análise exploratória e pré-processamento | Concluída (`01_eda.ipynb`) |
| **Fase 3** | Clusterização baseline (K-Means) | Concluída (`02_clustering.ipynb`) |
| **Fase 3b** | Comparação de algoritmos (DBSCAN, Agglomerative) | Pendente |
| **Fase 4** | Predição de churn e comparação de modelos | Pendente |
| **Fase 5** | Recomendações de retenção e relatório final | Pendente |

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
[⬜] Implementação DBSCAN
[⬜] Implementação Agglomerative Clustering
[⬜] Comparação e seleção do melhor modelo de cluster
[⬜] Treinamento dos modelos preditivos
[⬜] Experimento comparativo com/sem cluster
[⬜] Sistema de recomendação de retenção
[⬜] Relatório final e apresentação
```

---

## Resultados da Semana 2

| Item | Resultado |
|---|---|
| Algoritmo | K-Means (baseline) |
| K selecionado | 2 (empírico, k=2..12) |
| Silhouette Score | 0,344 |
| Davies-Bouldin | 1,146 |
| Clusters identificados | 2 (Insatisfeito com serviços / Econômico estável) |
| Notebook | `02_clustering.ipynb` |

---

## Desafios Técnicos

| Desafio | Estratégia | Status |
|---|---|---|
| Desbalanceamento de classes (~73/27) | SMOTE, `class_weight`, ajuste de threshold | Pendente (Fase 4) |
| Escolha do K ideal | Elbow + Silhouette + análise interpretativa | Resolvido (K=2) |
| Elbow difuso no Telco | Documentado como limitação; testar Agglomerative | Identificado |
| Dados mistos (numérico + categórico) | Gower distance / K-Prototypes na Semana 3 | Pendente |
| Colinearidade entre atributos numéricos | Exclusão de `TotalCharges` do clustering | Resolvido |
| Clusters desbalanceados (78/22) | Testar DBSCAN; avaliar K maior | Identificado |
| Overfitting em modelos complexos | Validação cruzada, regularização | Pendente (Fase 4) |

---

## Limitações do Projeto

- Dataset estático — sem captura de mudanças temporais no comportamento do cliente.
- O Telco Dataset é público e pode não refletir fielmente um cenário de produção real.
- K-Means baseline com K=2 produziu apenas 2 perfis; segmentação mais granular depende de algoritmos alternativos.
- Recomendações baseadas em regras estáticas, sem feedback loop.

---

## Melhorias Futuras

| Melhoria | Complexidade | Prioridade |
|---|---|---|
| DBSCAN + Agglomerative (Semana 3) | Média | Alta |
| Gower distance / K-Prototypes para dados mistos | Média | Alta |
| SHAP/LIME para explicabilidade das predições | Média | Média |
| Dashboard interativo com Streamlit | Média | Média |
| XGBoost / LightGBM como modelos adicionais | Baixa | Baixa |
| MLflow para rastreamento de experimentos | Média | Baixa |
| Séries temporais para captura de tendências | Alta | Baixa |
| API REST para predição em tempo real | Média | Baixa |
| Feedback loop com resultado das ações de retenção | Muito Alta | Baixa |
