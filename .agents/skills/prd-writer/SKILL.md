---
name: prd-writer
description: Produz ou revisa o PRD em português para o desafio de Webhooks de Notificação de Pedidos, usando o CHALLENGE.md, a TRANSCRICAO.md, o código existente e os documentos derivados como contexto, com rastreabilidade e checklist dos critérios de aceite.
---

# PRD Writer

## Objetivo e fronteira

Use esta skill para criar ou revisar docs/PRD.md no desafio de Webhooks de
Notificação de Pedidos. O PRD responde por que a feature existe e o que será
entregue: problema, público, escopo, requisitos, objetivos, métricas, riscos e
critérios de aceitação.

O PRD não substitui os demais documentos:

- o RFC apresenta a proposta arquitetural, as alternativas e as questões em
  aberto;
- os ADRs registram decisões arquiteturais isoladas e suas consequências;
- o FDD detalha fluxos, contratos, erros, resiliência, observabilidade e
  integração com o código;
- o Tracker registra a origem primária de cada item relevante.

Mantenha o nível de produto do PRD. Inclua detalhes técnicos somente quando
forem necessários para definir um requisito, uma restrição, uma dependência,
um risco ou um critério de aceitação. Não copie contratos e fluxos do FDD nem
transforme decisões de implementação em requisitos de produto.

Não altere src/, prisma/, tests/, configurações, TRANSCRICAO.md ou CHALLENGE.md.
A entrega desta skill é documental.

## Entradas

Do comando ou do usuário, use quando disponíveis:

- PROJECT_NAME: nome do produto ou da feature;
- OUTPUT_FOLDER: pasta de saída;
- PRD_PATH: caminho completo do PRD, normalmente docs/PRD.md;
- PRODUCT_DESCRIPTION: descrição fornecida pelo usuário.

O repositório é a fonte de contexto. Se os parâmetros não forem fornecidos,
assuma PRD_PATH=docs/PRD.md somente quando esse caminho fizer sentido no
repositório atual; não crie um caminho arbitrário.

## Evidência e classificação da informação

Antes de redigir, use esta ordem de autoridade:

1. CHALLENGE.md, para o formato da entrega e os critérios de aceite do
   desafio;
2. TRANSCRICAO.md, fonte primária para problema, atores, requisitos,
   restrições, decisões, alternativas, exclusões, métricas e questões abertas;
3. código existente, para comportamentos observáveis, integrações, nomes de
   módulos e caminhos reais;
4. docs/mapping.md, como índice de evidências que deve ser conferido na fonte
   original;
5. docs/RFC.md, docs/FDD.md, docs/adrs/, docs/TRACKER.md e um PRD existente,
   para coerência e para evitar duplicação. Esses documentos são derivados e
   não substituem a transcrição ou o código como fonte primária.

Classifique cada informação antes de usá-la:

- requisito ou restrição;
- comportamento observado no código;
- decisão fechada;
- alternativa descartada;
- item fora de escopo;
- questão aberta ou adiada;
- risco e mitigação discutidos;
- hipótese ou proposta do documento.

Não transforme uma sugestão em requisito, uma alternativa em decisão, uma
questão adiada em escopo atual ou uma inferência em fato. Se uma afirmação não
tiver origem verificável, remova-a ou marque-a explicitamente como hipótese,
proposta ou questão aberta. Toda informação da reunião deve preservar o
timestamp e o participante no formato [hh:mm] Nome; toda evidência de código
deve apontar para um caminho real, confirmado com rg --files ou inspeção
equivalente.

## Processo de trabalho

### 1. Descobrir o contexto mínimo necessário

Faça a leitura nesta sequência:

1. Leia CHALLENGE.md, principalmente o requisito de PRD, os critérios de
   aceite, as regras de rastreabilidade e a restrição de não alterar o código.
2. Leia docs/mapping.md, se existir, para localizar requisitos, exclusões,
   riscos, decisões e referências ao código. Confirme itens importantes em
   TRANSCRICAO.md ou no arquivo real.
3. Leia TRANSCRICAO.md e extraia os itens por estado: requisitos funcionais
   e não funcionais, decisões, alternativas, questões abertas, fora de escopo,
   riscos e métricas.
4. Confirme no código apenas os caminhos necessários para explicar contexto,
   dependências ou integração. Não apresente artefatos futuros como se já
   existissem.
5. Leia o RFC, FDD, ADRs e Tracker existentes para manter vocabulário,
   decisões e fronteiras consistentes. Como o desafio sugere produzir o PRD
   depois desses documentos, aproveite-os como consolidação, mas volte às
   fontes primárias quando houver conflito.

Não leia o repositório inteiro sem necessidade. Comece com rg --files e com
os caminhos concretos do mapping, dos ADRs e do FDD; só amplie a inspeção
quando uma afirmação depender dela.

