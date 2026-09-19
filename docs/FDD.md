# FDD — Sistema de Webhooks de Notificação de Pedidos

Versão: 1.0 documental
Data da reunião: quinta-feira; a data calendárica não foi informada em TRANSCRICAO.md
Responsável: não informado na transcrição
Status: proposta de implementação orientada por evidências

Este documento detalha como construir a feature no OMS existente. Decisões
fechadas são identificadas pela fonte na transcrição; nomes de artefatos, rotas,
campos ou mecanismos ainda inexistentes no repositório são tratados como
propostas. As referências de código foram verificadas e não significam que a
feature já esteja implementada.

## 1. Contexto e motivação técnica

O OMS atual mantém pedidos, clientes, produtos, estoque e histórico de
transições em Node.js com TypeScript, Express, Prisma e MySQL. A mudança de
status do pedido já ocorre em uma transação que atualiza estoque, pedido e
histórico em src/modules/orders/order.service.ts:126-179. A necessidade é
permitir que clientes B2B recebam notificações sem continuar fazendo polling de
GET /orders; a reunião aceitou latência inferior a 10 segundos como tempo real
([09:00]-[09:02] Marcos).

O escopo é exclusivamente outbound: a plataforma envia eventos para endpoints
dos clientes, e os clientes não enviam webhooks para a plataforma
([09:02]-[09:03] Sofia e Marcos). O fluxo envolve:

- a API de configuração, usada por usuários autenticados;
- o serviço de pedidos, que cria o evento na mesma transação da mudança de
  status;
- um worker separado, que lê a outbox e faz as chamadas HTTP;
- o endpoint externo do cliente, que valida HMAC e deduplica por X-Event-Id;
- administradores, que podem fazer replay de itens na DLQ.

O sistema não deve fazer chamada HTTP externa dentro da transação de pedidos.
Uma falha de entrega não pode desfazer um status já confirmado; uma falha ao
registrar o evento na outbox deve impedir o commit da mudança que originaria a
notificação ([09:04]-[09:06] Bruno, Larissa e Diego; [09:40]-[09:41] Bruno e
Diego).

## 2. Objetivos técnicos

1. Manter a consistência entre mudança de status, histórico e registro do
   evento: o status só pode ser confirmado se o snapshot correspondente também
   for gravado na mesma transação ([09:40]-[09:41] Bruno e Diego).
2. Alcançar latência percebida inferior a 10 segundos, aceitando o pior caso de
   aproximadamente 2 segundos introduzido pelo polling
   ([09:02] Marcos; [09:09]-[09:10] Diego, Marcos e Larissa).
3. Entregar com semântica at-least-once e fornecer um UUID estável em
   X-Event-Id, deixando a deduplicação sob responsabilidade do consumidor
   ([09:24]-[09:26] Diego, Sofia e Larissa).
4. Preservar o estado do evento no momento da transição por meio de snapshot
   persistido, sem reconstruí-lo com um estado posterior
   ([09:51]-[09:52] Bruno, Larissa e Diego).
5. Isolar cada endpoint com uma secret própria, assinar o corpo com HMAC-SHA256
   e permitir rotação com a secret anterior válida por 24 horas
   ([09:19]-[09:22] Sofia).
6. Desacoplar a disponibilidade do cliente externo do ciclo de vida da API:
   o worker deve ser processo separado, usando o mesmo banco e uma instância
   própria de Prisma por processo ([09:11] Diego e Larissa; [09:29]-[09:30]
   Diego e Bruno).

## 3. Escopo e exclusões

### Incluído

- Cadastro, edição, remoção e listagem de configurações de webhook por
  customer, com URL, status assinados, estado ativo e secret gerada pela
  plataforma ([09:31]-[09:33] Marcos, Bruno e Larissa).
- Seleção de status na configuração e filtro na inserção da outbox; quando
  nenhum endpoint do customer assinar o status, nenhum evento desse endpoint
  deve ser inserido ([09:33]-[09:34] Marcos, Bruno e Diego).
- Outbox transacional em MySQL, snapshot JSON do evento e worker separado em
  polling ([09:06]-[09:10] Diego e Larissa; [09:40]-[09:41] Bruno e Diego).
- Entrega HTTP com timeout de 10 segundos, retry limitado, backoff, DLQ
  persistida e replay administrativo ([09:15]-[09:18] Diego, Bruno e Larissa;
  [09:42] Sofia e Diego).
- Histórico dos últimos 100 deliveries, incluindo sucesso/falha, payload,
  resposta e tempo de resposta ([09:34] Marcos).
- HMAC-SHA256, secret por endpoint, rotação, HTTPS, limite de 64 KB e headers
  do contrato outbound ([09:19]-[09:24] Sofia, Diego e Larissa;
  [09:43]-[09:45] Diego, Bruno e Sofia).

### Fora de escopo

- Webhooks inbound enviados pelos clientes para a plataforma
  ([09:02]-[09:03] Sofia e Marcos).
- E-mail de alerta ou fallback para falhas consecutivas
  ([09:37]-[09:38] Marcos e Larissa).
- Dashboard visual para o cliente; a reunião separou esse trabalho para o time
  de frontend ([09:39]-[09:40] Marcos e Larissa).
