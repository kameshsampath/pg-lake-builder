# Upstream Monitoring Setup

This repository automatically monitors the upstream [Snowflake-Labs/pg_lake](https://github.com/Snowflake-Labs/pg_lake) repository and triggers builds when changes are detected.

## How It Works

### Workflow: `monitor-upstream.yml`

The monitoring workflow runs **nightly at 2 AM UTC** and performs two checks:

#### 1. **Commit Monitoring**
- Checks the latest commit SHA on the `main` branch
- Compares with the last built commit (stored in `.last-build-commit`)
- If different, triggers a build workflow
- Updates `.last-build-commit` after triggering the build

#### 2. **Tag Monitoring**
- Checks for the latest tag/release
- Compares with the last built tag (stored in `.last-build-tag`)
- If a new tag is found, triggers a build workflow
- Updates `.last-build-tag` after triggering the build

## State Tracking

The monitoring state is tracked in two files:

- `.last-build-commit` - Contains the SHA of the last built commit
- `.last-build-tag` - Contains the name of the last built tag

These files are committed to the repository to maintain state between workflow runs.

## Monitoring Schedule

```yaml
schedule:
  - cron: "0 2 * * *"  # Nightly at 2 AM UTC
```

**Schedule breakdown:**
- Checks once per day at 2 AM UTC
- Minimal API calls to GitHub
- Automatically catches upstream changes overnight

## Triggered Builds

When changes are detected, the workflow triggers `build-images.yml` with:

```yaml
pg_lake_repo: snowflake-labs/pg_lake
pg_lake_ref: main (or the new tag)
pg_versions: 16,17,18
base_image_os: almalinux
platforms: linux/amd64,linux/arm64
```

## Manual Testing

Test the monitoring workflow manually:

```bash
# Trigger the monitoring workflow
gh workflow run monitor-upstream.yml

# Watch the workflow run
gh run watch

# View recent runs
gh run list --workflow=monitor-upstream.yml
```

## View Monitoring Status

Check the workflow summary in GitHub Actions:
- Go to **Actions** → **Monitor Upstream pg_lake Repository**
- Click on any run to see the summary showing:
  - Latest upstream commit/tag
  - Last built commit/tag
  - Whether a build was triggered

## Customization

### Change Monitoring Frequency

Edit `.github/workflows/monitor-upstream.yml`:

```yaml
# Every hour (more frequent)
schedule:
  - cron: "0 * * * *"

# Every 12 hours (less frequent)
schedule:
  - cron: "0 */12 * * *"

# Daily at 2 AM UTC
schedule:
  - cron: "0 2 * * *"
```

### Monitor Different Branch

Change the `UPSTREAM_BRANCH` environment variable:

```yaml
env:
  UPSTREAM_REPO: snowflake-labs/pg_lake
  UPSTREAM_BRANCH: develop  # Change from 'main'
```

### Monitor Your Own Fork

```yaml
env:
  UPSTREAM_REPO: kameshsampath/pg_lake
  UPSTREAM_BRANCH: main
```

### Build Different Configurations

Modify the trigger parameters in the workflow:

```yaml
- name: Trigger build workflow
  run: |
    gh workflow run build-images.yml \
      -f pg_versions="18"              # Only PG 18
      -f base_image_os="debian"        # Use Debian instead
      -f platforms="linux/amd64"       # Single platform
```

## Disable Monitoring

To temporarily disable automatic monitoring:

1. **Via GitHub UI:**
   - Go to **Actions** → **Monitor Upstream pg_lake Repository**
   - Click the **⋮** menu → **Disable workflow**

2. **Or comment out the schedule:**
   ```yaml
   # schedule:
   #   - cron: "0 */6 * * *"
   ```

## Troubleshooting

### Builds Not Triggering

**Check:**
1. Workflow is enabled (Actions tab)
2. No API rate limiting (check workflow logs)
3. `.last-build-commit` and `.last-build-tag` files exist
4. Workflow has correct permissions

**Permissions required:**
```yaml
permissions:
  contents: write    # To commit tracking files
  actions: write     # To trigger workflows
```

### Duplicate Builds

If the same commit triggers multiple builds:
1. Ensure `.last-build-commit` is being updated correctly
2. Check for multiple workflow instances running simultaneously
3. Verify git commits are being pushed successfully

### API Rate Limiting

GitHub API allows:
- **Authenticated**: 5,000 requests/hour
- **Unauthenticated**: 60 requests/hour

The workflow uses the `GITHUB_TOKEN` (authenticated), so rate limiting should not be an issue with daily checks.

### Reset Tracking

To force rebuild of current upstream state:

```bash
# Reset commit tracking
echo "none" > .last-build-commit
git add .last-build-commit
git commit -m "reset: force rebuild on next check"
git push

# Reset tag tracking
echo "none" > .last-build-tag
git add .last-build-tag
git commit -m "reset: force rebuild of latest tag"
git push
```

## Security Considerations

- Uses `GITHUB_TOKEN` (no secrets needed)
- Only monitors public repository data
- Commits are made by `github-actions[bot]`
- No sensitive data in tracking files

## Cost

- **GitHub Actions minutes**: Free for public repositories
- **API calls**: Well within free tier limits
- **Storage**: Negligible (2 small text files)

## Workflow Dependencies

This monitoring workflow works with:
- `build-images.yml` - Triggered to build the images
- `.last-build-commit` - State file (tracked in git)
- `.last-build-tag` - State file (tracked in git)

## Benefits

✅ **Automatic**: No manual intervention needed  
✅ **Efficient**: Only builds when upstream changes  
✅ **Transparent**: Clear tracking of what was built  
✅ **Flexible**: Easy to customize or disable  
✅ **Free**: No additional costs  

## Alternatives

If you prefer different approaches:

1. **Webhook-based** (requires external service)
   - Set up a GitHub App
   - Listen for push events on upstream repo
   - More real-time but more complex

2. **Manual only** (disable monitoring)
   - Remove/disable `monitor-upstream.yml`
   - Trigger builds manually when needed
   - More control, less automation

3. **Always rebuild** (disable tracking)
   - Remove state checking logic
   - Build on every monitoring run
   - Simpler but less efficient

## References

- [GitHub Actions - Scheduled Events](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
- [GitHub REST API - Commits](https://docs.github.com/en/rest/commits/commits)
- [GitHub REST API - Tags](https://docs.github.com/en/rest/repos/repos#list-repository-tags)
- [Triggering Workflows](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow)

