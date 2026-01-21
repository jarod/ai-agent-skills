# AI Agent Skills

A collection of AI [agent skills](https://agentskills.io/) for Claude Code.

## Skills

### Rust Development

- **ripdoc-lookup**: Query and explore Rust crate APIs and documentation using the [ripdoc](https://crates.io/crates/ripdoc) CLI tool. Essential for Rust development tasks requiring quick access to API documentation.
- **rust-ddd**: Pragmatic Domain-Driven Design guidance for Rust projects. Provides tiered complexity levels (Simple → Layered → Full DDD) matching project scale, with framework integration patterns for axum, sea-orm, redis, and message queues.

### Go Development

- **gva-doc**: Gin-Vue-Admin (GVA) documentation reference. Provides quick access to official GVA documentation for backend Go/Gin development, frontend Vue.js, JWT+Casbin auth, code generator, deployment, and troubleshooting.

## Installation

Add the marketplace and install plugins:

```bash
# Add marketplace
claude plugin marketplace add jarod/ai-agent-skills

# Install plugins
claude plugin install rust-skills
claude plugin install go-skills
```

## Update

Update plugins to the latest version:

```bash
claude plugin marketplace update jarod/ai-agent-skills

claude plugin update rust-skills
claude plugin update go-skills
```

## Structure

```
ai-agent-skills/
├── .claude-plugin/
│   └── marketplace.json     # Marketplace configuration
├── skills/	# Skill implementation
└── README.md
```

## License

Apache License 2.0 - see [LICENSE](LICENSE) for details.
