---
name: Bug Report - CRLF Line Endings in .env
about: Windows line endings in .env file cause authorization header failures
title: '[BUG] Windows CRLF line endings in .env cause "authorization header missing" error'
labels: 'bug, good first issue, configuration'
---

## Bug Description
When the `.env` file (or `~/.opendev/.env`) has Windows-style CRLF (`\r\n`) line endings, API keys contain a trailing `\r` character. This causes `HeaderValue::from_str()` to reject the Authorization header value, resulting in "Header of type `authorization` was missing" errors from LLM providers.

## Root Cause
Rust's `http::HeaderValue::from_str()` rejects header values containing CR (`\r`) or LF (`\n`) characters as per HTTP specification. When `.env` files are created or edited with Windows line endings, the API key includes `\r` at the end.

## Reproduction
1. Create `.env` file with Windows line endings (CRLF)
2. Source the file: `source .env`
3. Run opendev with any provider
4. Observe: "authorization header missing" error

## Workaround
```bash
# Fix line endings
sed -i 's/\r$//' ~/.opendev/.env
# Or for project .env
sed -i 's/\r$//' .env
```

## Proposed Fix
1. Trim whitespace from API keys when loading from environment or config files
2. Add validation in `config.get_api_key()` to strip `\r\n` characters:
   ```rust
   std::env::var(env_var)
       .map(|s| s.trim().to_string())
       .map_err(|_| format!("No API key found. Set {} environment variable", env_var))
   ```
3. Document in README/PROVIDER_SETUP.md

## Environment
- OpenDev version: 0.1.0
- Affects: Windows users, users editing .env with Windows-aware editors (Winterm, VS Code with CRLF settings, etc.)
- All providers affected

## Additional Context
This issue commonly occurs when:
- Using Winterm's built-in editor
- VS Code with `files.eol` set to `\r\n`
- Copy/paste from Windows clipboard
- Git's `core.autocrlf=true` setting on Linux

Recommended: Set `git config --global core.autocrlf input` on Linux/Mac.
