---
nome: Análise de causa-raiz
descricao: Recebe config, métricas e logs de uma degradação em produção e leva a IA a raciocinar até a causa-raiz, separando causa de sintoma.
versao: 1.0.0
tags: [devops, sre, observabilidade, causa-raiz]
inputs:
  - nome: config
    descricao: "Arquivo de configuração do cluster/serviço degradado (ex.: cerebro.yaml)."
  - nome: metricas
    descricao: Série de métricas da janela do incidente (latência, vazão, heap, cache, etc.).
  - nome: logs
    descricao: Trecho dos logs do nó cobrindo a mesma janela das métricas.
---

Você é um engenheiro de plataforma investigando a causa-raiz de uma degradação em
produção. Não liste sintomas como se fossem causa — cruze os artefatos até chegar à
causa real.

CONFIGURAÇÃO do cluster:
{{config}}

MÉTRICAS da janela do incidente:
{{metricas}}

LOGS do nó na mesma janela:
{{logs}}

Tarefa:
1. Monte a linha do tempo da degradação cruzando métricas e logs.
2. Separe causa de consequência (o que disparou a degradação vs. o que é efeito dela).
3. Aponte a causa-raiz mais provável, citando evidências específicas (linha de log,
   valor de métrica) que a sustentam.
4. Proponha uma ação proporcional ao diagnóstico — nem mitigação superficial, nem
   reescrita do sistema.
5. Diga explicitamente o que os dados NÃO permitem concluir com certeza.

Responda em 4 seções: Linha do tempo / Causa-raiz / Ação recomendada / O que não dá
pra afirmar.
