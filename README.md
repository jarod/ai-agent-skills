# AI Agent Skills

A collection of AI [agent skills](https://agentskills.io/) for Claude Code.

## Skills

### Rust Development

- **ripdoc-lookup**: Query and explore Rust crate APIs and documentation using the [ripdoc](https://crates.io/crates/ripdoc) CLI tool. Essential for Rust development tasks requiring quick access to API documentation.

## Installation

Add this marketplace to Claude Code:

```bash
claude /plugin install jarod/ai-agent-skills
```

## Structure

```
ai-agent-skills/
├── .claude-plugin/
│   └── marketplace.json     # Marketplace configuration
├── skills/
│   └── ripdoc-lookup/
│       └── SKILL.md         # Skill definition
└── README.md
```

## License

Apache License 2.0 - see [LICENSE](LICENSE) for details.
