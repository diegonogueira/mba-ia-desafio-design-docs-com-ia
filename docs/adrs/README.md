# Architectural Decision Records — Webhooks de Notificação de Pedidos

Decisões da reunião técnica registrada em [`TRANSCRICAO.md`](../../TRANSCRICAO.md), no formato MADR (Status, Contexto, Decisão, Alternativas Consideradas e Consequências). Os arquivos seguem o padrão `ADR-NNN-titulo-em-kebab-case.md`.

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL existente, atômico com `changeStatus` | Aceito |
| [ADR-002](ADR-002-worker-separado-em-polling.md) | Worker em processo separado (`src/worker.ts`), polling de 2 s, single-worker | Aceito |
| [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md) | Retry 1m/5m/30m/2h/12h, DLQ em `webhook_dead_letter`, replay ADMIN | Aceito (contagem exata pendente, RFC-OQ-06) |
| [ADR-004](ADR-004-hmac-sha256-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint, rotação com 24 h de carência, https obrigatório | Aceito (revisão de segurança pendente) |
| [ADR-005](ADR-005-at-least-once-com-x-event-id.md) | Entrega at-least-once com deduplicação por `X-Event-Id` | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões do projeto (módulo, `AppError`, Pino, error middleware, Zod, `requireRole`) | Aceito |
| [ADR-007](ADR-007-snapshot-do-payload-e-filtro-na-insercao.md) | Payload snapshot e filtro de eventos aplicados na inserção da outbox | Aceito |

Para criar um ADR novo, use o próximo número livre. Um ADR substituído continua no diretório com status `Substituído por ADR-NNN`.
