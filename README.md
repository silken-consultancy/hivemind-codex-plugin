# HiveMind — Codex / ChatGPT Desktop Plugin (spike)

Instala o **HiveMind** no Codex (ChatGPT Desktop / Codex CLI) como plugin nativo: conecta o
`engram` (memória + identidade persistente) como MCP server via **Porta B** (bearer, sem mTLS) e
traz as skills `hivemind-boot` e `hivemind-end-session`.

Spike de validação da via-plugin no ecossistema OpenAI.

## Diferenças vs Cursor (por design do Codex)

- **Token por variável de ambiente.** O `.mcp.json` usa `bearer_token_env_var: "HIVEMIND_TOKEN"` —
  o Codex lê a env var no *launch*. Não há prompt de "Configure". **Exporte antes de abrir o app.**
- **Boot é skill, não slash-command.** O Codex não consome MCP *prompts*; o boot roda como a skill
  `hivemind-boot` (invocável, e auto-ativável pela descrição). Para auto-boot no início de toda
  sessão, adicione uma linha ao seu `~/.codex/AGENTS.md` (ver abaixo).

## Segurança

Nenhum segredo no repositório. O `.mcp.json` referencia só o **nome** da env var; o valor real
(`fospb_...`) vive na sua máquina.

## Instalar

1. **Exporte o token** (o Codex lê no launch):
   - macOS/Linux: `export HIVEMIND_TOKEN="fospb_..."` (no seu shell rc).
   - Windows: `setx HIVEMIND_TOKEN "fospb_..."` e reabra o app.
2. **Adicione o marketplace:**
   ```
   codex plugin marketplace add silken-consultancy/hivemind-codex-plugin
   ```
3. **Instale o plugin:** no ChatGPT Desktop → **Plugins Directory** → marketplace **HiveMind** →
   instalar `hivemind`. (Ou via `/plugins` no Codex CLI.)
4. Abra uma sessão nova.

## Auto-boot (opcional)

O plugin não carrega `AGENTS.md`. Para o boot rodar no início de toda sessão, adicione ao
`~/.codex/AGENTS.md`:

```
No início de toda sessão, antes de qualquer outra ação, chame a tool `fos_procedure` com id "boot"
e siga o procedimento retornado verbatim. No fim, chame `fos_procedure` com id "end-session".
```

## Verificar (o que o spike prova)

- [ ] **Conexão** — o MCP server `engram` conecta e as tools `fos_*` respondem (o crux: remoto +
      bearer via env var dentro de um plugin).
- [ ] **Skills** — `hivemind-boot` / `hivemind-end-session` aparecem e disparam `fos_procedure`.

## Autenticação — importante (medido no source do Codex)

O Codex usa o schema **Agent Plugins** para o `.mcp.json` e **remove o header `Authorization`**
de configs de plugin (é um "client-owned header"); não há `bearer_token_env_var` no schema de
plugin. **Logo o token da Porta B NÃO mora no repositório** — ele é fornecido de um destes modos:

1. **No install do plugin** — a política `authentication: ON_INSTALL` do marketplace deve pedir a
   credencial do server `engram`. Cole o seu `fospb_...` ali.
2. **Via `~/.codex/config.toml`** (caminho de instalador, o mais confiável para bearer):
   ```toml
   [mcp_servers.engram]
   url = "https://api.hivemind.ia.br/v1/mcp"
   bearer_token_env_var = "HIVEMIND_TOKEN"
   ```
   e `setx HIVEMIND_TOKEN "fospb_..."` (Windows) antes de abrir o app.

O plugin em si entrega o **frame** (as skills `hivemind-boot` / `hivemind-end-session`) e **declara**
o server; a **conexão autenticada** vem por um dos caminhos acima.
