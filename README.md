# pg_lake Docker Image Builder

> **⚠️ DISCLAIMER - Community/Developer Build**
>
> This is a **community-maintained** builder repository for creating Docker images from the [pg_lake](https://github.com/Snowflake-Labs/pg_lake) source code.
>
> **Important Notes:**
>
> - ❌ **NOT maintained** by the pg_lake project committers or Snowflake Labs
> - ❌ **NO official support** - use at your own risk
> - ❌ **NOT endorsed** by the pg_lake project team
> - ✅ **Community effort** - for development, testing, and personal use
> - ✅ Images are built from official pg_lake source but with custom build configurations
>
> **For official pg_lake support and releases**, please refer to:
>
> - Official repository: <https://github.com/Snowflake-Labs/pg_lake>
> - Official documentation: <https://github.com/Snowflake-Labs/pg_lake/blob/main/docs/README.md>
>
> ---

This repository contains GitHub Actions workflows to automatically build and publish Docker images for [pg_lake](https://github.com/snowflake-labs/pg_lake).

## Table of Contents

- [Images Published](#images-published)
- [Features](#features)
- [Supported Versions](#supported-versions)
- [Automated Builds](#automated-builds)
- [Manual Builds](#manual-builds)
- [Using Task Commands](#using-task-commands)
- [Image Naming Convention](#image-naming-convention)
- [Usage Examples](#usage-examples)
- [Monitoring](#monitoring)
- [License](#license)

## Images Published

**Community builds** published to:

- `ghcr.io/kameshsampath/pg_lake`
- `ghcr.io/kameshsampath/pgduck-server`

**Note**: These are development/testing images. For production use, please consult the [official pg_lake repository](https://github.com/Snowflake-Labs/pg_lake).

## Features

✅ **Automatic upstream monitoring** - Detects new commits and tags nightly  
✅ **Multi-version support** - PostgreSQL 16, 17, and 18  
✅ **Multi-OS support** - Both AlmaLinux and Debian base images  
✅ **Multi-architecture** - AMD64 and ARM64 platforms  
✅ **Tag-aware** - Automatically uses tag names for release builds  
✅ **Flexible triggers** - Automatic monitoring and manual on-demand builds  
✅ **Supply chain security** - Build provenance attestations included  

## Supported Versions

- **PostgreSQL**: 16, 17, 18
- **Base OS**: AlmaLinux 9, Debian (latest)
- **Architectures**: linux/amd64, linux/arm64
- **Images per build**: 12 total (3 PG versions × 2 OSes × 2 image types)

## Automated Builds

Images are automatically built in two ways:

### 1. **Upstream Monitoring** (Nightly) 🔍

The `monitor-upstream.yml` workflow:

- Monitors [Snowflake-Labs/pg_lake](https://github.com/snowflake-labs/pg_lake) for changes
- Checks for new commits on the `main` branch
- Checks for new tags/releases
- Automatically triggers builds when changes are detected
- Tracks state in `.last-build-commit` and `.last-build-tag` to avoid duplicates

**What gets built:**

- All 3 PostgreSQL versions (16, 17, 18)
- Both base OSes (AlmaLinux and Debian)
- Multi-architecture (AMD64 and ARM64)

**Frequency:** Runs nightly at 2 AM UTC

### 2. **Manual Trigger** (On-demand) 👆

Trigger builds manually via:

- GitHub Actions UI
- GitHub CLI (`gh workflow run`)
- Task commands (convenient shortcuts - see below)

**Use cases for manual builds:**

- Force rebuild for base image updates
- Build from a specific branch or commit
- Test builds with custom configurations
- Build from your own fork

## Manual Builds

### Via GitHub Actions UI

1. Go to [Actions](https://github.com/kameshsampath/pg_lake_builder/actions)
2. Select "Build and Push Docker Images"
3. Click "Run workflow"
4. Customize options:
   - **Source repository** (default: `snowflake-labs/pg_lake`)
   - **Branch/tag to build** (default: `main`)
   - **PostgreSQL versions** (default: `16,17,18`)
   - **Target platforms** (default: `linux/amd64,linux/arm64`)

### Via GitHub CLI

```bash
# Build from main (both OSes, all platforms)
gh workflow run build-images.yml \
  -f pg_lake_ref=main

# Build from a specific tag
gh workflow run build-images.yml \
  -f pg_lake_ref=v3.0.0

# Build specific PostgreSQL version only
gh workflow run build-images.yml \
  -f pg_versions=18

# Quick test build (AMD64 only)
gh workflow run build-images.yml \
  -f pg_versions=18 \
  -f platforms=linux/amd64
```

## Using Task Commands

This repository includes a `Taskfile.yml` with convenient commands for managing builds and workflows.

### Prerequisites

Install Task:

```bash
# macOS
brew install go-task

# Linux
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
```

### Build Commands

```bash
# Trigger full build (all PG versions, both OSes, multi-arch)
task build

# Build from upstream main branch
task build:main

# Build latest main with all versions, both OSes, all platforms
task build:latest

# Build latest main with single PG version (default: 18)
task build:latest-single-pg
task build:latest-single-pg PG_MAJOR=16
task build:latest-single-pg PG_MAJOR=17

# Build from a specific tag
task build:tag PG_LAKE_REF=v3.0.0

# Build from latest upstream tag (auto-detected)
task build:release

# Quick test build (PG 18, AlmaLinux only, AMD64)
task build:quick

# Build only PostgreSQL 18 (both OSes)
task build:pg18

# Build with AlmaLinux base only
task build:almalinux

# Build with Debian base only
task build:debian

# Build AMD64 only (both OSes)
task build:amd64

# Build ARM64 only (both OSes)
task build:arm64
```

### Custom Build Options

```bash
# Build from your own fork
task build PG_LAKE_REPO=kameshsampath/pg_lake

# Build specific commit
task build PG_LAKE_REF=abc1234

# Build latest main with PG 16 only (both OSes, multi-arch)
task build:latest-single-pg PG_MAJOR=16

# Build PG 17 only, ARM64
task build PG_VERSIONS=17 PLATFORMS=linux/arm64

# Build with custom options
task build \
  PG_LAKE_REPO=kameshsampath/pg_lake \
  PG_LAKE_REF=feature-branch \
  PG_VERSIONS=18 \
  PLATFORMS=linux/amd64
```

### Monitoring Commands

```bash
# Trigger upstream monitoring check
task monitor

# Watch latest workflow run
task watch

# Watch build workflow specifically
task watch:build

# View workflow status
task status

# View build workflow history
task status:build

# View monitoring workflow history
task status:monitor
```

### Log Management

```bash
# View logs of latest workflow
task logs

# View build workflow logs
task logs:build

# View monitoring workflow logs
task logs:monitor
```

### Workflow Control

```bash
# Cancel latest running workflow
task cancel

# Cancel build workflow
task cancel:build

# Rerun failed workflow
task rerun

# List all workflows
task workflows

# Enable all workflows
task workflows:enable

# Disable all workflows
task workflows:disable
```

### Image Management

```bash
# List published packages
task images:list

# List pg_lake image versions
task images:list-versions
```

### Get Help

```bash
# Show all available tasks
task --list-all

# Or
task help
```

### Quick Reference Table

| Command | PG Versions | Base OS | Platforms | Use Case |
|---------|-------------|---------|-----------|----------|
| `task build` | 16, 17, 18 | Both | Multi-arch | Full production build |
| `task build:main` | 16, 17, 18 | Both | Multi-arch | Build from upstream main |
| `task build:latest` | 16, 17, 18 | Both | Multi-arch | Build latest from main |
| `task build:latest-single-pg PG_MAJOR=18` | Custom (16/17/18) | Both | Multi-arch | Latest with single PG version |
| `task build:release` | 16, 17, 18 | Both | Multi-arch | Build latest upstream tag |
| `task build:quick` | 18 | AlmaLinux | AMD64 | Fast test build |
| `task build:pg18` | 18 | Both | Multi-arch | PG 18, both OSes |
| `task build:almalinux` | 16, 17, 18 | AlmaLinux | Multi-arch | AlmaLinux only |
| `task build:debian` | 16, 17, 18 | Debian | Multi-arch | Debian only |
| `task build:amd64` | 16, 17, 18 | Both | AMD64 | AMD64 only (faster) |
| `task build:arm64` | 16, 17, 18 | Both | ARM64 | ARM64 only |

## Image Naming Convention

Images are tagged based on what is being built from the **pg_lake source repository**.

### Building from a Branch/Commit (SHA-based tagging)

When building from a branch or specific commit:

```
ghcr.io/kameshsampath/pg_lake:<commit-sha>-pg<version>-<os>
ghcr.io/kameshsampath/pgduck-server:<commit-sha>-pg<version>-<os>
```

**Example:**

```
ghcr.io/kameshsampath/pg_lake:abc1234-pg18-almalinux
ghcr.io/kameshsampath/pg_lake:abc1234-pg18-debian
ghcr.io/kameshsampath/pgduck-server:abc1234-pg18-almalinux
ghcr.io/kameshsampath/pgduck-server:abc1234-pg18-debian
```

Where:

- `abc1234` = First 7 characters of the pg_lake source commit SHA
- `pg18` = PostgreSQL major version
- `almalinux`/`debian` = Base OS

### Building from a Tag (Tag-based tagging) 🏷️

When building from an official release tag, the tag name is used:

```
ghcr.io/kameshsampath/pg_lake:<tag>-pg<version>-<os>
ghcr.io/kameshsampath/pgduck-server:<tag>-pg<version>-<os>
```

**Example:**

```
ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux
ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-debian
ghcr.io/kameshsampath/pgduck-server:v3.0.0-pg18-almalinux
ghcr.io/kameshsampath/pgduck-server:v3.0.0-pg18-debian
```

Where:

- `v3.0.0` = The release tag name
- `pg18` = PostgreSQL major version
- `almalinux`/`debian` = Base OS

### Additional Tags

Each image also gets a shorter tag without the OS suffix:

- `ghcr.io/kameshsampath/pg_lake:abc1234-pg18` (from commit)
- `ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18` (from tag)

## Usage Examples

### Pull and Run Images

```bash
# Pull latest build from main
docker pull ghcr.io/kameshsampath/pg_lake:latest-pg18-almalinux

# Pull specific version
docker pull ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux

# Run pg_lake
docker run -d \
  --name pg_lake \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=mysecret \
  ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux

# Run with pgduck-server
docker run -d \
  --name pgduck \
  -p 5433:5432 \
  ghcr.io/kameshsampath/pgduck-server:v3.0.0-pg18-almalinux
```

### Docker Compose Example

```yaml
version: '3.8'

services:
  pg_lake:
    image: ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: mysecret
    volumes:
      - pgdata:/var/lib/postgresql/data

  pgduck:
    image: ghcr.io/kameshsampath/pgduck-server:v3.0.0-pg18-almalinux
    ports:
      - "5433:5432"
    environment:
      POSTGRES_PASSWORD: mysecret

volumes:
  pgdata:
```

## Monitoring

The repository includes automatic monitoring of the upstream pg_lake repository:

### How It Works

1. **Nightly at 2 AM UTC**, the monitoring workflow checks:
   - Latest commit on `main` branch
   - Latest release tag

2. **If changes detected**:
   - Compares with `.last-build-commit` and `.last-build-tag`
   - Triggers build workflow automatically
   - Updates tracking files with new commit/tag

3. **Build produces**:
   - 12 images total (3 PG versions × 2 OSes × 2 image types)
   - Properly tagged based on commit SHA or tag name

### Monitoring Status

Check monitoring status:

```bash
# Via Task
task status:monitor

# Via GitHub CLI
gh run list --workflow=monitor-upstream.yml --limit 5

# Watch monitoring run
task watch:monitor
```

### Customize Monitoring

Edit `.github/workflows/monitor-upstream.yml`:

```yaml
# Change frequency
schedule:
  - cron: "0 * * * *"  # Every hour (more frequent)
  - cron: "0 */12 * * *"  # Every 12 hours (less frequent)

# Change monitored branch
env:
  UPSTREAM_BRANCH: develop  # Monitor different branch
```

See [MONITORING.md](MONITORING.md) for detailed documentation.

## Repository Structure

```
pg-lake-builder/
├── .github/
│   └── workflows/
│       ├── build-images.yml      # Main build workflow
│       └── monitor-upstream.yml  # Upstream monitoring
├── .last-build-commit            # Tracks last built commit
├── .last-build-tag               # Tracks last built tag
├── Taskfile.yml                  # Task automation commands
├── MONITORING.md                 # Monitoring documentation
└── README.md                     # This file
```

## Build Schedule Summary

| Trigger Type | Frequency | Builds Both OSes | Description |
|--------------|-----------|------------------|-------------|
| **Upstream Monitoring** | Nightly (2 AM UTC) | ✅ Yes | Auto-detects commits and tags from upstream |
| **Manual (GitHub UI)** | On-demand | ✅ Yes (default) | Full control over options |
| **Manual (Task)** | On-demand | ✅ Yes (default) | Quick commands |
| **Manual (GitHub CLI)** | On-demand | ✅ Yes (default) | Command-line flexibility |

## Troubleshooting

### Builds Not Triggering Automatically

1. Check if workflows are enabled in Actions tab
2. Verify monitoring workflow ran successfully
3. Check `.last-build-commit` and `.last-build-tag` files

```bash
# Reset tracking to force rebuild
echo "none" > .last-build-commit
git add .last-build-commit
git commit -m "reset: force rebuild"
git push
```

### Build Failures

```bash
# View build logs
task logs:build

# Cancel and rerun
task cancel:build
task build
```

### Image Not Found

Images may take a few minutes to appear after build completes:

1. Check build completed successfully: `task status:build`
2. Verify image was pushed (check workflow logs)
3. Images are public - no authentication needed to pull

## Cost Considerations

- **GitHub Actions minutes**: Free for public repositories
- **GitHub Container Registry**: 500MB free, then $0.25/GB/month
- **Bandwidth**: Unlimited downloads for public images
- **Monitoring**: ~4 API calls per day (well within free tier)

## License

> **⚠️ DISCLAIMER**: This is a community/developer build repository.

- **This builder repository**: Configuration files are MIT licensed
- **pg_lake source code**: Licensed under Apache 2.0 by Snowflake Inc. (see [pg_lake LICENSE](https://github.com/Snowflake-Labs/pg_lake/blob/main/LICENSE))
- **Docker images produced**: Contain pg_lake software licensed under Apache 2.0
- **No warranty**: These community builds come with NO WARRANTY or official support

## Support

> **⚠️ IMPORTANT**: This is a community/developer build repository. **No official support is provided.**

For issues with:

- **pg_lake source code**: Open an issue in the [official pg_lake repository](https://github.com/snowflake-labs/pg_lake/issues)
- **Build workflow in this repo**: Open an issue in this repository (community support only)
- **Docker images from this builder**: These are **unofficial builds** - use at your own risk

**For production support**, please use official pg_lake releases and contact the Snowflake Labs team through their official channels.

## Related Links

- [pg_lake Official Repository](https://github.com/Snowflake-Labs/pg_lake)
- [pg_lake Documentation](https://github.com/Snowflake-Labs/pg_lake/blob/main/docs/README.md)
- [Monitoring Documentation](./MONITORING.md)
- [GitHub Packages Documentation](https://docs.github.com/en/packages)
- [Task Documentation](https://taskfile.dev/)

---

**Star this repository** if you find it useful! ⭐
