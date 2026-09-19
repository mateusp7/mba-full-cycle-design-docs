---
name: fdd-writer
description: Conduz uma análise orientada por evidências para produzir um Feature Design Document (FDD) em português. No desafio de Webhooks de Notificação de Pedidos, gera um FDD acionável para implementação, alinhado ao CHALLENGE.md, ao código existente, à transcrição, aos ADRs e ao tracker.
---

# FDD Writer

## Objetivo

Produzir um Feature Design Document técnico, claro, acionável e rastreável.
O FDD responde à pergunta **como construir** uma feature específica: detalha
fluxos, contratos, integração com o sistema existente, erros, resiliência,
observabilidade, compatibilidade, critérios de aceite e riscos.

No desafio "Da Reunião ao Documento: Design Docs Gerados por IA", o FDD deve
ser escrito para o Sistema de Webhooks de Notificação de Pedidos e atender ao
CHALLENGE.md. O arquivo final esperado é docs/FDD.md, em Markdown.

O FDD não substitui os demais documentos:

- o PRD responde por que a feature existe e o que será entregue;
- o RFC apresenta a proposta arquitetural, alternativas e questões abertas;
- os ADRs registram decisões isoladas e suas consequências;
- o FDD explica como implementar em detalhe;
- o Tracker registra de onde veio cada item.

Evite repetir conteúdo de outro documento. Quando o contexto já estiver
registrado em RFC ou ADR, faça uma referência curta e concentre o FDD no
comportamento implementável.

## Regra de rastreabilidade

Toda afirmação do FDD deve ser sustentada por uma fonte identificável. Use esta
ordem de autoridade:

1. CHALLENGE.md, para formato, escopo e critérios de aceite do desafio;
2. TRANSCRICAO.md, para requisitos, restrições, decisões, alternativas,
   exclusões e questões abertas da reunião;
3. código existente, para comportamento, padrões, nomes de módulos e caminhos
   reais;
4. docs/mapping.md, como matriz de evidências consolidada;
5. RFC, PRD, ADRs e Tracker, como documentos derivados que devem ser conferidos
   contra as fontes primárias.

Não transforme uma sugestão em decisão, uma inferência do código em requisito,
uma questão adiada em escopo atual ou uma alternativa descartada em solução.
Classifique cada informação como:

- decisão fechada;
- requisito ou restrição;
- comportamento observado no código;
- proposta de implementação;
- hipótese;
- questão aberta ou item fora de escopo.

Se algo não tiver origem verificável, remova-o, marque-o como hipótese ou
registre-o como questão aberta. Nunca preencha uma lacuna com uma decisão
inventada. Se um artefato futuro for mencionado, diferencie claramente
"caminho existente" de "caminho proposto".

## Quando usar

Use esta skill para uma feature que precisa de especificação de implementação,
especialmente quando houver fluxos assíncronos, contratos HTTP ou de eventos,
persistência, resiliência, segurança, observabilidade e integração com código
existente.

Não use esta skill para produzir um PRD, RFC ou ADR isolado. Se esses documentos
forem usados como contexto, preserve a fronteira de cada um.

## Modo de trabalho no desafio

Quando o repositório do desafio estiver disponível, produza o FDD a partir dos
documentos e das evidências existentes. Faça primeiro o levantamento abaixo e
use somente informações verificáveis nas fontes.

### 1. Ler o contrato e as regras do repositório

Leia:

- CHALLENGE.md, principalmente os requisitos do FDD, os critérios de aceite,
  a regra de rastreabilidade e a restrição de não alterar o código;
- docs/advise.md, se existir, como orientação operacional do desafio.

### 2. Montar o mapa de evidências antes de redigir

Leia docs/mapping.md e confirme as evidências na fonte primária
TRANSCRICAO.md. Organize os itens, no mínimo, nestes grupos:

- requisitos funcionais;
- requisitos não funcionais e restrições;
- decisões fechadas;
- alternativas descartadas;
- questões abertas ou adiadas;
- itens fora de escopo;
- riscos e mitigações;
- referências ao código existente.

Para a transcrição, preserve timestamp e participante no formato
[hh:mm] Nome. Para o código, preserve o caminho real e, quando possível,
linhas ou símbolos relevantes.

### 3. Inspecionar somente o código necessário, mas verificar cada caminho

Use rg --files para confirmar que os caminhos existem e leia os arquivos
referenciados pela matriz. No cenário deste desafio, dê atenção especial a:

- src/modules/orders/order.service.ts, especialmente changeStatus e a
  transação existente;
