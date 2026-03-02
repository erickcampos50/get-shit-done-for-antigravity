---
phase: 2
plan: 2
wave: 2
depends_on: [2.1]
files_modified:
  - .agent/skills/corteza-app-builder/SKILL.md
autonomous: true
must_haves:
  truths:
    - "SKILL.md orienta design de páginas com tipos de bloco corretos"
    - "SKILL.md orienta design de charts com os 4 tipos disponíveis"
    - "SKILL.md instrui como gerar YAML completo e ordenado"
    - "SKILL.md inclui checklist de validação antes da geração"
  artifacts:
    - ".agent/skills/corteza-app-builder/SKILL.md (completo — todas as seções)"
---

# Plan 2.2: SKILL.md — Páginas, Charts, Geração YAML e Validação

<objective>
Completar `.agent/skills/corteza-app-builder/SKILL.md` com as etapas 3-5:
design de páginas, design de charts, geração do YAML e checklist de validação.

A skill deve ser completa o suficiente para que o assistente consiga guiar a
criação de um app Corteza do início ao fim sem precisar consultar documentação externa.

Output: SKILL.md completamente funcional e pronto para uso no Claude Code.
</objective>

<context>
Load for context:
- .gsd/SPEC.md
- .gsd/phases/1/REFERENCE.md  ← todas as estruturas YAML necessárias
- .agent/skills/corteza-app-builder/SKILL.md  ← continua do Plan 2.1
</context>

<tasks>

<task type="auto">
  <name>Adicionar seções de Design de Páginas e Charts ao SKILL.md</name>
  <files>.agent/skills/corteza-app-builder/SKILL.md</files>
  <action>
    Adicionar ao final do SKILL.md as seções de páginas e charts:

    **Seção `## Etapa 3: Design de Páginas`**

    Antes de criar páginas, perguntar:
    - Para cada módulo: quais campos exibir na lista? (max. 5-7 campos para listagem)
    - Para cada módulo: como organizar o formulário de detalhe? (grupos de campos)
    - O formulário de detalhe precisa mostrar registros filhos? (ex: lista de atualizações)
    - Haverá uma página de dashboard com charts? Se sim, quais?

    Regras de design de páginas:
    - Cada módulo precisa de DUAS páginas: uma de lista + uma de detalhe
    - Página de lista: `moduleID: 0`, `visible: true`, contém bloco `RecordList`
    - Página de detalhe: `moduleID: NomeModulo`, `visible: false`, contém bloco(s) `Record` + opcionalmente `RecordList` de filhos
    - Handle da página de detalhe DEVE seguir o padrão: `RecordPageForModule{Handle}`
    - O campo `selfID` da página de detalhe deve apontar para o handle da lista pai
    - O bloco `RecordList` na página de detalhe usa `prefilter: CampoIdFilho = ${recordID}`

    Template de definição de página (preencher para cada módulo):
    ```
    Módulo: [Nome]
    Página de Lista:
      Handle: [NomeListPage]
      Campos na lista: [campo1, campo2, campo3...]
      Presort: [campo ASC/DESC]
    Página de Detalhe:
      Blocos Record:
        - Seção "[Título]": [campos...]
        - Seção "[Título]": [campos...]
      Blocos RecordList de filhos:
        - Módulo filho: [Nome], FK: [campo], Campos: [...]
    ```

    Seção de layout:
    - Grid de 48 colunas — indicar larguras típicas:
      - 1 bloco Record: xywh = [0, 0, 48, 60]
      - 2 blocos Record lado a lado: [0,0,24,60] e [24,0,24,60]
      - Record (topo) + RecordList (baixo): [0,0,48,60] e [0,60,48,80]

    **Seção `## Etapa 4: Design de Charts` (opcional)**

    Perguntar:
    - Quais métricas de negócio são importantes visualizar? (ex: "quantidade de casos por status")
    - Agrupamento por categoria (Select field) → pie/doughnut/bar
    - Evolução temporal → line (usar campo DateTime com `modifier: MONTH`)
    - Soma de valores numéricos → bar com `aggregate: SUM`
    - Percentuais → qualquer tipo com `relativeValue: true`

    Tabela de decisão de chart:
    | Caso de Uso | Tipo | Configuração |
    |-------------|------|-------------|
    | Distribuição por categoria | `doughnut` ou `pie` | `relativeValue: true` |
    | Comparação de totais | `bar` | `aggregate: SUM` |
    | Evolução mês a mês | `line` | `modifier: MONTH`, `fill: true` |
    | Ranking de valores | `bar` | `aggregate: SUM`, `presort` |
    | Contagem simples | `pie` | `field: count` |

    Regras de chart:
    - `module` deve ser um handle de módulo existente
    - `field: count` conta registros; para somar, use o nome de um campo `Number`
    - `dimensions[].field` deve ser um campo do módulo (Select, DateTime, ou String de categorias)
    - Charts são referenciados nas páginas pelo `handle` — handle deve ser único no namespace
  </action>
  <verify>grep -q "Etapa 3" .agent/skills/corteza-app-builder/SKILL.md && grep -q "Etapa 4" .agent/skills/corteza-app-builder/SKILL.md && grep -q "RecordPageForModule" .agent/skills/corteza-app-builder/SKILL.md</verify>
  <done>SKILL.md contém seções de Design de Páginas e Charts com templates, regras e tabelas de decisão</done>
