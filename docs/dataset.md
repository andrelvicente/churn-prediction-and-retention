# Dataset

**Telco Customer Churn Dataset** — [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data)

| Atributo | Valor |
|---|---|
| Volume | 7.043 registros |
| Dimensionalidade | 21 atributos originais |
| Variável alvo | `Churn` (Yes/No) |
| Desbalanceamento | ~73% Não-Churn / ~27% Churn |

---

## Dicionário de Dados

| Atributo | Tipo | Descrição |
|---|---|---|
| `customerID` | Identificador | ID único do cliente |
| `gender` | Categórico | Gênero (Male/Female) |
| `SeniorCitizen` | Binário | Cliente idoso (0/1) |
| `Partner` | Categórico | Possui cônjuge (Yes/No) |
| `Dependents` | Categórico | Possui dependentes (Yes/No) |
| `tenure` | Numérico | Meses de permanência |
| `PhoneService` | Categórico | Serviço telefônico (Yes/No) |
| `MultipleLines` | Categórico | Múltiplas linhas (Yes/No/No phone service) |
| `InternetService` | Categórico | Tipo de internet (DSL/Fiber optic/No) |
| `OnlineSecurity` | Categórico | Segurança online (Yes/No/No internet) |
| `OnlineBackup` | Categórico | Backup online (Yes/No/No internet) |
| `DeviceProtection` | Categórico | Proteção de dispositivo |
| `TechSupport` | Categórico | Suporte técnico (Yes/No/No internet) |
| `StreamingTV` | Categórico | Streaming de TV |
| `StreamingMovies` | Categórico | Streaming de filmes |
| `Contract` | Categórico | Tipo de contrato (Month-to-month/One year/Two year) |
| `PaperlessBilling` | Categórico | Faturamento sem papel (Yes/No) |
| `PaymentMethod` | Categórico | Método de pagamento |
| `MonthlyCharges` | Numérico | Cobrança mensal |
| `TotalCharges` | Numérico | Cobrança total acumulada |
| `Churn` | Binário (alvo) | Cliente cancelou? (Yes/No) |

---

## Artefatos Processados

| Arquivo | Origem | Descrição |
|---|---|---|
| `data/processed/telco_clean.csv` | `01_eda.ipynb` | Base limpa com 35 colunas (originais + engineered) |
| `data/processed/telco_clustering_base.csv` | `02_clustering.ipynb` | Subconjunto de 7 atributos + `customerID`, `Churn`, `Churn_label` |
| `data/processed/telco_with_clusters.csv` | `02_clustering.ipynb` | Base completa + `cluster`, `cluster_label`, `clustering_algorithm` |

---

## Considerações Técnicas

**Desbalanceamento de classes**: a proporção ~73/27 exige atenção no treinamento. Técnicas a considerar: SMOTE, `class_weight='balanced'`, ajuste de threshold de decisão.

**TotalCharges**: declarado como numérico, mas contém espaços em branco para clientes com `tenure = 0`. Tratado no EDA (conversão + imputação com 0).

**Multicolinearidade**: `MonthlyCharges`, `TotalCharges` e `tenure` são fortemente correlacionados. `TotalCharges` foi **excluído da clusterização** por ser derivado dos outros dois.

**customerID**: removido antes de qualquer modelagem (mantido apenas como identificador nos CSVs exportados).

**Outliers**: identificados via IQR no EDA para `tenure` e `MonthlyCharges`. Mantidos no clustering por serem valores plausíveis de clientes reais.

---

## Atributos Selecionados para Clusterização

A clusterização foi realizada sobre atributos que capturam **comportamento de uso e perfil financeiro**:

```
tenure, MonthlyCharges, Contract, InternetService,
TechSupport, OnlineSecurity, PaymentMethod
```

| Atributo | Justificativa | Excluído? |
|---|---|---|
| `tenure` | Indicador de fidelidade | Usado |
| `MonthlyCharges` | Segmentação financeira | Usado |
| `Contract` | Proxy de comprometimento | Usado |
| `InternetService` | Tipo de consumo (fibra = maior churn) | Usado |
| `TechSupport` | Engajamento com serviços premium | Usado |
| `OnlineSecurity` | Segurança digital contratada | Usado |
| `PaymentMethod` | Automatização vs. pagamento manual | Usado |
| `TotalCharges` | Colinear com tenure × MonthlyCharges | Excluído |
| `customerID` | Identificador | Excluído |
| `Churn` | Variável alvo (data leakage) | Excluído |

---

## Pré-processamento para Clustering

| Tipo | Atributos | Tratamento |
|---|---|---|
| Numérico contínuo | `tenure`, `MonthlyCharges` | `StandardScaler` |
| Ordinal | `Contract` | 0 / 1 / 2 |
| Binário com NA | `TechSupport`, `OnlineSecurity` | 1 / 0 / -1 |
| Nominal | `InternetService`, `PaymentMethod` | One-Hot Encoding (sem `drop_first`) |

**Matriz final:** 7.043 registros × 12 features.

---

## Colunas Adicionadas pelo Clustering

| Coluna | Tipo | Descrição |
|---|---|---|
| `cluster` | Inteiro (0–1) | ID do cluster atribuído pelo K-Means |
| `cluster_label` | String | Nome semântico do perfil |
| `clustering_algorithm` | String | Algoritmo utilizado (`kmeans`) |

### Distribuição atual (K-Means, K=2)

| cluster | cluster_label | Registros | % da base | Churn |
|---|---|---|---|---|
| 0 | Insatisfeito com serviços | 5.486 | 77,9% | 31,8% |
| 1 | Econômico estável | 1.557 | 22,1% | 8,0% |
