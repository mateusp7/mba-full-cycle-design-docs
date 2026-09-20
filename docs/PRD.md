# PRD — Sistema de Webhooks de Notificação de Pedidos

## 1. Resumo e contexto da feature

O Sistema de Webhooks de Notificação de Pedidos permitirá que clientes B2B
recebam notificações quando o status de um pedido mudar no OMS. A necessidade
foi apresentada por Atlas Comercial, MaxDistribuição e Nova Cargo, que hoje
consultam periodicamente `GET /orders` para descobrir mudanças ([09:00] Marcos).
Para esses clientes, uma latência inferior a 10 segundos é percebida como
tempo real ([09:01]–[09:02] Marcos).

A primeira versão será exclusivamente outbound: a plataforma envia eventos aos
clientes; os clientes não enviam webhooks para a plataforma ([09:02] Sofia;
[09:02] Marcos). O produto abrangerá configuração de endpoints, seleção de
status, entrega assíncrona, histórico de deliveries, tratamento de falhas e
replay administrativo. A implementação técnica detalhada permanece no
[FDD](FDD.md); este documento define o problema, o que será entregue e como o
resultado será aceito.

O sistema existente concentra a mudança de status, atualização de estoque e
registro do histórico em uma transação no método `changeStatus`
(`src/modules/orders/order.service.ts:126-179`). A feature deverá preservar
essa consistência sem fazer chamadas HTTP síncronas dentro da transação
([09:04] Bruno; [09:06] Diego).

## 2. Problema e motivação

Os clientes B2B precisam reagir às mudanças dos próprios pedidos, mas hoje
fazem polling de `GET /orders`. Esse modelo torna a integração lenta e cara e
exige atualização periódica para descobrir se algo mudou ([09:00] Marcos). A
motivação do produto é substituir essa descoberta ativa por notificações
outbound, mantendo a latência aceita pelos clientes abaixo de 10 segundos
([09:02] Marcos).

Há também um problema de confiabilidade: a transação de status já atualiza o
pedido, o histórico e o estoque; uma chamada HTTP síncrona poderia ser
bloqueada por um cliente lento e levantaria a questão indevida de desfazer o
status quando o cliente estivesse indisponível ([09:04] Bruno; [09:05] Larissa).
Por isso, o escopo exige registrar o evento junto com a mudança confirmada e
entregá-lo fora da transação, com tentativas posteriores para falhas
([09:06] Diego; [09:15]–[09:18] Diego e Larissa).

O resultado esperado é que o cliente consiga configurar o que deseja receber,
identificar e validar cada evento, consultar o histórico e solicitar replay de
falhas permanentes, sem transformar a disponibilidade do endpoint externo em
dependência síncrona do processamento do pedido.

## 3. Público-alvo e cenários de uso

### Clientes B2B consumidores

Atlas Comercial, MaxDistribuição e Nova Cargo são os clientes explicitamente
citados como demandantes ([09:00] Marcos). Eles precisam:

- cadastrar um endpoint e selecionar os status de interesse;
- receber eventos de mudança dos pedidos do próprio customer;
- validar autenticidade e deduplicar eventos repetidos;
- consultar deliveries recentes quando investigarem uma falha.

### Usuários autenticados que administram configurações

Usuários do sistema autenticados por JWT poderão operar o CRUD de configuração
na primeira fase; o `customer_id` será informado no body ou no path e não será
derivado do JWT do usuário operador ([09:31]–[09:33] Marcos, Bruno e Larissa;
[09:35]–[09:37] Sofia e Marcos).
O sistema já possui as roles `ADMIN` e `OPERATOR` em
`src/middlewares/auth.middleware.ts:6-10`.

### Operadores administrativos

O replay de uma falha permanente será uma ação administrativa. Apenas usuários
com role `ADMIN` poderão executá-lo, e o executor deverá ser registrado para
auditoria ([09:35]–[09:36] Sofia e Larissa).

### OMS e sistema de pedidos

