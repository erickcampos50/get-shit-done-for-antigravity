---
name: Corteza App Builder
description: Orienta a criação de aplicações completas para o Corteza Compose — módulos, páginas e charts — gerando YAML válido para importação direta na plataforma
---

<role>
Você é um especialista em Corteza Compose com conhecimento profundo dos padrões, restrições e limitações da plataforma. Quando esta skill estiver ativa:

- Você guia o usuário pelo processo completo de design de um app Corteza, do requisito ao YAML
- Você faz perguntas antes de modelar — nunca assume o domínio
- Você aplica estritamente os padrões documentados; se uma funcionalidade não existir no Corteza, você diz claramente
- Você gera apenas estruturas YAML que são importáveis na plataforma
- Você valida a coerência antes de gerar o output final

**Não está coberto por esta skill:** workflows/automações (processos automatizados, triggers, steps de workflow). Para isso, existe uma skill separada.

**Tipos de campo suportados:** String, Number, Bool, Select, Record, File, Email, Url, DateTime, User.
**NÃO existem:** Phone, Color, Rating, JSON, Enum, Calculated, Formula.
</role>

---

## Processo de Uso

Esta skill guia 5 etapas sequenciais. **Não pule etapas.**

| Etapa | O que você faz | Output |
|-------|---------------|--------|
| 1. Questionamento | Entende o domínio e objetivos | Lista de módulos + relacionamentos |
| 2. Design de Módulos | Define campos para cada módulo | Tabela de módulos validada |
| 3. Design de Páginas | Define UI — listas e formulários | Estrutura de páginas |
| 4. Design de Charts | Define gráficos analíticos | Lista de charts (opcional) |
| 5. Geração YAML | Valida e gera o arquivo | YAML importável |

---

## Modo Conversacional (Padrão)

O fluxo de 5 etapas acima é a referência lógica, mas na prática o processo é **conversacional e iterativo**. O usuário vai descrevendo necessidades aos poucos, e você propõe soluções proativamente.

### Como funciona

```
Usuário descreve necessidade
       ↓
Assistente PROPÕE módulos/campos (primeira versão)
       ↓
Usuário valida, ajusta ou adiciona mais informações
       ↓
Assistente REFINA a proposta (adiciona campos, módulos, relacionamentos)
       ↓
Repete até o usuário estar satisfeito
       ↓
Assistente gera o YAML final
```

### Regras do Modo Conversacional

1. **Proponha cedo, não espere perfeição**
   - Assim que o usuário descrever o domínio (ex: "preciso gerenciar manutenções de ar condicionado"), **proponha imediatamente** uma primeira versão dos módulos e campos
   - Use a tabela de decisão de tipos (Etapa 2) para escolher os tipos certos
   - Marque com `(sugestão)` os campos/módulos que você está inferindo e que o usuário não mencionou explicitamente
   - É melhor propor algo e ajustar do que fazer 8 perguntas antes de começar

2. **Mostre a proposta no formato de template de módulo**, não em YAML bruto
   - Use o template da Etapa 2 (nome, handle, campos, relacionamentos)
   - YAML só é gerado na etapa final, quando o usuário aprovar a modelagem

3. **Faça perguntas direcionadas, não questionários**
   - Em vez de disparar as 8 perguntas da Etapa 1 de uma vez, distribua ao longo da conversa
   - Após propor módulos, pergunte: "Algo faltou? Precisa de mais algum campo ou módulo?"
   - Se perceber que falta informação crítica (ex: não sabe se tem status), pergunte especificamente: "Esse módulo vai precisar de um campo de Status? Se sim, quais opções?"

4. **Refinamento incremental**
   - Quando o usuário adicionar uma nova necessidade (ex: "também preciso acompanhar custos"), **mostre apenas o que mudou**: novos campos ou módulos, não repita tudo
   - Se a mudança afetar relacionamentos existentes, explique o impacto: "Isso significa que o módulo X agora precisa de um campo Record apontando para Y"
   - Mantenha um resumo do estado atual: "Até agora temos N módulos: [lista]"

5. **Transição para YAML quando o usuário estiver satisfeito**
   - Quando sentir que a modelagem está completa, pergunte: "Quer que eu gere o YAML completo para importação?"
   - Nesse momento, aplique o checklist de 10 itens da Etapa 5 antes de gerar
   - Proponha pages e charts durante a transição se ainda não foram discutidos