- Arquivamento ou limpeza de linhas entregues da outbox nesta fase
  ([09:08] Diego).
- Garantia de ordering global e solução de escala para múltiplos workers
  ([09:12]-[09:13] Diego e Larissa).
- Exactly-once; a plataforma oferecerá at-least-once e o consumidor deverá
  deduplicar ([09:24]-[09:26] Diego, Sofia e Larissa).
- Rate limiting de saída na primeira versão; o comportamento será observado
  antes de uma decisão ([09:38]-[09:39] Diego e Larissa).

### Adiado

- Política de retenção e arquivamento da outbox e da DLQ ([09:08] Diego).
- Estratégia para múltiplos workers, particionamento por order_id, lock
  pessimista e ordering após escala ([09:12]-[09:13] Diego e Larissa).
- Endurecimento das permissões do CRUD de configuração; nesta fase, o CRUD
  usa autenticação normal e apenas o replay exige ADMIN
  ([09:35]-[09:37] Sofia, Marcos e Larissa).

### Hipóteses e questões abertas de implementação

- A transcrição fixa a intenção dos endpoints, mas não fecha todos os prefixos,
  nomes finais de campos, envelopes de resposta ou códigos específicos. As
  rotas abaixo são uma proposta compatível com o prefixo /api/v1 já usado.
- O tamanho exato do batch, a técnica de claim/lock, a recuperação de itens
  presos em PROCESSING e a retenção do histórico de deliveries precisam ser
  confirmados.
- A reunião registra cinco tentativas e cinco intervalos
  (1m/5m/30m/2h/12h), mas não esclarece se a tentativa inicial entra na
  contagem. Este FDD preserva os dois dados e deixa a indexação exata como
  questão aberta.
- O formato de codificação da assinatura, a canonicalização do timestamp e a
  identidade do evento em um replay ainda não foram fechados.

## 4. Fluxos detalhados e diagramas

### 4.1 Criação do evento na outbox

O fluxo abaixo é uma proposta sobre a transação existente. webhook_outbox,
WebhookConfiguration e publishWebhookEvent ainda não existem no código-base;
foram mencionados ou propostos na reunião ([09:06], [09:21], [09:41] Diego,
Bruno e Sofia).

1. OrderService.changeStatus inicia a transação Prisma existente e carrega o
   pedido com seus itens.
2. O serviço reutiliza canTransition, shouldDebitStock e
   shouldReplenishStock de src/modules/orders/order.status.ts.
3. O serviço atualiza estoque quando a transição exigir, altera o status e
   grava OrderStatusHistory, mantendo o comportamento atual.
4. Ainda dentro do mesmo tx, uma função proposta
   publishWebhookEvent(tx, order, fromStatus, toStatus) busca configurações
   ativas do customer cujo filtro contenha toStatus.
5. Para cada configuração elegível, cria um UUID de evento e monta o snapshot
   com event_id, event_type igual a order.status_changed, timestamp ISO 8601,
   order_id, order_number, from_status, to_status, customer_id e campos básicos
   como total_cents. O snapshot não contém items
   ([09:43]-[09:44] Diego e Bruno).
6. A função grava snapshot, customer, endpoint e estado inicial PENDING na
   outbox. O uso de UUID segue [09:51] Diego e Larissa.
7. Se não houver configuração elegível, nenhuma linha é criada para aquele
   endpoint.
8. Se qualquer inserção falhar, a exceção sobe pelo tx e causa rollback da
   mudança de status, do histórico e de qualquer atualização de estoque.
9. Depois do commit, o controller retorna o pedido atualizado. Nenhuma chamada
   HTTP externa é feita pelo changeStatus.

O esquema abaixo é proposto para tornar o contrato implementável:

| Artefato proposto | Campos mínimos | Estado/origem |
| --- | --- | --- |
| Configuração de webhook | id UUID, customer_id, url, lista de status, active, secret atual, secret anterior e expiração da anterior | URL, customer, secret e estado foram discutidos em [09:21] Sofia; nomes de coluna são proposta |
| Outbox | id UUID/event id, webhook_id, customer_id, order_id, event_type, payload snapshot, status, tentativa, created_at, próxima tentativa e timestamps de processamento | Status e índice por status/data vêm de [09:08] Diego; demais campos são proposta |
| Delivery | endpoint/event id, resultado, status HTTP, payload, resposta, duração, tentativa e timestamp | Histórico dos últimos 100 e campos de observação vêm de [09:34] Marcos; granularidade por tentativa é proposta |
| Dead letter | referência do evento, webhook, payload, motivo, tentativas, timestamp da falha e dados de replay | Payload, motivo e timestamp vêm de [09:18] Diego; auditoria vem de [09:36] Sofia |

### 4.2 Processamento pelo worker

O worker será uma entry point separada, proposta como src/worker.ts, e não
deve ser executado dentro de src/server.ts ([09:11] Diego e Larissa). O script
npm run worker também é proposto e não existe em package.json.

1. O processo cria sua própria instância de PrismaClient, aponta para a mesma
   DATABASE_URL da API e inicializa o logger Pino.
2. Em loop, consulta a outbox a cada 2 segundos.
3. Seleciona eventos pendentes elegíveis, ordenados pelos mais antigos em
   created_at, em batch pequeno ([09:08]-[09:10] Diego).
