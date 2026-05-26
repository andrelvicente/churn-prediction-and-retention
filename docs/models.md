# Modelos Preditivos

## Algoritmos

| Modelo | Papel no Projeto |
|---|---|
| **Logistic Regression** | Baseline interpretável; coeficientes revelam peso de cada atributo |
| **Decision Tree** | Regras explícitas de decisão; útil para explicabilidade |
| **Random Forest** | Modelo principal; robusto a overfitting, estima importância de atributos |
| **MLP (Rede Neural)** | Captura relações não lineares complexas |

---

## Estratégia de Treinamento

```
Dataset com Cluster
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
| **Baseline** | Atributos originais | Performance de referência |
| **+ Cluster** | Atributos originais + `cluster` | F1 e AUC superiores |

Os resultados serão apresentados com intervalos de confiança (validação cruzada).

---

## Metas de Performance

| Modelo | F1-Score Alvo | ROC-AUC Alvo |
|---|---|---|
| Baseline (sem cluster) | ≥ 0.70 | ≥ 0.78 |
| Com cluster incorporado | ≥ 0.75 | ≥ 0.82 |