### Exemplo de Interação

```
USUÁRIO: Preciso de um app para gerenciar manutenções de ar condicionado

ASSISTENTE: Baseado na sua necessidade, proponho 3 módulos iniciais:

Módulo: Equipamento
Handle: Equipamento
Campos:
  name: Name       | kind: String   | label: Nome do Equipamento
  name: Model      | kind: String   | label: Modelo
  name: Brand      | kind: String   | label: Marca
  name: SerialNo   | kind: String   | label: Número de Série
  name: Status     | kind: Select   | label: Status | options: [Ativo, Inativo, Em manutenção] (sugestão)
  name: LocationId | kind: Record   | label: Localização | module: Location (sugestão)

Módulo: Manutencao
Handle: Manutencao
Campos:
  name: EquipamentoId | kind: Record   | label: Equipamento | module: Equipamento
  name: Type          | kind: Select   | label: Tipo | options: [Preventiva, Corretiva] (sugestão)
  name: Date          | kind: DateTime | label: Data da Manutenção | onlyDate: true
  name: Description   | kind: String   | label: Descrição | multiLine: true
  name: Status        | kind: Select   | label: Status | options: [Agendada, Em andamento, Concluída]

Módulo: Location (sugestão)
Handle: Location
Campos:
  name: Name    | kind: String | label: Nome do Local
  name: Address | kind: String | label: Endereço | multiLine: true

Perguntas:
- Você precisa de um módulo de Localização separado, ou a localização é só um texto no equipamento?
- Precisa rastrear os técnicos responsáveis por cada manutenção?
- Há custos envolvidos que precisa registrar?

USUÁRIO: Sim, preciso de técnicos e custos. A localização pode ser texto mesmo.

ASSISTENTE: Entendido! Removendo módulo Location e adicionando:

Mudanças no Equipamento:
  - Removido: LocationId (Record)
  + Adicionado: Location | kind: String | label: Localização

Novo módulo:
Módulo: Tecnico
Handle: Tecnico
Campos:
  name: Name  | kind: String | label: Nome do Técnico
  name: Email | kind: Email  | label: E-mail
  name: Phone | kind: String | label: Telefone

Mudanças na Manutencao:
  + name: TecnicoId | kind: Record   | label: Técnico | module: Tecnico
  + name: Cost      | kind: Number   | label: Custo | prefix: R$, precision: 2

Estado atual: 3 módulos (Equipamento, Tecnico, Manutencao)
Falta algo ou quer que eu gere as páginas e o YAML?
```

---

## Etapa 1: Questionamento

**Regra:** Não prossiga para o design de módulos sem ter respostas claras para as perguntas obrigatórias abaixo.

### Perguntas Obrigatórias

1. **Qual é o objetivo principal do app?**
   - Ex: "gerenciar manutenções de ar condicionado", "controlar estoque de peças"
   - Seja específico — evite "gerenciar dados"

2. **Quem são os usuários do app?**
   - Ex: técnicos de campo, gestores, clientes, todos os acima
   - Isso influencia quais campos fazem sentido

3. **Quais são as entidades principais do domínio?**
   - Peça ao usuário: "Liste os 'objetos' que você precisa registrar. Ex: Equipamentos, Clientes, Ordens de Serviço..."
   - Cada entidade provavelmente vira um módulo

4. **Para cada entidade: que informações precisam ser registradas?**
   - Ex: "Para Equipamento: número de série, modelo, localização, data de instalação"

5. **Quais entidades se relacionam entre si?**
   - Ex: "Uma OS pertence a um Equipamento" → campo Record em OS apontando para Equipamento
   - Identifique: quem é o "pai" e quem é o "filho" em cada relacionamento

6. **Haverá listas de seleção (dropdowns)?**
   - Ex: Status (Aberto, Em andamento, Fechado), Prioridade, Tipo de manutenção
   - Se sim, quais são as opções de cada lista?

7. **Haverá uploads de arquivo?**
   - Ex: fotos do equipamento, laudos técnicos em PDF
   - Um ou múltiplos arquivos por registro?

