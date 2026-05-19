# Slides

Marp slide decks for the AI for Developers track. One deck per lesson.

| File | Lesson |
|---|---|
| [01-foundations.md](./01-foundations.md) | Lesson 1 — Foundations |
| [02-ai-workflows.md](./02-ai-workflows.md) | Lesson 2 — AI Workflows |
| [03-terminal-agents.md](./03-terminal-agents.md) | Lesson 3 — Terminal Agents & Sandboxing |
| [04-mcps.md](./04-mcps.md) | Lesson 4 — MCPs |

## Viewing the slides

### VS Code

Install the [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) extension. Open any slide file and click the preview icon.

### Marp CLI

```bash
# Install
npm install -g @marp-team/marp-cli

# Preview in browser with live reload
marp --server slides/

# Export to PDF
marp slides/01-foundations.md --pdf

# Export to HTML
marp slides/01-foundations.md --html

# Export to PowerPoint
marp slides/01-foundations.md --pptx
```

### Build them all

```bash
for f in slides/0*.md; do marp "$f" --pdf; done
```
