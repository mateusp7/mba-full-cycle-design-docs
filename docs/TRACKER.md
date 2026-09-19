# Tracker

Esta é a primeira versão incremental do Tracker. Ela cobre os itens relevantes
registrados nos seis ADRs e no RFC atualmente consolidados. Os itens do PRD e do
FDD serão adicionados ou revisados quando esses documentos forem estabilizados.

As linhas usam a fonte primária do item: `TRANSCRICAO` para decisões, requisitos
e restrições discutidos na reunião; `CODIGO` somente para arquivos existentes no
repositório. Artefatos propostos, mas ainda inexistentes, permanecem rastreados
pela transcrição.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| RFC-CTX-01 | `docs/RFC.md` | Contexto | A feature é exclusivamente outbound e substitui o polling contínuo dos clientes por notificações de mudança de status. | `TRANSCRICAO` | `[09:00] Marcos; [09:02] Sofia; [09:02] Marcos` |
| RFC-CTX-02 | `docs/RFC.md` | Integração | `changeStatus` é o ponto de integração que concentra a transação do pedido. | `CODIGO` | `src/modules/orders/order.service.ts:126-179` |
| RFC-CTX-03 | `docs/RFC.md` | Restrição | O banco existente usa MySQL via Prisma. | `CODIGO` | `prisma/schema.prisma:5-9` |
| RFC-SCP-01 | `docs/RFC.md` | Escopo | A proposta inclui configuração por customer, seleção de status, histórico de deliveries, retry, DLQ e replay administrativo. | `TRANSCRICAO` | `[09:31] Marcos; [09:34] Marcos; [09:35] Larissa` |
| RFC-SCP-02 | `docs/RFC.md` | Autorização | O CRUD de configuração usa autenticação normal; o replay exige `ADMIN` e registra o executor. | `TRANSCRICAO` | `[09:31] Marcos; [09:35] Larissa; [09:36] Sofia` |
| RFC-OOS-01 | `docs/RFC.md` | Fora de escopo | Email como fallback e dashboard visual não fazem parte desta fase. | `TRANSCRICAO` | `[09:37] Larissa; [09:39] Marcos; [09:40] Larissa` |
| RFC-DEC-01 | `docs/RFC.md` | Decisão | A mudança de status e o snapshot do evento devem ser registrados na mesma transação; falha no enqueue provoca rollback. | `TRANSCRICAO` | `[09:33] Marcos; [09:40] Bruno; [09:41] Diego; [09:51] Larissa; [09:52] Diego` |
| RFC-DEC-02 | `docs/RFC.md` | Decisão | Um worker separado fará polling a cada dois segundos, processando batches pequenos e priorizando eventos pendentes mais antigos. | `TRANSCRICAO` | `[09:08] Diego; [09:09] Diego; [09:10] Larissa; [09:11] Diego` |
| RFC-DEC-03 | `docs/RFC.md` | Decisão | Falhas terão cinco tentativas com backoff e depois serão persistidas em uma DLQ para replay administrativo. | `TRANSCRICAO` | `[09:15] Diego; [09:17] Diego; [09:18] Larissa` |
| RFC-SEC-01 | `docs/RFC.md` | Requisito não funcional | O corpo será assinado com HMAC-SHA256, usando secret exclusiva por endpoint e rotação com validade paralela de 24 horas. | `TRANSCRICAO` | `[09:19] Sofia; [09:20] Sofia; [09:21] Sofia; [09:22] Sofia` |
| RFC-SEC-02 | `docs/RFC.md` | Restrição | Os endpoints devem usar HTTPS e payloads acima de 64 KB devem ser rejeitados sem truncamento. | `TRANSCRICAO` | `[09:23] Sofia; [09:24] Diego; [09:24] Larissa` |
| RFC-SEC-03 | `docs/RFC.md` | Semântica de entrega | A entrega será at-least-once, com `X-Event-Id`; a deduplicação ficará com o cliente e exactly-once não será oferecido. | `TRANSCRICAO` | `[09:24] Diego; [09:25] Sofia; [09:26] Larissa` |
| RFC-SEC-04 | `docs/RFC.md` | Contrato | O payload será enxuto, sem `items`, e usará headers de identificação, assinatura, timestamp, webhook e tipo de conteúdo. | `TRANSCRICAO` | `[09:43] Diego; [09:44] Bruno; [09:44] Sofia; [09:45] Diego` |
| RFC-INT-01 | `docs/RFC.md` | Decisão de integração | A solução reutilizará módulos em camadas, `AppError`, códigos `WEBHOOK_*`, schemas Zod, middleware de erro, Pino e `requireRole('ADMIN')`. | `TRANSCRICAO` | `[09:27] Bruno; [09:28] Bruno; [09:29] Larissa; [09:35] Sofia; [09:36] Sofia` |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa descartada | HTTP síncrono dentro de `changeStatus` poderia bloquear a transação e causar rollback indevido por indisponibilidade do cliente. | `TRANSCRICAO` | `[09:04] Bruno; [09:06] Diego` |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa descartada | Redis Streams/Redis Cluster exigiria infraestrutura adicional e foi considerado overengineering para o time. | `TRANSCRICAO` | `[09:07] Larissa; [09:07] Diego` |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa descartada | Trigger/listener do banco não notificaria diretamente um processo externo; o polling de dois segundos atende à latência aceita. | `TRANSCRICAO` | `[09:09] Diego; [09:10] Marcos; [09:10] Larissa` |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa descartada | Retry indefinido poderia manter para sempre eventos de um cliente desaparecido. | `TRANSCRICAO` | `[09:15] Diego` |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa descartada | Exactly-once exigiria coordenação complexa entre plataforma e consumidor; at-least-once com identificador foi considerado suficiente. | `TRANSCRICAO` | `[09:25] Diego; [09:26] Larissa` |
| RFC-QA-01 | `docs/RFC.md` | Questão aberta | Rate limiting de saída será observado antes de uma decisão sobre política para cada customer. | `TRANSCRICAO` | `[09:38] Diego; [09:39] Larissa` |
| RFC-QA-02 | `docs/RFC.md` | Questão aberta | Retenção e arquivamento de eventos entregues, inclusive o tratamento da DLQ, ainda não têm política definida. | `TRANSCRICAO` | `[09:08] Diego` |
| RFC-QA-03 | `docs/RFC.md` | Questão aberta | Escala para múltiplos workers, claim/lock e ordering futuro foram adiados; a proposta inicial não oferece ordering global. | `TRANSCRICAO` | `[09:12] Diego; [09:13] Diego; [09:13] Larissa` |
| RFC-RISK-01 | `docs/RFC.md` | Risco | Cliente externo indisponível pode levar o evento a retries por quase 15 horas e depois à DLQ; a mitigação é backoff, limite, DLQ e replay. | `TRANSCRICAO` | `[09:15] Diego; [09:17] Diego; [09:18] Larissa` |
| RFC-RISK-02 | `docs/RFC.md` | Risco | Vazamento de secret pode falsificar chamadas do endpoint; isolamento, HMAC, HTTPS e rotação reduzem o impacto. | `TRANSCRICAO` | `[09:19] Sofia; [09:21] Sofia; [09:23] Sofia; [09:46] Sofia` |
| RFC-RISK-03 | `docs/RFC.md` | Risco | A retenção ainda indefinida pode aumentar o custo e dificultar a operação da outbox e da DLQ. | `TRANSCRICAO` | `[09:08] Diego` |
| RFC-RISK-04 | `docs/RFC.md` | Risco | Escala horizontal pode alterar a ordem dos eventos; o primeiro release assume single-worker. | `TRANSCRICAO` | `[09:12] Diego; [09:13] Diego; [09:13] Larissa` |
| RFC-PLAN-01 | `docs/RFC.md` | Restrição de planejamento | O plano registrado estima três sprints e reserva pelo menos dois dias úteis para revisão de segurança antes do deploy. | `TRANSCRICAO` | `[09:45] Marcos; [09:46] Larissa; [09:46] Sofia; [09:47] Larissa` |
| ADR-001-DEC-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | Persistir o evento na outbox do MySQL dentro da transação existente de `changeStatus`. | `TRANSCRICAO` | `[09:08] Larissa; [09:48] Larissa` |
| ADR-001-DEC-02 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | O enqueue usa a transação atual e sua falha provoca rollback da transação principal. | `TRANSCRICAO` | `[09:40] Bruno; [09:41] Diego` |
| ADR-001-DEC-03 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | A entrega HTTP ocorre depois do commit, desacoplada da transação por um worker separado. | `TRANSCRICAO` | `[09:06] Diego` |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa descartada | Chamada HTTP síncrona poderia bloquear mudanças de status e provocar rollback indevido. | `TRANSCRICAO` | `[09:04] Bruno; [09:06] Diego` |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa descartada | Redis foi descartado por exigir infraestrutura adicional e ser overengineering para o time. | `TRANSCRICAO` | `[09:07] Larissa; [09:07] Diego` |
| ADR-001-OOS-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Fora de escopo | Retenção e arquivamento de eventos entregues não são resolvidos pelo ADR. | `TRANSCRICAO` | `[09:08] Diego` |
| ADR-001-COD-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Evidência de integração | `changeStatus` usa `prisma.$transaction` para pedido, histórico e estoque. | `CODIGO` | `src/modules/orders/order.service.ts:126-179` |
| ADR-001-COD-02 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Evidência de integração | O datasource existente está configurado para MySQL. | `CODIGO` | `prisma/schema.prisma:5-9` |
| ADR-002-DEC-01 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Decisão | O worker terá no máximo cinco tentativas com backoff `1m/5m/30m/2h/12h`. | `TRANSCRICAO` | `[09:17] Diego; [09:17] Larissa` |
| ADR-002-DEC-02 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Decisão | Após o limite, a falha será persistida em uma DLQ separada com payload, motivo e timestamp para diagnóstico e replay. | `TRANSCRICAO` | `[09:17] Larissa; [09:18] Diego` |
| ADR-002-DEC-03 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Regra de resiliência | Timeout HTTP de 10 segundos conta como falha e entra no retry. | `TRANSCRICAO` | `[09:42] Sofia; [09:42] Diego` |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Alternativa descartada | Retry indefinido poderia manter o evento pendurado para sempre. | `TRANSCRICAO` | `[09:15] Diego` |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Alternativa descartada | Três tentativas cobririam uma janela curta para indisponibilidades prolongadas. | `TRANSCRICAO` | `[09:16] Bruno; [09:16] Diego` |
| ADR-002-ALT-03 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Alternativa descartada | Marcar a falha apenas na outbox foi substituído por uma tabela de DLQ separada. | `TRANSCRICAO` | `[09:17] Larissa; [09:18] Diego` |
| ADR-002-COD-01 | `docs/adrs/ADR-002-retry-com-backoff-e-dlq.md` | Evidência de integração | A persistência atual da aplicação usa MySQL. | `CODIGO` | `prisma/schema.prisma:5-9` |
| ADR-003-DEC-01 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Decisão | O corpo será assinado com HMAC-SHA256 e a assinatura será enviada em header. | `TRANSCRICAO` | `[09:19] Sofia; [09:20] Sofia` |
| ADR-003-DEC-02 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Decisão | Cada endpoint terá uma secret própria, sem secret global compartilhada. | `TRANSCRICAO` | `[09:21] Sofia` |
| ADR-003-DEC-03 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Decisão | A secret antiga continuará válida durante 24 horas após a rotação. | `TRANSCRICAO` | `[09:21] Sofia; [09:22] Sofia` |
| ADR-003-RNF-01 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Requisito não funcional | O endpoint deve ser HTTPS e payloads acima de 64 KB devem ser rejeitados sem truncamento. | `TRANSCRICAO` | `[09:23] Sofia; [09:24] Diego; [09:24] Larissa` |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Alternativa descartada | Secret global foi descartada porque um vazamento comprometeria todos os endpoints. | `TRANSCRICAO` | `[09:21] Sofia` |
| ADR-003-COD-01 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Evidência de integração | O projeto usa middleware Zod que converte `ZodError` em `ValidationError`. | `CODIGO` | `src/middlewares/validate.middleware.ts:11-36` |
| ADR-003-COD-02 | `docs/adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md` | Evidência de integração | O logger Pino existente possui redaction de credenciais. | `CODIGO` | `src/shared/logger/index.ts:4-20` |
| ADR-004-DEC-01 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Decisão | A entrega terá semântica at-least-once e poderá produzir duplicidades. | `TRANSCRICAO` | `[09:24] Diego` |
| ADR-004-DEC-02 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Decisão | `X-Event-Id` carregará um UUID único gerado quando o evento entrar na outbox. | `TRANSCRICAO` | `[09:25] Diego` |
| ADR-004-DEC-03 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Responsabilidade | O cliente consumidor será responsável por deduplicar entregas repetidas. | `TRANSCRICAO` | `[09:25] Sofia; [09:25] Diego; [09:26] Larissa` |
| ADR-004-OOS-01 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Fora de escopo | Exactly-once não será oferecido pela plataforma. | `TRANSCRICAO` | `[09:25] Sofia; [09:25] Diego; [09:26] Larissa` |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Alternativa descartada | Exactly-once exigiria coordenação mais complexa entre plataforma e consumidor. | `TRANSCRICAO` | `[09:25] Sofia; [09:25] Diego` |
| ADR-004-COD-01 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Evidência de integração | O middleware existente gera ou propaga UUID de correlação em `X-Request-Id`. | `CODIGO` | `src/middlewares/request-logger.middleware.ts:5-24` |
| ADR-004-COD-02 | `docs/adrs/ADR-004-entrega-at-least-once-com-x-event-id.md` | Evidência de integração | As entidades existentes usam IDs UUID. | `CODIGO` | `prisma/schema.prisma:25-130` |
| ADR-005-DEC-01 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Decisão | O worker fará polling em loop a cada dois segundos e buscará os pendentes mais antigos. | `TRANSCRICAO` | `[09:09] Diego; [09:10] Larissa; [09:10] Marcos` |
| ADR-005-DEC-02 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Decisão | O worker será um processo separado da API. | `TRANSCRICAO` | `[09:11] Diego; [09:11] Larissa` |
| ADR-005-DEC-03 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Decisão | API e worker usarão o mesmo banco e stack, com uma instância própria de `PrismaClient` por processo. | `TRANSCRICAO` | `[09:11] Diego; [09:29] Diego; [09:30] Bruno` |
| ADR-005-DEC-04 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Decisão | O arranjo inicial será single-worker, processando por `created_at` e mantendo ordenação por `order_id` sem prometer ordering global. | `TRANSCRICAO` | `[09:12] Diego; [09:13] Diego; [09:13] Larissa` |
| ADR-005-QA-01 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Questão aberta | Claim/lock, particionamento por `order_id` e escala para múltiplos workers ficam adiados para o FDD. | `TRANSCRICAO` | `[09:12] Diego; [09:13] Diego; [09:13] Larissa` |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Alternativa descartada | Trigger/listener do banco não notificaria um processo externo de forma adequada. | `TRANSCRICAO` | `[09:09] Diego` |
| ADR-005-ALT-02 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Alternativa descartada | Worker dentro da API faria o processamento depender do reinício e da disponibilidade da API. | `TRANSCRICAO` | `[09:11] Diego` |
| ADR-005-COD-01 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Evidência de integração | `src/server.ts` é a entry point HTTP atual. | `CODIGO` | `src/server.ts:1-27` |
| ADR-005-COD-02 | `docs/adrs/ADR-005-worker-separado-em-polling.md` | Evidência de integração | `createPrismaClient()` e o singleton Prisma são definidos no módulo de banco existente. | `CODIGO` | `src/config/database.ts:1-10` |
| ADR-006-DEC-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Decisão | O novo módulo seguirá a organização `controller`, `service`, `repository`, `routes` e `schemas`. | `TRANSCRICAO` | `[09:27] Bruno; [09:30] Larissa` |
| ADR-006-DEC-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Decisão | Erros de domínio usarão `AppError` e códigos com prefixo `WEBHOOK_`, reutilizando Pino, middleware de erro e schemas Zod. | `TRANSCRICAO` | `[09:28] Bruno; [09:29] Bruno; [09:29] Larissa; [09:30] Larissa` |
| ADR-006-DEC-03 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Decisão | O worker usará a mesma stack e banco, com instância própria de `PrismaClient`; o replay exigirá `ADMIN` via `requireRole`. | `TRANSCRICAO` | `[09:30] Bruno; [09:36] Larissa` |
| ADR-006-QA-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Questão adiada | O endurecimento das permissões do CRUD de configuração foi deixado para depois. | `TRANSCRICAO` | `[09:36] Larissa; [09:37] Sofia` |
| ADR-006-COD-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Evidência de integração | Rotas de pedidos usam autenticação, validação por schema e delegação ao controller. | `CODIGO` | `src/modules/orders/order.routes.ts:12-24` |
| ADR-006-COD-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Evidência de integração | Os schemas existentes usam Zod, UUID e `z.nativeEnum`. | `CODIGO` | `src/modules/orders/order.schemas.ts:1-34` |
| ADR-006-COD-03 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Evidência de integração | `AppError` define `statusCode`, `errorCode` e `details`. | `CODIGO` | `src/shared/errors/app-error.ts:3-15` |
| ADR-006-COD-04 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Evidência de integração | O middleware central trata `AppError`, Zod, Prisma e erros não tratados. | `CODIGO` | `src/middlewares/error.middleware.ts:14-65` |
| ADR-006-COD-05 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes.md` | Evidência de integração | O logger existente usa Pino, timestamp ISO e redaction. | `CODIGO` | `src/shared/logger/index.ts:1-32` |