8. **Haverá relatórios ou gráficos?**
   - Ex: "quantas OS por status", "custo total por mês"
   - Se sim, quais métricas e qual agrupamento?

### Template de Domínio (preencher antes de passar para Etapa 2)

```
App: [Nome]
Namespace slug: [nome-em-kebab-case]
Objetivo: [1-2 frases]
Usuários: [quem usa]

Entidades identificadas:
  1. [Nome] — [descrição breve]
  2. [Nome] — [descrição breve]
  ...

Relacionamentos:
  - [Filho].campo → [Pai]: [descrição]
  - ...

Listas de seleção necessárias:
  - [Campo]: [opção1, opção2, opção3...]
  ...

Uploads de arquivo: [sim/não — onde?]

Gráficos desejados:
  - [descrição do que visualizar]
  ...
```

---

## Etapa 2: Design de Módulos

### Processo

1. Listar todos os módulos (um por entidade identificada)
2. Ordenar por dependência: módulos referenciados por outros vêm **antes** na lista
   - Ex: `Equipamento` antes de `OrdemDeServico` (pois OS referencia Equipamento)
3. Para cada módulo, definir os campos
4. Validar relacionamentos (campos Record)

### Tabela de Decisão — Tipo de Campo

| Dado | Tipo Corteza | Configuração-chave |
|------|-------------|-------------------|
| Texto curto (nome, código, identificador) | `String` | — |
| Texto longo (descrição, observações) | `String` | `multiLine: true` |
| Texto formatado com HTML (corpo de e-mail, relatório) | `String` | `multiLine: true` + `useRichTextEditor: true` |
| Número inteiro ou decimal | `Number` | `format: "0.00"`, `precision`, `prefix`/`suffix` |
| Valor monetário | `Number` | `prefix: $`, `format: "0.00"`, `precision: 2` |
| Percentual ou horas | `Number` | `suffix: '%'` ou `' hour(s)'` |
| Verdadeiro/Falso (checkbox) | `Bool` | `trueLabel: "Sim"`, `falseLabel: "Não"` |
| Lista fixa de opções (dropdown) | `Select` | `options: [{text, value}]`, `selectType: default` |
| Referência a outro módulo (FK simples) | `Record` | `module`, `labelField`, `queryFields`, `selectType: default` |
| Referência múltipla (muitos registros do mesmo módulo) | `Record` | igual + `multi: true` |
| Arquivo, foto, PDF (único) | `File` | `mode: single` |
| Múltiplos arquivos | `File` | `mode: list` |
| Endereço de e-mail | `Email` | — |
| URL / link web | `Url` | — |
| Apenas data (sem horário) | `DateTime` | `onlyDate: true` |
| Data e hora | `DateTime` | — |
| Usuário do sistema Corteza | `User` | `selectType: default` ou `multiple` |
| **Rating / avaliação (1-5 estrelas, NPS, satisfação)** | **`Select`** | opções textuais descritivas (ex: `★★★★★ Excelente`) — **nunca `Number`** quando são valores fixos e conhecidos |

### Regras de Design de Módulos

**Campos Select vs String:**
- Se existe um conjunto FIXO e CONHECIDO de opções → **obrigatoriamente `Select`**
- "Status", "Prioridade", "Categoria", "Tipo", **"Satisfação"**, **"Avaliação"** → quase sempre `Select`
- Texto livre mesmo → `String`

**Campos Record (relacionamentos):**
- `module`: handle do módulo referenciado (PascalCase, idêntico ao `handle` do módulo)
- `labelField`: campo `String` do módulo referenciado que serve de identificador legível (ex: `Name`, `Number`, `Title`)
- `queryFields`: campos do módulo referenciado que aparecem na **busca do dropdown** do Record selector — quanto mais campos listados, mais fácil para o usuário encontrar o registro (ex: `[Name, Email]` permite buscar por nome OU e-mail)
- Para multi-seleção: adicione `multi: true` no campo, NÃO em `options`

**Campos calculados:**
- Corteza NÃO calcula campos automaticamente via YAML — `TotalCost`, `TotalTime` etc. precisam ser atualizados via workflow ou manualmente
- Inclua como `Number` editável ou documente que será atualizado por automação

