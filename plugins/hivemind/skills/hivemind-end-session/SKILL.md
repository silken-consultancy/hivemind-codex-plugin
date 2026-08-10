---
name: hivemind-end-session
description: Use ao encerrar uma sessão HiveMind para consolidar memórias e gravar o handoff de continuidade via o procedimento servido pelo engram.
---

Chame a tool `fos_procedure` com `{ "id": "end-session" }` e **execute o `body` retornado verbatim** — ele consolida as memórias da sessão e fecha com um `next_note` de continuidade garantido, para o próximo boot retomar de onde parou.
