# ADR-003 — Retry com backoff exponencial (1m/5m/30m/2h/12h) e DLQ em tabela separada

- **Status:** Aceito
- **Data da decisão:** reunião técnica de quinta-feira, 09:00. Retry fechado às [09:17] por Larissa; DLQ e replay às [09:18]–[09:19]; role ADMIN às [09:36].
- **Decisores:** Larissa, Diego, Bruno, Sofia (replay ADMIN); Marcos aceitou a janela
- **Relacionados:** [ADR-002](ADR-002-worker-separado-em-polling.md), [ADR-005](ADR-005-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto

O endpoint do cliente pode estar fora do ar ou lento. Já houve cliente com duas horas de indisponibilidade em manutenção planejada ([09:16] Diego). O worker trata como falha qualquer chamada sem resposta em 10 segundos ([09:42] Diego). Precisamos definir quantas vezes tentar, com qual espaçamento e o que fazer quando as tentativas acabam.

## Decisão

- `ADR-003-D1`: **backoff exponencial com teto de 5 tentativas**. Os intervalos são **1 min, 5 min, 30 min, 2 h e 12 h** ([09:15] Diego, [09:17] Diego, [09:17] Larissa).
  - **Como lemos a contagem:** o envio inicial é seguido de 5 retentativas, uma após cada intervalo. É a única leitura em que os cinco intervalos citados somam a janela de "quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego): 1 + 5 + 30 + 120 + 720 min = 14 h 36 min. Como a fala "5 tentativas" admite outra leitura, a confirmação está pedida no [RFC](../RFC.md#questões-em-aberto).
- `ADR-003-D2`: esgotadas as retentativas, o evento vira **falha permanente**. Ele é copiado para a tabela separada **`webhook_dead_letter`** com payload, motivo da falha e timestamp ([09:18] Diego). A linha da outbox fica com status terminal `FAILED`.
- `ADR-003-D3`: o **reprocessamento é manual**, pelo endpoint `POST /admin/webhooks/dead-letter/:id/replay`. Ele recoloca o evento na outbox como pendente ([09:18] Diego, [09:35] Diego). No código, o endpoint fica sob o prefixo `/api/v1` (`src/app.ts`).
- `ADR-003-D4`: o replay **exige a role `ADMIN`**, verificada pelo `requireRole` que já existe em `src/middlewares/auth.middleware.ts` ([09:36] Sofia, [09:36] Larissa). O replay **registra quem o executou**, para auditoria ([09:36] Sofia).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-003-ALT-01` | **Retry indefinido com backoff** ([09:15] Diego) | O evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego). |
| `ADR-003-ALT-02` | **3 tentativas**, mais agressivo ([09:16] Bruno) | Três retentativas acabariam em uns 30 minutos e matariam o evento de um cliente que ficou fora de manhã. Já houve indisponibilidade de duas horas ([09:16] Diego). |
| `ADR-003-ALT-03` | **Marcar como "failed" na própria outbox, sem tabela de DLQ** ([09:17] Larissa) | A tabela separada deixa a leitura da outbox principal mais limpa e guarda a evidência para debug e reprocessamento ([09:18] Diego). |

## Consequências

**Positivas**

- `ADR-003-CONS-01`: a janela de ~14,6 h cobre indisponibilidades longas, como as 2 h de manutenção citadas ([09:16] Diego). Marcos considerou a janela aceitável ([09:17] Marcos).
- `ADR-003-CONS-02`: nenhum evento fica preso para sempre. Todo evento termina entregue ou na DLQ, com motivo registrado.
- O replay é auditável e restrito a ADMIN.

**Negativas**

- `ADR-003-CONS-03`: um evento pode chegar até ~14,6 h depois da mudança de status. Isso se soma à limitação de ordem do [ADR-002](ADR-002-worker-separado-em-polling.md).
- `ADR-003-CONS-04`: o reprocessamento depende de ação humana. Não existe alerta automático ao cliente: o aviso por email ficou para a próxima fase ([09:37] Larissa).
- Retentar gera reenvios, o que reforça a semântica at-least-once ([ADR-005](ADR-005-at-least-once-com-x-event-id.md)).

**Trade-off explícito:** um teto finito com DLQ troca a garantia de "eventualmente entrega" por previsibilidade operacional e uma outbox limpa. O custo é exigir intervenção manual depois de ~14,6 h de falha contínua.
