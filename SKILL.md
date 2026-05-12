---
name: api-tester-init
description: Initialize a standalone api-tester project for the current frontend application. Auto-detects project name and environments from .env files, scaffolds the api-tester project (scripts, config, rules, structure), and generates a project-specific trigger skill at ~/.claude/skills/<project>-api-tester/ that orchestrates the curl-paste workflow in future sessions. Use when the user asks to "setup api tester", "create api tester", "init api testing", or wants a curl-based API testing workflow for a frontend project.
disable-model-invocation: true
argument-hint: <no args — run from inside the frontend project directory>
---

# api-tester-init

Initialize a standalone api-tester project for the current frontend application. This skill auto-detects project info, scaffolds the tester project, and creates a per-project trigger skill so future sessions can invoke `/<project>-api-tester` and follow the persisted-req+res workflow.

## Reference implementation

The canonical example is `cisgenx-api-tester` (skill) + `/Users/niafam/WORK/CISGENICS/web-cisgenx-worktree/cisgenics-api-tester` (project). This skill scaffolds the same shape for any frontend project.

## Invocation

```
/api-tester-init
```

Run from inside a frontend project directory. The skill detects everything from the working tree.

## Workflow overview

1. Detect project info from `package.json` + `.env*` files.
2. Confirm with user (one shot — show all detected info, allow edits).
3. Create `<parent>/<project>-api-tester/` with scripts, config, rules, structure.
4. Generate the project-specific trigger skill at `~/.claude/skills/<project>-api-tester/SKILL.md`.
5. Update the frontend project's `.claude/settings.local.json` to allow access to the api-tester path.
6. Optionally add an "API Tester" section to `CLAUDE.local.md` in the frontend project.
7. Show summary + next steps.

## Step 1: Auto-detect project info

From current working directory, detect:

1. **Project name** — read `package.json#name`, fall back to folder name. Normalize to kebab-case for `<project>-api-tester` folder + skill name.
2. **Project display name** — Title Case version of project name for human-readable references.
3. **Source path** — current working directory.
4. **Environments** — scan `.env*` files for API base URLs.

## Step 2: Scan environment files

| File | Default env name |
|------|------------------|
| `.env` | `development` |
| `.env.local` | `local` |
| `.env.development` | `development` |
| `.env.staging` | `staging` |
| `.env.preprod` | `preprod` |
| `.env.production` | `production` |
| `.env.prod` | `production` |

Extract API base URL from common variable names:

- `API_URL`, `BASE_URL`, `BACKEND_URL`
- `VITE_API_URL`, `VITE_BASE_URL`
- `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_BASE_URL`
- `REACT_APP_API_URL`
- `NUXT_PUBLIC_API_URL`

Derive the **host** of each URL — it will drive env detection from pasted curls later. If multiple env files point at the same host, prefer the more specific env name (e.g. prefer `staging` over `development`).

## Step 3: Confirm with user

Show everything at once and ask for confirmation. Example:

```
Detected:
  Project: my-dashboard (from package.json)
  Path:    /Users/me/projects/my-dashboard

Environments:
  - development → http://localhost:3001               (canExecute: true)
  - staging     → https://api-staging.example.com     (canExecute: true)
  - production  → https://api.example.com             (canExecute: false)

API tester will be created at:
  /Users/me/projects/my-dashboard-api-tester

Trigger skill will be created at:
  ~/.claude/skills/my-dashboard-api-tester/SKILL.md

Confirm? [yes / edit envs / change name]
```

If user wants to edit: allow adding/removing/renaming environments, changing canExecute flags, changing project name.

## Step 4: Create the api-tester project

Scaffold at `<parent>/<project>-api-tester/`:

