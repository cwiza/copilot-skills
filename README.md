# Copilot Skills

A collection of reusable skills for GitHub Copilot that help optimize your AI-assisted development workflow.

## What Are Copilot Skills?

Skills are instruction sets that teach Copilot best practices for specific scenarios. They help you:
- Avoid common token-wasting patterns
- Work more efficiently with AI assistants
- Follow proven workflows for specific tools

## Available Skills

| Skill | Description |
|-------|-------------|
| [Token Saver](./skills/token-saver/) | Avoid token limits when using Copilot, especially with Playwright tests |

## Installation

### Option 1: Add to Your Project (Recommended)

Copy the skill's `copilot-instructions.md` content to your project:

```bash
# In your project root
mkdir -p .github
cp skills/token-saver/copilot-instructions.md .github/copilot-instructions.md
```

Copilot will automatically read instructions from `.github/copilot-instructions.md`.

### Option 2: VS Code Settings

Add to your VS Code settings (`.vscode/settings.json`):

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "path/to/copilot-instructions.md"
    }
  ]
}
```

### Option 3: Manual Reference

Just keep the skill docs open and follow the guidelines manually.

## Creating Your Own Skills

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on creating and sharing skills.

## Skill File Format

Each skill contains:
- `copilot-instructions.md` - The actual instructions Copilot reads
- `QUICK_START.md` - Human-readable quick reference
- `README.md` - Full documentation and examples

## License

MIT - Use freely, share widely.
