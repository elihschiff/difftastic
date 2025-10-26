# GitHub Actions Workflows

## Semantic Diff for Pull Requests

This repository provides GitHub Actions workflows that automatically generate semantic diffs using [difftastic](https://github.com/Wilfred/difftastic) for pull requests.

### Available Workflows

#### `reusable-semantic-diff.yml` - Universal Reusable Workflow

This is a reusable workflow that works in any repository, including this one. It installs difftastic from crates.io and generates semantic diffs.

Features:
- Install difftastic from crates.io (configurable version)
- Generate semantic diffs for all changed files in a PR
- Post a comment on the PR with collapsible diff sections
- Upload an interactive HTML diff viewer as an artifact

#### `pr-semantic-diff.yml` - Local Workflow Caller

This workflow is used in the difftastic repository itself and simply calls the reusable workflow above.

### Usage

**In this repository:**

The workflow runs automatically on all PRs. No setup needed!

**In other repositories:**

Create a workflow file in your repository (e.g., `.github/workflows/semantic-diff.yml`):

```yaml
name: Semantic Diff

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  contents: read

jobs:
  diff:
    uses: Wilfred/difftastic/.github/workflows/reusable-semantic-diff.yml@master
```

**With custom options:**

```yaml
name: Semantic Diff

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  contents: read

jobs:
  diff:
    uses: Wilfred/difftastic/.github/workflows/reusable-semantic-diff.yml@master
    with:
      # Specify difftastic version (optional, uses latest if not specified)
      difftastic-version: '0.66.0'

      # Only include specific file patterns (optional)
      file-patterns: '.*\.(rs|py|js)$'

      # Exclude specific patterns (optional)
      exclude-patterns: 'test.*|.*\.md$'
```

### Workflow Inputs

The reusable workflow accepts the following inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `difftastic-version` | Version of difftastic to install (e.g., `0.66.0`). Leave empty for latest. | No | `''` (latest) |
| `file-patterns` | File patterns to include in diff (space-separated regex). Leave empty for all files. | No | `''` (all files) |
| `exclude-patterns` | File patterns to exclude from diff (space-separated regex). | No | `''` (no exclusions) |

### Features

#### 1. **PR Comment with Collapsible Sections**
- Automatically creates a comment on the PR with semantic diffs
- Each file's diff is in a collapsible `<details>` section
- Updates the same comment on subsequent pushes (doesn't spam with multiple comments)

#### 2. **Interactive HTML Diff Viewer**
- Generates a beautiful HTML page with all diffs
- Dark theme matching GitHub's interface
- Click to expand/collapse individual file diffs
- "Expand All" and "Collapse All" buttons for convenience
- Download as an artifact from the Actions run

#### 3. **Smart File Processing**
- Only processes files that actually exist in both base and head commits
- Skips new files and deleted files (no diff to show)
- Handles binary files gracefully
- Uses difftastic's semantic analysis to show meaningful changes

### Example Output

#### PR Comment:
```markdown
## 🔍 Semantic Diff Summary

<details>
<summary>📊 View Semantic Diffs (Click to expand)</summary>

<details>
<summary><code>src/main.rs</code></summary>

[Semantic diff output here]

</details>

</details>

---

📄 **[View Full HTML Diff Viewer](https://github.com/user/repo/actions/runs/123456)**

Download the `semantic-diff-viewer` artifact from the Actions run above.
```

#### HTML Viewer:
A standalone HTML page with:
- Repository and PR information at the top
- Statistics showing number of files with changes
- Each file in a collapsible card with syntax-highlighted diffs
- Fully self-contained (can be opened locally after download)

### Permissions Required

The workflows require the following permissions:

```yaml
permissions:
  pull-requests: write  # To create/update PR comments
  contents: read        # To read repository contents
```

### Troubleshooting

#### Issue: Workflow doesn't have permission to comment
**Solution:** Ensure your repository settings allow GitHub Actions to create comments. Go to Settings → Actions → General → Workflow permissions and select "Read and write permissions".

#### Issue: Cargo/Rust not found
**Solution:** The workflow installs Rust automatically using `dtolnay/rust-toolchain`. This should work on all standard GitHub runners.

#### Issue: Diffs are empty
**Solution:** This can happen if:
- Files are identical (no semantic changes)
- Files are binary
- Files failed to parse (difftastic falls back to line-based diff for parse errors)

#### Issue: Comment not updating
**Solution:** The workflow uses `peter-evans/find-comment` to find existing comments. Make sure the comment body includes "Semantic Diff Summary" to be found.

### Advanced Usage

#### Custom Difftastic Options

If you need to customize how difftastic runs (e.g., different display mode, parse error limits), you can fork this workflow and modify the `$DIFFT_CMD` invocations in the "Generate semantic diffs" step.

Example customizations:
```bash
# Allow more parse errors
export DFT_PARSE_ERROR_LIMIT=50

# Use inline display instead of side-by-side
$DIFFT_CMD --display=inline "$BASE_TEMP" "$HEAD_TEMP"

# Adjust context lines
$DIFFT_CMD --context=10 "$BASE_TEMP" "$HEAD_TEMP"
```

#### Filtering by File Type

To only show diffs for specific file types:

```yaml
jobs:
  diff:
    uses: Wilfred/difftastic/.github/workflows/reusable-semantic-diff.yml@master
    with:
      # Only Python and JavaScript files
      file-patterns: '.*\.(py|js|jsx|ts|tsx)$'
```

#### Excluding Generated Files

To exclude generated or minified files:

```yaml
jobs:
  diff:
    uses: Wilfred/difftastic/.github/workflows/reusable-semantic-diff.yml@master
    with:
      exclude-patterns: '.*\.min\.js$|dist/|build/|.*\.generated\..*'
```

### Contributing

Improvements to these workflows are welcome! Please test changes thoroughly before submitting PRs.

### License

These workflows are provided under the same MIT license as difftastic.
