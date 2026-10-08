# Da Reunião ao Documento: Design Docs do Sistema de Webhooks de Pedidos

Pacote de design docs (PRD, RFC, FDD, 7 ADRs e Tracker) produzido com IA a partir da transcrição de uma reunião técnica e do código de um Order Management System em Node.js + TypeScript + Prisma/MySQL.

> O enunciado original do desafio está no [repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia). Este README descreve **como** o pacote foi produzido. O código da aplicação (`src/`, `prisma/`, `tests/` e configurações) não foi alterado: `git diff upstream/main --stat` mostra só `README.md` e `docs/`.

## Sobre o desafio

Uma empresa com um OMS em produção decidiu, numa call de ~55 minutos entre Tech Lead, PM, dois engenheiros e Segurança, construir **webhooks de saída** para avisar clientes B2B quando o status de um pedido muda. Nada foi registrado além da transcrição ([`TRANSCRICAO.md`](TRANSCRICAO.md)). A tarefa foi transformar essa conversa, cruzada com o código existente, num pacote de documentação acionável. Cada documento opera numa altura: o PRD em produto, o RFC em arquitetura, os ADRs em decisões pontuais, o FDD em implementação, e o Tracker é transversal.

A restrição central é a **rastreabilidade**. Toda afirmação precisa apontar para uma fala `[hh:mm] Nome` ou para um arquivo real do repositório. Por isso o trabalho foi menos "gerar texto" e mais **separar o que foi decidido do que foi descartado, adiado, dito como exemplo ou deixado ambíguo**. O resto foi verificar, de forma adversarial e por script, que nada inventado sobreviveu.

## Ferramentas de IA utilizadas

| Ferramenta | Papel |
|---|---|
| **Claude Code** (app desktop, modelo Claude Opus 5.5) | Agente principal. Leu o código e a transcrição, planejou o pacote, redigiu ADRs, RFC, FDD e PRD e aplicou as correções. |
| **Subagentes do Claude Code** (`general-purpose`), lançados em paralelo | Papéis isolados e com contexto limpo: (1) extração dirigida da transcrição; (2) dois revisores adversariais na 1ª rodada, um caçando alucinações e outro checando fronteiras e o checklist; (3) três redatores do tracker em paralelo (PRD+RFC, FDD, ADRs); (4) dois revisores na 2ª rodada, um para a semântica do tracker e outro para regressões e lacunas. Os revisores eram **somente leitura** e respondiam com achados, não com edições. |
| **Scripts Python gerados pela IA** (fora do repositório) | Verificação mecânica: todo `[hh:mm] Nome` citado existe na transcrição; todo caminho de código existe, exceto os marcados como novos; todo ID dos documentos tem linha no tracker; percentual de fontes; seções obrigatórias dos ADRs; links e âncoras do Markdown; nenhum arquivo fora de `docs/` e `README.md` alterado. |
| **GitHub CLI** (`gh`) | Fork, leitura do repositório base antes do clone e PR. |

## Workflow adotado

1. **Leitura e plano.** Antes do fork, a IA leu a transcrição inteira e cerca de 25 arquivos do código direto do GitHub. Montei com ela um plano com os ganchos de código (`changeStatus` em `src/modules/orders/order.service.ts`, `AppError`, `validate()`, `requireRole`, logger Pino, `createPrismaClient`, `paginated`...) e uma lista inicial de ambiguidades da reunião.
2. **Extração dirigida.** Um subagente gerou um inventário em 8 listas, cada item com timestamp: decidido, requisito funcional, não funcional, descartado, adiado, aberto ou ambíguo, correções feitas na própria reunião e "armadilhas". Cruzei esse inventário com o plano.
3. **Geração na ordem sugerida.** ADRs primeiro, depois RFC, FDD e PRD. Cada item recebeu um **ID inline** (`PRD-FR-03`, `RFC-OQ-06`, `FDD-CONTRATO-05`, `ADR-004-D3`...). Assim o tracker vira uma junção verificável, e não uma lista solta. Fiz um commit por documento.
4. **1ª revisão adversarial.** Dois revisores independentes rodaram em paralelo, sem ver o raciocínio de quem escreveu. Triei os achados e corrigi.
5. **Tracker.** Três subagentes, com o mesmo prompt de regras rígidas, produziram as 283 linhas. O script validou 100% das âncoras.
6. **2ª revisão adversarial.** Um revisor fez a semântica linha a linha do tracker. Outro procurou regressões introduzidas pelas correções e lacunas do FDD como especificação. Triei e corrigi de novo.
7. **README por último**, escrito a partir do log de processo mantido durante o trabalho. Depois veio a checagem final do checklist do enunciado, item a item.

Organização da interação com a IA: um agente principal manteve o contexto do pacote inteiro. Cada tarefa que se beneficiava de **contexto isolado** foi para um subagente com prompt próprio: extrair sem viés, revisar sem conhecer as intenções do autor, paralelizar o tracker. As decisões de triagem (aceitar, rejeitar ou reinterpretar um achado) ficaram com o agente principal sob minha direção.

