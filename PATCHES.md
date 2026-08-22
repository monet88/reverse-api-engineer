# Local Patches — Custom Browser Executable Path

Local-only changes made by the reverse-api agent, not yet merged upstream.
Rebuild after `uv sync` / `uv tool reinstall` — `uv.lock` may wipe edits in `site-packages`
or the editable install.

## Change summary

Adds an `executable_path` config option, forwarded to the Playwright MCP server
(`rae-playwright-mcp`) as `--executable-path`, so RAE launches a custom
Chromium/Chrome build (e.g. cloakbrowser) instead of the stock browser.

## Files touched

| File | Change |
|------|--------|
| `src/reverse_api/config.py` | `DEFAULT_CONFIG["executable_path"] = None` |
| `src/reverse_api/cli.py` | resolve `executable_path` from `RAE_EXECUTABLE_PATH` env or config; pass to all 4 auto-engineer constructors (opencode/copilot/claude/cursor) |
| `src/reverse_api/auto_engineer.py` | `_build_playwright_mcp_args()` shared helper; `kwargs.pop("executable_path", None)` in `ClaudeAutoEngineer`, `OpenCodeAutoEngineer`, `CopilotAutoEngineer` `__init__`; use helper in `_get_mcp_config`, `_get_opencode_mcp_config`, and Copilot `pw_args` block |
| `src/reverse_api/cursor_engineer.py` | same `kwargs.pop("executable_path", None)` in `CursorAutoEngineer.__init__`; import and use `_build_playwright_mcp_args` in `_cursor_mcp_servers` |

## Current branch state

- `main` = `83308ef` (upstream HEAD, force-pushed back to drop the feature commit)
- `feat/custom-executable-path` = `ffc26df` (the patch commit)
- PR: https://github.com/monet88/reverse-api-engineer/pull/1

## User config (runtime)

`~/.reverse-api/config.json` already has:

```json
"executable_path": "C:\\Users\\monet\\.cloakbrowser\\chromium-146.0.7680.177.5\\chrome.exe"
```

## Verify

```bash
python -m py_compile src/reverse_api/auto_engineer.py src/reverse_api/cli.py src/reverse_api/config.py src/reverse_api/cursor_engineer.py
```

Config load check:

```bash
python -c "from pathlib import Path; from reverse_api.config import ConfigManager; print(ConfigManager(Path.home()/'.reverse-api'/'config.json').get('executable_path'))"
```

## Caveat

Patching `site-packages` (the `uv tool install` copy at
`C:\Users\monet\AppData\Roaming\uv\tools\reverse-api-engineer`) gets wiped on
`uv tool upgrade reverse-api-engineer`. To use this fork long-term, install
editable instead:

```bash
uv tool install -e F:\CodeBase\reverse-api-engineer
```
