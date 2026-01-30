# Contributing Skills

Want to add your own Copilot skill? Here's how.

## Skill Structure

Each skill lives in `skills/<skill-name>/` with these files:

```
skills/
└── your-skill-name/
    ├── README.md                 # Human documentation
    ├── QUICK_START.md            # Actionable checklist
    └── copilot-instructions.md   # What Copilot actually reads
```

## Writing copilot-instructions.md

This is the file that Copilot reads. Follow these guidelines:

### Use YAML Frontmatter

```markdown
---
name: your-skill-name
description: One-line description of what this skill does and when to use it.
---
```

### Structure for Clarity

1. **Why/Problem** - Explain the problem this solves
2. **Quick Fix** - Emergency steps for immediate relief
3. **Rules/Guidelines** - Core principles to follow
4. **Examples** - Good vs bad patterns
5. **Checklists** - Actionable items

### Write for AI Consumption

- Be direct and specific
- Use clear headings
- Include concrete examples
- Prefer tables for comparisons
- Keep instructions actionable

## Testing Your Skill

1. Copy to a test project's `.github/copilot-instructions.md`
2. Start a new Copilot chat
3. Test scenarios where the skill should help
4. Verify Copilot follows the guidelines

## Submitting

1. Fork this repo
2. Create your skill in `skills/your-skill-name/`
3. Add an entry to the main README.md table
4. Submit a PR with:
   - What problem does this skill solve?
   - What tools/workflows does it apply to?
   - How did you test it?

## Skill Ideas

Looking for inspiration? Here are skills that would be useful:

- **React Best Practices** - Component patterns, hooks usage
- **API Design** - REST/GraphQL conventions
- **Testing Strategy** - When to unit vs integration vs e2e test
- **Git Workflow** - Commit messages, branching strategies
- **Performance** - Optimization techniques for specific frameworks
- **Security** - Common vulnerability prevention
- **Accessibility** - A11y patterns and testing

## Questions?

Open an issue!