- src/modules/orders/order.status.ts, para os status e transições válidos;
- src/modules/orders/order.routes.ts e src/modules/orders/order.schemas.ts,
  para o padrão de rotas e validação;
- src/modules/orders/order.repository.ts, como referência de acesso a dados;
- src/app.ts e src/routes/index.ts, para composição e registro de módulos;
- src/middlewares/auth.middleware.ts, validate.middleware.ts e
  error.middleware.ts;
- src/shared/errors/app-error.ts, http-errors.ts e src/shared/logger/index.ts;
- src/middlewares/request-logger.middleware.ts;
- src/config/database.ts e src/server.ts, para Prisma, bootstrap e processos;
- prisma/schema.prisma, package.json e testes relacionados.

Essa lista é um ponto de partida, não uma lista para copiar sem verificar.
Adapte-a ao código real. Não diga que src/modules/webhooks/, src/worker.ts,
webhook_outbox ou webhook_dead_letter já existem se a inspeção mostrar que são
artefatos propostos.

### 4. Ler os documentos derivados para consolidar, não para inventar

Leia os ADRs relacionados e o RFC para entender as decisões já formalizadas,
alternativas, questões abertas e links que o FDD deve respeitar. Consulte o
PRD para alinhar escopo e termos. Consulte docs/TRACKER.md para não perder
rastreabilidade.

Se houver conflito, volte à transcrição e ao código. O FDD não deve resolver
silenciosamente uma divergência entre documentos.

### 5. Produzir uma matriz de implementação

Antes do texto final, faça uma matriz interna com estas colunas:

| Item | Estado | Implementação no FDD | Fonte | Validação |
| --- | --- | --- | --- | --- |
| requisito, decisão ou comportamento | fechado, proposto, aberto ou fora de escopo | seção e subseção | timestamp ou caminho real | teste, inspeção ou critério |

Use essa matriz para descobrir lacunas, remover conteúdo sem fonte e preparar
as linhas correspondentes do Tracker. O FDD pode incluir referências de fonte
nas tabelas e nos critérios de aceite sem transformar-se em uma cópia do
Tracker.

## Conteúdo obrigatório do FDD para este desafio

O documento deve conter as onze seções numeradas abaixo. Não omita uma seção
exigida para esconder uma lacuna. Se um detalhe ainda não foi decidido, escreva
"questão aberta" ou "proposta a confirmar" e cite sua origem.

### 1. Contexto e motivação técnica

Explique o problema técnico, a relação com o OMS, os atores, os limites do
escopo e a separação entre webhook outbound e inbound. Não repita a narrativa
de produto além do necessário para justificar a implementação.

### 2. Objetivos técnicos

Liste resultados mensuráveis e invariantes. Inclua, quando sustentado pelas
fontes, a latência percebida aceita, a consistência entre mudança de status e
registro do evento e a semântica de entrega.

### 3. Escopo e exclusões

Separe incluído, fora de escopo, adiado e hipótese. Preserve as exclusões da
reunião, como inbound, dashboard visual, e-mail de fallback, arquivamento ou
ordering global quando essas exclusões forem sustentadas pela transcrição.

### 4. Fluxos detalhados e diagramas

Detalhe os quatro fluxos que o desafio exige:

1. **Criação do evento na outbox:** mudança de status, seleção dos webhooks
   interessados, snapshot do payload, inserção na mesma transação e rollback
   se o enqueue falhar.
2. **Processamento pelo worker:** processo separado, polling, lote, ordem de
   leitura, claim ou lock, envio e marcação do resultado. Diferencie o que foi
   decidido do que ainda precisa ser escolhido.
3. **Retry:** timeout, falha HTTP ou outra condição de falha, contagem de
   tentativas, backoff, próxima tentativa e encerramento.
4. **DLQ e replay:** persistência da falha permanente, dados preservados,
   autorização administrativa, auditoria e retorno do item à outbox.

Inclua diagrama de sequência, fluxo ou estados quando ele reduzir ambiguidade.
O diagrama deve concordar com os contratos e com a matriz de erros.

### 5. Contratos públicos

Descreva contratos da API de configuração e operação e o contrato outbound de
entrega. Para cada contrato, inclua método, rota ou assinatura, autenticação,
request, response, headers, semântica dos status codes, validações, limites e
versionamento quando aplicável.

O FDD deste desafio deve conter pelo menos **quatro endpoints HTTP**, e cada
um deve ter:

- exemplo de request;
- exemplo de response;
- status codes de sucesso e erro;
- semântica dos headers.

