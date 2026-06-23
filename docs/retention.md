# Recomendações de Retenção

O sistema traduz predições de churn em ações práticas personalizadas por perfil de cluster.

> **Status:** perfis K-Means consolidados (Etapa 03). Modelo preditivo treinado (Etapa 04, F1 CV = 0,629). Lógica de recomendação a implementar no notebook `05_recommendations.ipynb` (Etapa 05 — pendente).

---

## Lógica de Recomendação

```
Cliente
  └── Probabilidade de churn ≥ threshold?
        └── Sim → Identificar cluster K-Means do cliente
                    └── Aplicar ação específica do perfil
```

A combinação de **predição** (quem vai cancelar) com **perfil** (por que vai cancelar) permite recomendações mais precisas do que um sistema de retenção genérico.

**Modelo de predição:** Logistic Regression (`churn_lr_baseline.joblib`)
- F1 CV = 0,629 | AUC CV = 0,844
- Recall classe Churn = **a ser otimizado via ajuste de threshold** na Etapa 05

---

## Perfis para Segmentação de Recomendações

Os perfis K-Means (K=2) são usados para personalizar as ações de retenção por representarem segmentos interpretáveis e distintos em comportamento de churn:

| Cluster | Rótulo | Tamanho | Churn | MonthlyCharges médio | Contrato dominante | Internet |
|---|---|---|---|---|---|---|
| 0 | Insatisfeito com serviços | 77,9% | 31,8% | R$ 77,10 | Month-to-month | Fiber optic |
| 1 | Econômico estável | 22,1% | 8,0% | R$ 21,20 | Two year | No |

> **Por que K-Means e não DBSCAN?** O DBSCAN produziu 108 micro-clusters (Etapa 03), impraticáveis para definição de ações de retenção consolidadas. Os 2 perfis K-Means oferecem segmentos interpretáveis e acionáveis.

---

## Ações por Perfil

### Cluster 0 — Insatisfeito com serviços (prioridade urgente)

Clientes com planos premium (fibra óptica), contrato mensal e gasto elevado. Taxa de churn ~4× superior ao Cluster 1. **Prioridade máxima de retenção.**

| Ação | Detalhe |
|---|---|
| Contato proativo | Antes do vencimento do contrato mensal |
| Upgrade para contrato anual | Desconto de 15–20% no primeiro ano |
| Suporte técnico gratuito | Período trial de 3 meses |
| Revisão do pacote | Avaliar se serviços contratados atendem ao perfil de uso |
| Ajuste de plano | Migração para pacote com melhor custo-benefício |

### Cluster 1 — Econômico estável (prioridade média)

Clientes fidelizados com planos básicos e baixo gasto. Baixo risco de churn. **Ações focadas em upsell, não retenção emergencial.**

| Ação | Detalhe |
|---|---|
| Oferta de serviços adicionais | Internet ou streaming com desconto especial |
| Programa de fidelidade | Benefícios por permanência (ex.: meses gratuitos) |
| Cross-sell | Upgrade gradual de plano com período de carência |
| Monitoramento preventivo | Alertas se comportamento mudar para o perfil do Cluster 0 |

---

## Catálogo de Ações

| Ação | Quando Aplicar |
|---|---|
| Contato prioritário | Probabilidade de churn ≥ threshold + Cluster 0 |
| Desconto personalizado | Cluster 0 (sensível a preço e contrato mensal) |
| Campanha segmentada | Comunicação direcionada por perfil de uso |
| Mudança de plano | Cliente com serviços inadequados ao perfil |
| Retenção preventiva | Antes de o cliente manifestar intenção de cancelar |
| Suporte dedicado | Cluster 0 (alta taxa de churn + fibra óptica) |
| Upsell de serviço | Cluster 1 (fidelizado, baixo risco, potencial de receita) |

---

## Insights Baseados nos Dados

```
"O Cluster 0 (Insatisfeito com serviços) representa 77,9% dos clientes
e concentra 31,8% de churn — ~4× superior ao Cluster 1 (8,0%).
Perfil: fibra óptica + contrato mensal + gasto médio de R$ 77,10.
Ação recomendada: migração para contrato anual com desconto
+ suporte técnico gratuito por 3 meses."

"O Cluster 1 (Econômico estável) representa 22,1% dos clientes
com apenas 8,0% de churn. Perfil: sem internet + contrato longo + R$ 21,20/mês.
Recursos de retenção emergencial neste grupo têm ROI baixo;
priorizar upsell de serviços adicionais."
```

---

## Integração com o Modelo Preditivo

O fluxo de inferência na Etapa 05 será:

```python
# 1. Carregar modelo e scaler
model = joblib.load("models/churn_lr_baseline.joblib")
scaler = joblib.load("models/scaler_pred.joblib")

# 2. Predizer probabilidade de churn
X_scaled = scaler.transform(X_novo)
prob_churn = model.predict_proba(X_scaled)[:, 1]

# 3. Aplicar threshold (a ser definido na Etapa 05 para maximizar Recall)
churn_flag = prob_churn >= threshold

# 4. Identificar cluster K-Means
kmeans = joblib.load("models/kmeans_model.joblib")
cluster = kmeans.predict(X_cluster_features)

# 5. Gerar recomendação
recomendacao = gerar_acao(churn_flag, cluster)
```

---

## Limitações

- As recomendações são regras estáticas baseadas em 2 perfis do K-Means.
- O modelo preditivo atual (Logistic Regression) tem Recall = ~0,61 na classe Churn — ajuste de threshold é necessário para capturar mais cancelamentos.
- Apenas 2 clusters foram identificados como interpretáveis; o DBSCAN gerou 108 micro-segmentos não acionáveis.
- Avaliar o impacto real exigiria um experimento A/B em ambiente de produção.
- Os perfis são recalibrados a cada re-execução do pipeline de clusterização.
- Sem feedback loop: a eficácia das ações de retenção não alimenta o modelo.
