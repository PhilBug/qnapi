# QNapi Development Setup & Testing Guide

This guide provides comprehensive instructions for building and testing QNapi on Fedora Linux and other distributions.

## Option 1: Traditional Build (Recommended for Development)

### Install Dependencies

#### Fedora/RHEL/CentOS

```bash
sudo dnf install qt5-qtbase-devel qt5-qttools-devel qt5-linguist
sudo dnf install gcc-c++ make cmake
sudo dnf install p7zip p7zip-plugins  # Critical for subtitle extraction
sudo dnf install mediainfo libmediainfo-devel
```

#### Ubuntu/Debian

```bash
sudo apt update
sudo apt install qt5-default qttools5-dev-tools qtbase5-dev
sudo apt install build-essential cmake
sudo apt install p7zip-full p7zip-rar  # Critical for subtitle extraction
sudo apt install mediainfo libmediainfo-dev
```

#### Arch Linux

```bash
sudo pacman -S qt5-base qt5-tools cmake make gcc
sudo pacman -S p7zip  # Critical for subtitle extraction
sudo pacman -S mediainfo
```

### Build from Source

```bash
# Clone the repository with submodules
git clone --recursive https://github.com/QNapi/qnapi.git
cd qnapi

# Build both GUI and CLI versions
qmake
make

# Or build specific components only
qmake CONFIG+=no_cli    # Skip CLI application
make

qmake CONFIG+=no_gui    # Skip GUI application
make
```

### Run and Test

```bash
# Test the CLI version
./qnapi --help

# Test the GUI version
./qnapi

# Test subtitle download (requires movie file)
./qnapi -c -l eng /path/to/your/movie.mkv
```

## Option 2: Flatpak Build (Using Updated Manifest)

### Install Flatpak Dependencies

```bash
# Install Flatpak
sudo dnf install flatpak flatpak-builder  # Fedora
# sudo apt install flatpak flatpak-builder  # Ubuntu
# sudo pacman -S flatpak flatpak-builder  # Arch

# Add Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Install KDE Platform runtime (current LTS version)
flatpak install flathub org.kde.Sdk//5.15-24.08
flatpak install flathub org.kde.Platform//5.15-24.08
```

### Build Your Local Flatpak

```bash
cd /path/to/your/qnapi/repository/flathub/

# Make sure 7za is available on your system
which 7za || sudo dnf install p7zip  # Adjust for your distro

# Copy 7za to flathub directory (required for Flatpak build)
cp /usr/bin/7za .

# Build and install your Flatpak
flatpak-builder --force-clean --user --install build-dir io.github.qnapi.yaml
```

### Test Your Flatpak

```bash
# Test 7zip integration (CRITICAL!)
flatpak run --command=sh io.github.qnapi -c "7z"

# Test the application
flatpak run io.github.qnapi --help

# Test subtitle download
flatpak run io.github.qnapi -c -l eng /path/to/your/movie.mkv
```

## Option 3: Install Existing Flatpak from Flathub

```bash
# Install the current version (note: may still have EOL runtime warnings)
flatpak install flathub io.github.qnapi

# Run it
flatpak run io.github.qnapi
```

## Testing Checklist

### Basic Functionality Tests

1. **Application Launch**

   ```bash
   # Traditional build
   ./qnapi --version

   # Flatpak
   flatpak run io.github.qnapi --version
   ```

2. **Help Text Display**

   ```bash
   ./qnapi --help  # Should show complete help text
   ```

3. **CLI Arguments Parsing**

   ```bash
   ./qnapi --hl  # Should show available languages
   ./qnapi --show-list  # Should list subtitle engines
   ```

4. **GUI Interface**

   ```bash
   ./qnapi  # Should open the main window
   ```

### Critical 7zip Testing

**IMPORTANT**: QNapi cannot extract downloaded subtitles without working 7zip!

```bash
# Test if p7zip works
7z  # Should show version and usage information

# Test archive creation/extraction
echo "test content" > test.txt
7z a test.7z test.txt
7z x test.7z  # Should extract without errors
ls -la test.txt  # Verify file was extracted
rm test.txt test.7z
```

### Subtitle Download Testing

```bash
# Find a video file to test with
VIDEO_FILE=$(find ~/Videos -name "*.mkv" -o -name "*.mp4" 2>/dev/null | head -1)

if [ -n "$VIDEO_FILE" ]; then
    echo "Testing with: $VIDEO_FILE"

    # Test CLI subtitle download
    ./qnapi -c -l eng "$VIDEO_FILE"

    # Test different languages
    ./qnapi -c -l pl "$VIDEO_FILE"  # Polish
    ./qnapi -c -l es "$VIDEO_FILE"  # Spanish

    # Check if subtitle file was created
    SUBTITLE_FILE="${VIDEO_FILE%.*}.srt"
    if [ -f "$SUBTITLE_FILE" ]; then
        echo "✅ SUCCESS: Subtitle file created: $SUBTITLE_FILE"
    else
        echo "❌ FAILED: No subtitle file created"
    fi
else
    echo "No video file found for testing. Please provide a video file path."
fi
```

