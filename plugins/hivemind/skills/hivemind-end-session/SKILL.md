---
name: hivemind-end-session
description: Use ao encerrar uma sessão HiveMind (quando o usuário disser "encerrar", "end session", "fechar a sessão" ou estiver finalizando) para consolidar memórias e gravar o handoff de continuidade no engram.
---

Rode o procedimento de **end-session** servido pelo MCP server `engram` (procedimento de id `end-session`) e siga-o exatamente como ele vier.
Se o engram não responder ou não devolver o procedimento, **pare** e diga que o encerramento não foi salvo — não finja um handoff.
