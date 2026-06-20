---
nome: Endurecimento de NetworkPolicy
descricao: Recebe um manifesto de NetworkPolicy permissivo, as regras do padrão e o mapa de serviços, e devolve a versão endurecida (default-deny + regras mínimas comentadas).
versao: 1.0.0
tags: [devops, kubernetes, seguranca, networkpolicy]
inputs:
  - nome: manifesto
    descricao: Manifesto de NetworkPolicy permissivo (allow-all) que precisa ser corrigido.
  - nome: regras
    descricao: Regras do padrão de segurança que a versão corrigida precisa cumprir.
  - nome: mapa_servicos
    descricao: Mapa de cada serviço com namespace, labels e portas.
---

## Objetivo
Transformar um manifesto de NetworkPolicy permissivo (allow-all) na versão endurecida
do padrão Aegis: default-deny explícito + ingress/egress restritos exatamente ao mapa
de serviços, cada regra comentada com o fluxo legítimo que libera.

## Casos de uso
- Corrigir um manifesto barrado pela revisão de segurança.
- Gerar a NetworkPolicy de um namespace novo a partir do mapa de serviços.
- Endurecer políticas existentes antes de virar item de playbook.

## Exemplo e iterações (verificação + refino — Checkpoint 06)
Parâmetros em `fixtures/` (`manifesto-permissivo.yaml`, `regras.txt`,
`mapa-servicos.txt`). Modelo: **Claude Sonnet 4.6**.

- **v1:** gerou default-deny + regras por `podSelector`, mas **sem `namespaceSelector`**.
- **Verificação (self-critique como revisor de segurança):** apontou que `podSelector`
  sozinho restringe ao mesmo namespace; como Relay/Forge/Cerebro/etc. estão em
  namespaces diferentes de `sentinel-prod`, **nenhuma regra de fato liberava o tráfego
  pretendido** — virava um default-deny disfarçado.
- **v2:** combinou `namespaceSelector` + `podSelector` em cada regra, com as portas
  5432 (Forge) e 9200 (Cerebro) e DNS na 53. Segunda rodada de verificação não achou
  problema novo. Fechado na v2.

## Limitações
- O modelo precisa do mapa de serviços correto; sem ele, inventa labels.
- Não valida a política contra o cluster real (não aplica `kubectl`); a verificação é
  semântica, não um teste de conectividade.

## Curadoria
Técnica de verificação: **self-critique guiado** — não pedir "revise" e sim "assuma o
papel de revisor de segurança e levante as perguntas que esse papel faria". Foi isso
que trouxe à tona o `namespaceSelector` ausente; sem o papel explícito, a revisão só
checava sintaxe YAML, não semântica de rede.
