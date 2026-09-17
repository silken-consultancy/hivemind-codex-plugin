---
name: hivemind-end-session
description: Close a HiveMind session — consolidate what happened into the engram as memory, and ALWAYS write a continuity handoff (next_note) so the next boot resumes the thread. Use at the end of a HiveMind session, or when the user says "end session", "/end-session", "close the session", "encerrar", "fechar sessão", or is wrapping up.
---

# HiveMind End-Session — thin carrier for the served end-session procedure

**This is a thin stub, not the close itself.** The authoritative end-session procedure is
**served by the engram** (config-as-data) and fetched at invocation — so this vessel always runs
the current canonical close (memory consolidation + the guaranteed continuity handoff), never a
forked or stale copy.

Codex is a **foreign vessel**: it has **no `SessionEnd` hook and no runtime** to close the
session for you, so **THIS skill is the primary close, not a safety net.** Skip it and the
session `hivemind-boot` opened is left orphaned and the handoff for the next boot is lost.

Do this now, in order:

1. **Make the engram tools callable.** The engram is connected as the MCP server `engram` via
   `config.toml` (Porta B bearer). If your client defers MCP tool schemas, load/select
   `fos_procedure` and `fos_session` first. If the `engram` server is not connected at all, that
   is a fail-closed stop (below) — do not claim the session closed.

2. **Fetch the procedure:** call `fos_procedure({ id: "end-session" })`.

3. **Execute the returned `body` verbatim and in full.** It consolidates the session's memories
   (write contract: dedup first, Update > Complement > Create) and closes the session with a
   mandatory `next_note` (`WIP:` + `NEXT:`), resolving the session by cascade — the `session_id`
   `hivemind-boot` opened this conversation, else `ENGRAM_SESSION_ID`, else discovery
   (`fos_session({ action: "list_active" })`) — and, if no open session is found, opening one
   solely to carry the handoff, so continuity is **ALWAYS** guaranteed. Follow it exactly, and
   write the memory as yourself, in the first person.

**FAIL-CLOSED.** If the `fos_procedure` call errors, returns an empty body, or the engram is
unreachable, **STOP** and say plainly that the close could not be served — do not fake a close
or improvise a handoff. Never commit anything.
