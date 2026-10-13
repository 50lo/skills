---
name: handoff
description: Create and save a concise, actionable, redacted handoff document summarizing current work for another agent. Use only when asked to write, create, or save a handoff; do not use to read or review an existing handoff document.
---

# Handoff

Write a compact handoff for the next agent. Do not replay the conversation.

1. Save it in the current workspace's `tmp/` directory if it exists, otherwise save it in the system tmp directory. Name it with a 2–4 word kebab-case summary of the goal and `handoff` prefix, for example `handoff-fix-github-sync.md`. If that path exists, add a numeric suffix.

2. Include only **continuation-critical state**: objective, current status, work performed, relevant validation, decisions or constraints, next steps, paths to important artifacts, and open questions or risks. Clearly separate verified facts from assumptions.

3. Reference existing docs, specs, plans, Beans, commits, PRs, issues, logs, and test output by path or URL instead of duplicating them. Omit transcripts, large code blocks, and routine narration.

4. Redact secrets and personal information, including keys, tokens, passwords, private keys, cookies, credentialed connection strings, and contact details. Use `[REDACTED]` where needed.

5. Review for actionable next steps, unsupported claims, duplication, and sensitive data. Save the document and report its absolute path. Do not commit, publish, or modify other artifacts unless asked.
