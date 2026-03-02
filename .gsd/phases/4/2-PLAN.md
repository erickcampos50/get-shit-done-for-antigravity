---
phase: 4
plan: 2
wave: 2
depends_on: [4.1]
files_modified:
  - .agent/skills/corteza-app-builder/SKILL.md
  - .gsd/phases/4/GAPS.md
autonomous: true
must_haves:
  truths:
    - "GAPS.md documenta gaps e problemas encontrados durante a geração do helpdesk"
    - "SKILL.md atualizado com refinamentos baseados nos gaps"
    - "Skill versão final pronta para uso no Claude Code"
  artifacts:
    - ".gsd/phases/4/GAPS.md"
    - ".agent/skills/corteza-app-builder/SKILL.md (refinado)"
---

# Plan 4.2: Refinamento da Skill Baseado em Validação

<objective>
Analisar o processo de geração do Helpdesk (Plan 4.1), identificar gaps ou fricções encontradas,
e atualizar a skill com melhorias concretas. O resultado é a versão final e refinada da skill
`corteza-app-builder`, pronta para uso produtivo no Claude Code.

Output: GAPS.md com análise crítica + SKILL.md atualizado.
</objective>

<context>
Load for context:
- .agent/skills/corteza-app-builder/SKILL.md  ← versão atual a ser refinada
- .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml  ← resultado do Plan 4.1
- .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml  ← referência da Phase 3
- .gsd/phases/1/REFERENCE.md  ← fonte de verdade dos padrões Corteza
</context>

<tasks>

<task type="auto">
  <name>Analisar gaps e escrever GAPS.md</name>
  <files>.gsd/phases/4/GAPS.md</files>
  <action>
    Comparar o processo de geração do helpdesk-ti.yaml com as instruções da SKILL.md e identificar:

    **Gap 1 — Preencha com análise real:**
    Verificar se o Modo Conversacional está suficientemente claro. Perguntar:
    - As instruções de "proposta proativa" são específicas o suficiente?
    - O template de módulo é fácil de seguir?
    - As perguntas direcionadas da Etapa 1 estão bem formuladas?

    **Gap 2 — Patterns que foram difíceis de aplicar:**
    - Houve dúvida sobre quando usar Select vs String em algum campo?
    - O prefilter ficou claro? (campo FK no filho, não no pai)
    - O padrão `RecordPageForModule{Handle}` ficou óbvio ou precisou de lembrança?
    - Os layouts de pageLayout ficaram intuitivos?

    **Gap 3 — Informações ausentes ou incompletas na skill:**
    - Algum padrão YAML precisou ser consultado no REFERENCE.md que deveria estar na skill?
    - A tabela de limitações do Corteza cobriu os casos encontrados?

    **Gap 4 — Validação do checklist:**
    - Os 10 itens do checklist foram suficientes?
    - Algum item novo poderia ser adicionado? (ex: verificar que `selfID: 0` só é válido para
      páginas de lista de primeiro nível)

    Criar `.gsd/phases/4/GAPS.md` com:
    - Seção "Gaps Encontrados" com cada problema identificado, nível de severidade (Alta/Média/Baixa)
    - Seção "O que Funcionou Bem" (padrões que ficaram claros e corretos)
    - Seção "Refinamentos Propostos" com lista de mudanças específicas na skill

    AVOID: Não escrever gaps vagos como "poderia ser melhor". Seja específico: qual instrução, qual campo, qual padrão.
  </action>
  <verify>grep -q "Gaps Encontrados" .gsd/phases/4/GAPS.md && grep -q "Refinamentos Propostos" .gsd/phases/4/GAPS.md</verify>
  <done>GAPS.md existe com pelo menos 2 gaps identificados, o que funcionou bem, e refinamentos propostos com ações específicas</done>
</task>

<task type="auto">
  <name>Aplicar refinamentos na SKILL.md</name>
  <files>.agent/skills/corteza-app-builder/SKILL.md</files>
  <action>
    Com base nos refinamentos listados no GAPS.md, aplicar melhorias na SKILL.md:

    **Refinamento obrigatório — clareza do selfID:**
    Adicionar uma nota na seção de páginas (Etapa 3) esclarecendo:
    - `selfID: 0` → apenas para páginas de lista de PRIMEIRO NÍVEL (raiz do menu)
    - `selfID: [HandleDaLista]` → para páginas de detalhe (aponta para a lista pai)
    - Nunca use UUID numérico em selfID

    **Refinamento obrigatório — item 11 no checklist:**
    Adicionar ao checklist da Etapa 5:
    - [ ] **11. selfID nos detalhes**: Todas as páginas de detalhe têm `selfID` apontando para
      o handle da sua lista (não para 0, não para UUID)?

    **Refinamento obrigatório — nota sobre SatisfactionScore:**
    Adicionar na tabela de decisão de tipos (Etapa 2) ou na seção de regras:
    - Rating/avaliação de satisfação (1-5 estrelas ou NPS) → usar `Select` com opções textuais,
      NÃO `Number` — porque o conjunto de valores é fixo e conhecido.

    **Refinamentos adicionais baseados no GAPS.md:**
    Aplicar os refinamentos adicionais identificados na tarefa anterior, se houver.

    Após as mudanças, verificar que o arquivo ainda tem todas as seções originais intactas:
    - Modo Conversacional
    - Etapas 1-5
    - Checklist (agora com 11 itens)
    - Limitações do Corteza
    - Exemplo Mínimo

    AVOID: Não remover nenhuma seção existente — apenas adicionar/ajustar.
    AVOID: Não mudar o formato geral da skill.
  </action>
  <verify>grep -q "selfID: 0" .agent/skills/corteza-app-builder/SKILL.md && grep -q "11\." .agent/skills/corteza-app-builder/SKILL.md && grep -q "Modo Conversacional" .agent/skills/corteza-app-builder/SKILL.md</verify>
  <done>SKILL.md atualizado com nota sobre selfID, item 11 no checklist, e nota sobre Rating/Select; todas as seções originais intactas</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `grep -q "Gaps Encontrados" .gsd/phases/4/GAPS.md`
- [ ] `grep -q "Refinamentos Propostos" .gsd/phases/4/GAPS.md`
- [ ] `grep -q "11\." .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Modo Conversacional" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "selfID" .agent/skills/corteza-app-builder/SKILL.md`
</verification>

<success_criteria>
- [ ] GAPS.md documenta o que funcionou + gaps encontrados + refinamentos propostos
- [ ] SKILL.md tem nota de selfID esclarecida
- [ ] SKILL.md tem 11 itens no checklist (item 11 = selfID nos detalhes)
- [ ] SKILL.md tem nota sobre Rating → Select
- [ ] Todas as seções originais da skill estão intactas
</success_criteria>