Cubra os endpoints sustentados pela transcrição, incluindo o CRUD de
configuração, o histórico de deliveries e o replay da DLQ. Se a reunião não
tiver fechado a rota exata, marque a rota como proposta ou questão aberta em
vez de apresentá-la como decisão histórica. Não esqueça o contrato da chamada
outbound, com payload JSON, X-Event-Id, X-Signature, X-Timestamp,
X-Webhook-Id e Content-Type quando sustentados pelas fontes.

### 6. Erros, exceções e fallback

Inclua uma matriz com condição, código, status HTTP, tratamento, retry ou não,
efeito sobre a outbox/DLQ e observações. No desafio, os códigos do módulo
devem usar o prefixo WEBHOOK_, por exemplo os códigos discutidos na reunião.
Não misture códigos genéricos sem explicar sua relação com o contrato.

Descreva timeouts, retries, backoff, circuit breaker se houver evidência,
fallback e invariantes. Deixe explícito que uma falha de entrega externa não
deve desfazer uma mudança de status já confirmada, enquanto uma falha ao
registrar a outbox deve impedir o commit da transação que a originaria.

### 7. Observabilidade

Especifique métricas, logs estruturados e tracing suficientes para verificar o
comportamento do fluxo. Inclua nomes ou dimensões, eventos de sucesso e falha,
correlation IDs, cardinalidade, redaction de secrets e dados sensíveis,
spans, amostragem, dashboards e alertas. A seção deve citar explicitamente
**métricas, logs e tracing**, pois esses três itens são critérios de aceite do
FDD.

### 8. Dependências e compatibilidade

Liste banco, Prisma, runtime, cliente HTTP, autenticação, configuração,
processos de execução e dependências de infraestrutura. Separe o que já existe
do que será criado. Registre garantias e limites de compatibilidade, incluindo
versões, migrações e coexistência entre API e worker quando houver evidência.

### 9. Integração com o sistema existente

Esta seção é obrigatória e deve ter esse título. Nomeie pelo menos **quatro
caminhos de arquivo reais** do código base e descreva como a feature se
integrará a cada um. Exemplos de pontos que podem ser relevantes, se ainda
existirem após a verificação:

- src/modules/orders/order.service.ts, para inserir o evento na transação de
  changeStatus;
- src/modules/orders/order.status.ts, para reutilizar status e transições;
- src/middlewares/auth.middleware.ts, para authenticate e
  requireRole('ADMIN');
- src/shared/errors/app-error.ts e src/middlewares/error.middleware.ts,
  para erros e resposta centralizada;
- src/shared/logger/index.ts, para Pino e redaction;
- src/routes/index.ts e src/app.ts, para registrar o módulo;
- src/config/database.ts e src/server.ts, para instâncias Prisma e processos
  separados.

Não conte arquivos futuros como arquivos reais. Caminhos como
src/modules/webhooks/ e src/worker.ts devem ser marcados como artefatos
propostos, ainda inexistentes, quando essa for a situação encontrada.

### 10. Critérios de aceite técnicos

Transforme requisitos em critérios objetivos e verificáveis. Prefira o formato
Dado / Quando / Então ou uma tabela com ID, condição, ação, resultado esperado,
validação e fonte. Cubra, quando houver evidência:

- persistência atômica do evento com a mudança de status e rollback do conjunto;
- filtro de status e snapshot do payload na inserção;
- polling do worker, batch, ordenação e limite de concorrência acordados;
- timeout, cinco tentativas, intervalos de backoff e ida para DLQ;
- replay administrativo, role ADMIN e auditoria do executor;
- HMAC-SHA256, secret por endpoint, rotação e janela de 24 horas;
- HTTPS, limite de 64 KB, headers e payload sem items quando aplicável;
- semântica at-least-once e deduplicação por X-Event-Id no consumidor;
- pelo menos quatro endpoints com contratos testáveis;
- códigos WEBHOOK_*, métricas, logs, tracing e integração com o código real.

Não crie metas quantitativas que não estejam na transcrição, no código ou no
contrato do desafio. Um critério não decidido deve ser marcado como questão
aberta, não como aceite fechado.

### 11. Riscos e mitigações

Liste riscos técnicos com probabilidade, impacto, mitigação e contingência.
Inclua, quando sustentado, indisponibilidade de clientes, vazamento de secret,
crescimento da outbox/DLQ, duplicidade, ordenação e escala horizontal. Não
converta uma mitigação proposta em garantia já implementada.

## Checklist final do FDD