O OMS deverá transformar uma mudança de status em evento somente para os
endpoints elegíveis do customer, persistindo o snapshot antes do commit da
transação ([09:33]–[09:34] Marcos, Bruno e Diego; [09:40]–[09:41] Bruno e
Diego). O cliente externo consumirá o evento por HTTP; não há cenário inbound
nesta fase ([09:02] Sofia; [09:02] Marcos).

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica e meta | Condição | Fonte |
| --- | --- | --- | --- | --- |
| PRD-OBJ-01 | Reduzir a dependência de polling para mudanças de status. | **Meta operacional:** a experiência deve atender à latência percebida inferior a 10 segundos; o worker deve consultar pendentes a cada 2 segundos. | Medir o intervalo entre a confirmação da mudança e a primeira tentativa de entrega; a reunião não forneceu uma linha de base já instrumentada. | [09:00]–[09:02] Marcos; [09:09]–[09:10] Diego e Larissa |
| PRD-OBJ-02 | Preservar a correspondência entre mudança confirmada e evento elegível. | **Meta de consistência:** para cada endpoint elegível, nenhuma mudança de status deve ser confirmada sem o respectivo evento registrado na outbox; falha no registro deve impedir o commit. | Validar em teste transacional a combinação pedido + histórico + evento e o rollback quando o enqueue falhar; validar também que um customer sem endpoint elegível não gera linha. | [09:33]–[09:34] Marcos, Bruno e Diego; [09:40]–[09:41] Bruno e Diego |
| PRD-OBJ-03 | Dar tratamento finito e recuperável às falhas de entrega. | **Meta operacional:** até 5 tentativas, com backoff de `1m/5m/30m/2h/12h`; após o limite, persistir a falha em DLQ. | Simular timeout ou indisponibilidade do endpoint e verificar retry, DLQ e possibilidade de replay. | [09:15]–[09:18] Diego, Bruno e Larissa |

Essas são metas operacionais e de consistência fechadas na reunião; não foi
definido KPI comercial, percentual de conversão ou SLA de disponibilidade.

## 5. Escopo

### Incluído

- Notificações outbound para mudanças de status de pedidos ([09:00]–[09:03]
  Marcos e Sofia).
- Cadastro, edição, remoção e listagem de configurações por customer, com URL,
  secret gerada pela plataforma e status/eventos selecionados ([09:31]–[09:34]
  Marcos, Bruno e Larissa).
- Filtragem do evento na inserção da outbox, para não criar linha quando nenhum
  webhook do customer assinar o status ([09:33]–[09:34] Marcos, Bruno e Diego).
- Registro transacional do evento e snapshot do payload no momento da mudança
  ([09:40]–[09:41] Bruno e Diego; [09:51]–[09:52] Larissa e Diego).
- Entrega assíncrona com timeout, retry, DLQ e replay manual administrativo
  ([09:09]–[09:18] Diego e Larissa).
- Histórico dos últimos 100 deliveries, com resultado, payload, resposta e
  tempo de resposta ([09:34] Marcos).
- HMAC-SHA256, secret por endpoint, rotação com validade paralela da secret
  antiga por 24 horas, HTTPS e limite de payload de 64 KB ([09:19]–[09:24]
  Sofia, Diego e Larissa).

### Fora de escopo

- **PRD-OOS-01:** Webhooks inbound enviados pelos clientes para a plataforma;
  a entrega desta fase é somente outbound ([09:02]–[09:03] Sofia e Marcos).
- **PRD-OOS-02:** E-mail de alerta ou fallback para falhas consecutivas; foi
  adiado para uma fase posterior ([09:37]–[09:38] Marcos e Larissa).
- **PRD-OOS-03:** Dashboard visual para o cliente; o painel foi separado como
  projeto do time de frontend ([09:39]–[09:40] Marcos e Larissa).
- **PRD-OOS-04:** Arquivamento ou limpeza das linhas entregues da outbox nesta
  fase ([09:08] Diego).
- **PRD-OOS-05:** Garantia de ordering global, escala para múltiplos workers e
  exactly-once; a primeira versão assume single-worker e at-least-once
  ([09:12]–[09:13] Diego e Larissa; [09:24]–[09:26] Diego, Sofia e Larissa).

