#  Code Agent

An AI Agent that operates Claude Code on GitHub Actions. By using this action, you can directly invoke Claude Code from GitHub Issues or Pull Request comments and automate code changes.

## Features

- Start Claude Code with the `/claude` command from GitHub Issues or PR comments
- Automatically create a Pull Request or commit changes if Claude Code modifies code
- Post Claude Code's output as a comment if there are no changes

## Security

* **Permission Checks:** Before executing core logic, the action verifies if the triggering user (`github.context.actor`) has `write` or `admin` permissions for the repository.
* **Sensitive Information Masking:** Any occurrences of the provided `github-token` and `anthropic-api-key`, `AWS Credentials` within the output posted to GitHub are automatically masked (replaced with `***`) to prevent accidental exposure.

## Usage

### Project Settings

#### Settings -> Actions -> General -> Workflow permissions

* Read and write permissions
* ✔ Allow GitHub Actions to create and approve pull requests

![image](https://github.com/user-attachments/assets/e78e60d0-9e16-425e-bcad-264c8f81b878)

#### Settings -> Secrets and variables -> Actions -> Secrets

* Repository secrets Set `ANTHROPIC_API_KEY`

![image](https://github.com/user-attachments/assets/8ae22808-9df5-4709-adaa-1e9d8c634f51)


### Workflow Configuration

```yaml
name: Code Agent

permissions:
  contents: write
  pull-requests: write
  issues: write

on:
  issues:
    types: [opened]
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  code-agent:
    runs-on: ubuntu-latest
    if: ${{ github.event.sender.type != 'Bot' }}
    steps:
      - uses: potproject/code-agent@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Optional settings
          # anthropic-base-url: "https://api.anthropic.com"
          # anthropic-model: "claude-3-7-sonnet-20250219"
          # anthropic-small-fast-model: "claude-3-5-haiku-20241022"
          # claude-code-use-bedrock: "1"
          # anthropic-bedrock-base-url: "https://bedrock.us-east-1.amazonaws.com"
          # aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          # aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          # aws-region: "us-east-1"
          # disable-prompt-caching: "1"
```

## Example

View on [code-agent-example Issues](https://github.com/potproject/code-agent-example/issues) / [code-agent-example Pulls](https://github.com/potproject/code-agent-example/pulls)

### Example Usage in Issues

Create a new Issue and add the following to the body:

```
/claude Please create a new API endpoint. This should be an endpoint that handles GET requests to retrieve user information.
```

Claude will then generate the necessary code according to the instructions and propose it as a Pull Request.

### Example Usage in PRs

Comment on an existing Pull Request to request code modifications:

```
/claude Please add unit tests to this code.
```

Claude will analyze the comment and add test code to the existing PR branch.


## Advanced Configuration

You can customize the behavior of Claude Code using the following environment variables:

| Input Name | Description |
|------------|-------------|
| `anthropic-base-url` | Anthropic API base URL |
| `anthropic-model` | Anthropic model to use |
| `anthropic-small-fast-model` | Small and fast model for commit message generation etc. |
| `claude-code-use-bedrock` | Use AWS Bedrock for Claude Code (0 or 1) |
| `anthropic-bedrock-base-url` | Anthropic Bedrock API base URL |
| `aws-access-key-id` | AWS Access Key ID (when using Bedrock) |
| `aws-secret-access-key` | AWS Secret Access Key (when using Bedrock) |
| `aws-region` | AWS region (when using Bedrock) |
| `disable-prompt-caching` | Disable prompt caching (0 or 1) |