# Design Docs gerados com IA

## Sobre o desafio

O desafio consiste em transformar a transcrição de uma reunião técnica sobre um Sistema de Webhooks de Notificação de Pedidos em um conjunto de documentos de design. A tarefa envolve analisar a transcrição, entender o código existente e registrar requisitos, decisões técnicas, alternativas, questões em aberto e critérios necessários para orientar a implementação.

O trabalho também busca manter a rastreabilidade das informações, conectando cada item documentado à sua fonte na transcrição ou no código. Dessa forma, a IA é usada como apoio à análise e à produção dos documentos, enquanto a consistência e a origem das decisões permanecem verificáveis.

## Ferramentas de IA utilizadas

- **OpenAI Codex**: usado para ler o repositório, analisar a transcrição, identificar decisões técnicas, revisar a consistência dos documentos e criar documentos.
- **Skill Creator**: utilizada para a criação/ajustes das skills de `fdd-writer`, `prd-writer`, `adr-writer` e `rfc-writer`.
- **Skill customizada `adr-writer`**: usada para registrar as decisões arquiteturais, alternativas, trade-offs e sua rastreabilidade até as fontes do projeto.
- **Skill customizada `rfc-writer`**: usada para estruturar o RFC arquitetural com base na transcrição, nos ADRs e nos requisitos do desafio.
- **Skill customizada `fdd-writer`**: usada para conduzir a estruturação do FDD, cobrindo fluxos, contratos, resiliência, observabilidade e critérios de aceite.
- **Skill customizada `prd-writer`**: usada como base para organizar o PRD, seus requisitos, funcionalidades, dependências e critérios de aceitação.

## Workflow adotado

### Criação do arquivo mapping

Neste momento, foi adicionada a interação para criação do arquivo [`docs/mapping.md`](docs/mapping.md). O objetivo foi manter um histórico mapeado das evidências, decisões e referências do projeto quando novos chats fossem abertos para a geração dos demais documentos.

### Criação da primeira versão incremental do Tracker

Após a consolidação dos seis ADRs e do RFC, foi criada a primeira versão do [`docs/TRACKER.md`](docs/TRACKER.md). Essa versão registra os itens relevantes desses documentos, usando a transcrição como fonte das decisões e o código existente como fonte das evidências de integração. O Tracker será ampliado nas revisões do PRD e do FDD e passará por uma auditoria final de cobertura quando o pacote estiver estabilizado.

### Atualização incremental do AGENTS.md

Para operacionalizar a estratégia do Tracker, o [`AGENTS.md`](AGENTS.md) passou a exigir sua atualização no mesmo ciclo de trabalho em que cada documento é criado ou alterado. O guia agora diferencia o papel do `mapping.md` como inventário de evidências, o papel do Tracker como registro dos itens efetivamente documentados e a auditoria final de cobertura do pacote completo.

### Geração da skill de adr-writer

Após a criação do arquivo [`docs/mapping.md`](docs/mapping.md), foi iniciado o processo da criação da skill de geração de um adr, sendo adaptado ao cenário atual do projeto. Para esse caso, solicitei que a IA me gerasse um prompt para criação da skill, com base no meu prompt, para que ela cubra todas as necessidades da seção do documento de ADR.

> Visualizei uma oportunidade de modificação da skill, possibilitando a instancia de 6 subagents para as tarefas, fazendo com que o agente principal seja apenas o orquestrador, evitando, assim, um estouro de janela principal de contexto.

### Geração da skill de rfc-writer

Após a consolidação dos ADRs, foi criada a skill `rfc-writer` para estruturar o RFC arquitetural com base na `TRANSCRICAO.md`, nos ADRs e nos requisitos do `CHALLENGE.md`. A skill também orienta a separação entre o RFC, o FDD e o PRD, mantendo no RFC apenas a proposta arquitetural, as alternativas, as questões em aberto, os impactos e os riscos.

### Criação do AGENTS.md

Foi criado o [`AGENTS.md`](AGENTS.md) como guia de manutenção da entrega documental. Ele consolida as regras do `CHALLENGE.md` para criação e revisão dos arquivos em `docs/`, define a responsabilidade de cada documento, orienta a atualização do Tracker e estabelece quando uma mudança deve ser registrada no próprio README. O guia também preserva a regra de não inventar a seção `Iterações e ajustes` antes que exista uma iteração real para documentar.

