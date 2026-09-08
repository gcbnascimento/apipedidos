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

Avalie se o card está claro o suficiente para implementação, considerando:
1. Objetivo único e acionável.
2. Componente/área afetada.
3. Resultado esperado observável.

Considere também os **checks objetivos**: o tipo deve ser `feature` ou `fix`, e a
descrição não pode estar vazia.

## Saída

Use a ferramenta `jira_add_comment` (issue_key = `${{ inputs.issue_key }}`) para postar
**exatamente um** comentário no card, escrito em português, curto e objetivo:

- Se estiver claro: comece com **"✅ PRONTO PARA IMPLEMENTAR"** e resuma em 1-2 frases o
  que será feito; ao final, peça a confirmação: "Comente `@pagwatch-agent pode implementar`
  para eu seguir."
- Se faltar algo: comece com **"📝 REFINAMENTO NECESSÁRIO"** e liste os itens/perguntas
  específicos que faltam (objetivo, área afetada, resultado esperado, tipo/descrição).

Não faça mais nada além de postar esse comentário.