Execute esta checklist antes de considerar docs/FDD.md concluído:

- [ ] O arquivo existe em Markdown no caminho docs/FDD.md.
- [ ] As onze seções obrigatórias estão presentes e numeradas.
- [ ] O FDD está no nível de implementação e não duplica o RFC, PRD ou ADRs.
- [ ] O escopo separa incluído, fora de escopo, adiado e hipótese.
- [ ] Os fluxos de outbox, worker, retry e DLQ estão detalhados e coerentes
      entre si.
- [ ] O fluxo de outbox deixa explícita a transação compartilhada com a mudança
      de status e o rollback em caso de falha no enqueue.
- [ ] Há pelo menos quatro endpoints HTTP documentados, cada um com request,
      response e status codes.
- [ ] Headers e semântica do contrato outbound estão descritos, incluindo os
      headers exigidos pelas fontes.
- [ ] A matriz de erros usa códigos com prefixo WEBHOOK_ e define o tratamento
      de cada condição.
- [ ] Timeout, retries, backoff, fallback e DLQ têm comportamento verificável.
- [ ] A seção de observabilidade cita métricas, logs e tracing e evita expor
      secrets ou dados sensíveis.
- [ ] A seção Integração com o sistema existente cita pelo menos quatro
      caminhos reais, verificados no repositório.
- [ ] Artefatos futuros, como módulo de webhooks, worker ou tabelas novas, não
      são apresentados como existentes.
- [ ] Os critérios de aceite são objetivos, testáveis, cobrem funcionalidade,
      segurança, resiliência, integração e observabilidade, e têm fonte.
- [ ] Toda decisão, requisito, restrição, exclusão e questão aberta relevante
      pode ser localizada na transcrição ou no código.
- [ ] O conteúdo não contradiz o RFC, os ADRs, o docs/mapping.md ou o
      docs/TRACKER.md.
- [ ] Os caminhos citados foram conferidos com rg --files ou inspeção
      equivalente.
- [ ] Nenhum arquivo de src/, prisma/, tests/ ou configuração foi alterado
      durante a produção documental.
- [ ] Os itens novos ou alterados têm linha correspondente no Tracker quando a
      entrega do desafio estiver sendo montada.

Se algum item falhar, corrija o FDD ou registre a lacuna como questão aberta.
Não marque a checklist como concluída por inferência.

## Estrutura de dados para exportação JSON

Durante a produção, mantenha internamente os dados abaixo. Só exporte JSON se
o usuário pedir. O JSON deve ser válido, ter chaves em inglês, conteúdo em
português e não conter campos vazios. A exportação não substitui o Markdown
obrigatório.

~~~json
{
  "meta": {
    "product_or_system": "",
    "feature_name": "",
    "fdd_owner": "",
    "version": "",
    "date": "YYYY-MM-DD"
  },
  "context": {
    "technical_motivation": "",
    "fit_with_existing_system": "",
    "actors": [],
    "assumptions": [],
    "constraints": [],
    "scope_status": []
  },
  "technical_objectives": [
    {
      "objective": "",
      "measure_or_invariant": "",
      "source": ""
    }
  ],
  "scope": {
    "included": [],
    "excluded": [],
    "deferred": [],
    "hypotheses": []
  },
  "detailed_flows": {
    "outbox_creation": [],
    "worker_processing": [],
    "retry": [],
    "dead_letter_and_replay": [],
    "alternative_flows": [],
    "diagrams": []
  },
  "public_contracts": [
    {
      "name": "",
      "kind": "http_endpoint|outbound_webhook|function|method|queue|stream|sdk",
      "signature_or_route": "",
      "method": "",
      "authentication": "",
      "request_example": {},
      "response_example": {},
      "headers_semantics": [],
      "status_semantics": [],
      "validation_and_limits": [],
      "versioning": "",
      "source": ""
    }
  ],
  "errors_exceptions_fallback": {
    "error_matrix": [
      {
        "condition": "",
        "code": "WEBHOOK_",
        "http_status": "",
        "treatment": "",
        "retry_or_dead_letter": "",
        "source": ""
      }
    ],
    "resilience_strategies": [],
    "fallback_policy": "",
    "invariants": []
  },
  "observability": {
    "metrics": [],
    "logs": {
      "format": "",
      "fields": [],
      "redactions": []
    },
    "tracing": {
      "spans": [],
      "sampling": ""
    },
    "dashboards_alerts": []
  },
  "dependencies_compatibility": {
    "existing_dependencies": [],
    "new_dependencies": [],
    "compatibility_guarantees": [],
    "open_questions": []
  },
  "existing_system_integration": [
    {
      "path": "",
      "symbol_or_area": "",
      "integration": "",
      "path_status": "existing|proposed",
      "source": ""
    }
  ],
  "acceptance_criteria": [
    {
      "id": "FDD-CA-01",
      "given": "",
      "when": "",
      "then": "",
      "validation": "",
      "source": ""
    }
  ],
  "risks": [
    {
      "risk": "",
      "probability": "low|medium|high",
      "impact": "",
      "mitigation": [],
      "contingency_plan": "",
      "source": ""
    }
  ],
  "traceability": [
    {
      "item_id": "",
      "document_section": "",
      "source_type": "TRANSCRICAO|CODIGO|CHALLENGE|DOCUMENTO_DERIVADO",
      "location": ""
    }
  ]
}
~~~