### Geração da skill de fdd-writer

Após revisar os requisitos do FDD no `CHALLENGE.md`, a skill `fdd-writer` foi adaptada para trabalhar de forma orientada por evidências. O fluxo passou a priorizar a leitura da transcrição, do `docs/mapping.md`, dos ADRs, do Tracker e dos caminhos reais do código antes da consolidação do documento, diferenciando decisões fechadas, propostas, hipóteses, questões abertas e artefatos ainda inexistentes.

A revisão também incorporou as exigências específicas do desafio: os fluxos de outbox, worker, retry e DLQ; no mínimo quatro endpoints com exemplos de request, response e status codes; matriz de erros com prefixo `WEBHOOK_*`; observabilidade com métricas, logs e tracing; a seção obrigatória de integração com pelo menos quatro arquivos reais; e uma checklist final para validar os critérios de aceite do FDD.

### Consolidação do FDD

Com a skill `fdd-writer`, foi produzido o [docs/FDD.md](docs/FDD.md) a partir da transcrição, do `docs/mapping.md`, dos seis ADRs, do RFC e da inspeção dos caminhos reais em `src/`, `prisma/`, `tests/` e `package.json`. O documento separa decisões fechadas de propostas e questões abertas, sem alterar o código da aplicação.

No mesmo ciclo, o [docs/TRACKER.md](docs/TRACKER.md) recebeu linhas para os fluxos, contratos, erros, observabilidade, integração e critérios de aceite do FDD. As lacunas de claim/lock, retenção, rotas finais e indexação do retry permaneceram explicitamente abertas.

### Revisão da skill de prd-writer

A skill `prd-writer` foi revisada contra o requisito de PRD e os critérios de aceite do `CHALLENGE.md`. O novo fluxo prioriza a leitura de `TRANSCRICAO.md`, `docs/mapping.md`, código real e documentos derivados antes da redação, mantendo o PRD no nível de produto e a rastreabilidade no `docs/TRACKER.md`.

Também foram substituídas as regras genéricas de PRD em inglês e de nove seções por uma estrutura em português com as doze áreas exigidas pelo desafio, incluindo no mínimo oito requisitos funcionais, métricas quantitativas, fora de escopo, riscos estruturados, critérios de aceitação e estratégia de testes.

### Extração do modelo do PRD para uma referência

O modelo mínimo de saída foi movido para [`references/prd-template.md`](.agents/skills/prd-writer/references/prd-template.md), mantendo o `SKILL.md` concentrado no processo, nas regras de evidência e na validação. A skill passou a referenciar esse arquivo quando o PRD é criado ou revisado.

## Iterações e ajustes

### Ajuste do `AGENTS.md` para explicitar iterações corretivas

- **Estado anterior:** o `AGENTS.md` orientava a não inventar histórico, mas não deixava suficientemente explícito que a modificação de um artefato já criado por não atender às expectativas do usuário deveria ser registrada no README na seção de **Iterações e ajustes**.
- **Problema identificado:** essa redação poderia fazer com que uma correção solicitada pelo usuário fosse aplicada no documento sem registrar a iteração no processo.
- **Ajuste realizado:** o `AGENTS.md` passou a exigir o registro em **Iterações e ajustes** para modificações corretivas em documentos, skills, prompts ou outros artefatos já existentes, incluindo estado anterior, problema, ação, resultado e contagem de iterações.
- **Resultado:** futuras correções motivadas por expectativas não atendidas ou critérios do desafio terão seu histórico documentado de forma obrigatória.

### Remoção da entrevista técnica da skill de FDD

- **Estado anterior:** a skill `fdd-writer` incluía uma etapa de entrevista técnica e uma orientação redundante sobre a leitura de contexto já disponível ao agente.
- **Problema identificado:** a geração do FDD deveria ser determinística e baseada nos documentos existentes, sem conduzir uma entrevista ou solicitar confirmações durante a produção.
- **Ajuste realizado:** a etapa de entrevista foi removida, a skill passou a produzir o documento diretamente a partir das fontes existentes e a instrução redundante foi retirada.
- **Resultado:** o fluxo do FDD ficou exclusivamente orientado por evidências documentais e pelo código existente. Esta foi a primeira iteração corretiva da skill após sua criação.

