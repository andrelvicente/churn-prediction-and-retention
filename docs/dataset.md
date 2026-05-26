# Dataset

**Telco Customer Churn Dataset** — [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data)

| Atributo | Valor |
|---|---|
| Volume | ~7.043 registros |
| Dimensionalidade | 21 atributos |
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

## Considerações Técnicas

**Desbalanceamento de classes**: a proporção ~73/27 exige atenção no treinamento. Técnicas a considerar: SMOTE, `class_weight='balanced'`, ajuste de threshold de decisão.

**TotalCharges**: declarado como numérico, mas contém espaços em branco para clientes com `tenure = 0`. Requer conversão forçada e tratamento dos nulos gerados.

**Multicolinearidade**: `MonthlyCharges`, `TotalCharges` e `tenure` são fortemente correlacionados. O atributo `TotalCharges` pode ser derivado dos outros dois, exigindo análise antes de incluí-lo nos modelos.

**customerID**: deve ser removido antes de qualquer modelagem.

---

## Atributos Selecionados para Clusterização

A clusterização será realizada sobre atributos que capturam **comportamento de uso e perfil financeiro**, evitando variáveis com alta colinearidade:

```
tenure, MonthlyCharges, Contract, InternetService,
TechSupport, OnlineSecurity, PaymentMethod
```
