# PRD — Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
|---|---|
| **Product Manager** | Marcos |
| **Tech Lead** | Larissa |
| **Status** | Aprovado em reunião, implementação a iniciar |
| **Data** | 2026-10-08 |
| **Prazo-alvo** | fim de novembro, estimado em 3 sprints com revisão de segurança ([09:45] Marcos, [09:47] Larissa) |
| **Documentos relacionados** | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/README.md) · [TRACKER](TRACKER.md) |

## 1. Resumo e contexto da feature

O Order Management System (OMS) gerencia pedidos com uma máquina de estados: PENDING → PAID → PROCESSING → SHIPPED → DELIVERED, com CANCELLED como saída. Hoje ele não tem nenhum mecanismo de notificação externa. Esta feature cria **webhooks de saída**: cada cliente cadastra um ou mais endpoints HTTPS e passa a receber automaticamente uma chamada assinada quando um pedido dele muda para um dos status que escolheu.

## 2. Problema e motivação

- `PRD-PROB-01`: três clientes B2B (Atlas Comercial, MaxDistribuição e Nova Cargo) pediram formalmente para ser notificados quando o status dos pedidos muda ([09:00] Marcos).
- `PRD-PROB-02`: hoje esses clientes consultam `GET /orders` periodicamente para descobrir mudanças. Isso deixa a integração **lenta e cara** para eles ([09:00] Marcos).
- `PRD-PROB-03`: **risco comercial.** A Atlas indicou que pode migrar para um concorrente se a solução não sair até o fim do trimestre ([09:00] Marcos). Depois ela fixou o fim de novembro como data desejada ([09:45] Marcos).

## 3. Público-alvo e cenários de uso

| Público | Papel | Cenário |
|---|---|---|
| Clientes B2B integrados (Atlas, MaxDistribuição, Nova Cargo) | Recebem as notificações | `PRD-CEN-01`: um cliente quer saber só quando o pedido vira SHIPPED ou DELIVERED (exemplo dado na reunião). Ele cadastra um endpoint com esse filtro e para de fazer polling em `GET /orders` ([09:33] Marcos, [09:00] Marcos). |
| Usuários do OMS que representam o cliente | Configuram webhooks pela API, autenticados com JWT ([09:32] Marcos) | `PRD-CEN-02`: cadastra a URL, guarda a secret devolvida e consulta o histórico das últimas entregas, com sucesso ou falha, payload, resposta e tempo de resposta ([09:31] Marcos, [09:34] Marcos). |
| Administradores (role ADMIN) | Operação | `PRD-CEN-03`: depois que o endpoint do cliente volta de uma indisponibilidade longa, o ADMIN reprocessa os eventos que foram para a fila de falhas ([09:18] Diego, [09:36] Sofia). |
| Cliente com várias integrações | Recebe por mais de um endpoint | `PRD-CEN-04`: identifica qual cadastro originou cada chamada pelo `X-Webhook-Id` ([09:44] Sofia). |
| Cliente cuja secret vazou | Segurança | `PRD-CEN-05`: pede uma nova secret e tem 24 h para trocar nos sistemas dele sem perder notificações ([09:21] Sofia, [09:22] Diego). |

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta |
|---|---|---|---|
| `PRD-MET-01` | Notificação percebida como "tempo real" | Tempo entre o commit da mudança de status e a primeira tentativa de entrega | **< 10 s** para endpoints saudáveis. O polling contribui com no máximo 2 s ([09:02] Marcos, [09:10] Larissa). |
| `PRD-MET-02` | Reter os clientes que pediram a feature | Clientes B2B solicitantes integrados recebendo webhooks | **3 de 3** até o fim de novembro. Meta proposta por este PRD a partir do pedido dos três clientes ([09:00] Marcos) e do prazo da Atlas ([09:45] Marcos). |
| `PRD-MET-03` | Nenhuma mudança de status sem notificação | Mudanças de status com endpoint interessado e sem evento na outbox | **0** (garantia transacional, [09:40] Bruno) |
| `PRD-MET-04` | Nenhuma notificação perdida em silêncio | Eventos que não terminam entregues nem na fila de falhas | **0**. Todo evento termina entregue ou na DLQ com motivo ([09:15] Diego, [09:18] Diego). |
| `PRD-MET-05` | Tolerar indisponibilidade do cliente | Janela coberta pelas retentativas antes de declarar falha | **~14,6 h** (1m + 5m + 30m + 2h + 12h; [09:17] Diego, [09:17] Marcos). Leitura de 1 envio + 5 retentativas, a confirmar (RFC-OQ-06). |

