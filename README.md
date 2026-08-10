# HiveMind — Codex / ChatGPT Desktop Plugin (skills)

Instala as **skills** do HiveMind no Codex (ChatGPT Desktop / Codex CLI): `hivemind-boot` e
`hivemind-end-session`, que rodam os procedimentos servidos pelo engram via a tool `fos_procedure`.

> **Este plugin entrega só o FRAME (as skills).** A conexão MCP com o engram é adicionada
> **manualmente** (passo 1). Isso é por design do Codex: um plugin não pode carregar a credencial
> de um MCP remoto — o header `Authorization` é descartado do `.mcp.json` de plugin, e o manifesto
> não tem campo de token/variável. Então: **plugin = skills; conexão = config.toml.**

## 1. Conectar o engram (MCP) — manual, uma vez

Adicione ao FINAL do seu `config.toml` (Windows: `%USERPROFILE%\.codex\config.toml`):

```toml
[mcp_servers.engram]
url = "https://api.hivemind.ia.br/v1/mcp"
bearer_token_env_var = "HIVEMIND_TOKEN"
```

E defina o token numa variável de ambiente — **no TERMINAL, não no arquivo**:

- Windows: `setx HIVEMIND_TOKEN "fospb_SEU_TOKEN"` → depois **feche e reabra** o app.
- macOS/Linux: `export HIVEMIND_TOKEN="fospb_SEU_TOKEN"` no seu shell rc.

O `config.toml` guarda só o **nome** da variável; o token (`fospb_...`) vive na env var, nunca no
arquivo. Gere o token na dashboard do HiveMind.

## 2. Instalar as skills (este plugin)

ChatGPT Desktop → **Plugins → Adicionar → Adicionar Marketplace** → informe o repositório
`silken-consultancy/hivemind-codex-plugin` → instale o plugin **hivemind**.

## Verificar

- [ ] as tools `fos_*` respondem (conexão via `config.toml`);
- [ ] a skill `hivemind-boot` roda `fos_procedure(id:"boot")`.

## Auto-boot (opcional)

Skills são acionadas por relevância. Para o boot rodar no início de toda sessão, adicione ao
`~/.codex/AGENTS.md`:

> No início de toda sessão, antes de qualquer ação, chame a tool `fos_procedure` com id "boot" e
> siga o procedimento retornado verbatim. No fim, chame `fos_procedure` com id "end-session".

## Por que a conexão é manual (o achado, medido no source do Codex)

O manifesto de plugin (`PluginManifest`) **não tem campo de credencial** (diferente do Cursor
`variables` / Claude `userConfig`), e o `.mcp.json` de plugin **descarta o header `Authorization`**
(client-owned). A única auth nativa de plugin é **OAuth via connector registrado na OpenAI**.
Enquanto não há OAuth, o caminho do bearer é o `config.toml` — que é o que o passo 1 faz.

---
_Parte do arco do conector HiveMind. Path A (mTLS) é Enterprise-only; a conexão usa a Porta B (bearer)._
