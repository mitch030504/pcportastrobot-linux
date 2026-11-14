# Session — RE MCP stack refresh

- **When:** 2026-11-14 11:43 CET
- **Agent:** opencode (DeepSeek V4.1 Flash)
- **Task:** Refresh the reverse-engineering MCP stack; it was pinned to old
  versions and the Ghidra GUI client was pointed at Steam.

## Changes

- **REA 6.1.0 -> 6.9.0** in `~/.config/opencode/opencode.jsonc`, and in the
  Claude/Gemini registrations (`~/.claude.json`,
  `~/.gemini/config/mcp_config.json`). REA 6.9 accepts the installed Ghidra
  **12.2** (`rea doctor`: `12.2; compatible 12.x (>=12.1)`); 6.1 rejected it.
  Backups: `*.bak-20261114`.
- **Ghidra GUI MCP port:** `127.0.0.1:8080` is owned by Steam's
  `steamwebhelper`, so the client was receiving Steam's 404. The client URL is
  now `http://127.0.0.1:18080/`; the plugin's Tool Option must be set to 18080
  too. Caveat: the latest GhidraMCP release targets Ghidra 12.1.4; the system
  has 12.2, so the GUI extension needs a source build for 12.2.
- **r2ghidra installed** with `r2pm -ci r2ghidra`; `pdg` now produces Ghidra C
  for a function without a full program import.
- Documented the stack in `Docs/REVERSE-ENGINEERING.md` and `.ai/PATHS.md`.

## Verification

- `GHIDRA_INSTALL_DIR=/opt/ghidra npx -y rea-agents@6.9.0 doctor` -> Ghidra
  12.2 healthy.
- REA 6.9 MCP imported `/usr/bin/ls` with `provider_version: 12.2`.
- `r2 -q -c 's 0x2a57e0; af; pdg' Build/elf-input/eboot.elf` produced Ghidra
  pseudocode for the resolution manager.

## Notes

- Large binaries: REA's full import of the 247 MB eboot can exceed the 330 s
  analysis timeout; use r2 `pdg` per function or a prebuilt Ghidra
  project/snapshot.
- Config changes need an OpenCode/Claude/Gemini restart to be picked up; the
  REA 6.9 MCP server in this session reloaded live.

## GhidraMCP extension (follow-up)

- Built the extension from source against Ghidra 12.2:
  `GHIDRA_INSTALL_DIR=/opt/ghidra /opt/ghidra/support/gradle/gradlew -PGHIDRA_INSTALL_DIR=/opt/ghidra`.
- Changed the plugin default port from 8080 to **18080** in the source before
  building (Steam owns 8080).
- Installed
  `ghidra_12.2_DEV_20261114_GhidraMCP.zip` in
  `~/.config/ghidra/ghidra_12.2_DEV/Extensions/Ghidra/`.
- Next: restart Ghidra, enable `GhidraMCPPlugin` under
  `File -> Configure -> Developer`; the backend should listen on 18080 and the
  MCP client's `get_program_info` should work.