### Revisão da skill `prd-writer` para atender ao `CHALLENGE.md`

- **Estado anterior:** a skill usava um modelo genérico de nove seções, exigia a produção do PRD em inglês e permitia inferir detalhes quando a resposta do usuário não os fornecia.
- **Problema identificado:** esse formato não cobria todas as áreas obrigatórias do PRD do desafio nem orientava adequadamente a leitura da transcrição, do mapping, do código e dos documentos derivados. Também não validava explicitamente os mínimos de oito requisitos funcionais, uma meta quantitativa, dois itens fora de escopo e dois riscos com probabilidade, impacto e mitigação.
- **Ajuste realizado:** a skill passou a operar em português e orientada por evidências, definiu as doze seções do PRD, preservou estados de decisão, adotou IDs `PRD-*`, instruiu a atualização do Tracker e incluiu uma checklist final interna alinhada aos critérios de aceite do desafio. A entrevista deixou de ser obrigatória quando o repositório já fornece contexto suficiente.
- **Resultado:** a geração do PRD ficou específica para o desafio, mais eficiente na descoberta das fontes e com validação explícita de completude e rastreabilidade. Contagem desta skill: 1 iteração corretiva registrada.

### Simplificação do contexto fixo e separação do modelo da skill `prd-writer`

- **Estado anterior:** a skill ainda mantinha uma seção de entradas com parâmetros já conhecidos no repositório, uma etapa de clarificação interativa e o modelo mínimo de saída dentro do `SKILL.md`.
- **Problema identificado:** essas instruções adicionavam decisões desnecessárias ao fluxo de criação do único arquivo esperado, `docs/PRD.md`, e deixavam o ponto de entrada mais extenso do que o necessário.
- **Ajuste realizado:** os valores do projeto, da pasta e do arquivo de saída foram fixados na própria skill; a etapa de clarificação foi removida; e o modelo foi extraído para `.agents/skills/prd-writer/references/prd-template.md`, com referência no `SKILL.md`.
- **Resultado:** a skill ficou focada exclusivamente na criação ou revisão documental do PRD, com contexto pré-preenchido e carregamento separado do modelo. Contagem desta skill: 2 iterações corretivas registradas.

## Prompts customizados

### Levantamento de evidências

```text
Leia TRANSCRICAO.md e o código existente. Crie docs/mapping.md
classificando requisitos funcionais, requisitos não funcionais,
decisões fechadas, alternativas descartadas, questões abertas,
itens fora de escopo e caminhos reais do código.

Toda informação deve ter origem identificável. Não invente
requisitos, decisões, restrições ou arquivos. Diferencie o que
existe no código, o que foi proposto na reunião e o que é apenas
inferência estrutural.
```

### Orquestração dos ADRs

```text
Ajuste a skill adr-writer para que, no modo pacote completo,
o agente principal analise as fontes uma vez, monte pacotes
compactos de evidências por decisão e delegue cada ADR a um
subagent quando houver suporte.

Reserve previamente os números e caminhos dos arquivos, limite
a concorrência, evite enviar TRANSCRICAO.md e docs/mapping.md
completos para cada subagent e faça a validação final no agente
principal. Cada subagent deve escrever apenas seu ADR e retornar
somente o caminho, o status, um resumo curto e eventuais
pendências.
```

## Como navegar a entrega

A ordem sugerida de leitura é:

1. [`README.md`](README.md): contexto do desafio e processo de produção.
2. [`AGENTS.md`](AGENTS.md): regras para manter a documentação e registrar o processo.
3. [`docs/mapping.md`](docs/mapping.md): matriz de evidências da transcrição e do código.
4. [`docs/PRD.md`](docs/PRD.md): problema, público, escopo e objetivos do produto.
5. [`docs/RFC.md`](docs/RFC.md): proposta técnica, alternativas e questões em aberto.
6. [`docs/adrs/`](docs/adrs/): decisões arquiteturais e seus trade-offs.
7. [`docs/FDD.md`](docs/FDD.md): fluxos, contratos e detalhes de implementação.
8. [`docs/TRACKER.md`](docs/TRACKER.md): rastreabilidade dos itens até suas fontes.
9. [`prd-template.md`](.agents/skills/prd-writer/references/prd-template.md): modelo estrutural usado pela skill de PRD.
