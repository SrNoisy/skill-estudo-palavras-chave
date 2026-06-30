# Estudo de Palavras-Chave Google Ads

Pesquisa de keywords com integração NotebookLM + Google Keyword Planner.
Skill auto-contida para qualquer agente de IA (OpenCode, Claude Code, Codex).

![License](https://img.shields.io/github/license/SrNoisy/skill-estudo-palavras-chave?style=for-the-badge)
![Stars](https://img.shields.io/github/stars/SrNoisy/skill-estudo-palavras-chave?style=for-the-badge)
![Node](https://img.shields.io/badge/Node.js-18%2B-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![NotebookLM](https://img.shields.io/badge/NotebookLM-FFD48A?style=for-the-badge&logo=google&logoColor=black)

---

## O que faz

1. Extrai ICP, produtos, concorrentes e dores do cliente via **NotebookLM**
2. Gera CSV de **100+ palavras-chave** agrupadas por categoria
3. Usuário exporta métricas do **Google Keyword Planner**
4. Gera **relatório HTML** com identidade Billions e dados enriquecidos

---

## Estrutura

```
├── SKILL.md                    # Instruções do fluxo de 5 passos
├── references/
│   └── billions-tokens.css     # Design system Billions (Red Command Center)
└── assets/
    └── template-base.html      # Template HTML base para relatório
```

---

## Requisitos

- **Node.js 18+** — https://nodejs.org
- **Chrome/Chromium** — autenticação NotebookLM
- **notebooklm-mcp** — `npm install -g notebooklm-mcp` (ou via `npx -y`)

## Setup

```bash
# Instalar dependência
npm install -g notebooklm-mcp

# Autenticar no Google
notebooklm-mcp setup_auth --show_browser true

# Verificar
notebooklm-mcp get_health
# → { authenticated: true, healthy: true }
```

## Fluxo

| Passo | O que acontece |
|-------|---------------|
| 1 | Usuário fornece link do NotebookLM do cliente |
| 2 | Agente entrevista o notebook (3 rodadas de perguntas) |
| 3 | Agente gera CSV de palavras-chave |
| 4 | Usuário exporta métricas do Keyword Planner |
| 5 | Agente gera relatório HTML com forecast e stats |

---

## Licença

MIT — use, modifique, distribua.
