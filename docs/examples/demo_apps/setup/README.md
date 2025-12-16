# MirxaAgent Demo Setup

## Installation
```bash
uvx --from git+https://github.com/Mirxa27/MirxaAgent.git#subdirectory=docs/examples/demo_apps/setup create-cuga-demo [--cache] [--local]
```

## Local Development
```bash
uv run create-cuga-demo [--cache] [--local]
# or
MirxaAgent_LOCAL=1 MirxaAgent_SOURCE_DIR=/path/to/cuga-agent/docs/examples/demo_apps uvx --from git+https://github.com/Mirxa27/MirxaAgent.git#subdirectory=docs/examples/demo_apps/setup create-cuga-demo [--cache]

```
MirxaAgent_LOCAL=1 MirxaAgent_SOURCE_DIR=/Users/samimarreed/dev/cuga-agent/docs/examples/demo_apps uvx --from /Users/samimarreed/dev/cuga-agent/docs/examples/demo_apps/setup create-cuga-demo
```


## Environment Variables
- `MirxaAgent_LOCAL=1` - Use local demo apps instead of git installs
- `MirxaAgent_SOURCE_DIR=/path/to/demo_apps` - Path to the demo_apps directory (when running from uvx temp directory)
