---
phase: 1
plan: 1
wave: 1
---

# Plan 1.1: Catalog de Tipos de Campo e Estrutura de Módulos

## Objective
Analisar exaustivamente o arquivo `exemplos_corteza/Case management.yaml` e extrair todos os tipos de campo (`moduleField`) com suas opções e restrições, além da estrutura de um módulo Corteza completo. O resultado será a primeira metade do `REFERENCE.md` — a fonte de verdade que a skill `corteza-app-builder` usará como base de conhecimento.

## Context
- `exemplos_corteza/Case management.yaml` — fonte principal
- `.gsd/SPEC.md` — objetivos do projeto
- `.gsd/REQUIREMENTS.md` — REQ-01, REQ-02, REQ-03

## Tasks

<task type="auto">
  <name>Extrair e documentar todos os tipos de campo de módulo</name>
  <files>.gsd/phases/1/REFERENCE.md</files>
  <action>
    Ler o arquivo `exemplos_corteza/Case management.yaml` e extrair exemplos reais de CADA tipo de campo (`kind`) presente.

    Tipos identificados para documentar (com exemplos YAML reais de cada um):
    - **String** — texto simples, texto longo (multiLine: true), rich text (useRichTextEditor: true)
    - **Number** — com format, prefix/suffix, precision
    - **Bool** — com trueLabel e falseLabel
    - **Select** — com lista de opções (options: [{text, value}]) e selectType
    - **Record** — relacionamento com outro módulo (module, labelField, queryFields, selectType)
    - **File** — com mode: single ou mode: list
    - **Email** — campo de e-mail
    - **Url** — campo de URL
    - **DateTime** — campo de data/hora
    - **User** — referência a usuário do sistema

    Para cada tipo:
    1. Mostrar a estrutura YAML mínima necessária
    2. Mostrar as opções disponíveis
    3. Incluir um exemplo real retirado do Case management.yaml
    4. Anotar restrições e comportamentos especiais

    Também documentar a estrutura completa de um módulo (campos obrigatórios: handle, name, namespaceID, moduleField + config mínima).

    NÃO incluir campos gerados automaticamente pelo Corteza (id, createdAt, moduleID com UUID — esses são gerados na importação).
  </action>
  <verify>grep -c "kind:" .gsd/phases/1/REFERENCE.md | test $(cat) -ge 8</verify>
  <done>REFERENCE.md existe com seção "Tipos de Campo" documentando pelo menos 8 tipos com exemplos YAML</done>
</task>

<task type="auto">
  <name>Documentar relacionamentos entre módulos (campos Record)</name>
  <files>.gsd/phases/1/REFERENCE.md</files>
  <action>
    Adicionar ao REFERENCE.md uma seção "Relacionamentos entre Módulos" que explique como o Corteza implementa relações entre tabelas via campos `Record`.

    Extrair do Case management.yaml exemplos reais de:
    1. **Um-para-muitos**: ex. Módulo Case tem campo CaseId no módulo Update (um Case tem muitos Updates)
    2. **Campo de lookup**: como configurar labelField (campo exibido) e queryFields (campos buscáveis)
    3. **selectType**: sempre "default" nos exemplos

    Incluir um diagrama de exemplo de relacionamentos do Case Management com setas mostrando as dependências.

    Documentar a diferença entre:
    - Campo `Record` no módulo filho (FK — ex: Update.CaseId → Case)
    - Como a RecordList na página do pai lista os filhos via `prefilter: FilhoCampoId = ${recordID}`
  </action>
  <verify>grep -q "prefilter" .gsd/phases/1/REFERENCE.md</verify>
  <done>REFERENCE.md tem seção de relacionamentos com pelo menos 2 exemplos reais e explicação do prefilter</done>
</task>

## Success Criteria
- [ ] REFERENCE.md existe em `.gsd/phases/1/REFERENCE.md`
- [ ] Documenta todos os 10 tipos de campo com exemplos YAML reais
- [ ] Documenta a estrutura mínima de um módulo
- [ ] Documenta relacionamentos entre módulos com explicação do prefilter
