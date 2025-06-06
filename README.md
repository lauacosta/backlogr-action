
<div align="center">

# 🏷️ Taiga Task Updater Action

**GitHub Action for automatically updating [Taiga](https://taiga.io/) tasks based on commit message patterns using [`backlogr`](https://github.com/lauacosta/backlogr).**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/release/lauacosta/backlogr-action.svg)](https://github.com/lauacosta/backlogr-action/releases)

</div>

## Features

- 🔄 **Automatic Task Updates**: Update Taiga tasks based on commit messages
- 📝 **Flexible Commit Patterns**: Support multiple commit message modifiers
- 🎯 **Precise Control**: Move tasks to specific states (WIP, Done, Delete)
- 🔧 **Configurable**: Customize backlogr version and commit message parsing
- 📊 **Rich Outputs**: Get detailed information about processed tasks
- 🚀 **Easy Integration**: Simple one-line addition to existing workflows

## Usage

### Basic Setup

Create a workflow file (e.g., `.github/workflows/taiga-update.yml`):

```yaml
name: Update Taiga Tasks
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
          fetch-depth: 2
      
      - name: Update Taiga Task
        uses: lauacosta/backlogr-action@v1
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

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `taiga-username` | Taiga username or email | ✅ | |
| `taiga-password` | Taiga password | ✅ | |
| `project-name` | Taiga project name | ✅ | |
| `backlogr-version` | Version of backlogr to use | ❌ | `v0.0.1` |
| `commit-message` | Custom commit message to parse | ❌ | (auto-detected) |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `task-id` | The task ID that was processed | `123` |
| `action-taken` | Action taken on the task | `wip`, `done`, `delete`, `skipped` |
| `success` | Whether the action was successful | `true`, `false` |

## Commit Message Format

The action expects commit messages in this specific format:

```
<modifier>: <description> (#<task-id>)
```

### Supported Modifiers

| Modifier | Taiga Action | Description |
|----------|-------------|-------------|
| `feat`, `feature`, `add`, `implement` | **Move to WIP** | Start working on a new feature |
| `fix`, `bugfix`, `patch`, `hotfix` | **Move to WIP** | Start fixing a bug |
| `done`, `complete`, `finish`, `resolve` | **Move to Done** | Mark task as completed |
| `delete`, `remove`, `cancel`, `drop` | **Delete Task** | Remove task from project |
| `wip`, `progress`, `start`, `begin` | **Move to WIP** | Explicitly move to work in progress |

### Commit Message Examples

```bash
# Feature development
feat: add user authentication system (#123)
feature: implement OAuth2 integration (#456)

# Bug fixes
fix: resolve login timeout issue (#789)
hotfix: patch critical security vulnerability (#101)

# Task completion
done: complete user profile feature (#234)
finish: finalize payment integration (#567)

# Task management
delete: remove deprecated API endpoint (#890)
wip: start working on notification system (#321)
```

## Advanced Usage

### Custom Commit Message

Process a specific commit message instead of auto-detecting:

```yaml
- name: Update Taiga Task
  uses: lauacosta/backlogr-action@v1
  with:
    taiga-username: ${{ secrets.TAIGA_USERNAME }}
    taiga-password: ${{ secrets.TAIGA_PASSWORD }}
    project-name: ${{ secrets.PROJECT_NAME }}
    commit-message: "feat: custom implementation (#999)"
```

### Specific backlogr Version

Use a different version of the backlogr tool:

```yaml
- name: Update Taiga Task
  uses: lauacosta/backlogr-action@v1
  with:
    taiga-username: ${{ secrets.TAIGA_USERNAME }}
    taiga-password: ${{ secrets.TAIGA_PASSWORD }}
    project-name: ${{ secrets.PROJECT_NAME }}
    backlogr-version: "v0.0.1"
```

### Using Outputs for Further Actions

```yaml
- name: Update Taiga Task
  id: taiga
  uses: lauacosta/backlogr-action@v1
  with:
    taiga-username: ${{ secrets.TAIGA_USERNAME }}
    taiga-password: ${{ secrets.TAIGA_PASSWORD }}
    project-name: ${{ secrets.PROJECT_NAME }}

- name: Comment on PR
  if: steps.taiga.outputs.success == 'true' && github.event_name == 'pull_request'
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: `✅ Taiga task #${{ steps.taiga.outputs.task-id }} updated: **${{ steps.taiga.outputs.action-taken }}**`
      })

- name: Slack Notification
  if: steps.taiga.outputs.success == 'true'
  uses: 8398a7/action-slack@v3
  with:
    status: custom
    custom_payload: |
      {
        text: "Task #${{ steps.taiga.outputs.task-id }} moved to ${{ steps.taiga.outputs.action-taken }}"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
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
          fetch-depth: 2
      - name: Update Taiga Task
        uses: lauacosta/backlogr-action@v1
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}
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
          fetch-depth: 2
      - name: Update Taiga Task - ${{ matrix.project }}
        uses: lauacosta/backlogr-action@v1
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
          fetch-depth: 2
      - name: Update Taiga Task
        uses: lauacosta/backlogr-action@v1
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}
```

### Error Handling and Notifications

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
          fetch-depth: 2
      
      - name: Update Taiga Task
        id: taiga
        uses: lauacosta/backlogr-action@v1
        continue-on-error: true
        with:
          taiga-username: ${{ secrets.TAIGA_USERNAME }}
          taiga-password: ${{ secrets.TAIGA_PASSWORD }}
          project-name: ${{ secrets.PROJECT_NAME }}

      - name: Handle Success
        if: steps.taiga.outputs.success == 'true'
        run: |
          echo "✅ Successfully updated task #${{ steps.taiga.outputs.task-id }}"
          echo "Action taken: ${{ steps.taiga.outputs.action-taken }}"

      - name: Handle Failure
        if: steps.taiga.outputs.success == 'false'
        run: |
          echo "❌ Failed to update Taiga task"
          echo "This might be due to invalid commit format or Taiga connectivity issues"
          
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
## Requirements

- **Git History**:  
  The action requires access to commit history. Set `fetch-depth: 2` (or `fetch-depth: 0`) in the `actions/checkout` step to ensure commit history is available.

- **Runner**:  
  - ✅ Compatible with: `ubuntu-latest`, `ubuntu-22.04`, `ubuntu-20.04`, and other Linux runners that support musl binaries.  

- **Internet Access**:  
  Required to download the Backlogr binary during the workflow run.

- **Taiga Access**:  
  Requires valid Taiga credentials with access to the specified project.

## Troubleshooting

### Common Issues

#### Invalid Commit Format
```
❌ Invalid commit format.
ℹ️  Expected: <modifier>: <message> (#<task-id>)
```
**Solution**: Ensure your commit messages follow the exact format: `feat: description (#123)`

#### Task Not Found
```
❌ Failed to execute backlogr command
```
**Solution**: Verify the task ID exists in your Taiga project and you have permissions to modify it

#### Authentication Failed
```
❌ Failed to execute backlogr command
```
**Solution**: Check your Taiga credentials and ensure the project name is correct

### Debug Mode

Enable debug logging by setting the `ACTIONS_STEP_DEBUG` secret to `true` in your repository.

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

- Support for additional Taiga task states
- Integration with other project management tools
- Enhanced error handling and reporting
- Documentation improvements
- Additional workflow examples

## License

Licensed under the [MIT License](LICENSE).

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you shall be licensed under the MIT License, without any additional terms or conditions.