## 5. Escopo

### 5.1 Incluso

- Cadastro, listagem, edição e remoção de endpoints de webhook por customer, com filtro de status.
- Secret por endpoint, gerada pela plataforma e rotacionável.
- Notificação automática e assinada a cada mudança de status de pedido que o endpoint queira receber.
- Retentativas automáticas, fila de falhas e reprocessamento manual por ADMIN.
- Histórico de entregas consultável pela API.

### 5.2 Fora de escopo

| ID | Item | Situação na reunião |
|---|---|---|
| `PRD-OUT-01` | **Email ao cliente quando o webhook dele falha** | Descartado nesta fase. Talvez na próxima, depois de medir o impacto ([09:37] Larissa, [09:38] Marcos). |
| `PRD-OUT-02` | **Dashboard ou painel visual** para o cliente ver os webhooks | Fora de escopo. É projeto separado do time de frontend. Nesta fase, só endpoints ([09:40] Larissa). |
| `PRD-OUT-03` | **Rate limiting de envio** por cliente | Adiado: "observar e decidir depois" ([09:39] Larissa). |
| `PRD-OUT-04` | **Webhooks de entrada** (o cliente enviando para nós) | Fora: só saída ([09:02] Marcos). |
| `PRD-OUT-05` | **Arquivamento** de eventos entregues (~30 dias) | Fora do escopo desta feature ([09:08] Diego). |
| `PRD-OUT-06` | **Ordem global** dos eventos e vários workers em paralelo | Fora. A ordem vale só por pedido e com um único worker ([09:13] Larissa, [09:14] Marcos). |
| `PRD-OUT-07` | **Entrega exactly-once** | Fora. A garantia é at-least-once ([09:25] Diego). |
| `PRD-OUT-08` | **Itens do pedido no payload** | Fora. O cliente consulta `GET /orders/:id` se precisar ([09:43] Diego). |

## 6. Requisitos funcionais

| ID | Requisito | Origem |
|---|---|---|
| `PRD-FR-01` | O sistema deve permitir **cadastrar um endpoint de webhook** para um customer, informando URL e lista de status de interesse. O `customer_id` vem no body ou no path, nunca do JWT. | [09:31] Marcos, [09:32] Larissa |
| `PRD-FR-02` | A **secret** deve ser **gerada pela plataforma** e devolvida ao cliente na criação. | [09:31] Marcos |
| `PRD-FR-03` | O sistema deve permitir **listar** os webhooks de um customer, **editar** e **remover** um webhook. | [09:33] Bruno |
| `PRD-FR-04` | Cada endpoint deve ter um **filtro de eventos**: a lista de status de pedido que quer receber. O sistema só notifica os status da lista. | [09:33] Marcos, [09:34] Bruno |
| `PRD-FR-05` | Toda **mudança de status** de pedido deve gerar uma notificação para cada endpoint ativo do customer interessado naquele status. | [09:00] Marcos, [09:40] Bruno |
| `PRD-FR-06` | A notificação deve identificar o evento, o pedido, o customer e a transição (status anterior e novo), com data e hora e campos básicos como o total, sem os itens. Deve refletir o pedido **no momento da mudança**. O contrato exato está no FDD. | [09:43] Diego, [09:52] Larissa |
| `PRD-FR-07` | Cada notificação deve ser **assinada com HMAC-SHA256** e trazer em headers a identificação do evento (`X-Event-Id`), do endpoint de origem (`X-Webhook-Id`), a assinatura (`X-Signature`) e o horário do envio (`X-Timestamp`). | [09:20] Sofia, [09:44] Diego, [09:44] Sofia |
| `PRD-FR-08` | O cliente deve poder **pedir uma nova secret** pela API. A anterior continua válida por **24 h**. | [09:21] Sofia |
| `PRD-FR-09` | Falhas de entrega devem ser **retentadas automaticamente** com intervalos crescentes (1m, 5m, 30m, 2h, 12h). | [09:15] Diego, [09:17] Larissa |
| `PRD-FR-10` | Esgotadas as retentativas, o evento deve ir para uma **fila de falhas (DLQ)** com payload, motivo e data. | [09:18] Diego |
| `PRD-FR-11` | Um **ADMIN** deve poder **reprocessar** um item da DLQ, e o sistema deve registrar quem reprocessou. | [09:18] Diego, [09:36] Sofia |
| `PRD-FR-12` | O cliente deve poder consultar o **histórico de entregas** de um endpoint (sucesso ou falha, payload, resposta e tempo de resposta), cobrindo as entregas mais recentes. O exemplo citado foi "os últimos 100". | [09:34] Marcos |
| `PRD-FR-13` | URLs que não sejam **https** devem ser recusadas no cadastro com erro de validação. | [09:23] Sofia |

