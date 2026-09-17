# HiveMind — Codex / ChatGPT Desktop Plugin (skills)

Instala as **skills** do HiveMind no Codex (ChatGPT Desktop / Codex CLI): `hivemind-boot` e
`hivemind-end-session`. Cada skill é um **stub fino** que busca o procedimento **servido pelo
engram** (config-as-data) via a tool `fos_procedure` e o executa verbatim — então o vessel roda
sempre o boot/close canônico atual, nunca uma cópia forkada ou obsoleta. A fonte única de verdade
vive no engram.

> **Este plugin entrega só o FRAME (as skills).** A conexão MCP com o engram é adicionada
> **manualmente** (passo 1). Isso é por design do Codex: um plugin não pode carregar a credencial
> de um MCP remoto — o header `Authorization` é descartado do `.mcp.json` de plugin, e o manifesto
> não tem campo de token/variável. Então: **plugin = skills; conexão = config.toml.**

O Codex é um **vessel estrangeiro**: não é lançado pelo `bin/hivemind`, não tem `SessionEnd` hook
nem `ENGRAM_SESSION_ID` no ambiente. Por isso as duas skills **são donas do ciclo de vida da
sessão**: `hivemind-boot` **abre** a sessão (via o corpo servido) e `hivemind-end-session` a
**fecha** com um `next_note` de continuidade garantido. Rodar o end-session não é opcional — é o
que evita órfãs e preserva o handoff para o próximo boot.

## 1. Conectar o engram (MCP) — manual, uma vez

Copie o bloco de [`config.toml.example`](./config.toml.example) para o FINAL do seu `config.toml`
do Codex (Windows: `%USERPROFILE%\.codex\config.toml`; macOS/Linux: `~/.codex/config.toml`):

```toml
[mcp_servers.engram]
url = "https://api.hivemind.ia.br/v1/mcp"
bearer_token_env_var = "HIVEMIND_TOKEN"
```

E defina o token numa variável de ambiente — **no TERMINAL, não no arquivo**:

- Windows: `setx HIVEMIND_TOKEN "fospb_YOUR_TOKEN_HERE"` → depois **feche e reabra** o app.
- macOS/Linux: `export HIVEMIND_TOKEN="fospb_YOUR_TOKEN_HERE"` no seu shell rc.

O Codex **rejeita um token inline** (`bearer_token = "..."`); a chave suportada é
`bearer_token_env_var`, cujo valor é o **nome** da variável de ambiente. Ou seja: o `config.toml`
guarda só o **nome** da variável; o token (`fospb_...`) vive na env var, **nunca** no arquivo. Gere
o token na dashboard do HiveMind.

Alternativa via CLI (faz a mesma edição):

```
codex mcp add engram --url https://api.hivemind.ia.br/v1/mcp --bearer-token-env-var=HIVEMIND_TOKEN
```

## 2. Instalar as skills (este plugin)

ChatGPT Desktop → **Plugins → Adicionar → Adicionar Marketplace** → informe o repositório
`silken-consultancy/hivemind-codex-plugin` → instale o plugin **hivemind**.

## Verificar

- [ ] as tools `fos_*` respondem (conexão via `config.toml`);
- [ ] a skill `hivemind-boot` roda `fos_procedure(id:"boot")` e o corpo servido **abre a sessão**;
- [ ] ao encerrar, `hivemind-end-session` roda `fos_procedure(id:"end-session")` e fecha com `next_note`.

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

## Solução de problemas

- **O server `engram` não conecta / as tools `fos_*` não aparecem:** confirme que o token está na
  env var **antes** de abrir o Codex (no Windows, `setx` só vale em processos novos — reabra o app).
- **`url` parece ignorado:** seu Codex é antigo. Atualize-o. Em builds pré-out/2025 o MCP remoto via
  HTTP ficava atrás de `experimental_use_rmcp_client = true` (topo do `config.toml`) — hoje deprecado
  e desnecessário num Codex atual.

---
_Parte do arco do conector HiveMind. Path A (mTLS) é Enterprise-only; a conexão usa a Porta B (bearer)._
