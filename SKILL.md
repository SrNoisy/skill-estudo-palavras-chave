# Estudo de Palavras-Chave Google Ads

Pesquisa de keywords com integração NotebookLM + Google Keyword Planner.
**Self-contained** para qualquer agente de IA — usa apenas `npx` + comandos shell.

## Pré-requisitos

| Ferramenta | Como verificar | Onde baixar |
|------------|---------------|-------------|
| **Node.js 18+** | `node --version` | https://nodejs.org (LTS) |
| **npm** | `npm --version` | Vem com Node.js |
| **Chrome/Chromium** | Abrir o navegador | https://www.google.com/chrome/ |

## Setup Único (primeira execução)

### 1. Instalar notebooklm-mcp

```bash
npm install -g notebooklm-mcp
```

> Alternativa (sem global): usar `npx -y notebooklm-mcp` em todos os comandos

### 2. Autenticar no Google

```bash
notebooklm-mcp setup_auth --show_browser true
```

Se Chrome não abrir automaticamente:
```bash
HEADLESS=false notebooklm-mcp setup_auth --show_browser true
```

### 3. Verificar

```bash
notebooklm-mcp get_health
# Esperado: { authenticated: true, healthy: true }
```

### 4. (Opcional) Registrar como MCP server

Dependendo da ferramenta que você usa, adicione ao arquivo de configuração MCP:

**OpenCode** (`opencode.json`):
```json
{
  "mcp": {
    "notebooklm": {
      "type": "local",
      "command": "npx",
      "args": ["notebooklm-mcp@latest"]
    }
  }
}
```

**Claude Code** (`claude.json`):
```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "npx",
      "args": ["-y", "notebooklm-mcp@latest"]
    }
  }
}
```

**Codex** (plugin MCP): configurar via interface ou `~/.codex/mcp.json`

Se preferir não configurar, use os comandos `npx -y notebooklm-mcp` diretamente — funciona em qualquer ferramenta.

---

## Fluxo de Trabalho

### Passo 1 — Obter Link do Notebook

Perguntar ao usuário:
> **"Qual o link do NotebookLM do cliente?"**

Extrair o `notebook_id` da URL:
```
https://notebooklm.google.com/notebook/{notebook_id}
```

Registrar e selecionar:
```bash
npx -y notebooklm-mcp add_notebook --url "https://notebooklm.google.com/notebook/{notebook_id}"
npx -y notebooklm-mcp select_notebook --id "{notebook_id}"
```

### Passo 2 — Entrevistar o NotebookLM

Fazer **3 rodadas de perguntas** e consolidar.

**Rodada 1 — ICP + Produtos + Concorrentes**
```bash
npx -y notebooklm-mcp ask_question \
  --question "Descreva o perfil do cliente ideal (porte, setor, cargo decisor, ticket médio, volume mínimo por pedido). Liste todos os produtos/serviços oferecidos, destacando o carro-chefe e produtos com nome proprietário. Quem são os concorrentes diretos? Quais os diferenciais competitivos?" \
  --source_format "footnotes"
```

**Rodada 2 — Dores + Sazonalidade + Segmentos**
```bash
npx -y notebooklm-mcp ask_question \
  --question "Por que os clientes compram da empresa? Qual problema foi resolvido? O que eles usavam antes? Quais objeções comuns aparecem? Quais épocas do ano têm mais vendas? Qual o lead time típico do pedido à entrega? Quais segmentos/indústrias são atendidos? A empresa atende nacionalmente ou regiões específicas? Quais canais de venda?" \
  --source_format "footnotes"
```

**Rodada 3 — Prova Social + Diferenciais**
```bash
npx -y notebooklm-mcp ask_question \
  --question "Liste clientes-prova ou grandes marcas atendidas. Existem cases de sucesso ou parcerias relevantes? Quais certificações ou diferenciais técnicos a empresa possui?" \
  --source_format "footnotes"
```

#### Formato Consolidado

```markdown
## Contexto do Cliente

**ICP:** {B2B/B2C, porte, ticket, decisor}
**Produtos:** {lista com carro-chefe em destaque}
**Dores:** {problema resolvido, substituto, objeções}
**Concorrentes:** {lista de marcas}
**Diferenciais:** {por que escolher essa empresa}
**Sazonalidade:** {picos e lead time}
**Segmentos:** {indústrias atendidas}
**Canais:** {canais de venda}
**Prova Social:** {clientes-prova, cases}
```

### Passo 3 — Gerar CSV de Palavras-Chave

**Formato:** CSV com header `Keyword`, uma palavra por linha.

**Critérios:**
- Mínimo **100 palavras** por cliente
- Agrupar por comentários (`# Grupo`)
- Cobrir: genérico, produto, nicho, segmento, sazonalidade, concorrente (patrocínio), cross-sell

**Estrutura:**
```
# Produto Principal
produto personalizado
...

# Linhas Específicas
produto linha1 personalizado
...

# Segmentos
produto para segmento
...

# Nichos
produto nicho personalizado
...

# Concorrentes (Patrocínio)
concorrente produto personalizado
...

# Cross-sell
outro produto personalizado
...
```

Salvar em: `{cliente}/input/palavras-chave.csv`

### Passo 4 — Instruir Usuário

> **Instruções:**
> 1. Acesse o **Google Keyword Planner**
> 2. **"Descobrir novas palavras-chave"** > **"Começar com palavras-chave"**
> 3. Upload do arquivo `{cliente}/input/palavras-chave.csv`
> 4. Local: **Brasil**, Rede: **Google Search**
> 5. **"Obter resultados"**
> 6. **"Métricas históricas do plano"** > exportar como **CSV**
> 7. Colocar em `{cliente}/input/`

Aguardar o usuário.

### Passo 5 — Gerar Relatório HTML

#### Ler CSV exportado

Converter de UTF-16 se necessário:
```bash
iconv -f UTF-16 -t UTF-8 "{cliente}/input/Keyword Stats *.csv" 2>/dev/null
```

#### Estrutura

| Página | Conteúdo |
|--------|----------|
| **1** | Visão Geral: ICP, forecast, top keywords, calendário sazonal |
| **2** | Arquitetura de campanhas |
| **3-4** | Palavras por grupo + match type |
| **5** | Negativas + recomendações |
| **6** | Stats: Buscas/mês, Concorrência, CPC, Tendência |

#### Design

Identidade **Billions Intelligence (Red Command Center)**:
- Paleta: vermelho (#FF2A1A, #A10F14, #3B0000), branco, dourado (#FFD48A), amarelo (#FFF200)
- Tipografia: IBM Plex Sans (Google Fonts)
- Glassmorphism + radial gradients + grid pattern
- Tabelas com header dourado
- Tags: exact (verde), phrase (dourado), broad (vermelho)

CSS inline — copiar tokens de `references/billions-tokens.css`.

**Salvar em:** `{cliente}/output/index.html`

**Abrir:**
```bash
open "{cliente}/output/index.html"
```

---

## Estrutura de Pastas

```
{cliente}/
├── input/
│   ├── palavras-chave.csv
│   └── Keyword Stats *.csv
├── {cliente} - Estudo de Palavras-Chave Google Ads.md
└── output/
    └── index.html
```

---

## Formatos

### CSV Palavras (input)
```csv
Keyword
palavra personalizada
```

### CSV Planner (exportado, UTF-16)
```
Keyword | Avg. monthly searches | Competition | Top of page bid (low/high) | Three month change
```

Sempre converter:
```bash
iconv -f UTF-16 -t UTF-8 "arquivo.csv" 2>/dev/null
```

---

## Referências

- `references/billions-tokens.css` — Design system para o relatório HTML
