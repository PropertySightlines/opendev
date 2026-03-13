---
name: Bug Report - spawn_subagent Not Available
about: The spawn_subagent tool is not registered, preventing subagent usage
title: '[BUG] spawn_subagent tool not available ("Unknown tool: spawn_subagent")'
labels: 'bug, enhancement, agents'
---

## Bug Description
The `spawn_subagent` tool is not registered in the tool registry, causing "Unknown tool: spawn_subagent" errors when the LLM tries to spawn subagents (Planner, Code-Explorer, etc.).

## Root Cause
In `crates/opendev-cli/src/runtime.rs`, the `SpawnSubagentTool` requires:
- `Arc<ToolRegistry>` 
- `Arc<AdaptedClient>`

However, these are created AFTER `register_default_tools()` is called. The comment at line 105-106 states:
```rust
// Note: SpawnSubagentTool requires shared Arc<ToolRegistry> and Arc<HttpClient>,
// which are created after registration. Deferred for now.
```

This "deferred" registration was never implemented.

## Impact
- All 5 subagents (code_explorer, planner, web_clone, web_generator, ask_user) are unusable
- LLM cannot delegate complex tasks to specialized subagents
- Reduces OpenDev's compound AI capabilities

## Proposed Fix
1. Make `ToolRegistry` cloneable (wrap internal HashMap in Arc)
2. Register `SpawnSubagentTool` after HTTP client creation but before wrapping ToolRegistry in Arc
3. Update `AgentRuntime` struct fields to use `Arc<ToolRegistry>` and `Arc<AdaptedClient>`

## Environment
- OpenDev version: 0.1.0
- All platforms affected

## Additional Context
This was discovered and fixed during multi-provider setup implementation. The fix involves:
- Changing `ToolRegistry.tools` from `HashMap` to `Arc<HashMap>`
- Using `Arc::make_mut()` for mutable access in register/unregister methods
- Registering SpawnSubagentTool after HTTP client creation in `AgentRuntime::new()`
