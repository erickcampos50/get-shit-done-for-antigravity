# App Helpdesk de TI — Design Document
# Gerado seguindo o Modo Conversacional da skill corteza-app-builder

## Objetivo do App

Sistema de registro e acompanhamento de chamados de suporte de TI internos.
Permite que colaboradores abram tickets, técnicos os assumam e resolvam, e gestores acompanhem métricas.

---

## Módulos (Entidades)

### Ordem de Dependência

```
Solicitante (sem dependências)
Tecnico (sem dependências)
Chamado → referencia → Solicitante + Tecnico
```

### 1. Solicitante
**Para que serve:** Cadastra os colaboradores que abrem chamados de suporte.
**Decisão:** Módulo separado (em vez de campo User do Corteza) para flexibilidade de cadastro
sem depender de conta ativa no sistema.

| Campo | Tipo | Rótulo | Config |
|-------|------|--------|--------|
| Name | String | Nome Completo | — |
| Email | Email | E-mail Corporativo | — |
| Department | String | Departamento | — |
| Extension | String | Ramal | — |

### 2. Tecnico
**Para que serve:** Técnicos da equipe de TI responsáveis por resolver os chamados.

| Campo | Tipo | Rótulo | Config |
|-------|------|--------|--------|
| Name | String | Nome do Técnico | — |
| Email | Email | E-mail | — |
| Specialty | Select | Especialidade | options: [Hardware, Software, Redes, Geral] |

### 3. Chamado
**Para que serve:** O ticket de suporte central do sistema — relaciona solicitante, técnico e o problema.

| Campo | Tipo | Rótulo | Config |
|-------|------|--------|--------|
| SolicitanteId | Record | Solicitante | module: Solicitante, labelField: Name, queryFields: [Name, Email] |
| TecnicoId | Record | Técnico Responsável | module: Tecnico, labelField: Name, queryFields: [Name, Specialty] |
| Title | String | Título do Problema | — |
| Category | Select | Categoria | options: [Hardware, Software, Redes, Acesso, Outro] |
| Priority | Select | Prioridade | options: [Crítica, Alta, Média, Baixa] |
| Status | Select | Status | options: [Aberto, Em andamento, Aguardando usuário, Resolvido, Fechado] |
| Description | String | Descrição do Problema | multiLine: true |
| Resolution | String | Solução Aplicada | multiLine: true |
| OpenedDate | DateTime | Data de Abertura | onlyDate: true |
| ResolvedDate | DateTime | Data de Resolução | onlyDate: true |
| SatisfactionScore | Select | Satisfação | options: [★★★★★, ★★★★, ★★★, ★★, ★] |

**Decisão:** `SatisfactionScore` como `Select` com 5 opções textuais — não `Number`.
Rationale: é um conjunto fixo de 5 valores conhecidos → regra "se opções fixas, use Select".

---

## Relacionamentos

```
Solicitante ←── Chamado.SolicitanteId
                  (1 solicitante → N chamados)

Tecnico ←── Chamado.TecnicoId
               (1 técnico → N chamados)
```

---

## Páginas

| Página | Handle | Tipo | Conteúdo |
|--------|--------|------|----------|
| Solicitantes | Solicitantes | Lista | RecordList |
| Detalhe Solicitante | RecordPageForModuleSolicitante | Detalhe | Record + RecordList de chamados do solicitante |
| Técnicos | Tecnicos | Lista | RecordList |
| Detalhe Técnico | RecordPageForModuleTecnico | Detalhe | Record + RecordList de chamados do técnico |
| Chamados | Chamados | Lista | RecordList + 2 Charts |
| Detalhe Chamado | RecordPageForModuleChamado | Detalhe | 2 blocos Record |

---

## Charts

| Handle | Tipo | Módulo | Dimensão | Métrica |
|--------|------|--------|----------|---------|
| ChamadosPorStatus | doughnut | Chamado | Status | count, relativeValue |
| ChamadosPorCategoria | bar | Chamado | Category | count |
| ChamadosPorTecnico | bar | Chamado | TecnicoId | count |

**Nota:** `ChamadosPorTecnico` agrupa pelo campo `TecnicoId` (Record) — o Corteza exibe
o `labelField` (Name do Técnico) no eixo X, o que é o comportamento correto.

---

## Decisões de Design

- **SatisfactionScore como Select:** Conjunto fixo de 5 valores → Select é mais correto que Number
- **ResolvedDate editável:** O Corteza não preenche automaticamente — técnico preenche ao resolver
- **TecnicoId como módulo próprio:** Flexibilidade de cadastrar técnicos sem conta ativa no Corteza
- **Chamados sem prefilter global:** A lista principal mostra todos os chamados (sem filtro por técnico)

---

## Limitações Documentadas

- Charts mostram dados de todos os chamados (sem filtro por técnico logado)
- `SatisfactionScore` precisa ser preenchido manualmente após resolução
- `ResolvedDate` não é calculado automaticamente
