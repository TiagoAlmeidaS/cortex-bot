# Rodar o CortexBot

O passo a passo base é o do OpenBot ([README](../../README.md#quick-start)). Aqui fica só o que muda
para o CortexBot e o que costuma dar errado.

## Pré-requisitos

- Docker, Bun 1.3+.
- **Conta no CopilotKit Intelligence.** É obrigatória: o servidor não sobe sem `INTELLIGENCE_API_URL`,
  `INTELLIGENCE_GATEWAY_WS_URL` e `INTELLIGENCE_API_KEY` (`docs/configuration.md`). O plano
  Developer é gratuito. Há também uma versão local em Docker, só para macOS, com licença de 30 dias
  renovável.
- Uma chave de modelo. O pacote de exemplo usa OpenAI. O Anthropic também está entre os provedores
  conhecidos (`shared/model-providers.json`).

## Local

```sh
cp .env.example .env
bun install
npx --yes copilotkit@latest login
npx --yes copilotkit@latest project select
bun scripts/setup-learning.ts
```

No `.env`, além da chave de modelo:

```sh
TENANT_PACKAGE_DIR=../examples/cortex   # resolvido a partir de server/
OPENBOT_SELF_HOST_BANNER=false
```

Depois:

```sh
bash scripts/start.sh    # Docker, migrations, API :3001, worker, app :3010
```

Abra <http://localhost:3010>. `bash scripts/stop.sh` derruba tudo, sem apagar volumes.

Enquanto `examples/cortex/` não existe, use o pacote de exemplo deles (o padrão) para conhecer o
produto.

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
