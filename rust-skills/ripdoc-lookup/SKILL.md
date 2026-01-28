---
name: ripdoc-lookup
description: Query and explore Rust crate APIs and documentation using the ripdoc CLI tool. Use this skill when users request information about Rust crate public APIs, want to search Rust documentation, need to understand crate structures, or ask to lookup specific types/functions/modules in Rust crates from crates.io or local filesystem. This skill is essential for Rust development tasks requiring quick access to API documentation without opening a browser.
metadata:
  version: "1.1"
---

# Ripdoc Lookup

## Overview

Enable quick access to Rust crate APIs and documentation through the ripdoc CLI tool (version 0.9.1). Ripdoc prints syntactical outlines of a crate's public API and documentation in a token-efficient format, perfect for AI-assisted Rust development. The tool works with both crates.io packages and local crate paths.

## When to Use This Skill

Invoke this skill when users:
- Request information about a Rust crate's public API (e.g., "Show me the tokio API")
- Want to search for specific functions, types, or modules in Rust crates (e.g., "Find spawn methods in tokio")
- Need to understand the structure of a Rust crate (e.g., "What modules does serde have?")
- Ask about Rust crate documentation (e.g., "How do I use reqwest's Client?")
- Want to explore dependencies or local Rust projects
- Need to lookup specific Rust types or traits (e.g., "Show me the Deserialize trait")
- Need to build incremental context for code understanding (use `skelebuild`)

## Prerequisites

Ripdoc requires the Rust nightly toolchain. Before using ripdoc commands, verify the nightly toolchain is installed:

```sh
rustup toolchain install nightly
```

The ripdoc command must be pre-installed and available in the current shell environment.

## Commands Overview

- `ripdoc print` - Render items as Markdown (default)
- `ripdoc list` - Produce a structured item listing
- `ripdoc skelebuild` - Stateful context builder for incremental builds
- `ripdoc raw` - Emit raw rustdoc JSON
- `ripdoc readme` - Fetch and print the README of a crate
- `ripdoc agents` - Print a dense guide for AI agents

## Core Operations

### 1. Print Crate API Documentation

Use `ripdoc print` to display the public API and documentation of a crate. The output is in Markdown format by default, which is token-efficient and immediately usable. Always use `--no-color` flag to disable ANSI colors for cleaner output.

**Basic syntax:**
```sh
ripdoc print [TARGET] [ITEM] --no-color
```

**Target specifications:**
```sh
# Current project in current directory
ripdoc print

# Crate from crates.io (by name)
ripdoc print tokio

# Crate from crates.io with specific version
ripdoc print serde@1.0.0

# Workspace crate by name
ripdoc print mypackage

# Local crate by path
ripdoc print /path/to/crate

# Specific module or item within a crate (compact form)
ripdoc print tokio::net::TcpStream

# Explicit target and item
ripdoc print serde serde::Deserialize

# Module within local crate
ripdoc print /path/to/crate crate::module
```

**Output format options:**
```sh
# Markdown format (default, token-efficient)
ripdoc print tokio --format markdown

# Raw Rust code skeleton
ripdoc print tokio --format rust
```

**Additional options:**
```sh
# Include private items
ripdoc print tokio --private

# Include auto trait implementations
ripdoc print tokio --auto-impls

# Include implementation spans (method bodies)
ripdoc print tokio::spawn --implementation

# Include raw source files
ripdoc print ./my-crate crate::Config --raw-source

# Enable specific features (like cargo features)
ripdoc print serde --features "derive,rc"

# Enable all features
ripdoc print tokio --all-features

# Disable default features
ripdoc print tokio --no-default-features

# Offline mode (no network)
ripdoc print serde --offline

# Verbose mode (show cargo output)
ripdoc print serde --verbose

# Disable source location labels
ripdoc print serde --no-source-labels
```

### 2. Search Within Crate Documentation

Use the `--search` or `-s` flag to query specific items instead of printing the entire crate. This returns matching public API items along with their ancestors for context.

**Basic search syntax:**
```sh
ripdoc print [TARGET] --search <term>
```

**Search examples:**
```sh
# Search for "status" across names, signatures, and doc comments (default)
ripdoc print reqwest --search status

# Search specific domains only
ripdoc print reqwest --search status --search-spec name,signature
ripdoc print reqwest --search error --search-spec doc
ripdoc print tokio --search spawn --search-spec name

# Case-sensitive search
ripdoc print reqwest --search "JSON" --search-case-sensitive

# Direct matches only (avoid auto-expanding parent containers)
ripdoc print tokio --search spawn --direct-match-only
ripdoc print tokio --search spawn -d
```

**Search domains:**
- `name` - Match against item names
- `signature` - Match against function/method signatures
- `doc` - Match against documentation comments
- `path` - Match against fully qualified paths

Default search spec when not specified: `name,doc,signature`

### 3. OR Searches

Use the pipe character `|` to search for multiple terms with OR logic. This works across all search domains.

