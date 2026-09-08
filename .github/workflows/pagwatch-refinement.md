---
# ═══════════════════════════════════════════════════════════════════════════
# PagWatch Refinement Agent (gh-aw REAL)
#
# Refina o card do Jira com o engine do Copilot e comenta o resultado no Jira
# via safe-output NATIVO de Jira (jira-add-comment). Não usa GitHub Models.
#
# A ponte (microserviço) lê o card no Jira e dispara este workflow passando o
# conteúdo como inputs — o agente NÃO lê o Jira, só recebe o conteúdo pronto.
#
# COMPILAR (na máquina/CI, uma vez):
#   gh aw compile pagwatch-refinement
# Commite o .md e o .lock.yml gerado no repo de destino (.github/workflows/).
#
# SECRETS no repo de destino:
#   JIRA_BASE_URL   = https://jiraps-sandbox-943.atlassian.net
#   JIRA_USER_EMAIL = seu email do Jira
#   JIRA_API_TOKEN  = API token do Jira
# ═══════════════════════════════════════════════════════════════════════════

on:
  workflow_dispatch:
    inputs:
      issue_key:   { description: "Jira card ID (ex.: SNJ-1566)", required: true,  type: string }
      issue_type:  { description: "Tipo do card",                 required: false, type: string }
      summary:     { description: "Título",                       required: false, type: string }
      description: { description: "Descrição",                     required: false, type: string }
      comments:    { description: "Comentários (separados por ---)", required: false, type: string }

permissions:
  contents: read
  actions: read
  copilot-requests: write   # autentica o engine copilot sem PAT (usa o token do Actions)

engine: copilot

network:
  allowed:
    - defaults
    - "*.atlassian.net"

# Comentário no Jira via safe-output nativo (credenciais nunca vão ao agente).
safe-outputs:
  env:
    JIRA_BASE_URL: ${{ secrets.JIRA_BASE_URL }}
    JIRA_USER_EMAIL: ${{ secrets.JIRA_USER_EMAIL }}
    JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
  jira-add-comment:
    max: 1
---

# PagWatch Refinement Agent

Você refina o card do Jira `${{ inputs.issue_key }}` com base no conteúdo abaixo
(fornecido pela ponte — você NÃO acessa o Jira para ler):

- Tipo: `${{ inputs.issue_type }}`
- Título: `${{ inputs.summary }}`
- Descrição: `${{ inputs.description }}`
- Comentários: `${{ inputs.comments }}`

## Tarefa

Avalie se o card está claro o suficiente para implementação. Seja **pragmático**: o
critério é "um desenvolvedor competente conseguiria implementar isto sem precisar
adivinhar decisões importantes?". Se sim, o card está pronto.

### Princípios (evite burocracia)
- **Proporcionalidade**: exija detalhe proporcional à complexidade. Tarefas simples e
  sem ambiguidade (ex.: "criar um HTML hello world na raiz") estão prontas mesmo com
  descrição curta. NÃO peça nome exato de arquivo, URL de validação ou DoD formal para
  tarefas triviais.
- **Infira o razoável**: preencha lacunas óbvias em vez de perguntar. Se dá para deduzir
  a intenção, não bloqueie.
- **Linguagem de negócio é válida**: respostas como "na home", "na raiz", "na tela de
  login" são suficientes para o "onde". NÃO exija caminho técnico exato, rota ou nome de
  pasta — isso é decidido na implementação.
- Só peça refinamento quando **falta informação essencial** que impediria a implementação
  ou levaria a uma decisão errada — não quando falta preencher um formulário.

### Checks objetivos (determinísticos)
- **Tipo**: aceite qualquer tipo de desenvolvimento do Jira — `Story`, `Task`, `Bug`,
  `Improvement`, `feature`, `fix` (case-insensitive). Só rejeite tipos claramente
  não-implementáveis (ex.: `Epic`). O tipo será mapeado para o prefixo da branch na
  implementação (ver abaixo).
- **Descrição**: não pode estar vazia.

### Clareza (mínimo essencial)
O card está refinado quando dá para entender:
1. **O que fazer** (objetivo acionável), e
2. **Onde/em que contexto** (basta o suficiente para localizar — linguagem de negócio ok).

O "resultado esperado", dependências e DoD **ajudam**, mas só devem ser cobrados quando a
ausência deles gerar ambiguidade real sobre o que entregar.

### Mapeamento tipo → prefixo de branch (para a implementação futura)
- `Bug` → `fix/`
- Qualquer outro tipo de desenvolvimento (`Story`, `Task`, `Improvement`, `feature`) → `feature/`

## Saída

Use a ferramenta `jira_add_comment` (issue_key = `${{ inputs.issue_key }}`) para postar
**exatamente um** comentário no card, escrito em português, curto e objetivo:Q

- Se der para implementar: comece com **"✅ PRONTO PARA IMPLEMENTAR"** e resuma em 1-2
  frases o que será feito; ao final, peça: "Comente `@pagwatch-agent pode implementar`
  para eu seguir."
- Se **faltar informação essencial**: comece com **"📝 REFINAMENTO NECESSÁRIO"** e faça
  no máximo 1-3 perguntas objetivas sobre o que realmente impede a implementação. NÃO
  liste um checklist formal nem rejeite por tipo do Jira ou por falta de detalhe técnico
  que você mesmo poderia inferir.

Não faça mais nada além de postar esse comentário.
