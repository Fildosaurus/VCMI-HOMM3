# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

VCMI is an open-source recreation of Heroes of Might & Magic III engine written in C++17. The project uses CMake build system with Conan for dependency management and supports cross-platform development (Windows, Linux, macOS, Android, iOS).

## Build Commands

### Windows Development (Recommended)

```bash
# Prerequisites: Install Visual Studio 2022, CMake, Conan 2.x

# Install dependencies using Conan
cd source
conan install . --output-folder=conan-msvc --build=missing --profile=msvc-x64

# Set up environment and generate solution
source\conan-msvc\conanrun.bat
cmake -S source -B build --toolchain source\conan-msvc\conan_toolchain.cmake

# Build
cmake --build build --config RelWithDebInfo
```

### CMake Presets (Alternative)

```bash
# For Windows MinGW
cmake --preset windows-mingw-release
cmake --build --preset windows-mingw-release

# For Linux
cmake --preset linux-release
cmake --build --preset linux-release
```

### Test Commands

```bash
# Run all tests
cd build
ctest --config RelWithDebInfo

# Run specific test
ctest -R BattleHexTest --config RelWithDebInfo

# Run tests in Debug mode
cmake --build build --config Debug
ctest --config Debug
```

### Lint and Code Quality

```bash
# Format code (requires clang-format)
find . -name "*.cpp" -o -name "*.h" | xargs clang-format -i

# Static analysis (built-in with CMake)
cmake -DENABLE_STRICT_COMPILATION=ON
```

## Architecture Overview

### Core Components

**Three-tier architecture:**
- **VCMI_lib** (shared library): Core game logic, serialization, file handling, bonus system
- **VCMI_client** (executable): UI, rendering, input handling, player interface
- **VCMI_server** (executable): Game state management, multiplayer coordination, AI hosting

**Key architectural principles:**
- Server maintains authoritative game state
- Clients render state and send player actions to server
- All game mechanics changes flow through server
- Bonus system handles complex game effect interactions

### Directory Structure

```
vcmi/
├── AI/                    # Artificial Intelligence modules
│   ├── BattleAI/         # Combat AI
│   ├── Nullkiller/       # Adventure map AI (current)
│   └── VCAI/             # Legacy adventure AI
├── client/               # Game client code
│   ├── adventureMap/     # Adventure map UI
│   ├── battle/           # Combat interface
│   ├── gui/              # UI framework
│   └── mainmenu/         # Main menu systems
├── lib/                  # Shared game library
│   ├── battle/           # Combat mechanics
│   ├── bonuses/          # Bonus/effect system
│   ├── filesystem/       # File I/O abstraction
│   ├── gameState/        # Core game state
│   └── serializer/       # Network/save serialization
├── server/               # Game server
├── launcher/             # Game launcher with mod support
├── mapeditor/           # Map editor
├── test/                # Unit tests
├── Mods/                # Mod content
└── config/              # Game configuration files
```

### Important Systems

**Bonus System**: Central to game mechanics, handles all temporary and permanent effects (spells, artifacts, skills). Located in `lib/bonuses/`.

**Serialization**: Network communication and save games use boost-based serialization framework in `lib/serializer/`.

**Threading Model**:
- Main thread: UI and rendering
- Network thread: Packet processing and animations
- Server thread: Game logic processing
- AI threads: TBB task-based parallelization

**Configuration System**: JSON-based configs in `config/` directory define game entities (creatures, spells, artifacts, etc.).

## Modding Support

VCMI has extensive modding capabilities:
- **Mod structure**: `Mods/modname/mod.json` + `Content/` directory
- **Entity definitions**: JSON files for creatures, spells, artifacts, heroes
- **Map objects**: Configurable via JSON
- **Scripting**: Lua scripting support (when ENABLE_LUA=ON)

## Development Guidelines

### Code Style
- **Indentation**: Tabs only
- **Braces**: Opening brace on new line for blocks
- **Standard**: C++17 features are acceptable
- **Formatting**: Use `.clang-format` configuration

### Key Conventions
- Use `VCMI_LIB_NAMESPACE_BEGIN/END` macros when referencing lib from other components
- Prefer RelWithDebInfo over Debug builds for development (Debug is extremely slow)
- All game state changes must go through server
- Use boost serialization for network packets

### Testing
- Unit tests in `test/` directory use Google Test framework
- Tests are organized by component (battle, gameState, etc.)
- Run tests before submitting changes

### Performance Considerations
- Use Intel TBB for parallel processing (enabled by default)
- Combat AI and RMG heavily utilize thread pools
- Enable PCH (precompiled headers) for faster builds: `ENABLE_PCH=ON`
- Consider CCache for development: `ENABLE_CCACHE=ON`

## Platform-Specific Notes

### Windows
- Requires Windows 10+ for automated systems
- Use Visual Studio 2022 (MSVC) or MSYS2 (MinGW)
- Launch executables via `.bat` files in build directory

### Mobile Platforms
- iOS/Android use monolithic builds (`ENABLE_SINGLE_APP_BUILD=ON`)
- Different namespace wrapping required for single-process builds
- Server runs as static library, not separate executable

## Common CMake Options

```cmake
-DENABLE_CLIENT=ON          # Build game client (default)
-DENABLE_SERVER=ON          # Build dedicated server (default)
-DENABLE_LAUNCHER=ON        # Build launcher (default)
-DENABLE_EDITOR=ON          # Build map editor (default)
-DENABLE_TEST=OFF           # Build unit tests
-DENABLE_LUA=OFF            # Enable Lua scripting
-DENABLE_STRICT_COMPILATION=OFF  # Treat warnings as errors
-DENABLE_CCACHE=OFF         # Use CCache for faster rebuilds
-DENABLE_PCH=ON             # Use precompiled headers
```