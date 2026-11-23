# AGENT.md

This directory is a **collection of embedded / SBC projects**, not a single project.

It covers boards and platforms such as Arduino, ESP32, STM32, Raspberry Pi, and similar MCUs / SBCs.

## Language

**Everything in this repo must be in English**: docs, comments, commit messages, folder/file names, serial/log strings, and agent-facing notes. Do not use other languages in project files.

Chat with the human may be Indonesian; repo files stay English.

## Layout

Each project lives in its own folder at the root:

```
arduino-v2/
├── AGENT.md
├── README.md
├── <project-name-1>/
├── <project-name-2>/
└── ...
```

## Rules for agents

- One folder = one project. Do not mix sources across projects.
- When working on a project, keep changes inside that project's folder (unless the user asks for cross-project work).
- New project = new folder at the root. Do not put firmware or app sources in the `arduino-v2/` root.
- Root is only for shared docs (`AGENT.md`, `README.md`) and project folders.
- **KISS**: smallest working change. No new abstraction, dependency, or refactor unless asked.
- Do not add Arduino libraries without explicit permission.
- Commit only when the user asks. Never commit secrets (`.env`, keys, passwords).
- After firmware changes: **`arduino-cli compile` must succeed** before suggesting upload.
- Do not flash/upload until the user confirms the board is ready (boot mode / port).
- If toolchain facts in this file are wrong or stale (CLI version, core path/version, library versions, tool paths, ports, etc.), the agent **may and should update `AGENT.md`** to match the machine. Append a row to the **AGENT.md change log** table below (newest last). Do not invent versions — verify with `arduino-cli` / filesystem first.

## Toolchain (this machine)

**Mandatory:** use **arduino-cli** for compile, upload, monitor, board/core/lib management. Do not use Arduino IDE GUI, PlatformIO, or ad-hoc compilers as the primary path.

| Tool / path                     | Location                                                                               |
| :------------------------------ | :------------------------------------------------------------------------------------- |
| `arduino-cli` binary            | `/usr/bin/arduino-cli` (v1.5.1)                                                        |
| CLI data dir                    | `/home/kai/.arduino15`                                                                 |
| Sketchbook (`directories.user`) | `/home/kai/Arduino`                                                                    |
| User libraries                  | `/home/kai/Arduino/libraries`                                                          |
| Board packages / cores          | `/home/kai/.arduino15/packages`                                                        |
| ESP32 Arduino core              | `/home/kai/.arduino15/packages/esp32/hardware/esp32/3.3.12` (`esp32:esp32` **3.3.12**) |
| `esptool` (bundled)             | `/home/kai/.arduino15/packages/esp32/tools/esptool_py/5.3.1/esptool`                   |
| `mklittlefs` (bundled)          | `/home/kai/.arduino15/packages/esp32/tools/mklittlefs/4.0.2-db0513a`                   |
| Python                          | `/usr/bin/python3`                                                                     |

Not installed as PATH tools: `platformio` / `pio`, system `esptool.py`. Prefer the bundled `esptool` above or `arduino-cli upload`.

### Installed user libraries

Under `/home/kai/Arduino/libraries`:

| Library                   | Version (approx.) |
| :------------------------ | :---------------- |
| ArduinoJson               | 7.4.2             |
| ArduinoWebsockets         | 0.5.4             |
| ESP32Servo                | 3.2.1             |
| RF24                      | 1.6.2             |
| SmartRC-CC1101-Driver-Lib | 3.0.2             |
| WebSockets                | 2.6.1             |

List / install via CLI only:

```bash
arduino-cli lib list
arduino-cli lib install 'LibraryName'
```

### Common CLI

```bash
cd /home/kai/Documents/ywow/arduino-v2/<project>

arduino-cli board list
arduino-cli core list
arduino-cli compile --fqbn '<FQBN>' .
arduino-cli upload -p /dev/ttyUSB0 --fqbn '<FQBN>' .
arduino-cli monitor -p /dev/ttyUSB0 -c baudrate=115200
```

- Default serial baud: **115200**.
- Ports: `/dev/ttyUSB0` (FTDI / many UART adapters), `/dev/ttyACM0` (native USB CDC). Always re-check with `arduino-cli board list`.
- FQBN comes from the project README (or ask). Examples: `esp32:esp32:wt32-eth01`, `esp32:esp32:esp32c3:CDCOnBoot=cdc`, `esp32:esp32:esp32s3:CDCOnBoot=cdc,FlashSize=16M,PartitionScheme=default_8MB,PSRAM=opi`.

### IDE note

If VS Code Arduino extension is used: `arduino.useArduinoCli: true`. Agents still drive builds through the CLI, not the IDE UI.

## AGENT.md change log

When correcting toolchain / path / version info in this file, add one row:

| Date (UTC+7) | What changed              | Old → new (short)                                  | Verified how                                                                                         |
| :----------- | :------------------------ | :------------------------------------------------- | :--------------------------------------------------------------------------------------------------- |
| 2026-11-24   | Initial toolchain section | — → CLI 1.5.1, esp32 core 3.3.12, user libs listed | `arduino-cli version`, `core list`, `lib list`, paths under `~/.arduino15` and `~/Arduino/libraries` |
