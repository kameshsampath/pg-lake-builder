# pg_lake_builder Repository Setup

> **⚠️ DISCLAIMER - Community/Developer Build**
>
> This is a **community-maintained** builder repository for creating Docker images from the [pg_lake](https://github.com/Snowflake-Labs/pg_lake) source code.
>
> **Important Notes:**
> - ❌ **NOT maintained** by the pg_lake project committers or Snowflake Labs
> - ❌ **NO official support** - use at your own risk
> - ❌ **NOT endorsed** by the pg_lake project team
> - ✅ **Community effort** - for development, testing, and personal use
> - ✅ Images are built from official pg_lake source but with custom build configurations
>
> **For official pg_lake support and releases**, please refer to:
> - Official repository: https://github.com/Snowflake-Labs/pg_lake
> - Official documentation: https://github.com/Snowflake-Labs/pg_lake/blob/main/docs/README.md
>
> ---

This document explains how to set up and use a separate `pg_lake_builder` repository to build and publish Docker images from the pg_lake source.

## Repository Structure

```
kameshsampath/pg_lake_builder/          # Builder repository (this repo)
├── .github/
│   └── workflows/
│       └── build-and-push-images.yml   # Main workflow
└── README.md                            # This file

kameshsampath/pg_lake/                   # Source repository (checked out during build)
└── docker/
    ├── Dockerfile                       # Build configuration
    └── Taskfile.yml                     # Build tasks
```

## Why a Separate Builder Repository?

1. **Separation of Concerns**: Keep build/CI configuration separate from source code
2. **Flexible Building**: Build from any branch/tag of the source repository
3. **Custom Scheduling**: Set your own build schedule independent of source commits
4. **Registry Control**: Publish to your own ghcr.io namespace

## Setup Instructions

### 1. Create the Builder Repository

```bash
# Create a new repository on GitHub
gh repo create kameshsampath/pg_lake_builder --public --description "Docker image builder for pg_lake"

# Clone it locally
git clone https://github.com/kameshsampath/pg_lake_builder.git
cd pg_lake_builder
```

### 2. Copy the Workflow File

Copy the workflow file from the pg_lake repository to your builder repository:

```bash
# Create the workflows directory
mkdir -p .github/workflows

# Copy the workflow file (adjust path as needed)
cp /path/to/pg_lake/.github/workflows/build-and-push-images.yml .github/workflows/
```

### 3. Create a README

Create a `README.md` in your builder repository:

```markdown
# pg_lake Docker Image Builder

> **⚠️ COMMUNITY BUILD - NOT OFFICIALLY SUPPORTED**
>
> These are **unofficial, community-built** Docker images for pg_lake.
> Not maintained or endorsed by Snowflake Labs or the pg_lake project team.

This repository contains GitHub Actions workflows to build and publish Docker images for [pg_lake](https://github.com/snowflake-labs/pg_lake).

## Images Published

**Community builds** published to:
- `ghcr.io/kameshsampath/pg_lake`
- `ghcr.io/kameshsampath/pgduck-server`

**Note**: These are development/testing images. For production use, please consult the [official pg_lake repository](https://github.com/Snowflake-Labs/pg_lake).

## Supported Versions

- PostgreSQL 16
- PostgreSQL 17
- PostgreSQL 18

## Automated Builds

Images are automatically built in two ways:

1. **Upstream Monitoring** (Every 6 hours)
   - Monitors the [upstream pg_lake repository](https://github.com/snowflake-labs/pg_lake) for new commits to `main`
   - Monitors for new tags/releases
   - Automatically triggers builds when changes are detected
   - Tracks last built commit/tag to avoid duplicate builds

2. **Weekly Scheduled Build** (Sundays at midnight UTC)
   - Ensures images are rebuilt regularly even without upstream changes
   - Picks up base image updates and security patches

## Manual Builds

To trigger a manual build:

1. Go to [Actions](https://github.com/kameshsampath/pg_lake_builder/actions)
2. Select "Build and Push Docker Images"
3. Click "Run workflow"
4. Customize options:
   - Source repository (default: snowflake-labs/pg_lake)
   - Branch/tag to build (default: main)
   - PostgreSQL versions (default: 16,17,18)
   - Base OS (default: almalinux)
   - Platforms (default: linux/amd64,linux/arm64)

## Using the Images

```bash
# Pull an image
docker pull ghcr.io/kameshsampath/pg_lake:abc1234-pg18-almalinux

# Run it
docker run -d --name pg_lake ghcr.io/kameshsampath/pg_lake:abc1234-pg18-almalinux
```

## License

> **⚠️ DISCLAIMER**: This is a community/developer build repository.

- **This builder repository**: Configuration files are MIT licensed
- **pg_lake source code**: Licensed under Apache 2.0 by Snowflake Inc. (see [pg_lake LICENSE](https://github.com/Snowflake-Labs/pg_lake/blob/main/LICENSE))
- **Docker images produced**: Contain pg_lake software licensed under Apache 2.0
- **No warranty**: These community builds come with NO WARRANTY or official support

```

### 4. Commit and Push

```bash
git add .github/workflows/build-and-push-images.yml README.md
git commit -m "Add workflow to build pg_lake Docker images"
git push origin main
```

### 5. Enable GitHub Actions

1. Go to your repository settings
2. Navigate to "Actions" → "General"
3. Ensure "Allow all actions and reusable workflows" is selected
4. Enable "Read and write permissions" for GITHUB_TOKEN under "Workflow permissions"

### 6. Configure Permissions

The workflow needs these permissions (already configured in the workflow file):

- `contents: read` - Read repository code
- `packages: write` - Push to GitHub Container Registry
- `attestations: write` - Generate build attestations
- `id-token: write` - OIDC token for attestations

These are automatically granted to `GITHUB_TOKEN` in GitHub Actions.

## Workflow Inputs

### Manual Trigger Options

| Input | Description | Default | Example |
|-------|-------------|---------|---------|
| `pg_lake_repo` | Source repository to build from | `snowflake-labs/pg_lake` | `kameshsampath/pg_lake` |
| `pg_lake_ref` | Branch, tag, or commit to build | `main` | `v3.1`, `feature-branch`, `abc1234` |
| `pg_versions` | PostgreSQL versions (comma-separated) | `16,17,18` | `18` or `17,18` |
| `base_image_os` | Base operating system | `almalinux` | `debian` |
| `platforms` | Target architectures | `linux/amd64,linux/arm64` | `linux/amd64` |

## Usage Examples

### Build from Your Own Fork

```bash
gh workflow run build-and-push-images.yml \
  -f pg_lake_repo=kameshsampath/pg_lake \
  -f pg_lake_ref=main
```

### Build a Specific Release Tag

```bash
gh workflow run build-images.yml \
  -f pg_lake_repo=snowflake-labs/pg_lake \
  -f pg_lake_ref=v3.0.0 \
  -f pg_versions=18
```

This will create images tagged as:
- `ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux`
- `ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-debian`

### Build from a Feature Branch

```bash
gh workflow run build-and-push-images.yml \
  -f pg_lake_repo=your-username/pg_lake \
  -f pg_lake_ref=feature/new-feature \
  -f pg_versions=18 \
  -f platforms=linux/amd64
```

### Test Build (Single Platform, Single Version)

For faster testing:

```bash
gh workflow run build-and-push-images.yml \
  -f pg_versions=18 \
  -f platforms=linux/amd64
```

## Image Naming Convention

Images are tagged based on what is being built from the **pg_lake source repository**:

### Building from a Branch/Commit (SHA-based tagging)

```
ghcr.io/kameshsampath/pg_lake:<source-commit-sha>-pg<version>-<os>
ghcr.io/kameshsampath/pgduck-server:<source-commit-sha>-pg<version>-<os>
```

Example:

```
ghcr.io/kameshsampath/pg_lake:abc1234-pg18-almalinux
ghcr.io/kameshsampath/pgduck-server:abc1234-pg18-almalinux
```

Where:
- `abc1234` = First 7 characters of the pg_lake source commit
- `pg18` = PostgreSQL major version
- `almalinux` = Base OS

### Building from a Tag (Tag-based tagging) 🏷️

When building from an official release tag, the tag name is used:

```
ghcr.io/kameshsampath/pg_lake:<tag>-pg<version>-<os>
ghcr.io/kameshsampath/pgduck-server:<tag>-pg<version>-<os>
```

Example:

```
ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18-almalinux
ghcr.io/kameshsampath/pgduck-server:v3.0.0-pg18-almalinux
```

Where:
- `v3.0.0` = The release tag name
- `pg18` = PostgreSQL major version
- `almalinux` = Base OS

### Additional Tags

Each image also gets a shorter tag without the OS:

- `ghcr.io/kameshsampath/pg_lake:abc1234-pg18` (from commit)
- `ghcr.io/kameshsampath/pg_lake:v3.0.0-pg18` (from tag)

## Monitoring Builds

### View Workflow Runs

```bash
# List recent runs
gh run list --workflow=build-and-push-images.yml --repo kameshsampath/pg_lake_builder

# Watch a specific run
gh run watch <run-id> --repo kameshsampath/pg_lake_builder

# View run details
gh run view <run-id> --repo kameshsampath/pg_lake_builder
```

### Check Published Images

Visit: <https://github.com/kameshsampath?tab=packages>

Or use the CLI:

```bash
# List your packages
gh api /users/kameshsampath/packages

# Pull an image
docker pull ghcr.io/kameshsampath/pg_lake:latest
```

## Build Schedule

The repository runs multiple automated workflows:

### 1. **Upstream Monitoring** (`monitor-upstream.yml`)

Runs every 6 hours and:
- Checks for new commits on `snowflake-labs/pg_lake` main branch
- Checks for new tags/releases
- Automatically triggers builds when changes are detected
- Stores last built commit/tag in `.last-build-commit` and `.last-build-tag`

To change the monitoring frequency:

```yaml
schedule:
  - cron: "0 */6 * * *"  # Every 6 hours (current)
  - cron: "0 */1 * * *"  # Every hour (more frequent)
  - cron: "0 */12 * * *" # Every 12 hours (less frequent)
```

### 2. **Weekly Scheduled Build** (`build-images.yml`)

Runs every Sunday at 00:00 UTC:
- Ensures regular rebuilds even without upstream changes
- Picks up base image updates and security patches

```yaml
schedule:
  - cron: "0 0 * * 0"  # Every Sunday at midnight UTC
```

Examples for custom schedules:

- Daily: `"0 2 * * *"` (2 AM UTC every day)
- Monthly: `"0 0 1 * *"` (First day of each month)
- Twice weekly: `"0 0 * * 0,3"` (Sunday and Wednesday)

### 3. **Manual Trigger**

Both workflows can be triggered manually via GitHub Actions UI or CLI

## Customization

### Build Only Specific Versions

Edit the matrix in the workflow file:

```yaml
matrix:
  pg_major: [18]  # Only build PG 18
  base_image_os: [almalinux]
```

### Add Debian Builds

```yaml
matrix:
  pg_major: [16, 17, 18]
  base_image_os: [almalinux, debian]  # Build both OS variants
```

### Change Default Source Repository

If you primarily build from your fork:

```yaml
env:
  PG_LAKE_REPO: ${{ inputs.pg_lake_repo || 'kameshsampath/pg_lake' }}
  PG_LAKE_REF: ${{ inputs.pg_lake_ref || 'main' }}
```

## Troubleshooting

### Build Fails: "Permission denied while trying to connect to Docker"

Ensure Docker Buildx is properly set up. The workflow includes:

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```

### Images Not Appearing in Packages

1. Check package visibility settings (public vs private)
2. Verify GITHUB_TOKEN has `packages: write` permission
3. Wait a few minutes for the registry to update

### Authentication Errors

The workflow uses `GITHUB_TOKEN` which is automatically provided. For manual local builds:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u kameshsampath --password-stdin
```

### Build Timeout

GitHub Actions has a 6-hour job limit. If builds timeout:

1. Build fewer versions in parallel
2. Use single platform (`linux/amd64`) instead of multi-arch
3. Split into separate workflow runs

## Advanced Usage

### Building with Custom Dockerfile

If you want to use a modified Dockerfile:

1. Fork pg_lake to your account
2. Modify the Dockerfile in your fork
3. Trigger a build from your fork:

```bash
gh workflow run build-and-push-images.yml \
  -f pg_lake_repo=kameshsampath/pg_lake \
  -f pg_lake_ref=custom-dockerfile
```

### Automated Builds on Source Updates

To automatically build when the upstream pg_lake is updated, add a scheduled workflow check or use a webhook.

### Multi-Registry Publishing

To publish to both ghcr.io and Docker Hub, modify the workflow to include additional push steps with Docker Hub credentials.

## Security Considerations

1. **Submodules**: The workflow checks out with `submodules: 'recursive'` to include DuckDB
2. **Attestations**: Build provenance is generated for supply chain security
3. **Token Permissions**: Uses least-privilege permissions
4. **Public Images**: Images are public by default (adjust package settings if needed)

## Cost Considerations

- GitHub Actions minutes are **free** for public repositories
- Storage: 500MB free for packages, then $0.25/GB/month
- Bandwidth: Unlimited downloads for public images

## Related Links

- [pg_lake Source Repository](https://github.com/snowflake-labs/pg_lake)
- [GitHub Packages Documentation](https://docs.github.com/en/packages)
- [Docker Buildx Documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## Support

> **⚠️ IMPORTANT**: This is a community/developer build repository. **No official support is provided.**

For issues with:

- **pg_lake source code**: Open an issue in the [official pg_lake repository](https://github.com/snowflake-labs/pg_lake/issues)
- **Build workflow in this repo**: Open an issue in this repository (community support only)
- **Docker images from this builder**: These are **unofficial builds** - use at your own risk

**For production support**, please use official pg_lake releases and contact the Snowflake Labs team through their official channels.
