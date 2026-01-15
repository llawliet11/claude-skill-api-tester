# Claude Skill: API Tester Init

A Claude Code skill that initializes API tester projects for frontend applications.

## Features

- Auto-detect project name from `package.json`
- Auto-detect environments from `.env*` files
- Generate curl paste workflow scripts
- Generate Postman collection from discovered endpoints
- Token management per environment
- Production safety (requires confirmation)

## Installation

Navigate to your Claude skills directory and clone this repository:

```bash
cd ~/.claude/skills
git clone https://github.com/llawliet11/claude-skill-api-tester.git api-tester-init
```

## Usage

1. Open your frontend project in Claude Code
2. Run the skill command:

```
/api-tester-init
```

3. Claude will:
   - Detect your project name from `package.json`
   - Scan `.env*` files for API URLs
   - Ask for confirmation
   - Create an api-tester project as a sibling directory

## Example

```
$ cd ~/projects/my-dashboard
$ claude
> /api-tester-init

Detected:
  Project: my-dashboard (from package.json)
  
Found environments:
  - development: http://localhost:3001 (auto-execute: YES)
  - staging: https://api-staging.example.com (auto-execute: YES)
  - production: https://api.example.com (auto-execute: NO)

API tester will be created at:
  ~/projects/my-dashboard-api-tester

[Confirm] [Edit]
```

## Generated Project Structure

```
<project-name>-api-tester/
├── .claude/
│   └── rules/
│       ├── curl-workflow.md      # Curl paste handling
│       └── api-testing.md        # API testing guidelines
├── config/
│   ├── environments.json         # Environment configs
│   └── tokens/                   # JWT tokens (git-ignored)
├── reference/
│   └── api-endpoints.json        # All known API endpoints
├── scripts/
│   ├── request.sh                # API request script
│   ├── parse-curl.sh             # Curl parser
│   └── generate-postman.sh       # Postman generator
├── responses/                    # Saved responses (git-ignored)
├── postman/                      # Generated Postman files
├── .gitignore
└── CLAUDE.local.md
```

## Workflow After Setup

### 1. Paste Curl from Browser

Copy curl from browser DevTools (Network tab > Right click > Copy as cURL) and paste to Claude. The skill will:

- Extract JWT token
- Detect environment from URL
- Execute request (staging) or ask confirmation (production)
- Save response

### 2. Test Endpoints

```bash
./scripts/request.sh GET /api/user/profile
./scripts/request.sh POST /api/endpoint '{"key": "value"}'
```

### 3. Generate Postman Collection

```bash
./scripts/generate-postman.sh
```

## Environment Detection

The skill scans these files for API URLs:

| File | Environment |
|------|-------------|
| `.env` | development |
| `.env.local` | local |
| `.env.development` | development |
| `.env.staging` | staging |
| `.env.production` | production |
| `.env.prod` | production |

And looks for these variable names:

- `API_URL`
- `VITE_API_URL`
- `NEXT_PUBLIC_API_URL`
- `REACT_APP_API_URL`
- `NUXT_PUBLIC_API_URL`
- `BASE_URL`
- `BACKEND_URL`

## Safety

- **Production environments** always require explicit confirmation before executing requests
- Tokens are stored locally and git-ignored
- Responses may contain sensitive data and are git-ignored

## Requirements

- Claude Code CLI
- `jq` for JSON processing
- `curl` for API requests

## License

MIT
