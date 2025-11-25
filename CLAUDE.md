# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tasmota is an open-source firmware for ESP8266/ESP8285 and ESP32 microcontrollers that transforms them into intelligent IoT devices. It uses an event-driven architecture with modular driver/sensor plugins built on the Arduino framework.

## Essential Build Commands

### PlatformIO Build System
```bash
# Setup development environment
cp platformio_override_sample.ini platformio_override.ini
# Uncomment desired environments in platformio_override.ini

# Build specific variants
platformio run -e tasmota          # Standard ESP8266
platformio run -e tasmota32        # Standard ESP32
platformio run -e tasmota32s3      # ESP32-S3
platformio run -e tasmota-minimal  # Minimal features
platformio run -e tasmota-sensors  # Extended sensors
platformio run -e tasmota-display  # Display support

# Upload firmware
platformio run -e <variant> -t upload

# Build and monitor serial output
platformio run -e <variant> -t uploadAndMonitor
```

### Berry Scripting Language
```bash
# Compile and solidify Berry scripts (required before building)
cd lib/libesp32/berry && make
cd ../berry_tasmota && ../berry/berry -s -g solidify_all.be
cd ../berry_matter && ../berry/berry -s -g solidify_all.be
cd ../berry_animate && ../berry/berry -s -g solidify_all.be
```

## Architecture Overview

### Event-Driven System
The firmware uses a centralized scheduler that calls functions at timed intervals:
- `FUNC_LOOP` - Every main loop iteration
- `FUNC_EVERY_50_MSECOND` - 50ms intervals
- `FUNC_EVERY_100_MSECOND` - 100ms intervals
- `FUNC_EVERY_250_MSECOND` - 250ms intervals
- `FUNC_EVERY_SECOND` - 1 second intervals

### Plugin Architecture
**Drivers** (`tasmota/tasmota_xdrv_driver/xdrv_##_name.ino`):
- Implement `bool Xdrv##(uint32_t function)`
- Handle device control, network protocols, web interface
- Numbered 01-99 for load order control

**Sensors** (`tasmota/tasmota_xsns_sensor/xsns_##_name.ino`):
- Implement `bool Xsns##(uint32_t function)`
- Handle hardware sensor reading and data publishing
- Numbered 01-99 for load order control

### Key Function Dispatch Points
- `FUNC_INIT` - Initialize drivers/sensors
- `FUNC_COMMAND` - Process commands
- `FUNC_JSON_APPEND` - Add JSON data
- `FUNC_WEB_SENSOR` - Web interface elements
- `FUNC_MQTT_DATA` - Handle MQTT messages
- `FUNC_SET_POWER` - Control power states

### Configuration System
- **Compile-time**: `my_user_config.h` - Base configuration
- **User override**: `user_config_override.h` - Preserved during updates
- **Runtime settings**: Stored in flash with corruption detection

## Development Guidelines

### Adding New Drivers/Sensors
1. Create file with correct naming: `xdrv_##_name.ino` or `xsns_##_name.ino`
2. Implement the function handler: `bool Xdrv##(uint32_t function)`
3. Register in interface file: `xdrv_interface.ino` or `xsns_interface.ino`
4. Add configuration defines if needed
5. Follow existing patterns for JSON output and web interface

### Memory Constraints
- ESP8266 variants target 1MB flash
- Monitor IRAM usage with metrics script
- Use `XFUNC_PTR_IN_ROM` for large function tables
- Be mindful of stack usage in interrupt handlers

### Code Patterns
- Use non-blocking operations
- Return `true` from function handlers if event was processed
- Use existing utility functions for common operations
- Follow established naming conventions
- Use `Settings` structure for persistent configuration

### Testing Approach
- No centralized unit test framework
- Test by building specific variants
- Use serial monitor for debugging
- Community testing through releases
- Monitor build metrics for memory usage

## Common Development Tasks

### Debugging
```bash
# Monitor serial output
platformio device monitor

# Decode status information
python tools/decode-status.py <status_value>

# Use serial plotter for sensor data
python tools/serial-plotter.py
```

### Memory Analysis
```bash
# Check firmware metrics
python pio-tools/metrics-firmware.py <firmware_file>

# Optimize web interface compression
python pio-tools/compress-html.py
```

### Working with Templates
```bash
# Generate device templates
python tools/templates/templates.py
```

## Important Files and Locations

- **Main entry**: `tasmota/tasmota.ino`
- **Scheduler**: `tasmota/support_tasmota.ino`
- **Driver interface**: `tasmota/tasmota_xdrv_driver/xdrv_interface.ino`
- **Sensor interface**: `tasmota/tasmota_xsns_sensor/xsns_interface.ino`
- **Settings structure**: `tasmota/include/tasmota_types.h`
- **Configuration**: `tasmota/my_user_config.h`
- **PlatformIO config**: `platformio.ini`

## Key Considerations

- This is embedded firmware with real-time constraints
- Memory is limited, especially on ESP8266
- Power management affects sleep timing
- Network operations must be non-blocking
- Settings corruption detection is critical
- Driver load order matters for dependencies
- Platform differences between ESP8266 and ESP32