## 7. Requisitos não funcionais

| ID | Requisito | Origem |
|---|---|---|
| `PRD-NFR-01` | **Latência:** notificação em menos de 10 s, com no máximo 2 s de espera do worker. | [09:02] Marcos, [09:09] Diego |
| `PRD-NFR-02` | **Consistência:** a mudança de status e o registro do evento são atômicos. Se o evento não puder ser registrado, o status não muda. | [09:06] Diego, [09:40] Bruno |
| `PRD-NFR-03` | **Isolamento:** um cliente lento ou fora do ar não pode atrasar a mudança de status de nenhum pedido. | [09:04] Bruno |
| `PRD-NFR-04` | **Entrega at-least-once:** o cliente pode receber duplicatas e deduplica pelo `X-Event-Id`. | [09:24] Diego, [09:26] Larissa |
| `PRD-NFR-05` | **Timeout** de 10 s por chamada ao cliente. | [09:42] Diego |
| `PRD-NFR-06` | **Tamanho máximo** do payload: 64 KB. Acima disso o evento não é enviado (erro, sem truncar). | [09:23] Sofia, [09:24] Larissa |
| `PRD-NFR-07` | **Segurança:** TLS obrigatório, secret única por endpoint, nunca global. | [09:21] Sofia, [09:23] Sofia |
| `PRD-NFR-08` | **Autorização:** o CRUD de configuração exige qualquer usuário autenticado. O reprocessamento da DLQ exige ADMIN. | [09:37] Sofia, [09:36] Larissa |
| `PRD-NFR-09` | **Operação:** o envio roda em processo separado da API e usa o mesmo banco, sem infraestrutura nova. | [09:07] Diego, [09:11] Diego |
| `PRD-NFR-10` | **Manutenibilidade:** o módulo segue os padrões do código (módulo, erros com prefixo `WEBHOOK_`, logger Pino). | [09:30] Larissa |

## 8. Decisões e trade-offs principais

O raciocínio completo de cada decisão está no ADR correspondente. Aqui fica só o efeito no produto.

| Decisão | Efeito para o produto | ADR |
|---|---|---|
| Outbox no MySQL, na mesma transação | Nenhuma mudança de status sem notificação. Em troca, a entrega deixa de ser instantânea. | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Worker separado com polling de 2 s | Atraso de até 2 s antes do envio, aceito pelo PM ([09:10] Marcos). Sem ordem global. | [ADR-002](adrs/ADR-002-worker-separado-em-polling.md) |
| Retentativas em 1m/5m/30m/2h/12h, depois DLQ | Cobre indisponibilidades longas, de até ~14,6 h se forem 5 retentativas além do envio inicial. A contagem exata está pendente (RFC-OQ-06). Depois disso, o reprocessamento é manual. | [ADR-003](adrs/ADR-003-retry-backoff-exponencial-e-dlq.md) |
| HMAC-SHA256 com secret por endpoint e rotação de 24 h | O cliente consegue validar a origem e trocar a secret sem parada. | [ADR-004](adrs/ADR-004-hmac-sha256-secret-por-endpoint.md) |
| At-least-once com `X-Event-Id` | O cliente precisa deduplicar. Isso será destacado no portal ([09:26] Marcos). | [ADR-005](adrs/ADR-005-at-least-once-com-x-event-id.md) |
| Reuso dos padrões do projeto | Menos risco e prazo menor. | [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) |
| Snapshot do payload e filtro na inserção | A notificação mostra o estado do momento da mudança. Mudanças de filtro valem para eventos futuros. | [ADR-007](adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md) |

## 9. Dependências