## Prompts customizados

**1. Extração dirigida da transcrição.** O foco é separar o que entra do que não entra.

```text
Você é um analista de requisitos. Leia INTEGRALMENTE TRANSCRICAO.md (reunião sobre um sistema de
webhooks de notificação de pedidos). Também leia src/modules/orders/order.service.ts e
src/modules/orders/order.status.ts para checar ganchos citados.

Produza um inventário estruturado, SEM inventar nada. Cada item DEVE ter a localização exata no
formato `[hh:mm] Nome` (o timestamp e o nome do falante exatamente como aparecem). Se um item se
apoia em mais de uma fala, liste todas.

Classifique em 6 listas (Item | Localização | Citação curta ≤15 palavras):
1. DECIDIDO — decisões arquiteturais fechadas (quem fechou e quando).
2. REQUISITO FUNCIONAL. 3. REQUISITO NÃO FUNCIONAL / RESTRIÇÃO.
4. DESCARTADO — alternativas rejeitadas, com o motivo/trade-off dito na reunião.
5. ADIADO / FORA DE ESCOPO.
6. EM ABERTO / AMBÍGUO — pontos não decididos, falas que se contradizem, números que não fecham.
Também liste à parte:
7. CORREÇÕES NA PRÓPRIA REUNIÃO — falas que corrigem uma fala anterior (crítico para não registrar
   a versão errada).
8. ARMADILHAS — coisas que um gerador de documentos poderia erroneamente transformar em requisito
   (ideias levantadas como pergunta, exemplos ilustrativos, "etc").
Percorra linha a linha do [09:00] ao [09:53], inclusive a conversa depois que Marcos e Sofia saem.
```

**2. Revisor adversarial de alucinações**, rodado depois da primeira versão completa.

```text
Você é um revisor adversarial de documentação técnica, e seu único trabalho é ACHAR PROBLEMAS — não
elogiar. (Somente leitura: NÃO edite nenhum arquivo.)
Documentos: docs/PRD.md, docs/RFC.md, docs/FDD.md, docs/adrs/ADR-00*.md. Fontes de verdade:
TRANSCRICAO.md e o código em src/, prisma/, tests/, package.json.
Regra do desafio: toda informação deve ser rastreável; itens descartados/adiados NÃO podem aparecer
como requisito; nenhum arquivo citado pode ser inexistente (exceto os marcados "(novo)").
1. Para CADA citação "[hh:mm] Nome": a fala existe e o conteúdo atribuído corresponde ao que a
   pessoa disse NAQUELA fala?
2. Toda afirmação sem origem e NÃO marcada como proposta/derivação do documento.
3. Toda afirmação sobre o código (funções, assinaturas, códigos de erro, versões, scripts,
   comportamento de middlewares): verifique lendo o arquivo.
4. Caminhos inexistentes. 5. Contradições com a transcrição. 6. Contradições ENTRE os documentos.
Formato: arquivo:linha, trecho, problema, evidência, severidade, correção. Não invente achados.
```

**3. Regras do tracker.** É o mesmo prompt para os três redatores paralelos (trecho).

```text
Para CADA ID dos documentos que lhe cabem, produza EXATAMENTE UMA linha:
| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
- Fonte: SOMENTE TRANSCRICAO ou CODIGO.
- Localização: se TRANSCRICAO, EXATAMENTE `[hh:mm] Nome` (um único timestamp e falante), e essa fala
  DEVE existir e SUSTENTAR o item. Se CODIGO, UM único caminho existente. Arquivos "(novo)" não
  existem — ancore no arquivo existente relacionado.
- Se o item é proposta/derivação do documento (não dita literalmente na reunião), o resumo começa
  com "Proposta: " ou "Derivado: " e diz em que se ancora.
- Se um item não tem nenhuma origem possível, produza a linha com "SEM ORIGEM: " — será revisado.
```

## Iterações e ajustes

Foram **5 iterações principais** depois do plano: extração, geração, 1ª revisão, tracker e 2ª revisão. Juntas somaram cerca de 90 achados triados. Os ajustes mais relevantes estão abaixo.

**Ambiguidades da reunião tratadas de forma explícita**, em vez de "resolvidas" pela IA em silêncio:
- **"5 tentativas" contra 5 intervalos.** Os intervalos 1m/5m/30m/2h/12h somam 14 h 36 min, e isso só bate com "quase 15 horas" ([09:17] Diego) se forem 1 envio + 5 retentativas. Mas o resumo da Tech Lead diz "total 5 tentativas" ([09:48] Larissa). A primeira versão chamava a nossa leitura de "a única possível". O revisor apontou a fala contrária, e o ponto virou a questão em aberto `RFC-OQ-06`, citando as duas evidências.
- **`customer_id` "no body ou no path"** ([09:32] Larissa). A primeira versão usava query string na listagem, que não é nem body nem path. Passou a ser path: `/api/v1/customers/:customerId/webhooks`.
- **Ordem por pedido durante o retry.** A reunião garantiu ordem por `order_id` com worker único. A IA tinha atribuído a Larissa a *quebra* dessa ordem durante o backoff, mas isso é uma dedução nossa. Ficou marcado como "Derivado" no texto e no tracker (`RFC-OQ-07`).

