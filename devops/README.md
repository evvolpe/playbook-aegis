# devops/

Categoria de prompts de operação da plataforma Aegis (SRE, observabilidade, dados,
segurança). Um prompt por pasta, nomeada pelo resultado — não pela técnica.

| Pasta | Objetivo | Parâmetros (`inputs`) | Versão |
|---|---|---|---|
| [triagem-de-pods](triagem-de-pods/) | Triagem de saúde de pods a partir de um snapshot | `snapshot` | 1.0.0 |
| [nota-de-triagem](nota-de-triagem/) | Nota de triagem padronizada de 5 campos | `alerta_cru` | 1.0.0 |
| [causa-raiz](causa-raiz/) | Causa-raiz de degradação cruzando config/métricas/logs | `config`, `metricas`, `logs` | 1.0.0 |
| [backpressure-relay](backpressure-relay/) | Comparar estratégias de backpressure antes de recomendar | `cenario` | 1.0.0 |
| [migracao-forge](migracao-forge/) | Cadeia de migração batch → event-driven | `estado_atual`, `diagnostico`, `restricoes`, `plano_etapas` | 1.0.0 |
| [networkpolicy-sentinel](networkpolicy-sentinel/) | Endurecer NetworkPolicy permissiva (default-deny) | `manifesto`, `regras`, `mapa_servicos` | 1.0.0 |
