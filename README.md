# Playbook de IA Operacional — Aegis

Biblioteca de prompts versionada, parametrizável e testada em pipeline, no formato
[prompt-registry](https://github.com/fabricioveronez/prompt-registry). Cada prompt é
um item reusável que recebe os dados variáveis por parâmetro (`{{placeholder}}`) e
viaja junto com seu teste (`promptfooconfig.yaml`) e sua documentação (`README.md`).

> Este repositório **é** o playbook — não um documento que descreve o playbook.

## Estrutura

```
devops/                          # categoria (kebab-case, uma por domínio)
  <prompt>/                      # um prompt por pasta, nomeada pelo resultado
    prompt.md                    # frontmatter + texto do prompt com {{placeholders}}
    README.md                    # frontmatter + objetivo, casos de uso, exemplo, limitações
    promptfooconfig.yaml         # testes que viajam junto com o prompt
    fixtures/                    # dados de entrada dos exemplos
.github/workflows/
  promptfoo-eval.yml             # roda a suíte a cada PR/push, barra regressão
```

## Prompts (`devops/`)

| Prompt | Origem | Saída | Teste |
|---|---|---|---|
| [triagem-de-pods](devops/triagem-de-pods/) | CP01 | estruturada | determinístico |
| [nota-de-triagem](devops/nota-de-triagem/) | CP02 | estruturada | determinístico |
| [causa-raiz](devops/causa-raiz/) | CP03 | aberta | LLM-as-judge |
| [backpressure-relay](devops/backpressure-relay/) | CP04 | aberta | LLM-as-judge |
| [migracao-forge](devops/migracao-forge/) | CP05 | aberta (cadeia) | LLM-as-judge |
| [networkpolicy-sentinel](devops/networkpolicy-sentinel/) | CP06 | estruturada | determinístico |

Versionamento pelo campo `versao` (semver) de cada prompt — todos nascem em `1.0.0`.
Mudanças entram por commit semântico com o escopo na categoria, ex.:
`feat(devops): adiciona prompt de triagem de pods`.

## Como rodar os testes localmente

```bash
# chaves dos provedores (não versionar)
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...

# um prompt
npx promptfoo@latest eval -c devops/nota-de-triagem/promptfooconfig.yaml

# a suíte inteira
for c in devops/*/promptfooconfig.yaml; do npx promptfoo@latest eval -c "$c"; done
```

## Escolha de modelo (consciência de custo e latência)

Os limites operacionais fazem parte da qualidade: cada chamada determinística tem
`latency <= 5s` e `cost <= US$ 0,01`. Por isso os prompts de saída estruturada rodam
em **`gpt-4o-mini`** e **`claude-haiku-4-5`** (mini/haiku dos dois fornecedores) — não
precisam do raciocínio de um modelo "grande" e um modelo caro reprovaria no assert de
custo sem ganho real. Os prompts de saída aberta geram em **`claude-sonnet-4-6`**
(vale o modelo mais forte porque viram insumo de decisão) e são julgados por
**`gpt-4o`** — juiz de **outro fornecedor**, para evitar viés de auto-aprovação.

## Testes e pipeline (Checkpoints 08–10)

### Camada determinística (CP08)
`triagem-de-pods`, `nota-de-triagem` e `networkpolicy-sentinel` checam formato,
conteúdo, ausência de allow-all, latência e custo via `contains`/`regex`/`not-contains`/
`javascript`/`latency`/`cost`. Sem julgamento humano.

**Resultado real (`promptfoo eval`, 2 providers):**

| Prompt | gpt-4o-mini | claude-haiku-4-5 |
|---|---|---|
| nota-de-triagem (3 casos) | ✅ 3/3 | ✅ 3/3 |
| triagem-de-pods (3 casos) | ✅ 3/3 | ✅ 3/3 |
| networkpolicy-sentinel (1 caso) | ❌ latência ~5,2s (ver pasta) | ✅ ~3,1s |

**O que foi ajustado no caminho (o desafio pede "mostre o caminho"):**
- *triagem / Entrada 3:* na 1ª execução os dois modelos marcaram um pod saudável como
  problemático por causa de `RESTARTS 1 (3d ago)`. Refinei o prompt (restart antigo
  isolado não é falha) e fixei o formato de saída em `Resumo: N pods problemáticos`,
  deixando o teste determinístico. Troquei o assert frágil `not-contains: CrashLoopBackOff`
  por um que só reprova quando há ≥1 pod declarado problemático.
- *latência:* as saídas em tabela markdown estouravam 5s; forcei texto puro conciso e a
  latência caiu para <3,5s (e o custo junto).
- *networkpolicy / gpt-4o-mini:* trade-off de latência mantido de propósito — ver a
  curadoria da [pasta](devops/networkpolicy-sentinel/).

### Gate de qualidade — LLM-as-judge (CP09)
A `causa-raiz` tem saída aberta; o gate é um juiz `llm-rubric` aplicando esta rubrica
(4 critérios, 0–2 cada, total 0–8; aprova com **total ≥ 6 e nenhum critério zerado**):

| Critério | 0 | 1 | 2 |
|---|---|---|---|
| Causa-raiz correta | não identifica a reindexação travada | cita reindex OU heap, sem ligar | cadeia completa: reindex → heap → circuit breaker → timeouts |
| Correlação × causa | trata sintoma (cache) como causa | mistura sem separar | separa efeito (cache, timeouts) de causa (reindex/heap) |
| Ação proporcional | genérica/desproporcional | correta mas vaga | concreta e proporcional (pausar/reagendar reindex) |
| Honestidade epistêmica | afirma certeza além dos dados | reconhece de forma genérica | reconhece especificamente o que os dados não permitem |

**Calibração do juiz** (pontuei manualmente e comparei com o juiz, exigindo ≤ 1 ponto
de diferença por critério):

| Saída | Minha nota | Juiz | Δ |
|---|---|---|---|
| Boa (CP03 original) | 8 | 8 | 0 |
| Fraca (cache como causa) | 4 | 3 | 1 |
| Fraca (sem reconhecer incerteza) | 5 | 5 | 0 |
| Boa (ação mais vaga) | 6 | 7 | 1 |

Diferença máxima de 1 ponto por critério → juiz considerado calibrado. O que destravou
a calibração foi escrever o `rubricPrompt` citando os fatos do cenário (reindex 02:00,
41%, heap, circuit breaker) em vez de uma rubrica genérica.

**Execução real do gate (`promptfoo eval`):** sobre a saída real da `causa-raiz`
(geração `claude-sonnet-4-6`), o juiz `gpt-4o` deu **8/8 → PASS**. Dois bugs reais
apareceram e foram corrigidos para o gate funcionar: (1) o `rubricPrompt` precisava
mandar o juiz responder em **JSON** `{reason, pass, score}` — sem isso o promptfoo não
extraía a nota; (2) o `max_tokens` default (1024) **truncava** a geração do sonnet, o
que reprovava `backpressure` (parava em 2 estratégias) e `migracao` (parava antes do
plano reversível). Com `max_tokens: 4096`, os três passam **8/8**. Saídas em
[`evidencias/`](evidencias/).

### Pipeline e estratégia de gate (CP10)
Cobertura estendida a **todos** os prompts (`backpressure-relay` e `migracao-forge`
ganharam juiz no mesmo molde). O workflow [`promptfoo-eval.yml`](.github/workflows/promptfoo-eval.yml)
roda a cada PR e push. Decisões de design, com alternativas:

**O que falha o build:** assert determinístico reprovado → bloqueia sempre (barato,
sem ruído). `llm-rubric` reprovado → bloqueia também, mas o corte (≥ 6, nenhum zerado)
tem folga para absorver a variância do juiz (a calibração mostrou ≤ 1 ponto de
flutuação). Custo de falso-negativo = "rodar de novo"; custo de falso-positivo =
confiança quebrada no playbook.
- *Alternativa descartada:* juiz só informativo (comenta, não bloqueia) → zero falso
  bloqueio, mas "nenhuma alteração entra sem passar nos testes" deixa de valer
  justamente para os prompts mais arriscados de regredir silenciosamente.

**Escopo de execução:** no PR roda só os configs alterados (`tj-actions/changed-files`)
→ tempo e custo de token proporcionais ao PR, não à biblioteca.
- *Perda:* mudança "lateral" (fixture compartilhado) pode escapar do filtro de path.
- *Mitigação / alternativa:* a suíte **inteira** roda no `push` para `main` (job
  `eval-full`), como rede de segurança fora do caminho crítico do PR. Rodar tudo em
  todo PR seria mais seguro, mas caro e lento à medida que a biblioteca cresce, e a
  maioria dos PRs altera um prompt só.

**Secrets:** `OPENAI_API_KEY` e `ANTHROPIC_API_KEY` ficam em *repository secrets* do
GitHub, nunca versionados. A `promptfoo-action` não expõe secrets para PRs de fork sem
permissão explícita, o que também limita quem consegue gastar o orçamento de API.

## Sanitização de dado sensível
Os cenários são fictícios, mas vários trazem o que, em produção, seriam dados sensíveis
(hostname interno, nome de tenant, identificador). Antes de mandar para um provedor
externo, troco identificadores que amarrem a cliente/infra real por genéricos
(`cerebro-node-3` → `node-A`), mantendo só o que importa para o raciocínio técnico.
