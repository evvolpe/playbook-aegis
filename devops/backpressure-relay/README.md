---
nome: Decisão de backpressure
descricao: Recebe o cenário de um barramento de eventos sob sobrecarga (estado + restrições) e faz a IA comparar mais de uma estratégia de backpressure antes de recomendar.
versao: 1.0.0
tags: [devops, arquitetura, filas, backpressure]
inputs:
  - nome: cenario
    descricao: Estado atual do barramento (throughput, picos, retenção, consumidores) somado às restrições do time (SLAs, orçamento, requisitos de não-perda).
---

## Objetivo
Apoiar uma decisão de arquitetura cara forçando a IA a comparar pelo menos 3
estratégias de backpressure, pesar prós/contras contra as restrições reais e só então
recomendar — em vez de cuspir uma resposta única "de mercado".

## Casos de uso
- Barramento sob pico de cliente barulhento ameaçando SLA de alerting.
- Decisão entre priorização de consumo, DLQ, particionamento por tenant e autoscaling.
- Qualquer trade-off de fila em que o raciocínio importa tanto quanto a escolha.

## Exemplo
Parâmetro `cenario` = `fixtures/cenario.txt` (Relay, Checkpoint 04). Modelo: **Claude
Sonnet 4.6**. Recomendação resumida:

> Combinar (1) priorização do Sentinel + (2) DLQ como rede de segurança do Forge +
> (4) autoscaling com teto de réplicas. Descartar (3) particionamento por tenant agora
> — é a opção certa estruturalmente, mas cara demais para o orçamento já 8% estourado e
> para um pico de só 25min. Trade-off aberto: pico muito mais longo ainda estouraria o
> SLA do Forge, e aí o particionamento volta como próximo passo.

## Limitações
- Saída aberta — qualidade checada por LLM-as-judge, não por regex.
- A recomendação é tão boa quanto o cenário fornecido; restrições omitidas não entram
  no raciocínio.

## Curadoria
Técnica: **comparação estruturada (tree-of-thought simplificado)**. Exigir "pelo menos
3 estratégias" + "trade-off que fica aberto mesmo na recomendada" evitou o LLM
recomendar algo e sumir com os contras. Sem isso, a IA recomendava o particionamento
sem citar o estouro de orçamento.
