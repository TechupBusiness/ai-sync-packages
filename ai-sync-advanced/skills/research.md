---
name: research
description: Conduct research with up-to-date information via aichat CLI. Use when you need current information, real-time data, or need to research topics beyond your knowledge cutoff.
version: 1.1.0
compatibility: Requires aichat CLI with OpenRouter API key configured
allowed-tools: Bash
---

# AI Research Skill

Use research commands to query AI models for real-time web research.

## When to Use

This skill should be used when:
- You need current/real-time information beyond your knowledge cutoff
- Researching library APIs, software features, or documentation
- Verifying facts or getting up-to-date data
- User explicitly asks for web research

## Research Commands

The following wrapper scripts provide short aliases for aichat models. Users can customize these by creating their own scripts in `~/.local/bin/`.

| Command | Default Model | Speed | Use For |
|---------|---------------|-------|---------|
| `research-grok-fast` | Grok 4.1 Fast | 4-6s | Quick answers, iteration |
| `research-grok` | Grok 4 | 10-15s | Code questions, reasoning |
| `research-perplexity` | Perplexity Deep | 20-40s | Web search, docs, citations |
| `research-claude` | Claude Sonnet 4.5 | 10-20s | Deep analysis, complex problems |

### Model Selection Guide

| Query Type | Recommended Command |
|------------|---------------------|
| Quick factual questions | `research-grok-fast` |
| Code questions, debugging | `research-grok` or `research-grok-fast` |
| Web search, current events | `research-perplexity` |
| API docs, library features | `research-perplexity` |
| Deep analysis, architecture | `research-claude` |
| Complex problem solving | `research-claude` |

## Setup (First-Time Only)

### 1. Create Wrapper Scripts

Create these scripts in `~/.local/bin/` (ensure this directory is in your PATH):

```bash
# Create directory if needed
mkdir -p ~/.local/bin

# research-grok-fast
cat > ~/.local/bin/research-grok-fast << 'EOF'
#!/bin/bash
exec aichat -m openrouter:x-ai/grok-4.1-fast "$@"
EOF

# research-grok
cat > ~/.local/bin/research-grok << 'EOF'
#!/bin/bash
exec aichat -m openrouter:x-ai/grok-4 "$@"
EOF

# research-perplexity
cat > ~/.local/bin/research-perplexity << 'EOF'
#!/bin/bash
exec aichat -m openrouter:perplexity/sonar-deep-research "$@"
EOF

# research-claude
cat > ~/.local/bin/research-claude << 'EOF'
#!/bin/bash
exec aichat -m openrouter:anthropic/claude-sonnet-4.5 "$@"
EOF

# Make executable
chmod +x ~/.local/bin/research-*
```

### 2. Install aichat (if not installed)

```bash
# Check if aichat is available
which aichat || command -v aichat

# macOS (Homebrew)
brew install aichat

# Linux (Cargo)
cargo install aichat

# Or download binary from: https://github.com/sigoden/aichat/releases
```

### 3. Configure OpenRouter

Get an API key from [openrouter.ai](https://openrouter.ai/keys), then configure:

```bash
mkdir -p ~/.config/aichat

cat > ~/.config/aichat/config.yaml << 'EOF'
model: openrouter:x-ai/grok-4.1-fast
clients:
  - type: openai-compatible
    name: openrouter
    api_base: https://openrouter.ai/api/v1
    api_key: YOUR_OPENROUTER_API_KEY
    models:
      - name: x-ai/grok-4.1-fast
      - name: x-ai/grok-4
      - name: perplexity/sonar-deep-research
      - name: anthropic/claude-sonnet-4.5
EOF
```

### 4. Verify Setup

```bash
research-grok-fast "What is 2+2?"
```

---

## Instructions

### Quick Research (4-6 sec)

```bash
research-grok-fast "your research question"
```

### Deep Web Research (20-40 sec)

```bash
research-perplexity "your research question"
```

### Code/Reasoning Questions

```bash
research-grok "how does React 19 handle concurrent rendering?"
```

### Complex Analysis

```bash
research-claude "compare microservices vs monolith architecture for this use case"
```

### Parallel Research (Recommended)

Run multiple models in parallel for comprehensive results:

```bash
research-grok-fast "What are the latest React 19 features?" &
research-perplexity "What are the latest React 19 features?" &
wait
```

### Save Output

```bash
research-perplexity "Research topic" > research.md
```

## Best Practices

- **Quick iteration**: Use `research-grok-fast` for rapid feedback loops
- **Documentation lookup**: Use `research-perplexity` - it has web search + citations
- **Code problems**: Use `research-grok` or `research-grok-fast` for code-related questions
- **Architecture decisions**: Use `research-claude` for nuanced analysis
- Include "provide sources" in your query for citations
- Be specific in your questions for better results
- For API documentation, ask for "official documentation" or "latest version"

## Customization

To use different models, edit the wrapper scripts in `~/.local/bin/`. For example, to use a different provider:

```bash
# Edit ~/.local/bin/research-grok-fast
#!/bin/bash
exec aichat -m your-provider:your-model "$@"
```
