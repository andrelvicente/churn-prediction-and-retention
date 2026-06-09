# Recomendações de Retenção

O sistema traduz predições de churn em ações práticas personalizadas por perfil de cluster.

> **Status:** perfis identificados pelo baseline K-Means (Semana 2). Lógica de recomendação será implementada na Fase 5.

---

## Lógica de Recomendação

```
Cliente
  └── Probabilidade de churn alta?
        └── Sim → Identificar cluster do cliente
                    └── Aplicar ação específica do perfil
```

A combinação de **predição** (quem vai cancelar) com **perfil** (por que vai cancelar) permite recomendações mais precisas do que um sistema de retenção genérico.

---

## Perfis Identificados (Baseline K-Means, K=2)

| Cluster | Rótulo | Tamanho | Churn | Perfil |
|---|---|---|---|---|
| 0 | Insatisfeito com serviços | 77,9% | 31,8% | Alto gasto (R$ 77), contrato mensal, fibra óptica |
| 1 | Econômico estável | 22,1% | 8,0% | Baixo gasto (R$ 21), contrato longo, sem internet |

---

## Ações por Perfil

### Cluster 0 — Insatisfeito com serviços (prioridade urgente)

Clientes com planos premium (fibra óptica), contrato mensal e gasto elevado. Taxa de churn ~4× superior ao Cluster 1.

| Ação | Detalhe |
|---|---|
| Contato proativo | Antes do vencimento do contrato mensal |
| Upgrade para contrato anual | Desconto de 15–20% no primeiro ano |
| Suporte técnico gratuito | Período trial de 3 meses |
| Revisão do pacote | Avaliar se serviços contratados atendem ao perfil de uso |

### Cluster 1 — Econômico estável (prioridade média)

Clientes fidelizados com planos básicos e baixo gasto. Baixo risco de churn — ações focadas em upsell, não retenção.

| Ação | Detalhe |
|---|---|
| Oferta de serviços adicionais | Internet ou streaming com desconto |
| Programa de fidelidade | Benefícios por permanência |
| Cross-sell | Upgrade gradual de plano |

---

## Ações por Perfil (referência teórica — Semana 3+)

Perfis adicionais que podem emergir com algoritmos alternativos ou K maior:

| Perfil | Ação Recomendada | Prioridade |
|---|---|---|
| Fidelizado de alto valor | Programa de fidelidade, upgrade de plano | Alta |
| Novo em risco | Contato proativo, desconto no 1º ano | Urgente |
| Econômico estável | Oferta de serviços adicionais com desconto | Média |
| Insatisfeito com serviços | Revisão do pacote, suporte dedicado | Urgente |

---

## Catálogo de Ações

| Ação | Quando Aplicar |
|---|---|
| Contato prioritário | Probabilidade de churn > threshold + Cluster 0 |
| Desconto personalizado | Cluster 0 (sensível a preço/contrato mensal) |
| Campanha segmentada | Comunicação direcionada por perfil de uso |
| Mudança de plano | Cliente com serviços inadequados ao perfil |
| Retenção preventiva | Antes de o cliente manifestar intenção de cancelar |
| Suporte dedicado | Cluster 0 (alta taxa de churn + fibra óptica) |

---

## Insights Baseados nos Dados (Semana 2)

```
"O Cluster 0 (Insatisfeito com serviços) representa 77,9% dos clientes
e concentra 31,8% de churn — ~4× superior ao Cluster 1 (8,0%).
Perfil: fibra óptica + contrato mensal + gasto médio de R$ 77.
Ação recomendada: migração para contrato anual com desconto
+ suporte técnico gratuito por 3 meses."

"O Cluster 1 (Econômico estável) representa 22,1% dos clientes
com apenas 8,0% de churn. Perfil: sem internet + contrato longo + R$ 21/mês.
Recursos de retenção neste grupo têm ROI baixo;
priorizar upsell de serviços adicionais."
```

---

## Limitações

- As recomendações são regras estáticas baseadas em 2 perfis do baseline K-Means.
- Apenas 2 clusters foram identificados; segmentações mais granulares dependem da Semana 3.
- Avaliar o impacto real exigiria um experimento A/B em ambiente de produção.
- Os perfis são recalibrados a cada re-execução do pipeline de clusterização.