4. Com um único worker, a expectativa aceita é ordem por created_at e ordenação
   implícita por order_id. A plataforma não promete ordering global
   ([09:12]-[09:13] Diego e Larissa).
5. Antes do envio, cada item deve ser reivindicado para não ser processado
   simultaneamente por outra execução. A reunião adiou a escolha entre claim,
   lock pessimista e estratégia para múltiplos workers
   ([09:12]-[09:13] Diego). A proposta é uma transição atômica PENDING para
   PROCESSING com marca temporal/identificador do worker, mas o SQL exato é
   questão aberta.
6. O worker envia o snapshot persistido, sem consultar novamente o pedido.
7. Em sucesso, registra o delivery e marca o item como DELIVERED.
8. Em falha, registra a tentativa, incrementa o contador e agenda retry ou
   move o item para DLQ conforme o limite.
9. Ao receber SIGINT/SIGTERM, interrompe novas aquisições e desconecta seu
   Prisma, seguindo o padrão de shutdown de src/server.ts:13-21. A drenagem
   do item corrente é proposta operacional.

### 4.3 Retry e backoff

O timeout fechado é de 10 segundos. Timeout, erro de conexão e resposta HTTP
considerada falha entram no retry ([09:42] Sofia e Diego). Este FDD propõe
considerar 2xx como sucesso e os demais códigos como falha; a reunião não
especificou a classificação de 3xx, 4xx e 5xx.

| Parâmetro | Valor |
| --- | --- |
| Limite | 5 tentativas |
| Intervalos registrados | 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas |
| Resultado após o limite | Falha permanente e persistência em DLQ separada |
| Timeout por chamada | 10 segundos |
| Semântica | At-least-once; reenvio pode produzir duplicidade |

A agenda deve ser configuração explícita. Antes de codificar o limite, a
equipe deve resolver a ambiguidade entre cinco tentativas e cinco intervalos:
se a chamada inicial conta, há quatro transições antes da quinta tentativa; se
os cinco intervalos são usados após a falha inicial, o total de chamadas é
maior que cinco. A decisão da reunião não permite escolher silenciosamente uma
interpretação.

Não há evidência de circuit breaker. O FDD não o introduz como decisão. Também
não há fallback de e-mail; o fallback operacional é DLQ e replay administrativo
([09:17]-[09:18] Diego e Larissa; [09:37]-[09:38] Marcos e Larissa).

### 4.4 DLQ e replay

1. Após o limite, o worker grava em webhook_dead_letter o payload snapshot,
   identificador do evento, endpoint, motivo, tentativas e timestamp.
2. O item deixa de ser elegível para polling normal. A representação exata
   entre marcar a outbox como FAILED, mover a linha ou manter referência é
   detalhe a confirmar; a reunião exige uma tabela separada
   ([09:17]-[09:18] Larissa e Diego).
3. Um administrador chama POST /api/v1/admin/webhooks/dead-letter/:id/replay.
4. authenticate valida o JWT e requireRole('ADMIN') restringe o replay,
   seguindo a decisão e o padrão real em src/middlewares/auth.middleware.ts:27-61.
5. O serviço registra o executor para auditoria e recoloca o evento como
   pendente na outbox ([09:35]-[09:36] Sofia e Larissa; [09:18] Diego).
6. A identidade do replay é questão aberta. A proposta é preservar event_id
   original, mas isso deve ser confirmado antes da implementação.

### 4.5 Diagramas

~~~mermaid
sequenceDiagram
    participant C as Cliente da API
    participant O as OrderService
    participant TX as Transação MySQL
    participant W as Worker
    participant E as Endpoint externo

    C->>O: PATCH /orders/:id/status
    O->>TX: atualizar pedido, estoque e histórico
    O->>TX: filtrar configurações e inserir snapshots
    alt erro no enqueue
        TX-->>O: rollback do conjunto
        O-->>C: erro da operação
    else commit
        TX-->>O: commit
        O-->>C: 200 pedido atualizado
        W->>TX: polling a cada 2s
        W->>E: POST snapshot + HMAC
        alt sucesso
            E-->>W: resposta HTTP
            W->>TX: DELIVERED e histórico
        else falha
            W->>TX: retry ou DLQ
        end
    end
~~~

Estados propostos:

~~~text
PENDING --claim--> PROCESSING --2xx--> DELIVERED
    |                    |
    |                    +--falha e ainda há tentativas--> PENDING
    |                    |
    |                    +--limite atingido--> DLQ / FAILED
    +--replay administrativo após DLQ---------------------> PENDING
~~~

A transição PROCESSING e a recuperação de itens abandonados exigem uma
estratégia de lease/lock ainda não decidida.

## 5. Contratos públicos

### 5.1 Convenções comuns

- As rotas usam o prefixo /api/v1, porque src/app.ts:55-73 monta o router nesse
  prefixo.
- Configuração, listagem, edição, remoção, deliveries e rotação exigem
  Authorization: Bearer JWT válido. O CRUD usa qualquer role autenticada nesta
  fase ([09:35]-[09:37] Sofia, Marcos e Larissa).
