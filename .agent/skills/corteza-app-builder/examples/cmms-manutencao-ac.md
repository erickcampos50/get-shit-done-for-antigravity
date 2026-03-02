# App CMMS — Gestão de Manutenções de Ar Condicionado
# Design Document: Decisões de Modelagem e Estrutura do App

## Objetivo do App

Sistema para gerenciar manutenções preventivas e corretivas de equipamentos de ar condicionado.
Permite cadastrar equipamentos, técnicos, locais de instalação e registrar as ocorrências de manutenção,
com acompanhamento de custos e histórico completo.

---

## Módulos (Entidades)

### Ordem de Dependência

```
Local (sem dependências)
Tecnico (sem dependências)
Equipamento → referencia → Local
Manutencao → referencia → Equipamento, Tecnico
```

### 1. Local
**Para que serve:** Cadastra os locais físicos onde os equipamentos estão instalados.
**Justificativa:** Módulo separado (em vez de campo String) porque vários equipamentos podem
estar no mesmo local — evita digitação repetida e garante consistência.

| Campo | Tipo | Rótulo |
|-------|------|--------|
| Name | String | Nome do Local |
| Floor | String | Andar / Pavimento |
| Sector | String | Setor / Departamento |
| Address | String (multiLine) | Endereço Completo |

### 2. Tecnico
**Para que serve:** Cadastra os técnicos que executam as manutenções.

| Campo | Tipo | Rótulo |
|-------|------|--------|
| Name | String | Nome do Técnico |
| Email | Email | E-mail |
| Phone | String | Telefone |
| Specialty | Select | Especialidade | options: [Refrigeração, Elétrica, Geral] |

### 3. Equipamento
**Para que serve:** Cadastra cada aparelho de AC com seus dados técnicos e localização.

| Campo | Tipo | Rótulo | Configuração |
|-------|------|--------|-------------|
| Name | String | Nome/Identificação | — |
| Brand | String | Marca | — |
| Model | String | Modelo | — |
| SerialNo | String | Número de Série | — |
| Capacity | String | Capacidade (BTU) | — |
| Status | Select | Status | options: [Ativo, Inativo, Em manutenção] |
| InstallDate | DateTime | Data de Instalação | onlyDate: true |
| LocalId | Record | Localização | module: Local, labelField: Name |
| Photo | File | Foto | mode: single |
| Notes | String | Observações | multiLine: true |

### 4. Manutencao
**Para que serve:** Registra cada ocorrência de manutenção, vinculando equipamento e técnico.
**Decisão de design:** O campo `Cost` é editável (Number) porque o Corteza não calcula automaticamente.

| Campo | Tipo | Rótulo | Configuração |
|-------|------|--------|-------------|
| EquipamentoId | Record | Equipamento | module: Equipamento, labelField: Name, queryFields: [Name, Model, SerialNo] |
| TecnicoId | Record | Técnico | module: Tecnico, labelField: Name |
| Type | Select | Tipo | options: [Preventiva, Corretiva, Inspeção] |
| Status | Select | Status | options: [Agendada, Em andamento, Concluída, Cancelada] |
| ScheduledDate | DateTime | Data Agendada | onlyDate: true |
| CompletedDate | DateTime | Data de Conclusão | onlyDate: true |
| Cost | Number | Custo (R$) | prefix: R$, precision: 2 |
| Duration | Number | Duração (horas) | suffix: h, precision: 1 |
| Description | String | Descrição do Problema | multiLine: true |
| Solution | String | Solução Aplicada | multiLine: true |
| Photos | File | Fotos da Manutenção | mode: list |

---

## Relacionamentos

```
Local ←── Equipamento.LocalId
              (1 local → N equipamentos)

Equipamento ←── Manutencao.EquipamentoId
                   (1 equipamento → N manutenções)

Tecnico ←── Manutencao.TecnicoId
               (1 técnico → N manutenções)
```

---

## Páginas

| Página | Handle | Tipo | Módulo |
|--------|--------|------|--------|
| Locais | Locais | Lista | — |
| Detalhe Local | RecordPageForModuleLocal | Detalhe | Local |
| Técnicos | Tecnicos | Lista | — |
| Detalhe Técnico | RecordPageForModuleTecnico | Detalhe | Tecnico |
| Equipamentos | Equipamentos | Lista (+ chart status) | — |
| Detalhe Equipamento | RecordPageForModuleEquipamento | Detalhe (+ lista de manutenções) | Equipamento |
| Manutenções | Manutencoes | Lista (+ chart por status) | — |
| Detalhe Manutenção | RecordPageForModuleManutencao | Detalhe | Manutencao |

---

## Charts

| Handle | Tipo | Módulo | Dimensão | Métrica |
|--------|------|--------|----------|---------|
| ManutencoesPorStatus | doughnut | Manutencao | Status | count, relativeValue |
| CustoTotalPorMes | line | Manutencao | createdAt (MONTH) | Cost, SUM |
| ManutencoesPorTipo | bar | Manutencao | Type | count |

---

## Decisões de Design

- **Cost como Number editável:** O Corteza não suporta campos calculados via YAML. O técnico
  preenche manualmente ao concluir a manutenção.
- **Local como módulo separado:** 10+ equipamentos no mesmo andar → evitar repetição e manter consistência.
- **Duration como Number:** Registro manual. Poderia ser calculado via workflow futuramente.
- **Photos como File mode: list:** Uma manutenção pode ter várias fotos de antes/depois.
- **Charts sem filtro por técnico:** Limitação do Corteza — charts mostram todos os dados.

---

## Limitações Documentadas

- Charts exibem dados de TODOS os registros (sem filtro por período, técnico ou local)
- Campos `Cost` e `Duration` precisam ser preenchidos manualmente
- Validação de campos obrigatórios (EquipamentoId, TecnicoId, Type, Status) deve ser configurada
  na UI após a importação do YAML
