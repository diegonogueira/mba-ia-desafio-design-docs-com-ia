# ADR-005 — Garantia de entrega at-least-once com deduplicação pelo `X-Event-Id`

- **Data da decisão:** reunião técnica de quinta-feira, 09:00 (fechada às [09:26] por Larissa)
- **Decisores:** Diego, Larissa; Sofia levantou a ressalva; Marcos fica com a comunicação aos clientes
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md)

## Status

Aceito

## Contexto

Com outbox, worker e retry ([ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md)), existem cenários de reenvio. O cliente pode processar o evento e responder depois do timeout de 10 s ([09:42] Diego). O worker pode cair entre o POST e a marcação de entregue. Um replay manual da DLQ reenvia o mesmo evento.

## Decisão

- `ADR-005-D1`: a plataforma garante **at-least-once**. O cliente pode receber o mesmo evento mais de uma vez e precisa estar preparado para isso ([09:24] Diego).
- `ADR-005-D2`: todo envio leva o header **`X-Event-Id`**, com um **UUID gerado quando o evento entra na outbox** e único por evento ([09:25] Diego). O cliente deduplica por esse id ([09:25] Diego, [09:26] Larissa). O id não muda entre retentativas nem no replay da DLQ, que reaproveita a mesma linha da outbox. Por isso a deduplicação continua valendo nesses casos.
- `ADR-005-D3`: a semântica vai **documentada em destaque no portal do desenvolvedor**, sob responsabilidade do PM ([09:26] Marcos).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-005-ALT-01` | **Exactly-once** | Exigiria coordenação dos dois lados e é bem mais complexo. At-least-once com event id resolve a grande maioria dos casos e é o padrão de mercado ([09:25] Diego). |
| `ADR-005-ALT-02` | **At-most-once**, sem retry (alternativa plausível, não levantada na reunião) | Incompatível com a política de retry e DLQ do [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md) e com a exigência de que o cliente seja notificado ([09:00] Marcos). |

## Consequências

**Positivas**

- `ADR-005-CONS-01`: implementação simples do nosso lado. O worker pode reenviar sem medo depois de qualquer falha ambígua (timeout, queda do processo).
- O id é estável do evento até a DLQ e o replay, o que também ajuda no suporte: dá para rastrear um evento ponta a ponta.

**Negativas**

- `ADR-005-CONS-02`: a deduplicação fica do lado do cliente ([09:25] Sofia). Um cliente que não deduplique pode processar a mesma mudança duas vezes.
- `ADR-005-CONS-03`: o PM precisa comunicar bem essa responsabilidade no portal ([09:26] Marcos).

**Trade-off explícito:** transferimos ao cliente o trabalho de deduplicar pelo `X-Event-Id` e, em troca, evitamos coordenação distribuída entre as duas partes. É a mesma escolha que Stripe e GitHub fazem ([09:25] Diego).
