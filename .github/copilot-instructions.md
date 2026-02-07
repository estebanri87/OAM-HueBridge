# OpenKNX Hue Gateway - AI Coding Assistant Instructions

## Project Overview
This is the **OpenKNX Hue Gateway** firmware for ESP32 devices that integrates Philips Hue lights into KNX home automation systems with bidirectional synchronization. Built on PlatformIO with the OpenKNX framework, it uses a modular architecture where functionality is split into reusable library modules.

**Data Flow**: `Hue Bridge API ↔ ESP32/HueModule ↔ KNX Bus`

## Critical Architecture Patterns

### Module System
The firmware uses **OpenKNX modules** registered in [src/main.cpp](src/main.cpp) via `openknx.addModule(ID, moduleInstance)`:
- Module ID determines initialization/execution order (lower IDs initialize first)
- Active modules: Logic (1), Network (2), WLAN (3), FileTransfer (6), FunctionBlocks (8), HueModule (9)
- Each module lives in `lib/OFM-*` or `lib/OGM-*` directories with their own READMEs
- Modules communicate through KNX Communication Objects (KOs)
- **Global instance pattern**: Each module defines a global instance in `.cpp` (e.g., `HueModule hueModule;` as `openknxHueModule`) externed in `.h`
- **Lifecycle hooks**: `setup()` once at boot, `loop()` continuously, `processInputKo()` for KNX object handling
- **Module naming**: `openknx` prefix + module name (e.g., `openknxLogic`, `openknxNetwork`, `openknxHueModule`)

### XML-Driven Configuration (OpenKNXproducer)
**Key concept**: ETS configuration is defined in XML, NOT manually coded:
- [src/HueGateway.xml](src/HueGateway.xml) (or HueGateway-Dev.xml/HueGateway-Release.xml variants) defines the KNX product structure
- [src/HueGateway.conf.xml](src/HueGateway.conf.xml) contains build-specific configuration values referenced by `<op:config>` tag
- Uses `<op:define>` to include modules with `.share.xml` and `.templ.xml` from lib directories
- Example: `<op:define prefix="HUE" ModuleType="14" share="../lib/OFM-HueModule/src/HueModule.share.xml" NumChannels="20" KoOffset="1010">`
- `OpenKNXproducer` generates [include/knxprod.h](include/knxprod.h) with parameter/KO defines
- Run via task: "OpenKNXproducer" (calls `~/bin/OpenKNXproducer.exe create --Debug HueGateway` from src directory)
- Generated defines follow pattern: `HUE_ParamBridgeIP`, `KoHUE_ChannelSwitch`, etc.
- `<op:verify>` blocks enforce module version compatibility (e.g., `ModuleVersion="%HUE_VerifyVersion%"`)

**Never hardcode KNX parameters** - they come from generated `knxprod.h`. Module prefix (e.g., `HUE`, `LOG`, `FCB`) maps to parameter names.

### Hardware Abstraction
- Hardware pins configured in [include/hardware.h](include/hardware.h) (includes `HardwareConfig.h`)
- Multiple hardware targets defined in `platformio.*.ini` configs (REG1, Adafruit Feather ESP32 V2, etc.)
- Build environments reference specific hardware profiles

## Essential Workflows

### Build & Release
**Development build:**
```powershell
pio run -e develop_ESP32_USB
```

**Beta/Release builds:** Use PowerShell tasks (NOT direct PlatformIO):
- Task: "Build-Beta" → runs [scripts/Build-Release.ps1](scripts/Build-Release.ps1)
- Task: "Build-Release" → runs `scripts/Build-Release.ps1 Release`
- These scripts call `lib/OGM-Common/scripts/setup/reusable/Build-Release-Preprocess.ps1` then build multiple firmware variants
- Project name defined in [scripts/OpenKNX-Build-Settings.ps1](scripts/OpenKNX-Build-Settings.ps1) (should be "HueGateway", not "SmartHomeBridge")
- Builds produce `.bin` files for ESP32 (or `.uf2` for RP2040)

**KNXprod generation:** 
1. Run task "OpenKNXproducer" to generate `src/HueGateway.h`
2. Manually copy to include: `Copy-Item src\HueGateway.h -Destination include\knxprod.h -Force`
3. Verify correct product name in knxprod.h (should be "Hue Gateway", not "Smart Home Bridge")

### Dependencies
- PlatformIO libraries managed via `platformio.ini` `extra_configs` that include base configurations from `lib/OGM-Common/platformio.*.ini`
- Git submodules for OpenKNX libraries (lib/OFM-*, lib/OGM-*)
- Custom dependencies in [platformio.custom.ini](platformio.custom.ini): ESPAsyncWebServer, AsyncTCP, HomeSpan
- Restore dependencies: `restore/Restore-Dependencies.ps1` (or `-Branch.ps1` for dev branches)
  - **Requires**: Windows Admin privileges OR Developer Mode enabled
  - Creates symlinks for lib directories to shared workspace modules

### Testing & Debugging
- Console access via USB serial (OpenKNX console with commands like `hk` for HomeKit config)
- Dual-core support on RP2040 with `OPENKNX_DUALCORE` define
- Watchdog configurable via `OPENKNX_WATCHDOG` (disable for debugging)
- LED/button pins defined in hardware.h (e.g., `PROG_LED_PIN`, `PROG_BUTTON_PIN`)

## Project-Specific Conventions

### Naming Patternscustom.ini
- KO (Communication Object) offsets defined per module in XML to prevent conflicts:
  - Logic: `KoOffset="100"` (50 channels)
  - FunctionBlocks: `KoOffset="250"` (15 channels)
  - HueModule: `KoOffset="1010"` (20 channels), `KoSingleOffset="3"` for shared objects