**OR search examples:**
```sh
# Find multiple related items
ripdoc print gix --search "init|clone|fetch|remote|config"

# Search for multiple method names
ripdoc print tokio --search "spawn|block_on|sleep"

# Search in specific domain with OR
ripdoc print reqwest --search "get|post|put|delete" --search-spec name
```

### 4. List Crate Structure

Use `ripdoc list` to get a concise catalog of crate items without full documentation. Each line shows the item kind, fully qualified path, and source location.

**List syntax:**
```sh
ripdoc list [TARGET]
```

**List examples:**
```sh
# List all public items in tokio
ripdoc list tokio

# List with search filter
ripdoc list serde --search deserialize

# List with OR search
ripdoc list tokio --search "spawn|sleep|timeout"

# Discover exact paths
ripdoc list serde --search "Deserialize" --search-spec path
```

**Example output format:**
```
crate  tokio         tokio-1.48.0/src/lib.rs:1
module tokio::io     tokio-1.48.0/src/io/mod.rs:1
module tokio::net    tokio-1.48.0/src/net/mod.rs:1
struct tokio::net::TcpStream tokio-1.48.0/src/net/tcp/stream.rs:15
```

The listing respects `--private` and feature flags. Use this for quick structural overview or to find source locations.

### 5. Print README Files

Fetch and display the README file for a crate:

```sh
ripdoc readme [TARGET]
```

**Examples:**
```sh
ripdoc readme tokio
ripdoc readme serde@1.0.0
```

### 6. Skelebuild - Incremental Context Building

Use `ripdoc skelebuild` when you need to build incremental context for complex code understanding tasks. It creates a stateful Markdown "source map" by mixing API skeletons, implementation spans, raw source snippets, and commentary.

For detailed usage, run: `ripdoc skelebuild --help`

### 7. Raw JSON Output

Use `ripdoc raw` when you need the raw rustdoc JSON output for custom tooling or processing with jq.

For detailed usage, run: `ripdoc raw --help`

### 8. AI Agents Guide

Use `ripdoc agents` to get a dense usage guide optimized for AI agents. Run this command when you need comprehensive ripdoc usage information.

For detailed usage, run: `ripdoc agents --help`

## Workflow Patterns

### Pattern 1: Exploring an Unfamiliar Crate

When encountering a new crate:

1. **Get structural overview:**
   ```sh
   ripdoc list [crate_name]
   ```

2. **Read the README for context:**
   ```sh
   ripdoc readme [crate_name]
   ```

3. **Print top-level API:**
   ```sh
   ripdoc print [crate_name]
   ```

4. **Search for specific functionality:**
   ```sh
   ripdoc print [crate_name] --search [relevant_term]
   ```

### Pattern 2: Finding Specific Functionality

When looking for a specific function or feature:

1. **Search with relevant terms:**
   ```sh
   ripdoc print [crate_name] --search "term1|term2|term3"
   ```

2. **Narrow to name matches if too broad:**
   ```sh
   ripdoc print [crate_name] --search [term] --search-spec name
   ```

3. **Get direct matches only:**
   ```sh
   ripdoc print [crate_name] --search [term] -d
   ```

### Pattern 3: Understanding a Specific Type/Module

When examining a specific type or module:

1. **Print the specific item directly:**
   ```sh
   ripdoc print [crate_name]::[module]::[Type]
   ```

2. **Include implementation details:**
   ```sh
   ripdoc print [crate_name]::[Type] --implementation
   ```

3. **Search within that scope:**
   ```sh
   ripdoc print [crate_name]::[module] --search [method_name]
   ```

### Pattern 4: Working with Local Crates

When working with local Rust projects:

1. **Print current project API:**
   ```sh
   ripdoc print
   ```

2. **List workspace crates:**
   ```sh
   ripdoc list
   ```

3. **Print specific workspace member:**
   ```sh
   ripdoc print [workspace_member_name]
   ```

4. **Print with path:**
   ```sh
   ripdoc print /path/to/local/crate
   ```

### Pattern 5: Building Incremental Context

When building context for complex code understanding, use `ripdoc skelebuild` to incrementally build a Markdown context file with API skeletons, implementation spans, and commentary.

Run `ripdoc skelebuild --help` for detailed usage instructions.

## Output Interpretation

### Markdown Format (Default)

The default Markdown output is optimized for token efficiency:

````markdown
```rust
impl Client {
```

Creates a new HTTP client with default settings.

```rust
pub fn new() -> Client {}
```

Sends a GET request to the specified URL.

```rust
pub async fn get(&self, url: &str) -> Result<Response> {}
```
````

This format alternates between code blocks and documentation, making it easy to understand both API signatures and their usage.

### Rust Format

The `--format rust` option provides traditional Rust code with doc comments:

```rust
impl Client {
    /// Creates a new HTTP client with default settings.
    pub fn new() -> Client {}

    /// Sends a GET request to the specified URL.
    pub async fn get(&self, url: &str) -> Result<Response> {}
}
```

Use this format when generating or comparing against actual Rust source code.

### List Format

The list format shows items in a tabular structure:
```
[kind]  [fully::qualified::path]  [source_file:line]
```

