---
phase: 1
plan: 2
wave: 1
---

# Plan 1.2: Catalog de Páginas, Charts e Estrutura YAML Completa

## Objective
Completar o `REFERENCE.md` com a segunda metade do conhecimento Corteza: tipos de blocos de página, configuração de charts e a estrutura completa do arquivo YAML de namespace. Esta é a referência que permitirá gerar apps Corteza válidos do zero.

## Context
- `exemplos_corteza/Case management.yaml` — fonte principal (seção `page` e `chart`)
- `.gsd/phases/1/REFERENCE.md` — continua do Plan 1.1 (deve existir)
- `.gsd/SPEC.md` — objetivos: REQ-04, REQ-05, REQ-06, REQ-07

## Tasks

<task type="auto">
  <name>Documentar tipos de blocos de página e sistema de layout</name>
  <files>.gsd/phases/1/REFERENCE.md</files>
  <action>
    Adicionar ao REFERENCE.md uma seção "Páginas e Blocos" documentando:

    **Tipos de página no Corteza:**
    1. **Página de lista** (`visible: true`, sem `moduleID`) — contém RecordList + Charts
    2. **Página de detalhe de registro** (`moduleID: NomeModulo`) — contém blocos Record + RecordList de filhos

    **Tipos de bloco (`kind`):**
    - `Record` — exibe campos do registro atual; opções: `fields: [{name: CampoX}]`, título de seção
    - `RecordList` — lista registros de outro módulo; opções: `module`, `fields`, `perPage`, `prefilter`, `presort`, `allowExport`, `page` (handle da página de detalhe)
    - `Chart` — exibe um chart; opções: `chart: HandleDoChart`
    - `SocialFeed` — feed de rede social (menos usado)

    **Sistema de layout (grid 48 colunas):**
    - Cada bloco tem `xywh: [x, y, width, height]` no pageLayout
    - Grid de 48 colunas; `width: 48` = largura total
    - Exemplo: `[0, 0, 48, 40]` = posição x=0, y=0, largura=48, altura=40

    **Estrutura de uma página:**
    ```yaml
    - blocks:         # definição dos blocos (content e presentation)
        - blockid: 1
          kind: Record
          title: "Informações principais"
          options:
            fields:
              - name: CampoA
              - name: CampoB
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
      handle: MinhaLista
      moduleID: 0        # 0 para páginas de lista
      title: "Título"
      visible: true       # aparece no menu lateral
      selfID: 0
      weight: 0
      pageLayout:         # posicionamento dos blocos no grid
        - blocks:
            - blockID: 1
              xywh: [0, 0, 48, 60]
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
  </action>
  <verify>grep -q "pageLayout" .gsd/phases/1/REFERENCE.md && grep -q "RecordList" .gsd/phases/1/REFERENCE.md</verify>
  <done>REFERENCE.md tem seção "Páginas e Blocos" com todos os tipos de bloco, sistema de grid e template YAML de página</done>
</task>

<task type="auto">
  <name>Documentar charts e estrutura YAML completa do namespace</name>
  <files>.gsd/phases/1/REFERENCE.md</files>
  <action>
    Adicionar ao REFERENCE.md duas seções finais:

    **Seção "Charts":**
    Tipos de chart disponíveis (todos via `config.reports[].metrics[].type`):
    - `bar` — barras; bom para comparações categóricas
    - `line` — linhas; bom para séries temporais (com `modifier: MONTH` ou `DATE`)
    - `pie` — pizza; bom para proporções (máx 6-8 fatias)
    - `doughnut` — rosca; igual ao pie mas com buraco no centro

    Para cada chart documentar:
    ```yaml
    - config:
        reports:
          - module: NomeModulo          # módulo fonte dos dados
            metrics:
              - field: count            # ou nome de campo numérico
                aggregate: SUM          # COUNT (default), SUM, AVG, MAX, MIN
                type: bar               # bar | line | pie | doughnut
                label: "Meu Label"
                backgroundColor: '#hex'
                relativeValue: true     # mostra % em vez de valor absoluto
                fill: true              # apenas para line (área preenchida)
                lineTension: 0.4        # suavização da linha (0-1)
            dimensions:
              - field: CampoCategoria   # campo de agrupamento
                modifier: (no grouping / buckets)  # ou MONTH, DATE, YEAR, WEEK
                conditions: {}
                skipMissing: true
      handle: MeuChart
      name: "Nome do Chart"
      namespaceID: meu-namespace
    ```

    **Seção "Estrutura Completa do YAML de Namespace":**
    O esqueleto mínimo de um arquivo YAML importável:
    ```yaml
    namespace:
      - name: "Nome da Aplicação"
        slug: handle-do-namespace        # identificador único URL-safe
        enabled: true
        meta:
          iconid: 0
          logoid: 0
          logoenabled: false
          hidesidebar: false
        chart:
          - # lista de charts (pode estar vazia: [])
        module:
          - # lista de módulos (obrigatório ter pelo menos 1)
        page:
          - # lista de páginas (obrigatório ter pelo menos 1)
    ```

    Documentar a ORDEM de dependências para geração:
    1. Módulos primeiro (pages referenciam modules; charts referenciam modules)
    2. Charts depois (pages referenciam charts pelo handle)
    3. Pages por último (referenciam modules e charts)
  </action>
  <verify>grep -q "doughnut" .gsd/phases/1/REFERENCE.md && grep -q "namespace:" .gsd/phases/1/REFERENCE.md</verify>
  <done>REFERENCE.md tem seção "Charts" com 4 tipos documentados e seção "Estrutura YAML Completa" com ordem de dependências</done>
</task>

## Success Criteria
- [ ] REFERENCE.md completo com seções: Intro, Tipos de Campo, Relacionamentos, Páginas e Blocos, Charts, Estrutura YAML
- [ ] Todos os 4 tipos de chart documentados com YAML de exemplo
- [ ] Esqueleto mínimo de namespace YAML documentado
- [ ] Ordem de dependências (módulos → charts → páginas) explicitada