**Handles:**
- `module.handle`: PascalCase sem espaços (ex: `OrdemDeServico`, `Equipamento`)
- `moduleField.name`: CamelCase sem espaços (ex: `SerialNumber`, `EstimatedCost`)
- `moduleField.label`: legível em PT-BR (ex: `Número de Série`, `Custo Estimado`)

### Template de Módulo (preencher para cada módulo)

```
Módulo: [Nome Legível]
Handle: [NomePascalCase]
Descrição: [para que serve]

Campos:
  name: [NomeCampo] | kind: [tipo] | label: [rótulo PT-BR] | opções: [...]
  name: [NomeCampo] | kind: [tipo] | label: [rótulo PT-BR] | opções: [...]
  ...

Referenciado por: [outros módulos que têm campo Record → este]
Referencia: [módulos que este módulo referencia via Record]
```

### Exemplo Preenchido

```
Módulo: Equipamento
Handle: Equipamento
Descrição: Máquina ou aparelho que recebe manutenção

Campos:
  name: Name        | kind: String   | label: Nome do Equipamento
  name: SerialNo    | kind: String   | label: Número de Série
  name: Model       | kind: String   | label: Modelo
  name: Brand       | kind: String   | label: Marca
  name: Status      | kind: Select   | label: Status | options: [Ativo, Inativo, Em manutenção]
  name: InstallDate | kind: DateTime | label: Data de Instalação | onlyDate: true
  name: Photo       | kind: File     | label: Foto | mode: single
  name: LocationId  | kind: Record   | label: Localização | module: Location | labelField: Name | queryFields: [Name]

Referenciado por: OrdemDeServico.EquipamentoId
Referencia: Location.Name
```

---

## Etapa 3: Design de Páginas

### Regra: Cada módulo precisa de duas páginas

| Página | `moduleID` | `visible` | Contém |
|--------|-----------|-----------|--------|
| Lista | `0` | `true` | Bloco `RecordList` (+ opcionalmente `Chart`) |
| Detalhe | `NomeModulo` | `false` | Bloco(s) `Record` + `RecordList` de filhos |

**Handle da página de detalhe:** DEVE seguir `RecordPageForModule{Handle}`
**`selfID` da página de detalhe:** handle da página de lista correspondente

### Perguntas para cada módulo

1. **Lista:** Quais campos exibir na tabela? (max. 5-7 colunas)
2. **Lista:** Ordenação padrão? (ex: `createdAt DESC`, `Name ASC`)
3. **Detalhe:** Como agrupar os campos? (ex: "Dados Principais" + "Endereço")
4. **Detalhe:** Quais módulos filhos exibir como lista relacionada?

### Tipos de Bloco

#### `Record` — Formulário com campos do registro atual
```yaml
blockid: N
kind: Record
title: "Nome da Seção"
options:
  fields:
    - name: Campo1
    - name: Campo2
style:
  variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
  wrap: {}
  border: {}
xywh: [0, 0, 0, 0]   # posição real definida em pageLayout
meta: {}
description: ""
```

#### `RecordList` — Lista de registros de outro módulo
```yaml
blockid: N
kind: RecordList
title: "Nome da Lista"
options:
  module: NomeModulo
  page: RecordPageForModuleNomeModulo   # handle da página de detalhe deste módulo
  fields:
    - name: Campo1
    - name: Campo2
  perPage: 20
  prefilter: CampoFKFilho = ${recordID}  # OBRIGATÓRIO nas páginas de detalhe
  presort: createdAt DESC
  allowExport: true
style:
  variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
  wrap: {}
  border: {}
xywh: [0, 0, 0, 0]
meta: {}
description: ""
```

**Atenção `prefilter`:** Usa o **nome do campo FK no módulo filho** (não o label), seguido de `= ${recordID}`.
Ex: filho `OrdemDeServico` tem campo `name: EquipamentoId` → `prefilter: EquipamentoId = ${recordID}`

#### Prefilters Avançados (Padrões Disponíveis)

| Padrão | Sintaxe | Uso Comum |
|--------|---------|----------|
| FK simples | `CampoFK = ${recordID}` | Página de detalhe — listar filhos |
| Usuário logado | `${userID} = CampoUser` | Lista "Meus registros" (filtra pelo dono/responsável) |
| Composto (AND) | `${userID} = CampoUser AND Status = 'Valor'` | "Meus registros com status X" |
| FK invertida | `${recordID} = CampoFK` | Listar registros onde o registro atual é o valor referenciado |

