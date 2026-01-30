---
name: copilot-token-saver
description: Practical techniques for avoiding token limits in GitHub Copilot and similar AI coding assistants. Use when hitting "rate limit" or "token limit" errors, when Copilot keeps retrying the same failed fix, or when auto-running tests (like Playwright) are burning through tokens. Focuses on manual techniques you can actually control as a user, including managing expensive test runner feedback loops.
---

# Copilot Token Saver

## Why You're Hitting Token Limits

The #1 reason: **Retry loops** - Copilot keeps trying the same fix that doesn't work, burning 5-10x the tokens.

**NEW: Test Runner Loops** - A huge hidden token drain:
```
You: "Make card corners more rounded"
[Playwright auto-runs full suite - 2 minutes, 500 lines output]
Copilot: [sees test output, suggests change]
[Playwright runs again - another 2 minutes, 500 lines]
Copilot: [suggests another change]
... [repeat until token limit]
```

Other common causes:
- Too many files open in your editor (Copilot uses them all as context)
- Long conversation history building up
- Vague requests that require multiple back-and-forths
- Complex codebases with lots of dependencies

## Emergency Fix (When You're Hitting Limits Right Now)

**Do these in order:**

1. **STOP** - Cancel any in-progress Copilot request AND any running tests
2. **DISABLE AUTO-TESTS** - (see below for how)
3. **RESET** - Start a new Copilot chat (clears conversation history)
4. **CLOSE** - Close all files except the one you're debugging
5. **FOCUS** - Ask a specific question instead of "fix this"

### Example Transformation:

❌ **Vague (wastes tokens):**
```
"This authentication isn't working, can you fix it?"
```

✅ **Specific (saves tokens):**
```
"In auth.js line 42, JWT verification fails with error 'invalid signature'.
The token is being generated in generateToken() on line 15.
What's the mismatch?"
```

**Why this works:** Copilot solves it in 1-2 tries instead of 5-10 tries.

## Playwright-Specific Token Drains

### The Problem: Auto-Running Tests for UI Tweaks

**Your workflow currently:**
```
You: "Make the card border radius 8px instead of 4px"
Copilot: [suggests CSS change]
Playwright: [runs full suite - 50 tests, 2 min, 1000 lines output]
Copilot: [sees output, might suggest more]
You: "Actually make it 12px"
Playwright: [runs again - another 1000 lines]
...
```

**Why this burns tokens:**
- Each test run = 500-2000 lines of output
- If Copilot sees this output, it counts toward your token limit
- For UI tweaks, you don't NEED automated tests running
- You can just LOOK at the UI with your eyes

### Solution 1: Disable Auto-Test-on-Save

**Check if you have this enabled:**

**VS Code settings.json:**
```json
{
  // ❌ These cause auto-running
  "playwright.runOnSave": true,
  "testing.automaticallyRunAfterSave": true,
  
  // ✅ Change to false or remove
  "playwright.runOnSave": false,
  "testing.automaticallyRunAfterSave": false
}
```

**Or check for watch mode in package.json:**
```json
{
  "scripts": {
    // ❌ Watch mode runs tests continuously
    "test:watch": "playwright test --watch",
    
    // ✅ Use manual test command instead
    "test": "playwright test"
  }
}
```

**Check VS Code extensions:**
- Playwright Test extension might have auto-run enabled
- Check extension settings: `Cmd/Ctrl+Shift+P` → "Preferences: Open Settings (UI)" → search "playwright"

### Solution 2: Use Playwright Intentionally

**The Better Workflow:**

**For UI/Style Changes (no tests needed):**
```
You: "Make card border radius 12px"
Copilot: [suggests change]
You: [Look at the UI in browser - does it look good?]
✅ Good? Move on. No tests needed.
❌ Bad? Ask Copilot to adjust.

[Later, when done with all UI changes]
You: Run tests once: `npx playwright test`
```

**For Logic/Functionality Changes (tests needed):**
```
You: "Fix the login validation"
Copilot: [suggests change]
You: Run just the relevant test: `npx playwright test -g "login validation"`
[If fails, give Copilot just the error]
```

### Solution 3: Categorize Your Work

**Create this mental model:**

