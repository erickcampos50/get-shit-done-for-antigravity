# SPEC.md — Project Specification

> **Status**: `FINALIZED`

## Vision

Criar uma skill principal (orquestradora) para o ambiente Claude Code que guie desenvolvedores na criação de aplicações completas para o Corteza Compose — desde a concepção dos módulos (tabelas) com seus campos tipados e relacionamentos, até a geração de páginas/formulários e gráficos analíticos — produzindo arquivos YAML válidos prontos para importação na plataforma.

## Goals

1. **Modelagem de dados**: Guiar o design de módulos Corteza com campos adequados (String, Number, Bool, Select, Record, File, Email, URL, DateTime, etc.) e relacionamentos entre módulos via campos `Record`
2. **Design de interface**: Orientar a criação de páginas e formulários com componentes corretos (RecordList, Chart, Record viewer/editor) respeitando as limitações e padrões do Corteza
3. **Geração de YAML**: Produzir YAML válido no formato de namespace Corteza (compatível com `Case management.yaml`) para importação direta na plataforma
4. **Qualidade de app**: Validar coerência entre módulos, páginas e gráficos antes da geração final

## Non-Goals (Out of Scope)

- Geração de workflows/automações (não contemplado nesta fase)
- Interface gráfica ou UI própria (funciona via prompts no Claude Code)
- Integração direta com API do Corteza (apenas geração de YAML local)

## Users

Desenvolvedor (Erick) usando Claude Code para projetar e gerar aplicações Corteza Compose. A skill precisa ser suficientemente detalhada para que o assistente de IA possa guiar todo o processo sem conhecimento prévio do Corteza.

## Constraints

- A skill deve seguir o formato SKILL.md existente no projeto (frontmatter YAML + instruções Markdown)
- A skill vai para `.agent/skills/corteza-app-builder/SKILL.md`
- Workflow de uso: dentro do Claude Code, o dev invoca a skill para orientar a criação de um novo app
- O YAML gerado deve ser compatível com o Corteza Compose (versão do exemplo: 2026-01)

## Success Criteria

- [ ] A skill guia corretamente a definição de módulos com todos os tipos de campo suportados pelo Corteza
- [ ] A skill orienta os relacionamentos entre módulos via campos `Record` com `labelField` e `queryFields`
- [ ] A skill orienta a criação de páginas com componentes RecordList, Record editor e Chart
- [ ] O YAML gerado é sintaticamente válido e importável no Corteza sem erros
- [ ] A skill inclui exemplos concretos baseados no `Case management.yaml` como referência
