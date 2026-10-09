# HiveMind — Codex / ChatGPT Desktop Plugin (skills)

Instala as **skills** do HiveMind no Codex (ChatGPT Desktop / Codex CLI): `hivemind-boot` e
`hivemind-end-session`. Cada skill é só um ponteiro: o procedimento de início e de fim de sessão é
**servido pelo engram**, então você roda sempre a versão atual, sem cópia local.

> **Este plugin entrega só as skills.** A conexão MCP com o engram é adicionada **manualmente**
> (passo 1): um plugin do Codex não consegue carregar a credencial de um MCP remoto. Então:
> **plugin = skills; conexão = config.toml.**

## 1. Conectar o engram (MCP) — manual, uma vez

Copie o bloco de [`config.toml.example`](./config.toml.example) para o FINAL do seu `config.toml`
do Codex (Windows: `%USERPROFILE%\.codex\config.toml`; macOS/Linux: `~/.codex/config.toml`):

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

Alternativa via CLI (faz a mesma edição):

```
codex mcp add engram --url https://api.hivemind.ia.br/v1/mcp --bearer-token-env-var=HIVEMIND_TOKEN
```

## 2. Instalar as skills (este plugin)

ChatGPT Desktop → **Plugins → Adicionar → Adicionar Marketplace** → informe o repositório
`silken-consultancy/hivemind-codex-plugin` → instale o plugin **hivemind**.

## Verificar

- [ ] o MCP server `engram` aparece conectado (conexão via `config.toml`);
- [ ] a skill `hivemind-boot` inicia a sessão com o procedimento servido pelo engram.

## Auto-boot (opcional)

Skills são acionadas por relevância. Para o boot rodar no início de toda sessão, adicione ao
`~/.codex/AGENTS.md`:

> No início de toda sessão, antes de qualquer ação, rode o procedimento de boot servido pelo MCP
> server `engram` (skill `hivemind-boot`) e siga-o exatamente. Ao encerrar, rode o procedimento de
> end-session servido por ele (skill `hivemind-end-session`).

## Solução de problemas

- **O server `engram` não conecta:** confirme que o token está na env var **antes** de abrir o
  Codex. No Windows, `setx` só vale em processos novos — feche e reabra o app.
- **macOS, app gráfico:** variáveis exportadas no shell rc (`~/.zshrc`, `~/.bashrc`) não chegam a
  apps abertos pelo Dock/Finder. Abra o app a partir de um terminal que já tenha a variável, ou
  defina-a para a sessão gráfica com `launchctl setenv HIVEMIND_TOKEN "fospb_SEU_TOKEN"` e reabra o
  app.
- **`url` parece ignorado:** seu Codex é antigo. Atualize-o. Em builds anteriores a out/2025 o MCP
  remoto via HTTP ficava atrás de `experimental_use_rmcp_client = true` (topo do `config.toml`) —
  hoje deprecado e desnecessário num Codex atual.

## Por que a conexão é manual

O manifesto de plugin do Codex **não tem campo de credencial**, e o `.mcp.json` de plugin
**descarta o header `Authorization`**. A única auth nativa de plugin é OAuth via connector
registrado na OpenAI. Enquanto não há OAuth, o caminho do token é o `config.toml` — que é o que o
passo 1 faz.
