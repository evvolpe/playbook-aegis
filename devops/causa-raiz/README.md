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

## Objetivo
Levar a IA a raciocinar de ponta a ponta sobre uma degradação — cruzando config,
métricas e logs — até a causa-raiz, separando o gatilho dos efeitos, em vez de listar
sintomas. É o prompt que o time reusa toda vez que uma degradação aparece, trocando só
o pacote de entrada.

## Casos de uso
- Degradação de latência de busca com saturação de heap/JVM.
- Job concorrente (reindexação) consumindo recurso e disparando circuit breaker.
- Qualquer incidente em que a causa só aparece cruzando 3 fontes distintas.

## Exemplo
Parâmetros = `fixtures/cerebro.yaml` + `fixtures/metricas.txt` + `fixtures/logs.txt`
(degradação do Cerebro, Checkpoint 03). Modelo: **Claude Sonnet 4.6**. Saída resumida:

> Causa-raiz: a reindexação agendada para 02:00 não terminou (41% às 10:00),
> consumindo heap continuamente até saturar o teto de 8gb e disparar o circuit
> breaker. Queda do cache_hit_pct e timeouts de busca são **consequência**, não causa.
> Ação: pausar/reagendar a reindexação e investigar por que está ~5x mais lenta — não
> subir o heap como primeira ação (mascara o problema).

## Tratamento de dado sensível
Antes de mandar para um provedor externo, troco identificadores que amarrem a
cliente/infra real (ex.: `cerebro-node-3` → `node-A`), mantendo só o que importa pro
raciocínio técnico. Os dados deste exemplo são fictícios.

## Limitações
- Saída aberta — não verificável por regex; a qualidade é checada pelo LLM-as-judge
  (ver `promptfooconfig.yaml`).
- Não tem visibilidade dos outros nós do cluster; conclui sobre o nó observado.

## Curadoria
Técnica: **cadeia de raciocínio guiada por seções**. Pedir explicitamente "separe
causa de consequência" foi o que impediu a IA de listar a queda do cache como "causa".
A seção "o que não dá pra afirmar" só apareceu depois de pedir honestidade epistêmica
de forma explícita.
