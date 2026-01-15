# API Tester Init Skill

Initialize an API tester project for the current frontend application. This skill auto-detects project info and environments from existing files.

## Invocation

```
/api-tester-init
```

Run this command from within a frontend project directory.

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  1. User is in frontend project                             │
│     cd /Users/me/projects/my-dashboard                      │
│     /api-tester-init                                        │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Auto-detect project info                                │
│     - Project name from package.json or folder name         │
│     - Environments from .env, .env.staging, .env.prod...    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Confirm with user                                       │
│     "Found: my-dashboard, 3 environments. Correct?"         │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  4. Create api-tester project                               │
│     ../my-dashboard-api-tester/                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  5. Update frontend project permissions                     │
│     Add api-tester path to .claude/settings.local.json      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  6. Done - Show next steps                                  │
└─────────────────────────────────────────────────────────────┘
```

## Step 1: Auto-Detect Project Info

From current directory, detect:

1. **Project name**: Read from `package.json` -> `name` field, or use folder name
2. **Source path**: Current working directory (no need to ask)
3. **Environments**: Scan `.env*` files for API URLs (no need to ask)

## Step 2: Scan Environment Files

Look for these files and extract API URLs:

| File | Environment Name |
|------|------------------|
| `.env` | development |
| `.env.local` | local |
| `.env.development` | development |
| `.env.staging` | staging |
| `.env.production` | production |
| `.env.prod` | production |

Extract API URL from common variable names:
- `API_URL`
- `VITE_API_URL`
- `NEXT_PUBLIC_API_URL`
- `REACT_APP_API_URL`
- `NUXT_PUBLIC_API_URL`
- `BASE_URL`
- `BACKEND_URL`

## Step 3: Confirm with User

Display detected info and ask for confirmation:

```
Detected project info:
  - Name: my-dashboard
  - Path: /Users/me/projects/my-dashboard

Found environments:
  - development: http://localhost:3001 (auto-execute: YES)
  - staging: https://api-staging.example.com (auto-execute: YES)
  - production: https://api.example.com (auto-execute: NO)

API tester will be created at:
  /Users/me/projects/my-dashboard-api-tester

