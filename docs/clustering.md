# Clusterização de Clientes

A clusterização é usada para descobrir perfis comportamentais nos dados de forma não supervisionada. Os clusters resultantes são interpretados semanticamente e incorporados ao pipeline preditivo como atributo adicional.

---

## Algoritmos Avaliados

### K-Means
Particionamento baseado em centroides. Indicado para clusters compactos e bem separados.

- **Seleção do K**: método do cotovelo (*Elbow Method*) combinado com Silhouette Score
- **Vantagem**: rápido, escalável e interpretável
- **Limitação**: assume clusters esféricos, sensível a outliers

### DBSCAN
Baseado em densidade. Identifica clusters de formato arbitrário e trata outliers nativamente.

- **Parâmetros críticos**: `eps` (raio de vizinhança) e `min_samples`
- **Vantagem**: não exige número de clusters pré-definido
- **Limitação**: sensível à escolha de parâmetros em alta dimensionalidade

### Agglomerative Clustering (Hierárquico)
Abordagem bottom-up que agrupa progressivamente os pontos mais similares.

- **Visualização**: dendrograma para análise da estrutura hierárquica
- **Linkage testados**: Ward, Complete, Average
- **Vantagem**: permite análise em múltiplos níveis de granularidade

---

## Métricas de Avaliação

| Métrica | Interpretação | Ideal |
|---|---|---|
| **Silhouette Score** | Coesão intra-cluster vs. separação inter-cluster | Próximo de +1 |
| **Davies-Bouldin Index** | Razão entre dispersão interna e separação entre clusters | Próximo de 0 |
| **Calinski-Harabasz Score** | Razão variância inter/intra-cluster | Quanto maior, melhor |

---

## Interpretação dos Clusters

Após a clusterização, cada grupo é nomeado com base nos valores médios de seus atributos. Exemplos de perfis esperados:

| Cluster | Perfil Esperado | Características |
|---|---|---|
| A | Fidelizado de alto valor | Longa permanência, alto gasto, contrato anual |
| B | Novo em risco | Baixa permanência, contrato mensal, fibra óptica |
| C | Econômico estável | Permanência média, baixo gasto, poucos serviços |
| D | Insatisfeito com serviços | Muitos serviços, alto histórico de churn |

> Os perfis reais serão determinados após a execução do pipeline sobre os dados.

---

## Descoberta de Padrões Comportamentais

A análise dos clusters permitirá responder perguntas como:

- Quais serviços estão mais associados a clientes fiéis vs. clientes em risco?
- Existe um perfil demográfico predominante entre quem cancela?
- Clientes com contrato mensal e fibra óptica formam um cluster de alto risco?
- Qual o valor médio de `MonthlyCharges` por cluster e sua correlação com churn?
