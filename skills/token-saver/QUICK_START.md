# Copilot Token Saver - Quick Start (Updated with Playwright)

## Your Actual Problem

You're burning tokens on **UI changes that don't need tests**:
- "Make card corners more rounded" → Playwright runs 50 tests
- "Change button color" → Playwright runs again
- "Add more spacing" → Playwright runs AGAIN
- **Token limit hit** 💥

Plus regular retry loops when debugging actual bugs.

## The Core Issue

**Playwright is expensive and you're using it for the wrong things.**
- UI tweaks? → Use your eyes
- Logic changes? → Use tests

## Immediate Fixes

### 1. Disable Auto-Test-on-Save

**Check VS Code settings** (Cmd/Ctrl + ,):
```json
{
  "playwright.runOnSave": false,
  "testing.automaticallyRunAfterSave": false
}
```

**Check package.json:**
```json
{
  "scripts": {
    // ❌ Remove or don't run
    "test:watch": "playwright test --watch"
  }
}
```

### 2. Stop Running Tests Right Now
- Kill any test processes
- Start a new Copilot chat
- Close extra files
- Work without tests running

### 3. Use Your Workflow

**For UI/Style Changes:**
```
1. Open localhost:3000 in browser
2. Make CSS change in VS Code
3. Hot reload shows it instantly
4. Look at it with your eyes ← No tests needed!
5. Happy? Move on. Not happy? Adjust.

[Later, when done with ALL UI changes]
6. Run tests once: npm test
```

**For Logic/Bug Fixes:**
```
1. Make code change
2. Run ONLY the relevant test:
   npx playwright test -g "login validation"
3. If fails, give Copilot JUST the error
4. Fix, test again
```

## The Mental Model

| What You're Changing | Need Tests While Working? | How to Verify |
|---------------------|--------------------------|---------------|
| Colors, fonts, spacing | ❌ NO | Look at it |
| Border radius, shadows | ❌ NO | Look at it |
| Layout (flexbox, grid) | ❌ NO | Look at it |
| Text/copy changes | ❌ NO | Look at it |
| Login validation | ✅ YES | Run specific test |
| API calls | ✅ YES | Run specific test |
| Form submission | ✅ YES | Run specific test |
| Auth/permissions | ✅ YES | Run specific test |

**Rule:** If you can see it's right by looking, DON'T run tests while iterating.

## Better Playwright Commands

Add to package.json:
```json
{
  "scripts": {
    "dev": "vite",  // or your dev server
    "test": "playwright test",
    "test:ui": "playwright test --ui",  // Interactive UI
    "test:one": "playwright test -g"     // Run one test
  }
}
```

**During development:**
```bash
# Run dev server (hot reload)
npm run dev

# When ready to test (not while coding UI!)
npm run test:one "login"  # Just one test

# Visual/interactive testing (great for debugging)
npm run test:ui

# Full suite (before commit)
npm test
```

## Your New Daily Workflow

### Morning Setup
```bash
npm run dev           # Start dev server
open localhost:3000   # Open browser
# That's it. NO tests running.
```

### Working on UI (Cards, Colors, etc.)
```
1. Make change in VS Code
2. See it in browser (hot reload)
3. Does it look good? 
   ✅ Yes → Move on
   ❌ No → Adjust
4. NO COPILOT NEEDED for simple CSS
5. NO TESTS RUNNING
```

### Working on Logic (Validation, API, etc.)
```
1. Make change
2. Run specific test: npm run test:one "the test name"
3. If fails:
   - Give Copilot JUST the error
   - NOT 1000 lines of output
4. Fix and test again
```

### Before Committing
```bash
npm test  # Now run full suite
# Fix any failures
# Commit when green ✅
```

## Stopping Token Waste

**When giving Copilot test failures:**

❌ **Don't paste this:**
```
[500 lines of Playwright output]
[Screenshots]
[Trace files]
[Full stack traces]
```

✅ **Paste this:**
```
"Test 'login validation works' failed at line 23:
Expected: button to be visible
Got: timeout after 5000ms"
```

## Configuration Tips

**Make Playwright faster for development:**

```javascript
// playwright.config.js
export default defineConfig({
  retries: process.env.CI ? 2 : 0,  // No retries during dev
  workers: process.env.CI ? 4 : 1,  // One worker during dev
  timeout: 10000,                   // Fail fast
  
  // Only in CI - run everything
  // In dev - skip unless you explicitly run
});
```

**Tag tests:**
```javascript
// Fast smoke tests for development
test('button exists', { tag: '@fast' }, async ({ page }) => {
  // Quick check
});

// Slow E2E for CI only
test('full user flow', { tag: '@slow' }, async ({ page }) => {
  // Complete workflow
});
```

**Run tagged tests:**
```bash
# Development - just fast tests
npx playwright test --grep @fast

# CI - everything
npx playwright test
```

## Copilot Best Practices

### For UI Changes
```
You: "Make the card border radius 12px"
Copilot: [suggests CSS]
You: [Look at browser]
✅ Looks good? Done.
❌ Not quite? "Make it 16px instead"

NO TESTS. Just looking.
```

### For Bug Fixes
```
You: "Login button click doesn't submit form"
Copilot: [suggests fix]
You: npm run test:one "login"
[Test fails]
You: "Test fails: Expected form submit, got nothing"
Copilot: [suggests fix]
You: npm run test:one "login"
✅ Passes? Done.
```

### General Rules
- After 3 tries → New chat
- Close extra files
- Be specific with line numbers
- Don't run tests for visual changes
- Stop tests if they're auto-running

## Quick Checklist

**Before starting work:**
- [ ] Dev server running? (npm run dev)
- [ ] Browser open?
- [ ] Auto-tests DISABLED?
- [ ] No test processes running?

**Working on UI/styling:**
- [ ] Making visual changes?
- [ ] Using hot reload to see them?
- [ ] Skipping tests while iterating?
- [ ] Will run tests when done?

**Working on logic:**
- [ ] Running specific test only?
- [ ] Not running full suite?
- [ ] Giving Copilot concise errors?

**Before committing:**
- [ ] Run full test suite?
- [ ] All tests green?

## Remember

**Token killers:**
1. Auto-running Playwright for UI changes (25%)
2. Unclear questions (30%)
3. Too many open files (25%)
4. Retry loops (20%)

**The golden rules:**
- UI changes → Eyes, not tests
- Logic changes → Specific test only
- Before commit → Full suite
- In token trouble → Stop tests, new chat, be specific

**You're working on a VISUAL product:**
- Trust your eyes for visual changes
- Save tests for behavior verification
- Run full suite before shipping
- Don't let tests run your workflow