### Engine-Specific Testing

```bash
# Test each subtitle engine separately
VIDEO_FILE="/path/to/test/video.mkv"

# NapiProjekt (Polish engine, uses password-protected 7z)
./qnapi -c -l pl "$VIDEO_FILE"

# OpenSubtitles (International)
./qnapi -c -l eng "$VIDEO_FILE"

# Napisy24 (Polish alternative)
./qnapi -c -l pl "$VIDEO_FILE"
```

## Troubleshooting Common Issues

### Missing p7zip/7za

**Symptoms**: Subtitle downloads fail, archive extraction errors

**Solutions**:

```bash
# Fedora/RHEL
sudo dnf install p7zip p7zip-plugins

# Ubuntu/Debian
sudo apt install p7zip-full

# Arch Linux
sudo pacman -S p7zip

# Verify installation
which 7za  # Should return /usr/bin/7za or similar
```

### Missing Qt5 Development Libraries

**Symptoms**: qmake fails, Qt-related compilation errors

**Solutions**:

```bash
# Fedora/RHEL
sudo dnf groupinstall "Development Tools"
sudo dnf install qt5-qtbase-devel qt5-qttools-devel

# Ubuntu/Debian
sudo apt install build-essential qtbase5-dev qttools5-dev-tools

# Arch Linux
sudo pacman -S base-devel qt5-base qt5-tools
```

### Git Submodule Issues

**Symptoms**: Missing source files, build errors about missing headers

**Solutions**:

```bash
cd qnapi
git submodule update --init --recursive
```

### Flatpak Build Fails

**Symptoms**: Flatpak build errors, missing 7za binary

**Solutions**:

```bash
# Check if 7za is available
which 7za

# Copy to flathub directory if missing
cp /usr/bin/7za /path/to/qnapi/flathub/

# Check KDE runtime installation
flatpak list --runtime | grep kde

# Reinstall runtime if needed
flatpak install flathub org.kde.Sdk//5.15-24.08
flatpak install flathub org.kde.Platform//5.15-24.08
```

### Permission Issues

**Symptoms**: Cannot write to directories, permission denied errors

**Solutions**:

```bash
# For traditional build
chmod +x qnapi

# For Flatpak
flatpak run --command=sh io.github.qnapi -c "ls -la /app/bin"
```

## Verification Commands

### Version Information

```bash
# QNapi version
./qnapi --version

# p7zip version
7z

# Qt version (shown in help output)
./qnapi --help | grep "Qt version"
```

### Flatpak Information

```bash
# Check Flatpak info
flatpak info io.github.qnapi

# Check runtime
flatpak info io.github.qnapi | grep Runtime

# Test inside Flatpak environment
flatpak run --command=sh io.github.qnapi -c "ls /app/bin"
```

### Dependency Verification

```bash
# Check critical libraries
ldd qnapi | grep -E "(Qt|mediainfo)"

# Check for required binaries
which qmake lrelease mediainfo 7z
```

## Development Workflow Tips

### Code Formatting

```bash
# Use the project's clang-format configuration
clang-format -i src/**/*.cpp src/**/*.h
```

### Clean Build

```bash
# Remove all build artifacts
make clean
rm -rf Makefile
qmake
make
```

### Debug Build

```bash
# Build with debug symbols
qmake CONFIG+=debug
make
```

### Test Specific Components

```bash
# Build only CLI
qmake CONFIG+=no_gui
make

# Build only GUI
qmake CONFIG+=no_cli
make
```

## Performance Testing

### Large File Handling

```bash
# Test with large video files (>2GB)
find /path/to/large/videos -name "*.mkv" -size +2G | head -1 | xargs ./qnapi -c -l eng
```

### Batch Operations

```bash
# Test with directory scanning
./qnapi -c -l eng /path/to/movies/directory/
```

### Memory Usage

```bash
# Monitor memory usage during operations
/usr/bin/time -v ./qnapi -c -l eng /path/to/video.mkv
```

## Common Development Scenarios

### Adding New Subtitle Formats

1. Create new format class in `libqnapi/src/subconvert/formats/`
2. Inherit from `SubtitleFormat`
3. Implement parsing/conversion methods
4. Register in `SubtitleFormatsRegistry`
5. Update CLI help and documentation

### Adding New Download Engines

1. Create new engine class in `libqnapi/src/engines/`
2. Inherit from `SubtitleDownloadEngine`
3. Implement required virtual methods
4. Register in `SubtitleDownloadEnginesRegistry`
5. Add configuration UI in `gui/src/forms/`
6. Test with various file formats

### Updating Dependencies

1. Update version numbers in build files
2. Test compilation with new versions
3. Verify functionality still works
4. Update documentation
5. Test with different distributions

## Support and Resources

- **Main Repository**: <https://github.com/QNapi/qnapi>
- **Documentation**: Check `doc/` directory for additional documentation
- **Issues**: Report bugs at <https://github.com/QNapi/qnapi/issues>
- **Community**: Check forums and discussions for community support

Remember to always test p7zip integration after any changes, as it's critical for QNapi's subtitle extraction functionality!