- ModuleType IDs: BASE=10, NET=12, UCT=13, HUE=14, LOG=10, FCB=11
- XML prefix maps to defines: `HUE` prefix → `HUE_ModuleVersion`, `KoHUE_*` in knxprod.h
- Build environments: `develop_*`, `release_*` naming in platformio.ini
- KO (Communication Object) offsets defined per module in XML to prevent conflicts (e.g., HueModule at `KoOffset="1010"`, Logic at `KoOffset="100"`)

### Multi-Language Support
- German documentation (README, Applikationsbeschreibung.md)
- ETS descriptions support ISO-8859-1/15 encoding via OpenKNXproducer
HueGateway-Dev.xml/HueGateway-Release.xml for specific builds)
- [src/HueGateway.conf.xml](src/HueGateway.conf.xml): Build configuration values (application numbers, version strings, etc.)
- [include/knxprod.h](include/knxprod.h): Generated - NEVER edit manually
- [include/hardware.h](include/hardware.h): Hardware pin definitions (includes `HardwareConfig.h` from lib/OGM-HardwareConfig)
- [platformio.ini](platformio.ini): Points to OGM-Common base configs
- [platformio.custom.ini](platformio.custom.ini): Project-specific build flags, dependencies, and hardware configuration
- [include/knxprod.h](include/knxprod.h): Generated - NEVER edit manually
- [include/hardware.h](include/hardware.h): Hardware pin definitions (includes `HardwareConfig.h`)
- [platformio.ini](platformio.ini): Points to OGM-Common base configs

## Common Pitfalls & Troubleshooting

### Build Errors
**Error: `ETH_PHY_GENERIC was not declared`**
- **Cause**: ESP32 framework doesn't define this constant used in REG1 hardware config
- **Fix**: Already added to [platformio.custom.ini](platformio.custom.ini) as `-D ETH_PHY_GENERIC=ETH_PHY_LAN8720`
- If removed accidentally, re-add to `custom_ESP32` build_flags section

**Error: `#error "OpenKNXproducer 3.12.8.0 or higher is required"`**
- **Cause**: `include/knxprod.h` is stale or wasn't updated after running OpenKNXproducer
- **Fix**: Run OpenKNXproducer task, then manually copy: `Copy-Item src\HueGateway.h -Destination include\knxprod.h -Force`
- **Root cause**: OpenKNXproducer generates `src/HueGateway.h` but may not copy it to `include/knxprod.h`

**Error: Missing defines like `ParamFCB_CHFormatStringStr`**
- **Cause**: Same as above - knxprod.h not updated from HueGateway.h
- **Verification**: Check if `src/HueGateway.h` has correct content (e.g., "Hue Gateway" product name, correct module IDs)
- Compare timestamps: `Get-Item src\HueGateway.h, include\knxprod.h | Select LastWriteTime, Name`

**Error: `FileNotFoundError: git not found` during build**
- **Cause**: Build script `lib/OGM-Common/scripts/pio/prepare.py` requires git in PATH
- **Fix 1 (Permanent)**: Add git.exe to system PATH or ensure Git for Windows is installed
- **Fix 2 (Quick workaround)**: Already patched in `prepare.py` - added `FileNotFoundError` and `OSError` to exception handling in `get_git_version()` function
- **Impact**: Without git, version strings won't include commit hashes, but build will succeed

**Warning: `"OPENKNX_LOOPTIME_WARNING" redefined`**
- **Cause**: Define appears in both custom.ini and release sections
- **Impact**: Harmless warning, last definition wins
- **Fix**: Remove duplicate from platformio.custom.ini if desired

### XML & Configuration
- **Don't edit knxprod.h** - regenerate via OpenKNXproducer task, then copy from src/HueGateway.h
- **Don't skip XML module verification** - `<op:verify>` blocks ensure library version compatibility
- **Module IDs must be unique** - collision causes initialization failure
- **KO offsets matter** - defined per module in XML to avoid conflicts
- **Wrong XML file processed**: Check task uses `HueGateway` (not `Hue`, `Gateway` or `SmartHomeBridge`) as argument

### Hardware & Runtime
- **GPIO1 special handling** - used for Serial TX on ESP32, requires explicit Serial.end() before GPIO use
- **Watchdog resets**: Disable with `-D OPENKNX_WATCHDOG=0` in build_flags for debugging

## Integration Points
  - See [lib/OFM-HueGatewayModule/.github/copilot-instructions.md](lib/OFM-HueGatewayModule/.github/copilot-instructions.md) for module-specific details
- **Network**: Ethernet (NET module) or WiFi (WLAN module) conditionally compiled based on `NET_ModuleVersion` and `WLAN_WifiSSID` defines
- **HomeSpan**: HomeKit integration (v2.1.2) - limit of 41 devices (`ESPALEXA_MAXDEVICES=41`)
- **Web Server**: ESPAsyncWebServer for HTTP endpoints (bridge discovery, configuration)
- **Hue Gateway Integration**: OFM-HueGatewayModule in `lib/OFM-HueGatewayModule/` handles Hue API v2 (not v1) communication
  - Authentication via button-press flow (`POST /api` with app-key generation)
  - Real-time updates via Server-Sent Events (SSE) on `/eventstream/clip/v2`
  - mDNS discovery for `_hue._tcp.local` service
  - Up to 20 Hue light channels configurable via ETS
- **Network**: Ethernet (NET module) or WiFi (WLAN module) conditionally compiled based on defines

## When Adding Features
1. Determine if it belongs in core firmware or a module (prefer modules)
2. Update XML definitions if adding KOs/parameters
3. Regenerate knxprod.h with OpenKNXproducer
4. Test with ETS import before firmware testing
5. Update relevant module README in lib/OFM-*/README.md
