---
phase: 3
plan: 2
wave: 1
depends_on: []
files_modified:
  - .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml
  - .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md
autonomous: true
must_haves:
  truths:
    - "Exemplo YAML de CMMS (manutenção de AC) gerado e comentado"
    - "Documento explicativo em Markdown acompanha o YAML"
    - "YAML é estruturalmente completo: módulos + páginas + charts"
  artifacts:
    - ".agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml"
    - ".agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md"
---

# Plan 3.2: Exemplo Anotado — App CMMS Manutenção de AC

<objective>
Criar um exemplo completo e comentado de app Corteza para gerenciamento de manutenções de
ar condicionado (CMMS — Computerized Maintenance Management System), demonstrando os padrões
da skill em um domínio já familiar (o contextualizado nas conversas anteriores).

Este exemplo serve como:
1. Prova de conceito que a skill funciona end-to-end
2. Referência de consulta para gerar apps similares
3. Material de validação da Phase 4

Output: 1 YAML comentado + 1 Markdown explicativo.
</objective>

<context>
Load for context:
- .gsd/phases/1/REFERENCE.md  ← estruturas YAML e padrões
- .agent/skills/corteza-app-builder/SKILL.md  ← tabelas de tipos e regras
- .agent/skills/corteza-app-builder/templates/  ← templates base (do Plan 3.1)
</context>

<tasks>

<task type="auto">
  <name>Criar documento explicativo do app CMMS em Markdown</name>
  <files>.agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md</files>
  <action>
    Criar `.agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md` documentando o design
    do app antes do YAML, com as decisões de modelagem explicadas:

    Seções:
    1. **Objetivo do App**: Sistema de gestão de manutenções preventivas e corretivas de ar condicionado
    2. **Módulos** (listar os 4 módulos com justificativa):
       - `Equipamento`: cadastro das máquinas de AC (modelo, marca, nº série, localização, status)
       - `Tecnico`: cadastro de técnicos responsáveis (nome, e-mail, telefone)
       - `Local`: localização dos equipamentos (nome, andar, setor, endereço)
       - `Manutencao`: registro de manutenções (equipamento, técnico, tipo, data, status, custo, observações)
    3. **Relacionamentos** com diagrama ASCII:
       ```
       Local ←── Equipamento.LocalId
       Equipamento ←── Manutencao.EquipamentoId
       Tecnico ←── Manutencao.TecnicoId
       ```
    4. **Campos Select e suas opções**:
       - Equipamento.Status: Ativo | Inativo | Em manutenção
       - Manutencao.Tipo: Preventiva | Corretiva | Inspeção
       - Manutencao.Status: Agendada | Em andamento | Concluída | Cancelada
    5. **Páginas** (6 páginas: 1 lista + 1 detalhe por módulo, exceto Local que é mais simples)
    6. **Charts** (3 gráficos):
       - Manutenções por Status (doughnut)
       - Custo Total por Mês (line com aggregate SUM)
       - Manutenções por Tipo (bar)
    7. **Decisões de design** e limitações documentadas:
       - Custo total não é calculado automaticamente (campo editável)
       - Charts sem filtro por técnico (limitação do Corteza)
  </action>
  <verify>grep -q "Relacionamentos" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md && grep -q "Manutencao" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md</verify>
  <done>cmms-manutencao-ac.md existe com seções de módulos, relacionamentos, campos Select, páginas, charts e decisões</done>
</task>

<task type="auto">
  <name>Gerar YAML comentado do app CMMS</name>
  <files>.agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml</files>
  <action>
    Gerar `.agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml` — YAML completo e
    importável do app CMMS, com comentários `#` nas partes-chave explicando o padrão.

    O YAML deve conter, nesta ordem:
    1. `namespace:` com slug `manutencao-ac`
    2. `chart:` com 3 charts:
       - `ManutencoesPorStatus` (doughnut, relativeValue: true, field: count, dimensão: Status)
       - `CustoTotalPorMes` (line, aggregate: SUM, field: Cost, dimensão: created_at modifier: MONTH)
       - `ManutencoesPorTipo` (bar, field: count, dimensão: Tipo)
    3. `module:` com 4 módulos na ordem de dependência:
       - Local (sem dependências)
       - Tecnico (sem dependências)
       - Equipamento (referencia Local)
       - Manutencao (referencia Equipamento e Tecnico)
    4. `page:` com 6 páginas:
       - Locais (lista) + RecordPageForModuleLocal (detalhe)
       - Tecnicos (lista) + RecordPageForModuleTecnico (detalhe)
       - Equipamentos (lista) + RecordPageForModuleEquipamento (detalhe — inclui RecordList de Manutencao)
       - Manutencoes (lista com chart ManutencoesPorStatus) + RecordPageForModuleManutencao (detalhe)

    Aplicar os padrões corretos documentados no REFERENCE.md:
    - Blocos com `blockid` (minúsculo) + pageLayout com `blockID` (maiúsculo)
    - `xywh: [0, 0, 0, 0]` nos blocos; xywh reais no pageLayout
    - `prefilter: EquipamentoId = ${recordID}` na RecordList de Manutencao dentro da página de Equipamento
    - `selfID` de cada página de detalhe aponta para a lista correspondente
    - `namespaceID: manutencao-ac` em todos os módulos, charts e páginas

    Comentar nas seções-chave:
    - "# Ordem de dependência: Local e Tecnico antes de Equipamento e Manutencao"
    - "# prefilter: usa o name do campo FK, não o label"
    - "# blockid (minúsculo) no bloco vs blockID (maiúsculo) no pageLayout"
    - "# selfID aponta para o handle da página de lista (não UUID)"

    AVOID: Não usar UUIDs numéricos para IDs — usar handles ou 0.
    AVOID: Não omitir o config completo de módulo (dal, privacy, discovery, recordrevisions, recorddedup).
    AVOID: Não omitir o config de campo (dal: encodingstrategy null, privacy, recordrevisions, expressions).
  </action>
  <verify>grep -q "manutencao-ac" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml && grep -q "prefilter" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml && grep -q "doughnut" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml</verify>
  <done>cmms-manutencao-ac.yaml existe com namespace, 3 charts, 4 módulos em ordem de dependência, 6 páginas com prefilter correto</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `test -f .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md`
- [ ] `grep -q "Relacionamentos" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.md`
- [ ] `test -f .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml`
- [ ] `grep -q "manutencao-ac" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml`
- [ ] `grep -q "prefilter" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml`
- [ ] `grep -q "doughnut" .agent/skills/corteza-app-builder/examples/cmms-manutencao-ac.yaml`
</verification>

<success_criteria>
- [ ] cmms-manutencao-ac.md documenta o design com módulos, relacionamentos, campos, páginas, charts e decisões
- [ ] cmms-manutencao-ac.yaml é estruturalmente completo: 4 módulos, 3 charts, 6 páginas
- [ ] Módulos ordenados por dependência no YAML
- [ ] prefilter correto nas RecordLists relacionais
- [ ] Comentários inline nas seções-chave do YAML
</success_criteria>
