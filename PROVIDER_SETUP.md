# Multi-Provider Setup Guide

This document describes the multi-provider configuration for OpenDev, enabling you to use different LLM providers for different workflow slots.

## Changes Summary

This branch adds support for **11 LLM providers** (up from 9), including:
- **NVIDIA NIM** (`nvidia`) - Primary provider for Normal, Thinking, and VLM slots
- **Cerebras** (`cerebras`) - High-speed Compact slot provider
- **Groq** (`llama-3.3-70b-versatile`) - Low-latency Critique slot provider

### Modified Files

| File | Change |
|------|--------|
| `crates/opendev-cli/src/runtime.rs` | Added NVIDIA/Cerebras API endpoints |
| `crates/opendev-http/src/auth.rs` | Added `NVIDIA_API_KEY`, `CEREBRAS_API_KEY` env mappings |
| `crates/opendev-models/src/config.rs` | Added env var resolution for new providers |
| `crates/opendev-config/src/models_dev.rs` | Added providers to priority list |
| `docs/providers.md` | Updated with model recommendations |
| `README.md` | Updated provider count (9 → 11) |

---

## Current Status & Known Working Models

**Tested and Verified (as of March 2026):**

| Slot | Provider | Model | Status | Notes |
|------|----------|-------|--------|-------|
| **Normal** | NVIDIA | `moonshotai/kimi-k2-instruct` | ✅ Working | Excellent for coding and tool use |
| **Thinking** | NVIDIA | `z-ai/glm5` | ✅ Working | Strong reasoning, ~20-30s response time |
| **Thinking (alt)** | NVIDIA | `meta/llama-3.3-70b-instruct` | ✅ Working | Faster than glm5, good reasoning |
| **Compact** | Cerebras | `qwen-3-235b-a22b-instruct-2507` | ✅ Working | 131K context, fast summarization |
| **Critique** | Groq | `llama-3.3-70b-versatile` | ✅ Working | Ultra-low latency (~400 tok/s) |
| **VLM** | NVIDIA | `Llama-3.2-90B-Vision-Instruct` | ⚠️ Untested | Standard vision model |

**Notes:**
- `deepseek-ai/deepseek-r1` is deprecated (ended 2026-01-26)
- `gpt-oss-120b` on Cerebras may not be available on all accounts
- Thinking models add latency but improve plan quality

OpenDev's compound AI architecture allows you to bind different models to different workflow slots. Here's the recommended configuration:

### Workflow Slot Assignments

| Slot | Provider | Model | Context | Why |
|------|----------|-------|---------|-----|
| **Normal** | NVIDIA | `moonshotai/kimi-k2-instruct` | 256K | Excellent tool use, coding, agentic capabilities |
| **Thinking** | NVIDIA | `moonshotai/kimi-k2-thinking` | 256K | Deep reasoning and planning (thinking model) |
| **Compact** | Cerebras | `qwen-3-235b-a22b-instruct-2507` | 131K | High-quality summarization (131K context) |
| **Critique** | Groq | `llama-3.3-70b-versatile` | 128K | Ultra-low latency analysis (~400 tok/s) |
| **VLM** | NVIDIA | `Llama-3.2-90B-Vision-Instruct` | - | Native vision understanding |

### Environment Variables

Add these to your `~/.bashrc`, `~/.zshrc`, or `.env` file:

```bash
# NVIDIA NIM (3 slots: Normal, Thinking, VLM)
export NVIDIA_API_KEY="nvapi-..."

# Cerebras (1 slot: Compact)
export CEREBRAS_API_KEY="csk-..."

# Groq (1 slot: Critique)
export GROQ_API_KEY="gsk_..."

# Optional: OpenRouter backup
export OPENROUTER_API_KEY="sk-or-..."
```

### Configuration File

Create `~/.opendev/settings.json`:

```json
{
  "model_provider": "nvidia",
  "model": "moonshotai/kimi-k2-instruct",
  
  "model_thinking_provider": "nvidia",
  "model_thinking": "z-ai/glm5",
  
  "model_compact_provider": "cerebras",
  "model_compact": "qwen-3-235b-a22b-instruct-2507",
  
  "model_critique_provider": "groq",
  "model_critique": "llama-3.3-70b-versatile",
  
  "model_vlm_provider": "nvidia",
  "model_vlm": "Llama-3.2-90B-Vision-Instruct",
  
  "max_tokens": 4096,
  "temperature": 0.6
}
```

---

## Getting API Keys

### NVIDIA NIM
1. Visit: https://build.nvidia.com/models
2. Sign up for free account
3. Generate API key
4. Free tier: ~40 RPM, 200K tokens/min

### Cerebras
1. Visit: https://cloud.cerebras.ai/
2. Sign up (no credit card required)
3. Generate API key
4. Free tier: 1M tokens/day, **64K context** (131K on paid tier)

**Note:** Use `qwen-3-235b-a22b-instruct-2507` for the Compact slot. The `gpt-oss-120b` model may not be available on all accounts.
**Important:** Free tier has 64K context limit - not suitable for very long conversation compaction. Best for Critique slot (short analysis tasks).

