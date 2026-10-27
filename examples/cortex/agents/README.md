# Um Bot por arquivo

Um arquivo `.yaml` aqui declara um Bot, e o servidor o lê junto com `../agents.yaml`. O pacote do
CortexBot começa com um Bot só, o Executor do Cortex, declarado em `../agents.yaml`.

Para outro Bot que execute Tasks do Cortex, copie o formato do Executor e dê a ele a sua própria
Responsibility com gatilho webhook. Modelos para copiar estão em
[`../../fintech/catalog`](../../fintech/catalog).
