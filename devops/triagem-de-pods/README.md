---
nome: Triagem de pods
descricao: Analisa um snapshot de cluster Kubernetes (pods + eventos + logs) e aponta quais pods estão problemáticos, com causa provável e ação recomendada.
versao: 1.0.0
tags: [devops, kubernetes, sre, triagem]
inputs:
  - nome: snapshot
    descricao: Saída combinada de `kubectl get pods`, `kubectl describe pod` e `kubectl logs` dos pods relevantes do namespace.
---

## Objetivo
Dar ao plantonista de SRE uma triagem confiável e rápida da saúde dos pods de um
namespace, sem depender de quem está de plantão saber interpretar describe/logs de
cabeça. Cruza STATUS + eventos + logs para chegar à causa provável, em vez de repetir
o STATUS.

## Casos de uso
- Pod em CrashLoopBackOff por OOM.
- Pod sem subir por ImagePullBackOff ou falta de recursos no cluster.
- Confirmação de que o cluster está saudável (nenhum falso positivo).

## Exemplo
Parâmetro `snapshot` = `fixtures/entrada-1.txt` (pod reiniciando). Modelo: **Claude
Sonnet 4.6**.

```
Resumo: 1 pod problemático de 4.

Pod: sentinel-api-7d9c8b6f4-h4m2t
Causa provável: OOMKilled — o container excedeu o limite de memória (512Mi).
Evidência: log mostra heap subindo de 410Mi para 498Mi/512Mi com "high GC pressure"
antes do "[FATAL] out of memory"; describe confirma Last State Terminated/OOMKilled,
exit code 137, e o Request (256Mi) está bem abaixo do Limit (512Mi).
Ação recomendada: aumentar o limite de memória do container ou investigar por que o
cache de alertas (6h) está consumindo tanta memória — pode ser caso de reduzir a
janela de cache antes de subir o limite.
```

As três entradas de teste estão em `fixtures/` (`entrada-1.txt`, `entrada-2.txt`,
`entrada-3.txt`) e são as do Checkpoint 01.

## Limitações
- Depende de quem coletou o snapshot ter pego o `describe`/`logs` do pod certo — o
  prompt não vai buscar mais contexto se a evidência fornecida for insuficiente.
- Não substitui investigação mais profunda em causas raras não cobertas pelos
  exemplos de calibração (ex.: falhas de rede entre nós).

## Curadoria
Framework: **role + contexto + tarefa + formato de saída** (sem few-shot — o problema
é bem definido pela própria estrutura do snapshot). O primeiro rascunho repetia o
STATUS como se fosse causa ("está em CrashLoopBackOff porque está em CrashLoopBackOff");
a instrução explícita "não confie só na coluna STATUS, cruze com eventos e logs" e o
caso "nada problemático" resolveram nos três exemplos.
