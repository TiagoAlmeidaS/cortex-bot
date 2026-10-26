# Fase 1: o CortexBot como agente do Symph

O objetivo é validar a experiência: delegar uma Task no Cortex e ver o CortexBot fazê-la, sem mexer no
Cortex Execution e com o mínimo de código.

O Symph já fala o protocolo de runner do Cortex. Na fase 1, o CortexBot entra como mais um agente do
Symph, ao lado de Claude Code e Codex. O Symph continua dono da sessão, do claim, das notas e da
execução. O CortexBot só recebe o trabalho e o faz.

## O que muda em cada lugar

### Cortex Execution: nada

O campo `provider` do catálogo de agentes é texto livre (`packages/types/src/agent-catalog.ts`,
`AgentProvider`), então um perfil com `provider: "cortex-bot"` aparece no painel Agentes e em
"Delegar a agente" como os outros.

### Cortex Symph: um adaptador

- `src/agent/contract.ts`: adicionar `'cortex-bot'` em `AGENT_PROVIDERS`.
- `src/agent/cortexbot.ts`: o adaptador. Em vez de abrir um processo de CLI, ele:
  1. monta o payload com a Task e o briefing (`GET /api/tasks/:id/briefing`, que o Symph já lê);
  2. envia ao gatilho webhook do CortexBot, assinado com o segredo do gatilho no padrão Standard
     Webhooks (`webhook-id`, `webhook-timestamp`, `webhook-signature`). O `webhook-id` é o id do run
     no Symph, para que um reenvio não dispare dois trabalhos;
  3. acompanha o run da Responsibility até terminar (ver "Pontos a confirmar");
  4. traduz o resultado para os eventos do Symph (`session.started`, `assistant.message`,
     `session.completed`, `session.failed`) e devolve o texto final como resumo do run.
- Perfil de exemplo no `symph.json`:

```json
{
  "id": "cortex-bot",
  "provider": "cortex-bot",
  "command": "https://cortexbot.exemplo.com/api/events/triggers/<triggerId>",
  "args": [],
  "authEnv": ["CORTEXBOT_WEBHOOK_SECRET", "CORTEXBOT_API_TOKEN"]
}
```

Para o CortexBot, `command` é a URL do gatilho, não um binário. Segredo só por nome de variável, como
nos outros perfis.

- **Sem worktree.** O CortexBot não edita o repositório. O Symph pula a etapa de verificação e de PR
  para esse provider, e a entrega é a nota de handoff com os links que o Bot produziu.

### CortexBot: só configuração

1. Pacote `examples/cortex/` com `product_name: CortexBot` e um Bot "Executor do Cortex" (built-in),
   com `system_prompt` em português dizendo que ele recebe uma Task com aceite e devolve o que fez,
   o que não fez e os links.
2. Uma Responsibility "Executar Task do Cortex" nesse Bot, com gatilho `webhook`. O CortexBot gera o
   segredo, que vai para `CORTEXBOT_WEBHOOK_SECRET` no Symph.
3. Política de ações começando restrita: browser e arquivos liberados, shell e envio de e-mail só com
   aprovação, até a experiência mostrar o que pode ser solto.

## Payload do webhook

```json
{
  "source": "cortex",
  "task": {
    "id": "…",
    "key": "PR-1234",
    "title": "…",
    "url": "https://<instância>/tasks/…"
  },
  "briefing": "texto do briefing, já no orçamento de tokens",
  "symph": { "runId": "…", "profileId": "cortex-bot" }
}
```

Limite do OpenBot: 256 KB por entrega (`TRIGGER_BODY_LIMIT`). O briefing tem de caber nisso.

## Pontos a confirmar na implementação

- **Acompanhar o run.** A entrada do webhook responde 200 quando enfileira, nunca quando termina. O
  histórico está em `GET /api/responsibilities/:id/runs`, protegido pela sessão de usuário. É preciso
  achar a forma de máquina de ler esse histórico (token de serviço, se houver) ou, se não houver,
  fazer o Bot avisar o fim chamando o Symph de volta. A segunda opção é código nosso no Symph, não
  no OpenBot.
- **Custo.** Na fase 1 o run vai com custo nulo. É a mesma limitação das execuções antigas do Symph,
  e ela fica registrada na nota.
- **Tempo.** O teto de tempo do perfil no Symph vale. Run que passa dele é marcado como falha e a
  Task é devolvida.

## Pronto quando

Uma Task não-código, num Project de teste, é delegada ao perfil `cortex-bot` no painel Agentes. O
Symph a pega, o CortexBot executa, e a Task recebe a nota de handoff com o resultado e fica concluída
ou devolvida com o motivo. A auditoria do CortexBot mostra o run com `initiator_kind=responsibility`.
