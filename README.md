# Skills Hub

A collection of Claude Code skills for enhanced AI-assisted development workflows.

## Skills

| Skill | Description |
|-------|-------------|
| **[brainstorm](./brainstorm/)** | Multi-agent deliberation with 3 parallel perspectives debating solutions |
| **[explain](./explain/)** | Deep code explanation and documentation |
| **[verify](./verify/)** | Verify volatile knowledge, trust stable knowledge |
| **[suggest](./suggest/)** | Intelligent code improvement suggestions |
| **[update](./update/)** | Automated code modernization and updates |
| **[write-for-claude](./write-for-claude/)** | Generate Claude-optimized documentation |
| **[skills-hub](./skills-hub/)** | Meta-skill that routes and orchestrates other skills |

## Installation

Each skill can be installed individually. See the `INSTALL.md` in each skill directory.

### Quick Install (all skills)

```bash
# Clone the repository
git clone https://github.com/PSthelyBlog/skills-hub.git

# Install a specific skill (example: brainstorm)
cat brainstorm/INSTALL.md
```

## Usage

After installation, invoke skills using Claude Code's slash commands:

```
/explain src/api.ts
/suggest --category security src/
/brainstorm "Should we use REST or GraphQL?"
/verify "What are the latest React 19 features?"
```

### Skills Hub (Meta-Skill)

The skills-hub acts as an intelligent router:

```
/skills-hub explain then suggest src/api.ts
```

It can:
- Route requests to the appropriate skill based on intent
- Chain skills together with "then" keyword
- Execute independent skills in parallel

## Architecture

Each skill follows a consistent structure:

```
skill-name/
├── SKILL.md          # Main skill instructions (for Claude)
├── DESIGN.md         # Design rationale and decisions
├── EXAMPLES.md       # Real-world usage examples
├── INSTALL.md        # Installation guide
├── DELIVERABLES.md   # Summary of deliverables
├── QUICKSTART.txt    # Quick reference card
├── CHANGELOG.md      # Version history
└── LICENSE           # MIT License
```

## License

MIT License - see [LICENSE](./LICENSE)
