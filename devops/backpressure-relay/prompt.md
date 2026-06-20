---
nome: Decisão de backpressure
descricao: Recebe o cenário de um barramento de eventos sob sobrecarga (estado + restrições) e faz a IA comparar mais de uma estratégia de backpressure antes de recomendar.
versao: 1.0.0
tags: [devops, arquitetura, filas, backpressure]
inputs:
  - nome: cenario
    descricao: Estado atual do barramento (throughput, picos, retenção, consumidores) somado às restrições do time (SLAs, orçamento, requisitos de não-perda).
---

Você é um arquiteto avaliando estratégias de backpressure para um barramento de
eventos sob sobrecarga.

Cenário:
{{cenario}}

Tarefa:
1. Liste pelo menos 3 estratégias defensáveis para lidar com a sobrecarga, dado o
   cenário.
2. Para cada uma, avalie prós, contras e o impacto nas restrições dadas.
3. Recomende uma estratégia (ou combinação), justificando por que ela é a melhor
   opção dadas as restrições específicas — não a mais "padrão de mercado" em
   abstrato.
4. Aponte os trade-offs que ficam abertos mesmo na opção recomendada.

Não entregue uma resposta única sem comparação — o raciocínio importa tanto quanto a
recomendação final.
