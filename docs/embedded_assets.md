# 📦 Embedded Assets for MirxaAgent

This document explains how to use embedded assets to create a standalone MirxaAgent server without requiring external frontend files.

## 🎯 Overview

MirxaAgent can embed both the frontend (`dist` folder) and Chrome extension (`chrome-mv3` folder) as compressed base64 data directly in the Python code. This eliminates the need to distribute hundreds of separate files and creates a more portable application.

## 🚀 Quick Start

### Build with Embedded Assets

```bash
# Build and embed assets in one command
uv run scripts/build_embedded.py
```

This will:
1. 📦 Build the frontend (`frontend-workspaces/frontend/dist`)
2. 🔧 Build the extension (`frontend-workspaces/extension/releases/chrome-mv3`)
3. 📄 Embed both as base64 data in `cuga/backend/server/embedded_assets.py`

### Run with Embedded Assets

```bash
# Enable embedded assets (disabled by default)
export USE_EMBEDDED_ASSETS=true
uv run cuga/backend/server/main.py

# Or run with embedded assets enabled in one command
USE_EMBEDDED_ASSETS=true uv run cuga/backend/server/main.py

# Run with file system assets (default behavior)
uv run cuga/backend/server/main.py
```

## 📁 File Structure

```
cuga/
├── backend/server/
│   ├── main.py              # Server with embedded assets support
│   └── embedded_assets.py   # Generated file with embedded data
scripts/
├── embed_assets.py          # Script to embed assets
└── build_embedded.py        # Complete build + embed workflow
```

## 🔧 How It Works

### 1. Feature Flag Control
- Embedded assets are **disabled by default**
- Controlled by `USE_EMBEDDED_ASSETS` environment variable
- Accepts: `true`, `1`, `yes`, `on` (case-insensitive)

### 2. Asset Compression
- Frontend and extension folders are compressed into ZIP archives
- ZIP data is encoded as base64 strings
- Base64 strings are embedded in Python code

### 3. Runtime Extraction
- When enabled, assets are extracted to temporary directory
- Server uses extracted assets instead of file system paths
- Temporary files are cleaned up on shutdown

### 4. Fallback Support
- If embedded assets fail, falls back to file system paths
- Graceful degradation ensures compatibility

## 📊 Size Comparison

| Component | Original Size | Compressed Size | Compression Ratio |
|-----------|---------------|-----------------|-------------------|
| Frontend  | ~44 MB        | ~9.4 MB         | 78% reduction     |
| Extension | ~12 MB        | ~7.7 MB         | 36% reduction     |
| **Total** | **~56 MB**    | **~17 MB**      | **70% reduction** |

## 🛠️ Manual Usage

### Embed Assets Only
```bash
# Just embed existing built assets
uv run scripts/embed_assets.py
```

### Build Assets Only
```bash
# Build frontend
cd frontend-workspaces
pnpm --filter "@carbon/ai-chat-examples-web-components-basic" run build

# Build extension
pnpm --filter extension run release
```

## 🔍 Advanced Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `USE_EMBEDDED_ASSETS` | `false` | Enable embedded assets. Accepts: `true`, `1`, `yes`, `on` |

```bash
# Examples of enabling embedded assets
export USE_EMBEDDED_ASSETS=true
export USE_EMBEDDED_ASSETS=1
export USE_EMBEDDED_ASSETS=yes
export USE_EMBEDDED_ASSETS=on

# Examples of disabling (default)
export USE_EMBEDDED_ASSETS=false
export USE_EMBEDDED_ASSETS=0
export USE_EMBEDDED_ASSETS=no
export USE_EMBEDDED_ASSETS=off
# Or simply don't set the variable
```

### Custom Asset Paths
The `embedded_assets.py` file can be customized to change extraction behavior:

```python
# Extract to specific directory
embedded_assets.temp_dir = Path("/custom/path")
frontend_path, extension_path = embedded_assets.extract_assets()
```

### Compression Settings
Modify `scripts/embed_assets.py` to change compression:

```python
# Change compression level
with zipfile.ZipFile(zip_buffer, 'w', zipfile.ZIP_DEFLATED, compresslevel=9) as zip_file:
```

## 🚀 Distribution Options

### Option 1: Python Package with Embedded Assets
- Single Python package
- Assets embedded as base64 data
- ~23MB total size
- No external files needed

### Option 2: Standalone Executable (Future)
```bash
# Build standalone executable (requires PyInstaller)
uv run scripts/build_standalone.py
```

## 🧹 Cleanup

Embedded assets are automatically cleaned up when the server shuts down. For manual cleanup:

```python
from cuga.backend.server.embedded_assets import embedded_assets
embedded_assets.cleanup()
```

## 🐛 Troubleshooting

### Assets Not Found
```
⚠️ Embedded assets not found, falling back to file system
```
**Solution**: Run `uv run scripts/build_embedded.py` to generate embedded assets.

### Extraction Failed
```
❌ Failed to extract embedded assets: [error]
```
**Solution**: Check disk space and permissions in temp directory.

### Large Memory Usage
The embedded base64 data is loaded into memory. For production deployments with memory constraints, consider using file system assets instead.

## 📈 Benefits

✅ **Simplified Distribution**: Single file deployment
✅ **Reduced File Count**: From hundreds of files to one
✅ **Better Compression**: 70% size reduction
✅ **Faster Startup**: No file system scanning
✅ **Portable**: Works without external dependencies

## ⚠️ Considerations

⚠️ **Memory Usage**: Assets loaded into memory
⚠️ **Build Time**: Longer initial build process
⚠️ **Development**: Requires rebuild after frontend changes

## 🔄 Development Workflow

For development, you can switch between embedded and file system assets:

```python
# Force file system assets (for development)
USE_EMBEDDED_ASSETS = False
```

For production builds:
```bash
# Always build fresh embedded assets
uv run scripts/build_embedded.py
```
