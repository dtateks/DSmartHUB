# Project Overview
- **Tasmota** is an alternative firmware for ESP8266 and ESP32 based devices, providing local control via MQTT, HTTP, Serial, or KNX, along with a WebUI, OTA updates, and automation features.
- **Tech Stack**: C/C++ (Arduino framework), PlatformIO, LittleFS.
- **Key Components**:
    - **Core**: Modified Arduino core for ESP8266/ESP32.
    - **Build System**: PlatformIO with Python scripts for build automation.
    - **Web UI**: Embedded web server for configuration and control.

# Build & Run Commands
- **Install Dependencies**: Install PlatformIO (usually via VS Code extension or CLI).
- **Build Firmware**:
    - Run `pio run` to build the default environment.
    - Build specific environments: `pio run -e <environment_name>` (e.g., `tasmota`, `tasmota32`, `tasmota-lite`).
    - See `platformio.ini` and `BUILDS.md` for available environments.
- **Flash Firmware**:
    - **Serial**: `pio run -t upload` (requires device connected via serial).
    - **OTA**: Use the WebUI of an existing Tasmota device or `http://ota.tasmota.com`.
- **Custom Configuration**:
    - Copy `tasmota/user_config_override_sample.h` to `tasmota/user_config_override.h`.
    - Uncomment `-DUSE_CONFIG_OVERRIDE` in `platformio.ini` (or ensure it's active).
    - Add custom `#define` settings in `user_config_override.h`.

# Test & Quality Commands
- **CI/CD**: GitHub Actions workflows (`.github/workflows/`) handle automated builds and checks.
- **Static Analysis**: PlatformIO `check` tool (configured in `platformio.ini`).
- **Manual Testing**: Flash to a test device and verify functionality.
- **Linting/Formatting**: Follow existing code style. Avoid auto-formatting unless configured to match the project style exactly.

# Architecture & Code Structure
- **`tasmota/`**: Main source code directory.
    - `tasmota.ino`: Main entry point (Arduino sketch structure).
    - `*.ino`: Modules and features (often split into multiple `.ino` files).
    - `*.h`: Header files for configuration and definitions.
- **`lib/`**: External and internal libraries.
- **`boards/`**: Board definitions.
- **`pio-tools/`**: Python scripts for PlatformIO build process (pre/post scripts).
- **`tools/`**: Miscellaneous tools.
- **`variants/`**: Variant-specific configurations.

# Coding Style & Conventions
- **Language**: C/C++ (Arduino style).
- **Formatting**:
    - **Indentation**: 2 spaces.
    - **Braces**: K&R style (opening brace on the same line).
    - **Naming**: CamelCase for functions/classes, UPPERCASE for macros/constants.
- **Best Practices**:
    - **Memory**: Critical constraint (especially on ESP8266). Avoid large static allocations or excessive heap usage.
    - **Flash**: Code size matters. Use `#ifdef` to guard optional features.
    - **Non-Blocking**: The main loop must not block. Use state machines and timers.
    - **No `delay()`**: Avoid `delay()` in the main loop; use `TasmotaGlobal.uptime` or similar timers.

# Testing Strategy
- **Unit Tests**: Limited unit testing infrastructure visible in root. Reliance on integration/system testing on actual hardware.
- **Integration Tests**: "Ensure tests work" implies verifying the build and functionality on hardware.
- **CI**: Automated builds ensure compilation across variants.

# Environment & Secrets
- **Secrets**: **NEVER** commit `user_config_override.h` with real Wi-Fi/MQTT credentials. Use placeholders or local-only files.
- **Environment Variables**: PlatformIO handles build environments.
- **Local Development**:
    - Use `user_config_override.h` for local settings.
    - `platformio_override.ini` can be used to override `platformio.ini` settings locally.

# Workflow & Common Tasks
- **Contribution Workflow**:
    1. Fork the repo.
    2. Create a branch from `development`.
    3. Implement feature/fix.
    4. Verify compilation (`pio run`).
    5. Open PR against `development`.
    6. Sign CLA (check box in PR template).
- **Adding New Device**:
    - Check `templates.blakadder.com` first.
    - Use "Templates" feature in WebUI if possible.
    - If code changes needed, add to `tasmota/` (potentially new driver).
- **Adding New Driver**:
    - Create `xsns_<number>_<name>.ino` (sensor) or `xdrv_<number>_<name>.ino` (driver).
    - Register in `support.ino` or relevant initialization sequence.
    - Guard with `#ifdef USE_<FEATURE>`.

# Guidelines For AI Agents
- **Context**: This is embedded C/C++. Memory and flash space are scarce resources.
- **Modifications**:
    - **Respect `#ifdef`s**: Most features are compile-time optional. Ensure new code is properly guarded.
    - **Minimize Impact**: Don't touch core files unless necessary.
    - **Follow Patterns**: Copy the structure of existing drivers/sensors when adding new ones.
- **Safety**:
    - **Mains Power**: Warn the user about mains voltage risks when discussing hardware installation.
    - **Bricking**: Warn about potential bricking risks when flashing, especially with incorrect bootloader/fuse settings.
- **Documentation**: Rely on `tasmota.github.io` for authoritative usage info, but use the code as the source of truth for implementation details.