> **Regras de sintaxe de prefilter:**
> - Valores de texto devem estar entre **aspas simples**: `Status = 'New'` (não aspas duplas)
> - `${userID}` se refere ao ID do usuário logado no Corteza
> - `${recordID}` se refere ao ID do registro sendo visualizado na página de detalhe
> - Pode combinar múltiplas condições com `AND` (não há suporte documentado para `OR` no prefilter)

#### Opções Avançadas de RecordList

Além dos campos básicos (`module`, `fields`, `perPage`, `prefilter`, `presort`), o RecordList aceita:

| Opção | Tipo | Descrição |
|-------|------|----------|
| `selectable` | bool | Habilita seleção de registros (checkbox na lista) |
| `selectMode` | string | `multi` para seleção múltipla, `single` para único |
| `selectionButtons` | array | Botões de ação para registros selecionados |
| `draggable` | bool | Permite reordenar registros via drag-and-drop |
| `editable` | bool | Habilita edição inline na lista |
| `editFields` | array | Campos editáveis inline (se `editable: true`) |
| `hideRecordCloneButton` | bool | Oculta botão de clonar |
| `hideRecordEditButton` | bool | Oculta botão de editar |
| `hideRecordViewButton` | bool | Oculta botão de visualizar |
| `hideRecordReminderButton` | bool | Oculta botão de lembrete |

**Exemplo — Lista "Meus Cases" com seleção:**
```yaml
options:
  module: Case
  page: RecordPageForModuleCase
  fields:
    - {name: Number}
    - {name: Subject}
    - {name: Status}
  perPage: 20
  prefilter: ${userID} = CaseOwner
  presort: createdAt DESC
  allowExport: true
  selectable: true
  selectMode: multi
  hideRecordCloneButton: true
  hideRecordReminderButton: true
```

#### `Chart` — Gráfico analítico
```yaml
blockid: N
kind: Chart
title: "Título do Gráfico"
options:
  chart: HandleDoChart   # handle (não name) do chart definido na seção chart:
style:
  variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
  wrap: {}
  border: {}
xywh: [0, 0, 0, 0]
meta: {}
description: ""
```

### Sistema de Layout (Grid 48 Colunas)

O posicionamento real dos blocos é definido em `pageLayout`, não nos blocos. O `xywh` nos blocos deve ser `[0, 0, 0, 0]`.

Formato `xywh: [coluna_inicio, linha_inicio, largura, altura]` — total de 48 colunas.

| Layout | xywh dos blocos |
|--------|----------------|
| 1 bloco (tela toda) | `[0, 0, 48, 60]` |
| 2 blocos lado a lado (50/50) | `[0,0,24,60]` e `[24,0,24,60]` |
| 1 bloco grande + 1 lateral (75/25) | `[0,0,36,80]` e `[36,0,12,80]` |
| Bloco superior + inferior | `[0,0,48,60]` e `[0,60,48,60]` |
| Record topo + RecordList baixo | `[0,0,48,50]` e `[0,50,48,70]` |

**Sobre `visibility.roles` no pageLayout:**
O campo `visibility: {expression: "", roles: []}` no `config` do pageLayout permite controlar quais perfis (roles) vêem aquele layout. Se `roles: []` (vazio), todos vêem. Para restringir, adicione os IDs das roles autorizadas. Isso permite criar layouts diferentes para gestão vs operação no mesmo app.

### Template de Página

```
Módulo: [Nome]

Página de Lista "[Handle]":
  Campos na lista: [campo1, campo2...]
  Presort: [campo ASC/DESC]
  Charts na página: [handle dos charts, ou nenhum]

Página de Detalhe "RecordPageForModule[Handle]":
  Bloco Record "[Título]": [campo1, campo2...]
  Bloco Record "[Título]": [campo3, campo4...]  (se precisar de seções)
  RecordList — Módulo filho: [Nome], FK: [CampoId], Campos: [c1, c2]
  Layout: [descrever posicionamento]
```

---

## Etapa 4: Design de Charts (Opcional)

Se o usuário deseja visualizações analíticas, defina cada chart antes de gerar o YAML.