Is this correct? [Confirm / Edit]
```

If user wants to edit:
- Allow changing project name
- Allow adding/removing/editing environments
- Allow changing output location

## Step 4: Create API Tester Project

Create project structure at `<parent-dir>/<project-name>-api-tester/`:

```
<project-name>-api-tester/
├── .claude/
│   └── rules/
│       ├── curl-workflow.md
│       └── api-testing.md
├── config/
│   ├── environments.json
│   └── tokens/           (empty, git-ignored)
├── reference/
│   └── api-endpoints.json
├── scripts/
│   ├── request.sh
│   ├── parse-curl.sh
│   └── generate-postman.sh
├── responses/            (empty, git-ignored)
├── postman/
├── .gitignore
└── CLAUDE.local.md
```

## Step 5: Update Frontend Project

Add api-tester directory to frontend's `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(<api-tester-path>/**)",
      "Write(<api-tester-path>/**)"
    ]
  }
}
```

## Step 6: Display Summary

```
API Tester created successfully!

Location: /Users/me/projects/my-dashboard-api-tester

Environments configured:
  - development (http://localhost:3001) - auto-execute: YES
  - staging (https://api-staging.example.com) - auto-execute: YES
  - production (https://api.example.com) - auto-execute: NO

Next steps:
  1. cd ../my-dashboard-api-tester
  2. Copy a curl from browser DevTools (Network tab -> Right click -> Copy as cURL)
  3. Paste it here and I'll handle token extraction and request execution

Or use from this project:
  - Curl paste workflow works here too (api-tester path is now allowed)
```

## Environment Safety Rules

When creating `environments.json`:

| Environment Name | canExecute | requireConfirmation |
|------------------|------------|---------------------|
| local | true | false |
| development | true | false |
| dev | true | false |
| staging | true | false |
| stage | true | false |
| test | true | false |
| **production** | **false** | **true** |
| **prod** | **false** | **true** |

## URL Pattern Detection

Generate URL patterns for `parse-curl.sh` based on detected URLs:

```bash
# Example generated patterns
if echo "$URL" | grep -qE "localhost|127.0.0.1"; then
    ENV="development"
elif echo "$URL" | grep -qE "api-staging\.example\.com"; then
    ENV="staging"
elif echo "$URL" | grep -qE "api\.example\.com"; then
    ENV="production"
fi
```

## Template Files

Templates are stored in: `~/.claude/skills/api-tester-init/templates/`

| Template | Output |
|----------|--------|
| `scripts/request.sh.template` | `scripts/request.sh` |
| `scripts/parse-curl.sh.template` | `scripts/parse-curl.sh` |
| `scripts/generate-postman.sh.template` | `scripts/generate-postman.sh` |
| `rules/curl-workflow.md.template` | `.claude/rules/curl-workflow.md` |
| `rules/api-testing.md.template` | `.claude/rules/api-testing.md` |
| `config/environments.json.template` | `config/environments.json` |
| `config/api-endpoints.json.template` | `reference/api-endpoints.json` |
| `gitignore.template` | `.gitignore` |
| `CLAUDE.local.md.template` | `CLAUDE.local.md` |

## Template Variables

Replace these placeholders when generating files:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Slug name (lowercase, dashes) | `my-dashboard` |
| `{{PROJECT_DISPLAY_NAME}}` | Display name | `My Dashboard` |
| `{{SOURCE_PROJECT_PATH}}` | Full path to frontend | `/Users/me/projects/my-dashboard` |
| `{{SOURCE_PROJECT_NAME}}` | Frontend folder name | `my-dashboard` |
| `{{DEFAULT_ENV}}` | Default environment | `development` |
| `{{DEFAULT_BASE_URL}}` | Default API URL | `http://localhost:3001` |
| `{{TODAY}}` | Current date | `2026-01-15` |
| `{{ENVIRONMENTS_JSON}}` | Generated environments config | (JSON content) |
| `{{ENVIRONMENT_TABLE}}` | Markdown table of environments | (table content) |
| `{{URL_PATTERNS}}` | Bash URL detection code | (bash code) |

## Example Session

```
User: /api-tester-init

Claude: Let me scan your project...

Detected:
  Project: my-dashboard (from package.json)
  Path: /Users/me/projects/my-dashboard

Scanning .env files...
  .env.local -> VITE_API_URL=http://localhost:3001
  .env.staging -> VITE_API_URL=https://api-staging.example.com
  .env.production -> VITE_API_URL=https://api.example.com

Found 3 environments:
  1. development: http://localhost:3001 (auto-execute: YES)
  2. staging: https://api-staging.example.com (auto-execute: YES)
  3. production: https://api.example.com (auto-execute: NO)

API tester location: /Users/me/projects/my-dashboard-api-tester

[Confirm] [Edit environments] [Change location]

User: Confirm

Claude: Creating api-tester project...

Created:
  - scripts/request.sh
  - scripts/parse-curl.sh
  - scripts/generate-postman.sh
  - config/environments.json
  - reference/api-endpoints.json
  - .claude/rules/curl-workflow.md
  - .claude/rules/api-testing.md
  - .gitignore
  - CLAUDE.local.md

Updated frontend project:
  - Added api-tester path to .claude/settings.local.json

Done! Your API tester is ready at:
  /Users/me/projects/my-dashboard-api-tester

To start testing:
  1. Copy a curl from browser DevTools
  2. Paste it here or in the api-tester project
  3. I'll extract the token and execute the request
```

## Error Handling

| Scenario | Action |
|----------|--------|
| No `.env*` files found | Ask user to manually input environments |
| No API URL found in `.env` | Ask user for API URL variable name or manual input |
| No `package.json` | Use folder name as project name |
| Output directory exists | Ask to overwrite or choose different name |
| No write permission | Show error and suggest alternative location |

## Notes

- Always run from within the frontend project directory
- Production environments always require confirmation before executing requests
- Token files and responses are git-ignored by default
- The api-tester project is created as a sibling directory by default
