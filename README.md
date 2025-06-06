
<div align="center">

# 🏷️ Backlogr Action

**GitHub Action for automatically updating [Taiga](https://taiga.io/) user stories based on commit message patterns using [`backlogr`](https://github.com/lauacosta/backlogr).**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/release/lauacosta/backlogr-action.svg)](https://github.com/lauacosta/backlogr-action/releases)

</div>

## Table of Contents
- [Quick Start](#quick-start)
- [Features](#features)
- [Usage](#usage)
- [Configuration Reference](#configuration-reference)
- [Commit Message Format](#commit-message-format)
- [Workflow Examples](#workflow-examples)
- [FAQ](#faq)
- [Security](#security)
- [Performance](#performance)
- [Troubleshooting](#troubleshooting)
- [Migration](#migration)
- [Contributing](#contributing)

## Quick Start

Get up and running in 30 seconds:

1. **Add secrets** to your repository: `TAIGA_USERNAME`, `TAIGA_PASSWORD`, `PROJECT_NAME`
2. **Create** `.github/workflows/taiga.yml`:
   ```yaml
   name: Update Taiga
   on: [push]
   jobs:
     taiga:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
           with:
             fetch-depth: 0 # Required to access full git history for commit parsing
         - uses: lauacosta/backlogr-action@v0.0.2
           with:
             taiga-username: ${{ secrets.TAIGA_USERNAME }}
             taiga-password: ${{ secrets.TAIGA_PASSWORD }}
             project-name: ${{ secrets.PROJECT_NAME }}
   ```
3. **Commit** with format: `feat: your description (#123)`

## Features

- 🔄 **Automatic User Story Updates**: Update Taiga user stories based on commit messages
- 📝 **Flexible Commit Patterns**: Support multiple commit message modifiers
- 🎯 **Precise Control**: Move user stories to specific states (WIP, Done) or delete them permanently
- 🔧 **Configurable**: Customize backlogr version and commit message parsing
- 📊 **Rich Outputs**: Get detailed information about processed user stories
- 🚀 **Easy Integration**: Simple one-line addition to existing workflows

## Usage

### Basic Setup

Create a workflow file (e.g., `.github/workflows/taiga-update.yml`):

```yaml
name: Update Taiga user stories
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  update-taiga:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing
                
      - name: Update Taiga User Story
        uses: lauacosta/backlogr-action@v0.0.2
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}
```

### Required Secrets

Set up these secrets in your repository settings:

- `TAIGA_USERNAME`: Your Taiga username or email
- `TAIGA_PASSWORD`: Your Taiga password
- `PROJECT_NAME`: The name of your Taiga project

## Configuration Reference

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `taiga-username` | Taiga username or email | ✅ | |
| `taiga-password` | Taiga password | ✅ | |
| `project-name` | Taiga project name | ✅ | |
| `backlogr-version` | Version of backlogr to use | ❌ | `v0.0.2` |
| `commit-message` | Custom commit message to parse | ❌ | (auto-detected) |

### Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `user-story-id` | The User Story ID that was processed | `123` |
| `action-taken` | Action taken on the User Story | `wip`, `done`, `delete`, `skipped` |
| `success` | Whether the action was successful | `true`, `false` |

### Workflow Permissions

Required permissions for full functionality:

```yaml
permissions:
  contents: read        # For checkout
  issues: write         # For PR comments
  pull-requests: write  # For PR comments
```

### Taiga Requirements

- ✅ Active Taiga account
- ✅ Project access (member or admin)
- ✅ User Story modification permissions
- ✅ Valid User Story IDs in commits

## Commit Message Format

### Pattern

```
<modifier>: <description> (#<user-story-id>)
    ↑           ↑         ↑
 Action    What you did  User Story ID
```

### Supported Modifiers

| Modifier | Taiga Action | Description |
|----------|-------------|-------------|
| `feat`, `feature`, `add`, `implement` | **Move to WIP** | Start working on a new feature |
| `fix`, `bugfix`, `patch`, `hotfix` | **Move to WIP** | Start fixing a bug |
| `done`, `complete`, `finish`, `resolve` | **Move to Done** | Mark User Story as completed |
| `delete`, `remove`, `cancel`, `drop` | **Delete User Story** | Remove User Story from project |
| `wip`, `progress`, `start`, `begin` | **Move to WIP** | Explicitly move to work in progress |

### Examples with Results
```bash
# Feature development → WIP
feat: add user authentication system (#123)
feature: implement OAuth2 integration (#456)

# Bug fixes → WIP
fix: resolve login timeout issue (#789)
hotfix: patch critical security vulnerability (#101)

# User Story completion → Done
done: complete user profile feature (#234)
finish: finalize payment integration (#567)
```

## Workflow Examples

### Basic CI/CD Integration

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  update-taiga:
    needs: build
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing
          
      - name: Update Taiga User Story
        uses: lauacosta/backlogr-action@v0.0.2
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}
```

### Complete CI/CD Integration with PR Comments

```yaml
name: Update Taiga User Story on PR

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  issues: write
  pull-requests: write
  contents: read

jobs:
  update-taiga:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing

      - name: Update Taiga User Story
        id: taiga
        uses: lauacosta/backlogr-action@v0.0.2
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}

      - name: Handle Success
        if: steps.taiga.outputs.success == 'true'
        run: |
          echo "✅ Successfully updated User Story #${{ steps.taiga.outputs.user-story-id }}"
          echo "Action taken: ${{ steps.taiga.outputs.action-taken }}"

      - name: Handle Failure
        if: steps.taiga.outputs.success == 'false'
        run: |
          echo "❌ Failed to update Taiga User Story"
          echo "This might be due to invalid commit format or Taiga connectivity issues"

      - name: Comment on PR
        if: steps.taiga.outputs.success == 'true' && github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `✅ Taiga User Story #${{ steps.taiga.outputs.user-story-id }} updated: **${{ steps.taiga.outputs.action-taken }}**`
            })

      - name: Slack Notification
        if: steps.taiga.outputs.success == 'true'
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              "text": "User Story #${{ steps.taiga.outputs.user-story-id }} moved to ${{ steps.taiga.outputs.action-taken }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Multiple Projects Setup

```yaml
name: Update Multiple Taiga Projects
on:
  push:
    branches: [main]

jobs:
  update-taiga:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        project: [frontend, backend, mobile]
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing
          
      - name: Update Taiga User Story for ${{ matrix.project }}
        uses: lauacosta/backlogr-action@v0.0.2
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets[format('PROJECT_NAME_{0}', matrix.project)] }}
```

### Conditional Updates Based on File Changes

```yaml
name: Smart Taiga Updates
on:
  push:
    branches: [main]

jobs:
  check-changes:
    runs-on: ubuntu-latest
    outputs:
      has-code-changes: ${{ steps.changes.outputs.code }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            code:
              - 'src/**'
              - 'tests/**'
              - 'package.json'

  update-taiga:
    needs: check-changes
    if: needs.check-changes.outputs.has-code-changes == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing
          
      - name: Update Taiga User Story
        uses: lauacosta/backlogr-action@v0.0.2
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}
```

## Advanced Usage

### Custom Commit Message

Process a specific commit message instead of auto-detecting:

```yaml
- name: Update Taiga User Story
  uses: lauacosta/backlogr-action@v0.0.2
  with:
    taiga-username: ${{ secrets.TAIGA_USERNAME }}
    taiga-password: ${{ secrets.TAIGA_PASSWORD }}
    project-name: ${{ secrets.PROJECT_NAME }}
    commit-message: "feat: custom implementation (#999)"
```

### Specific backlogr Version

Use a different version of the backlogr tool:

```yaml
- name: Update Taiga User Story
  uses: lauacosta/backlogr-action@v0.0.2
  with:
    taiga-username: ${{ secrets.TAIGA_USERNAME }}
    taiga-password: ${{ secrets.TAIGA_PASSWORD }}
    project-name: ${{ secrets.PROJECT_NAME }}
    backlogr-version: "v0.0.2"
```

#### Error Handling and Notifications

```yaml
name: Robust Taiga Updates
on:
  push:
    branches: [main]

jobs:
  update-taiga:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Required to access full git history for commit parsing
      
      - name: Update Taiga User Story
        id: taiga
        uses: lauacosta/backlogr-action@v0.0.2
        continue-on-error: true
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}

      - name: Report Status
        run: |
          if [ "${{ steps.taiga.outputs.success }}" == "true" ]; then
            echo "✅ User Story #${{ steps.taiga.outputs.user-story-id }} → ${{ steps.taiga.outputs.action-taken }}"
          else
            echo "❌ User Story update failed - check commit format and Taiga connection"
            exit 1
          fi
          
      - name: Always notify team
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: |
            Taiga update ${{ job.status }} for commit: ${{ github.event.head_commit.message }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```


## FAQ

### Why isn't my User Story updating?

1. **Check User Story ID**: Ensure the User Story exists in your Taiga project
2. **Verify permissions**: You need modification rights for the User Story
3. **Validate format**: Use exact format `modifier: description (#123)`
4. **Debug logs**: Enable `ACTIONS_STEP_DEBUG=true`

### Can I use custom Taiga states?

Currently supports: New → WIP → Done. Custom states require backlogr updates.

### Does it work with Taiga Cloud and self-hosted?

Yes, both are supported. Ensure your Taiga instance is accessible from GitHub Actions.

### How do I handle multiple user stories in one commit?

Each commit processes one User Story. For multiple user stories, use separate commits or multiple User Story references.

### What if my commit doesn't match the pattern?

The action will skip processing and output `action-taken: skipped`. No error is thrown.

### Can I use this with other project management tools?

Currently only Taiga is supported. The action uses the backlogr tool which is Taiga-specific.

## Security

### Credential Management

- ✅ Always use GitHub Secrets for credentials
- ✅ Use environment-specific secrets for different stages
- ❌ Never hardcode credentials in workflows
- ❌ Avoid logging sensitive information

### Repository Secrets Setup

```bash
# Required secrets
gh secret set TAIGA_USERNAME
gh secret set TAIGA_PASSWORD  
gh secret set PROJECT_NAME

# Optional: Environment-specific
gh secret set TAIGA_USERNAME_STAGING
gh secret set TAIGA_PASSWORD_STAGING
```

### Permissions Principle

Grant minimal required permissions:

```yaml
permissions:
  contents: read
  issues: write      # Only if using PR comments
  pull-requests: write # Only if using PR comments
```

## Performance

### Action Speed

- ⚡ ~5-10 seconds per execution
- 📦 Downloads backlogr binary once per run
- 🌐 Network latency depends on Taiga instance location

### Rate Limits

- Taiga API: Depends on your instance configuration
- GitHub Actions: Standard limits apply
- Recommendation: Use for pushes to main branches, not every commit

## Requirements

- **Git History**: The action requires access to commit history. Set `fetch-depth: 0` in the `actions/checkout` step to ensure commit history is available.

- **Runner**: Compatible with: `ubuntu-latest`, `ubuntu-22.04`, `ubuntu-20.04`, and other Linux runners that support musl binaries.

- **Internet Access**: Required to download the Backlogr binary during the workflow run.

- **Taiga Access**: Requires valid Taiga credentials with access to the specified project.

## Troubleshooting

### Common Issues

#### Invalid Commit Format
```
❌ Invalid commit format.
ℹ️  Expected: <modifier>: <message> (#<user-story-id>)
```
**Solution**: Ensure your commit messages follow the exact format: `feat: description (#123)`

#### User Story Not Found
```
❌ Failed to execute backlogr command
```
**Solution**: Verify the User Story ID exists in your Taiga project and you have permissions to modify it

#### Authentication Failed
```
❌ Failed to execute backlogr command
```
**Solution**: Check your Taiga credentials and ensure the project name is correct

### Testing Locally

You can test the commit message parsing logic locally:

```bash
echo "feat: add new feature (#123)" | grep -E '^([^:]+):[[:space:]]*(.+)[[:space:]]*\(#([0-9]+)\)$'
```

## Development

### Project Structure

```
backlogr-action/
├── action.yml        
├── README.md         
├── LICENSE           
```

## Related Projects

- [backlogr](https://github.com/lauacosta/backlogr) - The underlying tool for Taiga integration
- [Taiga](https://taiga.io/) - The project management platform

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Areas for Contribution

- Support for additional Taiga User Story states
- Integration with other project management tools
- Enhanced error handling and reporting
- Documentation improvements
- Additional workflow examples

## License

Licensed under the [MIT License](LICENSE).

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you shall be licensed under the MIT License, without any additional terms or conditions.