</task>

<task type="auto">
  <name>Adicionar seções de Geração YAML, Validação e Exemplos ao SKILL.md</name>
  <files>.agent/skills/corteza-app-builder/SKILL.md</files>
  <action>
    Adicionar ao SKILL.md a parte final da skill:

    **Seção `## Etapa 5: Geração e Validação do YAML`**

    Instrução para o assistente:
    "Antes de gerar o YAML, execute mentalmente o checklist de validação abaixo. Corrija qualquer problema antes de produzir o output."

    **Checklist de Validação (10 itens obrigatórios):**
    1. [ ] Todo campo `Record` referencia um `module` cujo handle existe na lista de módulos?
    2. [ ] Os módulos estão ordenados: os referenciados aparecem ANTES dos que os referenciam?
    3. [ ] Todo chart referencia um `module` cujo handle existe?
    4. [ ] Toda página de lista tem `moduleID: 0` e `visible: true`?
    5. [ ] Toda página de detalhe tem `moduleID: NomeModulo` e `visible: false`?
    6. [ ] O `selfID` de cada página de detalhe aponta para o handle da lista pai (que já existe no YAML)?
    7. [ ] Os handles de páginas de detalhe seguem o padrão `RecordPageForModule{Handle}`?
    8. [ ] Cada `RecordList` em páginas de detalhe tem `prefilter` correto: `CampoFKFilho = ${recordID}`?
    9. [ ] Todo bloco `Chart` em páginas usa o `chart: HandleDoChart` correto (não o name)?
    10. [ ] O `namespaceID` é idêntico em todos os módulos, charts e páginas — e bate com o `slug` do namespace?

    **Instrução de Geração:**
    "Gere o YAML na seguinte ordem:
    1. Nível `namespace:` com `name`, `slug`, `enabled: true`, `meta`
    2. Seção `chart:` com todos os charts
    3. Seção `module:` com módulos na ordem de dependência
    4. Seção `page:` com páginas de lista primeiro, depois páginas de detalhe"

    "Use APENAS as estruturas documentadas no REFERENCE.md. Não invente campos ou opções não listados."
    "Para IDs numéricos (`moduleID`, `chartID`, etc.) que precisam de UUID: deixe-os AUSENTES ou use o handle como string — o Corteza resolve por handle durante a importação."

    **Seção `## Limitações do Corteza` (armadilhas comuns):**
    - Corteza NÃO tem campos calculados automáticos — totais precisam ser atualizados via workflow ou manualmente
    - Corteza NÃO tem validação de campo obrigatório nativa via YAML (é configurada na UI)
    - Campos `Record` com `multi: true` salvam múltiplos IDs — considere isso em filtros
    - O `prefilter` usa o nome do campo (name), não o label
    - Charts NÃO suportam filtros dinâmicos — mostram sempre todos os dados do módulo (com filtros fixos apenas)
    - O grid é de 48 colunas — valores de `xywh` devem somar ≤48 na dimensão x

    **Seção `## Exemplo Rápido — App Mínimo`**
    Mostrar um exemplo compacto de namespace com 2 módulos, 1 relacionamento, 2 páginas e 1 chart,
    usando apenas estruturas do REFERENCE.md e comentários explicativos:
    ```yaml
    namespace:
      - name: "Gestão de Tarefas"
        slug: task-management
        enabled: true
        meta: {iconid: 0, logoid: 0, logoenabled: false, hidesidebar: false}
        chart:
          - config:
              reports:
                - module: Task
                  metrics:
                    - field: count
                      type: doughnut
                      relativeValue: true
                      label: Status
                  dimensions:
                    - field: Status
                      modifier: (no grouping / buckets)
                      conditions: {}
                      skipMissing: true
            handle: TasksByStatus
            name: Tasks by Status
            namespaceID: task-management
        module:
          - config: {dal: {connectionid: 0, constraints: {}, ident: "", systemfieldencoding: {id: null, moduleid: null, namespaceid: null, revision: null, meta: null, ownedby: null, createdat: null, createdby: null, updatedat: null, updatedby: null, deletedat: null, deletedby: null}}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, discovery: {public: {result: []}, private: {result: []}, protected: {result: []}}, recordrevisions: {enabled: false, ident: ""}, recorddedup: {strict: false, rules: []}}
            handle: Project
            name: Projeto
            namespaceID: task-management
            meta: [123, 125]
            moduleField:
              - config: {dal: {encodingstrategy: null}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, recordrevisions: {skip: false}}
                expressions: {disabledefaultvalidators: false, disabledefaultformatters: false}
                kind: String
                label: Nome do Projeto
                name: Name
                place: 0
          - config: {dal: {connectionid: 0, constraints: {}, ident: "", systemfieldencoding: {id: null, moduleid: null, namespaceid: null, revision: null, meta: null, ownedby: null, createdat: null, createdby: null, updatedat: null, updatedby: null, deletedat: null, deletedby: null}}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, discovery: {public: {result: []}, private: {result: []}, protected: {result: []}}, recordrevisions: {enabled: false, ident: ""}, recorddedup: {strict: false, rules: []}}
            handle: Task
            name: Tarefa
            namespaceID: task-management
            meta: [123, 125]
            moduleField:
              - config: {dal: {encodingstrategy: null}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, recordrevisions: {skip: false}}
                expressions: {disabledefaultvalidators: false, disabledefaultformatters: false}
                kind: String
                label: Título
                name: Title
                place: 0
              - config: {dal: {encodingstrategy: null}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, recordrevisions: {skip: false}}
                expressions: {disabledefaultvalidators: false, disabledefaultformatters: false}
                kind: Select
                label: Status
                name: Status
                options:
                  selectType: default
                  options:
                    - {text: "A fazer", value: "todo"}
                    - {text: "Em andamento", value: "in_progress"}
                    - {text: "Concluída", value: "done"}
                place: 0
              - config: {dal: {encodingstrategy: null}, privacy: {sensitivitylevelid: 0, usagedisclosure: ""}, recordrevisions: {skip: false}}
                expressions: {disabledefaultvalidators: false, disabledefaultformatters: false}
                kind: Record
                label: Projeto
                name: ProjectId
                options:
                  module: Project
                  labelField: Name
                  selectType: default
                  queryFields: [Name]
                place: 0
        page:
          - blocks:
              - blockid: 1
                kind: RecordList
                title: "Tasks"
                options: {module: Task, page: RecordPageForModuleTask, fields: [{name: Title}, {name: Status}, {name: ProjectId}], perPage: 20, prefilter: "", presort: "created_at DESC", allowExport: true}
                style: {variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}, wrap: {}, border: {}}
                xywh: [0, 0, 0, 0]
                meta: {}
                description: ""
              - blockid: 2
                kind: Chart
                title: "Tasks por Status"
                options: {chart: TasksByStatus}
                style: {variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}, wrap: {}, border: {}}
                xywh: [0, 0, 0, 0]
                meta: {}
                description: ""
            config: {navitem: {expanded: false, icon: null}}
            handle: Tasks
            moduleID: 0
            namespaceID: task-management
            selfID: 0
            title: Tasks
            visible: true
            weight: 0
            pageLayout:
              - blocks:
                  - {blockID: 1, xywh: [0, 0, 36, 80], meta: {}}
                  - {blockID: 2, xywh: [36, 0, 12, 80], meta: {}}
                config:
                  visibility: {expression: "", roles: []}
                  buttons: {new: {enabled: true, label: ""}, edit: {enabled: true, label: ""}, submit: {enabled: true, label: ""}, delete: {enabled: true, label: ""}, clone: {enabled: true, label: ""}, back: {enabled: true, label: ""}}
                  usetitle: false
                handle: primary
          - blocks:
              - blockid: 1
                kind: Record
                title: "Detalhes da Tarefa"
                options:
                  fields: [{name: Title}, {name: Status}, {name: ProjectId}]
                style: {variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}, wrap: {}, border: {}}
                xywh: [0, 0, 0, 0]
                meta: {}
                description: ""
            config: {navitem: {expanded: false, icon: null}}
            handle: RecordPageForModuleTask
            moduleID: Task
            namespaceID: task-management
            selfID: Tasks
            title: 'Record page for module "Task"'
            visible: false
            weight: 0
            pageLayout:
              - blocks:
                  - {blockID: 1, xywh: [0, 0, 48, 60], meta: {}}
                config:
                  visibility: {expression: "", roles: []}
                  buttons: {new: {enabled: true, label: ""}, edit: {enabled: true, label: ""}, submit: {enabled: true, label: ""}, delete: {enabled: true, label: ""}, clone: {enabled: true, label: ""}, back: {enabled: true, label: ""}}
                  usetitle: false
                handle: primary
    ```

    AVOID: Não truncar o exemplo — deve ser um YAML funcional e completo, mesmo que compacto.
  </action>
  <verify>grep -q "Etapa 5" .agent/skills/corteza-app-builder/SKILL.md && grep -q "Checklist" .agent/skills/corteza-app-builder/SKILL.md && grep -q "Limitações" .agent/skills/corteza-app-builder/SKILL.md && grep -q "task-management" .agent/skills/corteza-app-builder/SKILL.md</verify>
  <done>SKILL.md completo com checklist de validação, instrução de geração, limitações e exemplo mínimo funcional</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `grep -q "Etapa 3" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Etapa 4" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Etapa 5" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Checklist" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Limitações" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "task-management" .agent/skills/corteza-app-builder/SKILL.md`
</verification>

<success_criteria>
- [ ] SKILL.md completo com todas as 5 etapas
- [ ] Seção de Design de Páginas com template e regras de layout
- [ ] Seção de Design de Charts com tabela de decisão
- [ ] Checklist de validação com 10 itens
- [ ] Seção de Limitações do Corteza (armadilhas comuns)
- [ ] Exemplo mínimo funcional de app completo (2 módulos, 1 chart, 2 páginas)
</success_criteria>
