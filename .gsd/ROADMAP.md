# ROADMAP.md

> **Current Phase**: Not started
> **Milestone**: v1.0 — Corteza App Builder Skill

## Must-Haves (from SPEC)
- [ ] Skill documenta todos os tipos de campo Corteza com exemplos
- [ ] Skill orienta relacionamentos entre módulos
- [ ] Skill orienta criação de páginas (lista, detalhe, dashboard)
- [ ] Skill gera YAML válido importável no Corteza
- [ ] Skill valida coerência entre componentes

---

## Phases

### Phase 1: Pesquisa e Documento de Referência
**Status**: ⬜ Not Started
**Objective**: Extrair e documentar exaustivamente os padrões do Corteza a partir dos arquivos de exemplo (`Case management.yaml` + JSONs de workflow), criando um documento de referência interno que a skill usará como base de conhecimento
**Requirements**: REQ-01, REQ-02, REQ-03, REQ-09
**Deliverables**:
- Catálogo de todos os tipos de campo com suas opções
- Catálogo de tipos de componentes de página
- Catálogo de tipos de chart com configurações
- Padrões de relacionamento entre módulos

---

### Phase 2: Criação da Skill Principal (corteza-app-builder)
**Status**: ⬜ Not Started
**Objective**: Criar a skill orquestradora em `.agent/skills/corteza-app-builder/SKILL.md` com instruções completas para guiar o design de módulos, páginas e charts
**Requirements**: REQ-04, REQ-05, REQ-06, REQ-08, REQ-10
**Deliverables**:
- `SKILL.md` com processo de questionamento e modelagem
- Templates de módulos, páginas e charts
- Checklist de validação de coerência

---

### Phase 3: Templates e Exemplos
**Status**: ⬜ Not Started
**Objective**: Criar templates YAML reutilizáveis para padrões comuns (módulo simples, módulo com relacionamento, página de lista, página de detalhe, dashboard) e exemplos comentados baseados no Case Management
**Requirements**: REQ-07, REQ-09
**Deliverables**:
- Templates YAML em `.agent/skills/corteza-app-builder/templates/`
- Exemplos anotados em `.agent/skills/corteza-app-builder/examples/`

---

### Phase 4: Validação e Refinamento
**Status**: ⬜ Not Started
**Objective**: Testar a skill gerando um app Corteza completo (ex: app de helpdesk ou gestão de clientes) e validar que o YAML gerado é importável — refinando as instruções com base nos problemas encontrados
**Requirements**: REQ-07, REQ-08
**Deliverables**:
- App de exemplo gerado com a skill
- YAML validado (importado no Corteza ou revisado manualmente)
- Skill refinada com ajustes baseados nos testes
