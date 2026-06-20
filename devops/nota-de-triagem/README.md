---
nome: Nota de triagem
descricao: Transforma um alerta cru do Sentinel em uma nota de triagem padronizada de 5 campos (ALERTA, IMPACTO, HIPÓTESE INICIAL, AÇÃO IMEDIATA, ESCALAR PARA).
versao: 1.0.0
tags: [devops, sre, plantao, incidentes]
inputs:
  - nome: alerta_cru
    descricao: Texto cru do alerta disparado pelo Sentinel (sistema, condição, tenant, métricas), exatamente como chega no plantão.
---

## Objetivo
Padronizar a nota de triagem que o plantonista abre a cada alerta, para que quem
assume o turno seguinte leia sempre o mesmo formato de 5 campos, sem cada um escrever
do seu jeito.

## Casos de uso
- Autoscaler no limite com fila crescendo.
- Taxa de rejeição de ingestão acima do limite após deploy.
- Lag de consumer subindo e impactando dashboards downstream.

## Exemplo
Parâmetro `alerta_cru` = Entrada 1 do Checkpoint 02. Modelo: **GPT-4o-mini**.

```
ALERTA: Sentinel - autoscaler no limite (60/60) com fila do Relay crescendo
IMPACTO: risco de atraso nos alertas do Sentinel para todos os tenants, fila +2k/min
HIPÓTESE INICIAL: tenant stark-industries enviando 4x o volume base após onboarding de nova região
AÇÃO IMEDIATA: avaliar aumento do limite de réplicas do autoscaler do sentinel-api
ESCALAR PARA: @sentinel-core se a fila não estabilizar em 15min
```

## Limitações
- Infere a hipótese inicial só do que está no alerta; não substitui investigação.
- Se o alerta cru vier muito incompleto, o campo HIPÓTESE INICIAL fica genérico.

## Curadoria
Escolha de método: **zero-shot com regra de formato explícita** em vez de few-shot.
Colar os três exemplos de nota boa arriscava a IA "copiar" frases deles (Argo CD,
shard quente) em vez de generalizar pro alerta real. Descrevi os 5 campos + as regras
e validei nos três alertas crus — formato correto e consistente sem exemplo colado.
