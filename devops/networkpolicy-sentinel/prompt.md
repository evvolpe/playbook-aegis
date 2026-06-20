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
    descricao: Mapa de cada serviço com namespace, labels e portas, para escrever os seletores corretos sem inventar labels.
---

Você é um engenheiro de segurança endurecendo uma NetworkPolicy de Kubernetes.

Manifesto permissivo a corrigir:
{{manifesto}}

Regras que a versão corrigida precisa seguir:
{{regras}}

Mapa de serviços (namespace, labels, portas):
{{mapa_servicos}}

Tarefa: produza a NetworkPolicy corrigida em YAML, com:
- política default-deny explícita no namespace
- ingress e egress restritos exatamente ao mapa de serviços, nunca com `{}` vazio
- um comentário (#) em cada regra dizendo qual fluxo legítimo ela libera
- nada de allow-all

Devolva só o YAML final, comentado.
