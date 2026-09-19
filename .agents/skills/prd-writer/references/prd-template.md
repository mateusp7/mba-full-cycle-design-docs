# Modelo mínimo de saída do PRD

Use este arquivo como modelo estrutural ao criar ou revisar `docs/PRD.md`.
Substitua os marcadores por conteúdo rastreável; não mantenha placeholders no
PRD final.

```markdown
# PRD — Sistema de Webhooks de Notificação de Pedidos

## 1. Resumo e contexto da feature

[O que é a feature, contexto do OMS, motivação e limites.]

## 2. Problema e motivação

[Dores atuais, impacto e relação com o escopo.] 

## 3. Público-alvo e cenários de uso

[Atores confirmados e cenários concretos.] 

## 4. Objetivos e métricas de sucesso

| ID | Objetivo | Métrica e meta | Condição | Fonte |
| --- | --- | --- | --- | --- |
| PRD-OBJ-01 | [objetivo] | [métrica e meta quantitativa] | [condição] | [fonte] |

## 5. Escopo

### Incluído

- [capacidade incluída e fonte]

### Fora de escopo

- **PRD-OOS-01:** [item explicitamente descartado ou adiado] — [fonte]
- **PRD-OOS-02:** [item explicitamente descartado ou adiado] — [fonte]

### Questões abertas ou hipóteses

- [item não decidido, claramente identificado como aberto ou hipótese]

## 6. Requisitos funcionais

### PRD-FR-01 — [título]

- Ator: [ator ou sistema]
- Requisito: [comportamento observável e regra de negócio]
- Prioridade: [quando houver fonte]
- Dependências: [IDs PRD-DEP-* ou nenhuma]
- Fonte: [timestamp e participante]
- Critérios: [IDs PRD-CA-*]

[Repita para pelo menos oito requisitos funcionais distintos e discutidos na
reunião.]

## 7. Requisitos não funcionais

| ID | Requisito | Medida ou condição verificável | Fonte |
| --- | --- | --- | --- |
| PRD-NFR-01 | [restrição ou qualidade] | [condição] | [fonte] |

## 8. Decisões e trade-offs principais

| ID | Decisão | Alternativa e trade-off | Estado | Fonte |
| --- | --- | --- | --- | --- |
| PRD-DEC-01 | [decisão fechada] | [alternativa e consequência] | [fechada ou aberta] | [fonte] |

## 9. Dependências

| ID | Dependência | Requisitos afetados | Impacto | Fonte |
| --- | --- | --- | --- | --- |
| PRD-DEP-01 | [dependência existente ou proposta] | [IDs] | [impacto] | [fonte] |

## 10. Riscos e mitigação

| ID | Risco | Probabilidade | Impacto | Mitigação | Fonte |
| --- | --- | --- | --- | --- | --- |
| PRD-RISK-01 | [risco] | [baixa/média/alta e se é avaliação] | [impacto] | [mitigação] | [fonte] |
| PRD-RISK-02 | [risco] | [baixa/média/alta e se é avaliação] | [impacto] | [mitigação] | [fonte] |

## 11. Critérios de aceitação

### PRD-FR-01 — [título]

- **PRD-CA-01:** Dado [pré-condição], quando [ação], então [resultado observável].
- **PRD-CA-02:** Dado [condição de falha], quando [ação], então [erro ou comportamento esperado].

[Cada requisito deve ter pelo menos um critério verificável; cubra integrações
entre requisitos quando houver dependência de dados ou fluxo.]

## 12. Estratégia de testes e validação

[Como validar os critérios funcionais e não funcionais em alto nível, sem
duplicar os detalhes do FDD.]
```