### 2. Montar uma matriz interna de evidências

Antes do texto final, organize uma matriz interna com:

| Item | Estado | Seção do PRD | Fonte primária | ID documental | Validação |
| --- | --- | --- | --- | --- | --- |
| requisito, decisão, risco etc. | fechado, aberto, hipótese ou fora de escopo | seção alvo | timestamp ou caminho real | PRD-* | critério, revisão ou teste |

Use a matriz para verificar que:

- existem pelo menos oito requisitos funcionais realmente discutidos na
  reunião;
- há pelo menos um objetivo com métrica e meta quantitativa sustentadas por
  uma fonte. Limites fechados na reunião, como timeout, tentativas, polling,
  tamanho de payload ou janela de rotação, podem ser usados como metas
  operacionais quando não houver KPI de negócio explícito, desde que sejam
  descritos como tal;
- há pelo menos dois itens explicitamente descartados ou adiados em Fora de
  escopo;
- há pelo menos dois riscos com probabilidade, impacto e mitigação. Se a
  probabilidade não foi dita literalmente, rotule-a como avaliação qualitativa
  do documento baseada na evidência, e não como fato da reunião;
- cada requisito tem pelo menos um critério de aceitação verificável;
- nenhum item foi criado somente para atingir uma contagem.

### 3. Clarificação, quando necessária

Preserve a essência iterativa da criação de PRD, mas não conduza uma entrevista
obrigatória quando o repositório já contém evidências suficientes. Pergunte ao
usuário apenas quando uma lacuna impedir a definição do escopo ou quando ele
pedir explicitamente uma entrevista interativa. Caso contrário, registre a
lacuna como hipótese ou questão aberta e continue sem inventar uma resposta.

Se PRODUCT_DESCRIPTION for curto, use primeiro a documentação do desafio e as
fontes do repositório. Só peça contexto adicional quando nem essas fontes
permitirem produzir um PRD rastreável.

### 4. Redigir o PRD

Escreva o documento em português claro e objetivo. Use o nome original da
feature no título e IDs estáveis para permitir rastreabilidade:

- PRD-OBJ-* para objetivos;
- PRD-FR-* para requisitos funcionais;
- PRD-NFR-* para requisitos não funcionais;
- PRD-DEP-* para dependências;
- PRD-RISK-* para riscos;
- PRD-CA-* para critérios de aceitação;
- PRD-OOS-* para itens fora de escopo, quando houver necessidade de
  referência individual.

Preserve os IDs ao revisar a redação do mesmo item. Crie um novo ID somente
quando o item documental for realmente diferente.

## Estrutura obrigatória do PRD

O arquivo deve conter as doze seções abaixo, nesta ordem. Elas cobrem todas as
áreas exigidas no requisito de PRD do CHALLENGE.md; subseções são permitidas.
Não acrescente uma seção de validação, próximos passos, versão ou data apenas
por hábito.

### 1. Resumo e contexto da feature

Explique o que é o Sistema de Webhooks de Notificação de Pedidos, o contexto
do OMS, por que a feature é necessária e qual é seu limite. Diferencie o
problema de produto do desenho técnico. Cite a evidência quando a narrativa
depender da reunião ou do sistema existente.

### 2. Problema e motivação

Descreva as dores atuais, quem é afetado, o impacto e a motivação para a
mudança. Relacione cada problema a uma resposta do escopo, sem prometer
benefícios que não tenham base. Não confunda polling dos clientes, webhook
outbound, webhook inbound ou fallback de e-mail se a transcrição os tratar de
forma diferente.

### 3. Público-alvo e cenários de uso

Liste os atores realmente sustentados pelas fontes, por exemplo clientes
consumidores, operadores administrativos e o sistema de pedidos, apenas quando
confirmados. Para cada ator, descreva necessidades e cenários concretos. Use
user stories ou cenários Dado/Quando/Então como apoio, mas não force personas
ou uma quantidade fixa de histórias.

### 4. Objetivos e métricas de sucesso

Liste objetivos acionáveis e mensuráveis. Para cada objetivo, informe:

- ID PRD-OBJ-*;
- resultado esperado;
- métrica;
- meta quantitativa;
- condição e fonte de medição.

Não invente percentuais, SLAs ou metas comerciais. Quando a fonte só oferecer
um limite operacional, diferencie “meta/limite técnico fechado” de “KPI de
sucesso do produto”.

### 5. Escopo

Separe claramente:

- Incluído: capacidades que a reunião colocou no release;
- Fora de escopo: pelo menos dois itens explicitamente descartados ou
  adiados, cada um com fonte;
- Questões abertas ou hipóteses: itens que não podem ser tratados como
  compromisso do release.