### Questões abertas ou hipóteses

- **PRD-QA-01 — Rate limiting de saída:** será observado antes de uma decisão; não é
  compromisso do release ([09:38]–[09:39] Diego e Larissa).
- **PRD-QA-02 — Escala futura:** particionamento por `order_id`, lock/claim e ordering após
  múltiplos workers foram adiados ([09:12]–[09:13] Diego e Larissa).
- **PRD-QA-03 — Retenção:** a duração exata de retenção e o arquivamento da outbox/DLQ não
  foram decididos ([09:08] Diego).
- **PRD-QA-04 — Permissões do CRUD:** a reunião aceitou qualquer role autenticada nesta
  fase, mas deixou o endurecimento para depois ([09:36]–[09:37] Sofia e
  Marcos).
- **PRD-QA-05 — Indexação do retry:** a correspondência exata entre as cinco
  tentativas e os cinco intervalos de backoff deve ser confirmada antes do teste
  final da agenda ([09:15]–[09:17] Diego e Larissa).

## 6. Requisitos funcionais

A reunião não atribuiu prioridades formais. Portanto, todos os requisitos
abaixo têm prioridade registrada como “não informada”; a ordem apresentada
organiza o fluxo do produto, não cria uma priorização nova.

### PRD-FR-01 — Notificar mudança de status

- **Ator:** OMS/sistema de pedidos.
- **Requisito:** deve criar uma notificação outbound quando o status de um
  pedido mudar, direcionada aos endpoints ativos e elegíveis do customer.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-01, PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:00]–[09:03] Marcos e Sofia.
- **Critérios:** PRD-CA-01 e PRD-CA-02.

### PRD-FR-02 — Cadastrar configuração de webhook

- **Ator:** usuário autenticado/cliente B2B.
- **Requisito:** deve permitir criar uma configuração via `POST`, informando
  `url` e a lista de status/eventos desejados; a plataforma deve gerar a secret
  e devolvê-la na criação.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03, PRD-DEP-04 e PRD-DEP-05.
- **Fonte:** [09:31] Marcos.
- **Critérios:** PRD-CA-03 e PRD-CA-04.

### PRD-FR-03 — Associar configuração ao customer informado

- **Ator:** usuário autenticado/sistema de configuração.
- **Requisito:** deve associar a configuração ao `customer_id` informado no
  body ou no path; o vínculo não deve ser derivado do JWT do usuário operador.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03.
- **Fonte:** [09:32] Bruno; [09:32] Marcos; [09:32] Larissa.
- **Critérios:** PRD-CA-05.

### PRD-FR-04 — Gerenciar e listar configurações

- **Ator:** usuário autenticado.
- **Requisito:** deve permitir editar via `PATCH`, remover via `DELETE` e
  listar via `GET` as configurações de webhook de um customer.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:33] Bruno.
- **Critérios:** PRD-CA-06.

### PRD-FR-05 — Filtrar status na criação do evento

- **Ator:** OMS/sistema de pedidos.
- **Requisito:** deve selecionar, por endpoint, os status de pedido que serão
  recebidos e aplicar o filtro quando o evento for inserido na outbox; se nenhum
  endpoint do customer assinar o status, não deve criar evento para ele.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-01, PRD-DEP-03 e PRD-DEP-04.
- **Fonte:** [09:33]–[09:34] Marcos, Bruno e Diego.
- **Critérios:** PRD-CA-07.

### PRD-FR-06 — Consultar histórico de deliveries

- **Ator:** cliente B2B/usuário autenticado.
- **Requisito:** deve disponibilizar os últimos 100 deliveries de um webhook,
  incluindo sucesso/falha, payload, resposta e tempo de resposta.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:34] Marcos.
- **Critérios:** PRD-CA-08.

### PRD-FR-07 — Reprocessar item da DLQ

- **Ator:** administrador.
- **Requisito:** deve permitir replay manual de um item da DLQ por endpoint
  administrativo, recolocando o evento na outbox como pendente.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-02, PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:18] Diego; [09:35] Larissa.