| Type of Change | Need Tests While Iterating? | When to Test |
|----------------|----------------------------|--------------|
| UI styling (colors, spacing, fonts) | ❌ NO | After you're done |
| Layout changes (flexbox, grid) | ❌ NO | After you're done |
| Copy/text changes | ❌ NO | After you're done |
| Form validation | ✅ YES | After each change |
| Authentication/auth | ✅ YES | After each change |
| Data operations (CRUD) | ✅ YES | After each change |
| API integration | ✅ YES | After each change |

**Rule of thumb:** If you can verify it by looking at it, skip tests while iterating.

### Solution 4: Configure Playwright for Development

**Create a development-friendly config:**

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Use different configs for dev vs CI
  ...(process.env.CI 
    ? {
        // CI: Run everything, retries, full reports
        retries: 2,
        workers: 4,
        reporter: 'html'
      }
    : {
        // DEV: Fast, targeted, fail-fast
        retries: 0,           // Don't retry during dev
        workers: 1,           // One at a time
        maxFailures: 1,       // Stop after first failure
        reporter: 'line',     // Minimal output
        timeout: 10000,       // Fail fast on hangs
      }
  ),
  
  // Only run tests in specific directory during dev
  testDir: './tests',
  
  // Skip slow tests during development
  grep: process.env.CI ? undefined : /@fast/,
});
```

**Tag your tests:**
```javascript
// Fast tests for development (UI smoke tests)
test('login button visible', { tag: '@fast' }, async ({ page }) => {
  await page.goto('/login');
  await expect(page.locator('button[type="submit"]')).toBeVisible();
});

// Slow E2E tests for CI only
test('complete user registration flow', { tag: '@slow' }, async ({ page }) => {
  // ... full registration flow
});
```

**Run only fast tests during dev:**
```bash
# Development - only fast smoke tests
npx playwright test --grep @fast

# Before committing - run everything
npx playwright test
```

### Solution 5: Playwright Commands for Intentional Testing

**Keep these commands handy:**

```bash
# Run just one test file
npx playwright test auth.spec.js

# Run just tests matching a pattern
npx playwright test -g "login"

# Run in UI mode (visual, interactive - great for dev)
npx playwright test --ui

# Run in debug mode (step through)
npx playwright test --debug

# Run just the last failed test
npx playwright test --last-failed

# Skip all tests (when you just want to code)
# Just don't run the command!
```

**Add these to your package.json:**
```json
{
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:debug": "playwright test --debug",
    "test:one": "playwright test -g",
    "test:fast": "playwright test --grep @fast"
  }
}
```

### Solution 6: Visual Testing Instead of Automated

**For UI changes, use your eyes:**

```
❌ Don't do this:
  Change CSS → Wait for Playwright → Read test output → Repeat

✅ Do this:
  Open localhost:3000 in browser
  Open Chrome DevTools (F12)
  Make CSS changes live in DevTools
  When happy, copy to your code
  THEN run tests once
