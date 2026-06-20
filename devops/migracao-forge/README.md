---
nome: Migração batch para event-driven (cadeia)
descricao: Cadeia de 3 prompts encadeados que migra o pipeline Forge de lote para orientado a eventos — diagnóstico, plano de etapas reversíveis e detalhamento executável da primeira etapa.
versao: 1.0.0
tags: [devops, dados, migracao, prompt-chaining]
inputs:
  - nome: estado_atual
    descricao: "Elo 1 — estado atual do pipeline em lote."
  - nome: diagnostico
    descricao: "Elo 2 — saída do Prompt 1."
  - nome: restricoes
    descricao: "Elo 2 — restrições da migração."
  - nome: plano_etapas
    descricao: "Elo 3 — saída do Prompt 2."
---

## Objetivo
Quebrar uma migração grande (batch → event-driven) numa **cadeia de prompts** em que
cada elo recebe a saída do anterior, em vez de um único prompt gigante que sai raso.
Diagnóstico → plano de etapas reversíveis → detalhamento executável da Etapa 1.

## Casos de uso
- Migrar pipeline de processamento de lote para streaming sem big-bang.
- Planejar transição reversível mantendo consumidores funcionando.
- Qualquer mudança grande que se beneficie de decompor em elos encadeados.

## Como usar a cadeia
1. Rode o **Prompt 1** com `estado_atual` → produz `diagnostico`.
2. Rode o **Prompt 2** com `diagnostico` + `restricoes` → produz `plano_etapas`.
3. Rode o **Prompt 3** com `plano_etapas` → detalha a Etapa 1 executável.

Os três elos vivem em `prompt.md` (mesma unidade de uso). Fixtures em `fixtures/`.

## Exemplo
`estado_atual` = `fixtures/estado-atual.txt` (Forge, Checkpoint 05). Modelo: **Claude
Sonnet 4.6**. A cadeia produz: diagnóstico (pontos presos a "lote fechado", riscos por
dependente, pré-condições) → 5 etapas reversíveis (shadow ingestion → paridade →
Sentinel → Cerebro → billing por último) → detalhamento da Etapa 1 com critério de
sucesso (<0,1% de diff por 24h) e rollback (desligar o consumer, zero impacto).

## Limitações
- Saída aberta — qualidade checada por LLM-as-judge.
- A qualidade do elo N depende do elo N-1; um diagnóstico raso contamina o plano.

## Curadoria
Técnica: **prompt chaining**. Pedir diagnóstico + plano + execução num prompt só gerava
resposta genérica ("migre em fases, valide, finalize"). Quebrando em 3 elos, cada
resposta ficou ancorada na anterior e bem mais concreta.
