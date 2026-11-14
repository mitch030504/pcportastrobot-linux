# Reverse-engineering tooling

Last updated: 2026-11-14. This is the working stack for the AstroHybrid RE work
(FSR4 resolution selection, Kyty/AnyPS5 patches). It replaces the older setup
where REA was pinned to 6.1.0 (Ghidra 12.1 only) and the Ghidra GUI MCP was
pointed at a port owned by Steam.

## Working stack

| Tool | Version / path | Use |
|---|---|---|
| REA | `rea-agents@6.9.0`, Ghidra `12.2` at `/opt/ghidra`, Java 27 | headless Ghidra decompiles, function dossiers, xrefs |
| radare2 | `6.2.2` + `r2ghidra` | fast per-function decompile (`pdg`) on huge binaries |
| Ghidra GUI MCP | LaurieWired GhidraMCP | optional interactive GUI; see caveats |
| decomp.wtf | `decomp` MCP | matching decompilation projects |

`rea doctor` with `GHIDRA_INSTALL_DIR=/opt/ghidra` reports `ghidra-version`
healthy: `12.2; compatible 12.x (>=12.1)`. Hopper/IDA/JADX/Kuna are optional and
not installed; that does not affect the Ghidra provider.

## Quick checks

```sh
# REA provider
GHIDRA_INSTALL_DIR=/opt/ghidra npx -y rea-agents@6.9.0 doctor

# r2ghidra (should print C, not an error)
r2 -q -c 's 0x2a57e0; af; pdg' Build/elf-input/eboot.elf | head

# xrefs / function inventory
r2 -q -c 'axt @ 0x2a70e2' Build/elf-input/eboot.elf
r2 -q -c 'afl~main' <binary>
```

## Large binaries (Astro Bot eboot, 247 MB)

- REA's full Ghidra import can exceed the 330 s analysis timeout. Use r2 for
  per-function work; use REA on smaller artifacts, or a prebuilt Ghidra project
  / snapshot for repeated queries.
- Address mapping:
  - `Build/elf-input/eboot.elf` (original): first LOAD maps file `0x4000` to VA
    `0`, so **VA = file offset - 0x4000**.
  - `Build/RunHybrid/AstroBot` (relinked): first LOAD maps file `0x4000` to VA
    `0x400000`, so **VA = file offset + 0x3FC000**.
  - Example: the resolution manager is original VA `0x2a57e0`, relinked VA
    `0x6a57e0`, both at file offset `0x2a97e0`.
- The original eboot is the better target for Ghidra/r2 analysis; the relinked
  binary is for patches and runtime addresses.

## Ghidra GUI MCP caveats

- Steam's `steamwebhelper` owns `127.0.0.1:8080`; the GhidraMCP client used to
  point there and received Steam's 404. The client URL is now
  `http://127.0.0.1:18080/`.
- Set the plugin port to match: Ghidra -> `Edit` -> `Tool Options` ->
  `GhidraMCP HTTP Server` -> `Server Port` = `18080`, then restart Ghidra.
- The latest GhidraMCP release targets Ghidra **12.1.4**; the system Ghidra is
  **12.2**. For 12.2, build the extension from source against `/opt/ghidra`, or
  use REA/r2 instead. Do not expect the 12.1.4 release zip to load in 12.2.
- If the GUI MCP is not needed, leave it configured but idle; REA and r2 cover
  headless work.

## Config changes made (2026-11-14)

- `~/.config/opencode/opencode.jsonc`: `rea-agents@6.1.0` -> `6.9.0`
  (backup: `opencode.jsonc.bak-20261114`).
- `~/.config/opencode/opencode.json`: GhidraMCP client
  `http://127.0.0.1:8080/` -> `http://127.0.0.1:18080/`
  (backup: `opencode.json.bak-20261114`).
- `~/.claude.json`, `~/.gemini/config/mcp_config.json`: REA registration
  `6.1.0` -> `6.9.0` (backups: `*.bak-20261114`).
- `r2pm -ci r2ghidra` installed the Ghidra decompiler plugin for radare2.

MCP client config changes take effect after the client (OpenCode/Claude/Gemini)
is restarted. The REA 6.9 server used for this session was loaded live.