Não coloque no escopo atual uma alternativa descartada ou uma decisão ainda não
tomada. O Fora de escopo deve ser explícito, não apenas implícito pela
ausência de uma funcionalidade.

### 6. Requisitos funcionais

Identifique no mínimo oito requisitos funcionais discutidos na reunião. Cada
item deve conter:

- ID PRD-FR-* e título curto;
- ator ou sistema responsável;
- comportamento observável e regra de negócio;
- prioridade, se houver evidência suficiente;
- dependências relevantes;
- fonte primária.

Formule requisitos testáveis, com verbos como “deve registrar”, “deve
notificar”, “deve permitir” ou “deve rejeitar”. Separe capacidades distintas;
não conte variações editoriais do mesmo requisito. Relacione cada requisito a
um ou mais PRD-CA-*.

### 7. Requisitos não funcionais

Registre somente restrições e qualidades sustentadas pela transcrição, pelo
código ou pelo contrato do desafio, como segurança, semântica de entrega,
limites, timeout, disponibilidade operacional, rastreabilidade e compatibilidade.
Use IDs PRD-NFR-*, indique a medida ou condição verificável e não repita a
matriz técnica completa que pertence ao FDD.

### 8. Decisões e trade-offs principais

Consolide em nível de produto as decisões fechadas que afetam o escopo ou a
experiência, por exemplo semântica de entrega, retry/DLQ, segurança ou
integração transacional. Para cada item, indique decisão, alternativa e
trade-off, estado e fonte. Linke ADRs existentes quando isso ajudar a
navegação, mas não use um ADR como fonte primária nem copie seu contexto
completo. Questões não decididas devem permanecer marcadas como abertas.

### 9. Dependências

Descreva dependências de produto, dados, processos, autenticação, integração e
infraestrutura. Separe dependências já existentes no código de artefatos
propostos. Para cada dependência, use PRD-DEP-*, indique qual requisito ela
afeta, o impacto se não estiver disponível e a fonte. Não transforme um caminho
futuro como src/modules/webhooks/ em arquivo existente.

### 10. Riscos e mitigação

Inclua pelo menos dois riscos reais. Para cada PRD-RISK-*, informe:

- descrição e causa;
- probabilidade: baixa, média ou alta, marcada como avaliação quando não
  estiver explícita na fonte;
- impacto;
- mitigação sustentada ou proposta claramente rotulada;
- contingência, se discutida;
- fonte primária.

Cubra riscos relevantes como indisponibilidade do cliente, duplicidade,
vazamento de secret, crescimento da outbox/DLQ ou ordenação somente quando
forem sustentados pelas evidências ou derivados explicitamente da proposta.

### 11. Critérios de aceitação

Organize os critérios por requisito funcional e use IDs PRD-CA-*. Cada critério
deve ser objetivo, verificável e cobrir sucesso e falha quando a falha for
relevante. Prefira:

~~~text
Dado [pré-condição], quando [ação], então [resultado observável].
~~~

Inclua critérios para requisitos funcionais e para requisitos não funcionais
que sejam essenciais ao aceite do produto. Verifique integrações entre
requisitos quando um requisito consome o resultado de outro. Não deixe um
requisito sem critério correspondente e não use “funciona corretamente” ou
“boa experiência” sem condição observável.

### 12. Estratégia de testes e validação

Explique como o produto será validado em alto nível: testes de aceitação por
requisito, cenários de sucesso e erro, integração entre mudança de status e
notificação, segurança, resiliência e observabilidade quando aplicáveis.
Relacione a estratégia aos IDs PRD-FR-*, PRD-NFR-* e PRD-CA-*. Deixe detalhes
de endpoints, schemas, matriz completa de erros, algoritmos e instrumentação
para o FDD.

## Rastreabilidade e documentos relacionados

Ao criar ou revisar docs/PRD.md no repositório do desafio:

1. Leia o PRD inteiro ao final e extraia seus itens atômicos: requisitos,
   restrições, decisões, trade-offs, exclusões, riscos e evidências de
   integração.
2. Crie ou atualize em docs/TRACKER.md uma linha para cada item documental
   relevante, preservando IDs estáveis. Use TRANSCRICAO com timestamp e
   participante ou CODIGO com caminho real; não use RFC, FDD, ADR ou Tracker
   como fonte primária.
3. Se uma nova evidência foi encontrada ou uma evidência deixou de sustentar um
   item, atualize docs/mapping.md no mesmo ciclo.
4. Remova do Tracker linhas de itens que deixaram de existir no PRD, mas
   preserve no mapping.md a evidência que ainda for relevante como hipótese,
   exclusão ou questão aberta.
5. Confira os links para ADRs e os IDs usados por RFC, FDD e demais documentos.
   Se houver conflito, volte à transcrição ou ao código e não resolva a
   divergência silenciosamente.
