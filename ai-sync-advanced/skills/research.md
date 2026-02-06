---
name: research
description: Conduct research with up-to-date information via aichat CLI. Use when you need current information, real-time data, or need to research topics beyond your knowledge cutoff.
allowed-tools: Bash(aichat:*)
triggers:
  - "research"
  - "web search"
  - "look up"
  - "find out"
---

# AI Research Skill

Use research commands to query AI models for real-time web research.

## When to Use

This skill should be used when:
- You need current/real-time information beyond your knowledge cutoff
- Researching library APIs, software features, or documentation
- Verifying facts or getting up-to-date data
- User explicitly asks for web research

## Available Models

| Model | Speed | Use For |
|-------|-------|---------|
| `openrouter:x-ai/grok-4.1-fast` | 4-6s | Quick answers, iteration |
| `openrouter:x-ai/grok-4` | 10-15s | Code questions, reasoning |
| `openrouter:perplexity/sonar-pro` | 5-10s | Fast web search, citations |
| `openrouter:perplexity/sonar-reasoning-pro` | 15-30s | Multi-step reasoning + search |
| `openrouter:perplexity/sonar-deep-research` | 30-60s | Comprehensive research reports |

### Model Selection Guide

| Query Type | Recommended Model |
|------------|-------------------|
| Quick factual questions | `x-ai/grok-4.1-fast` |
| Code questions, debugging | `x-ai/grok-4` |
| Web search, current events | `perplexity/sonar-pro` |
| API docs, library features | `perplexity/sonar-pro` |
| Multi-step reasoning problems | `perplexity/sonar-reasoning-pro` |
| Comprehensive research reports | `perplexity/sonar-deep-research` |

## Error Handling

If `aichat` command is not found:
- Tell the user to install aichat from https://github.com/sigoden/aichat
- Use OpenRouter (https://openrouter.ai) for access to web search models like Perplexity and Grok

If API/authentication errors occur:
- Ask user to verify their OpenRouter API key in `~/.config/aichat/config.yaml`
- Check they have credits at https://openrouter.ai

If models are not found or OpenRouter is not configured:
- Ask the user to set up OpenRouter models in aichat
- They need an OpenRouter API key from https://openrouter.ai/keys
- Run these commands to add the research models:

```bash
# Add OpenRouter client with API key
aichat --set-client openrouter api_key=YOUR_OPENROUTER_API_KEY

# Add research models
aichat --set-model openrouter:x-ai/grok-4.1-fast
aichat --set-model openrouter:x-ai/grok-4
aichat --set-model openrouter:perplexity/sonar-pro
aichat --set-model openrouter:perplexity/sonar-reasoning-pro
aichat --set-model openrouter:perplexity/sonar-deep-research
```

- After setup, verify with: `aichat --list-models | grep openrouter`

---

## Instructions

### Quick Research (4-6 sec)

```bash
aichat -m openrouter:x-ai/grok-4.1-fast "your research question"
```

### Web Search (5-10 sec)

```bash
aichat -m openrouter:perplexity/sonar-pro "your research question"
```

### Multi-Step Reasoning (15-30 sec)

```bash
aichat -m openrouter:perplexity/sonar-reasoning-pro "explain the tradeoffs between X and Y"
```

### Deep Research Report (30-60 sec)

```bash
aichat -m openrouter:perplexity/sonar-deep-research "comprehensive analysis of [topic]"
```

### Code/Reasoning Questions

```bash
aichat -m openrouter:x-ai/grok-4 "how does React 19 handle concurrent rendering?"
```

### Save Output

```bash
aichat -m openrouter:perplexity/sonar-deep-research "Research topic" > research.md
```

## Best Practices

- **Quick iteration**: Use `grok-4.1-fast` for rapid feedback loops
- **Documentation lookup**: Use `perplexity/sonar-pro` - fast web search + citations
- **Code problems**: Use `grok-4` or `grok-4.1-fast` for code-related questions
- **Complex reasoning**: Use `perplexity/sonar-reasoning-pro` for multi-step problems
- **Comprehensive reports**: Use `perplexity/sonar-deep-research` for thorough research with many sources
- Include "provide sources" in your query for citations
- Be specific in your questions for better results
- For API documentation, ask for "official documentation" or "latest version"
