# Arquitetura

## As peças

| Peça | Papel | Repositório |
|---|---|---|
| **Cortex Execution** | Fonte do que fazer: Tasks, aceite, quem pode delegar, fila, custo por execução | `cortex-execution` |
| **Cortex Symph** | Runner de código: puxa Tasks delegadas, roda Claude Code, Codex, Gemini, OpenCode numa worktree e abre PR | `cortex-symph` |
| **CortexBot** | Runner de trabalho que não é código: browser, e-mail, documentos, apps conectados | este |

A divisão é por tipo de trabalho, não por modelo:

- **Symph:** Tasks que terminam em PR (worktree, testes, CI).
- **CortexBot:** Tasks que terminam num documento, numa mensagem enviada, num formulário preenchido,
  num relatório ou numa triagem. Exemplos: pesquisar um fornecedor, montar o resumo semanal, responder
  um lead, conferir um cadastro num sistema web.

## O que o OpenBot já dá

Estas peças são do OpenBot e não precisamos escrever:

- **Bot como endpoint AG-UI.** Qualquer framework (LangGraph, Mastra, CrewAI, Pydantic AI, Claude
  Agent SDK…) ou um Bot "built-in" descrito só em YAML.
- **Computador por Bot.** Um container com Chromium, arquivos e logins, criado pelo `supervisor/`
  (`agent-computer/`).
- **Gateway de ações.** Toda ação de browser, arquivo, shell ou MCP é decidida por política CEL
  (falha fechada) e auditada antes e depois (`docs/architecture.md`, "Browser action governance").
- **Quem disparou o run.** Cada linha de auditoria grava `initiator_kind` (`person`, `routine`,
  `responsibility`, `handoff`…). É assim que separamos o que rodou sem ninguém olhando.
- **Responsibilities com gatilho.** Um trabalho fixo de um Bot, disparado por evento externo:
  `webhook` (assinado no padrão Standard Webhooks, com idempotência por `webhook-id`), `github`,
  `linear`, `sentry`, `pagerduty`, `email`, `slack` (`server/src/responsibilities/`). A entrada é
  `POST /api/events/triggers/:triggerId`, e o resultado fica no histórico de runs da Responsibility.
- **Pedir para uma pessoa.** O Bot pausa e pergunta quando chega a algo que não deve fazer sozinho.
- **MCP como plugin.** Servidor MCP próprio, com token de implantação (`deployment-bearer`) ou OAuth
  por pessoa (`user-oauth`), e toda chamada passa pelo mesmo gateway.

## O que falta e é nosso

1. **Ponte com a fila do Cortex.** Alguém tem de ler as Tasks delegadas e entregá-las ao Bot. Na fase 1
   é o Symph ([fase-1-symph.md](fase-1-symph.md)). Na fase 2 é uma ponte própria, `cortex/bridge`
   ([fase-2-nativo.md](fase-2-nativo.md)).
2. **Devolver a entrega na Task.** Nota de handoff, link do artefato, estado final.
3. **Custo por execução.** O OpenBot não mede tokens nem custo por run. Sem isso, a cota de agentes do
   Cortex não enxerga o CortexBot.
4. **Pacote do Cortex.** Nome, Bots, skills e o MCP do Cortex em `examples/cortex/`.

## Fluxo, fase 1

```
Cortex Execution ──(Task delegada ao perfil cortex-bot)──▶ Symph
Symph ──(webhook assinado com o briefing)──▶ CortexBot: Responsibility "Executar Task do Cortex"
CortexBot ──(run no computador do Bot, com gateway e auditoria)──▶ resultado no histórico de runs
Symph ──(lê o run, grava nota, execução e estado)──▶ Cortex Execution
```

## Fluxo, fase 2

```
Cortex Execution ◀──(protocolo de runner: sessão, fila, claim, heartbeat)── cortex/bridge
cortex/bridge ──(webhook assinado)──▶ CortexBot: Responsibility
CortexBot ──(MCP do Cortex: notas, artefatos)──▶ Cortex Execution
cortex/bridge ──(POST /executions com tokens e custo)──▶ Cortex Execution
```

## Regras que valem nas duas fases

- **Quem executa nunca autoriza.** Delegar é decisão humana no Cortex. O CortexBot só usa credencial
  de runner (preset `runner`), nunca uma que tenha `execution.authorize`.
- **A política do OpenBot é a borda do que o Bot faz; o Cortex é a borda do que ele recebe.** Uma não
  substitui a outra.
- **Nada de segredo no repositório.** O token de runner do Cortex e o segredo do webhook vivem no
  `.env` da implantação.