```
<project>-api-tester/
├── CLAUDE.local.md
├── .claude/
│   └── rules/
│       ├── curl-workflow.md
│       └── api-testing.md
├── config/
│   ├── environments.json
│   ├── state.json                # activeEnvironment + lastRequest (git-ignored)
│   └── tokens/                   # bearer per env (git-ignored)
├── reference/
│   └── api-endpoints.json
├── scripts/
│   ├── request.sh                # persists requests/<env>/ + responses/<env>/
│   ├── parse-curl.sh
│   ├── generate-postman.sh
│   ├── analyze.js                # large-response summariser
│   └── slice.js                  # JS-filter slicer
├── requests/                     # req body + _meta per call (git-ignored)
├── responses/
│   ├── <env>/                    # pretty JSON responses per env (git-ignored)
│   └── sliced/                   # filtered subsets (reusable)
├── postman/                      # generated Postman collections
├── .gitignore
```

Write `config/state.json` as `{"activeEnvironment": "<DEFAULT_ENV>", "lastRequest": null}` so the first `request.sh` call has a valid file to update.

## Step 5: Generate trigger skill

Create `~/.claude/skills/<project>-api-tester/SKILL.md` from `templates/skill/trigger-skill.md.template`.

Variables to substitute:

| Variable | Description | Example |
|---|---|---|
| `{{PROJECT_NAME}}` | kebab-case slug | `my-dashboard` |
| `{{PROJECT_DISPLAY_NAME}}` | Title Case name | `My Dashboard` |
| `{{API_TESTER_PATH}}` | absolute path to api-tester | `/Users/me/projects/my-dashboard-api-tester` |
| `{{API_HOSTS_INLINE}}` | comma-separated hosts for `description:` | `api-staging.example.com / api.example.com` |
| `{{ENV_HOST_TABLE}}` | markdown rows: `\| <host> \| <env> \|` | (one line per env) |

After writing, the user can run `/<project>-api-tester` in any session and it will follow the cisgenx pattern.

## Step 6: Update frontend project

Add the api-tester path to the frontend's `.claude/settings.local.json` so Claude Code in the frontend project can reach across:

```json
{
  "permissions": {
    "additionalDirectories": [
      "<absolute-path-to-api-tester>"
    ]
  }
}
```

Merge into existing `additionalDirectories` if the array already exists. Create the file if missing.

## Step 7: Update project memory (optional)

Check `<source-project>/CLAUDE.local.md` for an existing "API Tester" section. If absent, ask the user once:

```
Add an "API Tester" section to <source-project>/CLAUDE.local.md? [yes / skip]
```

If yes, append:

```markdown
## API Tester

Standalone project for testing APIs across environments.

- **Location**: <api-tester-path>
- **Trigger skill**: `/<project>-api-tester` (paste a curl, the skill handles env detection, token save, request execution, response persistence)
- **Environments**: <list>

Manual commands:

```bash
cd <relative-path-to-api-tester>
./scripts/request.sh GET /api/v1/endpoint
./scripts/request.sh POST /api/v1/endpoint '{"key": "value"}'
```

```

If the section already exists, skip silently.

## Step 8: Summary

Show:

```
API Tester scaffolded at:
  <api-tester-path>

Trigger skill registered at:
  ~/.claude/skills/<project>-api-tester/SKILL.md

Environments:
  - <env>: <url> (canExecute: <bool>)
  ...

Next steps:
1. Open browser DevTools → Network → "Copy as cURL" on an API call
2. In any Claude Code session: /<project>-api-tester
3. Paste the curl — the skill saves the token, executes via scripts/request.sh, and persists req + res
```

## Environment safety defaults

When generating `environments.json`:

| Env name pattern | canExecute |
|---|---|
| `local`, `development`, `dev`, `staging`, `stage`, `test`, `preprod` | `true` |
| `production`, `prod` | `false` |

`canExecute: false` does NOT block the script — production warning is handled inside `request.sh` and by the trigger skill's "Hard rules" (never mutate prod without explicit user confirmation).

## URL pattern detection (for parse-curl.sh)

Generated `parse-curl.sh` derives env from host. Pattern:

```bash
case "$URL" in
    *localhost*|*127.0.0.1*) ENV="development" ;;
    *api-staging.example.com*) ENV="staging" ;;
    *api.example.com*) ENV="production" ;;
esac
```

One branch per detected env host.

## Templates

