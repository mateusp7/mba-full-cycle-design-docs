# AGENTS.md — Guia de produção e manutenção documental

Este arquivo orienta o Agente (LLM) a revisar ou
modificar a documentação deste desafio. O foco é exclusivamente o conteúdo
documental em `docs/` e o registro do processo no `README.md`. Não é um guia de
implementação, arquitetura, estrutura do repositório ou execução de scripts.

## Objetivo

Manter um pacote de design docs coerente, acionável e auditável, atendendo ao
`CHALLENGE.md` e preservando a rastreabilidade de cada requisito, decisão,
restrição, trade-off e questão em aberto.

O `README.md` registra como o trabalho foi produzido. Os documentos em `docs/`
registram o resultado do trabalho. O `AGENTS.md` registra as regras para manter
ambos consistentes.

## Regras obrigatórias

1. **Toda afirmação precisa de origem.** Use a `TRANSCRICAO.md` ou uma
   evidência verificável do sistema existente. Se não for possível apontar a
   origem, remova a afirmação ou marque-a claramente como hipótese/questão em
   aberto; nunca preencha lacunas com uma decisão inventada.
2. **Diferencie estados da informação.** Não trate como decisão fechada algo
   que foi apenas sugerido, adiado, descartado ou inferido. Preserve a
   distinção entre requisito, restrição, decisão, alternativa, questão aberta e
   item fora de escopo.
3. **Respeite a altura de cada documento.** O PRD explica o problema e o que
   será entregue; o RFC propõe a solução e abre pontos para revisão; os ADRs
   registram decisões isoladas; o FDD detalha como construir; o Tracker aponta
   a origem. Conteúdo duplicado entre esses níveis deve ser reduzido ou
   substituído por referência ao documento responsável.
4. **Atualize as referências cruzadas.** Ao criar ou modificar uma decisão,
   requisito, alternativa, exclusão ou questão aberta, verifique o `mapping`,
   o `TRACKER.md`, os links entre documentos e o impacto nos demais artefatos.
5. **Registre toda iteração corretiva real.** Quando algo já criado — documento,
   skill, prompt ou outro artefato do processo — precisar ser modificado porque
   não atendeu ou deixou de atender às expectativas do usuário ou aos critérios
   do desafio, a mudança deve ser registrada na seção **Iterações e ajustes**
   do `README.md`. O registro deve explicar o estado anterior, o problema
   identificado, o ajuste realizado e o resultado obtido. Não invente
   histórico: a seção só deve ser criada quando houver uma iteração real para
   relatar.
6. **Trate o `CHALLENGE.md` como contrato de aceite.** Uma preferência de
   redação nunca pode remover uma seção, cobertura, quantidade, formato ou
   critério exigido pelo desafio.

## Papel de cada documento

| Documento | Deve responder | Não deve fazer |
| --- | --- | --- |
| `docs/PRD.md` | Por que a feature existe, para quem, qual o escopo, objetivos, métricas e critérios de aceitação. | Descrever a implementação em profundidade ou registrar uma decisão arquitetural isolada. |
| `docs/RFC.md` | O que está sendo proposto, por quê, quais alternativas foram descartadas e o que ainda está em aberto. | Repetir os contratos e fluxos detalhados do FDD. |
| `docs/adrs/ADR-NNN-*.md` | Por que uma decisão foi tomada, quais alternativas existiam e quais são suas consequências. | Ser um catálogo genérico de requisitos ou uma cópia do RFC. |
| `docs/FDD.md` | Como a feature deve ser construída: fluxos, contratos, erros, resiliência, observabilidade e integração. | Introduzir decisões sem origem ou substituir os ADRs como registro de contexto decisório. |
| `docs/TRACKER.md` | De onde veio cada item relevante dos documentos. | Servir como lista de opiniões, hipóteses sem fonte ou resumo solto do projeto. |
| `docs/mapping.md` | Organizar as evidências antes e durante a produção dos documentos. | Ser usado para validar uma afirmação que não possui fonte primária. |
| `README.md` | Explicar a jornada de produção, ferramentas, prompts, workflow, iterações reais e navegação. | Ser usado para registrar requisitos técnicos da feature. |

## Checklist de aceite por artefato

Use esta checklist sempre que um documento for criado, revisado ou considerado
pronto:

- **PRD:** manter todas as seções exigidas, pelo menos 8 requisitos funcionais,
  uma métrica com meta quantitativa, pelo menos 2 itens fora de escopo e pelo
  menos 2 riscos com probabilidade, impacto e mitigação.
- **RFC:** manter metadados com os participantes como revisores, proposta em
  nível arquitetural, pelo menos 2 alternativas reais descartadas com seus
  trade-offs, pelo menos 2 questões abertas e links para pelo menos 2 ADRs.
- **FDD:** manter os fluxos de outbox, worker, retry e DLQ; pelo menos 4
  endpoints com exemplos de request/response e status codes; matriz de erros
  com prefixo `WEBHOOK_`; resiliência; métricas, logs e tracing; e a seção de
  integração com pelo menos 4 referências reais ao sistema existente.
- **ADRs:** manter entre 5 e 8 arquivos separados no formato exigido; cada um
  com Status, Contexto, Decisão, Alternativas Consideradas e Consequências;
  cobrir pelo menos 5 das 6 decisões principais; e ter pelo menos uma
  referência explícita a uma evidência do sistema existente.
