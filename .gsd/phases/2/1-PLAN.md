---
phase: 2
plan: 1
wave: 1
depends_on: []
files_modified:
  - .agent/skills/corteza-app-builder/SKILL.md
autonomous: true
must_haves:
  truths:
    - "SKILL.md exists no path .agent/skills/corteza-app-builder/"
    - "SKILL.md tem frontmatter YAML com name e description"
    - "SKILL.md guia o processo de questionamento e design de módulos"
    - "SKILL.md documenta todos os tipos de campo com critérios de escolha"
  artifacts:
    - ".agent/skills/corteza-app-builder/SKILL.md (parcial — seções 1 a 4)"
---

# Plan 2.1: SKILL.md — Estrutura Base, Processo e Design de Módulos

<objective>
Criar o arquivo `.agent/skills/corteza-app-builder/SKILL.md` com as seções iniciais da skill:
metadata, role (papel do assistente), processo de uso, e as seções de questionamento e design de módulos.

Esta é a parte mais crítica da skill: o processo de levantamento de requisitos e a
orientação sobre como modelar os módulos Corteza com os campos corretos.

Output: SKILL.md com seções 1-4 (metadata, role, processo, questionamento, módulos).
</objective>

<context>
Load for context:
- .gsd/SPEC.md
- .gsd/phases/1/REFERENCE.md  ← base de conhecimento Corteza
- .agent/skills/planner/SKILL.md ← formato de referência de uma skill existente
</context>

<tasks>

<task type="auto">
  <name>Criar estrutura base do SKILL.md com metadata, role e processo</name>
  <files>.agent/skills/corteza-app-builder/SKILL.md</files>
  <action>
    Criar o diretório `.agent/skills/corteza-app-builder/` e o arquivo `SKILL.md`.

    O arquivo deve começar com frontmatter YAML:
    ```yaml
    ---
    name: Corteza App Builder
    description: Orienta a criação de aplicações completas para o Corteza Compose — módulos, páginas e charts — gerando YAML válido para importação
    ---
    ```

    Em seguida, a seção `<role>` explicando o papel do assistente quando esta skill é ativa:
    - Especialista em Corteza Compose que conhece profundamente os padrões da plataforma
    - Guia o processo de design aplicando as regras e restrições do Corteza
    - Faz perguntas para entender o domínio antes de modelar
    - Gera YAML válido e importável
    - NÃO inventa funcionalidades que o Corteza não suporta

    Depois, a seção `## Processo de Uso` descrevendo o fluxo em 5 etapas:
    1. Questionamento (entender o domínio e objetivos)
    2. Design de Módulos (tabelas e campos)
    3. Design de Páginas (UI — listas e formulários)
    4. Design de Charts (gráficos analíticos, opcional)
    5. Geração e Validação do YAML

    Incluir uma nota sobre o que NÃO está coberto por esta skill (workflows/automações).

    AVOID: Não mencionar tecnologias que não existem no Corteza (ex: "stored procedures", "triggers SQL").
    AVOID: Não inventar tipos de campo que não existem (ex: "Phone", "Color", "Rating").
  </action>
  <verify>grep -q "name: Corteza App Builder" .agent/skills/corteza-app-builder/SKILL.md && grep -q "Processo de Uso" .agent/skills/corteza-app-builder/SKILL.md</verify>
  <done>SKILL.md existe com frontmatter válido, role definido e seção de processo de uso com 5 etapas</done>
</task>

