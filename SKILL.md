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
│  6. Check & Update project memory                           │
│     Check CLAUDE.local.md or .claude/rules/ for api-tester  │
│     info. If not found, ask user to add it.                 │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  7. Done - Show next steps                                  │
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
│   ├── generate-postman.sh
│   ├── analyze.js        # Quick summary of large responses
│   └── slice.js          # Filter response with JS expression
├── responses/
│   ├── <env>/            # Raw responses per environment
│   └── sliced/           # Filtered/sliced data (reusable)
├── postman/
├── .gitignore
└── CLAUDE.local.md
```

## Step 5: Update Frontend Project

### 5.1 Add api-tester to allowed directories

Add api-tester directory to frontend's `.claude/settings.local.json`:

```json
{
  "permissions": {
    "additionalDirectories": [
      "<api-tester-path>"
    ]
  }
}
```

### 5.2 Create API Tester Trigger Rule

**IMPORTANT**: Create a trigger rule in the **source frontend project** so Claude knows when to use the api-tester.

Create file: `<source-project>/.claude/rules/api-tester-trigger.md`

Content:

    # API Tester Trigger Rules

    ## When to Use API Tester

    Automatically use the API Tester project when:

    1. **User mentions "api-tester"** - Any mention of api-tester, API tester, or similar
    2. **User pastes a curl command** - Detected by `curl '...` or `curl "...` pattern
    3. **User wants to test an API endpoint** - Phrases like "test this API", "call this endpoint"
    4. **User mentions API URLs** - Any of these domains:
       <list detected environment URLs>

    ## API Tester Location

    <api-tester-path>

    ## Curl Paste Workflow

    When user pastes a curl command:

    1. **Parse curl** - Extract URL, method, headers, body, and token
    2. **Detect environment** from URL domain
    3. **Save token** to `config/tokens/<env>.json`
    4. **Execute using script** - ALWAYS use `./scripts/request.sh` instead of raw curl
    5. **Response auto-saved** to `responses/<env>/<endpoint>-<METHOD>.json`

    ### Execute Request Command

    ```bash
    cd <api-tester-path>
    ./scripts/request.sh <METHOD> <ENDPOINT> '<BODY>'
    ```

    ## Important

    - **NEVER** run raw curl commands for project APIs
    - **ALWAYS** use `./scripts/request.sh` to ensure responses are saved
    - **ALWAYS** save tokens to the appropriate environment file before executing

## Step 6: Check & Update Project Memory

After creating the api-tester project, check if the frontend project's memory files contain information about the api-tester.

### Files to Check

1. `CLAUDE.local.md` - Look for "API Tester" section
2. `.claude/rules/*.md` - Look for api-tester related rules

### If NOT Found

Ask user if they want to add api-tester info to project memory:

```
I noticed your project memory doesn't have information about the API Tester.
Would you like me to add it to CLAUDE.local.md?

This will help Claude remember how to use the api-tester in future sessions.

[Add to memory] [Skip]
```

### If User Confirms

Add the following section to `CLAUDE.local.md`:

    ## API Tester

    Standalone project for testing APIs across environments.

    - **Location**: `<api-tester-path>`
    - **Usage**: Paste curl from browser DevTools to extract token and execute requests
    - **Environments**: <list environments with auto-execute status>

    Quick commands:

    ```bash
    cd <relative-path-to-api-tester>
    ./scripts/request.sh GET /api/v1/endpoint
    ./scripts/request.sh POST /api/v1/endpoint '{"key": "value"}'
    ./scripts/generate-postman.sh  # Generate Postman collection
    ```

### If Already Exists

Skip this step and proceed to summary.

## Step 7: Display Summary

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
| `scripts/analyze.js.template` | `scripts/analyze.js` |
| `scripts/slice.js.template` | `scripts/slice.js` |
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

Checking project memory...
  - CLAUDE.local.md: No api-tester info found
  - Would you like me to add it? [Add to memory] [Skip]

User: Add to memory

Claude: Added API Tester section to CLAUDE.local.md

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

## Handling Large Responses (>=15KB)

**IMPORTANT**: The generated api-tester includes scripts to handle large responses without overflowing Claude's context.

### Workflow for Large Responses

After saving response, check file size. If file >= 15KB:

1. **Run analyze.js** to get summary:

   ```bash
   node scripts/analyze.js responses/<env>/<file>.json
   ```

   Output: file size, record count, fields, unique values, time range

2. **Ask user for filter criteria** (natural language):
   - "Response has 3205 records (554KB). What filter criteria do you want?"
   - User may say: "Get records from Jan 7" or "Filter by status = active"

3. **Generate JS filter expression** based on user request:

   ```javascript
   data.filter(r => r.status === 'active')
   data.filter(r => r.ts.startsWith('2026-01-07'))
   data.slice(0, 10)
   ```

4. **Execute slice.js** to filter and save:

   ```bash
   node scripts/slice.js \
     responses/<env>/<source>.json \
     responses/sliced/<descriptive-name>.json \
     "<filter-expression>"
   ```

5. **Read sliced file** (now small enough for context)

### Sliced File Naming Convention

Format: `<env>-<source>-<filter-description>.json`

Examples:

- `staging-users-active.json`
- `production-orders-jan2026.json`

## Notes

- Always run from within the frontend project directory
- Production environments always require confirmation before executing requests
- Token files and responses are git-ignored by default
- The api-tester project is created as a sibling directory by default
