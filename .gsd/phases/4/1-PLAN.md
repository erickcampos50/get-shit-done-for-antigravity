---
phase: 4
plan: 1
wave: 1
depends_on: []
files_modified:
  - .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml
  - .agent/skills/corteza-app-builder/examples/helpdesk-ti.md
autonomous: true
must_haves:
  truths:
    - "helpdesk-ti.yaml existe com namespace, módulos, charts e páginas"
    - "Validação estrutural passa: módulos em ordem, prefilters presentes, handles corretos"
    - "helpdesk-ti.md documenta domínio e decisões"
  artifacts:
    - ".agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml"
    - ".agent/skills/corteza-app-builder/examples/helpdesk-ti.md"
---

# Plan 4.1: Gerar e Validar App Helpdesk de TI

<objective>
Exercitar a skill `corteza-app-builder` gerando um segundo app completo — um sistema de helpdesk
de TI — em um domínio diferente do CMMS. O objetivo é validar empiricamente que a skill guia
corretamente e que os padrões documentados produzem YAML estruturalmente válido.

O app será gerado seguindo o Modo Conversacional da skill (proposta proativa → refinamento),
e em seguida passará por 15 verificações estruturais automatizadas.

Output: helpdesk-ti.yaml + helpdesk-ti.md + resultado da validação estrutural.
</objective>

<context>
Load for context:
- .agent/skills/corteza-app-builder/SKILL.md  ← seguir este guia durante a geração
- .agent/skills/corteza-app-builder/templates/  ← usar como base
- .gsd/phases/1/REFERENCE.md  ← padrões YAML de referência
</context>

<tasks>

<task type="auto">
  <name>Gerar documento de design do Helpdesk</name>
  <files>.agent/skills/corteza-app-builder/examples/helpdesk-ti.md</files>
  <action>
    Seguindo o Modo Conversacional da skill, modelar um app de Helpdesk de TI com o seguinte domínio:
    - Objetivo: registrar e acompanhar chamados de suporte de TI internos
    - Usuários: técnicos de suporte, gestores, colaboradores que abrem chamados

    Módulos a modelar (com justificativas de design):
    1. `Solicitante` — colaborador que abre o chamado (Nome, Email, Departamento, Ramal)
    2. `Tecnico` — membro da equipe de TI (Nome, Email, Especialidade: Select [Hardware, Software, Redes, Geral])
    3. `Chamado` — o ticket de suporte com:
       - SolicitanteId (Record → Solicitante)
       - TecnicoId (Record → Tecnico)
       - Title (String — título do problema)
       - Category (Select — Hardware | Software | Redes | Acesso | Outro)
       - Priority (Select — Crítica | Alta | Média | Baixa)
       - Status (Select — Aberto | Em andamento | Aguardando usuário | Resolvido | Fechado)
       - Description (String multiLine)
       - Resolution (String multiLine)
       - OpenedDate (DateTime onlyDate)
       - ResolvedDate (DateTime onlyDate)
       - SatisfactionScore (Select — ★★★★★ | ★★★★ | ★★★ | ★★ | ★ — opcional, pós-resolução)

    Páginas: Solicitantes (lista) + detalhe, Tecnicos (lista) + detalhe, Chamados (lista+charts) + detalhe
    Charts: ChamadosPorStatus (doughnut), ChamadosPorCategoria (bar), ChamadosPorTecnico (bar)

    Criar `.agent/skills/corteza-app-builder/examples/helpdesk-ti.md` com:
    - Objetivo e domínio
    - Tabela de módulos com campos e tipos
    - Diagrama ASCII de relacionamentos
    - Tabela de páginas e charts
    - Decisões de design:
      - SatisfactionScore como Select em vez de Number (5 opções fixas → Select é mais correto)
      - SolicitanteId e TecnicoId como módulos separados (não User do Corteza) para flexibilidade
      - ResolvedDate editável manualmente (Corteza não calcula datas automaticamente)
  </action>
  <verify>grep -q "Relacionamentos" .agent/skills/corteza-app-builder/examples/helpdesk-ti.md && grep -q "SolicitanteId" .agent/skills/corteza-app-builder/examples/helpdesk-ti.md</verify>
  <done>helpdesk-ti.md existe com módulos, relacionamentos, páginas e charts documentados</done>
