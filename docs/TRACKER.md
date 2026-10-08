# Tracker de Rastreabilidade — Webhooks de Notificação de Pedidos

Este tracker liga cada item identificável do pacote à sua origem: uma fala da reunião ([`TRANSCRICAO.md`](../TRANSCRICAO.md)) ou um arquivo do código.

Como ler:

- **ID**: o mesmo identificador que aparece entre crases no documento (por exemplo `PRD-FR-03`, `RFC-ALT-02`, `FDD-CONTRATO-05`, `ADR-004-D3`). Buscar o ID no documento leva direto ao item.
- **Fonte / Localização**: `TRANSCRICAO` + `[hh:mm] Nome` da fala que origina o item, ou `CODIGO` + caminho do arquivo existente que o fundamenta. Quando o item cita mais de uma fala, a linha registra a mais decisiva. As demais aparecem entre parênteses no próprio documento.
- Resumos que começam com **"Proposta:"** ou **"Derivado:"** marcam detalhes de desenho que não foram ditos literalmente na reunião. São escolhas de implementação ou deduções. A âncora indica em qual fala ou arquivo elas se apoiam. Os pontos que dependem de confirmação estão nas Questões em aberto do [RFC](RFC.md#5-questões-em-aberto).

Distribuição: ver a tabela [Resumo](#resumo) no fim.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
|---|---|---|---|---|---|
| PRD-PROB-01 | `docs/PRD.md` | Problema | Três clientes B2B (Atlas, MaxDistribuição, Nova Cargo) pediram formalmente notificação quando o status dos pedidos muda. | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-02 | `docs/PRD.md` | Problema | Clientes consultam GET /orders periodicamente, deixando a integração lenta e cara para eles. | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-03 | `docs/PRD.md` | Problema | Risco comercial: Atlas pode migrar para concorrente se não houver entrega até o fim do trimestre. | TRANSCRICAO | [09:00] Marcos |
| PRD-CEN-01 | `docs/PRD.md` | Cenário | Cliente cadastra endpoint filtrando só SHIPPED e DELIVERED e deixa de fazer polling em GET /orders. | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-02 | `docs/PRD.md` | Cenário | Usuário que representa o cliente cadastra URL, guarda a secret e consulta histórico de entregas com payload, resposta e tempo. | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-03 | `docs/PRD.md` | Cenário | ADMIN reprocessa eventos da fila de falhas depois que o endpoint do cliente volta de indisponibilidade longa. | TRANSCRICAO | [09:18] Diego |
| PRD-CEN-04 | `docs/PRD.md` | Cenário | Cliente com vários endpoints identifica o cadastro de origem de cada chamada pelo X-Webhook-Id. | TRANSCRICAO | [09:44] Sofia |
| PRD-CEN-05 | `docs/PRD.md` | Cenário | Cliente cuja secret vazou pede nova secret e tem 24 h para trocar sem perder notificações. | TRANSCRICAO | [09:21] Sofia |
| PRD-MET-01 | `docs/PRD.md` | Métrica | Tempo entre commit da mudança de status e primeira tentativa de entrega menor que 10 s; polling contribui no máximo 2 s. | TRANSCRICAO | [09:02] Marcos |
| PRD-MET-02 | `docs/PRD.md` | Métrica | Proposta: 3 de 3 clientes solicitantes integrados até fim de novembro, ancorada no pedido dos três clientes e no prazo da Atlas. | TRANSCRICAO | [09:00] Marcos |
| PRD-MET-03 | `docs/PRD.md` | Métrica | Zero mudanças de status com endpoint interessado e sem evento na outbox, por garantia transacional. | TRANSCRICAO | [09:40] Bruno |
| PRD-MET-04 | `docs/PRD.md` | Métrica | Derivado: zero eventos perdidos em silêncio; todo evento termina entregue ou na DLQ com motivo, ancorado no desenho retry mais DLQ. | TRANSCRICAO | [09:18] Diego |
| PRD-MET-05 | `docs/PRD.md` | Métrica | Retentativas cobrem cerca de 14,6 h (1m+5m+30m+2h+12h) antes de declarar falha; contagem exata pendente. | TRANSCRICAO | [09:17] Diego |
| PRD-OUT-01 | `docs/PRD.md` | Fora de Escopo | Email ao cliente quando o webhook falha fica fora desta fase, talvez na próxima após medir impacto. | TRANSCRICAO | [09:37] Larissa |
| PRD-OUT-02 | `docs/PRD.md` | Fora de Escopo | Dashboard ou painel visual fica fora; é projeto do time de frontend, nesta fase só endpoints. | TRANSCRICAO | [09:40] Larissa |
| PRD-OUT-03 | `docs/PRD.md` | Fora de Escopo | Rate limiting de envio por cliente adiado: observar e decidir depois. | TRANSCRICAO | [09:39] Larissa |
| PRD-OUT-04 | `docs/PRD.md` | Fora de Escopo | Webhooks de entrada (cliente enviando para a plataforma) estão fora; só saída. | TRANSCRICAO | [09:02] Marcos |
| PRD-OUT-05 | `docs/PRD.md` | Fora de Escopo | Arquivamento de eventos entregues após cerca de 30 dias está fora do escopo desta feature. | TRANSCRICAO | [09:08] Diego |
| PRD-OUT-06 | `docs/PRD.md` | Fora de Escopo | Ordem global e múltiplos workers em paralelo ficam fora; ordem vale só por pedido com worker único. | TRANSCRICAO | [09:13] Larissa |
| PRD-OUT-07 | `docs/PRD.md` | Fora de Escopo | Entrega exactly-once fica fora; a garantia é at-least-once. | TRANSCRICAO | [09:25] Diego |
| PRD-OUT-08 | `docs/PRD.md` | Fora de Escopo | Itens do pedido não vão no payload; cliente consulta GET /orders/:id se precisar. | TRANSCRICAO | [09:43] Diego |
| PRD-FR-01 | `docs/PRD.md` | Requisito Funcional | Cadastrar endpoint de webhook para um customer com URL e status de interesse; customer_id no body ou path, nunca do JWT. | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | `docs/PRD.md` | Requisito Funcional | A secret é gerada pela plataforma e devolvida ao cliente na criação. | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-03 | `docs/PRD.md` | Requisito Funcional | Listar os webhooks de um customer, editar e remover um webhook. | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | `docs/PRD.md` | Requisito Funcional | Cada endpoint tem filtro de eventos com a lista de status desejados; só esses status são notificados. | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-05 | `docs/PRD.md` | Requisito Funcional | Toda mudança de status gera notificação para cada endpoint ativo do customer interessado naquele status. | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-06 | `docs/PRD.md` | Requisito Funcional | Notificação identifica evento, pedido, customer, transição, data/hora e total, sem itens, refletindo o pedido no momento da mudança. | TRANSCRICAO | [09:43] Diego |
| PRD-FR-07 | `docs/PRD.md` | Requisito Funcional | Notificação assinada com HMAC-SHA256 e headers X-Event-Id, X-Webhook-Id, X-Signature e X-Timestamp. | TRANSCRICAO | [09:44] Diego |
| PRD-FR-08 | `docs/PRD.md` | Requisito Funcional | Cliente pode pedir nova secret pela API; a anterior continua válida por 24 h. | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-09 | `docs/PRD.md` | Requisito Funcional | Falhas de entrega são retentadas automaticamente com intervalos 1m, 5m, 30m, 2h, 12h. | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-10 | `docs/PRD.md` | Requisito Funcional | Esgotadas as retentativas, o evento vai para a DLQ com payload, motivo e data. | TRANSCRICAO | [09:18] Diego |
| PRD-FR-11 | `docs/PRD.md` | Requisito Funcional | ADMIN pode reprocessar item da DLQ e o sistema registra quem reprocessou. | TRANSCRICAO | [09:36] Sofia |
| PRD-FR-12 | `docs/PRD.md` | Requisito Funcional | Cliente consulta histórico de entregas do endpoint (sucesso/falha, payload, resposta, tempo), entregas recentes, exemplo últimos 100. | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-13 | `docs/PRD.md` | Requisito Funcional | URLs que não sejam https são recusadas no cadastro com erro de validação. | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-01 | `docs/PRD.md` | Requisito Não Funcional | Latência: notificação em menos de 10 s, com no máximo 2 s de espera do worker. | TRANSCRICAO | [09:02] Marcos |
| PRD-NFR-02 | `docs/PRD.md` | Requisito Não Funcional | Consistência: mudança de status e registro do evento atômicos; se o evento falhar, o status não muda. | TRANSCRICAO | [09:40] Bruno |
| PRD-NFR-03 | `docs/PRD.md` | Requisito Não Funcional | Isolamento: cliente lento ou fora do ar não pode atrasar a mudança de status de nenhum pedido. | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-04 | `docs/PRD.md` | Requisito Não Funcional | Entrega at-least-once; cliente pode receber duplicatas e deduplica pelo X-Event-Id. | TRANSCRICAO | [09:26] Larissa |
| PRD-NFR-05 | `docs/PRD.md` | Requisito Não Funcional | Timeout de 10 s por chamada ao cliente. | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-06 | `docs/PRD.md` | Requisito Não Funcional | Payload máximo de 64 KB; acima disso o evento não é enviado, com erro e sem truncar. | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-07 | `docs/PRD.md` | Requisito Não Funcional | Segurança: TLS obrigatório e secret única por endpoint, nunca global. | TRANSCRICAO | [09:21] Sofia |
| PRD-NFR-08 | `docs/PRD.md` | Requisito Não Funcional | Autorização: CRUD de configuração para qualquer usuário autenticado; reprocessamento da DLQ exige ADMIN. | TRANSCRICAO | [09:36] Larissa |
| PRD-NFR-09 | `docs/PRD.md` | Requisito Não Funcional | Envio roda em processo separado da API, no mesmo banco, sem infraestrutura nova. | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-10 | `docs/PRD.md` | Requisito Não Funcional | Manutenibilidade: módulo segue padrões do código (módulo, erros com prefixo WEBHOOK_, logger Pino). | TRANSCRICAO | [09:30] Larissa |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | Revisão de segurança da Sofia com pelo menos 2 dias úteis antes do deploy, cobrindo HMAC e geração de secret. | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Documentação no portal do desenvolvedor pelo PM: semântica at-least-once e integração via API. | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Capacidade do time: 3 sprints divididas entre outbox/DLQ, worker/retry, CRUD/histórico, integração/testes e HMAC/validações. | TRANSCRICAO | [09:46] Larissa |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Confirmação do prazo com a Atlas, pelo PM. | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | Novo processo em produção (worker) usando o mesmo banco MySQL. | TRANSCRICAO | [09:11] Diego |
| PRD-RISK-01 | `docs/PRD.md` | Risco | Atraso na entrega e perda da Atlas para concorrente; mitigado por escopo enxuto e estimativa de 3 sprints. | TRANSCRICAO | [09:00] Marcos |
| PRD-RISK-02 | `docs/PRD.md` | Risco | Cliente processar o mesmo evento duas vezes por não deduplicar; mitigado por X-Event-Id e documentação no portal. | TRANSCRICAO | [09:25] Sofia |
| PRD-RISK-03 | `docs/PRD.md` | Risco | Vazamento de secret de cliente, como já ocorreu em log; mitigado por secret por endpoint, rotação e revisão. | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-04 | `docs/PRD.md` | Risco | Rajada de notificações quando muitos pedidos mudam juntos; monitorar e decidir rate limiting com dados. | TRANSCRICAO | [09:38] Diego |
| PRD-RISK-05 | `docs/PRD.md` | Risco | Derivado: eventos do mesmo pedido fora de ordem durante retry (caso não discutido; reunião limitou ordem a order_id e worker único); reconciliar por status. | TRANSCRICAO | [09:13] Larissa |
| PRD-RISK-06 | `docs/PRD.md` | Risco | Indisponibilidade do cliente maior que cerca de 14,6 h; eventos ficam na DLQ e ADMIN reprocessa. | TRANSCRICAO | [09:17] Marcos |
| PRD-AC-01 | `docs/PRD.md` | Critério de Aceite | Derivado do filtro e da meta de 10 s: endpoint filtrando SHIPPED recebe POST assinado em menos de 10 s. | TRANSCRICAO | [09:33] Marcos |
| PRD-AC-02 | `docs/PRD.md` | Critério de Aceite | Derivado do filtro na inserção: pedido indo para PAID fora do filtro não gera notificação. | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-03 | `docs/PRD.md` | Critério de Aceite | Derivado: cadastro com http:// recusado com erro de validação; com https:// devolve a secret uma única vez. | TRANSCRICAO | [09:23] Sofia |
| PRD-AC-04 | `docs/PRD.md` | Critério de Aceite | Derivado: cliente valida X-Signature recalculando HMAC-SHA256 do corpo com a secret recebida. | TRANSCRICAO | [09:20] Sofia |
| PRD-AC-05 | `docs/PRD.md` | Critério de Aceite | Derivado: após rotação, notificações continuam verificáveis com a secret antiga por 24 h. | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-06 | `docs/PRD.md` | Critério de Aceite | Derivado: com endpoint fora do ar, retenta em 1m, 5m, 30m, 2h, 12h e depois o evento aparece na DLQ com motivo. | TRANSCRICAO | [09:17] Larissa |
| PRD-AC-07 | `docs/PRD.md` | Critério de Aceite | Derivado: OPERATOR não consegue reprocessar a DLQ; ADMIN consegue e o reprocessamento registra quem o fez. | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-08 | `docs/PRD.md` | Critério de Aceite | Derivado: histórico mostra por tentativa sucesso ou falha, payload, resposta e tempo de resposta. | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-09 | `docs/PRD.md` | Critério de Aceite | Derivado: se o registro do evento falhar, a mudança de status do pedido não acontece. | TRANSCRICAO | [09:40] Bruno |
| PRD-TEST-01 | `docs/PRD.md` | Teste | Proposta: testes de integração da API no padrão dos testes existentes cobrindo CRUD, validações, autorização e histórico. | CODIGO | `tests/orders.test.ts` |
| PRD-TEST-02 | `docs/PRD.md` | Teste | Derivado da atomicidade: testar mudança de status com e sem endpoint interessado e falha forçada na publicação. | TRANSCRICAO | [09:40] Bruno |
| PRD-TEST-03 | `docs/PRD.md` | Teste | Proposta: testes do worker contra servidor HTTP local (assinatura, timeout, backoff, DLQ, 64 KB, replay), ancorada no plano de sprints. | TRANSCRICAO | [09:46] Larissa |
| PRD-TEST-04 | `docs/PRD.md` | Teste | Teste ponta a ponta com API e worker rodando, previsto no plano da sprint. | TRANSCRICAO | [09:46] Larissa |
| PRD-TEST-05 | `docs/PRD.md` | Teste | Revisão de segurança da Sofia sobre HMAC e geração de secret antes do deploy. | TRANSCRICAO | [09:46] Sofia |
| RFC-TLDR-01 | `docs/RFC.md` | Decisão | Webhooks de saída via outbox MySQL transacional, worker separado com polling 2 s, HMAC, at-least-once, backoff e DLQ com replay ADMIN. | TRANSCRICAO | [09:48] Larissa |
| RFC-CTX-01 | `docs/RFC.md` | Contexto | Três clientes B2B querem saber em tempo real das mudanças de status e hoje fazem polling em GET /orders. | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | `docs/RFC.md` | Contexto | OMS sem notificação externa; changeStatus aplica transição numa transação que atualiza orders, grava histórico e mexe no estoque. | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-PROP-01 | `docs/RFC.md` | Decisão | CRUD autenticado de endpoints com URL https, status de interesse, estado ativo e secret gerada, rotacionável com 24 h. | TRANSCRICAO | [09:31] Marcos |
| RFC-PROP-02 | `docs/RFC.md` | Decisão | changeStatus chama função de publicação que recebe o tx e grava evento por endpoint interessado com payload renderizado. | TRANSCRICAO | [09:41] Bruno |
| RFC-PROP-03 | `docs/RFC.md` | Decisão | Worker único em processo próprio, polling 2 s, assina, envia com timeout 10 s e registra tentativas no histórico. | TRANSCRICAO | [09:09] Diego |
| RFC-PROP-04 | `docs/RFC.md` | Decisão | Falhas: backoff exponencial com teto, DLQ em tabela separada e replay manual restrito a ADMIN e auditado. | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-05 | `docs/RFC.md` | Decisão | Contrato com o cliente: at-least-once com X-Event-Id estável para dedup e payload enxuto sem itens. | TRANSCRICAO | [09:26] Larissa |
| RFC-PROP-06 | `docs/RFC.md` | Decisão | Módulo segue padrões: src/modules/webhooks, AppError com WEBHOOK_*, Pino, error middleware, Zod e requireRole, sem dependência nova. | TRANSCRICAO | [09:30] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo HTTP síncrono em changeStatus descartado: acopla latência e disponibilidade do cliente à transação de pedidos. | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Redis Streams ou fila dedicada descartada: infraestrutura nova para time pequeno, overengineering frente à outbox no MySQL. | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger MySQL ou LISTEN/NOTIFY descartado: MySQL não tem listener nativo; polling de 2 s cabe nos 10 s. | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Exactly-once descartado: exigiria coordenação entre as duas partes com complexidade bem maior. | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa Descartada | Secret global da plataforma descartada: vazamento de uma comprometeria todos os clientes. | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-06 | `docs/RFC.md` | Alternativa Descartada | Retry indefinido ou só 3 tentativas descartados: eventos pendurados ou morte em cerca de 30 min, menos que indisponibilidades de 2 h. | TRANSCRICAO | [09:16] Diego |
| RFC-OQ-01 | `docs/RFC.md` | Questão em Aberto | Rate limiting de saída por cliente; fora da v1, observar e decidir depois. | TRANSCRICAO | [09:39] Diego |
| RFC-OQ-02 | `docs/RFC.md` | Questão em Aberto | Aviso ao cliente quando o webhook falha, por exemplo email; próxima fase após medir impacto. | TRANSCRICAO | [09:37] Larissa |
| RFC-OQ-03 | `docs/RFC.md` | Questão em Aberto | Escala para vários workers particionando por order_id ou com lock pessimista; single-worker na v1. | TRANSCRICAO | [09:13] Diego |
| RFC-OQ-04 | `docs/RFC.md` | Questão em Aberto | Endurecer permissões do CRUD de configuração, hoje aberto a qualquer role autenticada; mais pra frente. | TRANSCRICAO | [09:37] Sofia |
| RFC-OQ-05 | `docs/RFC.md` | Questão em Aberto | Arquivamento das linhas entregues após cerca de 30 dias, fora do escopo desta feature. | TRANSCRICAO | [09:08] Diego |
| RFC-OQ-06 | `docs/RFC.md` | Questão em Aberto | Derivado: divergência entre 5 intervalos (1 envio + 5 retentativas) e resumo final total 5 tentativas; Larissa precisa confirmar. | TRANSCRICAO | [09:48] Larissa |
| RFC-OQ-07 | `docs/RFC.md` | Questão em Aberto | Derivado: evento em backoff pode deixar o próximo do mesmo pedido sair antes, mesmo com worker único; ancorado na ordering por created_at. | TRANSCRICAO | [09:12] Diego |
| RFC-OQ-08 | `docs/RFC.md` | Questão em Aberto | Derivado: guarda da secret em repouso e inclusão do X-Timestamp na assinatura, para a revisão de segurança da Sofia. | TRANSCRICAO | [09:46] Sofia |
| RFC-RISK-01 | `docs/RFC.md` | Risco | Derivado: transação de changeStatus fica mais longa ao consultar endpoints e inserir eventos; sem HTTP na transação. | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-RISK-02 | `docs/RFC.md` | Risco | Outbox cresce sem arquivamento; índices agora e arquivamento como trabalho futuro. | TRANSCRICAO | [09:08] Diego |
| RFC-RISK-03 | `docs/RFC.md` | Risco | Derivado: worker único como ponto único de vazão, endpoints lentos atrasam outros; ancorado na decisão single-worker. | TRANSCRICAO | [09:12] Diego |
| RFC-RISK-04 | `docs/RFC.md` | Risco | Derivado: ordem por pedido não garantida durante retry, caso não discutido; limitação registrada em RFC-OQ-07, payload permite reconciliação. | TRANSCRICAO | [09:13] Larissa |
| RFC-RISK-05 | `docs/RFC.md` | Risco | Proposta: worker parado sem ninguém perceber acumula eventos; alerta sobre idade do pendente mais antigo, ancorado no processo separado. | TRANSCRICAO | [09:11] Diego |
| FDD-OBJ-01 | `docs/FDD.md` | Objetivo | Mudança de status com endpoint interessado gera evento na outbox na mesma transação (Derivado: uma linha por endpoint); falha impede a mudança. | TRANSCRICAO | [09:40] Bruno |
| FDD-OBJ-02 | `docs/FDD.md` | Objetivo | Primeira tentativa sai em até 2 s após o commit mais o tempo HTTP, abaixo dos 10 s de "tempo real". | TRANSCRICAO | [09:02] Marcos |
| FDD-OBJ-03 | `docs/FDD.md` | Objetivo | Nenhuma chamada HTTP dentro da transação de pedidos. | TRANSCRICAO | [09:04] Bruno |
| FDD-OBJ-04 | `docs/FDD.md` | Objetivo | Todo envio é assinado com HMAC-SHA256 e identificável por X-Event-Id e X-Webhook-Id. | TRANSCRICAO | [09:22] Sofia |
| FDD-OBJ-05 | `docs/FDD.md` | Objetivo | Todo evento termina DELIVERED ou na DLQ com motivo; nenhum fica pendurado. | TRANSCRICAO | [09:15] Diego |
| FDD-OBJ-06 | `docs/FDD.md` | Objetivo | Zero dependência nova de runtime e zero infraestrutura nova. | TRANSCRICAO | [09:07] Diego |
| FDD-EXC-01 | `docs/FDD.md` | Fora de Escopo | Webhooks de entrada (cliente enviando para o OMS) não serão implementados. | TRANSCRICAO | [09:02] Marcos |
| FDD-EXC-02 | `docs/FDD.md` | Fora de Escopo | Aviso por email ao cliente quando o webhook falha fica fora desta fase. | TRANSCRICAO | [09:37] Larissa |
| FDD-EXC-03 | `docs/FDD.md` | Fora de Escopo | Rate limiting de saída não será implementado; a decisão é observar antes. | TRANSCRICAO | [09:39] Larissa |
| FDD-EXC-04 | `docs/FDD.md` | Fora de Escopo | Dashboard ou painel visual fora do escopo. | TRANSCRICAO | [09:40] Larissa |
| FDD-EXC-05 | `docs/FDD.md` | Fora de Escopo | Arquivamento das linhas entregues (~30 dias) fora do escopo. | TRANSCRICAO | [09:08] Diego |
| FDD-EXC-06 | `docs/FDD.md` | Fora de Escopo | Vários workers em paralelo e ordem global não fazem parte desta entrega. | TRANSCRICAO | [09:13] Larissa |
| FDD-EXC-07 | `docs/FDD.md` | Fora de Escopo | Items do pedido não entram no payload. | TRANSCRICAO | [09:43] Diego |
| FDD-EXC-08 | `docs/FDD.md` | Fora de Escopo | Derivado: sem evento na criação do pedido; OrderService.create grava histórico inicial com fromStatus null fora de changeStatus. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-EXC-09 | `docs/FDD.md` | Fora de Escopo | Derivado: sem endpoint para listar a DLQ, só o replay foi pedido; ADMIN consulta a tabela webhook_dead_letter. | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-01 | `docs/FDD.md` | Modelo de Dados | Outbox com índice em status e em created_at; Proposta: índice de status composto com nextAttemptAt para a consulta do worker. | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-02 | `docs/FDD.md` | Modelo de Dados | O id da linha da outbox é o event_id UUID; Derivado: cada endpoint interessado recebe linha e event_id próprios (filtro na inserção). | TRANSCRICAO | [09:25] Diego |
| FDD-DADOS-03 | `docs/FDD.md` | Modelo de Dados | Proposta: relações inversas e política de remoção (cascade endpoint/outbox/histórico, DLQ sem FK), ancorada nas convenções do schema Prisma. | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-04 | `docs/FDD.md` | Modelo de Dados | Secret em coluna própria, nunca devolvida em leituras; só aparece na criação e rotação. Cifrar em repouso pendente. | TRANSCRICAO | [09:31] Marcos |
| FDD-FLUXO-01 | `docs/FDD.md` | Fluxo | publishWebhookEvent(tx, order, from, to) dentro da transação filtra endpoints ativos por events e insere uma linha PENDING por endpoint. | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-02 | `docs/FDD.md` | Fluxo | Worker faz polling a cada 2 s, reserva lote PENDING, assina, envia; Proposta: histórico e status gravados na mesma transação. | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-02a | `docs/FDD.md` | Fluxo | Lote processado em sequência por created_at; Proposta: tamanho vira WEBHOOK_WORKER_BATCH_SIZE (padrão 10, a calibrar); endpoint lento atrasa o lote. | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-02b | `docs/FDD.md` | Fluxo | Derivado: recuperação de PROCESSING para PENDING no boot, segura só por haver um único worker; reenvio coberto pelo at-least-once. | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-02c | `docs/FDD.md` | Fluxo | Proposta: shutdown gracioso em SIGINT/SIGTERM com prisma.$disconnect(), no modelo de src/server.ts. | CODIGO | `src/server.ts` |
| FDD-FLUXO-03 | `docs/FDD.md` | Fluxo | Retry 1m/5m/30m/2h/12h; Derivado (leitura 1+5, pendente RFC-OQ-06): após a 6ª falha vai para DLQ com WEBHOOK_MAX_ATTEMPTS_EXCEEDED. | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-04 | `docs/FDD.md` | Fluxo | Numa transação marca outbox FAILED e cria linha em webhook_dead_letter com payload, motivo e tentativas; loga webhook_dead_lettered. | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-05 | `docs/FDD.md` | Fluxo | Replay ADMIN recoloca o evento na outbox como PENDING; Proposta: mesma linha e mesmo event_id, com replayedAt/replayedById. | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-06 | `docs/FDD.md` | Fluxo | Rotação guarda secret anterior válida 24 h e devolve a nova uma vez; proposta recusar nova rotação durante a carência. | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | POST /api/v1/customers/:customerId/webhooks cria endpoint com url https e events; secret gerada e devolvida só aqui. | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | GET /api/v1/customers/:customerId/webhooks lista paginada dos endpoints do customer, sem secret. | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | PATCH /api/v1/webhooks/:id edita url, events e active; events vale para inserções futuras, url e active também para pendentes. | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato | DELETE /api/v1/webhooks/:id responde 204; Proposta (FDD-DADOS-03): cascata em pendentes e histórico, DLQ preservada. | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato | POST /api/v1/webhooks/:id/rotate-secret emite nova secret; anterior vale 24 h; Proposta: 409 se já houver carência ativa. | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | GET /api/v1/webhooks/:id/deliveries lista tentativas decrescentes com sucesso, payload, resposta e duração; pageSize até 100. | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | POST /api/v1/admin/webhooks/dead-letter/:id/replay exige ADMIN, responde 202 e registra quem fez o replay. | TRANSCRICAO | [09:36] Sofia |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | Chamada de saída POST JSON com X-Event-Id, X-Webhook-Id, X-Timestamp, X-Signature e payload enxuto; 2xx em 10 s é entregue. | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | Assinatura HMAC-SHA256 do corpo bruto com a secret; formato sha256= e duas assinaturas na carência são propostas. | TRANSCRICAO | [09:22] Sofia |
| FDD-RES-01 | `docs/FDD.md` | Resiliência | Timeout de 10 s via AbortSignal.timeout; estouro é falha retentável WEBHOOK_DELIVERY_TIMEOUT. | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | `docs/FDD.md` | Resiliência | Derivado do TLS obrigatório: só 2xx é sucesso, 3xx não é seguido, 4xx/5xx retentam. | TRANSCRICAO | [09:23] Sofia |
| FDD-RES-03 | `docs/FDD.md` | Resiliência | Retentativas com backoff 1m/5m/30m/2h/12h e depois DLQ. | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-04 | `docs/FDD.md` | Resiliência | DLQ mais replay manual por ADMIN são o único fallback; sem canal alternativo. | TRANSCRICAO | [09:18] Diego |
| FDD-RES-05 | `docs/FDD.md` | Resiliência | Cliente lento nunca afeta a API (outro processo); no worker pode atrasar outros do mesmo lote. | TRANSCRICAO | [09:04] Bruno |
| FDD-RES-06 | `docs/FDD.md` | Resiliência | Derivado do at-least-once: ao reiniciar, worker devolve PROCESSING para PENDING; reenvio possível é aceito. | TRANSCRICAO | [09:24] Diego |
| FDD-RES-07 | `docs/FDD.md` | Resiliência | Derivado da outbox: banco indisponível gera log de erro e o loop continua no próximo ciclo; eventos preservados. | TRANSCRICAO | [09:06] Diego |
| FDD-RES-08 | `docs/FDD.md` | Resiliência | Payload acima de 64 KB não é enviado nem truncado; Proposta: vai para DLQ com WEBHOOK_PAYLOAD_TOO_LARGE. | TRANSCRICAO | [09:23] Sofia |
| FDD-RES-09 | `docs/FDD.md` | Resiliência | Falha ao publicar na outbox faz rollback da mudança de status; nunca há status mudado sem evento. | TRANSCRICAO | [09:40] Bruno |
| FDD-ERR-01 | `docs/FDD.md` | Erro | WEBHOOK_NOT_FOUND 404 para id de endpoint inexistente em PATCH, DELETE, rotate e deliveries. | TRANSCRICAO | [09:28] Bruno |
| FDD-ERR-02 | `docs/FDD.md` | Erro | Proposta: WEBHOOK_CUSTOMER_NOT_FOUND 404 para customerId inexistente, seguindo o prefixo WEBHOOK_ definido na reunião. | TRANSCRICAO | [09:29] Larissa |
| FDD-ERR-03 | `docs/FDD.md` | Erro | WEBHOOK_INVALID_URL em details de VALIDATION_ERROR (400) quando a URL não é https. | TRANSCRICAO | [09:23] Sofia |
| FDD-ERR-04 | `docs/FDD.md` | Erro | Derivado do filtro de eventos: WEBHOOK_INVALID_EVENT_FILTER em details (400) para events vazio, com PENDING ou desconhecido. | TRANSCRICAO | [09:33] Marcos |
| FDD-ERR-05 | `docs/FDD.md` | Erro | Derivado do replay: WEBHOOK_DEAD_LETTER_NOT_FOUND 404 para id de DLQ inexistente. | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-06 | `docs/FDD.md` | Erro | Proposta ancorada no replay: WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED 409 quando replayedAt já está preenchido. | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-07 | `docs/FDD.md` | Erro | Proposta ancorada no replay: WEBHOOK_ENDPOINT_INACTIVE 409 no replay com endpoint desativado ou removido. | TRANSCRICAO | [09:18] Diego |
| FDD-ERR-15 | `docs/FDD.md` | Erro | Proposta ancorada na rotação com uma secret anterior: WEBHOOK_SECRET_ROTATION_IN_PROGRESS 409 durante carência ativa. | TRANSCRICAO | [09:21] Sofia |
| FDD-ERR-08 | `docs/FDD.md` | Erro | Derivado do timeout de 10 s: WEBHOOK_DELIVERY_TIMEOUT, retentável, quando não há resposta em 10 s; segue backoff. | TRANSCRICAO | [09:42] Diego |
| FDD-ERR-09 | `docs/FDD.md` | Erro | Derivado do retry com backoff: WEBHOOK_DELIVERY_HTTP_ERROR retentável para resposta não 2xx. | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-10 | `docs/FDD.md` | Erro | Derivado do retry com backoff: WEBHOOK_DELIVERY_NETWORK_ERROR retentável para DNS, conexão recusada ou falha TLS. | TRANSCRICAO | [09:15] Diego |
| FDD-ERR-11 | `docs/FDD.md` | Erro | Derivado (leitura 1+5, RFC-OQ-06): WEBHOOK_MAX_ATTEMPTS_EXCEEDED quando o 6º envio falha; destino DLQ. | TRANSCRICAO | [09:17] Larissa |
| FDD-ERR-12 | `docs/FDD.md` | Erro | Derivado do limite de 64 KB: WEBHOOK_PAYLOAD_TOO_LARGE, não retentável, corpo acima de 65.536 bytes; DLQ direto. | TRANSCRICAO | [09:24] Larissa |
| FDD-ERR-13 | `docs/FDD.md` | Erro | Derivado do estado ativo do endpoint: WEBHOOK_ENDPOINT_INACTIVE não retentável quando desativado após inserção; DLQ direto. | TRANSCRICAO | [09:21] Bruno |
| FDD-ERR-14 | `docs/FDD.md` | Erro | WEBHOOK_SECRET_REQUIRED não retentável, defensivo, para endpoint sem secret ao assinar; DLQ direto. | TRANSCRICAO | [09:28] Bruno |
| FDD-OBS-01 | `docs/FDD.md` | Observabilidade | Logs JSON estruturados via Pino existente, eventos snake_case, logger.child no worker, redação de secret, nada logado na transação. | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | `docs/FDD.md` | Observabilidade | Derivado de não adicionar libs: métricas (lag, backlog, latência, sucesso, DLQ aberta) por SQL e logs de ciclo. | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-03 | `docs/FDD.md` | Observabilidade | Proposta: sem tracing distribuído; correlação por eventId, orderId e log http_request com requestId do requestLogger existente. | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-INT-01 | `docs/FDD.md` | Integração | Em changeStatus, após orderStatusHistory.create e antes do refresh, chamar publishWebhookEvent(tx, order, from, to) com o TxClient existente. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-02 | `docs/FDD.md` | Integração | Sem alteração; a tabela transitions define os valores aceitos em events (todos exceto PENDING). | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-INT-03 | `docs/FDD.md` | Integração | Schema ganha enum WebhookOutboxStatus, quatro modelos webhook e relações inversas em Customer e User; migration via db:migrate. | CODIGO | `prisma/schema.prisma` |
| FDD-INT-04 | `docs/FDD.md` | Integração | Sem alteração; classes WEBHOOK_* estendem AppError (404) e ConflictError (409) existentes. | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-INT-05 | `docs/FDD.md` | Integração | Sem alteração; errorMiddleware já converte AppError no envelope de erro padrão. | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-INT-06 | `docs/FDD.md` | Integração | Sem alteração; validate usado nas rotas novas, códigos de validação saem em details. | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-INT-07 | `docs/FDD.md` | Integração | authenticate em todas as rotas e requireRole('ADMIN') só no replay, como em user.routes. | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | buildApiRouter registra três montagens de rotas webhook, a de customers com mergeParams antes de /customers. | CODIGO | `src/routes/index.ts` |
| FDD-INT-09 | `docs/FDD.md` | Integração | buildControllers instancia WebhookRepository, WebhookService e WebhookController como faz com orders. | CODIGO | `src/app.ts` |
| FDD-INT-10 | `docs/FDD.md` | Integração | Sem alteração; worker importa o prisma exportado, instância própria por ser outro processo. | CODIGO | `src/config/database.ts` |
| FDD-INT-11 | `docs/FDD.md` | Integração | envSchema ganha WEBHOOK_POLL_INTERVAL_MS, WEBHOOK_HTTP_TIMEOUT_MS e WEBHOOK_WORKER_BATCH_SIZE com padrões; .env.example recebe as chaves. | CODIGO | `src/config/env.ts` |
| FDD-INT-12 | `docs/FDD.md` | Integração | redactPaths ganha '*.secret' e '*.previousSecret'; worker usa logger.child. | CODIGO | `src/shared/logger/index.ts` |
| FDD-INT-13 | `docs/FDD.md` | Integração | Sem alteração; server.ts é modelo do worker (bootstrap, SIGINT/SIGTERM com $disconnect, logger.fatal). | CODIGO | `src/server.ts` |
| FDD-INT-14 | `docs/FDD.md` | Integração | paginated() usado nas listagens de endpoints e entregas. | CODIGO | `src/shared/http/response.ts` |
| FDD-INT-15 | `docs/FDD.md` | Integração | Novos scripts worker e worker:dev; nenhuma dependência nova (uuid já existe, crypto e fetch nativos no Node 20). | CODIGO | `package.json` |
| FDD-INT-16 | `docs/FDD.md` | Integração | beforeEach apaga tabelas webhook antes das demais por FKs; factories ganham createTestWebhook. | CODIGO | `tests/setup.ts` |
| FDD-INT-17 | `docs/FDD.md` | Integração | Sem alteração; delete de customer continua funcionando com webhooks graças ao cascade de FDD-DADOS-03. | CODIGO | `src/modules/customers/customer.service.ts` |
| FDD-INT-18 | `docs/FDD.md` | Integração | Sem alteração; contrato de PATCH /orders/:id/status mantido, detalhes via GET /orders/:id. | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-AC-01 | `docs/FDD.md` | Critério de Aceite | PATCH de status com endpoint ativo e filtro casando cria exatamente uma linha PENDING por endpoint com payload do §6.8. | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-02 | `docs/FDD.md` | Critério de Aceite | Se webhookOutbox.create lança, status, histórico e estoque ficam inalterados. | TRANSCRICAO | [09:40] Bruno |
| FDD-AC-03 | `docs/FDD.md` | Critério de Aceite | Customer sem endpoint ou com filtro sem o status: nenhuma linha inserida. | TRANSCRICAO | [09:34] Bruno |
| FDD-AC-04 | `docs/FDD.md` | Critério de Aceite | Criação com http:// responde 400 com WEBHOOK_INVALID_URL; com https 201 com secret; GET nunca devolve secret. | TRANSCRICAO | [09:23] Sofia |
| FDD-AC-05 | `docs/FDD.md` | Critério de Aceite | Worker entrega com Content-Type, X-Event-Id, X-Webhook-Id, X-Timestamp, X-Signature, e o HMAC-SHA256 confere. | TRANSCRICAO | [09:44] Diego |
| FDD-AC-06 | `docs/FDD.md` | Critério de Aceite | Servidor acima de 10 s gera timeout; falhas seguem 1m/5m/30m/2h/12h; Derivado (leitura 1+5): na 6ª vira FAILED com DLQ. | TRANSCRICAO | [09:17] Larissa |
| FDD-AC-07 | `docs/FDD.md` | Critério de Aceite | Payload acima de 65.536 bytes vai para DLQ com WEBHOOK_PAYLOAD_TOO_LARGE sem chamada HTTP. | TRANSCRICAO | [09:24] Larissa |
| FDD-AC-08 | `docs/FDD.md` | Critério de Aceite | Replay com OPERATOR 403; ADMIN 202 com replayedById e log com userId; Proposta: mesma linha volta PENDING e segundo replay dá 409. | TRANSCRICAO | [09:36] Sofia |
| FDD-AC-09 | `docs/FDD.md` | Critério de Aceite | Proposta: após rotate-secret, envios por 24 h trazem duas assinaturas, depois uma; segunda rotação na carência responde 409. | TRANSCRICAO | [09:21] Sofia |
| FDD-AC-10 | `docs/FDD.md` | Critério de Aceite | GET deliveries lista tentativas decrescentes com success, responseStatus, durationMs e payload; Proposta: pageSize acima de 100 dá 400. | TRANSCRICAO | [09:34] Marcos |
| FDD-AC-11 | `docs/FDD.md` | Critério de Aceite | Com worker parado a API segue mudando status; ao subir, worker entrega o backlog em ordem de created_at. | TRANSCRICAO | [09:11] Diego |
| FDD-AC-12 | `docs/FDD.md` | Critério de Aceite | Nenhum log contém valor de secret ou previousSecret. | TRANSCRICAO | [09:22] Diego |
| FDD-AC-13 | `docs/FDD.md` | Critério de Aceite | Derivado do delete de customer existente: DELETE de customer sem pedidos e com webhook responde 204 e DLQ permanece. | CODIGO | `src/modules/customers/customer.service.ts` |
| FDD-AC-14 | `docs/FDD.md` | Critério de Aceite | Derivado dos scripts do projeto: lint, build e test passam, incluindo tests/orders.test.ts sem alterar asserts. | CODIGO | `package.json` |
| FDD-RISK-01 | `docs/FDD.md` | Risco | Derivado: ordem por pedido quebrada quando um evento entra em backoff e o seguinte é entregue; caso não discutido, decisão em RFC-OQ-07. | TRANSCRICAO | [09:12] Diego |
| FDD-RISK-02 | `docs/FDD.md` | Risco | Reenvio duplicado por queda ou timeout; mitigado por at-least-once com X-Event-Id estável. | TRANSCRICAO | [09:24] Diego |
| FDD-RISK-03 | `docs/FDD.md` | Risco | Derivado do worker em processo separado: worker parado sem ninguém perceber; alerta de lag acima de 10 s. | TRANSCRICAO | [09:11] Diego |
| FDD-RISK-04 | `docs/FDD.md` | Risco | Derivado: secret em claro no banco por exigência do HMAC; cifrar em repouso fica para a revisão da Sofia. | TRANSCRICAO | [09:46] Sofia |
| FDD-RISK-05 | `docs/FDD.md` | Risco | Derivado: X-Timestamp fora da assinatura permite replay de corpo capturado; mitigado por TLS e dedup por X-Event-Id. | TRANSCRICAO | [09:44] Diego |
| FDD-RISK-06 | `docs/FDD.md` | Risco | Derivado: contenção na transação de changeStatus pelos INSERTs na outbox; consulta indexada e monitorar duração do PATCH. | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-RISK-07 | `docs/FDD.md` | Risco | Rajada de envios sem rate limiting; Derivado: worker único limita concorrência a 1; observar e decidir depois. | TRANSCRICAO | [09:38] Diego |
| FDD-RISK-09 | `docs/FDD.md` | Risco | Derivado do envio sequencial: head-of-line blocking de endpoints lentos atrasa clientes saudáveis além dos 10 s. | TRANSCRICAO | [09:02] Marcos |
| FDD-RISK-08 | `docs/FDD.md` | Risco | Qualquer usuário autenticado gerencia webhooks de qualquer customer; endurecimento previsto para depois. | TRANSCRICAO | [09:37] Sofia |
| ADR-001 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Adotar padrão Transactional Outbox no MySQL existente para eventos de mudança de status, sem infraestrutura nova. | TRANSCRICAO | [09:08] Larissa |
| ADR-001-D1 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Inserir o evento em webhook_outbox na mesma transação de changeStatus; uma linha por endpoint interessado é derivação do filtro na inserção. | TRANSCRICAO | [09:06] Diego |
| ADR-001-D2 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Se a inserção na outbox falhar, a transação inteira faz rollback e o status do pedido não muda. | TRANSCRICAO | [09:40] Bruno |
| ADR-001-D3 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Outbox com índices em status e created_at; status PENDING, PROCESSING, FAILED, DELIVERED. | TRANSCRICAO | [09:08] Diego |
| ADR-001-D4 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Id da linha da outbox é UUID, seguindo o padrão do schema do projeto. | TRANSCRICAO | [09:51] Larissa |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Disparo HTTP síncrono dentro de changeStatus: travaria outras mudanças de status e não faz sentido rollback se cliente cair. | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Redis Streams ou fila externa: exigiria infraestrutura nova, overengineering para time pequeno. | TRANSCRICAO | [09:07] Diego |
| ADR-001-CONS-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência | Atomicidade: transação commitada implica evento registrado; rollback descarta o evento junto. | TRANSCRICAO | [09:06] Diego |
| ADR-001-CONS-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência | Derivado: transação de changeStatus ganha consulta de endpoints e INSERT por endpoint, aumentando latência do PATCH de status; ancorado na transação existente. | CODIGO | `src/modules/orders/order.service.ts` |
| ADR-001-CONS-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência | Tabela outbox cresce sem limite; arquivamento de entregues após ~30 dias fica fora do escopo. | TRANSCRICAO | [09:08] Diego |
| ADR-002 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Worker em processo separado da API lendo a outbox por polling a cada 2 segundos. | TRANSCRICAO | [09:10] Larissa |
| ADR-002-D1 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Polling em loop a cada 2s buscando eventos pendentes mais antigos em lote pequeno, processando e marcando resultado. | TRANSCRICAO | [09:09] Diego |
| ADR-002-D2 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Worker roda em processo separado da API (src/worker.ts, npm run worker) para não cair com restart da API. | TRANSCRICAO | [09:11] Diego |
| ADR-002-D3 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Worker usa mesmo banco e stack, com instância própria de PrismaClient e mesma DATABASE_URL. | TRANSCRICAO | [09:30] Bruno |
| ADR-002-D4 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Decisão | Um único worker processando por created_at; ordem só por order_id e enquanto single-worker. | TRANSCRICAO | [09:12] Diego |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Alternativa Descartada | Trigger no banco ou LISTEN/NOTIFY: MySQL não tem listener nativo e trigger não notifica processo externo. | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Alternativa Descartada | Worker dentro do processo da API: se a API reinicia, o worker cai junto. | TRANSCRICAO | [09:11] Diego |
| ADR-002-ALT-03 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Alternativa Descartada | Vários workers paralelos particionados por order_id ou lock pessimista: adiado como problema do futuro. | TRANSCRICAO | [09:13] Diego |
| ADR-002-CONS-01 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Consequência | Atraso de polling de no máximo 2s cabe no requisito de menos de 10 segundos. | TRANSCRICAO | [09:09] Diego |
| ADR-002-CONS-02 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Consequência | Derivado: polling gera consultas constantes mesmo sem eventos, baratas pelo índice em status e created_at. | TRANSCRICAO | [09:08] Diego |
| ADR-002-CONS-03 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Consequência | Derivado: um processo a mais para implantar e monitorar, consequência do worker em processo separado. | TRANSCRICAO | [09:11] Diego |
| ADR-002-CONS-04 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Consequência | Limitação conhecida de ordem por order_id só com um worker; backoff pode inverter ordem (caso não discutido, questão em aberto). | TRANSCRICAO | [09:13] Larissa |
| ADR-002-CONS-05 | `docs/adrs/ADR-002-worker-separado-em-polling.md` | Consequência | Um worker é ponto único de vazão; escalar horizontalmente exige nova decisão com particionamento ou lock. | TRANSCRICAO | [09:13] Diego |
| ADR-003 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Decisão | Retry com backoff exponencial 1m/5m/30m/2h/12h, 5 tentativas, e DLQ em tabela separada com replay manual ADMIN. | TRANSCRICAO | [09:17] Larissa |
| ADR-003-D1 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Decisão | Backoff exponencial com teto de 5 tentativas nos intervalos 1m, 5m, 30m, 2h, 12h; contagem exata pendente (RFC-OQ-06). | TRANSCRICAO | [09:17] Larissa |
| ADR-003-D2 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Decisão | Esgotadas as retentativas, evento copiado para webhook_dead_letter com payload, motivo e timestamp; outbox fica FAILED. | TRANSCRICAO | [09:18] Diego |
| ADR-003-D3 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Decisão | Reprocessamento manual via POST /admin/webhooks/dead-letter/:id/replay, recolocando o evento na outbox como pendente. | TRANSCRICAO | [09:18] Diego |
| ADR-003-D4 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Decisão | Replay exige role ADMIN via requireRole existente e registra quem executou, para auditoria. | TRANSCRICAO | [09:36] Sofia |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Retry indefinido com backoff: evento ficaria pendurado para sempre se o cliente sumiu. | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Alternativa Descartada | 3 tentativas: acabaria em ~30 minutos, insuficiente para indisponibilidades de duas horas já ocorridas. | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Marcar failed na própria outbox sem tabela de DLQ: tabela separada deixa outbox limpa e guarda evidência. | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-01 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Consequência | Janela de ~14,6h cobre indisponibilidades longas; considerada aceitável pelo PM. | TRANSCRICAO | [09:17] Marcos |
| ADR-003-CONS-02 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Consequência | Derivado: nenhum evento fica preso para sempre; todo evento termina entregue ou na DLQ com motivo. | TRANSCRICAO | [09:15] Diego |
| ADR-003-CONS-03 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Consequência | Derivado: evento pode chegar até ~14,6h após a mudança de status, somando-se à limitação de ordem. | TRANSCRICAO | [09:17] Diego |
| ADR-003-CONS-04 | `docs/adrs/ADR-003-retry-backoff-exponencial-e-dlq.md` | Consequência | Reprocessamento depende de ação humana; alerta por email ao cliente fica para próxima fase. | TRANSCRICAO | [09:37] Larissa |
| ADR-004 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Decisão | Envios autenticados por HMAC-SHA256 sobre o corpo, secret por endpoint e rotação com carência de 24h. | TRANSCRICAO | [09:22] Sofia |
| ADR-004-D1 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Decisão | Cada envio assinado com HMAC-SHA256 sobre o corpo do request, assinatura no header X-Signature. | TRANSCRICAO | [09:20] Sofia |
| ADR-004-D2 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Decisão | Secret única por endpoint, armazenada com url, customer_id e estado ativo; gerada pela plataforma e devolvida na criação. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-D3 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Decisão | Secret rotacionável por API; antiga válida em paralelo por 24h; mecanismo de verificação é proposta do FDD. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-D4 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Decisão | TLS obrigatório: URLs http recusadas com erro de validação no schema Zod. | TRANSCRICAO | [09:23] Sofia |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Alternativa Descartada | Secret global da plataforma: se vazar uma, vaza tudo, e impede rotação isolada por cliente. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-02 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Alternativa Descartada | Derivado: rotação sem carência descartada implicitamente pela carência de 24h para o cliente migrar sistemas. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-01 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Consequência | Autenticidade e integridade com padrão de mercado suportado por todo cliente sério. | TRANSCRICAO | [09:20] Sofia |
| ADR-004-CONS-02 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Consequência | Derivado: vazamento de secret restrito a um endpoint, rotacionável sem janela de quebra; ancorado em secret por endpoint. | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-03 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Consequência | Derivado: HMAC exige secret em claro, não pode ser hash como passwordHash; cifrar em repouso fica para revisão de segurança. | CODIGO | `prisma/schema.prisma` |
| ADR-004-CONS-04 | `docs/adrs/ADR-004-hmac-sha256-secret-por-endpoint.md` | Consequência | Derivado: assinatura cobre só o corpo; X-Timestamp não é assinado; questão para a revisão de segurança. | TRANSCRICAO | [09:44] Diego |
| ADR-005 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Decisão | Entrega at-least-once com deduplicação do lado do cliente pelo header X-Event-Id. | TRANSCRICAO | [09:26] Larissa |
| ADR-005-D1 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Decisão | Plataforma garante at-least-once; cliente pode receber o mesmo evento mais de uma vez e deve estar preparado. | TRANSCRICAO | [09:24] Diego |
| ADR-005-D2 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Decisão | Header X-Event-Id com UUID gerado na entrada da outbox, único por evento, estável em retentativas e replay. | TRANSCRICAO | [09:25] Diego |
| ADR-005-D3 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Decisão | Semântica at-least-once documentada em destaque no portal do desenvolvedor, responsabilidade do PM. | TRANSCRICAO | [09:26] Marcos |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Alternativa Descartada | Exactly-once: exigiria coordenação dos dois lados, muito mais complexo; at-least-once é padrão de mercado. | TRANSCRICAO | [09:25] Diego |
| ADR-005-ALT-02 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Alternativa Descartada | Proposta: at-most-once sem retry, não levantada na reunião; incompatível com a política de retry e DLQ decidida. | TRANSCRICAO | [09:15] Diego |
| ADR-005-CONS-01 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Consequência | Derivado: implementação simples; worker pode reenviar sem medo após falha ambígua; ancorado na justificativa de at-least-once. | TRANSCRICAO | [09:25] Diego |
| ADR-005-CONS-02 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Consequência | Deduplicação fica do lado do cliente; cliente que não deduplique pode processar a mudança duas vezes. | TRANSCRICAO | [09:25] Sofia |
| ADR-005-CONS-03 | `docs/adrs/ADR-005-at-least-once-com-x-event-id.md` | Consequência | PM precisa comunicar bem a responsabilidade de deduplicação no portal. | TRANSCRICAO | [09:26] Marcos |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso dos padrões do projeto: módulo src/modules/webhooks, AppError, Pino, error middleware, Zod e requireRole. | TRANSCRICAO | [09:30] Larissa |
| ADR-006-D1 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reusar ao máximo AppError, Pino, error middleware, padrão de módulos, schemas Zod e códigos de erro. | TRANSCRICAO | [09:30] Larissa |
| ADR-006-D2 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Módulo src/modules/webhooks com controller, service, repository, routes, schemas; processador nomeado webhook.processor.ts (escolha do documento). | TRANSCRICAO | [09:27] Bruno |
| ADR-006-D3 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Erros novos estendem src/shared/errors com códigos prefixo WEBHOOK_, tratados pelo errorMiddleware sem mudanças. | TRANSCRICAO | [09:28] Bruno |
| ADR-006-D4 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Log usa o Pino já existente, sem biblioteca nova de log. | TRANSCRICAO | [09:29] Bruno |
| ADR-006-D5 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Endpoint de replay usa o requireRole('ADMIN') já existente. | TRANSCRICAO | [09:36] Larissa |
| ADR-006-D6 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Outbox e demais tabelas usam UUID (@default(uuid()) @db.Char(36)) como as entidades do schema. | TRANSCRICAO | [09:51] Larissa |
| ADR-006-D7 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Integração com pedidos via função publishWebhookEvent(tx, order, fromStatus, toStatus) recebendo o TransactionClient. | TRANSCRICAO | [09:41] Bruno |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Injetar WebhookRepository inteiro no construtor do OrderService: função pura recebendo tx basta. | TRANSCRICAO | [09:41] Diego |
| ADR-006-ALT-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Proposta: estrutura própria com nova lib de log ou hierarquia de erros própria; contraria o reuso, ancorada em não acrescentar nada novo. | TRANSCRICAO | [09:29] Bruno |
| ADR-006-ALT-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Compartilhar PrismaClient da API com o worker: PrismaClient é por processo, worker cria instância própria. | TRANSCRICAO | [09:30] Bruno |
| ADR-006-CONS-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Erros do módulo saem no envelope { error: { code, message, details? } } já conhecido, gerado pelo error middleware. | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR-006-CONS-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Derivado: publishWebhookEvent(tx, ...) reduz a integração com changeStatus a uma chamada dentro da transação existente. | TRANSCRICAO | [09:41] Bruno |
| ADR-006-CONS-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | ZodError vira VALIDATION_ERROR; recusa de URL http chega com esse code e WEBHOOK_INVALID_URL só em details. | CODIGO | `src/middlewares/validate.middleware.ts` |
| ADR-006-CONS-04 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Projeto sem biblioteca de métricas ou tracing; observabilidade restrita a logs estruturados e consultas às tabelas. | CODIGO | `package.json` |
| ADR-007 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Decisão | Payload gravado como snapshot na inserção e filtro de eventos por status aplicado na inserção da outbox. | TRANSCRICAO | [09:52] Bruno |
| ADR-007-D1 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Decisão | Filtro na inserção: publishWebhookEvent grava linha por endpoint ativo cujo filtro inclui to_status; nenhum casa, nada inserido. | TRANSCRICAO | [09:34] Bruno |
| ADR-007-D2 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Decisão | Payload gravado já renderizado (snapshot) na inserção, refletindo o estado do pedido quando o status mudou. | TRANSCRICAO | [09:52] Larissa |
| ADR-007-D3 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Decisão | Payload enxuto: identificação do evento, pedido, transição e campos básicos como total; sem items. | TRANSCRICAO | [09:43] Diego |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Alternativa Descartada | Filtrar na hora do envio: gravaria linhas nunca enviadas; filtrar na inserção economiza linhas. | TRANSCRICAO | [09:34] Bruno |
| ADR-007-ALT-02 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Alternativa Descartada | Gravar só order_id e renderizar no envio: pedido pode mudar e evento não refletiria a transição. | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-03 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Alternativa Descartada | Incluir items no payload: infla sem necessidade; cliente consulta GET /orders/:id quando precisar. | TRANSCRICAO | [09:43] Diego |
| ADR-007-CONS-01 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Consequência | Derivado: retentativas e replay enviam exatamente o mesmo corpo, estabilizando HMAC e deduplicação; ancorado no snapshot. | TRANSCRICAO | [09:52] Diego |
| ADR-007-CONS-02 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Consequência | Derivado: worker não precisa ler orders nem montar payload, só lê a linha e envia; ancorado no snapshot. | TRANSCRICAO | [09:52] Larissa |
| ADR-007-CONS-03 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Consequência | Derivado: endpoint criado ou filtro alterado depois da mudança não recebe eventos passados; ancorado no filtro na inserção. | TRANSCRICAO | [09:34] Bruno |
| ADR-007-CONS-04 | `docs/adrs/ADR-007-snapshot-do-payload-e-filtro-na-insercao.md` | Consequência | Derivado: snapshot fica desatualizado de propósito; quem quiser estado atual consulta a API. | TRANSCRICAO | [09:52] Larissa |

## Resumo

| Documento | Itens | TRANSCRICAO | CODIGO |
|---|---|---|---|
| `docs/PRD.md` | 69 | 68 | 1 |
| `docs/RFC.md` | 28 | 26 | 2 |
| `docs/FDD.md` | 105 | 80 | 25 |
| `docs/adrs/ADR-*.md` | 81 | 76 | 5 |
| **Total** | **283** | **250 (88%)** | **33** |

Todos os IDs citados nos documentos têm linha aqui, uma cobertura de 100%. Cada `[hh:mm] Nome` foi conferido contra a transcrição, e cada caminho de `CODIGO` existe no repositório.
