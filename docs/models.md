# Modelos Preditivos

> **Status:** Etapa 04 concluída. Modelo baseline persistido em `models/churn_lr_baseline.joblib`. Etapa 05 (recomendações) pendente.

**Notebook de referência:** [`notebooks/04_churn_prediction.ipynb`](../notebooks/04_churn_prediction.ipynb)

---

## Algoritmos

| Modelo | Papel no Projeto |
|---|---|
| **Logistic Regression** | Baseline interpretável; coeficientes revelam peso de cada atributo — **melhor modelo empírico** |
| **Decision Tree** | Regras explícitas de decisão; útil para explicabilidade |
| **Random Forest** | Modelo principal teórico; robusto a overfitting, estima importância de atributos |
| **MLP (Rede Neural)** | Captura relações não lineares complexas |

---

## Input Utilizado no Treinamento

| Dataset | Colunas | Uso |
|---|---|---|
| `telco_clean.csv` | 35 colunas (sem cluster) | Experimento baseline |
| Reconstruído via `kmeans_model.joblib` | + coluna `kmeans_cluster` | Experimento + cluster |

> **Nota:** O arquivo `telco_with_clusters.csv` contém rótulos DBSCAN (108 clusters). Para o experimento comparativo, os clusters K-Means foram reconstruídos diretamente a partir do modelo salvo, por serem mais interpretáveis como feature categórica binária.

---

## Estratégia de Treinamento

```
telco_clean.csv
       │
       ├── Divisão Treino/Teste (80/20 estratificada por Churn)
       │       ├── Treino: 5.634 instâncias
       │       └── Teste: 1.409 instâncias
       │
       ├── SMOTE aplicado APENAS no conjunto de treino
       │       └── Sobreamostragem da classe minoritária (Churn=Yes)
       │
       └── Validação Cruzada (Stratified K-Fold, k=5, SMOTE por fold)
```

**Tratamento do desbalanceamento:**
- `class_weight='balanced'` em LR, DT e RF
- SMOTE aplicado dentro de cada fold de CV (`imblearn.pipeline.Pipeline`)
- Desbalanceamento original: 73,5% Não Cancela / 26,5% Cancela

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

## Resultados — Validação Cruzada (5-fold, SMOTE, SEM cluster)

| Modelo | F1 Médio | F1 Desvio | AUC Médio | AUC Desvio |
|---|---|---|---|---|
| **Logistic Regression** | **0,629** | 0,010 | **0,844** | 0,013 |
| Decision Tree | 0,577 | 0,014 | 0,813 | 0,007 |
| Random Forest | 0,558 | 0,033 | 0,824 | 0,013 |
| MLP | 0,569 | 0,021 | 0,808 | 0,010 |

---

## Resultados — Hold-out Teste (20%, SEM cluster)

| Modelo | F1 | AUC | Accuracy |
|---|---|---|---|
| **Logistic Regression** | **0,613** | **0,840** | — |
| Decision Tree | 0,583 | 0,809 | — |
| Random Forest | 0,566 | 0,824 | 0,79 |
| MLP | 0,600 | 0,821 | — |

### Classification Report — Random Forest (SEM cluster)

```
                 precision    recall  f1-score   support

Não Cancela (0)     0,84      0,88      0,86      1035
    Cancela (1)     0,62      0,52      0,57       374

       accuracy                         0,79      1409
      macro avg     0,73      0,70      0,71      1409
   weighted avg     0,78      0,79      0,78      1409
```

> O Random Forest identificou apenas **52% dos clientes que realmente cancelam** (Recall = 0,52). Os 48 falsos negativos por 100 cancelamentos representam clientes em risco sem ação de retenção.

---

## Experimento Comparativo: SEM vs. COM Cluster K-Means