- **Critérios:** PRD-CA-09.

### PRD-FR-08 — Autorizar e auditar replay

- **Ator:** sistema de autorização/auditoria.
- **Requisito:** deve exigir role `ADMIN` para replay e registrar qual usuário
  executou a operação.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:35]–[09:36] Sofia e Larissa.
- **Critérios:** PRD-CA-10.

### PRD-FR-09 — Rotacionar secret por endpoint

- **Ator:** usuário autenticado/cliente B2B.
- **Requisito:** deve permitir solicitar nova secret para um endpoint; a secret
  antiga deve permanecer válida por 24 horas e depois ser invalidada.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:21]–[09:22] Sofia.
- **Critérios:** PRD-CA-11.

### PRD-FR-10 — Registrar evento atomicamente com a mudança

- **Ator:** OMS/sistema de pedidos.
- **Requisito:** deve inserir o evento na mesma transação da atualização do
  pedido e do histórico; se a inserção do evento falhar, a mudança inteira deve
  sofrer rollback.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-01 e PRD-DEP-02.
- **Fonte:** [09:40]–[09:41] Bruno e Diego.
- **Critérios:** PRD-CA-12.

### PRD-FR-11 — Persistir snapshot do evento

- **Ator:** OMS/sistema de pedidos.
- **Requisito:** deve persistir o snapshot do payload no momento da mudança, em
  vez de reconstruí-lo no momento do envio.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-01 e PRD-DEP-02.
- **Fonte:** [09:51]–[09:52] Larissa, Diego e Bruno.
- **Critérios:** PRD-CA-13.

### PRD-FR-12 — Entregar payload e headers do evento

- **Ator:** worker de entrega/cliente B2B.
- **Requisito:** deve enviar JSON com `event_id`, `event_type`, timestamp ISO
  8601, pedido, customer, status anterior/novo e dados básicos como
  `total_cents`, sem `items`; deve enviar `X-Event-Id`, `X-Signature`,
  `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`.
- **Prioridade:** não informada na reunião.
- **Dependências:** PRD-DEP-02, PRD-DEP-03 e PRD-DEP-05.
- **Fonte:** [09:43]–[09:45] Diego, Bruno e Sofia.
- **Critérios:** PRD-CA-14.

## 7. Requisitos não funcionais

| ID | Requisito | Medida ou condição verificável | Fonte |
| --- | --- | --- | --- |
| PRD-NFR-01 | Latência e processamento assíncrono | O worker deve fazer polling a cada 2 segundos; a experiência esperada para o cliente é inferior a 10 segundos; nenhuma chamada HTTP de entrega ocorre dentro da transação de status. | [09:02] Marcos; [09:09]–[09:11] Diego e Larissa |
| PRD-NFR-02 | Retry finito e DLQ | Timeout de 10 segundos conta como falha; o evento tem no máximo cinco tentativas com `1m/5m/30m/2h/12h` e depois é persistido em DLQ separada. | [09:15]–[09:18] Diego, Bruno e Larissa; [09:42] Sofia e Diego |
| PRD-NFR-03 | Segurança do endpoint e do payload | URL deve ser HTTPS; o corpo deve usar HMAC-SHA256 com secret exclusiva por endpoint; a secret antiga permanece válida por 24 horas após rotação. | [09:19]–[09:24] Sofia, Diego e Larissa |
| PRD-NFR-04 | Limite de payload | Payload acima de 64 KB deve ser rejeitado, sem truncamento. | [09:23]–[09:24] Sofia, Diego e Larissa |
| PRD-NFR-05 | Semântica de entrega | A entrega é at-least-once; duplicidades são possíveis e o consumidor deve usar `X-Event-Id` para deduplicar; exactly-once não é oferecido. | [09:24]–[09:26] Diego, Sofia e Larissa |
| PRD-NFR-06 | Ordenação limitada | A primeira versão usa um único worker e processa por `created_at`, mantendo a ordenação aceita por `order_id`; não promete ordering global nem ordering após escala horizontal. | [09:12]–[09:13] Diego e Larissa |
| PRD-NFR-07 | Compatibilidade com os padrões existentes | O módulo deve reutilizar `AppError`, Pino, middleware de erro, schemas Zod, módulos em camadas e códigos de erro com prefixo `WEBHOOK_`; o replay deve usar a autorização `ADMIN` existente. | [09:27]–[09:30] Bruno e Larissa; [09:35]–[09:36] Sofia e Larissa; `src/shared/errors/app-error.ts:3-15`; `src/shared/logger/index.ts:1-32` |

