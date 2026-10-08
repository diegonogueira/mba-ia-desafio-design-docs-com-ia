# ADR-007 — Payload gravado como snapshot e filtro de eventos aplicado na inserção da outbox

- **Data da decisão:** reunião técnica de quinta-feira, 09:00. Filtro fechado às [09:34] por Bruno e Diego; snapshot às [09:52] por Larissa, Diego e Bruno, depois da saída de Marcos e Sofia.
- **Decisores:** Bruno, Diego, Larissa; Marcos definiu o filtro por status
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-005](ADR-005-at-least-once-com-x-event-id.md)

## Status

Aceito

## Contexto

Cada endpoint escolhe quais status de pedido quer receber. O exemplo dado foi "só quero saber quando vira SHIPPED e DELIVERED" ([09:33] Marcos). Ficaram duas escolhas de desenho para a linha da outbox:

1. Onde aplicar o filtro: ao gravar o evento ou ao enviá-lo.
2. O que gravar: o payload pronto ou só a referência ao pedido.

## Decisão

- `ADR-007-D1`: **o filtro é aplicado na inserção.** `publishWebhookEvent` consulta os endpoints **ativos** do customer do pedido cuja lista de status inclui o `to_status`. Grava uma linha na outbox para cada endpoint que casa. Se nenhum endpoint casa, nada é inserido ([09:34] Bruno, [09:34] Diego).
- `ADR-007-D2`: **o payload é gravado já renderizado (snapshot)** no momento da inserção. O evento reflete o estado do pedido quando o status mudou, mesmo que o pedido mude depois ([09:52] Larissa, [09:52] Diego, [09:52] Bruno).
- `ADR-007-D3`: o payload é **enxuto**. Tem só a identificação do evento, do pedido e da transição, mais campos básicos como o total. **Não traz `items`**. Quem quiser detalhes consulta `GET /orders/:id` ([09:43] Diego, [09:44] Bruno). O contrato campo a campo está no [FDD §6.8](../FDD.md#68-fdd-contrato-08--chamada-de-saída-para-o-cliente-o-webhook-em-si). No código, essa rota é `GET /api/v1/orders/:id` (`src/modules/orders/order.routes.ts`).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-007-ALT-01` | **Filtrar na hora do envio**, com o worker descartando os eventos não assinados ([09:34] Diego) | Gravaria linhas que nunca seriam enviadas. Filtrar na inserção economiza linhas na tabela ([09:34] Bruno). |
| `ADR-007-ALT-02` | **Gravar só o `order_id` e renderizar o payload no envio** ([09:51] Bruno) | Se o pedido mudar entre a inserção e o envio, o evento não refletiria a transição que o gerou e haveria "caso esquisito" ([09:52] Larissa). O efeito piora com retentativas que podem durar ~14,6 h ([ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md)). |
| `ADR-007-ALT-03` | **Incluir `items` no payload** | Infla o payload sem necessidade. O cliente consulta o pedido quando precisar ([09:43] Diego). |

## Consequências

**Positivas**

- `ADR-007-CONS-01`: cada retentativa e o replay enviam **exatamente o mesmo corpo**. Assim a assinatura HMAC ([ADR-004](ADR-004-hmac-sha256-secret-por-endpoint.md)) e a deduplicação pelo `event_id` ([ADR-005](ADR-005-at-least-once-com-x-event-id.md)) ficam estáveis.
- `ADR-007-CONS-02`: o worker não precisa ler `orders` nem montar o payload. Ele lê a linha e envia.
- Sem linhas inúteis na outbox de customers sem endpoint interessado.

**Negativas**

- `ADR-007-CONS-03`: um endpoint criado ou com filtro alterado **depois** da mudança de status não recebe eventos passados. A assinatura vale para a frente.
- `ADR-007-CONS-04`: o snapshot fica desatualizado de propósito. O cliente que quiser o estado atual precisa consultar a API.
- A transação de `changeStatus` passa a incluir a consulta aos endpoints do customer (ver [ADR-001](ADR-001-outbox-no-mysql.md), consequência `ADR-001-CONS-02`).

**Trade-off explícito:** fazer mais trabalho na transação (filtrar e renderizar) deixa o worker simples e garante que todo reenvio seja idêntico ao original. O custo é que mudanças de assinatura só valem dali em diante.