| ID | Dependência | Origem |
|---|---|---|
| `PRD-DEP-01` | **Revisão de segurança da Sofia**, com pelo menos 2 dias úteis reservados antes do deploy (HMAC e geração de secret) | [09:46] Sofia, [09:49] Sofia |
| `PRD-DEP-02` | **Documentação no portal do desenvolvedor**, sob responsabilidade do PM: semântica at-least-once e como integrar via API | [09:26] Marcos, [09:40] Marcos |
| `PRD-DEP-03` | **Capacidade do time:** 3 sprints (outbox e DLQ: 1; worker e retry: 1; CRUD e histórico: ½; integração com pedidos e testes: ½; HMAC e validações: o restante) | [09:46] Larissa |
| `PRD-DEP-04` | **Confirmação do prazo com a Atlas**, pelo PM | [09:47] Marcos |
| `PRD-DEP-05` | **Novo processo em produção** (worker), no mesmo banco MySQL | [09:11] Diego |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|---|
| `PRD-RISK-01` | Atraso na entrega e **perda da Atlas** para o concorrente | Média | Alto | Escopo enxuto (email, dashboard e rate limiting fora). Estimativa de 3 sprints com a revisão de segurança incluída. Prazo a confirmar com a Atlas pelo PM ([09:46] Larissa, [09:47] Marcos). |
| `PRD-RISK-02` | Cliente **processar o mesmo evento duas vezes** por não deduplicar | Média | Médio | `X-Event-Id` estável e documentação em destaque no portal ([09:25] Sofia, [09:26] Marcos). |
| `PRD-RISK-03` | **Vazamento de secret** de um cliente, como já aconteceu em log de aplicação dele | Baixa | Alto | Secret por endpoint, rotação com 24 h de carência e revisão de segurança ([09:21] Sofia, [09:22] Diego). |
| `PRD-RISK-04` | **Rajada de notificações** para um cliente quando muitos pedidos mudam de uma vez | Média | Baixo | Monitorar. O rate limiting será decidido com dados ([09:38] Diego, [09:39] Larissa). |
| `PRD-RISK-05` | Cliente receber **eventos do mesmo pedido fora de ordem** | Baixa | Médio | Limitação conhecida e documentada ([09:13] Larissa). Mitigação proposta: o payload traz status anterior e novo, e o cliente reconcilia por eles. |
| `PRD-RISK-06` | Indisponibilidade do cliente **maior que ~14,6 h** | Baixa | Médio | Os eventos ficam na DLQ e um ADMIN os reprocessa ([09:17] Marcos, [09:18] Diego). |

## 11. Critérios de aceitação

| ID | Critério |
|---|---|
| `PRD-AC-01` | Dado um customer com endpoint ativo filtrando `SHIPPED`, quando um pedido dele vai para SHIPPED, o endpoint recebe um POST assinado em menos de 10 s. |
| `PRD-AC-02` | Dado o mesmo endpoint, quando o pedido vai para PAID (fora do filtro), nenhuma notificação é enviada. |
| `PRD-AC-03` | O cadastro com URL `http://` é recusado com erro de validação. Com `https://`, o cadastro devolve a secret uma única vez. |
| `PRD-AC-04` | O cliente consegue validar o `X-Signature` recalculando o HMAC-SHA256 do corpo com a secret recebida. |
| `PRD-AC-05` | Depois de rotacionar a secret, notificações assinadas continuam verificáveis com a secret antiga por 24 h. |
| `PRD-AC-06` | Com o endpoint fora do ar, o sistema retenta em 1m, 5m, 30m, 2h e 12h. Depois disso o evento aparece na DLQ com o motivo. |
| `PRD-AC-07` | Um OPERATOR não consegue reprocessar a DLQ. Um ADMIN consegue, e o reprocessamento registra quem o fez. |
| `PRD-AC-08` | O histórico de entregas mostra, para cada tentativa, sucesso ou falha, payload, resposta e tempo de resposta. |
| `PRD-AC-09` | Se o registro do evento falhar, a mudança de status do pedido não acontece. |

## 12. Estratégia de testes e validação

- `PRD-TEST-01` **Testes de integração da API**, no padrão dos testes existentes (`tests/orders.test.ts`): CRUD, validações (https, filtro), autorização (OPERATOR × ADMIN no replay) e histórico.
- `PRD-TEST-02` **Testes de atomicidade:** mudança de status com e sem endpoint interessado, e falha forçada na publicação (o status precisa permanecer).
- `PRD-TEST-03` **Testes do worker** contra um servidor HTTP local de teste: entrega com sucesso e verificação de assinatura, timeout de 10 s, backoff, DLQ, payload acima de 64 KB e replay.
- `PRD-TEST-04` **Teste ponta a ponta** com API e worker rodando, previsto no plano da sprint ("integração no order.service e testes ponta a ponta", [09:46] Larissa).
- `PRD-TEST-05` **Revisão de segurança** da Sofia sobre HMAC e geração de secret antes do deploy ([09:46] Sofia).
