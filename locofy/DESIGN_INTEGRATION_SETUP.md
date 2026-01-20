# 🎨 Locofy Design Integration System - Setup Guide

This document provides complete setup instructions for the automated Locofy to Main Project design integration system.

## 📋 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Setup Steps](#setup-steps)
5. [Secrets Configuration](#secrets-configuration)
6. [Testing the Workflow](#testing-the-workflow)
7. [Troubleshooting](#troubleshooting)
8. [Customization](#customization)

---

## Overview

This system automates the integration of Figma designs (via Locofy) into your main project while:

- ✅ Preserving existing functionality
- ✅ Applying new design/styling
- ✅ Following your project's coding standards
- ✅ Creating backups before any changes
- ✅ Creating PRs for human review
- ✅ Sending notifications (Slack + Email)

### Workflow Summary

```
Figma Design
     ↓
Locofy (generates React code)
     ↓
Push to Locofy Repository (this repo)
     ↓
GitHub Action triggers
     ↓
Calls Main Project Repository workflow
     ↓
┌─────────────────────────────────────┐
│ 1. Create backup branch             │
│ 2. Fetch Locofy code                │
│ 3. Find matching component          │
│ 4. AI merges design + functionality │
│ 5. Create PR for review             │
│ 6. Send notifications               │
└─────────────────────────────────────┘
     ↓
Human reviews PR
     ↓
Merge to test branch
```

---

## Architecture

### Repository Structure

```
LOCOFY REPOSITORY (this repo)
├── .github/
│   ├── workflows/
│   │   └── locofy-design-sync.yaml          # Triggers on push
│   ├── prompts/
│   │   └── integration-prompts.json         # AI prompt templates
│   └── config/
│       └── integration-config.json          # Configuration
├── src/
│   └── pages/                               # Locofy-generated components
└── DESIGN_INTEGRATION_SETUP.md              # This file

MAIN PROJECT REPOSITORY (Saleslabdev/frontend)
├── .github/
│   └── workflows/
│       └── design-integration.yaml          # Main integration workflow
├── frontend/
│   ├── services/Auth/pages/LoginPage/       # Target components
│   └── ...
└── _backups/                                # Created during integration
    ├── original/                            # Original file backups
    └── locofy_design/                       # Locofy design backups
```

---

## Prerequisites

1. **GitHub Account** with access to both repositories
2. **Anthropic API Key** for Claude AI (get from https://console.anthropic.com/)
3. **Personal Access Token (PAT)** with `repo` scope
4. **(Optional) Slack Webhook URL** for notifications
5. **(Optional) SMTP Server** for email notifications

---

## Setup Steps

### Step 1: Create Personal Access Token (PAT)

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Name: `locofy-integration-token`
4. Expiration: Choose appropriate (recommend 90 days minimum)
5. Scopes: Select `repo` (full control of private repositories)
6. Click "Generate token"
7. **Copy the token immediately** (you won't see it again)

### Step 2: Get Anthropic API Key

1. Go to https://console.anthropic.com/
2. Sign up or log in
3. Navigate to API Keys
4. Create a new API key
5. Copy the key

### Step 3: Configure Locofy Repository Secrets

Go to: `https://github.com/shivam-visions/locofy-react-vite/settings/secrets/actions`

Add these secrets:

| Secret Name | Value | Required |
|-------------|-------|----------|
| `MAIN_REPO_PAT` | Your PAT from Step 1 | ✅ Yes |

### Step 4: Configure Main Project Repository Secrets

Go to: `https://github.com/Saleslabdev/frontend/settings/secrets/actions`

Add these secrets:

| Secret Name | Value | Required |
|-------------|-------|----------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key | ✅ Yes |
| `LOCOFY_REPO_PAT` | Your PAT from Step 1 | ✅ Yes |
| `SLACK_WEBHOOK_URL` | Slack incoming webhook URL | ❌ Optional |
| `SMTP_SERVER` | SMTP server address (e.g., smtp.gmail.com) | ❌ Optional |
| `SMTP_PORT` | SMTP port (e.g., 587) | ❌ Optional |
| `SMTP_USERNAME` | SMTP username | ❌ Optional |
| `SMTP_PASSWORD` | SMTP password or app password | ❌ Optional |
| `NOTIFICATION_EMAIL` | Email to receive notifications | ❌ Optional |

### Step 5: Copy Workflow to Main Repository

Copy the file `.github/workflows/design-integration-workflow-for-main-repo.yaml` to the main repository:

```bash
# Clone main repository
git clone https://github.com/Saleslabdev/frontend.git
cd frontend

# Create workflow directory if not exists
mkdir -p .github/workflows

# Copy the workflow file (rename it)
cp /path/to/locofy/.github/workflows/design-integration-workflow-for-main-repo.yaml \
   .github/workflows/design-integration.yaml

# Commit and push
git add .github/workflows/design-integration.yaml
git commit -m "feat: add Locofy design integration workflow"
git push origin test
```

### Step 6: Create Required Labels in Main Repository

Go to: `https://github.com/Saleslabdev/frontend/labels`

Create these labels:
- `design` (color: #7057ff)
- `automated` (color: #0e8a16)
- `needs-review` (color: #d93f0b)

---

## Secrets Configuration

### Slack Webhook Setup (Optional)

1. Go to https://api.slack.com/apps
2. Create a new app or select existing
3. Enable "Incoming Webhooks"
4. Add new webhook to workspace
5. Choose the channel (e.g., #design-updates)
6. Copy the webhook URL
7. Add as `SLACK_WEBHOOK_URL` secret

### Email Setup with Gmail (Optional)

1. Enable 2-Factor Authentication on your Google account
2. Go to Google Account → Security → App passwords
3. Generate an app password for "Mail"
4. Use these values:
   - `SMTP_SERVER`: smtp.gmail.com
   - `SMTP_PORT`: 587
   - `SMTP_USERNAME`: your-email@gmail.com
   - `SMTP_PASSWORD`: (the app password you generated)

---

## Testing the Workflow

### Manual Test

1. Go to the Locofy repository Actions tab
2. Select "Locofy Design Sync" workflow
3. Click "Run workflow"
4. Enter:
   - Component name: `Login`
   - Force sync: `true`
5. Click "Run workflow"

### Automatic Test

1. Create or modify a file in `src/pages/Login/` in the Locofy repo
2. Commit and push to `main` branch
3. Watch the Actions tab for the workflow to trigger

### Expected Results

1. ✅ Locofy workflow detects changes
2. ✅ Triggers main repository workflow
3. ✅ Creates backup branch (e.g., `backup/login-20260120_143000`)
4. ✅ Creates integration branch (e.g., `design-update/login-20260120_143000`)
5. ✅ AI merges the code
6. ✅ Creates a PR with detailed description
7. ✅ Sends Slack/Email notification

---

## Troubleshooting

### Common Issues

#### 1. "Resource not accessible by integration"

**Cause:** PAT doesn't have sufficient permissions

**Fix:** 
- Ensure PAT has `repo` scope
- Ensure PAT owner has write access to both repositories

#### 2. "No component files changed"

**Cause:** Files not in expected location

**Fix:**
- Ensure Locofy output is in `src/pages/` or `src/components/`
- Check file extensions (.tsx, .ts, .jsx, .js)

#### 3. "AI integration failed"

**Cause:** Claude API error

**Fix:**
- Check `ANTHROPIC_API_KEY` is valid
- Check API usage limits
- Review the API response in workflow logs

#### 4. "Could not find matching files"

**Cause:** Component name mismatch

**Fix:**
- Use explicit mappings in `integration-config.json`
- Ensure consistent naming between repos

### Viewing Logs

1. Go to repository → Actions
2. Click on the failed workflow run
3. Expand the failed step
4. Review the logs for error details

---

## Customization

### Adding Explicit Component Mappings

Edit `.github/config/integration-config.json`:

```json
{
  "component_mapping": {
    "explicit_mappings": {
      "MyComponent": "path/to/MyComponent.tsx",
      "AnotherComponent": "different/path/Component.tsx"
    }
  }
}
```

### Modifying AI Prompts

Edit `.github/prompts/integration-prompts.json` to:
- Add project-specific instructions
- Include more context about your coding standards
- Add examples of good merges

### Changing Branch Names

Edit the workflow files to modify:
- `MAIN_REPO_BRANCH` - target branch in main repo
- Branch naming patterns in the workflow

### Adding More Reviewers

Edit the PR creation step to add reviewers:

```yaml
gh pr create \
  --reviewer username1,username2 \
  ...
```

---

## Files Reference

| File | Location | Purpose |
|------|----------|---------|
| `locofy-design-sync.yaml` | Locofy repo: `.github/workflows/` | Triggers on push |
| `design-integration.yaml` | Main repo: `.github/workflows/` | Main integration logic |
| `integration-prompts.json` | Locofy repo: `.github/prompts/` | AI prompt templates |
| `integration-config.json` | Locofy repo: `.github/config/` | Configuration settings |

---

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review workflow logs in GitHub Actions
3. Ensure all secrets are correctly configured

---

## Security Notes

- Never commit secrets to the repository
- Use GitHub Secrets for all sensitive values
- Rotate PATs periodically
- Monitor API usage to detect anomalies
- The `_backups` folder is created in the PR branch, not in the main branch

---

*Last updated: January 2026*