## Modelo de saída

Use este modelo para docs/FDD.md. Preserve os onze títulos numerados e não
preencha lacunas com conteúdo fictício. Inclua fontes em pontos críticos ou
garanta que o item esteja representado no Tracker.

~~~~markdown
# FDD: Sistema de Webhooks de Notificação de Pedidos

Versão: [versão]
Data: [data ou não informada na fonte]
Responsável: [responsável técnico ou não informado]

---

## 1. Contexto e motivação técnica

[Problema técnico, encaixe no OMS, atores e limites.]

## 2. Objetivos técnicos

- [Objetivo com medida ou invariante.]

## 3. Escopo e exclusões

### Incluído

- [Item com fonte.]

### Fora de escopo

- [Item explicitamente descartado ou adiado, com fonte.]

### Questões abertas e hipóteses

- [Item não decidido ou hipótese claramente identificada.]

## 4. Fluxos detalhados e diagramas

### 4.1 Criação do evento na outbox

[Passos, transação, filtro, snapshot e rollback.]

### 4.2 Processamento pelo worker

[Processo, polling, batch, claim ou lock e marcação.]

### 4.3 Retry e backoff

[Falhas, tentativas, intervalos e encerramento.]

### 4.4 DLQ e replay

[Persistência, autorização, auditoria e reprocessamento.]

### 4.5 Diagramas

[Diagrama de sequência, fluxo ou estados, se necessário.]

## 5. Contratos públicos

### 5.1 [Contrato ou endpoint]

- Tipo: [endpoint HTTP ou outbound webhook]
- Rota: [rota]
- Autenticação: [detalhes]
- Headers: [semântica]
- Status codes: [semântica]

#### Request

~~~json
{}
~~~

#### Response

~~~json
{}
~~~

## 6. Erros, exceções e fallback

| Condição | Código WEBHOOK_* | Status | Tratamento | Retry/DLQ | Fonte |
| --- | --- | --- | --- | --- | --- |
| [condição] | [código] | [status] | [tratamento] | [política] | [fonte] |

## 7. Observabilidade

### Métricas

- [métrica]

### Logs

- [campos, correlação e redaction]

### Tracing

- [spans, propagação e amostragem]

### Dashboards e alertas

- [painel ou alerta]

## 8. Dependências e compatibilidade

[Dependências existentes, novas e garantias.]

## 9. Integração com o sistema existente

| Caminho real | Símbolo ou área | Integração | Status do caminho | Fonte |
| --- | --- | --- | --- | --- |
| [src/...] | [símbolo] | [como integrar] | existente | [fonte] |

## 10. Critérios de aceite técnicos

| ID | Dado | Quando | Então | Validação | Fonte |
| --- | --- | --- | --- | --- | --- |
| FDD-CA-01 | [pré-condição] | [ação] | [resultado] | [teste/inspeção] | [fonte] |

## 11. Riscos e mitigação

### [Risco]

- Probabilidade: [baixa, média ou alta]
- Impacto: [impacto]
- Mitigação: [ação]
- Contingência: [plano]
- Fonte: [fonte]
~~~~

Após gerar o FDD, pergunte se o usuário deseja uma exportação JSON conforme o
esquema acima. Não substitua nem reescreva o Markdown para produzir o JSON.

## Mensagem inicial

Use esta mensagem quando iniciar a atividade:

> Olá! Vou cruzar o CHALLENGE.md, a transcrição, o mapa de evidências, os documentos derivados e os caminhos reais do código. Em seguida, vou consolidar o FDD com foco em implementação, mantendo explícitas as hipóteses e questões abertas. No final, valido a checklist do desafio e posso exportar os dados em JSON.