Use this for quick navigation or understanding crate organization.

## Advanced Features

### Feature Flags

Enable crate features just like with cargo:

```sh
ripdoc print serde --features "derive,rc"
ripdoc print tokio --all-features
ripdoc print tokio --no-default-features
```

### Private Items

Include private items in the output:

```sh
ripdoc print my_crate --private
```

Useful when working with local crates and needing to see internal implementation.

### Implementation Mode

Include method bodies and full impl blocks:

```sh
ripdoc print serde::Deserialize --implementation
```

This is useful when you need to understand *how* something works, not just its API.

### Raw Source Mode

Include the entire source file for matched items:

```sh
ripdoc print ./my-crate crate::Config --raw-source
```

Useful when code isn't fully captured by rustdoc or you need surrounding context.

### Auto Trait Implementations

Show automatically implemented traits:

```sh
ripdoc print tokio --auto-impls
```

## Tips for Effective Usage

1. **Start broad, then narrow:** Begin with `ripdoc list` or `ripdoc print` without search, then use `--search` to drill down.

2. **Use OR searches liberally:** When unsure of exact terminology, search multiple terms: `--search "read|write|stream|buffer"`

3. **Combine search specs:** Use `--search-spec name,signature` to exclude noisy doc matches when looking for specific APIs.

4. **Leverage direct match:** Add `-d` flag when parent expansion creates too much output.

5. **Check README first:** Run `ripdoc readme [crate]` for high-level understanding before diving into API details.

6. **Use list for navigation:** The list command with search is excellent for finding source file locations.

7. **Version pinning:** When reproducing issues or working with specific versions, use `@version` syntax: `ripdoc print serde@1.0.195`

8. **Use skelebuild for complex analysis:** When understanding complex codebases, build context incrementally with `skelebuild`.

9. **Discover paths with list:** Use `ripdoc list <target> --search <name> --search-spec path` to find exact paths.

## Common Use Cases

### "How do I use [crate]?"
```sh
ripdoc readme [crate]
ripdoc print [crate]
```

### "Find all methods related to [topic]"
```sh
ripdoc print [crate] --search [topic] --search-spec name,signature
```

### "Show me the [Type] implementation"
```sh
ripdoc print [crate]::[Type]
ripdoc print [crate]::[Type] --implementation
```

### "What's available in [crate]::[module]?"
```sh
ripdoc print [crate]::[module]
ripdoc list [crate] --search [module]
```

### "Where is [function] defined?"
```sh
ripdoc list [crate] --search [function]
```

### "Build context for understanding [feature]"
```sh
ripdoc skelebuild --help  # See available subcommands
```

## Troubleshooting

### "No matches found"

1. Check the exact path: `ripdoc list <target> --search "<name>" --search-spec path --private`
2. The item might be re-exported; search by name to find the definition path
3. For private items, ensure you're using `--private`

### Missing items

- Feature-gated items need `--features` flag
- Private items need `--private` flag
- Proc-macro crates have limited rustdoc output

### Path confusion

- Bin crates use the binary name as crate root, not the package name
- Re-exports appear at their definition site, not the re-export location
- Use `ripdoc list --search <name> --search-spec path` to discover exact paths

### Issue: Nightly Toolchain Not Being Used

**Symptoms:**
```
error: the option `Z` is only accepted on the nightly compiler
```

**Root Cause:**
In Nix-based development environments, the system may have multiple cargo installations:
- Nix-managed cargo (typically stable version in `/nix/store/...`)
- rustup-managed cargo (in `~/.cargo/bin/`)

The Nix cargo may take precedence in PATH, causing ripdoc to use the stable toolchain even after installing nightly.

**Solutions:**

1. **Install ripdoc with nightly toolchain:**
   ```sh
   rustup run nightly cargo install ripdoc --force
   ```

2. **Ensure rustup cargo is in PATH before Nix cargo:**
   ```sh
   export PATH="$HOME/.cargo/bin:$PATH"
   ripdoc print [crate_name]
   ```

3. **Verify active toolchain:**
   ```sh
   rustup show
   which cargo
   cargo --version
   ```

4. **Set directory override (if needed):**
   ```sh
   rustup override set nightly
   ```

**Prevention:**
- Always install ripdoc using `rustup run nightly cargo install ripdoc`
- In Nix environments, consider prioritizing rustup's cargo in your PATH configuration
- Check `which cargo` to ensure you're using the rustup-managed version

### Issue: head Command Not Found in Nix Shell

**Symptoms:**
```
bash: line 1: head: command not found
```

**Root Cause:**
Some Nix environments may not include coreutils by default.

**Solution:**
Avoid using shell utilities like `head`, `tail` in command pipelines. Let ripdoc output naturally or redirect to files if needed.

## Additional Resources

For the most up-to-date and detailed usage information, use the built-in help commands:

```sh
ripdoc --help             # General help
ripdoc agents             # Dense usage overview for AI agents
ripdoc skelebuild --help  # Incremental context building
ripdoc raw --help         # Raw JSON output
```
