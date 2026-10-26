# Roadmap

Cada item é uma PR. Item no `cortex-symph` ou no `cortex-execution` vai para aquele repositório.

## Fase 0: base

| # | Entrega | Onde | Pronto quando |
|---|---|---|---|
| 0.1 | Esta documentação | cortex-bot | mergeada |
| 0.2 | Workflows de release e assinatura desligados | GitHub do fork | só `ci.yml` e `security_zizmor.yml` ativos |
| 0.3 | OpenBot rodando local com o pacote de exemplo | máquina do Tigo | `localhost:3010` abre e um Bot responde |

## Fase 1: com o Symph

| # | Entrega | Onde | Pronto quando |
|---|---|---|---|
| 1.1 | Pacote `examples/cortex/` (nome CortexBot, Bot "Executor do Cortex" em português, modelo, política inicial) | cortex-bot | o servidor sobe com `TENANT_PACKAGE_DIR=../examples/cortex` |
| 1.2 | Responsibility "Executar Task do Cortex" com gatilho webhook, e um roteiro para criá-la | cortex-bot (doc) | um `curl` assinado dispara um run visível no histórico |
| 1.3 | Adaptador `cortex-bot` no Symph | cortex-symph | teste do adaptador com o gatilho simulado passando |
| 1.4 | Leitura do fim do run (token de serviço ou aviso de volta) | cortex-symph, talvez cortex-bot | o Symph sabe quando o run terminou e com que texto |
| 1.5 | Piloto: uma Task não-código delegada ao `cortex-bot` | Cortex (Project de teste) | Task concluída com nota de handoff e auditoria no CortexBot |

## Fase 2: nativo

| # | Entrega | Onde | Pronto quando |
|---|---|---|---|
| 2.1 | Ponte `cortex/bridge/` com o protocolo de runner | cortex-bot | CortexBot aparece no painel Agentes como runner próprio |
| 2.2 | MCP do Cortex como servidor MCP do Bot, só leitura e notas | cortex-bot (config + doc) | o Bot lê o contexto do Project durante o run |
| 2.3 | Proxy de modelo na ponte, com custo por run | cortex-bot | `POST /executions` com tokens e custo, e a cota desconta |
| 2.4 | PR para a CopilotKit com uso por run | CopilotKit/openbot | aberta (merge é deles) |

## Fora de escopo por enquanto

- Traduzir a interface para português.
- Desktop e mobile.
- Login pela conta do Cortex.
- Trocar o CopilotKit Intelligence por outra peça. É a dependência mais cara de remover, e só vale
  se o CortexBot virar produto.