- Replay exige JWT e role ADMIN.
- IDs de recursos são UUID, seguindo prisma/schema.prisma:25-130 e os schemas
  Zod de src/modules/orders/order.schemas.ts:1-30.
- Rotas propostas devem usar validate e schemas Zod, como as rotas de pedidos em
  src/modules/orders/order.routes.ts:12-24.
- Erros de domínio usam AppError e códigos WEBHOOK_*. O envelope existente é
  { "error": { "code": "...", "message": "...", "details": ... } }, conforme
  src/middlewares/error.middleware.ts:14-65.
- X-Request-Id pode ser recebido ou gerado pelo middleware e é devolvido na
  resposta; ele não substitui X-Event-Id
  (src/middlewares/request-logger.middleware.ts:5-24).
- Nomes exatos das rotas de configuração e envelopes são propostas, pois a
  transcrição fechou casos de uso, mas não todos os detalhes HTTP.

### 5.2 Criar configuração

Método e rota proposta: POST /api/v1/webhooks

Autenticação: Bearer JWT; qualquer role autenticada nesta fase.

Request:

~~~json
{
  "customerId": "11111111-1111-1111-1111-111111111111",
  "url": "https://hooks.example.com/oms",
  "statuses": ["SHIPPED", "DELIVERED"]
}
~~~

Response proposta, status 201:

~~~json
{
  "id": "22222222-2222-2222-2222-222222222222",
  "customerId": "11111111-1111-1111-1111-111111111111",
  "url": "https://hooks.example.com/oms",
  "statuses": ["SHIPPED", "DELIVERED"],
  "active": true,
  "secret": "[SECRET GERADA PELA PLATAFORMA]"
}
~~~

Status codes: 201 para criação e secret gerada/devolvida ([09:31] Marcos);
400 para body, UUID, status ou URL inválidos; 401 para Bearer ausente/inválido;
404 para customer inexistente. Authorization e Content-Type são obrigatórios;
X-Request-Id é opcional e retornado na resposta.

O formato da secret e a regra de exposição posterior são propostas a confirmar;
a reunião fixa que a secret é gerada e devolvida na criação.

### 5.3 Listar configurações de um customer

Método e rota proposta: GET /api/v1/customers/:customerId/webhooks

Request:

~~~text
GET /api/v1/customers/11111111-1111-1111-1111-111111111111/webhooks
Authorization: Bearer <JWT>
X-Request-Id: 33333333-3333-3333-3333-333333333333
~~~

Response proposta, status 200:

~~~json
{
  "data": [
    {
      "id": "22222222-2222-2222-2222-222222222222",
      "customerId": "11111111-1111-1111-1111-111111111111",
      "url": "https://hooks.example.com/oms",
      "statuses": ["SHIPPED", "DELIVERED"],
      "active": true
    }
  ]
}
~~~

Status codes: 200; 400 para customer ID inválido; 401 para token ausente ou
inválido; 404 se a política escolhida exigir validação de existência do
customer. A resposta sem paginação é proposta; os limites de listagem não
foram definidos.

### 5.4 Editar configuração

Método e rota proposta: PATCH /api/v1/webhooks/:id

Request:

~~~json
{
  "url": "https://hooks.example.com/oms-v2",
  "statuses": ["PROCESSING", "SHIPPED"],
  "active": true
}
~~~

Response proposta, status 200:

~~~json
{
  "id": "22222222-2222-2222-2222-222222222222",
  "customerId": "11111111-1111-1111-1111-111111111111",
  "url": "https://hooks.example.com/oms-v2",
  "statuses": ["PROCESSING", "SHIPPED"],
  "active": true
}
~~~

Status codes: 200; 400 para body, UUID ou URL inválidos; 401 para autenticação
ausente/inválida; 404 para webhook inexistente. HTTPS é obrigatório. O PATCH
não deve receber secret em claro; essa política de exposição é proposta a
confirmar na revisão de segurança.

### 5.5 Remover configuração

Método e rota proposta: DELETE /api/v1/webhooks/:id

Request:

~~~text
DELETE /api/v1/webhooks/22222222-2222-2222-2222-222222222222
Authorization: Bearer <JWT>
X-Request-Id: 44444444-4444-4444-4444-444444444444
~~~

Response: 204, sem body.

Status codes: 204; 400 para UUID inválido; 401 para token ausente ou inválido;
404 para webhook inexistente. A política para eventos já existentes na outbox
quando a configuração é removida é questão aberta.

### 5.6 Histórico de deliveries

Método e rota conceitualmente definida: GET /api/v1/webhooks/:id/deliveries

Request:

~~~text
GET /api/v1/webhooks/22222222-2222-2222-2222-222222222222/deliveries
Authorization: Bearer <JWT>
X-Request-Id: 55555555-5555-5555-5555-555555555555
~~~

Response proposta, status 200:

~~~json
{
  "data": [
    {
      "eventId": "66666666-6666-6666-6666-666666666666",
      "status": "DELIVERED",
      "attempt": 1,
      "payload": {
        "event_id": "66666666-6666-6666-6666-666666666666",
        "event_type": "order.status_changed",
        "timestamp": "2026-09-19T12:00:00.000Z",
        "order_id": "77777777-7777-7777-7777-777777777777",
        "order_number": "ORD-000001",
        "from_status": "PROCESSING",
        "to_status": "SHIPPED",
        "customer_id": "11111111-1111-1111-1111-111111111111",
        "total_cents": 12990
      },
      "response": { "statusCode": 200 },
      "responseTimeMs": 142,
      "createdAt": "2026-09-19T12:00:01.000Z"
    }
  ]
}
~~~

Status codes: 200 com no máximo os últimos 100 deliveries; 400 para UUID
inválido; 401 para token ausente/inválido; 404 para webhook inexistente.
O conceito de últimos 100 vem de [09:34] Marcos; se cada item representa uma
tentativa ou um evento agregado é decisão de modelagem aberta.

### 5.7 Rotacionar secret

Método e rota proposta: POST /api/v1/webhooks/:id/secret/rotate

Request:

~~~json
{}
~~~

Response proposta, status 200:

~~~json
{
  "webhookId": "22222222-2222-2222-2222-222222222222",
  "secret": "[NOVA SECRET GERADA]",
  "previousSecretValidUntil": "2026-09-20T12:00:00.000Z"
}
~~~

Status codes: 200; 400 para UUID inválido; 401 para token ausente/inválido;
404 para webhook inexistente. A secret antiga permanece válida por 24 horas e
depois é invalidada ([09:21]-[09:22] Sofia).

### 5.8 Replay administrativo da DLQ

Método e rota conceitualmente definida: POST
/api/v1/admin/webhooks/dead-letter/:id/replay

Request:

~~~json
{}
~~~

Response proposta, status 202:

~~~json
{
  "deadLetterId": "88888888-8888-8888-8888-888888888888",
  "eventId": "66666666-6666-6666-6666-666666666666",
  "status": "PENDING",
  "replayedBy": "99999999-9999-9999-9999-999999999999"
}
~~~

Status codes: 202 para requeue; 400 para UUID inválido; 401 para token
ausente/inválido; 403 sem role ADMIN; 404 para item inexistente; 409 para
item já replayado ou em estado incompatível. A distinção entre 202 e 200 e o
estado terminal do registro original são propostas.

Headers: Authorization Bearer JWT é obrigatório e X-Request-Id correlaciona a
auditoria. A rota aplica requireRole('ADMIN'), fechado em [09:35]-[09:36]
Sofia e Larissa.

### 5.9 Contrato outbound de entrega

Método: POST para a URL HTTPS cadastrada. O corpo é o snapshot JSON persistido,
limitado a 64 KB e sem items.

Payload:

~~~json
{
  "event_id": "66666666-6666-6666-6666-666666666666",
  "event_type": "order.status_changed",
  "timestamp": "2026-09-19T12:00:00.000Z",
  "order_id": "77777777-7777-7777-7777-777777777777",
  "order_number": "ORD-000001",
  "from_status": "PROCESSING",
  "to_status": "SHIPPED",
  "customer_id": "11111111-1111-1111-1111-111111111111",
  "total_cents": 12990
}
~~~

| Header | Semântica |
| --- | --- |
| Content-Type: application/json | O corpo é JSON ([09:44] Diego). |
| X-Event-Id | UUID único criado na entrada da outbox; o consumidor deduplica reenvios ([09:24]-[09:26] Diego e Larissa). |
| X-Signature | HMAC-SHA256 sobre o corpo exato, com a secret do endpoint ([09:19]-[09:20] Sofia). Codificação final é proposta a confirmar. |
| X-Timestamp | Timestamp do envio, para detecção de replay attack ([09:44] Diego). Representação final é proposta a confirmar. |
| X-Webhook-Id | Identificador da configuração do endpoint ([09:44]-[09:45] Sofia e Diego). |

Este FDD propõe resposta HTTP 2xx como sucesso e demais códigos como falha; a
classificação exata não foi fechada. O timeout é 10 segundos. A plataforma não
promete exactly-once: o mesmo payload e X-Event-Id podem ser enviados novamente.

## 6. Erros, exceções e fallback

Erros de domínio devem seguir AppError e o prefixo WEBHOOK_, conforme
[09:28]-[09:30] Bruno e Larissa. A implementação pode reutilizar statusCode,
errorCode e details de src/shared/errors/app-error.ts:3-15 e o middleware
centralizado. Autenticação continua usando os códigos genéricos UNAUTHORIZED e
FORBIDDEN do middleware compartilhado.