| Modelo | F1 sem cluster | F1 com cluster | ΔF1 | AUC sem | AUC com | ΔAUC |
|---|---|---|---|---|---|---|
| Logistic Regression | 0,613 | 0,618 | **+0,005** | 0,840 | 0,840 | +0,000 |
| Decision Tree | 0,583 | 0,583 | 0,000 | 0,809 | 0,809 | 0,000 |
| Random Forest | 0,566 | 0,572 | **+0,006** | 0,824 | 0,825 | +0,001 |
| MLP | 0,600 | 0,594 | **−0,007** | 0,821 | 0,821 | 0,000 |

### Por que o cluster não melhorou significativamente?

A hipótese de que o cluster K-Means adicionaria poder preditivo **não se confirmou** de forma relevante:

1. **Redundância de informação:** o K-Means foi treinado sobre as mesmas features usadas na predição (`Contract`, `TechSupport`, `OnlineSecurity`, `InternetService`, `PaymentMethod`, `tenure`, `MonthlyCharges`). O cluster codifica o que as features já expressam diretamente.
2. **Baixa importância:** `kmeans_cluster` ficou em **23º lugar** (último) no ranking de importância do Random Forest, com importância = 0,0036.
3. **MLP com leve queda:** o cluster adicionou ruído para a rede neural (ΔF1 = −0,007).

---

## Importância de Atributos — Random Forest (SEM cluster, top 5)

Com base nos experimentos, os atributos com maior peso preditivo foram:
1. `tenure` — tempo de contrato
2. `MonthlyCharges` — gasto mensal
3. `Contract_Month-to-month` — contrato mensal (risco)
4. `InternetService_Fiber optic` — fibra óptica (associada ao churn)
5. `PaymentMethod_Electronic check` — pagamento eletrônico (associado ao churn)

---

## Modelo Persistido

| Artefato | Caminho | Descrição |
|---|---|---|
| Modelo baseline | `models/churn_lr_baseline.joblib` | Logistic Regression (F1 CV = 0,629) |
| Scaler de predição | `models/scaler_pred.joblib` | StandardScaler ajustado em X_treino (5.634 instâncias) |
| Comparativo CSV | `reports/comparison_summary.csv` | F1/AUC sem vs. com cluster por modelo |

**Critério de seleção:** maior F1 médio em validação cruzada 5-fold.

---

## Visualizações Geradas

| Arquivo | Descrição |
|---|---|
| `10_confusion_matrices_sem_cluster.png` | Matrizes de confusão dos 4 modelos (SEM cluster) |
| `11_roc_curves_sem_cluster.png` | Curvas ROC dos 4 modelos (SEM cluster) |
| `12_cv_sem_cluster.png` | F1 e AUC médios com desvio padrão (5-fold CV) |
| `13_feature_importance_sem_cluster.png` | Top 15 atributos — Random Forest SEM cluster |
| `14_comparison_sem_com_cluster.png` | Barras comparativas F1 e AUC: SEM vs. COM cluster |
| `15_feature_importance_com_cluster.png` | Top 16 atributos — Random Forest COM cluster (destaque em laranja) |

---

## Conclusões da Etapa 04

- **Melhor modelo:** Logistic Regression (F1 CV = 0,629, AUC CV = 0,844). As relações entre features e churn no Telco Dataset são aproximadamente lineares (`Contract`, `InternetService`, `PaymentMethod`), favorecendo um classificador linear.
- **Random Forest** ficou abaixo na CV (F1 = 0,558), mas possui maior estabilidade teórica e permanece candidato a tuning na Etapa 05.
- **Recall é a métrica crítica:** identificar o máximo de clientes que vão cancelar é mais importante que evitar falsos alarmes.
- **Cluster como feature:** ganho marginal e não significativo — a segmentação não supervisionada não adicionou informação nova ao modelo preditivo com o conjunto de features atual.

---

## Próximos Passos (Etapa 05)

- Ajuste de threshold de decisão para priorizar Recall sobre Precision.
- Sistema de recomendação de retenção baseado no perfil do cluster K-Means.
- Avaliação de impacto estimado das ações por segmento.