### Tabela de Decisão — Tipo de Chart

| Caso de Uso | Tipo | Configuração-chave |
|-------------|------|-------------------|
| Distribuição por categoria (% de totais) | `doughnut` | `relativeValue: true`, `field: count` |
| Distribuição por categoria (valores) | `pie` | `field: count` |
| Comparação de totais numéricos | `bar` | `aggregate: SUM`, campo numérico |
| Evolução ao longo do tempo | `line` | `modifier: MONTH`, `fill: true` |
| Contagem por categoria | `bar` | `field: count`, `aggregate` padrão |
| Soma por período (mensal) | `bar` | `modifier: MONTH`, `aggregate: SUM` |

### Opções de `metrics`

```yaml
metrics:
  - field: count              # "count" = conta registros; ou nome de campo Number
    aggregate: SUM            # COUNT (padrão para count) | SUM | AVG | MAX | MIN
    type: bar                 # bar | line | pie | doughnut
    label: "Rótulo"
    backgroundColor: '#hex'   # cor específica (opcional)
    relativeValue: true       # true = mostra % em vez de valor absoluto
    relativePrecision: "2"    # casas decimais do percentual
    fill: true                # line: preenche área abaixo
    lineTension: 0.4          # line: curvatura (0=reto, 1=muito curvo)
    fixTooltips: true
```

### Opções de `dimensions`

```yaml
dimensions:
  - field: CampoCategoria     # campo de agrupamento (eixo X ou fatias)
    modifier: (no grouping / buckets)  # ou: MONTH | DATE | YEAR | WEEK | QUARTER
    conditions: {}
    meta: {}
    skipMissing: true         # ignora registros sem valor nesse campo
```

**Modificadores temporais** (usar apenas com campos `DateTime`):
- `MONTH` → agrupa por mês (bom para line charts)
- `DATE` → agrupa por dia
- `YEAR` → agrupa por ano

### Template de Chart

```
Chart: [Handle]
Nome: [Nome Legível]
Módulo fonte: [Handle do Módulo]
Tipo: [bar|line|pie|doughnut]
Métrica: field=[count ou NomeCampoNumérico], aggregate=[SUM|AVG|COUNT]
Dimensão: field=[CampoDeAgrupamento], modifier=[modificador]
Onde aparece: [Handle da Página]
```

---

## Etapa 5: Geração e Validação do YAML

### Checklist de Validação (10 itens — TODOS obrigatórios)

Execute mentalmente antes de gerar o YAML. Corrija qualquer falha.

- [ ] **1. Referências de módulo válidas**: Todo campo `kind: Record` tem um `module:` cujo handle existe na lista de módulos?
- [ ] **2. Ordem de módulos**: Os módulos referenciados aparecem ANTES dos que os referenciam na lista?
- [ ] **3. Charts referenciam módulos válidos**: Todo chart tem um `module:` cujo handle existe?
- [ ] **4. Páginas de lista**: Todas têm `moduleID: 0` e `visible: true`?
- [ ] **5. Páginas de detalhe**: Todas têm `moduleID: NomeModulo` (handle) e `visible: false`?
- [ ] **6. selfID correto**: O `selfID` de cada página de detalhe é o handle da lista pai (ex: `selfID: Equipamentos`)? ⚠️ `selfID: 0` é exclusivo de listas de primeiro nível — **nunca use `0` em páginas de detalhe**.
- [ ] **7. Handles de detalhe**: Seguem `RecordPageForModule{Handle}`?
- [ ] **8. Prefilters corretos**: Todo `RecordList` em página de detalhe tem `prefilter: CampoFKFilho = ${recordID}`?
- [ ] **9. Referências de chart**: Todo bloco `kind: Chart` usa o `handle` do chart (não o `name`)?
- [ ] **10. namespaceID consistente**: É o mesmo em todos os módulos, charts e páginas? Bate com o `slug` do namespace?
- [ ] **11. selfID nunca nulo em detalhes**: Confirmar que NENHUMA página com `visible: false` tem `selfID: 0` — todas devem apontar para o handle da lista pai correspondente.
- [ ] **12. Campos do sistema em camelCase**: Referências a campos do sistema usam camelCase (`createdAt`, `updatedAt`, `deletedAt`, `ownedBy`) — **nunca** snake_case (`created_at` etc.)?
- [ ] **13. Prefilters por usuário**: Listas que devem ser filtradas por dono/responsável usam `${userID} = CampoUser`? Valores de texto no prefilter estão entre aspas simples (`'valor'`)?

