# ADR-002 — Worker em processo separado, lendo a outbox por polling de 2 segundos

- **Data da decisão:** reunião técnica de quinta-feira, 09:00 (polling fechado por Larissa às [09:10]; processo separado proposto por Diego às [09:11] e anotado por Larissa às [09:12])
- **Decisores:** Larissa, Diego, Bruno; Marcos validou a latência
- **Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito

## Contexto

Com a outbox ([ADR-001](ADR-001-outbox-no-mysql.md)), algum processo precisa ler as linhas pendentes e fazer as chamadas HTTP. Para os clientes, qualquer atraso abaixo de 10 segundos conta como "tempo real" ([09:02] Marcos). Hoje a aplicação tem um único entry-point, `src/server.ts`. Ele sobe o Express, usa o `prisma` de `src/config/database.ts` e encerra de forma graciosa em `SIGINT`/`SIGTERM`.

## Decisão

- `ADR-002-D1`: o worker faz **polling em loop a cada 2 segundos**. Em cada ciclo, busca os eventos pendentes mais antigos em lote pequeno, processa e marca o resultado ([09:09] Diego, [09:10] Larissa).
- `ADR-002-D2`: o worker roda em **processo separado** da API, porque um restart da API não pode derrubar o worker ([09:11] Diego). O entry-point novo é `src/worker.ts` **(novo)**, e o script é `npm run worker` ([09:11] Larissa). A lógica de processamento fica dentro do módulo, em `src/modules/webhooks/` ([09:28] Bruno).
- `ADR-002-D3`: o worker usa o **mesmo banco e a mesma stack**, mas com uma **instância própria de `PrismaClient`**, com a mesma `DATABASE_URL`. O PrismaClient é por processo ([09:30] Bruno). Isso refina a fala anterior de "usar o mesmo Prisma client" ([09:11] Bruno): é o mesmo banco e a mesma stack, mas não a mesma instância. Na prática, o `prisma` exportado por `src/config/database.ts` já é instanciado no carregamento do módulo, então, importado em `src/worker.ts` **(novo)**, ele já é a instância própria do processo do worker.
- `ADR-002-D4`: roda **um único worker**. Ele processa na ordem de `created_at`, então a ordem só vale por `order_id` e só enquanto houver um worker ([09:12] Diego). Isso é uma **limitação conhecida**, não uma garantia de ordem global ([09:13] Larissa).

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-002-ALT-01` | **Trigger no banco ou LISTEN/NOTIFY** para reagir a cada inserção ([09:09] Bruno) | O MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres. Uma trigger só executa SQL e não avisa processo externo; avisar o worker exigiria improviso, como escrever em arquivo ou chamar um endpoint ([09:09] Diego). |
| `ADR-002-ALT-02` | **Worker dentro do processo da API** | Se a API reinicia, o worker cai junto ([09:11] Diego). |
| `ADR-002-ALT-03` | **Vários workers em paralelo**, particionados por `order_id` ou com lock pessimista ([09:13] Diego) | Adiado: "problema do futuro, não agora" ([09:13] Diego). Os clientes não pediram ordem global ([09:14] Marcos). |

## Consequências

**Positivas**

- `ADR-002-CONS-01`: o atraso de polling é de no máximo 2 segundos. Isso cabe no requisito de "abaixo de 10 segundos" ([09:09] Diego) e foi aceito ([09:10] Marcos).
- O ciclo de vida da API e o do worker ficam independentes: deploy ou restart de um não afeta o outro.
- Nenhum componente novo de infraestrutura.

**Negativas**

- `ADR-002-CONS-02`: o polling gera consultas constantes, mesmo sem eventos. As consultas são baratas por causa do índice em `status` e `created_at` ([09:08] Diego).
- `ADR-002-CONS-03`: um processo a mais para implantar e monitorar.
- `ADR-002-CONS-04`: **limitação conhecida de ordem.** A ordem por `order_id` só vale com um worker ([09:13] Larissa). Mesmo com um worker, quando um evento entra em backoff ([ADR-003](ADR-003-retry-backoff-exponencial-e-dlq.md)), o evento seguinte do mesmo pedido pode ser entregue antes. Esse caso não foi discutido na reunião e está registrado como questão em aberto no [RFC](../RFC.md#5-questões-em-aberto). O cliente consegue reconciliar a ordem pelos campos `from_status`, `to_status` e `timestamp` do payload ([09:43] Diego).
- `ADR-002-CONS-05`: um worker só é ponto único de vazão. Escalar horizontalmente exige uma decisão nova, com particionamento ou lock.

**Trade-off explícito:** trocamos reatividade imediata e escala horizontal por simplicidade operacional, ficando no mesmo MySQL e com um único processo, dentro do orçamento de latência de 10 segundos.
