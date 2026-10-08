# ADR-001 — Padrão Outbox no MySQL existente

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, 09:00 (fechada às [09:08] por Larissa)
- **Decisores:** Larissa (Tech Lead), Diego (Plataforma), Bruno (Pedidos)
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-e-filtro-na-insercao.md)

## Contexto

Três clientes B2B querem ser avisados quando o status dos pedidos deles muda ([09:00] Marcos). O evento nasce em `OrderService.changeStatus` (`src/modules/orders/order.service.ts`). Esse método roda dentro de `this.prisma.$transaction(...)` e, na mesma transação, valida a transição (`canTransition`, em `src/modules/orders/order.status.ts`), debita ou repõe estoque quando a transição exige, atualiza `orders` e insere em `order_status_history`.

A reunião tinha duas restrições:

- Um cliente lento não pode travar a mudança de status de outros pedidos ([09:04] Bruno).
- Não pode existir mudança de status sem evento, nem evento de uma mudança que sofreu rollback ([09:06] Diego, [09:40] Bruno).

O time é pequeno e não quer subir infraestrutura nova ([09:07] Larissa, [09:07] Diego).

## Decisão

Adotamos o **padrão Transactional Outbox no MySQL que já existe**:

- `ADR-001-D1`: dentro da **mesma transação** de `changeStatus`, insere-se uma linha na tabela nova `webhook_outbox` para cada endpoint interessado no evento ([09:06] Diego).
- `ADR-001-D2`: se a inserção na outbox falhar, a transação inteira faz rollback e o status não muda ([09:40] Bruno, [09:41] Diego).
- `ADR-001-D3`: a outbox tem índices em `status` e em `created_at` ([09:08] Diego). Os status são pendente, processando, falhou e entregue: `PENDING`, `PROCESSING`, `FAILED`, `DELIVERED`.
- `ADR-001-D4`: o id da linha é UUID, como no resto do schema (`prisma/schema.prisma`, [09:51] Larissa).
- A entrega HTTP acontece fora da transação, num worker separado (ver [ADR-002](ADR-002-worker-separado-em-polling.md)).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-001-ALT-01` | **Disparo HTTP síncrono dentro de `changeStatus`** ([09:03] Larissa) | A transação já é pesada. Um cliente lento seguraria a transação e travaria as mudanças de status de outros pedidos ([09:04] Bruno). Se o cliente estiver fora do ar, não faz sentido dar rollback na mudança de status ([09:04] Bruno). "Síncrono está fora de questão" ([09:06] Diego). |
| `ADR-001-ALT-02` | **Redis Streams / fila externa** ([09:07] Larissa) | Exigiria subir e operar infraestrutura nova. Redis Cluster para isso seria overengineering para um time pequeno ([09:07] Diego). Também não resolve sozinho a atomicidade com a transação do MySQL. |

## Consequências

**Positivas**

- `ADR-001-CONS-01`: atomicidade. Se a transação commita, o evento está registrado; se faz rollback, o evento some junto ([09:06] Diego).
- Nenhuma infraestrutura nova: usamos o mesmo MySQL e o mesmo Prisma.
- A latência do cliente fica fora do caminho crítico da API de pedidos.

**Negativas**

- `ADR-001-CONS-02`: a transação de `changeStatus` ganha um passo a mais (consulta dos endpoints interessados e um INSERT por endpoint), o que aumenta um pouco a latência do `PATCH /api/v1/orders/:id/status`.
- `ADR-001-CONS-03`: a tabela cresce sem limite. O arquivamento das linhas entregues, depois de cerca de 30 dias, ficou **fora do escopo** desta feature ([09:08] Diego).
- A entrega deixa de ser imediata: passa a depender do intervalo de polling do worker ([ADR-002](ADR-002-worker-separado-em-polling.md)).

**Trade-off explícito:** aceitamos uma transação um pouco mais longa e uma tabela que cresce em troca de consistência garantida entre o status e o evento, sem operar um broker novo.
