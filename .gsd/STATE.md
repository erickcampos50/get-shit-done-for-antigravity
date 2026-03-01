# STATE.md — Project Memory

> **Last Updated**: 2026-03-01
> **Current Phase**: Not started (run `/plan 1` to begin)

## Current Status

Project initialized. Ready to begin Phase 1 execution.

## Context

- **Project**: Corteza App Builder Skill
- **Goal**: Skill orquestradora para criar apps Corteza completos via Claude Code
- **Key Files**:
  - `exemplos_corteza/Case management.yaml` — referência principal de estrutura YAML
  - `exemplos_corteza/CRM - Account - Create New Contract.json` — exemplo de workflow
  - `exemplos_corteza/CRM - Opportunity - Update Campaigns.json` — exemplo de workflow complexo
- **Skill destination**: `.agent/skills/corteza-app-builder/SKILL.md`

## Decisions Made

- Workflows/automações fora do escopo inicial (Phase 1-4)
- Skill segue formato do projeto: frontmatter YAML + Markdown
- Saída: YAML compatível com formato de namespace Corteza

## Blockers

None.
