# Installing swift-type-driven-design for OpenCode

Enable the Type-Driven Design skill in [OpenCode](https://opencode.ai) by pointing OpenCode's skill loader at this repo's skills directory.

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed
- Git

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/MathisDetourbet/swift-type-driven-design-skill.git \
     ~/.config/opencode/swift-type-driven-design-skill
   ```

2. **Register the skills directory** in your `opencode.json` (global or project-level). Add the path under `skills.paths`:

   ```json
   {
     "skills": {
       "paths": [
         "~/.config/opencode/swift-type-driven-design-skill/skills"
       ]
     }
   }
   ```

   If `skills.paths` already exists, append the new entry to the array.

3. **Restart OpenCode.**

## Verify

Use OpenCode's native `skill` tool:

```
use skill tool to list skills
```

You should see `type-driven-design` in the output. Then ask: *"Review this struct using type-driven-design principles."*

## Updating

```bash
cd ~/.config/opencode/swift-type-driven-design-skill && git pull
```

Restart OpenCode to pick up changes.

## Uninstalling

Remove the path from `skills.paths` in `opencode.json`, then:

```bash
rm -rf ~/.config/opencode/swift-type-driven-design-skill
```