| Condição | Código WEBHOOK_* | Status HTTP/worker | Tratamento | Retry/DLQ | Fonte |
| --- | --- | --- | --- | --- | --- |
| URL ausente, malformada ou não HTTPS | WEBHOOK_INVALID_URL | 400 | Rejeitar no schema | Não | [09:23] Sofia; src/middlewares/validate.middleware.ts:11-36 |
| Body, UUID ou status inválidos | WEBHOOK_VALIDATION_ERROR | 400 | Retornar issues no envelope | Não | [09:33]-[09:34] Marcos e Bruno |
| Customer ou webhook não encontrado | WEBHOOK_NOT_FOUND | 404 | Não criar/alterar recurso | Não | [09:31]-[09:33] Marcos e Bruno |
| Payload outbound acima de 64 KB | WEBHOOK_PAYLOAD_TOO_LARGE | 413 | Rejeitar sem truncar | Não; corrigir configuração | [09:23]-[09:24] Sofia, Diego e Larissa |
| Secret necessária para assinar indisponível | WEBHOOK_SECRET_REQUIRED | 500 | Não enviar sem assinatura; sinalizar operação | Não automático | [09:28]-[09:29] Bruno e Larissa |
| Falha ao inserir snapshot na outbox | WEBHOOK_OUTBOX_ENQUEUE_FAILED | 500 na operação de status | Propagar no tx e fazer rollback | Não é retry de delivery | [09:40]-[09:41] Bruno e Diego |
| Timeout de 10 segundos | WEBHOOK_DELIVERY_TIMEOUT | Worker | Registrar duração e motivo | Sim; DLQ após limite | [09:42] Sofia e Diego |
| Erro de rede/TLS ou resposta não 2xx | WEBHOOK_DELIVERY_FAILED | Worker | Registrar tentativa, resposta e duração | Sim; DLQ após limite | [09:15]-[09:18] Diego e Larissa |
| Limite de tentativas atingido | WEBHOOK_RETRY_EXHAUSTED | Worker | Persistir payload, motivo e timestamp na DLQ | Encerrar e aguardar replay | [09:17]-[09:18] Larissa e Diego |
| Item de DLQ inexistente | WEBHOOK_DLQ_NOT_FOUND | 404 | Não reencaminhar | Não | [09:18] Diego; código é proposta |
| Replay em estado incompatível | WEBHOOK_DLQ_REPLAY_CONFLICT | 409 | Não duplicar replay | Não | [09:18] Diego; comportamento é proposta |

### Estratégias de resiliência

- Snapshot transacional evita perda de evento; falha externa não toca no status
  já commitado.
- Timeout, falha de conexão e resposta não aceita seguem retry e backoff.
- DLQ separada preserva evidência para diagnóstico e reprocessamento
  ([09:17]-[09:18] Larissa e Diego).
- O worker deve ser idempotente na transição de estado, mas a entrega permanece
  at-least-once. Claim/lock e replay idempotente são propostas abertas.
- Não existe fallback de e-mail nesta fase; o fallback operacional é inspeção e
  replay administrativo.

### Invariantes

1. Não existe commit de mudança de status sem snapshot correspondente quando há
   webhook elegível.
2. Falha externa jamais desfaz pedido, estoque ou histórico já commitados.
3. O payload enviado é o snapshot persistido.
4. Secret não aparece em logs, métricas, traces ou erros.
5. O consumidor trata X-Event-Id como chave de deduplicação.

## 7. Observabilidade

Esta seção define a instrumentação necessária. Os nomes são propostas derivadas
do requisito de observabilidade do CHALLENGE.md e do logger existente.

### Métricas

- webhook_outbox_enqueued_total, por resultado de inserção;
- webhook_outbox_pending, gauge de pendentes;
- webhook_outbox_oldest_age_seconds, idade do pendente mais antigo;
- webhook_delivery_attempt_total, por resultado success, timeout, network_error,
  http_error ou dlq;
- webhook_delivery_duration_ms, histograma de duração;
- webhook_dlq_inserted_total e webhook_dlq_replayed_total;
- webhook_payload_rejected_total, por motivo de validação.

Não usar event_id ou customer_id como label de métrica. Alertas para idade
crescente da outbox, timeouts, erros e crescimento da DLQ são proposta
operacional; thresholds e ferramenta de dashboard não foram definidos.

### Logs

Usar Pino, timestamp ISO e campos estruturados, conforme
src/shared/logger/index.ts:13-29. Eventos mínimos:

- webhook_outbox_enqueued: requestId, eventId, webhookId, customerId, orderId,
  toStatus, resultado e duração;
- webhook_delivery_attempt: eventId, webhookId, tentativa, resultado, status
  HTTP e duração;
- webhook_dlq_inserted: eventId, webhookId, tentativa final e motivo;
- webhook_dlq_replayed: deadLetterId, eventId, replayedBy, requestId e resultado;
- webhook_config_changed: webhookId, customerId, operação e requestId.

Não registrar secret, X-Signature, Authorization, corpo completo ou resposta
externa em logs. O logger atual redige authorization, cookie, password,
passwordHash e token, mas não declara secret; a implementação deve ampliar a
redaction e passar pela revisão de segurança
([09:45]-[09:49] Sofia e Larissa; src/shared/logger/index.ts:4-20).

### Tracing

Não existe biblioteca de tracing em package.json; os spans abaixo são proposta:

- order.change_status;
- webhook.outbox.insert;
- webhook.worker.poll;
- webhook.delivery;
- webhook.dlq.replay.

Propagar requestId entre API, logs e auditoria; usar eventId para correlacionar
outbox, deliveries, retries e DLQ. Trace não deve carregar secret, assinatura ou
payload completo. Amostragem e backend ficam para a plataforma.

## 8. Dependências e compatibilidade

### Já existentes

- Node >=20, TypeScript, Express 4.21.1 e processo HTTP de src/server.ts:1-27,
  conforme package.json:7-20.
