# Workflows Shared

This repository contains shared workflow implementations and tools.

## Automated Git Commit Workflow

### Overview

An n8n workflow that automatically commits and pushes Git changes with AI-generated commit messages, running every 15 minutes.

### Features

- **Automated Scheduling**: Runs every 15 minutes to check for changes
- **Smart Detection**: Only commits when there are actual file changes
- **AI Commit Messages**: Uses AI to generate meaningful commit messages based on file changes
- **Git Operations**: Handles add, commit, and push operations automatically

### Workflow Structure

1. **Schedule Trigger** - Every 15 minutes
2. **Git Status** - Check current repository state
3. **If Changes Exist** - Only proceed if files were modified/added/deleted
4. **Generate Commit Message** - AI creates meaningful commit message
5. **Git Add** - Stage all changes
6. **Git Commit** - Commit with AI-generated message
7. **Git Push** - Push to remote repository

### Files

- `automated-git-commit-workflow.json` - Complete n8n workflow export
- `implementation_plan.md` - Detailed implementation documentation

### Setup Requirements

- n8n instance with Git nodes enabled
- Local Git repository at specified path (`/home/krusty/projects/workflows-shared`)
- AI language model configured (OpenAI, Anthropic, etc.)
- Git credentials configured for push operations

### Configuration Notes

- AI Agent requires a language model connection (currently empty in export - configure in n8n UI)
- Repository path is hardcoded - adjust for your environment
- Schedule interval can be modified in the Schedule Trigger node
- Error handling can be enhanced by connecting error outputs to notification nodes

### Usage

1. Import the JSON file into n8n
2. Configure AI language model connection for the "Generate Commit Message" node
3. Update repository path if different from `/home/krusty/projects/workflows-shared`
4. Configure Git credentials if needed
5. Activate the workflow

### Security Considerations

- Ensure repository credentials are properly secured
- Test in a non-production environment first
- Monitor for conflicts if multiple users push to the same repository
- Consider repository permissions and access controls

### Maintenance

- Monitor workflow executions in n8n
- Check for failed executions and resolve issues
- Update AI prompts if needed for better commit messages
- Adjust schedule timing if conflicts arise

Created as part of automated workflow implementation project.
