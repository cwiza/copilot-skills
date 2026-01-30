---
name: token-saver-lite
description: Concise rules for avoiding Copilot token limits.
---

# Token Saver Rules

## When User Hits Token Limits
1. Start new chat (clears history)
2. Close all files except the one being debugged
3. Stop any auto-running tests

## Prevent Retry Loops
- After 3 failed attempts → suggest new chat + different approach
- If same fix suggested twice → stop, reframe the question
- Never say "that didn't work, try again" repeatedly

## Guide Users to Be Specific

❌ Vague: "Fix the authentication"
✅ Specific: "In auth.js line 42, JWT fails with 'invalid signature'"

Template: "In [file] line [X], [behavior]. Expected [Y], got [Z]. Why?"

## Test Runner Guidance
- UI/styling changes → verify visually, skip tests while iterating
- Logic changes → run only the specific test: `npx playwright test -g "name"`
- Full suite → only before committing

## Context Management
- Fewer open files = less token usage per request
- Suggest closing unrelated files
- Keep error messages concise (just the relevant line, not full stack)

## Quick Reference
| Issue | Action |
|-------|--------|
| Token limit | New chat, close files, stop tests |
| Same fix twice | Different approach |
| Vague question | Ask for file + line number |
| UI change | Skip tests, use eyes |
