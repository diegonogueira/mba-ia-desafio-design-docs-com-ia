# ADR-006 — Reuso dos padrões do projeto: módulo em `src/modules/webhooks`, `AppError`, Pino, error middleware, Zod e `requireRole`

- **Data da decisão:** reunião técnica de quinta-feira, 09:00 (fechada às [09:30] por Larissa)
- **Decisores:** Larissa, Bruno, Diego; Sofia no uso do `requireRole`
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md), [ADR-004](ADR-004-hmac-sha256-secret-por-endpoint.md)

## Status

Aceito

## Contexto

O OMS segue um padrão claro, e cada domínio é um módulo em `src/modules/<dominio>/` ([09:27] Bruno). Os padrões existentes no código são:

| Padrão | Onde está no código |
|---|---|
| Módulo com controller, service, repository, routes e schemas | `src/modules/orders/order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts`, `order.schemas.ts` (o mesmo vale para `customers`, `products` e `users`) |
| Erros de domínio herdam de `AppError(message, statusCode, errorCode, details)` e usam códigos em UPPER_SNAKE (`INSUFFICIENT_STOCK`, `INVALID_STATUS_TRANSITION`) | `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/shared/errors/index.ts` |
| Tratamento centralizado de `AppError`, `ZodError` e erros conhecidos do Prisma (`P2002` → 409, `P2025` → 404) | `src/middlewares/error.middleware.ts` |
| Validação de body, query e params com Zod via `validate({...})` | `src/middlewares/validate.middleware.ts` |
| Autenticação JWT e autorização por role (`authenticate`, `requireRole`) | `src/middlewares/auth.middleware.ts` (uso em `src/modules/users/user.routes.ts`) |
| Logger Pino com redação de campos sensíveis | `src/shared/logger/index.ts` |
| Composição de dependências e montagem das rotas | `src/app.ts` (`buildControllers`), `src/routes/index.ts` (`buildApiRouter`) |
| Listagens paginadas | `src/shared/http/response.ts` (`paginated`) |
| Configuração validada por Zod | `src/config/env.ts` |

## Decisão

`ADR-006-D1`: **reusar ao máximo o que já existe**. São reaproveitados `AppError`, Pino, o error middleware, o padrão de módulos, o padrão de schemas Zod e o padrão de códigos de erro. O módulo de webhooks fica igual aos outros ([09:30] Larissa). Em concreto:

- `ADR-006-D2`: tudo fica no módulo novo **`src/modules/webhooks/`**, com controller, service, repository, routes e schemas ([09:27] Bruno). O processamento do worker também mora no módulo, e o arquivo de entrada é `src/worker.ts` ([09:28] Bruno). Entre os dois nomes sugeridos para o processador, `webhook.worker.ts` e `webhook.processor.ts` ([09:28] Bruno), adotamos `webhook.processor.ts`, para não confundir com o entry-point `src/worker.ts`.
- `ADR-006-D3`: os erros novos estendem a hierarquia de `src/shared/errors/` com **códigos de prefixo `WEBHOOK_`** ([09:28] Bruno, [09:29] Larissa). Os 404 estendem `AppError` direto, porque `NotFoundError` fixa o código `NOT_FOUND`. Os 409 estendem `ConflictError(message, code)`. A lista está no [FDD §8](../FDD.md#8-matriz-de-erros). Eles são tratados pelo `errorMiddleware` sem nenhuma mudança nele ([09:29] Bruno).
- `ADR-006-D4`: o log usa o **Pino que já existe**, sem biblioteca nova de log ([09:29] Bruno).
- `ADR-006-D5`: o endpoint de replay usa o **`requireRole('ADMIN')` que já existe** ([09:36] Larissa).
- `ADR-006-D6`: a outbox e as demais tabelas usam **UUID** (`@default(uuid()) @db.Char(36)`), como todas as entidades de domínio de `prisma/schema.prisma` (a exceção é o contador `OrderNumberSequence`) ([09:51] Larissa).
- `ADR-006-D7`: a integração com pedidos é feita pela **função `publishWebhookEvent(tx, order, fromStatus, toStatus)`**, que recebe o `Prisma.TransactionClient` da transação corrente. Assim, `OrderService` não recebe o repository de webhooks por injeção ([09:41] Bruno, [09:41] Diego).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-006-ALT-01` | **Injetar um `WebhookRepository` inteiro no construtor do `OrderService`** | Uma função pura que recebe o `tx` basta. Não precisa injetar o repository inteiro ([09:41] Diego). Isso também evita mudar a assinatura do construtor de `OrderService`, hoje instanciado em `buildControllers` (`src/app.ts`). |
| `ADR-006-ALT-02` | **Estrutura própria para webhooks**, com biblioteca nova de log, hierarquia de erros própria ou código de erro sem prefixo (alternativa plausível) | Contraria a decisão de reuso máximo ([09:30] Larissa). Não vamos acrescentar nada novo ([09:29] Bruno). Duplicaria o tratamento que o `errorMiddleware` já faz. |
| `ADR-006-ALT-03` | **Compartilhar a instância de `PrismaClient` da API com o worker** ([09:29] Diego) | O PrismaClient é por processo. O worker cria a própria instância com a mesma `DATABASE_URL` ([09:30] Bruno). |

## Consequências

**Positivas**

- `ADR-006-CONS-01`: as respostas de erro do módulo saem no envelope `{ error: { code, message, details? } }` que os clientes da API já conhecem. Esse envelope vem de `src/middlewares/error.middleware.ts`.
- Curva de aprendizado zero para o time de Pedidos. A revisão de código fica mais simples, inclusive a de segurança.
- `ADR-006-CONS-02`: `publishWebhookEvent(tx, ...)` deixa a integração com `changeStatus` em **uma chamada**, dentro da transação que já existe.

**Negativas**

- `ADR-006-CONS-03`: o `validate()` de `src/middlewares/validate.middleware.ts` transforma todo `ZodError` em `VALIDATION_ERROR`. Por isso, uma recusa de URL `http://` feita no schema Zod ([09:23] Sofia) chega ao cliente com `code: "VALIDATION_ERROR"`. O código `WEBHOOK_INVALID_URL` aparece em `details[].message`. Ter `WEBHOOK_INVALID_URL` como `code` de primeiro nível exigiria mudar o middleware compartilhado, e isso contraria esta decisão.
- `ADR-006-CONS-04`: o projeto não tem biblioteca de métricas nem de tracing (`package.json`). Por isso, a observabilidade do módulo fica restrita a logs estruturados e a consultas sobre as tabelas (ver [FDD](../FDD.md#9-observabilidade)).
- Herdamos os limites do modelo atual de autorização. Qualquer usuário autenticado gerencia webhooks de qualquer customer, assim como hoje gerencia pedidos de qualquer customer. Endurecer isso ficou para depois ([09:37] Sofia).

**Trade-off explícito:** a consistência com a base de código vem antes de otimizações locais do módulo, como códigos de erro de primeiro nível na validação ou uma stack de observabilidade dedicada.
