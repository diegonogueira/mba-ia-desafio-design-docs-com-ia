# ADR-004 — Autenticação dos envios por HMAC-SHA256 com secret por endpoint e rotação com 24h de carência

- **Data da decisão:** reunião técnica de quinta-feira, 09:00 (fechada às [09:22] por Sofia)
- **Decisores:** Sofia (Segurança), com Bruno, Diego e Larissa
- **Relacionados:** [ADR-005](ADR-005-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito, com revisão de segurança obrigatória da Sofia antes do deploy ([09:46] Sofia)

## Contexto

Os eventos levam dados de pedidos para um endpoint fora da nossa infraestrutura. O cliente precisa conseguir verificar que a requisição veio de nós e que o payload não foi adulterado no caminho ([09:19] Sofia). Já houve cliente que vazou uma secret em log de aplicação ([09:22] Diego).

## Decisão

- `ADR-004-D1`: cada envio é assinado com **HMAC-SHA256 sobre o corpo do request**. A assinatura vai no header `X-Signature` ([09:20] Sofia, [09:22] Sofia).
- `ADR-004-D2`: **a secret é única por endpoint de webhook**, nunca global da plataforma ([09:21] Sofia). Ela é armazenada na configuração do webhook junto com URL, `customer_id` e estado ativo ([09:21] Bruno, [09:21] Sofia). É **gerada pela plataforma e devolvida ao cliente na criação** ([09:31] Marcos).
- `ADR-004-D3`: a secret é **rotacionável por API**. Na rotação, a secret antiga continua válida em paralelo por **24 horas** e depois deixa de valer ([09:21] Sofia). O mecanismo que mantém a antiga verificável nesse período é uma proposta de implementação, a validar na revisão de segurança. Ele está no [FDD §6.9](../FDD.md#69-fdd-contrato-09--assinatura).
- `ADR-004-D4`: **TLS é obrigatório**. URLs `http://` são recusadas com erro de validação no schema Zod ([09:23] Sofia). Isso é validação, não uma decisão arquitetural separada ([09:23] Sofia); está registrado aqui porque protege o mesmo canal.

## Alternativas Consideradas

| ID | Alternativa | Por que foi descartada |
|---|---|---|
| `ADR-004-ALT-01` | **Uma secret global da plataforma** para todos os endpoints | Se vazar uma, vaza tudo ([09:21] Sofia). Também impede rotacionar um cliente sem afetar os outros. |
| `ADR-004-ALT-02` | **Rotação sem carência**, com a secret antiga invalidada na hora | Descartada implicitamente pela carência de 24 h. O cliente precisa de tempo para migrar os sistemas dele ([09:21] Sofia). |

## Consequências

**Positivas**

- `ADR-004-CONS-01`: autenticidade e integridade com um padrão que todo cliente sério já suporta ([09:20] Sofia).
- `ADR-004-CONS-02`: o vazamento de uma secret fica restrito a um endpoint, e esse endpoint pode ser rotacionado sem janela de quebra.

**Negativas**

- `ADR-004-CONS-03`: o HMAC precisa da secret em claro no momento de assinar, então ela não pode ser guardada como hash de mão única, como `passwordHash` em `prisma/schema.prisma`. Cifrar em repouso não foi discutido e fica para a revisão de segurança ([09:46] Sofia).
- `ADR-004-CONS-04`: a assinatura cobre só o corpo ([09:22] Sofia). O `X-Timestamp` permite ao cliente detectar replay "se quiser" ([09:44] Diego), mas não é assinado. Fica como questão para a revisão de segurança.
- A secret precisa entrar nos `redactPaths` do logger Pino (`src/shared/logger/index.ts`) para não repetir o incidente de vazamento em log ([09:22] Diego).

**Trade-off explícito:** secrets por endpoint com rotação custam mais estado (secret atual, secret anterior e prazo de expiração) e uma assinatura dupla temporária. Em troca, o vazamento de uma secret fica contido e a troca de credencial é segura para o cliente.
