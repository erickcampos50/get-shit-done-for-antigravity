# DECISIONS.md — Architecture Decision Records

## ADR-001: Skill única orquestradora

**Date**: 2026-03-01
**Status**: Accepted

### Context
Considerou-se criar skills separadas por domínio (módulos, páginas, charts) vs. uma skill orquestradora.

### Decision
Criar uma única skill principal que guia todo o processo end-to-end.

### Rationale
- O processo de criação de um app Corteza é intrinsecamente sequencial e interdependente (módulos antes de páginas, antes de charts)
- Uma skill orquestradora reduz fricção — o dev não precisa saber qual skill invocar em cada etapa
- Mais fácil de manter e evolucionar

---

## ADR-002: Workflows fora do escopo inicial

**Date**: 2026-03-01
**Status**: Accepted

### Context
Workflows/automações são parte importante do Corteza, mas aumentam significativamente a complexidade da skill.

### Decision
Excluir geração de workflows do escopo da v1.0.

### Rationale
- Permite foco na qualidade da modelagem de dados e UI primeiro
- Workflows podem ser adicionados como uma skill complementar depois