6. Quando a manutenção do pacote estiver em escopo, atualize o README.md na
   seção apropriada: workflow para uma nova etapa ou ferramenta e Iterações e
   ajustes somente quando uma correção real de um artefato existente tiver
   ocorrido.

## Checklist final dos critérios de aceite do PRD

Execute esta checklist internamente antes de salvar ou considerar a revisão
concluída. Ela valida os requisitos do CHALLENGE.md; não a inclua como uma
seção adicional do docs/PRD.md.

- [ ] docs/PRD.md existe, está em Markdown e contém o título da feature.
- [ ] As doze seções obrigatórias estão presentes, na ordem, sem substituir
      nenhuma por uma seção genérica.
- [ ] O documento responde claramente por que a feature existe e o que será
      entregue, sem assumir o nível de implementação do FDD.
- [ ] Há pelo menos 8 PRD-FR-* distintos, todos discutidos na reunião,
      observáveis e rastreáveis à TRANSCRICAO.md.
- [ ] Há pelo menos 1 PRD-OBJ-* com métrica e meta quantitativa, com fonte e
      condição de medição.
- [ ] Fora de escopo contém pelo menos 2 itens explicitamente descartados ou
      adiados na reunião, sem tratar questões abertas como decisões.
- [ ] Riscos e mitigação contém pelo menos 2 riscos e, para cada um,
      probabilidade, impacto e mitigação.
- [ ] A seção 11 organiza os PRD-CA-* por requisito funcional, usa condições
      verificáveis, cobre sucesso e falha quando aplicável e testa integrações
      entre requisitos quando houver dependência de dados ou fluxo.
- [ ] Cada requisito funcional tem pelo menos um PRD-CA-* correspondente; não
      há requisito sem critério nem critério sem requisito identificável.
- [ ] Requisitos não funcionais, dependências, decisões, trade-offs, riscos e
      exclusões têm origem identificável ou estão explicitamente marcados como
      proposta, hipótese ou questão aberta.
- [ ] Não há contradição entre escopo, requisitos, critérios de aceitação,
      RFC, ADRs, FDD, docs/mapping.md e docs/TRACKER.md.
- [ ] Nenhum item descartado, adiado ou fora de escopo aparece como requisito
      atual.
- [ ] Caminhos de código mencionados existem; artefatos futuros estão marcados
      como propostos.
- [ ] docs/TRACKER.md foi atualizado no mesmo ciclo e mantém a cobertura
      exigida pelo desafio, incluindo as fontes primárias dos itens do PRD.
- [ ] Nenhum arquivo de código, Prisma, teste, configuração, transcrição ou
      contrato do desafio foi alterado.

Se algum item falhar, corrija o PRD, o mapping ou o Tracker conforme a causa e
execute a checklist novamente. Não invente conteúdo para eliminar uma lacuna;
registre-a como hipótese ou questão aberta e informe-a ao usuário quando ela
impedir o aceite.

## Salvar e entregar

1. Salve em PRD_PATH.
2. Releia o arquivo salvo e confirme a presença das doze seções, dos oito
   requisitos funcionais mínimos, do objetivo quantificado, das exclusões, dos
   riscos e dos critérios de aceitação.
3. Confirme que o arquivo não contém a checklist interna, uma seção extra de
   validação, placeholders ou afirmações sem fonte.
4. Informe ao usuário o caminho exato do PRD e qualquer lacuna que permaneça
   explicitamente aberta.

## Modelo mínimo de saída

~~~markdown
# PRD — Sistema de Webhooks de Notificação de Pedidos

## 1. Resumo e contexto da feature

...

## 2. Problema e motivação

...

## 3. Público-alvo e cenários de uso

...

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica e meta | Condição | Fonte |
| --- | --- | --- | --- | --- |
| PRD-OBJ-01 | ... | ... | ... | [hh:mm] Nome |

## 5. Escopo

### Incluído

...

### Fora de escopo

- PRD-OOS-01: ... — Fonte: [hh:mm] Nome.
- PRD-OOS-02: ... — Fonte: [hh:mm] Nome.

## 6. Requisitos funcionais

### PRD-FR-01 — [título]

- Ator: ...
- Requisito: ...
- Prioridade: ...
- Fonte: [hh:mm] Nome
- Critérios: PRD-CA-01

## 7. Requisitos não funcionais

...

## 8. Decisões e trade-offs principais

...

## 9. Dependências

...

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação | Fonte |
| --- | --- | --- | --- | --- | --- |
| PRD-RISK-01 | ... | ... | ... | ... | [hh:mm] Nome |

## 11. Critérios de aceitação

### PRD-FR-01

- PRD-CA-01: Dado ..., quando ..., então ...

## 12. Estratégia de testes e validação

...
~~~