- Prisma Client 5.22.0 e MySQL via DATABASE_URL, conforme package.json:25-33
  e prisma/schema.prisma:5-9.
- Pino, Zod, UUID, autenticação JWT, AppError e error middleware.

### A criar ou confirmar

- Migração Prisma aditiva para configuração, outbox, deliveries e DLQ. Esses
  modelos não existem em prisma/schema.prisma; webhook_outbox e
  webhook_dead_letter são artefatos propostos ([09:06] e [09:18] Diego).
- Módulo src/modules/webhooks/ com controller, service, repository, routes,
  schemas e processador; a pasta ainda não existe ([09:27]-[09:28] Bruno).
- Entry point src/worker.ts e script npm run worker; ainda não existem
  ([09:11] Larissa).
- Cliente HTTP com timeout controlável. A escolha entre API nativa do runtime e
  dependência adicional não foi decidida.
- Instrumentação de métricas e tracing; não aparece no código atual.

### Compatibilidade e rollout

- Preservar o prefixo /api/v1 e não alterar rotas existentes.
- Aplicar a migration antes de iniciar versão que escreva ou leia os novos
  modelos. O ordenamento exato do rollout é procedimento a confirmar.
- API e worker usam a mesma DATABASE_URL, mas instâncias Prisma distintas por
  processo; o padrão está em src/config/database.ts:4-10.
- O parser global aceita 1 MB em src/app.ts:55-60; o limite de 64 KB deve ser
  verificado especificamente no webhook, sem reduzir outras rotas.
- Novos códigos WEBHOOK_* devem manter o envelope de AppError e do middleware
  atual.

## 9. Integração com o sistema existente

| Caminho real | Símbolo/área | Integração | Status | Fonte |
| --- | --- | --- | --- | --- |
| src/modules/orders/order.service.ts:126-179 | OrderService.changeStatus | Chamar publicação dentro de prisma.$transaction; falha sobe para rollback. | existente | Código; [09:40]-[09:41] |
| src/modules/orders/order.status.ts:1-37 | canTransition e regras de estoque | Reutilizar status e transições, sem duplicar máquina. | existente | Código; [09:33]-[09:34] |
| src/modules/orders/order.routes.ts:12-24 | Router de pedidos | Reutilizar authenticate, validate e controller. | existente | Código |
| src/modules/orders/order.schemas.ts:1-34 | Schemas Zod | Reutilizar padrão de UUID e enum; criar schema de HTTPS e limites. | existente | Código; [09:23] |
| src/modules/orders/order.repository.ts:18-68 | Repository Prisma | Referência para repositories; enqueue aceita Prisma.TransactionClient. | existente | Código; [09:41] |
| src/app.ts:22-73 | buildControllers e buildApp | Instanciar webhooks, registrar /api/v1 e manter middlewares. | existente | Código |
| src/routes/index.ts:13-30 | Controllers e buildApiRouter | Adicionar router e tipos de webhooks. | existente | Código |
| src/middlewares/auth.middleware.ts:27-61 | authenticate e requireRole | Proteger CRUD e replay ADMIN. | existente | Código; [09:35]-[09:36] |
| src/middlewares/validate.middleware.ts:11-36 | validate | Parse de body, params e query com Zod. | existente | Código |
| src/shared/errors/app-error.ts:3-15 | AppError | Criar erros WEBHOOK_* com status, code e details. | existente | Código; [09:28]-[09:30] |
| src/middlewares/error.middleware.ts:14-65 | errorMiddleware | Manter envelope e tratamento central. | existente | Código; [09:29] |
| src/shared/logger/index.ts:1-32 | Pino | Logs estruturados e redaction ampliada. | existente | Código; [09:29]-[09:30] |
| src/middlewares/request-logger.middleware.ts:5-27 | X-Request-Id | Correlacionar endpoints e auditoria. | existente | Código |
| src/config/database.ts:1-10 | createPrismaClient | API e worker usam instância própria no mesmo banco. | existente | Código; [09:29]-[09:30] |
| src/server.ts:1-27 | bootstrap e shutdown | Referência para entry point e desconexão do worker. | existente | Código; [09:11] |
| prisma/schema.prisma:5-130 | datasource, UUIDs, customer, order e status history | Adicionar migration; outbox/DLQ não existem. | existente | Código; [09:06], [09:18] |

Artefatos futuros não contabilizados como arquivos existentes: src/modules/webhooks/,
src/worker.ts, webhook_outbox, webhook_dead_letter e npm run worker.

## 10. Critérios de aceite técnicos

Os critérios abaixo são verificáveis por testes de integração, testes do worker,
inspeção de migration e revisão de segurança.

