---
phase: 3
plan: 1
wave: 1
depends_on: []
files_modified:
  - .agent/skills/corteza-app-builder/templates/module.yaml
  - .agent/skills/corteza-app-builder/templates/field-types.yaml
  - .agent/skills/corteza-app-builder/templates/page-list.yaml
  - .agent/skills/corteza-app-builder/templates/page-detail.yaml
  - .agent/skills/corteza-app-builder/templates/chart.yaml
  - .agent/skills/corteza-app-builder/templates/namespace.yaml
  - .agent/skills/corteza-app-builder/templates/README.md
autonomous: true
must_haves:
  truths:
    - "6 templates YAML criados com comentários inline explicativos"
    - "README.md documenta quando usar cada template"
  artifacts:
    - ".agent/skills/corteza-app-builder/templates/ (diretório com 7 arquivos)"
---

# Plan 3.1: Templates YAML Reutilizáveis

<objective>
Criar templates YAML comentados para os 5 padrões mais comuns no Corteza, organizados em
`.agent/skills/corteza-app-builder/templates/`. Cada template serve duas finalidades:
1. Ser colado diretamente pelo assistente quando gerar um app
2. Ensinar o padrão ao assistente com comentários inline

Output: 6 arquivos YAML + 1 README.md de índice.
</objective>

<context>
Load for context:
- .gsd/phases/1/REFERENCE.md  ← estruturas YAML com todos os detalhes
- .agent/skills/corteza-app-builder/SKILL.md  ← contexto geral da skill
</context>

<tasks>

<task type="auto">
  <name>Criar templates de módulo e tipos de campo</name>
  <files>
    .agent/skills/corteza-app-builder/templates/module.yaml
    .agent/skills/corteza-app-builder/templates/field-types.yaml
  </files>
  <action>
    Criar `.agent/skills/corteza-app-builder/templates/module.yaml`:
    Template de um módulo completo com comentários em cada linha explicando o que é e quando usar.
    Incluir:
    - Bloco `config:` completo (dal, privacy, discovery, recordrevisions, recorddedup) com comentários explicando que campos NÃO devem ser omitidos
    - `handle`, `name`, `namespaceID`, `meta` com exemplos e regras de nomenclatura
    - `moduleField` com 3 campos de exemplo: String simples, Select, Record

    Criar `.agent/skills/corteza-app-builder/templates/field-types.yaml`:
    Um arquivo com um campo de exemplo para cada tipo disponível (String, Number, Bool, Select, Record, File, Email, Url, DateTime, User), cada um com:
    - Comentário descrevendo o caso de uso ideal
    - Config mínima necessária (dal + privacy + expressions)
    - Todas as opções documentadas para o tipo
    - Nota sobre variações (ex: String tem simples/multiLine/richText)

    AVOID: Não omitir o bloco `config:` nos campos — o Corteza exige `dal: {encodingstrategy: null}` mesmo que vazio.
    AVOID: Não usar IDs numéricos UUID — usar handles ou 0.
  </action>
  <verify>grep -q "encodingstrategy" .agent/skills/corteza-app-builder/templates/module.yaml && grep -q "useRichTextEditor" .agent/skills/corteza-app-builder/templates/field-types.yaml</verify>
  <done>module.yaml existe com config completo; field-types.yaml existe com todos os 10 tipos documentados com comentários</done>
</task>

<task type="auto">
  <name>Criar templates de páginas, chart, namespace e README</name>
  <files>
    .agent/skills/corteza-app-builder/templates/page-list.yaml
    .agent/skills/corteza-app-builder/templates/page-detail.yaml
    .agent/skills/corteza-app-builder/templates/chart.yaml
    .agent/skills/corteza-app-builder/templates/namespace.yaml
    .agent/skills/corteza-app-builder/templates/README.md
  </files>
  <action>
    Criar `.agent/skills/corteza-app-builder/templates/page-list.yaml`:
    Template de página de lista com 2 blocos (RecordList + Chart opcional).
    Comentários explicando: moduleID deve ser 0, visible true, handle naming, blockid/blockID distinção,
    significado dos campos xywh no bloco vs no pageLayout.

    Criar `.agent/skills/corteza-app-builder/templates/page-detail.yaml`:
    Template de página de detalhe de registro com 2 blocos (Record principal + RecordList de filhos).
    Comentários destacando: moduleID = handle do módulo (não UUID), visible false,
    selfID = handle da lista pai, prefilter obrigatório no RecordList,
    pattern do handle: RecordPageForModule{Handle}.

    Criar `.agent/skills/corteza-app-builder/templates/chart.yaml`:
    4 templates de chart (um de cada tipo: bar, line, pie, doughnut) no mesmo arquivo,
    com comentários inline sobre quando usar cada tipo e as configurações-chave.
    Incluir exemplos de modifier temporal (MONTH) e aggregate (SUM).

    Criar `.agent/skills/corteza-app-builder/templates/namespace.yaml`:
    O esqueleto mínimo do arquivo de namespace completo, estruturado como:
    namespace → chart (vazio []) → module (um exemplo) → page (uma lista e uma de detalhe).
    Comentário destacando a ordem obrigatória de dependências.

    Criar `.agent/skills/corteza-app-builder/templates/README.md`:
    Índice dos templates com: nome do arquivo, o que é, quando usar, placeholder a substituir.
    Tabela simples de referência rápida.

    AVOID: Não omitir o `pageLayout` em nenhum template de página — é obrigatório no Corteza.
    AVOID: Não misturar `blockid` (minúsculo, no bloco) com `blockID` (maiúsculo, no pageLayout).
  </action>
  <verify>grep -q "pageLayout" .agent/skills/corteza-app-builder/templates/page-list.yaml && grep -q "prefilter" .agent/skills/corteza-app-builder/templates/page-detail.yaml && grep -q "doughnut" .agent/skills/corteza-app-builder/templates/chart.yaml</verify>
  <done>5 arquivos criados (page-list, page-detail, chart, namespace, README); page-detail tem prefilter; chart tem todos os 4 tipos</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `grep -q "encodingstrategy" .agent/skills/corteza-app-builder/templates/module.yaml`
- [ ] `grep -q "useRichTextEditor" .agent/skills/corteza-app-builder/templates/field-types.yaml`
- [ ] `grep -q "pageLayout" .agent/skills/corteza-app-builder/templates/page-list.yaml`
- [ ] `grep -q "prefilter" .agent/skills/corteza-app-builder/templates/page-detail.yaml`
- [ ] `grep -q "doughnut" .agent/skills/corteza-app-builder/templates/chart.yaml`
- [ ] `test -f .agent/skills/corteza-app-builder/templates/README.md`
</verification>

<success_criteria>
- [ ] 7 arquivos criados em `.agent/skills/corteza-app-builder/templates/`
- [ ] Todos os templates têm comentários inline explicativos
- [ ] module.yaml tem config completo (dal + privacy + discovery)
- [ ] field-types.yaml cobre todos os 10 tipos de campo
- [ ] page-detail.yaml tem prefilter documentado
- [ ] chart.yaml tem os 4 tipos de chart
- [ ] README.md serve como índice de referência rápida
</success_criteria>