Located at `~/.claude/skills/api-tester-init/templates/`:

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
| `CLAUDE.local.md.template` | `CLAUDE.local.md` |
| `gitignore.template` | `.gitignore` |
| `skill/trigger-skill.md.template` | `~/.claude/skills/<project>-api-tester/SKILL.md` |

## Template variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | kebab-case slug | `my-dashboard` |
| `{{PROJECT_DISPLAY_NAME}}` | Title Case name | `My Dashboard` |
| `{{SOURCE_PROJECT_PATH}}` | absolute path to frontend | `/Users/me/projects/my-dashboard` |
| `{{SOURCE_PROJECT_NAME}}` | frontend folder name | `my-dashboard` |
| `{{API_TESTER_PATH}}` | absolute path to api-tester | `/Users/me/projects/my-dashboard-api-tester` |
| `{{DEFAULT_ENV}}` | default env name | `development` |
| `{{DEFAULT_BASE_URL}}` | default API URL | `http://localhost:3001` |
| `{{TODAY}}` | ISO date | `2026-05-12` |
| `{{ENVIRONMENTS_JSON}}` | JSON body for environments.json (no enclosing braces) | (multiline) |
| `{{ENVIRONMENT_TABLE}}` | markdown rows for env tables | (multiline) |
| `{{ENVIRONMENT_USE_CASES}}` | markdown rows for env use-cases | (multiline) |
| `{{ENV_HOST_TABLE}}` | markdown rows: host → env (for trigger skill) | (multiline) |
| `{{API_HOSTS_INLINE}}` | comma-separated hosts | `api-staging.example.com / api.example.com` |
| `{{URL_PATTERNS}}` | bash case-pattern lines for parse-curl.sh | (multiline) |
| `{{URL_DETECTION_RULES}}` | markdown bullet list of host→env | (multiline) |
| `{{TOKEN_FILES_LIST}}` | markdown bullet list of token file paths | (multiline) |

## Hard rules

- **NEVER overwrite an existing api-tester project** without confirmation. If `<parent>/<project>-api-tester/` exists, ask first.
- **NEVER overwrite an existing trigger skill** at `~/.claude/skills/<project>-api-tester/` without confirmation.
- **NEVER skip Step 5 (trigger skill generation)** — it's what makes the api-tester usable from any session. The old "trigger rule" pattern (`.claude/rules/api-tester-trigger.md` in the frontend) is deprecated; do not generate it.
- **NEVER skip writing `requests/`** in the project structure — `request.sh` writes there and the workflow assumes it exists.

## Error handling

| Scenario | Action |
|----------|--------|
| No `.env*` files | Ask user to manually input env name + base URL |
| No API URL in any `.env` | Ask user for the variable name |
| No `package.json` | Use folder name; ask user to confirm |
| Output directory exists | Ask: overwrite, merge, or choose new name |
| No write permission | Show error; suggest alternative location |
| Trigger skill folder exists | Ask before overwriting |

## Example session

```
User: /api-tester-init

Claude: Scanning project...
  Project: my-dashboard
  Path: /Users/me/projects/my-dashboard

  Found environments:
    - development: http://localhost:3001 (canExecute: true)
    - staging: https://api-staging.example.com (canExecute: true)
    - production: https://api.example.com (canExecute: false)

  Will create:
    - /Users/me/projects/my-dashboard-api-tester/
    - ~/.claude/skills/my-dashboard-api-tester/SKILL.md

  Confirm?

User: yes

Claude: Created scaffolding.
  - scripts/{request.sh,parse-curl.sh,generate-postman.sh,analyze.js,slice.js}
  - config/{environments.json,state.json}
  - reference/api-endpoints.json
  - .claude/rules/{curl-workflow.md,api-testing.md}
  - .gitignore, CLAUDE.local.md

  Trigger skill registered: /my-dashboard-api-tester

  Updated /Users/me/projects/my-dashboard/.claude/settings.local.json (additionalDirectories).

  Add "API Tester" section to source CLAUDE.local.md? [yes / skip]

User: yes

Claude: Done. Try /my-dashboard-api-tester and paste a curl.
```