```

**Hot reload is your friend:**
- Most frameworks (React, Vue, etc.) have hot reload
- Changes appear instantly in browser
- No tests needed to see if a card looks right
- Use Playwright to verify it WORKS, not how it LOOKS

## The 5 Rules for Token-Efficient Copilot Usage

### Rule 1: Start Fresh When Stuck

**When to reset:**
- After 3 failed attempts at the same thing
- When you've been chatting for 30+ messages
- When Copilot suggests the same thing twice
- When hitting token limits
- **When tests keep failing and you're just tweaking UI**

**How to reset:**
- GitHub Copilot Chat: Click the "+" for new conversation
- Close and reopen the chat panel
- Restart VS Code (nuclear option)
- **Stop any running tests**

### Rule 2: Control Your Context Window

**Close files you don't need:**
```
✅ Open: The file you're debugging
✅ Open: Related files mentioned in the error
❌ Open: 20 other files from your last task
❌ Open: Test output files
❌ Open: Playwright trace files
```

**VS Code tip:** Use `Ctrl+W` (Windows/Linux) or `Cmd+W` (Mac) to close tabs quickly.

**Why this matters:** Copilot sends snippets from ALL open files with EVERY request. 10 open files = 10x the context = 10x the tokens.

### Rule 3: Be Surgical, Not General

**Bad requests (waste tokens):**
- "Debug this file"
- "Fix all the tests"
- "Improve this code"
- "What's wrong here?"

**Good requests (save tokens):**
- "Why does line 42 throw 'undefined is not a function'?"
- "The test on line 67 fails - expected 200, got 404. Why?"
- "This forEach on line 23 skips the first item. Bug?"
- "What's the type mismatch on line 15?"

**Template:**
```
"In [file] line [number], [specific behavior] happens.
Expected: [X]
Actual: [Y]
Why?"
```

### Rule 4: Stop Retry Loops Yourself

**Signs you're in a retry loop:**
- Copilot suggests same fix 2+ times
- You keep saying "that didn't work"
- Token limit errors appear
- You've tried 5+ variations of the same approach
- **Tests keep running and failing**

**What to do:**
1. **STOP** - Don't try again, STOP the tests
2. **RESET** - New conversation
3. **REFRAME** - Ask a different question:
   - Instead of: "Fix this bug"
   - Try: "Why might this error occur?" (diagnostic first)
   - Then: "How do I verify the token format?" (check assumptions)
   - Finally: "Given [finding], what's the fix?" (specific solution)

### Rule 5: Break Big Tasks Into Small Steps

**Instead of:**
"Refactor this entire authentication system to use OAuth2"

**Do this:**
```
Step 1: "What files will need to change to add OAuth2?"
Step 2: "Show me the OAuth2 flow for the login endpoint only"
Step 3: "Now update the token validation middleware"
... (continue)
```

**Why:** Each step uses fewer tokens AND you can catch issues early.

## Copilot-Specific Optimizations

### Use Copilot's Features Strategically

**Inline completions (cheap):**
- Use for autocomplete, writing boilerplate
- Generates locally, uses minimal tokens
- Accept with Tab

**Chat (expensive):**
- Use for debugging, explanations, complex logic
- Burns tokens with every message
- Use sparingly

**Slash commands (efficient):**
- `/explain` - Cheaper than asking "what does this do?"
- `/fix` - More direct than open-ended debugging
- `/tests` - Scoped to just generating tests

### Comment-Driven Development

**Good comments = less back-and-forth:**

❌ **Bad (requires clarification):**
```javascript
// Fix the authentication
function authenticate(user) { ... }
```

✅ **Good (Copilot knows what to do):**
```javascript
// Validate JWT token from Authorization header
// Return decoded user object if valid
// Throw 401 error if invalid or expired
function authenticate(token) { ... }
```

### File Organization Matters

**Keep files focused:**
- ✅ `auth.js` - 200 lines of auth logic
- ❌ `utils.js` - 2000 lines of random helpers

**Why:** When Copilot includes a file as context, it includes MORE of focused files and gets better results.

## Common Copilot Token Mistakes

### Mistake 1: Leaving Error Logs in the Chat
```
❌ Pasting 500 lines of stack trace
❌ Pasting 1000 lines of Playwright test output
✅ Pasting the relevant error: "TypeError: Cannot read property 'id' of undefined at line 42"
✅ Pasting just the test failure: "Test 'login works' failed: expected visible, got timeout"
```

### Mistake 2: Asking About Code Not in Your Editor
```
❌ "How does the login system work?" (Copilot has to guess)
✅ Open login.js, then ask "How does this file's login flow work?"
```

### Mistake 3: Not Using Code References
```
❌ "The user authentication thing isn't working"
✅ Select the code, then click "Add to chat" or use "#file:auth.js"
```

### Mistake 4: Fighting Copilot Instead of Resetting
```
After 3 failed attempts:
❌ "No that's still wrong, try again"
❌ "That didn't work either"
❌ "No, different approach" <- You're here, wasting tokens
✅ Start new chat with different question
```

### Mistake 5: Running Tests for Visual Changes
```
❌ "Change button color to blue" → [runs 50 Playwright tests]
✅ "Change button color to blue" → [look at it in browser]
```

## Recommended Daily Workflow

### Morning Setup
```bash
# Start dev server with hot reload
npm run dev

# Open browser to localhost
# Open Chrome DevTools (F12)

# In VS Code, disable auto-test
# Check: no watch mode running
```

### During UI Work
```
1. Make CSS/styling changes
2. See results instantly in browser (hot reload)
3. Use Copilot for suggestions, but verify visually
4. NO TESTS while iterating
```

### During Logic Work
```
1. Make code change
2. Run specific test: `npm run test:one "login"`
3. If fails, give Copilot just the error
4. Don't let full suite run
```

### Before Committing
```bash
# Now run full test suite
npm test