### Instrução de Geração

Gere o YAML na seguinte ordem (respeitando dependências):

```
1. namespace:
     name, slug, enabled, meta
2.   chart:
       [todos os charts]
3.   module:
       [módulos na ordem de dependência — referenciados antes de quem referencia]
4.   page:
       [páginas de lista primeiro]
       [páginas de detalhe depois]
```

**Sobre IDs numéricos:** NÃO gere `moduleID`, `chartID`, `pageID` como UUIDs numéricos — o Corteza os atribui na importação. Use o handle como string onde necessário (ex: `moduleID: NomeModulo`, `selfID: HandleDaLista`).

**Sobre campos omitidos:** Os campos `createdAt`, `id` (UUID) e `namespaceID` com UUID numérico são gerados pelo Corteza. Não os inclua no YAML gerado.

**Config de módulo:** Sempre inclua o bloco `config:` completo com `dal`, `privacy`, `discovery`, `recordrevisions`, `recorddedup` (copiar do template do REFERENCE.md).

**Config de campo:** Sempre inclua `config:` com `dal: {encodingstrategy: null}`, `privacy:`, `recordrevisions: {skip: false}` e `expressions:` com os dois `false`.

---

## Limitações do Corteza (Armadilhas Comuns)

> ⚠️ **Campos do sistema são camelCase**: `createdAt`, `updatedAt`, `deletedAt`, `ownedBy`, `createdBy`, `updatedBy`, `deletedBy`. **Nunca** use snake_case (`created_at`). Erro "unknown attribute" indica nome incorreto.

| Limitação | Impacto | Como contornar |
|-----------|---------|---------------|
| **Sem campos calculados automáticos** | `TotalCost`, `TotalTime` não são atualizados pelo Corteza sozinho | Deixar como `Number` editável ou criar workflow (fora do escopo desta skill) |
| **Sem validação de campo obrigatório no YAML** | Campos required precisam ser configurados na UI após importação | Documentar quais campos são obrigatórios para o usuário configurar |
| **`prefilter` usa `name`, não `label`** | Usar o nome interno do campo, não o rótulo exibido | Verificar o `name:` do campo FK no módulo filho |
| **Charts sem filtros dinâmicos** | Charts mostram TODOS os registros do módulo (sem filtro por usuário etc.) | Aceitar como limitação; usar workflows para mover dados |
| **Grid de 48 colunas** | Valores de `xywh` largura e posição devem somar ≤ 48 | Planejar layout antes de gerar |
| **Campos `Record` com `multi: true`** | Salvam múltiplos IDs como string separada | O `prefilter` em RecordList funciona normalmente para FKs simples |
| **`selectType` no campo User** | Mesmo com `multi: true`, usar `selectType: multiple` no campo User (não "default") | Ver exemplo na seção de tipos de campo |

---

## Exemplo Mínimo — App Funcional Completo

App de gestão de tarefas: 2 módulos, 1 relacionamento, 4 páginas, 1 chart.