- **Tracker:** usar a tabela obrigatória, manter pelo menos 80% de cobertura
  dos itens identificáveis, pelo menos 70% das linhas com `TRANSCRICAO` e pelo
  menos 5 linhas com `CODIGO`, todas com localização concreta.
- **README:** manter as seis áreas obrigatórias — sobre o desafio, ferramentas
  de IA, workflow, prompts customizados, iterações e ajustes e navegação —,
  pelo menos 2 prompts em blocos de código e somente relatos verdadeiros do
  processo.

## Fluxo para criar ou modificar um documento

### 1. Classificar a mudança

Antes de editar, identifique:

- qual documento é dono do conteúdo;
- se a mudança é requisito, decisão, trade-off, alternativa, questão aberta,
  exclusão, evidência ou apenas edição editorial;
- quais fontes sustentam a mudança;
- quais documentos e links podem ser afetados.

Uma correção de ortografia ou formatação, sem relação com atendimento de
expectativas, não exige registro em **Iterações e ajustes**. Já a modificação
corretiva de qualquer artefato que já existia, motivada por uma insuficiência
identificada pelo usuário ou pela revisão contra o desafio, exige esse registro
no README. Uma criação de documento, skill ou prompt deve ser registrada no
workflow; ela só entra também em **Iterações e ajustes** quando corrigir algo
que já havia sido criado.

### 2. Validar a evidência

Consulte a fonte antes de redigir. Para informação da reunião, registre o
timestamp e o participante no formato `[hh:mm] Nome`. Para informação do
sistema existente, registre a referência verificável usada no documento e no
Tracker. Quando a fonte não fechar a questão, use uma formulação de incerteza
ou registre-a como questão aberta.

Não transforme uma sugestão em requisito, uma alternativa em decisão, uma
inferência em fato ou um item adiado em escopo atual.

### 3. Editar o documento responsável

Mantenha a estrutura exigida pelo `CHALLENGE.md`, a linguagem objetiva e o
nível de detalhe adequado ao documento. Ao criar um novo documento, inclua os
metadados e as seções obrigatórias desde a primeira versão, mesmo que algumas
partes ainda estejam marcadas como pendentes de evidência.

### 4. Atualizar rastreabilidade e referências

Depois da edição:

- atualize `docs/mapping.md` quando a matriz de evidências ganhar ou perder
  informação;
- adicione ou ajuste a linha correspondente no `docs/TRACKER.md` para todo
  item identificável relevante;
- revise links, IDs, nomes de ADRs e referências entre PRD, RFC, ADRs e FDD;
- procure contradições, duplicações e afirmações sem fonte.

O Tracker deve manter, no mínimo, 80% de cobertura dos itens identificáveis,
70% das linhas com fonte `TRANSCRICAO` e pelo menos 5 linhas com fonte
`CODIGO`, conforme os critérios do desafio. Cada linha deve usar a tabela
obrigatória e uma localização concreta.

### 5. Registrar a mudança no README

Toda mudança material deve ser refletida no `README.md`, no local adequado:

- nova ferramenta, skill ou plugin usado: atualizar **Ferramentas de IA
  utilizadas**;
- criação ou modificação de documento, skill, prompt ou etapa de trabalho:
  atualizar **Workflow adotado** com o que foi feito e sua finalidade;
- novo prompt relevante: adicionar em **Prompts customizados**, em bloco de
  código;
- modificação corretiva de algo que já existia por não atender às expectativas
  do usuário ou aos critérios do desafio: criar ou atualizar obrigatoriamente
  **Iterações e ajustes** com o artefato afetado, o estado anterior, o problema,
  a ação tomada, o resultado e a contagem real de iterações;
- novo artefato que o leitor precisa conhecer: atualizar **Como navegar a
  entrega**.

No caso específico da criação ou evolução de uma skill, registre pelo menos o
papel da skill em **Ferramentas de IA utilizadas** e a etapa em que ela foi
adotada em **Workflow adotado**. Se uma skill já criada precisar evoluir por
não atender às expectativas do usuário ou do desafio, o registro em
**Iterações e ajustes** é obrigatório, seguindo os mesmos campos: estado
anterior, problema, alteração e resultado.

No estado atual, a seção **Iterações e ajustes** ainda pode não existir. Não
crie um texto placeholder nem atribua problemas fictícios; adicione-a quando o
primeiro ajuste real ocorrer.

## Requisitos de qualidade por entrega

Antes de considerar uma alteração concluída, confirme:

- as seções obrigatórias do documento continuam presentes;
- o conteúdo está no nível correto e não duplica outro documento;
- cada requisito, decisão, restrição, alternativa e exclusão relevante tem
  origem identificável;
- o Tracker foi atualizado e continua atendendo às metas de cobertura;
- links internos, IDs e nomes dos arquivos estão corretos;
- os números e formatos exigidos pelo desafio continuam atendidos, incluindo
  a quantidade de ADRs, os endpoints e códigos de erro do FDD, as alternativas
  e questões abertas do RFC, e os prompts e ferramentas do README;
- nenhuma mudança transformou uma questão aberta ou item fora de escopo em
  requisito sem nova evidência;
- o README descreve o processo real e continua navegável.

## Critério de conclusão

Uma mudança documental só está concluída quando o documento alterado, suas
fontes, o Tracker, as referências cruzadas e o README contam a mesma história.
Se a informação não puder ser rastreada, a alteração deve ser corrigida,
explicitamente limitada ou removida antes de ser considerada pronta.