## 8. Decisões e trade-offs principais

| ID | Decisão | Alternativa e trade-off | Estado | Fonte |
| --- | --- | --- | --- | --- |
| PRD-DEC-01 | Usar transactional outbox no MySQL existente e registrar o evento na transação do pedido. | HTTP síncrono poderia bloquear a mudança; Redis exigiria infraestrutura adicional. O trade-off é usar persistência e polling no banco para preservar consistência. | Fechada; ver [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md) | [09:04]–[09:08] Bruno, Larissa e Diego |
| PRD-DEC-02 | Executar a entrega em worker separado, com polling de 2 segundos e inicialmente um worker. | Trigger/listener não notifica adequadamente um processo externo; worker dentro da API ficaria acoplado ao ciclo de reinício. O trade-off é aceitar polling e não oferecer ordering global após escala. | Fechada; ver [ADR-005](adrs/ADR-005-worker-separado-em-polling.md) | [09:09]–[09:13] Diego, Marcos e Larissa |
| PRD-DEC-03 | Aplicar cinco tentativas com backoff e mover falha permanente para DLQ separada. | Retry indefinido deixaria eventos pendurados; três tentativas cobririam janela menor. O trade-off é aceitar DLQ e replay manual. | Fechada; ver [ADR-002](adrs/ADR-002-retry-com-backoff-e-dlq.md) | [09:15]–[09:18] Diego, Bruno e Larissa |
| PRD-DEC-04 | Usar HMAC-SHA256 e secret única por endpoint, com rotação e grace period de 24 horas. | Secret global ampliaria o impacto de um vazamento. O trade-off é gerir secrets por endpoint e rotação. | Fechada; ver [ADR-003](adrs/ADR-003-hmac-sha256-com-secret-por-endpoint.md) | [09:19]–[09:22] Sofia |
| PRD-DEC-05 | Usar at-least-once com `X-Event-Id` para deduplicação pelo consumidor. | Exactly-once exigiria coordenação complexa entre plataforma e cliente. O trade-off é aceitar duplicidade e transferir a deduplicação ao consumidor. | Fechada; ver [ADR-004](adrs/ADR-004-entrega-at-least-once-com-x-event-id.md) | [09:24]–[09:26] Diego, Sofia e Larissa |
| PRD-DEC-06 | Persistir o payload como snapshot no momento da inserção. | Renderizar no envio poderia refletir um estado posterior do pedido. O trade-off é armazenar mais dados na outbox para preservar o significado original. | Fechada; relacionada a [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md) | [09:51]–[09:52] Larissa, Diego e Bruno |
| PRD-DEC-07 | Reutilizar os padrões de módulos, erros, logging e validação existentes. | Criar uma estrutura própria foi substituído pelo reuso máximo; o trade-off é que o novo módulo seguirá os contratos e convenções já existentes. | Fechada; ver [ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes.md) | [09:27]–[09:30] Bruno e Larissa |

## 9. Dependências