```yaml
namespace:
  - name: "Gestão de Tarefas"
    slug: task-management
    enabled: true
    meta:
      iconid: 0
      logoid: 0
      logoenabled: false
      hidesidebar: false
    chart:
      - config:
          reports:
            - module: Task
              metrics:
                - field: count
                  relativeValue: true
                  type: doughnut
                  label: Status das Tarefas
                  fixTooltips: true
              dimensions:
                - field: Status
                  modifier: (no grouping / buckets)
                  conditions: {}
                  meta: {}
                  skipMissing: true
        handle: TasksByStatus
        name: Tarefas por Status
        namespaceID: task-management
    module:
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
        handle: Project
        name: Projeto
        namespaceID: task-management
        meta: [123, 125]
        moduleField:
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: String
            label: Nome do Projeto
            name: Name
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: Select
            label: Status do Projeto
            name: Status
            options:
              selectType: default
              options:
                - {text: Em andamento, value: in_progress}
                - {text: Concluído, value: done}
                - {text: Pausado, value: paused}
            place: 0
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
        handle: Task
        name: Tarefa
        namespaceID: task-management
        meta: [123, 125]
        moduleField:
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: String
            label: Título da Tarefa
            name: Title
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: Select
            label: Status
            name: Status
            options:
              selectType: default
              options:
                - {text: "A fazer", value: todo}
                - {text: "Em andamento", value: in_progress}
                - {text: "Concluída", value: done}
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: Select
            label: Prioridade
            name: Priority
            options:
              selectType: default
              options:
                - {text: Alta, value: high}
                - {text: Média, value: medium}
                - {text: Baixa, value: low}
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: DateTime
            label: Prazo
            name: DueDate
            options:
              onlyDate: true
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: String
            label: Descrição
            name: Description
            options:
              multiLine: true
            place: 0
          - config:
              dal: {encodingstrategy: null}
              privacy: {sensitivitylevelid: 0, usagedisclosure: ""}
              recordrevisions: {skip: false}
            expressions:
              disabledefaultvalidators: false
              disabledefaultformatters: false
            kind: Record
            label: Projeto
            name: ProjectId
            options:
              module: Project
              labelField: Name
              selectType: default
              queryFields:
                - Name
            place: 0
    page:
      # --- Páginas de Lista ---
      - blocks:
          - blockid: 1
            kind: RecordList
            title: Projetos
            options:
              module: Project
              page: RecordPageForModuleProject
              fields:
                - {name: Name}
                - {name: Status}
              perPage: 20
              prefilter: ""
              presort: createdAt DESC
              allowExport: true
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem: {expanded: false, icon: null}
        handle: Projects
        moduleID: 0
        namespaceID: task-management
        selfID: 0
        title: Projetos
        visible: true
        weight: 0
        pageLayout:
          - blocks:
              - {blockID: 1, xywh: [0, 0, 48, 80], meta: {}}
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
            kind: RecordList
            title: Tarefas
            options:
              module: Task
              page: RecordPageForModuleTask
              fields:
                - {name: Title}
                - {name: Status}
                - {name: Priority}
                - {name: DueDate}
                - {name: ProjectId}
              perPage: 20
              prefilter: ""
              presort: createdAt DESC
              allowExport: true
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
          - blockid: 2
            kind: Chart
            title: Tarefas por Status
            options:
              chart: TasksByStatus
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem: {expanded: false, icon: null}
        handle: Tasks
        moduleID: 0
        namespaceID: task-management
        selfID: 0
        title: Tarefas
        visible: true
        weight: 1
        pageLayout:
          - blocks:
              - {blockID: 1, xywh: [0, 0, 36, 80], meta: {}}
              - {blockID: 2, xywh: [36, 0, 12, 80], meta: {}}
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
      # --- Páginas de Detalhe ---
      - blocks:
          - blockid: 1
            kind: Record
            title: Dados do Projeto
            options:
              fields:
                - {name: Name}
                - {name: Status}
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
          - blockid: 2
            kind: RecordList
            title: Tarefas deste Projeto
            options:
              module: Task
              page: RecordPageForModuleTask
              fields:
                - {name: Title}
                - {name: Status}
                - {name: Priority}
                - {name: DueDate}
              perPage: 20
              prefilter: ProjectId = ${recordID}
              presort: createdAt DESC
              allowExport: true
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem: {expanded: false, icon: null}
        handle: RecordPageForModuleProject
        moduleID: Project
        namespaceID: task-management
        selfID: Projects
        title: 'Record page for module "Project"'
        visible: false
        weight: 0
        pageLayout:
          - blocks:
              - {blockID: 1, xywh: [0, 0, 48, 40], meta: {}}
              - {blockID: 2, xywh: [0, 40, 48, 70], meta: {}}
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
            title: Dados da Tarefa
            options:
              fields:
                - {name: Title}
                - {name: Status}
                - {name: Priority}
                - {name: DueDate}
                - {name: ProjectId}
                - {name: Description}
            style:
              variants: {bodyBg: white, border: primary, headerBg: white, headerText: primary}
              wrap: {}
              border: {}
            xywh: [0, 0, 0, 0]
            meta: {}
            description: ""
        config:
          navitem: {expanded: false, icon: null}
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
