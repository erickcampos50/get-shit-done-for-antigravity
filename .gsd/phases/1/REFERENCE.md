# Corteza Compose — Guia de Referência

> **Fonte**: Extraído de `exemplos_corteza/Case management.yaml` e JSONs de workflow CRM.
> **Uso**: Base de conhecimento para a skill `corteza-app-builder`.
> **Versão Corteza**: 2026-01 (compatível com Corteza v2023+)

---

## Índice

1. [Estrutura de um Módulo](#1-estrutura-de-um-módulo)
2. [Tipos de Campo (moduleField)](#2-tipos-de-campo-modulefield)
3. [Relacionamentos entre Módulos](#3-relacionamentos-entre-módulos)
4. [Páginas e Blocos de UI](#4-páginas-e-blocos-de-ui)
5. [Charts (Gráficos)](#5-charts-gráficos)
6. [Estrutura Completa do YAML de Namespace](#6-estrutura-completa-do-yaml-de-namespace)

---

## 1. Estrutura de um Módulo

Um módulo no Corteza equivale a uma tabela de banco de dados. Ele define os campos e aparece como uma entidade gerenciável na aplicação.

### Estrutura Mínima

```yaml
- config:
    dal:
      connectionid: 0
      constraints: {}
      ident: ""
      systemfieldencoding:
        id: null
        moduleid: null
        namespaceid: null
        revision: null
        meta: null
        ownedby: null
        createdat: null
        createdby: null
        updatedat: null
        updatedby: null
        deletedat: null
        deletedby: null
    privacy:
      sensitivitylevelid: 0
      usagedisclosure: ""
    discovery:
      public:
        result: []
      private:
        result: []
      protected:
        result: []
    recordrevisions:
      enabled: false
      ident: ""
    recorddedup:
      strict: false
      rules: []
  handle: NomeModulo             # identificador único, PascalCase, sem espaços
  name: Nome Legível do Módulo   # rótulo exibido na interface
  namespaceID: meu-namespace     # deve bater com o slug do namespace
  meta:
    - 123
    - 125
  moduleField:
    # lista de campos — ver seção 2
```

### Campos Gerados Automaticamente pelo Corteza

**NÃO incluir no YAML gerado** — são criados na importação:
- `moduleID` (UUID numérico)
- `createdAt` (timestamp)
- Campos de sistema: `id`, `created_at`, `updated_at`, `owned_by`, etc.

### Template Mínimo de Campo

Cada item em `moduleField` deve ter esta estrutura base:

```yaml
- config:
    dal:
      encodingstrategy: null
    privacy:
      sensitivitylevelid: 0
      usagedisclosure: ""
    recordrevisions:
      skip: false
  expressions:
    disabledefaultvalidators: false
    disabledefaultformatters: false
  kind: TipoDoCampo         # ver seção 2
  label: Rótulo Visível     # exibido no formulário
  name: NomeCampo           # identificador, CamelCase, sem espaços
  place: 0                  # ordem de exibição (0 = usa ordem do arquivo)
  options:
    # opções específicas de cada tipo — ver seção 2
```

---

## 2. Tipos de Campo (moduleField)

### 2.1 String — Texto

O tipo mais versátil. Suporta texto simples, texto longo e rich text.

```yaml
# Texto simples (linha única)
kind: String
label: Número do Case
name: Number
# sem options necessárias

# Texto longo (multi-linha)
kind: String
label: Descrição
name: Description
options:
  multiLine: true

# Rich Text (editor WYSIWYG)
kind: String
label: Conteúdo
name: Content
options:
  multiLine: true
  useRichTextEditor: true
```

### 2.2 Number — Número

Suporta formatação customizada com prefixo, sufixo e casas decimais.

```yaml
# Número monetário
kind: Number
label: Custo Total
name: TotalCost
options:
  format: "0.00"
  prefix: $          # aparece antes do número
  precision: 2       # casas decimais

# Número com sufixo
kind: Number
label: Tempo Gasto
name: TimeSpend
options:
  format: "0.00"
  suffix: ' hour(s)'
  precision: 2
```

### 2.3 Bool — Booleano (Sim/Não)

```yaml
kind: Bool
label: Escalado
name: Escalated
options:
  trueLabel: "Yes"
  falseLabel: "No"
```

### 2.4 Select — Lista de Opções

Dropdown com opções predefinidas.

```yaml
# Select simples (uma opção)
kind: Select
label: Prioridade
name: Priority
options:
  selectType: default          # sempre "default" para seleção única
  options:
    - text: Muito baixa        # texto exibido ao usuário
      value: Very low          # valor salvo no banco
    - text: Baixa
      value: Low
    - text: Média
      value: Medium
    - text: Alta
      value: High
    - text: Muito alta
      value: Very high
```

**Nota**: `text` e `value` podem ser iguais. Use `value` em inglês/sem acentos se precisar referenciar em filtros ou workflows.

### 2.5 Record — Relacionamento com Outro Módulo

Cria um campo de "foreign key" que aponta para registros de outro módulo.

```yaml
# Relacionamento simples (seleciona um registro)
kind: Record
label: Conta
name: AccountId
options:
  module: Account              # handle do módulo referenciado
  labelField: AccountName      # campo do módulo referenciado exibido no dropdown
  selectType: default          # "default" = seleção única
  queryFields:                 # campos usados na busca/filtro do dropdown
    - AccountName

# Relacionamento que permite múltiplos registros
kind: Record
label: Lista de Contatos
name: MailingList
multi: true                    # permite selecionar vários registros
options:
  module: Contact
  labelField: RecordLabel
  selectType: default          # mesmo com multi: true, use "default" aqui
  queryFields:
    - RecordLabel
```

**Regra**: `labelField` deve ser o nome de um campo `String` do módulo referenciado que seja identificador legível.

### 2.6 File — Arquivo/Anexo

```yaml
# Upload de um único arquivo
kind: File
label: Foto
name: Photo
options:
  mode: single

# Upload de múltiplos arquivos
kind: File
label: Manuais
name: Manual
options:
  mode: list
```

### 2.7 Email — Endereço de E-mail

Valida formato de e-mail automaticamente.

```yaml
kind: Email
label: E-mail
name: ContactEmail
# sem options necessárias
```

### 2.8 Url — Endereço Web

```yaml
kind: Url
label: LinkedIn
name: LinkedIn
# sem options necessárias
```

### 2.9 DateTime — Data e/ou Hora

```yaml
# Apenas data (sem hora)
kind: DateTime
label: Data de Fechamento
name: CloseDate
options:
  onlyDate: true

# Data e hora completa
kind: DateTime
label: Data e Hora do Evento
name: EventDateTime
# sem options = data + hora
```

### 2.10 User — Referência a Usuário do Sistema

Cria um seletor de usuários cadastrados no Corteza.

```yaml
# Seleção múltipla de usuários
kind: User
label: Responsável pelo Case
name: CaseOwner
multi: true
options:
  presetWithAuthenticated: true  # pré-preenche com o usuário logado
  selectType: multiple

# Seleção única de usuário
kind: User
label: Criado por
name: CreatedBy
options:
  presetWithAuthenticated: false
  selectType: default
```

---

## 3. Relacionamentos entre Módulos

### 3.1 Padrão de Relacionamento Um-para-Muitos

O Corteza não tem `hasMany` nativo. O relacionamento é sempre definido no lado "filho" com um campo `Record`, e a exibição dos filhos na página do pai é feita via bloco `RecordList` com `prefilter`.

**Exemplo**: Um `Case` tem muitos `Update`s.

```
Case (pai)          Update (filho)
   id <──────────── CaseId (kind: Record → module: Case)
                    Content
                    Type
                    TimeSpend
```

**Módulo filho** (`Update`):
```yaml
kind: Record
label: Case
name: CaseId
options:
  module: Case
  labelField: Number        # campo do Case que aparece no dropdown
  selectType: default
  queryFields:
    - ContactName
    - Number
```

**Bloco RecordList na página do pai** (`Case`):
```yaml
kind: RecordList
title: Atualizações deste Case
options:
  module: Update
  page: RecordPageForModuleUpdate    # handle da página de detalhe do Update
  fields:
    - name: Type
    - name: Content
    - name: TimeSpend
  perPage: 20
  prefilter: CaseId = ${recordID}    # filtra filhos pelo ID do registro pai atual
  presort: created_at DESC
  allowExport: true
```

### 3.2 Relacionamentos Muitos-para-Muitos

Implemente via módulo de junção intermediário com dois campos `Record`.

**Exemplo**: `Case` ↔ `Product` via `CaseProduct`:
```
Case ←── CaseProduct.CaseId (kind: Record → module: Case)
Product ←── CaseProduct.ProductId (kind: Record → module: Product)
```

### 3.3 Diagrama de Relacionamentos (Case Management)

```
Account ←── Case.AccountId
            Case.ContactId ──→ Contact
            Case.ProductId ──→ Product

Case ←── Update.CaseId
Case ←── Attachement.CaseId

Contact ←── Update.ContactId
Account ←── Update.AccountId

Product ←── EntitlementTemplate.ProductId (via EntitlementTemplateId)
```

### 3.4 Campos de Sistema Usáveis em Prefilter e Presort

```yaml
prefilter: CampoId = ${recordID}     # ID do registro atual
presort: created_at DESC             # ordenação decrescente por data de criação
presort: Number ASC                  # ordenação crescente por campo Number
```

---

## 4. Páginas e Blocos de UI

### 4.1 Tipos de Página

| Tipo | `moduleID` | `visible` | Uso |
|------|-----------|-----------|-----|
| Lista | `0` ou ausente | `true` | Tela principal de um módulo, com RecordList + Charts |
| Detalhe | `NomeModulo` | `false` | Formulário de um registro específico |
| Dashboard | `0` | `true` | Página com apenas Charts |

### 4.2 Tipos de Bloco

#### `Record` — Exibe/edita campos do registro atual

```yaml
- blockid: 1
  kind: Record
  title: "Informações Principais"
  options:
    fields:
      - name: Status
      - name: Priority
      - name: Category
      - name: AccountId
  style:
    variants:
      bodyBg: white
      border: primary
      headerBg: white
      headerText: primary
    wrap: {}
    border: {}
  xywh: [0, 0, 0, 0]    # posição definida no pageLayout, não aqui
  meta: {}
  description: ""
```

#### `RecordList` — Lista registros de um módulo

```yaml
- blockid: 2
  kind: RecordList
  title: "Cases relacionadas"
  options:
    module: Case                         # módulo listado
    page: RecordPageForModuleCase        # handle da página de detalhe
    fields:
      - name: Number
      - name: Status
      - name: Priority
      - name: Subject
    perPage: 20
    prefilter: ContactId = ${recordID}   # filtro relacional (vazio para listas gerais)
    presort: created_at DESC
    allowExport: true
    draggable: false
    editable: false
    selectable: true
    selectMode: multi
    hideRecordCloneButton: true
    hideRecordEditButton: false
    hideRecordReminderButton: true
    hideRecordViewButton: true
  style:
    variants:
      bodyBg: white
      border: primary
      headerBg: white
      headerText: primary
    wrap: {}
    border: {}
  xywh: [0, 0, 0, 0]
  meta: {}
  description: ""
```

#### `Chart` — Exibe um gráfico

```yaml
- blockid: 3
  kind: Chart
  title: "Cases por Status"
  options:
    chart: CasesByStatus    # handle do chart definido na seção chart:
  style:
    variants:
      bodyBg: white
      border: primary
      headerBg: white
      headerText: primary
    wrap: {}
    border: {}
  xywh: [0, 0, 0, 0]
  meta: {}
  description: ""
```

### 4.3 Sistema de Layout (Grid de 48 Colunas)

As posições dos blocos são definidas no `pageLayout`, não nos blocos:

```yaml
pageLayout:
  - blocks:
      - blockID: 1          # referencia o blockid do bloco acima
        xywh:
          - 0               # coluna de início (0-47)
          - 0               # linha de início
          - 36              # largura em colunas
          - 60              # altura em unidades de linha
        meta: {}
      - blockID: 2
        xywh:
          - 36              # começa na coluna 36
          - 0
          - 12              # ocupa 12 colunas (36+12=48 = total)
          - 60
        meta: {}
    config:
      visibility:
        expression: ""
        roles: []
      buttons:
        new: {enabled: true, label: ""}
        edit: {enabled: true, label: ""}
        submit: {enabled: true, label: ""}
        delete: {enabled: true, label: ""}
        clone: {enabled: true, label: ""}
        back: {enabled: true, label: ""}
      usetitle: false
    handle: primary
```

**Dicas de layout:**
- `width: 48` = largura total da tela
- `width: 36` + `width: 12` = divisão 75%/25%
- `width: 24` + `width: 24` = divisão 50%/50%
- `height: 40-80` é típico para blocos `Record`; `height: 60-100` para `RecordList`

### 4.4 Estrutura Completa de uma Página de Lista

```yaml
- blocks:
    - blockid: 1
      kind: RecordList
      title: "Nome da Lista"
      options:
        module: NomeModulo
        page: RecordPageForModuleNomeModulo
        fields:
          - name: Campo1
          - name: Campo2
        perPage: 20
        prefilter: ""
        presort: created_at DESC
        allowExport: true
      style:
        variants:
          bodyBg: white
          border: primary
          headerBg: white
          headerText: primary
        wrap: {}
        border: {}
      xywh: [0, 0, 0, 0]
      meta: {}
      description: ""
  config:
    navitem:
      expanded: false
      icon: null
  handle: NomeListaPage              # handle único (PascalCase)
  moduleID: 0                        # 0 para páginas de lista
  namespaceID: meu-namespace
  title: "Título da Página"
  visible: true                      # aparece no menu lateral
  selfID: 0
  weight: 0                          # ordem no menu (menor = primeiro)
  pageLayout:
    - blocks:
        - blockID: 1
          xywh: [0, 0, 48, 80]
          meta: {}
      config:
        visibility: {expression: "", roles: []}
        buttons:
          new: {enabled: true, label: ""}
          edit: {enabled: true, label: ""}
          submit: {enabled: true, label: ""}
          delete: {enabled: true, label: ""}
          clone: {enabled: true, label: ""}
          back: {enabled: true, label: ""}
        usetitle: false
      handle: primary
```

### 4.5 Estrutura Completa de uma Página de Detalhe (Record Page)

```yaml
- blocks:
    - blockid: 1
      kind: Record
      title: "Dados Principais"
      options:
        fields:
          - name: Campo1
          - name: Campo2
          - name: Campo3
      style:
        variants:
          bodyBg: white
          border: primary
          headerBg: white
          headerText: primary
        wrap: {}
        border: {}
      xywh: [0, 0, 0, 0]
      meta: {}
      description: ""
    - blockid: 2
      kind: RecordList
      title: "Registros Filhos"
      options:
        module: ModuloFilho
        page: RecordPageForModuleModuloFilho
        fields:
          - name: Campo1
          - name: Campo2
        perPage: 20
        prefilter: CampoFKId = ${recordID}
        presort: created_at DESC
        allowExport: true
      style:
        variants:
          bodyBg: white
          border: primary
          headerBg: white
          headerText: primary
        wrap: {}
        border: {}
      xywh: [0, 0, 0, 0]
      meta: {}
      description: ""
  config:
    navitem:
      expanded: false
      icon: null
  handle: RecordPageForModuleNomeModulo    # convenção: RecordPageForModule + handle do módulo
  moduleID: NomeModulo                     # handle do módulo ao qual esta página pertence
  namespaceID: meu-namespace
  selfID: NomeListaPage                    # handle da página de lista pai (para breadcrumb)
  title: 'Record page for module "NomeModulo"'
  visible: false                           # NÃO aparece no menu lateral
  weight: 0
  pageLayout:
    - blocks:
        - blockID: 1
          xywh: [0, 0, 24, 60]
          meta: {}
        - blockID: 2
          xywh: [0, 60, 48, 60]
          meta: {}
      config:
        visibility: {expression: "", roles: []}
        buttons:
          new: {enabled: true, label: ""}
          edit: {enabled: true, label: ""}
          submit: {enabled: true, label: ""}
          delete: {enabled: true, label: ""}
          clone: {enabled: true, label: ""}
          back: {enabled: true, label: ""}
        usetitle: false
      handle: primary
```

---

## 5. Charts (Gráficos)

### 5.1 Tipos de Chart

| Tipo | `type` | Melhor Para |
|------|--------|-------------|
| Barras | `bar` | Comparações entre categorias |
| Linhas | `line` | Evolução temporal (séries com `modifier: MONTH`) |
| Pizza | `pie` | Proporções de poucas categorias (máx. 6-8) |
| Rosca | `doughnut` | Igual ao pie com buraco central, mais moderno |

### 5.2 Estrutura de um Chart

```yaml
- config:
    reports:
      - module: NomeModulo              # módulo fonte dos dados
        metrics:
          - field: count                # "count" ou nome de campo numérico
            aggregate: SUM             # COUNT (default) | SUM | AVG | MAX | MIN
            type: bar                  # bar | line | pie | doughnut
            label: "Rótulo do Chart"
            backgroundColor: '#bf94ff' # cor hex (optional)
            relativeValue: true        # true = mostra % em vez de valor absoluto
            fixTooltips: true
        dimensions:
          - field: StatusField          # campo de agrupamento (eixo X / fatias)
            modifier: (no grouping / buckets)  # ou: MONTH | DATE | YEAR | WEEK | QUARTER
            conditions: {}
            meta: {}
            skipMissing: true          # ignora registros sem valor neste campo
  handle: MeuChart                     # identificador único, PascalCase
  name: "Nome Exibido do Chart"
  namespaceID: meu-namespace
```

### 5.3 Exemplos Reais

```yaml
# Pie chart — Cases por Categoria
- config:
    reports:
      - module: Case
        metrics:
          - backgroundColor: '#ff0074'
            field: count
            fixTooltips: true
            label: Category
            relativePrecision: "2"
            relativeValue: true
            type: doughnut
        dimensions:
          - conditions: {}
            field: Category
            meta: {}
            modifier: (no grouping / buckets)
            skipMissing: true
  handle: CasesByCategory
  name: Cases by category
  namespaceID: service-solution

# Line chart — Novos Cases por Mês
- config:
    reports:
      - module: Case
        metrics:
          - field: count
            fill: true              # preenche a área abaixo da linha
            fixTooltips: null
            label: New cases
            lineTension: 0.2        # curvatura da linha (0=reto, 1=muito curvo)
            type: line
        dimensions:
          - autoSkip: true
            conditions: {}
            field: created_at
            meta: {}
            modifier: MONTH         # agrupa por mês
            skipMissing: true
  handle: NewCasesByMonth
  name: New cases per month
  namespaceID: service-solution

# Bar chart — Custo Total por Categoria
- config:
    reports:
      - module: Case
        metrics:
          - aggregate: SUM          # soma os valores do campo
            field: TotalCost
            label: Cost per category (in $)
            type: bar
        dimensions:
          - conditions: {}
            field: Category
            meta: {}
            modifier: (no grouping / buckets)
            skipMissing: false
  handle: CostPerCategory
  name: Total cost per category
  namespaceID: service-solution
```

### 5.4 Opções Avançadas de Métricas

```yaml
metrics:
  - field: count            # "count" = conta registros; qualquer campo Number = soma/média/etc
    aggregate: SUM          # SUM | AVG | MAX | MIN (COUNT é default quando field=count)
    type: pie
    label: "Meu Label"
    backgroundColor: '#hex' # cor específica da fatia/barra (opcional)
    relativeValue: true     # exibe porcentagem
    relativePrecision: "2"  # casas decimais do percentual
    fixTooltips: true       # posiona tooltips fixos
    fill: true              # line chart: preenche área (área chart)
    lineTension: 0.4        # line chart: suavização (0 = angular, 1 = muito curvo)
    beginAtZero: true       # bar chart: eixo Y começa em 0
```

---

## 6. Estrutura Completa do YAML de Namespace

### 6.1 Esqueleto Mínimo

```yaml
namespace:
  - name: "Nome da Aplicação"
    slug: handle-do-namespace        # identificador único, lowercase com hífens
    enabled: true
    meta:
      iconid: 0
      logoid: 0
      logoenabled: false
      hidesidebar: false
    chart:
      # lista de charts — pode ser [] se não houver
      - config:
          reports:
            - module: NomeModulo
              metrics:
                - field: count
                  type: pie
              dimensions:
                - field: CampoCategoria
                  modifier: (no grouping / buckets)
                  conditions: {}
                  skipMissing: true
        handle: MeuChart
        name: "Nome do Chart"
        namespaceID: handle-do-namespace
    module:
      # lista de módulos — pelo menos 1 obrigatório
      - config:
          dal:
            connectionid: 0
            constraints: {}
            ident: ""
            systemfieldencoding:
              id: null
              moduleid: null
              namespaceid: null
              revision: null
              meta: null
              ownedby: null
              createdat: null
              createdby: null
              updatedat: null
              updatedby: null
              deletedat: null
              deletedby: null
          privacy:
            sensitivitylevelid: 0
            usagedisclosure: ""
          discovery:
            public: {result: []}
            private: {result: []}
            protected: {result: []}
          recordrevisions:
            enabled: false
            ident: ""
          recorddedup:
            strict: false
            rules: []
        handle: NomeModulo
        name: "Nome Legível"
        namespaceID: handle-do-namespace
        meta:
          - 123
          - 125
        moduleField:
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: String
            label: "Nome"
            name: Name
            place: 0
    page:
      # lista de páginas — pelo menos 1 obrigatório
      # ordem recomendada: páginas de lista primeiro, depois páginas de detalhe
      - blocks:
          - blockid: 1
            kind: RecordList
            title: "Lista de NomeModulo"
            options:
              module: NomeModulo
              page: RecordPageForModuleNomeModulo
              fields:
                - name: Name
              perPage: 20
              prefilter: ""
              presort: created_at DESC
              allowExport: true
            style:
              variants:
                bodyBg: white
                border: primary
                headerBg: white
                headerText: primary
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem:
            expanded: false
            icon: null
        handle: NomeModuloListPage
        moduleID: 0
        namespaceID: handle-do-namespace
        selfID: 0
        title: "NomeModulo"
        visible: true
        weight: 0
        pageLayout:
          - blocks:
              - blockID: 1
                xywh: [0, 0, 48, 80]
                meta: {}
            config:
              visibility: {expression: "", roles: []}
              buttons:
                new: {enabled: true, label: ""}
                edit: {enabled: true, label: ""}
                submit: {enabled: true, label: ""}
                delete: {enabled: true, label: ""}
                clone: {enabled: true, label: ""}
                back: {enabled: true, label: ""}
              usetitle: false
            handle: primary
      - blocks:
          - blockid: 1
            kind: Record
            title: "Dados de NomeModulo"
            options:
              fields:
                - name: Name
                - name: OutroCampo
            style:
              variants:
                bodyBg: white
                border: primary
                headerBg: white
                headerText: primary
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem:
            expanded: false
            icon: null
        handle: RecordPageForModuleNomeModulo
        moduleID: NomeModulo
        namespaceID: handle-do-namespace
        selfID: NomeModuloListPage
        title: 'Record page for module "NomeModulo"'
        visible: false
        weight: 0
        pageLayout:
          - blocks:
              - blockID: 1
                xywh: [0, 0, 48, 80]
                meta: {}
            config:
              visibility: {expression: "", roles: []}
              buttons:
                new: {enabled: true, label: ""}
                edit: {enabled: true, label: ""}
                submit: {enabled: true, label: ""}
                delete: {enabled: true, label: ""}
                clone: {enabled: true, label: ""}
                back: {enabled: true, label: ""}
              usetitle: false
            handle: primary
```

### 6.2 Ordem de Dependências (Crítico!)

O Corteza resolve referências por `handle` durante a importação. A ordem no YAML importa:

```
1. module:   → Independentes (definem a estrutura de dados)
              → Módulos referenciados por Record devem aparecer ANTES dos que os referenciam
              → Ex: Contact antes de Case (Case.ContactId → Contact)

2. chart:    → Dependem de módulos (references module handles)
              → Não dependem de páginas

3. page:     → Dependem de módulos (moduleID, RecordList.module)
              → Dependem de charts (Chart block options.chart)
              → Páginas de lista ANTES das páginas de detalhe
              → selfID da página de detalhe = handle da lista (deve existir antes)
```

### 6.3 Convenções de Nomenclatura

| Elemento | Convenção | Exemplo |
|----------|-----------|---------|
| `namespace.slug` | lowercase-com-hifens | `case-management` |
| `module.handle` | PascalCase | `CaseUpdate` |
| `page.handle` (lista) | PascalCase | `Cases` |
| `page.handle` (detalhe) | `RecordPageForModule` + handle | `RecordPageForModuleCase` |
| `chart.handle` | PascalCase descritivo | `CasesByStatus` |
| `moduleField.name` | CamelCase | `ContactEmail` |
| `moduleField.label` | Legível em PT | `E-mail do Contato` |

### 6.4 Campos de Sistema Disponíveis nos Filtros

```yaml
prefilter: CampoId = ${recordID}    # ID do registro pai atual (em RecordList)
presort: created_at DESC            # data de criação
presort: updated_at DESC            # data de atualização
presort: owned_by ASC               # proprietário do registro
```

---

## 7. Checklist de Validação Antes de Gerar o YAML

Antes de produzir o YAML final, verificar:

- [ ] **Módulos**: Todo campo `Record` referencia um módulo cujo `handle` existe no arquivo?
- [ ] **Módulos**: A ordem garante que módulos referenciados aparecem antes dos que os referenciam?
- [ ] **Charts**: Todo chart referencia um `module.handle` que existe?
- [ ] **Páginas de lista**: `moduleID: 0` e `visible: true`
- [ ] **Páginas de detalhe**: `moduleID: NomeModulo` (handle, não UUID) e `visible: false`
- [ ] **selfID das páginas de detalhe**: aponta para o handle da página de lista correspondente?
- [ ] **RecordList em páginas de detalhe**: `prefilter` usa o nome correto do campo FK do filho?
- [ ] **Blocos Chart em páginas**: `options.chart` usa o handle do chart, não o nome?
- [ ] **namespaceID**: é o mesmo em `module`, `chart` e `page` e bate com `namespace.slug`?