</task>

<task type="auto">
  <name>Gerar YAML do Helpdesk e executar validação estrutural</name>
  <files>.agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml</files>
  <action>
    Gerar `.agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml` seguindo o checklist de
    validação de 10 itens da Etapa 5 da skill ANTES e DURANTE a geração.

    Estrutura do app (namespace slug: `helpdesk-ti`):
    - 3 charts: ChamadosPorStatus (doughnut), ChamadosPorCategoria (bar), ChamadosPorTecnico (bar)
    - 3 módulos na ordem: Solicitante → Tecnico → Chamado
    - 6 páginas: 3 listas + 3 detalhes
      - Solicitantes (lista) + RecordPageForModuleSolicitante (detalhe — mostra chamados do solicitante)
      - Tecnicos (lista) + RecordPageForModuleTecnico (detalhe — mostra chamados do técnico)
      - Chamados (lista + charts) + RecordPageForModuleChamado (detalhe — dados do ticket)

    Aplicar padrões corretos:
    - `blockid` (minúsculo) nos blocos, `blockID` (maiúsculo) no pageLayout
    - `xywh: [0,0,0,0]` nos blocos; posições reais no pageLayout
    - prefilter: `SolicitanteId = ${recordID}` e `TecnicoId = ${recordID}`
    - `selfID` de cada detalhe aponta para sua lista
    - `namespaceID: helpdesk-ti` em todos os elementos

    Após gerar, executar estas 15 verificações estruturais com grep/bash:
    1. `grep -q "slug: helpdesk-ti"` → namespace correto
    2. `grep -c "kind: Record" | test >= 2` → pelo menos 2 campos Record
    3. `grep -c "prefilter:" | test >= 2` → prefilters nas páginas de detalhe
    4. `grep -q "RecordPageForModuleSolicitante"` → handle de detalhe correto
    5. `grep -q "RecordPageForModuleTecnico"` → handle correto
    6. `grep -q "RecordPageForModuleChamado"` → handle correto
    7. `grep -q "moduleID: 0"` → páginas de lista têm moduleID 0
    8. `grep -q "visible: false"` → páginas de detalhe têm visible false
    9. `grep -q "visible: true"` → páginas de lista têm visible true
    10. `grep -q "selfID: Solicitantes"` → selfID correto em detalhe de Solicitante
    11. `grep -q "selfID: Tecnicos"` → selfID correto
    12. `grep -q "selfID: Chamados"` → selfID correto
    13. `grep -q "doughnut"` → chart ChamadosPorStatus
    14. `grep -q "pageLayout"` → pageLayout presente (obrigatório)
    15. `grep -q "encodingstrategy"` → config de campo completo

    Documentar o resultado de cada verificação no SUMMARY.md desta task.

    AVOID: Não omitir config completo de módulo e campo.
    AVOID: Não usar UUIDs numéricos.
    AVOID: Não colocar xywh reais nos blocos (devem ficar no pageLayout).
  </action>
  <verify>grep -q "helpdesk-ti" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml && grep -q "prefilter" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml && grep -q "pageLayout" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml</verify>
  <done>helpdesk-ti.yaml existe; pelo menos 2 prefilters; pageLayout presente; 15 verificações estruturais executadas e documentadas</done>
</task>

</tasks>

<verification>
After all tasks, verify:
- [ ] `test -f .agent/skills/corteza-app-builder/examples/helpdesk-ti.md`
- [ ] `grep -q "Relacionamentos" .agent/skills/corteza-app-builder/examples/helpdesk-ti.md`
- [ ] `test -f .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml`
- [ ] `grep -q "helpdesk-ti" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml`
- [ ] `grep -q "prefilter" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml`
- [ ] `grep -q "pageLayout" .agent/skills/corteza-app-builder/examples/helpdesk-ti.yaml`
</verification>

<success_criteria>
- [ ] helpdesk-ti.md documenta domínio, módulos, relacionamentos, charts e decisões
- [ ] helpdesk-ti.yaml tem namespace, 3 charts, 3 módulos em ordem de dependência, 6 páginas
- [ ] Todas as 15 verificações estruturais passam
- [ ] App é um domínio diferente do CMMS (valida reusabilidade dos padrões)
</success_criteria>
