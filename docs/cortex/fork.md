# Regras do fork

O objetivo é acompanhar a CopilotKit sem perder o que ela entrega e sem que cada sincronização vire
uma semana de conflitos. A regra é: **o nosso fica em pastas nossas, e o deles fica como está.**

## O que é nosso

| Caminho | O que tem |
|---|---|
| `docs/cortex/` | Esta documentação |
| `examples/cortex/` | Pacote de tenant do CortexBot: `brand.yaml`, `agents.yaml`, `skills.yaml`, `model.yaml`, `channels.yaml`, `knowledge.yaml` |
| `cortex/` | Código nosso que roda ao lado do OpenBot (a ponte da fase 2 e os scripts) |

O adaptador da fase 1 mora no `cortex-symph`, não aqui.

## O que não mudamos

- **Textos da interface.** O nome visível vem de `tenant.product_name` no `brand.yaml`, que a
  sidebar, o login, o onboarding e a tela inicial já leem. Cerca de 34 componentes em `app/src` ainda
  dizem "OpenBot" em texto fixo. Ficam assim, porque a CopilotKit revisa esses textos com frequência
  e cada troca nossa viraria conflito.
- **Cores, animações e layout.**
- **Variáveis `OPENBOT_*`, nomes de banco e volume (`openbot_postgres`), imagens `openbot-agent-*`,
  nomes de pacote e classe, charts do Helm.**
- **Migrations.** Nenhuma tabela nossa no banco deles. Estado nosso, se precisar, fica no Cortex.
- **Desktop (Tauri) e mobile.** Fora do escopo enquanto o CortexBot roda numa VPS.

Se uma mudança no código deles for indispensável, ela vai num commit pequeno e separado, com a
explicação no corpo, e entra na lista abaixo. Se ela servir a qualquer usuário do OpenBot (medir custo
por run, por exemplo), abrimos também uma PR para a CopilotKit, e o fork diminui quando ela entrar.

### Mudanças no código deles

Nenhuma até agora.

## Pull requests

Este repositório é um fork, e o GitHub abre a PR por padrão contra `CopilotKit/openbot`. **Toda PR
aponta para `TiagoAlmeidaS/cortex-bot`, branch `main`.** Uma PR para a CopilotKit é sempre uma
decisão explícita.

## Workflows

Os workflows de `.github/workflows/` são da CopilotKit. `desktop-signing.yml`, `desktop.yml`,
`publish-release.yml` e `release.yml` publicam e assinam builds com segredos que não temos, e devem
ficar desligados no fork (Settings › Actions, ou "Disable workflow" em cada um). O `ci.yml` pode
ficar. Sem minutos de Actions, "verde" é a suíte rodando local (`bun test` e o que `docs/development.md`
pede).

## Sincronizar com a CopilotKit

Uma vez por semana, numa PR própria:

```sh
git remote add upstream https://github.com/CopilotKit/openbot.git   # uma vez
git fetch upstream main
git checkout -b sync/upstream-AAAA-MM-DD origin/main
git merge upstream/main
```

Antes de abrir a PR:

1. Leia o `CHANGELOG.md` deles, seção "Before upgrading". Eles listam migrations, mudanças de padrão e
   variáveis novas. Copie para a descrição da PR o que afeta o CortexBot.
2. Rode a suíte local e suba o ambiente (`bash scripts/start.sh`).
3. Confira que `examples/cortex/` ainda valida. O servidor recusa iniciar com pacote de tenant
   inválido, então um campo novo obrigatório aparece na hora.
4. Confira que a Responsibility "Executar Task do Cortex" ainda recebe o webhook.

Pelo GitHub, o botão "Sync fork" faz o mesmo merge direto na `main`. Use-o só quando a semana não
trouxe aviso de upgrade.

Semanal, não diário: a CopilotKit entrega todo dia e muda bastante (no `CHANGELOG` de 26/10 havia 9
avisos de upgrade e 7 migrations novas).
