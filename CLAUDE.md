# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QNapi is a Qt5-based application for automatically downloading subtitles for movie files. It supports multiple subtitle engines (NapiProjekt, OpenSubtitles, Napisy24) and includes both GUI and CLI interfaces.

## Build System & Development Commands

### Building from Source

```bash
# Clone with submodules (required for dependencies)
git clone --recursive https://github.com/QNapi/qnapi.git

# Build all components (default)
qmake
make

# Build specific components only
qmake CONFIG+=no_cli    # Skip CLI application
qmake CONFIG+=no_gui    # Skip GUI application

# Platform-specific tasks
make appdmg            # Build OSX .dmg installer (OSX only)
make install            # Windows: copy binaries to win32/out directory
make doxygen           # Generate documentation
```

### Output Locations

- **Linux**: `qnapi` executable in root directory
- **OSX**: `macx/QNapi.app` bundle
- **Windows**: `win32/out/qnapi.exe` (after `make install`)

### Code Style

Format using Google style with attached braces (configured in `.clang-format`):

```bash
clang-format -i [files]
```

## Architecture

### Project Structure

```
qnapi/                  # Root project file (qnapi.pro)
├── libqnapi/           # Core static library - subtitle download logic
├── gui/               # Qt5 GUI application
├── cli/               # Command-line interface application
├── deps/              # Third-party dependencies (libmediainfo, libmaia)
├── flathub/           # Flatpak manifest files for distribution
└── translations/      # Internationalization files
```

### Core Components

#### libqnapi/ (Static Library)

The core functionality is organized into several key modules:

- **Config System** (`src/config/`):
  - `QNapiConfig`: Main configuration management
  - `EngineConfig`, `GeneralConfig`, `PostProcessingConfig`: Specific config sections
  - `StaticConfig`: Build-time constants

- **Download Engines** (`src/engines/`):
  - `SubtitleDownloadEngine`: Abstract base class
  - `NapiProjektDownloadEngine`, `OpenSubtitlesDownloadEngine`, `Napisy24DownloadEngine`: Concrete implementations
  - `SubtitleDownloadEnginesRegistry`: Factory and registry pattern

- **Movie Information** (`src/movieinfo/`):
  - `MovieInfoProvider`: Abstract interface
  - `LibMediaInfoMovieInfoProvider`: MediaInfo library integration

- **Subtitle Processing** (`src/subconvert/`):
  - `SubtitleConverter`: Format conversion
  - `SubtitleFormat`: Base class for subtitle formats
  - Format implementations: MicroDVD, MPL2, SubRip, TMPlayer

- **Utilities** (`src/utils/`):
  - `P7ZipDecoder`: **CRITICAL** - 7z archive extraction required for subtitle download engines
  - `SyncHttp`, `SyncXmlRpc`: Network communication
  - `EncodingUtils`: Text encoding handling
  - `Console`: CLI output formatting

- **CLI Parser** (`src/parser/`):
  - Command-line argument parsing using visitor pattern
  - Individual parsers for each CLI option

#### GUI Application (`gui/`)

- **Main Application**: `QNapiApp` (singleton), `GuiMain`
- **Forms**: Qt UI forms for main dialog, configuration, progress, etc.
- **QCumber**: Custom UI components (`QSingleApplication`, `QManagedRequest`, inter-process communication)

#### CLI Application (`cli/`)

- **Main Entry**: `CliMain`
- **Downloader**: `CliSubtitlesDownloader` - orchestrates the download process

### Key Design Patterns

- **Registry Pattern**: Used for both download engines and subtitle formats
- **Factory Pattern**: Engine and format instantiation
- **Visitor Pattern**: CLI argument parsing
- **Singleton Pattern**: Main application classes

## Dependencies

### Runtime Dependencies

- **Qt 5.2+**: Core framework (network, xml, gui widgets)
- **p7zip (7za)**: **CRITICAL** - Archive extraction for subtitle files, especially password-protected 7z archives from NapiProjekt engine
- **libmediainfo**: Movie file metadata extraction

### Build Dependencies

- **qmake**: Qt build system
- **C++11 compiler**: g++, clang++, or MinGW

### Included Submodules

- **libmaia**: Utility library
- **qt-maybe**: Qt compatibility helpers

## Development Notes

### Adding New Subtitle Engines

1. Inherit from `SubtitleDownloadEngine` in `libqnapi/src/engines/`
2. Implement required virtual methods
3. Register in `SubtitleDownloadEnginesRegistry`
4. Add configuration UI in `gui/src/forms/`

### Adding New Subtitle Formats

1. Inherit from `SubtitleFormat` in `libqnapi/src/subconvert/formats/`
2. Implement parsing/conversion methods
3. Register in `SubtitleFormatsRegistry`

### Testing

No automated test suite is currently present. Manual testing involves:

- Testing each subtitle engine with various file formats
- Verifying subtitle conversion between formats
- CLI argument parsing validation
- **CRITICAL**: Always test p7zip/7za integration when making Flatpak changes - QNapi cannot extract downloaded subtitles without it

### Flatpak Distribution (flathub/)

**IMPORTANT**: QNapi Flatpak packaging is maintained separately from the main source repository in the `flathub/` directory.

#### Key Components

- **Manifest**: `io.github.qnapi.yaml` - Flatpak build configuration
- **Metadata**: `io.github.qnapi.metainfo.xml` - App store information

#### Critical Dependencies

- **Runtime**: `org.kde.Platform 5.15-24.08 LTS` (current supported version)
- **p7zip**: **ESSENTIAL** - Must include working `7za` binary for subtitle extraction
  - Used by `P7ZipDecoder` class in `src/utils/p7zipdecoder.h`
  - Required for NapiProjekt engine (password-protected 7z archives)
  - Source compilation has C++17 compatibility issues - binary inclusion preferred

#### Build Process

```bash
# Build and install Flatpak
cd flathub/
flatpak-builder --force-clean --user --install build-dir io.github.qnapi.yaml

# Test critical functionality
flatpak run --command=sh io.github.qnapi -c "7z"  # Verify 7zip works
flatpak run io.github.qnapi --help                # Verify app launches
```

#### Runtime Updates

- KDE Platform versions age out and require migration
- Monitor for EOL warnings and update `runtime-version` in manifest
- Current LTS runtime: 5.15-24.08 (supported through 2025)
- Always test subtitle download functionality after runtime updates

#### Known Issues

- p7zip source code incompatible with modern C++17 standards
- Flatpak build isolation prevents direct system binary access
- Solution: Copy system `7za` binary into Flatpak sources during build

### Useful Context7 MCP Integration

When working with Qt-specific questions, use the context7 MCP server to fetch up-to-date Qt documentation. For general web searches about subtitle formats or API integration, use the perplexity MCP server.
