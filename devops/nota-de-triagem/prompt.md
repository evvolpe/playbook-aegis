---
nome: Nota de triagem
descricao: Transforma um alerta cru do Sentinel em uma nota de triagem padronizada de 5 campos (ALERTA, IMPACTO, HIPÓTESE INICIAL, AÇÃO IMEDIATA, ESCALAR PARA).
versao: 1.0.0
tags: [devops, sre, plantao, incidentes]
inputs:
  - nome: alerta_cru
    descricao: Texto cru do alerta disparado pelo Sentinel (sistema, condição, tenant, métricas), exatamente como chega no plantão.
---

Você é o plantonista do Sentinel e precisa abrir uma nota de triagem padronizada a
partir de um alerta cru.

Alerta cru recebido:
{{alerta_cru}}

Gere a nota seguindo EXATAMENTE este formato, um campo por linha, sem texto antes ou
depois dos campos:

ALERTA: <sistema + condição que disparou, resumida>
IMPACTO: <quem/o que é afetado e em que grau>
HIPÓTESE INICIAL: <causa mais provável, inferida só do que está no alerta>
AÇÃO IMEDIATA: <primeira ação que o plantonista deve tomar agora>

Regras:
- No máximo 8 linhas no total.
- Não invente dado que não está no alerta cru; se faltar informação, generalize sem
  fingir certeza.
- Use os termos técnicos que já aparecem no próprio alerta (sistema, tenant, métrica).