| ID | Dependência | Requisitos afetados | Impacto se indisponível | Fonte |
| --- | --- | --- | --- | --- |
| PRD-DEP-01 | Transação de `OrderService.changeStatus`, que hoje reúne atualização do pedido, histórico e estoque. | PRD-FR-01, PRD-FR-05, PRD-FR-10 e PRD-FR-11 | Não será possível garantir que a mudança confirmada e o evento tenham o mesmo commit. | `src/modules/orders/order.service.ts:126-179`; [09:40]–[09:41] Bruno e Diego |
| PRD-DEP-02 | MySQL via Prisma e modelos propostos de configuração, outbox e DLQ. As tabelas de webhook ainda precisam ser criadas; não existem no código atual. | PRD-FR-01, PRD-FR-06, PRD-FR-07, PRD-FR-10, PRD-FR-11 e PRD-FR-12 | Não haverá persistência transacional, histórico ou replay. | `prisma/schema.prisma:5-9`; [09:06] Diego; [09:18] Diego; [09:21] Bruno |
| PRD-DEP-03 | Autenticação JWT, `customer_id` explícito e autorização `requireRole('ADMIN')` para replay. | PRD-FR-02, PRD-FR-03, PRD-FR-04, PRD-FR-06, PRD-FR-07, PRD-FR-08 e PRD-FR-09 | Configurações e replay não poderão respeitar o modelo de acesso definido. | `src/middlewares/auth.middleware.ts:27-61`; [09:32] Larissa; [09:35]–[09:37] Sofia, Larissa e Marcos |
| PRD-DEP-04 | Catálogo e máquina de estados de pedidos existentes. | PRD-FR-01, PRD-FR-05 e PRD-FR-12 | Filtros poderiam aceitar status inválidos ou divergir das transições do OMS. | `prisma/schema.prisma:16-23`; `src/modules/orders/order.status.ts:3-37`; [09:33]–[09:34] Marcos, Bruno e Diego |
| PRD-DEP-05 | Infraestrutura compartilhada de schemas Zod, `AppError`, middleware de erro, Pino e composição da API. | PRD-FR-02, PRD-FR-04, PRD-FR-06, PRD-FR-07, PRD-FR-08, PRD-FR-09 e PRD-FR-12 | O novo módulo perderia compatibilidade com validação, respostas de erro, logging e integração de rotas existentes. | `src/middlewares/validate.middleware.ts:11-36`; `src/shared/errors/app-error.ts:3-15`; `src/middlewares/error.middleware.ts:14-65`; `src/shared/logger/index.ts:1-32`; [09:27]–[09:30] Bruno e Larissa |
| PRD-DEP-06 | Novos artefatos propostos para o worker separado e para o módulo de webhooks, ainda inexistentes. | PRD-FR-01, PRD-FR-06, PRD-FR-07 e PRD-FR-12 | A entrega assíncrona não poderá ser executada até que a entry point e o módulo sejam implementados. | [09:11] Larissa; [09:27]–[09:28] Bruno |

## 10. Riscos e mitigação

As probabilidades abaixo são avaliações qualitativas deste PRD baseadas nos
cenários discutidos; a reunião não atribuiu probabilidades formais.

| ID | Risco | Probabilidade | Impacto | Mitigação | Fonte |
| --- | --- | --- | --- | --- | --- |
| PRD-RISK-01 | O endpoint do cliente ficar indisponível ou lento. | Média — avaliação baseada na discussão de clientes offline e timeout. | Retries por quase 15 horas, aumento da DLQ e atraso para o consumidor. | Timeout de 10 segundos, limite de cinco tentativas, backoff, DLQ com motivo/payload e replay ADMIN. E-mail não é mitigação desta fase porque está fora de escopo. | [09:15]–[09:18] Diego, Bruno e Larissa; [09:42] Sofia e Diego |
| PRD-RISK-02 | Secret de um endpoint ser vazada ou registrada de forma indevida. | Média — avaliação baseada no caso de vazamento relatado. | Falsificação de chamadas para o endpoint comprometido. | Secret isolada por endpoint, HMAC-SHA256, HTTPS, rotação com validade antiga por 24 horas e revisão de segurança antes do deploy. | [09:19]–[09:24] Sofia; [09:22] Diego; [09:45]–[09:47] Larissa e Sofia |
| PRD-RISK-03 | Crescimento de linhas entregues, pendentes ou na DLQ sem política de retenção. | Média — avaliação baseada na retenção ainda não decidida. | Custo e dificuldade operacional para consultar e processar as tabelas. | **Mitigação proposta:** monitorar volume e idade e decidir política de arquivamento em trabalho posterior. A mitigação definitiva permanece uma questão aberta. | [09:08] Diego |
| PRD-RISK-04 | Duplicidade ou mudança de ordem percebida pelo cliente. | Média — avaliação derivada da semântica at-least-once e do limite de single-worker. | Consumidor pode processar o mesmo evento duas vezes ou receber expectativas de ordering não garantidas. | Enviar UUID em `X-Event-Id`, documentar deduplicação no consumidor e explicitar que não há ordering global. | [09:12]–[09:13] Diego e Larissa; [09:24]–[09:26] Diego, Sofia e Larissa |

