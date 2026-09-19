# Tracker

Esta é uma versão incremental do Tracker. Ela cobre os itens relevantes
registrados nos seis ADRs, no RFC e no FDD atualmente consolidados. O PRD ainda
é um placeholder e não acrescenta itens documentais rastreáveis nesta etapa.

As linhas usam a fonte primária do item: `TRANSCRICAO` para decisões, requisitos
e restrições discutidos na reunião; `CODIGO` somente para arquivos existentes no
repositório. Artefatos propostos, mas ainda inexistentes, permanecem rastreados
pela transcrição.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| FDD-CTX-01 | `docs/FDD.md` | Contexto | A feature é outbound e integra mudança de status do OMS a clientes externos. | `TRANSCRICAO` | [09:00]–[09:03] Marcos e Sofia |
| FDD-OBJ-01 | `docs/FDD.md` | Objetivo técnico | Latência inferior a 10 segundos é suficiente e o polling do worker ocorre a cada 2 segundos. | `TRANSCRICAO` | [09:02] Marcos; [09:09]–[09:10] Diego, Marcos e Larissa |
| FDD-DEC-01 | `docs/FDD.md` | Decisão | Mudança de status e snapshot da outbox compartilham a transação; falha no enqueue provoca rollback. | `TRANSCRICAO` | [09:40]–[09:41] Bruno e Diego |
| FDD-DEC-02 | `docs/FDD.md` | Decisão | Snapshot é renderizado na inserção e não no envio posterior. | `TRANSCRICAO` | [09:51]–[09:52] Larissa, Diego e Bruno |
| FDD-FLOW-01 | `docs/FDD.md` | Fluxo | Configurações são filtradas por status na inserção da outbox. | `TRANSCRICAO` | [09:33]–[09:34] Marcos, Bruno e Diego |
| FDD-FLOW-02 | `docs/FDD.md` | Fluxo | Worker separado faz polling dos pendentes mais antigos em batches pequenos. | `TRANSCRICAO` | [09:08]–[09:11] Diego e Larissa |
| FDD-FLOW-03 | `docs/FDD.md` | Fluxo | Worker inicia single-worker com ordering por created_at/order_id, sem ordering global. | `TRANSCRICAO` | [09:12]–[09:13] Diego e Larissa |
| FDD-FLOW-04 | `docs/FDD.md` | Resiliência | Falhas seguem timeout, cinco tentativas, backoff registrado e DLQ separada. | `TRANSCRICAO` | [09:15]–[09:18] Diego, Bruno e Larissa |
| FDD-FLOW-05 | `docs/FDD.md` | Fluxo | Replay de DLQ recoloca o evento na outbox e exige ADMIN com auditoria do executor. | `TRANSCRICAO` | [09:18] Diego; [09:35]–[09:36] Sofia e Larissa |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | CRUD de configuração autenticado cobre criação, edição, remoção e listagem por customer. | `TRANSCRICAO` | [09:31]–[09:33] Marcos, Bruno e Larissa |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | Histórico expõe os últimos 100 deliveries com payload, resposta e tempo. | `TRANSCRICAO` | [09:34] Marcos |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | Replay usa POST na rota administrativa de dead-letter. | `TRANSCRICAO` | [09:34]–[09:36] Marcos, Larissa e Diego |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato de segurança | Outbound usa payload enxuto, HMAC, X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id e Content-Type JSON. | `TRANSCRICAO` | [09:19]–[09:24] Sofia, Diego e Larissa; [09:43]–[09:45] Diego, Bruno e Sofia |
| FDD-ERRO-01 | `docs/FDD.md` | Erro | Erros de domínio do módulo usam AppError e prefixo WEBHOOK_. | `TRANSCRICAO` | [09:28]–[09:30] Bruno e Larissa |
| FDD-ERRO-02 | `docs/FDD.md` | Resiliência | Falha externa não desfaz status commitado; falha ao inserir outbox desfaz o conjunto transacional. | `TRANSCRICAO` | [09:04]–[09:06] Bruno, Larissa e Diego; [09:40]–[09:41] Bruno e Diego |
| FDD-SEC-01 | `docs/FDD.md` | Segurança | HMAC-SHA256 usa secret por endpoint e a antiga permanece válida por 24 horas após rotação. | `TRANSCRICAO` | [09:19]–[09:22] Sofia |
| FDD-SEC-02 | `docs/FDD.md` | Restrição | URL deve ser HTTPS e payload acima de 64 KB deve ser rejeitado sem truncamento. | `TRANSCRICAO` | [09:23]–[09:24] Sofia, Diego e Larissa |
| FDD-DEL-01 | `docs/FDD.md` | Semântica de entrega | Entrega at-least-once permite duplicidade; consumidor deduplica por X-Event-Id. | `TRANSCRICAO` | [09:24]–[09:26] Diego, Sofia e Larissa |
| FDD-OBS-01 | `docs/FDD.md` | Observabilidade | Feature reutiliza o logger Pino estruturado e redaction do sistema existente. | `CODIGO` | src/shared/logger/index.ts:1-32 |
| FDD-INT-01 | `docs/FDD.md` | Integração | changeStatus usa transação Prisma e é o ponto de extensão do enqueue. | `CODIGO` | src/modules/orders/order.service.ts:126-179 |
| FDD-INT-02 | `docs/FDD.md` | Integração | Status e transições válidos estão centralizados no módulo de pedidos. | `CODIGO` | src/modules/orders/order.status.ts:1-37 |
| FDD-INT-03 | `docs/FDD.md` | Integração | Rotas existentes usam autenticação, validação e controllers. | `CODIGO` | src/modules/orders/order.routes.ts:12-24 |
| FDD-INT-04 | `docs/FDD.md` | Integração | AppError, middleware de erro, logger e request ID formam a infraestrutura reutilizável. | `CODIGO` | src/shared/errors/app-error.ts:3-15; src/middlewares/error.middleware.ts:14-65; src/shared/logger/index.ts:1-32; src/middlewares/request-logger.middleware.ts:5-27 |
| FDD-INT-05 | `docs/FDD.md` | Integração | API registra controllers/routers no composition root e usa o prefixo /api/v1. | `CODIGO` | src/app.ts:22-73; src/routes/index.ts:13-30 |
| FDD-INT-06 | `docs/FDD.md` | Integração | Worker e API devem usar instâncias Prisma próprias no mesmo banco. | `CODIGO` | src/config/database.ts:1-10; src/server.ts:1-27 |
| FDD-CA-01 | `docs/FDD.md` | Critério de aceite | Commit atômico de pedido, histórico, estoque e snapshot. | `TRANSCRICAO` | [09:40]–[09:41] Bruno e Diego |
| FDD-CA-02 | `docs/FDD.md` | Critério de aceite | Filtro por status, snapshot, timeout, retry, DLQ e replay ADMIN devem ser testáveis. | `TRANSCRICAO` | [09:33]–[09:36]; [09:42]; [09:51]–[09:52] |
| FDD-CA-03 | `docs/FDD.md` | Critério de aceite | Segurança, headers, at-least-once e ausência de items devem estar no contrato outbound. | `TRANSCRICAO` | [09:19]–[09:26]; [09:43]–[09:45] |
| FDD-OOS-01 | `docs/FDD.md` | Fora de escopo | Inbound, e-mail de fallback e dashboard não fazem parte desta fase. | `TRANSCRICAO` | [09:02]–[09:03]; [09:37]–[09:40] Sofia, Marcos e Larissa |
| FDD-OOS-02 | `docs/FDD.md` | Fora de escopo | Arquivamento, ordering global, multi-worker, exactly-once e rate limiting ficam fora ou adiados. | `TRANSCRICAO` | [09:08] Diego; [09:12]–[09:13] Diego e Larissa; [09:24]–[09:26] Diego, Sofia e Larissa; [09:38]–[09:39] Diego e Larissa |
| FDD-QA-01 | `docs/FDD.md` | Questão aberta | Rotas finais, envelopes, batch, claim/lock, retenção, granularidade do delivery e replay ainda precisam de confirmação. | `TRANSCRICAO` | [09:12]–[09:13] Diego e Larissa; [09:31]–[09:36] Marcos, Bruno, Diego, Sofia e Larissa |
| FDD-QA-02 | `docs/FDD.md` | Questão aberta | A indexação entre cinco tentativas e os cinco intervalos de backoff não foi fechada. | `TRANSCRICAO` | [09:15]–[09:17] Diego e Larissa |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato de segurança | Rotação de secret mantém a antiga válida por 24 horas. | `TRANSCRICAO` | [09:21]–[09:22] Sofia |
| FDD-ERRO-03 | `docs/FDD.md` | Matriz de erros | Validação de URL/HTTPS, payload de 64 KB, secret indisponível e falha de enqueue usam códigos WEBHOOK_* e tratamentos distintos. | `TRANSCRICAO` | [09:23]–[09:24] Sofia, Diego e Larissa; [09:28]–[09:29] Bruno e Larissa; [09:40]–[09:41] Bruno e Diego |
| FDD-ERRO-04 | `docs/FDD.md` | Matriz de erros | Timeout, erro de rede, exaustão de retry, DLQ inexistente e replay conflitante têm tratamento próprio. | `TRANSCRICAO` | [09:15]–[09:18] Diego, Bruno e Larissa; [09:42] Sofia e Diego |
| FDD-OBS-02 | `docs/FDD.md` | Observabilidade | Métricas propostas cobrem outbox, idade do pendente, tentativas, duração, DLQ e rejeições. | `CODIGO` | src/shared/logger/index.ts:13-29 |
| FDD-OBS-03 | `docs/FDD.md` | Observabilidade | Logs e traces correlacionam requestId/eventId e não registram secrets ou payload completo. | `CODIGO` | src/shared/logger/index.ts:4-21; src/middlewares/request-logger.middleware.ts:5-24 |
| FDD-CA-04 | `docs/FDD.md` | Critério de aceite | Snapshot permanece igual quando o pedido muda após o enqueue. | `TRANSCRICAO` | [09:51]–[09:52] Larissa, Diego e Bruno |
| FDD-CA-05 | `docs/FDD.md` | Critério de aceite | Worker consulta a cada 2 segundos, em batch pequeno e por created_at. | `TRANSCRICAO` | [09:08]–[09:10] Diego e Larissa |
| FDD-CA-06 | `docs/FDD.md` | Critério de aceite | Single-worker preserva a ordenação aceita por pedido, sem ordering global. | `TRANSCRICAO` | [09:12]–[09:13] Diego e Larissa |
| FDD-CA-07 | `docs/FDD.md` | Critério de aceite | Timeout de 10 segundos é registrado e entra no retry. | `TRANSCRICAO` | [09:42] Sofia e Diego |
| FDD-CA-08 | `docs/FDD.md` | Critério de aceite | Retry respeita limite de cinco e intervalos registrados, com indexação final pendente. | `TRANSCRICAO` | [09:15]–[09:17] Diego e Larissa |
| FDD-CA-09 | `docs/FDD.md` | Critério de aceite | Falha permanente preserva payload, motivo, tentativas e timestamp na DLQ. | `TRANSCRICAO` | [09:17]–[09:18] Larissa e Diego |
| FDD-CA-10 | `docs/FDD.md` | Critério de aceite | Usuário sem ADMIN não pode fazer replay de DLQ. | `TRANSCRICAO` | [09:35]–[09:36] Sofia e Larissa |
| FDD-CA-11 | `docs/FDD.md` | Critério de aceite | ADMIN consegue reencaminhar e o executor é auditável. | `TRANSCRICAO` | [09:18] Diego; [09:35]–[09:36] Sofia e Larissa |
| FDD-CA-12 | `docs/FDD.md` | Critério de aceite | Assinatura HMAC usa a secret própria do endpoint. | `TRANSCRICAO` | [09:19]–[09:21] Sofia |
| FDD-CA-13 | `docs/FDD.md` | Critério de aceite | Secret antiga funciona por 24 horas após rotação. | `TRANSCRICAO` | [09:21]–[09:22] Sofia |
| FDD-CA-14 | `docs/FDD.md` | Critério de aceite | URL HTTP é rejeitada e payload acima de 64 KB não é truncado. | `TRANSCRICAO` | [09:23]–[09:24] Sofia, Diego e Larissa |
| FDD-CA-15 | `docs/FDD.md` | Critério de aceite | Headers outbound exigidos existem e items não aparece no payload. | `TRANSCRICAO` | [09:43]–[09:45] Diego, Bruno e Sofia |
| FDD-CA-16 | `docs/FDD.md` | Critério de aceite | X-Event-Id permite deduplicação e exactly-once não é oferecido. | `TRANSCRICAO` | [09:24]–[09:26] Diego, Sofia e Larissa |
| FDD-CA-17 | `docs/FDD.md` | Critério de aceite | CRUD, histórico, rotação e replay têm contratos HTTP sob /api/v1. | `TRANSCRICAO` | [09:31]–[09:36] Marcos, Bruno, Diego, Sofia e Larissa |
| FDD-CA-18 | `docs/FDD.md` | Critério de aceite | Telemetria cobre métricas, logs com redaction e tracing correlacionado. | `CODIGO` | src/shared/logger/index.ts:4-29; src/middlewares/request-logger.middleware.ts:5-24 |
| FDD-CA-19 | `docs/FDD.md` | Critério de aceite | Composition root, AppError e changeStatus mantêm os padrões existentes. | `CODIGO` | src/app.ts:22-73; src/shared/errors/app-error.ts:3-15; src/modules/orders/order.service.ts:126-179 |
| FDD-RISK-01 | `docs/FDD.md` | Risco | Indisponibilidade do cliente pode acumular retry e DLQ. | `TRANSCRICAO` | [09:15]–[09:18] Diego, Bruno e Larissa; [09:42] Sofia e Diego |
| FDD-RISK-02 | `docs/FDD.md` | Risco | Vazamento de secret ou assinatura em log pode permitir falsificação. | `TRANSCRICAO` | [09:19]–[09:24]; [09:45]–[09:49] Sofia e Larissa |
| FDD-RISK-03 | `docs/FDD.md` | Risco | Retenção indefinida pode fazer outbox/DLQ crescer sem limite. | `TRANSCRICAO` | [09:08] Diego |
| FDD-RISK-04 | `docs/FDD.md` | Risco | At-least-once permite duplicidade de entrega. | `TRANSCRICAO` | [09:24]–[09:26] Diego, Sofia e Larissa |
| FDD-RISK-05 | `docs/FDD.md` | Risco | Escala horizontal e claim/lock podem alterar ordering ou deixar item preso. | `TRANSCRICAO` | [09:12]–[09:13] Diego e Larissa |
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
