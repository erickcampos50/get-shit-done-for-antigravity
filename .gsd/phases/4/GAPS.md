# Phase 4 — GAPS.md: Análise do Processo de Geração

**App gerado:** Helpdesk de TI (helpdesk-ti.yaml)
**Verificações estruturais:** 15/15 ✅

---

## O que Funcionou Bem

| Padrão | Evidência |
|--------|-----------|
| Ordenação de módulos por dependência | Solicitante → Tecnico → Chamado sem nenhuma ambiguidade |
| Regra Select vs String | `Category`, `Priority`, `Status`, `SatisfactionScore` identificados como Select naturalmente |
| Prefilters nas páginas de detalhe | 5 prefilters gerados corretamente (3 listados + 2 em `prefilter: ""` nas listas) |
| `blockid` vs `blockID` | Sem confusão — a distinção no template é clara |
| `xywh: [0,0,0,0]` nos blocos | Aplicado consistentemente; posições corretas no pageLayout |
| `visible: true/false` | Listas visíveis, detalhes invisíveis |
| handles de detalhe | `RecordPageForModule{Handle}` aplicado sem precisar de lembrança extra |
| Charts com campo Record como dimensão | `ChamadosPorTecnico` usando `TecnicoId` (campo Record) — funcionou corretamente |

---

## Gaps Encontrados

### Gap 1 — `selfID: 0` pode ser ambíguo **[Severidade: Alta]**
**Problema:** A skill instrui `selfID: 0` para páginas de lista de primeiro nível, mas não
esclarece de forma explícita que `0` significa "sem pai" e é EXCLUSIVO de páginas de lista.
Nas páginas de detalhe, `selfID` deve ser o handle da lista correspondente, nunca `0`.

Um usuário que não sabe disso pode usar `selfID: 0` em detalhes por seguir o padrão das listas.

**Impacto:** A página de detalhe não ficaria vinculada à lista pai — o breadcrumb e o botão
"voltar" quebrariam no Corteza.

---

### Gap 2 — Rating/avaliação não está na tabela de decisão de tipos **[Severidade: Média]**
**Problema:** `SatisfactionScore` foi corretamente identificado como `Select` ao rever a regra
"se opções fixas, use Select". Mas não existe nenhuma linha na tabela de decisão para o caso
"avaliação de 1 a N" que explicitamente aponte para Select.

Um usuário poderia instintivamente usar `Number` para uma escala de 1 a 5.

**Impacto:** YAML tecnicamente funcional, mas ergonomicamente inferior (editor numérico em vez
de dropdown com rótulos descritivos).

---

### Gap 3 — Checklist não verifica `selfID` nas páginas de detalhe **[Severidade: Média]**
**Problema:** Os 10 itens do checklist atual não incluem nenhum item sobre `selfID`.
Um gerador poderia passar o checklist e ainda ter `selfID: 0` em todas as páginas.

---

### Gap 4 — Chart com campo Record como dimensão não tem exemplo direto **[Severidade: Baixa]**
**Problema:** `ChamadosPorTecnico` usa `TecnicoId` (campo Record) como dimensão de agrupamento.
Isso funciona — o Corteza exibe o `labelField` do módulo referenciado. Mas a skill e o template
de chart não mencionam essa possibilidade explicitamente: "é possível agrupar por campo Record".

**Impacto:** Baixo — funciona sem dificuldade, mas um usuário pode hesitar se é suportado.

---

## Refinamentos Propostos

| # | Alvo | Mudança | Prioridade |
|---|------|---------|-----------|
| R1 | SKILL.md — Etapa 3 (páginas) | Adicionar bloco de alerta sobre `selfID: 0` × `selfID: handle` | Alta |
| R2 | SKILL.md — Checklist (Etapa 5) | Adicionar item 11: verificar selfID em detalhes | Alta |
| R3 | SKILL.md — Tabela de tipos (Etapa 2) | Adicionar linha para "Rating/Avaliação → Select" | Média |
| R4 | SKILL.md — template chart | Adicionar nota sobre usar campo Record como dimensão | Baixa |