## 11. Critérios de aceitação

### PRD-FR-01 — Notificar mudança de status

- **PRD-CA-01:** Dado um pedido cujo status foi alterado com sucesso e um endpoint ativo do customer elegível para o status, quando a transação for confirmada, então deve existir um evento pendente para entrega desse endpoint.
- **PRD-CA-02:** Dado um customer sem endpoint elegível para o status, quando o status mudar, então nenhum evento deve ser criado para esse customer.

### PRD-FR-02 — Cadastrar configuração de webhook

- **PRD-CA-03:** Dado um usuário autenticado e uma URL HTTPS com uma lista válida de status, quando enviar a criação da configuração, então o sistema deve persistir o endpoint e devolver a secret gerada na resposta de criação.
- **PRD-CA-04:** Dado uma URL `http`, quando enviar a criação da configuração, então o sistema deve rejeitar a solicitação como inválida e não criar a configuração.

### PRD-FR-03 — Associar configuração ao customer informado

- **PRD-CA-05:** Dado um `customer_id` explícito no body ou no path, quando um usuário autenticado criar ou consultar a configuração, então o resultado deve estar associado ao customer indicado e não ao subject do JWT.

### PRD-FR-04 — Gerenciar e listar configurações

- **PRD-CA-06:** Dada uma configuração existente, quando um usuário autenticado executar `PATCH`, `DELETE` ou `GET` para o customer correspondente, então a alteração, remoção ou listagem deve refletir a operação solicitada sem expor configurações de outro customer.

### PRD-FR-05 — Filtrar status na criação do evento

- **PRD-CA-07:** Dado um customer com endpoints que assinam conjuntos diferentes de status, quando ocorrer uma transição, então a outbox deve conter eventos apenas para os endpoints que assinam o novo status.

### PRD-FR-06 — Consultar histórico de deliveries

- **PRD-CA-08:** Dado um webhook com deliveries registrados, quando o usuário consultar o histórico, então a resposta deve conter no máximo os 100 deliveries mais recentes, com resultado, payload, resposta e tempo de resposta.

### PRD-FR-07 — Reprocessar item da DLQ

- **PRD-CA-09:** Dado um item persistido na DLQ, quando um administrador solicitar replay, então o evento deve voltar à outbox com estado pendente para nova tentativa.

### PRD-FR-08 — Autorizar e auditar replay

- **PRD-CA-10:** Dado um usuário sem role `ADMIN`, quando tentar executar replay, então a operação deve ser recusada; dado um usuário `ADMIN`, quando o replay for aceito, então o executor deve ficar registrado para auditoria.

### PRD-FR-09 — Rotacionar secret por endpoint

- **PRD-CA-11:** Dado um endpoint com secret atual, quando a rotação for solicitada, então uma nova secret deve ser disponibilizada, a antiga deve validar entregas durante 24 horas e deve deixar de ser válida após essa janela.

### PRD-FR-10 — Registrar evento atomicamente com a mudança

- **PRD-CA-12:** Dado um pedido com transição válida e um endpoint elegível, quando a mudança for confirmada, então pedido, histórico e evento devem compartilhar o mesmo commit; se a inserção do evento falhar, então pedido, estoque, histórico e evento devem ser revertidos.

### PRD-FR-11 — Persistir snapshot do evento

- **PRD-CA-13:** Dado um evento criado para uma transição, quando o pedido mudar novamente antes da entrega, então o payload armazenado do primeiro evento deve continuar representando a transição original.

### PRD-FR-12 — Entregar payload e headers do evento

