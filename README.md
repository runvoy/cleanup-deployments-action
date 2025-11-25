# Cleanup Deployments Action

A GitHub Action to automatically clean up old deployments in your GitHub repository, keeping only the most recent N deployments for a specified environment.

## Features

- ✅ Automatically delete old deployments from a GitHub environment
- ✅ Configurable retention count (keep last N deployments)
- ✅ Optional exclusion of the current/most recent deployment
- ✅ Comprehensive logging of all operations
- ✅ Error handling and detailed reporting

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `token` | Yes | - | GitHub token with `deployments:write` permission |
| `environment` | Yes | - | Name of the environment to clean up deployments for |
| `keep_count` | No | `10` | Number of recent deployments to keep |
| `exclude_deployment_id` | No | `` | Optional deployment ID to exclude from deletion (e.g., the current deployment) |
| `exclude_most_recent` | No | `false` | If true, automatically exclude the most recent deployment from deletion |

## Outputs

None (this action performs cleanup operations only)

## Usage

### Basic Example

Clean up deployments keeping the last 5 for the `production` environment:

```yaml
name: Cleanup Deployments
on:
  workflow_run:
    workflows: ['Deploy']
    types: [completed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup old deployments
        uses: runvoy/github-actions-cleanup-deployments@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production
          keep_count: 5
```

### Exclude Current Deployment

Keep the 5 most recent deployments, ensuring the current deployment is never deleted:

```yaml
- name: Cleanup old deployments
  uses: runvoy/github-actions-cleanup-deployments@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    environment: production
    keep_count: 5
    exclude_most_recent: true
```

### Exclude Specific Deployment

Exclude a specific deployment ID from being deleted (useful for keeping the currently active production deployment):

```yaml
- name: Cleanup old deployments
  uses: runvoy/github-actions-cleanup-deployments@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    environment: production
    keep_count: 10
    exclude_deployment_id: '123456789'
```

### Multiple Environments

Clean up deployments for multiple environments in a single workflow:

```yaml
- name: Cleanup staging deployments
  uses: runvoy/github-actions-cleanup-deployments@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    environment: staging
    keep_count: 3

- name: Cleanup production deployments
  uses: runvoy/github-actions-cleanup-deployments@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    environment: production
    keep_count: 10
    exclude_most_recent: true
```

## Permissions

This action requires the following GitHub permissions:

```yaml
permissions:
  deployments: write
```

Note: The `GITHUB_TOKEN` automatically has `deployments:write` permission in most workflows, but you may need to explicitly grant it if your workflow has restricted permissions.

## How It Works

1. Fetches all deployments for the specified environment
2. Sorts deployments by creation date (newest first)
3. Optionally excludes the most recent deployment or a specific deployment ID
4. Deletes all deployments older than the last N kept deployments
5. Reports the number of deleted, kept, and any failed deletions

## Error Handling

- If a deployment fails to delete, the action continues processing other deployments
- Failed deletions are logged as warnings but don't cause the action to fail
- The final summary shows how many deployments were successfully deleted and any failures

## Examples

### Cleanup on Schedule

Clean up deployments on a schedule (e.g., daily):

```yaml
name: Daily Cleanup
on:
  schedule:
    - cron: '0 2 * * *'  # 2 AM UTC daily

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup old deployments
        uses: runvoy/github-actions-cleanup-deployments@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production
          keep_count: 20
```

### Cleanup After Release

Clean up deployments after a successful release:

```yaml
name: Deploy and Cleanup
on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    outputs:
      deployment_id: ${{ steps.deploy.outputs.deployment_id }}
    steps:
      - name: Deploy
        id: deploy
        run: echo "deployment_id=12345" >> $GITHUB_OUTPUT
        # Replace with your actual deployment step

  cleanup:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup old deployments
        uses: runvoy/github-actions-cleanup-deployments@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          environment: production
          keep_count: 10
          exclude_deployment_id: ${{ needs.deploy.outputs.deployment_id }}
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a pull request.
