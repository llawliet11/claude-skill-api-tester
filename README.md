# Claude Skill: API Tester Init

A Claude Code skill that scaffolds a standalone api-tester project for any frontend application AND registers a project-specific trigger skill for future sessions.

## What it does

1. Auto-detects project name (from `package.json`) and environments (from `.env*` files).
2. Scaffolds `<parent>/<project>-api-tester/` with scripts, config, rules, and folder structure.
3. Registers a per-project trigger skill at `~/.claude/skills/<project>-api-tester/SKILL.md` that orchestrates the curl-paste workflow in any future session.
4. Updates the frontend project's `.claude/settings.local.json` to grant access to the api-tester path.
5. Optionally adds an "API Tester" section to the frontend's `CLAUDE.local.md`.

Both skills are configured with `disable-model-invocation: true` — they only run on explicit user invocation, never auto-trigger on a raw curl paste.

## Reference implementation

The canonical example shipped with this dotfiles repo:

- Trigger skill: `~/.claude/skills/cisgenx-api-tester/SKILL.md`
- Generated project: `/Users/niafam/WORK/CISGENICS/web-cisgenx-worktree/cisgenics-api-tester/`

Read those to see the end-state this skill produces.

## Usage

```
cd ~/projects/my-dashboard
claude
> /api-tester-init
```

The skill detects the project, asks for confirmation once, then scaffolds everything.

After init, in any future session:

```
/my-dashboard-api-tester
<paste curl from browser DevTools>
```

The trigger skill parses the curl, saves the bearer to `config/tokens/<env>.json`, runs `scripts/request.sh`, and persists both the request body and response under `requests/<env>/` and `responses/<env>/`.

## Generated project structure

```
<project>-api-tester/
├── CLAUDE.local.md
├── .claude/
│   └── rules/
│       ├── curl-workflow.md
│       └── api-testing.md
├── config/
│   ├── environments.json         # baseUrl + canExecute per env
│   ├── state.json                # activeEnvironment + lastRequest (git-ignored)
│   └── tokens/                   # bearer per env (git-ignored)
├── reference/
│   └── api-endpoints.json
├── scripts/
│   ├── request.sh                # persists req + res; warns on prod mutating
│   ├── parse-curl.sh
│   ├── generate-postman.sh
│   ├── analyze.js                # large-response summariser
│   └── slice.js                  # JS-filter slicer
├── requests/<env>/               # body + _meta per call (git-ignored)
├── responses/
│   ├── <env>/                    # pretty JSON responses (git-ignored)
│   └── sliced/                   # filtered subsets (reusable)
├── postman/
└── .gitignore
```

## Environment detection

Scans these files:

| File | Default env name |
|------|------------------|
| `.env` | development |
| `.env.local` | local |
| `.env.development` | development |
| `.env.staging` | staging |
| `.env.preprod` | preprod |
| `.env.production` | production |
| `.env.prod` | production |

Looks for these variable names:

- `API_URL`, `BASE_URL`, `BACKEND_URL`
- `VITE_API_URL`, `VITE_BASE_URL`
- `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_BASE_URL`
- `REACT_APP_API_URL`
- `NUXT_PUBLIC_API_URL`

## Workflow after setup

1. Copy curl from browser DevTools (Network tab → Right click → Copy as cURL).
2. In Claude Code: `/<project>-api-tester` and paste.
3. Trigger skill saves the token, runs `request.sh`, surfaces status + file paths.
4. For responses `>= 15KB`, the skill runs `analyze.js`, asks for filter criteria, runs `slice.js`, and reads the sliced result.

## Safety

- `disable-model-invocation: true` on both skills — only explicit `/<skill>` invocation.
- Production POST/PUT/DELETE/PATCH always warned (script + skill); never silently mutated.
- Tokens and responses are git-ignored.
- Production response read is fine — only mutating verbs need confirmation.

## Requirements

- Claude Code CLI
- `jq` for JSON processing
- `curl` for API requests
- `node` for `analyze.js` / `slice.js`
