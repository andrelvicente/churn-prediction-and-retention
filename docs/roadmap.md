# Roadmap

## Status das Fases

| Fase | Descrição | Status |
|---|---|---|
| **Fase 1** | Estruturação do repositório e documentação | ✅ Concluída |
| **Fase 2** | Análise exploratória e pré-processamento | 🔄 Em andamento (EDA concluída) |
| **Fase 3** | Clusterização e interpretação dos perfis | ⬜ Pendente |
| **Fase 4** | Predição de churn e comparação de modelos | ⬜ Pendente |
| **Fase 5** | Recomendações de retenção e relatório final | ⬜ Pendente |

## Checklist Detalhado

```
[✅] Definição do escopo e documentação inicial
[✅] Estrutura do repositório
[✅] EDA — distribuições, correlações, análise da variável alvo (`notebooks/01_eda.ipynb`)
[✅] Tratamento de nulos e inconsistências (TotalCharges, duplicatas)
[🔄] Encoding e normalização (início no EDA; completo em `02_preprocessing`)
[⬜] Implementação K-Means + seleção do K ideal
[⬜] Implementação DBSCAN e Agglomerative
[⬜] Comparação e seleção do melhor modelo de cluster
[⬜] Interpretação semântica dos clusters
[⬜] Treinamento dos modelos preditivos
[⬜] Experimento comparativo com/sem cluster
[⬜] Sistema de recomendação de retenção
[⬜] Relatório final e apresentação
```

---

## Desafios Técnicos

| Desafio | Estratégia |
|---|---|
| Desbalanceamento de classes (~73/27) | SMOTE, `class_weight`, ajuste de threshold |
| Escolha do K ideal | Elbow Method + Silhouette + análise interpretativa |
| Colinearidade entre atributos numéricos | Seleção criteriosa + análise de correlação |
| Overfitting em modelos complexos | Validação cruzada, regularização |
| Alta dimensionalidade após encoding | Feature selection, PCA exploratório |

---

## Limitações do Projeto

- Dataset estático — sem captura de mudanças temporais no comportamento do cliente.
- O Telco Dataset é público e pode não refletir fielmente um cenário de produção real.
- Recomendações baseadas em regras estáticas, sem feedback loop.

---

## Melhorias Futuras

| Melhoria | Complexidade |
|---|---|
| SHAP/LIME para explicabilidade das predições | Média |
| Dashboard interativo com Streamlit | Média |
| XGBoost / LightGBM como modelos adicionais | Baixa |
| MLflow para rastreamento de experimentos | Média |
| Séries temporais para captura de tendências | Alta |
| API REST para predição em tempo real | Média |
| Feedback loop com resultado das ações de retenção | Muito Alta |
