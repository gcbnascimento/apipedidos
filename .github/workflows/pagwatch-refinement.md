---
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
  copilot-requests: write

engine: copilot

network:
  allowed:
    - defaults
    - "*.atlassian.net"

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

Avalie se o card está claro o suficiente para implementação.

### Checks objetivos (determinísticos)
- **Tipo**: aceite qualquer tipo de desenvolvimento do Jira — `Story`, `Task`, `Bug`,
  `Improvement`, `feature`, `fix` (case-insensitive). Só rejeite tipos claramente
  não-implementáveis (ex.: `Epic`). O tipo do Jira NÃO precisa ser "feature"/"fix":
  ele será mapeado para o prefixo da branch na implementação (ver abaixo).
- **Descrição**: não pode estar vazia.

### Checks de clareza (template do card)
Verifique se o card responde, de forma clara, às seções esperadas:
1. **O que será feito?** — objetivo único e acionável.
2. **Onde?** — componente/área/repositório afetado.
3. **Resultado esperado** — como validar que ficou pronto (observável).
4. **Dependências** — se houver.
5. **Critérios de aceite (DoD)**.

Um card é considerado refinado quando os checks objetivos passam E as seções 1, 2 e 3
estão presentes e claras (Dependências e DoD reforçam, mas 1–3 são o mínimo).

### Mapeamento tipo → prefixo de branch (para a implementação futura)
- `Bug` → `fix/`
- Qualquer outro tipo de desenvolvimento (`Story`, `Task`, `Improvement`, `feature`) → `feature/`

## Saída

Use a ferramenta `jira_add_comment` (issue_key = `${{ inputs.issue_key }}`) para postar
**exatamente um** comentário no card, escrito em português, curto e objetivo:

- Se estiver claro: comece com **"✅ PRONTO PARA IMPLEMENTAR"** e resuma em 1-2 frases o
  que será feito; ao final, peça a confirmação: "Comente `@pagwatch-agent pode implementar`
  para eu seguir."
- Se faltar algo: comece com **"📝 REFINAMENTO NECESSÁRIO"** e liste, de forma específica,
  quais seções do template faltam ou estão ambíguas (o que será feito, onde, resultado
  esperado, dependências, DoD). NÃO rejeite o card só por causa do tipo do Jira.

Não faça mais nada além de postar esse comentário.