# Fix any failures
# Commit when green
```

### Weekly/As-Needed
```bash
# Update Playwright
npm update @playwright/test

# Review test speeds - tag slow ones
# Refactor flaky tests
```

## Advanced Techniques

### The "Rubber Duck" Approach
When stuck, ask Copilot diagnostic questions instead of "fix it":
- "What are the possible causes of this error?"
- "What should I check to debug this?"
- "What assumptions might be wrong here?"

**Why it works:** Helps you solve it yourself OR gives you specific info for a targeted fix request.

### The "Divide and Conquer" Pattern
For complex bugs:
1. "Is the input correct?" (verify assumptions)
2. "Is the processing logic correct?" (isolate problem)
3. "Is the output formatted correctly?" (narrow scope)
4. "Fix the [specific part] found in step 2"

### The "Context Snapshot" Trick
Before starting a complex task:
1. Close all files
2. Open ONLY files needed for this specific task
3. Pin them (right-click tab → Pin)
4. Now start Copilot chat

This creates a "clean context" for just this work.

## Workflow Checklist

**Before asking Copilot anything:**
- [ ] Closed unrelated files?
- [ ] Stopped any auto-running tests?
- [ ] Know the specific line/file with the issue?
- [ ] Have the error message ready?
- [ ] Tried reading the code first?
- [ ] Is this a visual change I can just look at?

**When hitting limits:**
- [ ] Tried more than 3 times? → Reset conversation
- [ ] Been chatting 30+ messages? → Start fresh
- [ ] Same suggestion twice? → Different approach
- [ ] Are tests running on every change? → Disable auto-test

**For best results:**
- [ ] Include line numbers
- [ ] Include error messages (concise version)
- [ ] Ask one thing at a time
- [ ] Reference specific files with #file:
- [ ] Skip tests for UI tweaks

## Quick Reference Card

| Situation | What To Do |
|-----------|------------|
| Token limit error | Start new chat, close files, **stop tests**, be specific |
| Same fix suggested twice | Stop, reset, ask different question |
| Vague about the issue | Find exact line, error, then ask |
| Big refactoring | Break into 5-10 small steps |
| Copilot seems confused | Close all but essential files |
| 30+ message conversation | Start fresh, summarize context |
| **UI/styling change** | **Use browser DevTools, skip tests** |
| **Tests keep failing** | **Stop auto-run, test manually when ready** |
| **Working on colors/fonts** | **Visual verification, no tests needed** |

## Playwright-Specific Checklist

**Before starting work:**
- [ ] Disabled auto-test-on-save?
- [ ] Dev server running with hot reload?
- [ ] Browser open for visual verification?

**During UI work:**
- [ ] Making visual changes? → Look, don't test
- [ ] Making logic changes? → Run specific test only

**Before committing:**
- [ ] Run full test suite once
- [ ] All tests passing?

**If tests are slow:**
- [ ] Tag slow tests with @slow
- [ ] Run only @fast during development
- [ ] Refactor flaky tests

## Model-Specific Notes

**GitHub Copilot (GPT-4 based):**
- Very good at seeing patterns across open files
- Can get confused with too much context
- Best with specific, focused questions
- **Gets overwhelmed by test output**

**Claude Code (if using):**
- Better at longer conversations
- More token capacity
- Still benefits from focused questions
- **Can handle more test output, but still wasteful**

**Cursor AI (if using):**
- Can reference entire codebase
- More expensive per query
- Use @codebase sparingly
- **Test output still counts against limits**

## Remember

**The golden rule:** If you've asked the same question 3 times without success, the problem isn't Copilot - it's your question. Reset and reframe.

**Token limits are usually a symptom of:**
1. Unclear questions (30%)
2. Too much context / open files (25%)
3. Retry loops (20%)
4. **Auto-running test output (25%)** ← NEW!

**For UI work specifically:**
- Your eyes are better than automated tests
- Hot reload is faster than test runs
- Test AFTER you're happy with the look
- Save tokens, save time, stay sane

Fix these and you'll rarely hit limits.
