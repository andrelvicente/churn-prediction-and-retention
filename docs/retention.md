# Recomendações de Retenção

O sistema traduz predições de churn em ações práticas personalizadas por perfil de cluster.

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

## Ações por Perfil

| Cluster | Perfil | Ação Recomendada | Prioridade |
|---|---|---|---|
| A | Fidelizado de alto valor | Programa de fidelidade, upgrade de plano | Alta |
| B | Novo em risco | Contato proativo, desconto no 1º ano | Urgente |
| C | Econômico estável | Oferta de serviços adicionais com desconto | Média |
| D | Insatisfeito com serviços | Revisão do pacote, suporte dedicado | Urgente |

---

## Catálogo de Ações

| Ação | Quando Aplicar |
|---|---|
| Contato prioritário | Probabilidade de churn > threshold definido |
| Desconto personalizado | Clusters sensíveis a preço |
| Campanha segmentada | Comunicação direcionada por perfil de uso |
| Mudança de plano | Cliente com serviços inadequados ao perfil |
| Retenção preventiva | Antes de o cliente manifestar intenção de cancelar |
| Suporte dedicado | Clusters com histórico de insatisfação com atendimento |

---

## Exemplos de Insights Esperados

```
"Clientes do Cluster B (contrato mensal + fibra óptica + sem suporte técnico)
apresentam probabilidade de churn 3.2× superior à média.
Ação recomendada: upgrade para contrato anual com 20% de desconto
+ suporte técnico gratuito por 3 meses."

"O Cluster D representa 18% dos clientes, mas concentra 41% dos cancelamentos.
Priorizar ações de retenção neste segmento pode reduzir o churn geral em ~15%."

"Clientes com tenure > 36 meses raramente cancelam, independente do cluster.
Recursos de retenção neste grupo têm ROI negativo e devem ser redirecionados."
```

---

## Limitações

- As recomendações são regras estáticas baseadas em perfis — não um sistema de recomendação dinâmico.
- Avaliar o impacto real exigiria um experimento A/B em ambiente de produção.
- Os perfis são recalibrados a cada re-execução do pipeline de clusterização.
