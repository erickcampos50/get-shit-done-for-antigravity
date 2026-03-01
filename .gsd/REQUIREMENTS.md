# REQUIREMENTS.md

| ID | Requirement | Source | Status |
|----|-------------|--------|--------|
| REQ-01 | A skill deve documentar todos os tipos de campo suportados pelo Corteza (String, Number, Bool, Select, Record, File, Email, URL, DateTime, etc.) com exemplos de uso de cada tipo | SPEC goal 1 | Pending |
| REQ-02 | A skill deve orientar como criar relacionamentos entre módulos via campos `Record` com `labelField`, `queryFields` e `module` | SPEC goal 1 | Pending |
| REQ-03 | A skill deve orientar como estruturar campos `Select` com opções (text/value pairs) | SPEC goal 1 | Pending |
| REQ-04 | A skill deve orientar a criação de páginas de lista (`RecordList`) com filtros e ordenação | SPEC goal 2 | Pending |
| REQ-05 | A skill deve orientar a criação de páginas de detalhe/edição de registros (Record viewer/editor) | SPEC goal 2 | Pending |
| REQ-06 | A skill deve orientar a criação de charts (bar, line, pie, doughnut) com métricas e dimensões | SPEC goal 2 | Pending |
| REQ-07 | A skill deve gerar YAML válido no formato de namespace Corteza (namespace → module/chart/page) | SPEC goal 3 | Pending |
| REQ-08 | A skill deve validar coerência: referências a módulos em campos `Record` existem; módulos usados em charts existem; campos referenciados em páginas existem nos módulos | SPEC goal 4 | Pending |
| REQ-09 | A skill deve incluir exemplos concretos baseados no `exemplos_corteza/Case management.yaml` | SPEC goal 4 | Pending |
| REQ-10 | A skill deve seguir o formato SKILL.md do projeto: frontmatter YAML + instruções Markdown | SPEC constraint | Pending |
