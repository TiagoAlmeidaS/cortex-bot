# CortexBot

O CortexBot é o fork do [OpenBot](https://github.com/CopilotKit/openbot), da CopilotKit, adaptado ao
ecossistema Cortex. O OpenBot dá a cada agente ("Bot") um computador próprio: um navegador real com
logins, arquivos e ferramentas concedidas. Toda ação passa por um gateway que decide pela política e
grava auditoria. O CortexBot liga isso ao fluxo do Cortex: o Bot pega uma Task delegada no Cortex
Execution, faz o trabalho e devolve a entrega na própria Task.

Esta pasta (`docs/cortex/`) é a nossa. O resto do repositório é o OpenBot, e continuamos puxando o que
a CopilotKit entrega.

## Leitura

| Documento | O que tem |
|---|---|
| [arquitetura.md](arquitetura.md) | Onde o CortexBot entra no ecossistema e o que ele faz que o Symph não faz |
| [fork.md](fork.md) | Regras do fork: o que mudamos, o que nunca mudamos e como sincronizar com a CopilotKit |
| [fase-1-symph.md](fase-1-symph.md) | Fase 1: o CortexBot como agente do Symph |
| [fase-2-nativo.md](fase-2-nativo.md) | Fase 2: o CortexBot como runner nativo do Cortex Execution, com custo |
| [rodar.md](rodar.md) | Rodar local e numa VPS |
| [roadmap.md](roadmap.md) | Entregas em ordem, com critério de pronto |

## Estado

Nada de código ainda. Este é o ponto de partida: o fork está idêntico ao `CopilotKit/openbot` de
26/10/2026 (commit `3592f85`), mais esta documentação.

## Licença

O OpenBot é MIT, © 2026 CopilotKit. O `LICENSE` da raiz fica como está. O que escrevemos aqui também é
MIT. Qualquer material público sobre o CortexBot diz que ele é baseado no OpenBot.
