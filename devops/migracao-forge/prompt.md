---
nome: Migração batch para event-driven (cadeia)
descricao: Cadeia de 3 prompts encadeados que migra o pipeline Forge de lote para orientado a eventos — diagnóstico, plano de etapas reversíveis e detalhamento executável da primeira etapa.
versao: 1.0.0
tags: [devops, dados, migracao, prompt-chaining]
inputs:
  - nome: estado_atual
    descricao: "Elo 1 — estado atual do pipeline em lote (ingestão, transformação, destino, dependentes, pontos frágeis)."
  - nome: diagnostico
    descricao: "Elo 2 — saída do Prompt 1 (pontos que dependem de lote fechado, riscos por dependente, pré-condições)."
  - nome: restricoes
    descricao: "Elo 2 — restrições da migração (continuidade dos dependentes, sem big-bang, reversibilidade)."
  - nome: plano_etapas
    descricao: "Elo 3 — saída do Prompt 2 (plano de migração em etapas incrementais e reversíveis)."
---

# Cadeia de migração do Forge — 3 elos encadeados
# Cada elo recebe a saída do anterior como variável de entrada. Rode em ordem.

## Prompt 1 — Diagnóstico

Você é um arquiteto de dados analisando um pipeline de processamento em lote que vai
migrar para um modelo orientado a eventos.

Estado atual do pipeline:
{{estado_atual}}

Tarefa: diagnostique o estado atual, identificando:
1. Os pontos que dependem da semântica de "lote fechado" e por isso não migram
   direto pra streaming sem ajuste.
2. Os riscos da migração para cada sistema dependente listado no estado atual.
3. As pré-condições que precisam existir antes de iniciar a migração.

Saída: lista objetiva. Não proponha o plano de migração ainda, só o diagnóstico.

## Prompt 2 — Plano de etapas

Você é o mesmo arquiteto. Com base no diagnóstico abaixo, proponha o plano de
migração em etapas incrementais — nunca big-bang, cada etapa reversível.

Diagnóstico:
{{diagnostico}}

Restrições:
{{restricoes}}

Tarefa: liste as etapas em ordem, cada uma com objetivo, o que muda, como reverter se
der problema, e o que precisa continuar rodando em paralelo (old + new) até a etapa
seguinte.

## Prompt 3 — Plano executável e reversível (Etapa 1)

Você é o mesmo arquiteto, agora detalhando a primeira etapa do plano para execução
real.

Plano de etapas:
{{plano_etapas}}

Tarefa: detalhe a Etapa 1 como plano executável: passos técnicos concretos, critério
de sucesso mensurável, ponto de rollback, e quem precisa validar antes de avançar
pra etapa 2.