<task type="auto">
  <name>Adicionar seção de Questionamento e Design de Módulos ao SKILL.md</name>
  <files>.agent/skills/corteza-app-builder/SKILL.md</files>
  <action>
    Adicionar ao SKILL.md as seções mais importantes da skill:

    **Seção `## Etapa 1: Questionamento`**
    Questões que o assistente DEVE fazer antes de começar a modelar:
    - Qual é o objetivo principal do app? (ex: "gerenciar manutenções de ar condicionado")
    - Quem são os usuários? (técnicos, gestores, clientes?)
    - Que entidades existem no domínio? (peça ao usuário listar os "objetos" do negócio)
    - Para cada entidade: que informações precisam ser registradas?
    - Quais entidades se relacionam? (ex: "uma manutenção pertence a um equipamento")
    - Haverá listas de seleção (status, categorias, prioridades) ou tudo é texto livre?
    - Haverá uploads de arquivo (fotos, PDFs, manuais)?
    - Haverá relatórios/gráficos? Se sim, quais métricas?

    Incluir a instrução: "Não prossiga para o design de módulos até ter respostas claras para pelo menos: objetivo, entidades principais, e relacionamentos."

    **Seção `## Etapa 2: Design de Módulos`**
    Explicar o processo:
    1. Listar todos os módulos identificados no questionamento
    2. Para cada módulo, identificar os campos necessários
    3. Escolher o tipo correto para cada campo (ver tabela abaixo)
    4. Identificar campos de relacionamento (kind: Record)
    5. Ordenar módulos por dependência (módulos referenciados → módulos que referenciam)

    Incluir a **tabela de decisão de tipos de campo**:
    | Dado | Tipo Corteza | Configuração-chave |
    |------|-------------|-------------------|
    | Texto curto (nome, código) | String | - |
    | Texto longo | String | `multiLine: true` |
    | Texto formatado (HTML) | String | `multiLine: true`, `useRichTextEditor: true` |
    | Número inteiro ou decimal | Number | `format`, `precision`, `prefix`/`suffix` |
    | Sim/Não | Bool | `trueLabel`, `falseLabel` |
    | Lista fixa de opções | Select | `options: [{text, value}]` |
    | Referência a outro módulo | Record | `module`, `labelField`, `queryFields` |
    | Múltiplas referências | Record + `multi: true` | igual + `multi: true` |
    | Arquivo/foto/PDF | File | `mode: single` ou `mode: list` |
    | E-mail | Email | - |
    | URL/link | Url | - |
    | Data (sem hora) | DateTime | `onlyDate: true` |
    | Data e hora | DateTime | - |
    | Usuário do sistema | User | `selectType: default` ou `multiple` |

    Incluir regras de design:
    - EVITAR criar campos String que deveriam ser Select (se há opções fixas, use Select)
    - Para "status" e "prioridade" quase sempre use Select
    - Campos de "total" ou "soma" geralmente são calculated via workflow (mas podem ser Number editável)
    - Campo Record: `labelField` deve ser um campo String identificador do módulo referenciado
    - `queryFields` lista os campos do módulo referenciado que aparecem na busca do dropdown

    Incluir template de definição de módulo que o assistente deve preencher:
    ```
    Módulo: [Nome]
    Handle: [NomePascalCase]
    Descrição: [Para que serve]
    Campos:
      - Nome: [campo] | Tipo: [tipo] | Label: [rótulo PT] | Opções: [...]
      - ...
    Relacionamentos recebidos: [outros módulos que referenciam este]
    Relacionamentos que faz: [módulos que este referencia via Record]
    ```
  </action>
  <verify>grep -q "Etapa 1" .agent/skills/corteza-app-builder/SKILL.md && grep -q "Etapa 2" .agent/skills/corteza-app-builder/SKILL.md && grep -q "useRichTextEditor" .agent/skills/corteza-app-builder/SKILL.md</verify>
  <done>SKILL.md contém seções de Questionamento e Design de Módulos com tabela de decisão e regras de design</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `grep -q "name: Corteza App Builder" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Etapa 1" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "Etapa 2" .agent/skills/corteza-app-builder/SKILL.md`
- [ ] `grep -q "useRichTextEditor" .agent/skills/corteza-app-builder/SKILL.md` (tabela de tipos presente)
</verification>

<success_criteria>
- [ ] `.agent/skills/corteza-app-builder/SKILL.md` criado com frontmatter válido
- [ ] Role do assistente definido (especialista Corteza, sem inventar features)
- [ ] Processo de 5 etapas documentado
- [ ] Seção de questionamento com perguntas obrigatórias
- [ ] Tabela de decisão de tipos de campo completa (13 tipos)
- [ ] Regras de design de módulos documentadas
</success_criteria>
