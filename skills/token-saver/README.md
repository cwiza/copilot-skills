# Token Saver Skill

Practical techniques for avoiding token limits in GitHub Copilot and similar AI coding assistants.

## When to Use This Skill

- Hitting "rate limit" or "token limit" errors
- Copilot keeps retrying the same failed fix
- Auto-running tests (like Playwright) are burning through tokens
- Long conversations that seem to go in circles

## Quick Start

See [QUICK_START.md](./QUICK_START.md) for an actionable checklist.

## Installation

Choose your version:

| Version | Tokens | Best For |
|---------|--------|----------|
| **Lite** (~300 tokens) | Low overhead | Most users |
| Full (~3,500 tokens) | Comprehensive | Learning/reference |

```bash
# Lite version (recommended)
cp copilot-instructions-lite.md /path/to/your/project/.github/copilot-instructions.md

# Full version
cp copilot-instructions.md /path/to/your/project/.github/copilot-instructions.md
```

## The Problem

The #1 reason for hitting token limits: **Retry loops** - Copilot keeps trying the same fix that doesn't work, burning 5-10x the tokens.

**Test Runner Loops** are a huge hidden token drain:
```
You: "Make card corners more rounded"
[Playwright auto-runs full suite - 2 minutes, 500 lines output]
Copilot: [sees test output, suggests change]
[Playwright runs again - another 2 minutes, 500 lines]
... [repeat until token limit]
```

## Key Techniques

### 1. Disable Auto-Test-on-Save

```json
// VS Code settings.json
{
  "playwright.runOnSave": false,
  "testing.automaticallyRunAfterSave": false
}
```

### 2. Categorize Your Work

| Type of Change | Need Tests While Iterating? |
|----------------|----------------------------|
| UI styling (colors, spacing) | ❌ NO - use your eyes |
| Layout changes | ❌ NO - use your eyes |
| Form validation | ✅ YES - run specific test |
| API integration | ✅ YES - run specific test |

### 3. Be Specific, Not Vague

❌ **Vague (wastes tokens):**
```
"This authentication isn't working, can you fix it?"
```

✅ **Specific (saves tokens):**
```
"In auth.js line 42, JWT verification fails with 'invalid signature'.
The token is generated in generateToken() on line 15.
What's the mismatch?"
```

### 4. The 3-Strike Rule

After 3 failed attempts at the same fix:
1. **STOP** - Cancel any in-progress request
2. **RESET** - Start a new Copilot chat
3. **REFRAME** - Ask a different question

## The 5 Rules

1. **Start Fresh When Stuck** - Reset after 3 failed attempts
2. **Control Your Context Window** - Close files you don't need
3. **Be Surgical, Not General** - Specific questions with line numbers
4. **Stop Retry Loops Yourself** - Don't keep saying "that didn't work"
5. **Break Big Tasks Into Small Steps** - Each step uses fewer tokens

## Token Limit Causes

- Auto-running test output (25%)
- Unclear questions (30%)
- Too many open files (25%)
- Retry loops (20%)

## Full Documentation

See the complete `copilot-instructions.md` for detailed guidance including:
- Playwright-specific configurations
- Daily workflow recommendations
- Advanced techniques
- Model-specific notes