| ID | Dado | Quando | Então | Validação | Fonte |
| --- | --- | --- | --- | --- | --- |
| FDD-CA-01 | Pedido em transição válida e webhook elegível | Executa changeStatus | Pedido, estoque, histórico e snapshot entram na mesma transação | Teste de commit e consulta das persistências | [09:40]-[09:41]; código order.service |
| FDD-CA-02 | Falha na inserção da outbox | Transação tenta confirmar status | Pedido, estoque e histórico sofrem rollback | Teste de falha transacional | [09:40]-[09:41] |
| FDD-CA-03 | Um webhook assina SHIPPED e outro não | Pedido muda para SHIPPED | Só o endpoint elegível recebe linha | Teste com duas configurações | [09:33]-[09:34] |
| FDD-CA-04 | Pedido muda novamente depois do enqueue | Worker processa evento antigo | Payload preserva estado original | Comparação com estado posterior | [09:51]-[09:52] |
| FDD-CA-05 | Há eventos pendentes | Worker inicia loop | Consulta a cada 2s, em batch pequeno e por created_at | Teste de scheduler/ordenação | [09:08]-[09:10] |
| FDD-CA-06 | Único worker processa sequência do pedido | Há várias mudanças | Mantém ordenação aceita por created_at/order_id e não promete global | Teste de sequência | [09:12]-[09:13] |
| FDD-CA-07 | Chamada outbound não responde | Passam-se 10s | Registra timeout e agenda retry | Teste com servidor lento | [09:42] |
| FDD-CA-08 | Delivery falha | Worker agenda tentativas | Não ultrapassa cinco e usa 1m/5m/30m/2h/12h; indexação final precisa confirmação | Teste parametrizado e revisão | [09:15]-[09:17] |
| FDD-CA-09 | Limite de retry é excedido | Falha permanente ocorre | Payload, motivo, tentativas e timestamp vão para DLQ | Teste de exaustão | [09:17]-[09:18] |
| FDD-CA-10 | Usuário não é ADMIN | Chama replay | Responde 403 e não altera outbox | Teste com OPERATOR | [09:35]-[09:36]; auth.middleware |
| FDD-CA-11 | ADMIN possui DLQ | Chama replay | Item volta a pendente e executor fica auditável | Teste de requeue/log | [09:18]; [09:35]-[09:36] |
| FDD-CA-12 | Endpoint possui secret própria | Worker envia JSON | HMAC-SHA256 usa a secret daquele endpoint | Teste de assinatura | [09:19]-[09:21] |
| FDD-CA-13 | Secret é rotacionada | Rotação ocorre | Antiga funciona 24h e depois é invalidada | Teste com relógio controlado | [09:21]-[09:22] |
| FDD-CA-14 | URL é HTTP ou payload supera 64 KB | Validação ocorre | URL é rejeitada e payload grande não é truncado | Teste de schema/limite | [09:23]-[09:24] |
| FDD-CA-15 | Evento é entregue/repetido | Consumidor examina request | Headers exigidos existem e items não aparece | Teste de contrato outbound | [09:43]-[09:45] |
| FDD-CA-16 | Mesmo evento é reenviado | Consumidor processa entregas | X-Event-Id permite deduplicação; plataforma não promete exactly-once | Teste de retry/replay | [09:24]-[09:26] |
| FDD-CA-17 | API recebe CRUD, deliveries e replay | Rotas são chamadas | Contratos documentados operam sob /api/v1 | Testes HTTP de integração | [09:31]-[09:36] |
| FDD-CA-18 | Sucesso, falha, retry e DLQ ocorrem | Telemetria é emitida | Há métricas, logs com redaction e tracing correlacionados | Inspeção e teste de redaction | CHALLENGE.md; logger e request-logger |
| FDD-CA-19 | Código da feature é integrado | Build/testes executam | Composition root, AppError e changeStatus mantêm padrões existentes | Inspeção e testes | [09:27]-[09:30]; seção 9 |

Não são critérios fechados: rate limiting, retenção, escala multi-worker,
formato exato de resposta, claim/lock, indexação final do retry, política para
evento já enfileirado após remoção do webhook e identidade do replay. Esses
pontos permanecem questões abertas.

## 11. Riscos e mitigação

As probabilidades abaixo são avaliação operacional; a existência e natureza
dos riscos são apoiadas pelas fontes indicadas.

| Risco | Probabilidade | Impacto | Mitigação | Contingência | Fonte |
| --- | --- | --- | --- | --- | --- |
| Cliente externo indisponível ou lento | Alta | Retries, latência e DLQ | Timeout, backoff, limite, DLQ e histórico | Investigar DLQ e replay ADMIN; sem e-mail nesta fase | [09:15]-[09:18]; [09:42] |
| Vazamento de secret ou assinatura em log | Média | Falsificação de requests | Secret por endpoint, HMAC, HTTPS, redaction e revisão | Rotacionar secret e investigar logs | [09:19]-[09:24]; [09:45]-[09:49]; logger |
| Crescimento sem limite da outbox/DLQ | Alta | Custo, lentidão e operação degradada | Índices, batches, métricas e alertas | Definir retenção/arquivamento posteriormente | [09:08] |
| Duplicidade de entrega | Alta | Consumidor processa transição mais de uma vez | X-Event-Id e deduplicação no cliente | Reprocessamento idempotente do consumidor | [09:24]-[09:26] |
| Escala horizontal altera ordenação ou claims | Média | Ordem diferente ou envio paralelo | Primeiro release single-worker e sem promessa global | Decidir particionamento ou lock antes da escala | [09:12]-[09:13] |
| Claim/lock deixa item preso | Média | Evento sem retry ou reenviado indevidamente | Definir lease e transição atômica antes do worker | Job de recuperação e replay administrativo | [09:12]-[09:13]; questão aberta |
