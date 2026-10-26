# Fase 2: o CortexBot como runner nativo do Cortex Execution

Depois da fase 1, o Symph sai do meio. O CortexBot aparece no painel Agentes como runner próprio
(`runner.kind = "cortex-bot"`) e segue o protocolo de runner do Cortex
(`cortex-execution/docs/api/runner-protocol.md`) com uma ponte nossa.

## Por que uma ponte e não uma rotina do OpenBot

As Routines do OpenBot têm intervalo mínimo de 15 minutos e um teto de 20 ativas por pessoa
(`docs/routines.md`), de propósito. Isso é lento demais para puxar trabalho, e o protocolo de runner
pede sessão aberta com heartbeat. Por isso o laço fica num processo pequeno ao lado do OpenBot, em
`cortex/bridge/`, e a Responsibility com gatilho webhook continua sendo a porta de entrada do trabalho.
Nenhum código do OpenBot muda.

## A ponte (`cortex/bridge/`)

Um serviço Bun/TypeScript, na mesma stack do OpenBot, com um laço:

| # | Passo | Cortex |
|---|---|---|
| 1 | Abrir sessão com `runner: { kind: "cortex-bot", name: "CortexBot", scope: "shared" }` e o catálogo de Bots como `agents` | `POST /api/sessions` |
| 2 | Ler a fila | `GET /api/tasks/dispatchable?delegated=true&runner=cortex-bot` |
| 3 | Ler o briefing | `GET /api/tasks/:id/briefing` |
| 4 | Reivindicar com `expectedTaskHash` e `agent` | `POST /api/tasks/:id/claim` |
| 5 | Disparar o gatilho do Bot escolhido | `POST <CortexBot>/api/events/triggers/:triggerId` |
| 6 | Heartbeat enquanto o run dura (cancelamento vindo do Cortex encerra o run) | `POST /api/sessions/:id/heartbeat` |
| 7 | Nota de handoff e estado | `POST /api/tasks/:id/notes`, `PATCH /api/tasks/:id` |
| 8 | Custo e resultado | `POST /api/tasks/:id/executions` |
| 9 | Liberar ou concluir, e encerrar | `POST /api/tasks/:id/release`, `DELETE /api/sessions/:id` |

- **Catálogo:** cada Bot do `examples/cortex/agents.yaml` com Responsibility de execução vira um
  perfil (`profileId` = id do Bot, `provider` = `"cortex-bot"`, `model` = o do `model.yaml`).
- **Credencial:** token de runner da Conta (`cortex token create --name "runner:cortex-bot" --caps
  runner --project <slug>`), em `CORTEX_API_TOKEN` no `.env` da ponte. Nunca com `execution.authorize`.
- **Tetos:** respeitar os limites que vêm em `execution` na fila (tempo, custo). O run que passa do
  teto é encerrado e a Task é devolvida com o motivo.

## MCP do Cortex dentro do Bot

Para o Bot ler contexto (outras Tasks do Project, notas, artefatos) e anexar entregas durante o run,
o MCP do Cortex entra como servidor MCP próprio no OpenBot, cadastrado pela tela de administração
(sem tocar no catálogo do código deles), com credencial `deployment-bearer` = token de runner. Só as
tools de leitura e de nota ficam liberadas na política. Claim, release e execução continuam com a
ponte, para que o estado da Task tenha um dono só.

## Custo por execução

O OpenBot não mede tokens. Opções, da mais simples à mais limpa:

1. **Proxy de modelo na ponte.** O `model.yaml` aponta para um endpoint compatível com OpenAI
   servido pela ponte, que repassa ao provedor real e soma `usage` por run. Não muda o OpenBot.
2. **PR para a CopilotKit** expondo o uso por run no histórico da Responsibility. Serve a qualquer
   usuário do OpenBot, e o fork não carrega a mudança.

Começar pela 1 e propor a 2.

Lembrete do lado do Cortex: execução sem custo informado não consome cota. O CortexBot só entra de
verdade na cobrança de agentes quando reportar custo.

## Pronto quando

O CortexBot aparece no painel Agentes como runner próprio, recebe Task arrastada, executa sem o Symph
e grava uma execução com tokens e custo. A cota da Conta no Cortex desconta o run.
