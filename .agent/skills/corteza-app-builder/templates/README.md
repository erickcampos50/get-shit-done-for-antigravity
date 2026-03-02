# Templates — Índice de Referência Rápida

Snippets YAML prontos para montar apps Corteza. Copie e substitua os placeholders em `[MAIÚSCULAS]`.

## Arquivos

| Arquivo | Uso | Placeholders principais |
|---------|-----|------------------------|
| `module.yaml` | Template de módulo completo | `[HANDLE]`, `[NOME]`, `[NAMESPACE-SLUG]` |
| `field-types.yaml` | Catálogo de todos os 10 tipos de campo | qualquer campo como ponto de partida |
| `page-list.yaml` | Página de lista (aparece no menu) | `[MODULO]`, `[HANDLE-LISTA]`, `[CAMPOS]` |
| `page-detail.yaml` | Página de detalhe de um registro | `[MODULO]`, `[HANDLE-LISTA]`, `[FK-CAMPO]` |
| `chart.yaml` | 4 tipos de gráfico prontos | `[MODULO]`, `[CAMPO-DIMENSAO]`, `[HANDLE]` |
| `namespace.yaml` | Esqueleto completo de namespace | todos os placeholders acima |

## Convenções de Nomenclatura

| Elemento | Padrão | Exemplo |
|----------|--------|---------|
| `namespace.slug` | kebab-case | `gestao-clientes` |
| `module.handle` | PascalCase | `OrdemDeServico` |
| `page.handle` (lista) | PascalCase | `OrdensDeServico` |
| `page.handle` (detalhe) | `RecordPageForModule` + handle | `RecordPageForModuleOrdemDeServico` |
| `chart.handle` | PascalCase descritivo | `OSPorStatus` |
| `moduleField.name` | CamelCase | `DataDeAbertura` |
| `moduleField.label` | Legível em PT-BR | `Data de Abertura` |

## Regras Críticas

1. **Ordem no YAML**: `chart:` → `module:` (referenciados antes dos que referenciam) → `page:` (listas antes de detalhes)
2. **IDs**: Nunca use UUIDs numéricos nos templates — use handles ou `0`. O Corteza atribui IDs na importação.
3. **`blockid` vs `blockID`**: Minúsculo (`blockid`) nos blocos; maiúsculo (`blockID`) no `pageLayout`.
4. **`xywh` nos blocos**: Sempre `[0, 0, 0, 0]`. As posições reais ficam no `pageLayout`.
5. **`config:` de módulo e campo**: Nunca omita. O Corteza rejeita YAML sem `dal`, `privacy`, `expressions`.
