# MirxaAgent Frontend Workspaces

Monorepo for the MirxaAgent browser extension and frontend tooling, managed with **pnpm workspaces**. 

## 📦 Workspace Packages

- **`extension`**: Browser extension (Chrome/Edge/Firefox) that serves as the entry point to MirxaAgent
- **`runtime`**: DOM/content-script runtime used by the extension and libraries  
- **`shared`** (`@uiagent/shared`): Shared TypeScript utilities and types consumed by other packages
- **`agentic_chat`**: React-based frontend for the extension's side panel
- **`frontend`**: Alternative frontend implementation

## 🔧 Prerequisites

- **Node.js 18+** 
- **pnpm 8+** (Package manager - install with `npm install -g pnpm`)
- **Chromium-based browser** (Chrome or Edge). Firefox supported via dedicated scripts.

## 🚀 Quick Start

### 1. Install Dependencies

From the workspace root (`frontend-workspaces/`):

```bash
pnpm install
```

This installs all workspace dependencies and runs post-install steps (e.g., `wxt prepare` in the extension).

### 2. Start the Backend Server

- Add `use_extension = true` to your `settings.toml` file under `advanced_features`
- Run the debug VS Code configuration or start the server through the main entrypoint

### 3. Run the Extension

Copy the environment file and start development:

```bash
cd extension
cp .env-example .env  # Edit as needed
cd ..
pnpm --filter extension run dev
```

WXT will automatically open Chrome with the extension loaded.

## 📋 pnpm Commands Reference

### Core Commands

| Task | Command |
|------|---------|
| **Install all dependencies** | `pnpm install` |
| **Build all workspaces** | `pnpm -r build` |
| **Clean all node_modules** | `pnpm -r clean` |

### Extension Commands

| Task | Command |
|------|---------|
| **Development (Chrome)** | `pnpm --filter extension run dev` |
| **Development (Firefox)** | `pnpm --filter extension run dev:firefox` |
| **Build for production** | `pnpm --filter extension run build` |
| **Build for Firefox** | `pnpm --filter extension run build:firefox` |
| **Release build (copy to releases/)** | `pnpm --filter extension run release` |
| **Release build for Firefox** | `pnpm --filter extension run release:firefox` |
| **Create distribution zip** | `pnpm --filter extension run zip` |
| **Run E2E tests** | `pnpm --filter extension run e2etest` |
| **Type checking** | `pnpm --filter extension run compile` |

### Frontend Commands

| Task | Command |
|------|---------|
| **Development server** | `pnpm --filter frontend run dev` |
| **Build for production** | `pnpm --filter frontend run build` |
| **Start production server** | `pnpm --filter frontend run start` |

### Agentic Chat Commands

| Task | Command |
|------|---------|
| **Development server** | `pnpm --filter agentic_chat run dev` |
| **Build for production** | `pnpm --filter agentic_chat run build` |
| **Run tests** | `pnpm --filter agentic_chat run test` |
| **Type checking** | `pnpm --filter agentic_chat run compile` |

### Shared Library Commands

| Task | Command |
|------|---------|
| **Build shared library** | `pnpm --filter @uiagent/shared run build` |
| **Watch mode** | `pnpm --filter @uiagent/shared run watch` |
| **Clean build** | `pnpm --filter @uiagent/shared run clean` |

### Runtime Commands

| Task | Command |
|------|---------|
| **Run tests** | `pnpm --filter runtime run test` |

## 🏗️ Build Process

### Extension Build

1. **Prepare environment**:
   ```bash
   cd extension
   cp .env-example .env  # Edit values as needed
   ```

2. **Build the extension**:
   ```bash
   pnpm --filter extension run build
   ```

3. **Load in browser**:
   - Open Chrome/Edge → Manage Extensions
   - Enable Developer mode  
   - Load Unpacked → select `extension/.output/chrome-mv3/`

### Release Build

Build and copy to releases folder for distribution:

```bash
# Chrome/Edge release
pnpm --filter extension run release

# Firefox release  
pnpm --filter extension run release:firefox
```

This builds the extension, cleans the releases folder, and copies the fresh output to:
- `extension/releases/chrome-mv3/` (Chrome/Edge)
- `extension/releases/firefox-mv2/` (Firefox)

### Distribution Build

Create a distributable zip file:

```bash
pnpm --filter extension run zip
```

Output will be in `extension/.output/`

## 🔍 Development Tips

### Working with Multiple Packages

```bash
# Run command in specific workspace
pnpm --filter extension run [command]

# Run command in all workspaces
pnpm -r run [command]

# Run command in workspaces matching pattern
pnpm --filter "*chat*" run build
```

### Dependency Management

```bash
# Add dependency to specific workspace
pnpm --filter extension add [package]

# Add dev dependency
pnpm --filter extension add -D [package]

# Add workspace dependency
pnpm --filter extension add agentic_chat@workspace:*
```

### Debugging

```bash
# Watch mode for shared library
pnpm --filter @uiagent/shared run watch

# Development with hot reload
pnpm --filter extension run dev -- --watch --mode development
```

## 🌐 Browser Support

### Chrome/Edge
- Full support with Manifest V3
- Hot reload during development
- Production builds ready for store submission

### Firefox  
- Beta support with Manifest V2
- Requires manual `:has` pseudo-selector enablement
- Use `*:firefox` script variants

## 🐛 Troubleshooting

### Common Issues

**Extension not appearing after build:**
- Check terminal output for build directory
- Verify you're loading the correct `.output/` folder

**Type errors in shared library:**
```bash
pnpm --filter @uiagent/shared run build
# Then restart dev servers
```

**Dependency resolution issues:**
```bash
# Clean and reinstall
rm -rf node_modules pnpm-lock.yaml
find . -name "node_modules" -type d -exec rm -rf {} +
pnpm install
```

**Build failures:**
- Ensure all workspace dependencies use `workspace:*` protocol
- Check that shared library is built before extension
- Verify environment files are properly configured

### Firefox Specific

- Enable `:has` pseudo-selector: `about:config` → `layout.css.has-selector.enabled` → `true`
- Full support coming in Firefox 121+

## 📁 Project Structure

```
frontend-workspaces/
├── pnpm-workspace.yaml          # pnpm workspace configuration
├── package.json                 # Root package.json
├── extension/                   # Browser extension
│   ├── src/
│   ├── wxt.config.ts           # WXT configuration
│   └── .output/                # Build output
├── agentic_chat/               # React frontend
│   ├── src/
│   └── dist/
├── frontend/                   # Alternative frontend
├── shared/                     # Shared utilities
│   ├── src/
│   └── dist/
└── runtime/                    # Runtime utilities
```

## 🔄 Workflow Examples

### Full Development Setup
```bash
# 1. Install everything
pnpm install

# 2. Build shared library first
pnpm --filter @uiagent/shared run build

# 3. Start extension development
pnpm --filter extension run dev
```

### Production Build
```bash
# Build all packages
pnpm -r build

# Or build specific packages in order
pnpm --filter @uiagent/shared run build
pnpm --filter agentic_chat run build  
pnpm --filter extension run build
```

### Testing
```bash
# Run all tests
pnpm -r test

# Test specific package
pnpm --filter extension run e2etest
```