# Rodar o CortexBot

O passo a passo base é o do OpenBot ([README](../../README.md#quick-start)). Aqui fica só o que muda
para o CortexBot e o que costuma dar errado.

## Pré-requisitos

- Docker, Bun 1.3+.
- **Conta no CopilotKit Intelligence.** É obrigatória: o servidor não sobe sem `INTELLIGENCE_API_URL`,
  `INTELLIGENCE_GATEWAY_WS_URL` e `INTELLIGENCE_API_KEY` (`docs/configuration.md`). O plano
  Developer é gratuito. Há também uma versão local em Docker, só para macOS, com licença de 30 dias
  renovável.
- Uma chave de modelo. O pacote `examples/cortex/` usa Anthropic (`ANTHROPIC_API_KEY`). O pacote de
  exemplo deles usa OpenAI (`OPENAI_API_KEY`). Os dois são os únicos provedores que o `model.yaml`
  aceita.

## Local

```sh
cp .env.example .env
bun install
npx --yes copilotkit@latest login
npx --yes copilotkit@latest project select
bun scripts/setup-learning.ts
```

No `.env`:

```sh
TENANT_PACKAGE_DIR=../examples/cortex   # resolvido a partir de server/
ANTHROPIC_API_KEY=...                   # ou cadastre a chave na administração, que tem prioridade
OPENBOT_SELF_HOST_BANNER=false
```

Depois:

```sh
bash scripts/start.sh    # Docker, migrations, API :3001, worker, app :3010
```

Abra <http://localhost:3010>. `bash scripts/stop.sh` derruba tudo, sem apagar volumes.

Sem `TENANT_PACKAGE_DIR`, o servidor sobe com o pacote de exemplo deles (`examples/fintech`), que
serve para conhecer o produto.

## O pacote `examples/cortex/`

| Arquivo | O que tem |
|---|---|
| `brand.yaml` | `tenant.id: cortex`, `product_name: CortexBot` |
| `agents.yaml` | Um Bot built-in, `executor-cortex` ("Executor do Cortex"), com instruções em português e o formato de entrega |
| `skills.yaml` | `entregar-task-cortex` (Feito, Não feito, Critério de aceite) e `pesquisar-na-web` (tools `@research/*`) |
| `channels.yaml` | Canal `execucao-cortex`, só com o Executor |
| `model.yaml` | Anthropic, `claude-sonnet-5-5` |
| `knowledge.yaml` | Vazio. O arquivo é obrigatório, mas nada no servidor o lê hoje |
| `agents/` | Um Bot por arquivo, para os próximos |

Para conferir o pacote sem subir o ambiente, chame `loadTenantPackage` de `server/src/tenant-package.ts`
com o caminho do pacote. Ela recusa o mesmo que o servidor recusaria ao iniciar.

### Política inicial

A política de ações não faz parte do pacote. Ela vem de `AGENT_COMPUTER_POLICY` no `.env` (JSON
`{"mode":"enforce","deny":[...],"allow":[...]}`, regras em CEL) ou da política salva pela
administração, que tem prioridade. O padrão do OpenBot permite tudo (`allow: ["true"]`).

Para o Executor do Cortex, comece restrito:

- **Aprovações:** em `/bots/executor-cortex/approvals`, faça o Bot perguntar antes de shell e de
  qualquer envio (e-mail, mensagem, formulário). Browser e arquivos ficam liberados.
- **Política:** se precisar barrar algo de vez, use `deny` com `bot.id == "executor-cortex"` e o campo
  que importa (`tool.name`, `page.host`, `mcp.server`). Os campos estão em `docs/architecture.md`,
  "Browser action governance". Comece em `"mode": "dry-run"` para ver o que a regra pegaria sem
  bloquear nada.

Solte uma restrição só quando os runs na auditoria mostrarem que ela é desnecessária.

## VPS

O OpenBot publica uma imagem só, com app, API, navegador e Postgres embutido opcional
(README, "Deploy it"). Para a VPS:

1. Mesmo `.env` do local, com `OPENBOT_SINGLE_USER` desligado e login configurado (README,
   "Sign in"), porque a VPS é alcançável por outras pessoas.
2. `KEY_ENCRYPTION_KEY`, `COMPUTER_TOKEN` e os tokens do supervisor gerados de verdade
   (`openssl rand -base64 32`). Os do `.env.example` são públicos.
3. HTTPS na frente (o gatilho webhook recebe chamadas de fora).
4. Na fase 1, a URL do gatilho vai no perfil `cortex-bot` do Symph. Na fase 2, a ponte roda ao lado,
   com `CORTEX_API_URL` e `CORTEX_API_TOKEN` (token de runner).

Mesma VPS do Symph é possível. Cada Bot com computador próprio sobe um container com Chromium, então
conte memória por Bot ativo.

## O que costuma dar errado

- **Servidor recusa iniciar:** falta uma das chaves `INTELLIGENCE_*`, ou o pacote de tenant tem campo
  inválido. A mensagem de log diz qual.
- **Intelligence local:** o comando `npx copilotkit@latest local connect` grava a chave como
  `CPK_INTELLIGENCE_API_KEY`, mas o OpenBot lê `INTELLIGENCE_API_KEY`. Copie o valor.
- **Gatilho responde 503:** o gatilho ainda espera o segredo. Responde 409 quando a Responsibility
  está pausada.