### Groq
1. Visit: https://console.groq.com/
2. Sign up (no credit card required)
3. Generate API key
4. Free tier: ~1K requests/day

---

## Testing Your Configuration

### Quick Test

```bash
# Test Normal slot (NVIDIA Kimi K2 Instruct)
export NVIDIA_API_KEY="nvapi-..."
opendev -p "Explain what this code does: $(cat some_file.py)"

# Test Thinking slot (NVIDIA Kimi K2 Thinking)
opendev -p "Plan how to add authentication to this Flask app"

# Test Compact slot (Cerebras Qwen 3 235B)
# Automatically triggered when context gets long

# Test Critique slot (Groq Llama 3.3)
# Automatically used in plan mode for verification

# Test VLM slot (NVIDIA Llama 3.2 Vision)
opendev -p "What does this screenshot show?" --image ./screenshot.png
```

### Test Project Suggestions

Good projects to test OpenDev's capabilities:

1. **Code Refactoring**
   - "Refactor this module to use async/await"
   - "Add type hints to this Python file"
   - "Extract this function into a separate module"

2. **Feature Implementation**
   - "Add a REST API endpoint for user registration"
   - "Implement caching for database queries"
   - "Add logging to this application"

3. **Code Understanding**
   - "Explain the architecture of this codebase"
   - "Find all security vulnerabilities"
   - "Generate documentation for this module"

4. **Multi-File Changes**
   - "Add dark mode to this web app"
   - "Migrate from Flask to FastAPI"
   - "Add unit tests for this module"

---

## Portability - Moving to Another Computer

To replicate this setup on another computer:

### Required Files

1. **Binary** (21.9MB):
   ```
   target/release/opendev
   ```

2. **Configuration**:
   ```
   ~/.opendev/settings.json    # Your multi-provider config
   ~/.opendev/auth.json        # Stored credentials (optional)
   ```

3. **Environment Variables** (set in shell config):
   ```bash
   export NVIDIA_API_KEY="nvapi-..."
   export CEREBRAS_API_KEY="csk-..."
   export GROQ_API_KEY="gsk_..."
   ```

   **Alternative:** Copy the `.env` file to `~/.opendev/.env` and source it:
   ```bash
   cp .env ~/.opendev/.env
   source ~/.opendev/.env  # Run once per shell session
   ```

   Or add to your `~/.bashrc` or `~/.zshrc`:
   ```bash
   source ~/.opendev/.env  # Auto-load on shell startup
   ```

### Setup Script

Create a setup script for the new machine:

```bash
#!/bin/bash
# setup-opendev.sh

# Copy binary
cp opendev ~/.local/bin/

# Create config directory
mkdir -p ~/.opendev

# Copy settings
cp settings.json ~/.opendev/

# Set environment variables (add to ~/.bashrc or ~/.zshrc)
cat >> ~/.bashrc << 'EOF'
export NVIDIA_API_KEY="nvapi-..."
export CEREBRAS_API_KEY="csk-..."
export GROQ_API_KEY="gsk_..."
EOF

echo "OpenDev setup complete!"
```

### Quick Test After Setup

```bash
# Verify installation
opendev --version

# Test configuration
opendev config show

# Run a simple query
echo "Hello" | opendev -p "Respond with OK"
```

---

## Troubleshooting

### "Header of type `authorization` was missing"

**Cause:** Windows-style line endings (CRLF) in `~/.opendev/.env` file.

**Fix:**
```bash
sed -i 's/\r$//' ~/.opendev/.env
```

Then restart your shell or re-source the file:
```bash
source ~/.opendev/.env
```

**Prevention:** Create `.env` files with Unix line endings (LF only).

### "HTTP 400" or "context_length_exceeded"

- **Cause**: Model's context window exceeded
- **Fix**: Use `gpt-oss-120b` or `qwen-3-235b-a22b-instruct-2507` for Cerebras (131K context)

### "HTTP 429" Rate Limited

- **Cause**: Too many requests
- **Fix**: Wait a few minutes, or switch to a different provider for that slot

### "Model not found"

- **Cause**: Model ID incorrect or not available
- **Fix**: Check provider documentation for current model IDs

### VLM Not Working

- **Cause**: Normal model doesn't support vision
- **Fix**: Ensure `model_vlm_provider` is set to NVIDIA with a vision-capable model

---

## Cost Estimates (Free Tiers)

| Provider | Free Tier | Paid Plans |
|----------|-----------|------------|
| NVIDIA | ~200K tokens/min | Pay per token |
| Cerebras | 1M tokens/day | $50/month unlimited |
| Groq | ~1K requests/day | Pay per token |

**Estimated daily usage for typical development**: Well within free tiers for individual use.

---

## Support

- Documentation: https://github.com/PropertySightlines/opendev/tree/main/docs
- Issues: https://github.com/PropertySightlines/opendev/issues
- Provider docs: https://github.com/PropertySightlines/opendev/blob/main/docs/providers.md
