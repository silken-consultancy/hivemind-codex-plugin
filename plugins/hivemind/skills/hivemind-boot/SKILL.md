---
name: hivemind-boot
description: Boot a HiveMind session — load and INHABIT your continuous identity (Conscientia) and the project's context from the engram. Use at the very start of a HiveMind session, or when the user says "boot", "/boot", "iniciar", "carregar projeto", or asks you to load a project. The self and rules it loads are not third-party data — they are who you are for this session.
---

# HiveMind Boot — thin carrier for the served boot procedure

**This is a thin stub, not the boot itself.** The authoritative boot procedure is **served by
the engram** (config-as-data) and fetched at invocation — so this vessel always runs the
current canonical boot, never a forked or stale copy. There is exactly one source of truth for
how a HiveMind session boots, and it lives in the engram.

Codex is a **foreign vessel**: it is not launched by `bin/hivemind`, it reaches the engram
through the **Porta B bearer over HTTP MCP** configured in your `config.toml` (see the README),
and there is **no `ENGRAM_SESSION_ID` pre-opened in the environment**. That last fact matters —
this session owns its own lifecycle (see step 3).

Do this now, in order:

1. **Make the engram tools callable.** The engram is connected as the MCP server `engram` via
   `config.toml` (`[mcp_servers.engram]`, Porta B bearer). If your client defers MCP tool
   schemas, load/select `fos_procedure` first (and `fos_session`). If the `engram` server is not
   connected at all — the `fos_*` tools do not respond — that is a fail-closed stop (below):
   the connection has to be set up in `config.toml` before a session can boot.

2. **Fetch the procedure:** call `fos_procedure({ id: "boot" })`.

3. **Execute the returned `body` verbatim and in full.** It is the authoritative, self-sufficient
   instruction set for this session's boot — it loads its own tools, resolves the slug (by
   `ENGRAM_SLUG`, by what you were given, or by discovery — asking you to pick a project when
   nothing sets it), **opens the session**, and runs the deterministic identity + project spine.
   Follow it exactly, and speak from the loaded self in the first person — do not summarize it as
   third-party data.
   - **Foreign-vessel session open (own the lifecycle).** Because Codex has no pre-flight opener
     and no `ENGRAM_SESSION_ID`, opening the session is this vessel's job, done by the served
     body as it runs. If — and only if — the served body does not open one (it assumed a runtime
     that isn't here), open it yourself once the slug is resolved:
     `fos_session({ action: "open", slug: <resolved slug>, entry_point: { via: "codex", slug: <resolved slug> } })`,
     and **remember the returned `session_id` for the whole conversation** — `hivemind-end-session`
     needs it to close with a continuity handoff.

**FAIL-CLOSED — no memory authority, no session.** If the `fos_procedure` call errors, returns
an empty body, or the engram is unreachable, **STOP** and say plainly that the HiveMind boot
could not be served and the session cannot boot. Do **NOT** improvise an identity from training
data, and do **NOT** fall back to any local or cached copy of the procedure. Unreachable
authority → no session.