**Invenções da IA que removi ou marquei:**
- No PRD: um "piloto com a Atlas" e um "p95" na meta de latência, ambos sem fonte. A frase "a Atlas quer SHIPPED/DELIVERED" era, na verdade, o exemplo genérico do PM. "Pelo menos as 100 últimas" virou o exemplo que era. A meta "3 de 3 clientes" ficou marcada como meta proposta.
- No ADR-006, uma afirmação de que "testes instanciam `OrderService`". Um `grep` mostrou que só `src/app.ts` instancia.
- A assinatura dupla durante a rotação de secret aparecia como decisão "Aceita" num ADR. Saiu do ADR e ficou no FDD como proposta a validar na revisão de segurança.

**Erros técnicos pegos ao confrontar com o código:**
- Os logs do FDD usavam `logger.info('msg', {obj})`. No Pino isso descarta o objeto, e o log de auditoria do replay perderia o `userId` exigido pela Sofia. Passaram para `(obj, msg)`, como em `src/server.ts`.
- Apagar um endpoint removia em cascata a DLQ e o `replayedById`, ou seja, a evidência e a auditoria pedidas na reunião. A DLQ ficou sem FK.
- Uma FK obrigatória `WebhookEndpoint → Customer` faria `DELETE /api/v1/customers/:id` passar a responder 500 (P2003 não é tratado em `error.middleware.ts`). Isso mudaria o contrato de uma rota existente. Passou para `onDelete: Cascade`.
- `NotFoundError` fixa o código `NOT_FOUND`, então os 404 `WEBHOOK_*` estendem `AppError` direto. O ADR e o FDD estavam inconsistentes nisso.
- O worker sequencial, com lote de 10 e timeout de 10 s, pode atrasar clientes saudáveis em até ~100 s (head-of-line blocking). A primeira versão dizia que um cliente lento "nunca afeta" os outros. Isso virou o risco `FDD-RISK-09`.
- O PATCH dizia que tudo "vale para eventos futuros". Mas `url` e `active` são lidos pelo worker no envio. Era a única ambiguidade de severidade alta da 2ª rodada.
- Dois replays simultâneos passariam ambos, porque um SELECT não trava a linha sob REPEATABLE READ. A solução foi `updateMany` condicional, e o mesmo vale para a rotação.

**Fronteira entre documentos:** os riscos de negócio do PRD estavam copiados no RFC, e hoje o RFC só traz riscos de arquitetura. O payload aparecia em três documentos e agora vive só no FDD §6.8. O "Status" dos ADRs era um bullet de metadado e virou uma seção `## Status`.

**Tracker:** a 2ª revisão achou 27 linhas a ajustar. Eram âncoras fracas (por exemplo, `[09:06] Diego` quando o argumento era de `[09:04] Bruno`), 20 propostas sem o prefixo "Proposta:" ou "Derivado:" e um tipo errado.

## Como navegar a entrega

| Ordem | Documento | Para quê |
|---|---|---|
| 1 | [`docs/PRD.md`](docs/PRD.md) | Por que e o quê: problema, público, métricas, escopo e **fora de escopo**, 13 requisitos funcionais, riscos e critérios de aceitação |
| 2 | [`docs/RFC.md`](docs/RFC.md) | Proposta de arquitetura em ~3 páginas: alternativas descartadas na reunião e **questões em aberto** |
| 3 | [`docs/adrs/`](docs/adrs/README.md) | 7 decisões no formato MADR: outbox, worker com polling, retry e DLQ, HMAC, at-least-once, reuso de padrões, snapshot e filtro |
| 4 | [`docs/FDD.md`](docs/FDD.md) | Como construir: modelo de dados, fluxos, 7 endpoints mais o contrato de saída, matriz `WEBHOOK_*`, resiliência, observabilidade e **integração com o sistema existente** (18 pontos de integração em arquivos reais) |
| 5 | [`docs/TRACKER.md`](docs/TRACKER.md) | De onde veio cada item: 283 linhas, 100% dos IDs, 88% com fonte TRANSCRICAO e 33 com fonte CODIGO |

Dica de leitura: todo item tem um ID entre crases. Buscar o ID (por exemplo, `FDD-CONTRATO-07`) no `TRACKER.md` mostra a fala ou o arquivo de origem. Buscar a mesma fala em [`TRANSCRICAO.md`](TRANSCRICAO.md) mostra o contexto.