- **PRD-CA-14:** Dado um delivery enviado ao endpoint, quando o cliente receber a requisição, então ela deve conter JSON com os campos de evento, pedido, customer e transição definidos, os headers `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`, e não deve conter `items`.

### Requisitos não funcionais essenciais

- **PRD-CA-15:** Dado o worker em operação, quando houver eventos pendentes, então a consulta deve seguir o intervalo de 2 segundos e processar os pendentes mais antigos; o teste de validação deve medir a latência para verificar a meta inferior a 10 segundos.
- **PRD-CA-16:** Dado um endpoint que não responde, quando a chamada atingir 10 segundos, então ela deve ser tratada como falha, seguir os cinco intervalos de retry registrados e, após o limite, preservar o evento na DLQ.
- **PRD-CA-17:** Dado um reenvio do mesmo evento, quando o consumidor receber a requisição, então o `X-Event-Id` deve permanecer igual para permitir deduplicação; o aceite não deve exigir exactly-once.
- **PRD-CA-18:** Dado um payload acima de 64 KB, quando o sistema tentar validá-lo para entrega, então deve rejeitá-lo sem truncamento; dado um endpoint HTTP, quando for cadastrado, então deve rejeitar a URL por não usar HTTPS.
- **PRD-CA-19:** Dado o processamento inicial com single-worker, quando houver várias mudanças do mesmo pedido, então os eventos devem ser processados pela ordem de `created_at`; o produto não deve anunciar ordering global.
- **PRD-CA-20:** Dado um erro de domínio do módulo, quando ele for retornado pela API, então a resposta deve seguir a infraestrutura existente e usar código com prefixo `WEBHOOK_`; dado um replay, então a autorização deve usar `ADMIN`.

## 12. Estratégia de testes e validação

A validação será orientada pelos critérios acima e deverá distinguir testes do
produto futuro de comportamentos que já existem no OMS:

- **Configuração e autorização:** testar criação, vínculo por `customer_id`,
  edição, remoção, listagem, rotação de secret, URL HTTP, limite de payload e
  replay com e sem `ADMIN` (PRD-FR-02 a PRD-FR-09; PRD-CA-03 a PRD-CA-11 e
  PRD-CA-18). A estratégia deve reutilizar o padrão de autenticação e validação
  observado em `src/modules/orders/order.routes.ts:12-24` e
  `src/middlewares/validate.middleware.ts:11-36`.
- **Integração transacional:** testar transições válidas e inválidas, estoque,
  histórico e outbox no mesmo commit, incluindo rollback quando o enqueue
  falhar (PRD-FR-01, PRD-FR-05, PRD-FR-10 e PRD-FR-11; PRD-CA-01, PRD-CA-02,
  PRD-CA-07, PRD-CA-12 e PRD-CA-13). O ponto de partida é a transação existente
  em `src/modules/orders/order.service.ts:131-178` e as regras de
  `src/modules/orders/order.status.ts:3-37`.
- **Worker e resiliência:** simular sucesso, timeout de 10 segundos, erro de
  rede, cinco tentativas, backoff, ida para DLQ e replay; verificar polling,
  snapshot, histórico e preservação do `X-Event-Id` (PRD-OBJ-01, PRD-OBJ-03,
  PRD-CA-08, PRD-CA-09, PRD-CA-13 e PRD-CA-15 a PRD-CA-17).
- **Contrato outbound e segurança:** validar HMAC-SHA256 com secret do
  endpoint, rotação dentro e fora da janela de 24 horas, headers, ausência de
  `items`, HTTPS e rejeição do payload acima de 64 KB (PRD-FR-09 e PRD-FR-12;
  PRD-CA-11, PRD-CA-14 e PRD-CA-18).
- **Observabilidade e regressão:** verificar auditoria do replay, códigos
  `WEBHOOK_*`, correlação de requisições e ausência de regressão nos testes já
  existentes de pedidos e autenticação (`tests/orders.test.ts` e
  `tests/auth.test.ts`). Os detalhes de métricas, logs, tracing, schemas e
  contratos permanecem especificados no FDD.
