# central-workflows

> Centralized AI-powered CI pipeline for all/required repositories under `PranavPuranik-hub`.

---

## What This Does

Every time a Pull Request is opened in any connected repository, this system automatically:

- 🔍 **Detects** what languages changed (Python, JavaScript, HTML/CSS)
- ✅ **Lints** the code (ESLint, Ruff, Flake8, HTMLHint, Stylelint)
- 🧪 **Runs tests** (Pytest, Jest) if they exist
- 🔒 **Scans for security vulnerabilities** (GitHub CodeQL)
- 🤖 **Generates an AI code review** (Google Gemini) and posts it as a PR comment
- 📧 **Emails the review** to team members with the PR link

All logic lives here. Client repos only need one 5-line file.

---

## Repository Structure

```
central-workflows/
├── .github/
│   └── workflows/
│       └── reusable-pr-review.yml   ← All CI logic
└── configs/
    ├── .eslintrc.json               ← JavaScript linting rules
    ├── .flake8                      ← Python linting rules
    ├── ruff.toml                    ← Python linting rules
    ├── .htmlhintrc                  ← HTML linting rules
    └── .stylelintrc.json            ← CSS linting rules
```

---

## How to Connect a New Repository

### Step 1 — Add `caller.yml` to the client repo

Create `.github/workflows/caller.yml` with this content:

```yaml
name: PR Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  review:
    uses: PranavPuranik-hub/central-workflows/.github/workflows/reusable-pr-review.yml@main
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
      MAIL_USERNAME: ${{ secrets.MAIL_USERNAME }}
      MAIL_PASSWORD: ${{ secrets.MAIL_PASSWORD }}
```

### Step 2 — Add secrets to the client repo

Go to: `client-repo → Settings → Secrets and variables → Actions`

| Secret | Value |
|---|---|
| `GEMINI_API_KEY` | Your Google Gemini API key |
| `MAIL_USERNAME` | Gmail address to send from |
| `MAIL_PASSWORD` | Gmail App Password |

### Step 3 — Enable Actions permissions

Go to: `client-repo → Settings → Actions → General`

Select: **"Allow all actions and reusable workflows"**

Also set **Workflow permissions** to: **"Read and write permissions"**

Check: **"Allow GitHub Actions to create and approve pull requests"**

That's it. Create a PR and the review runs automatically.

---

## How It Works (Architecture)

```
PR opened in client-repo
        ↓
caller.yml triggers (5-line file in client repo)
        ↓
Calls reusable-pr-review.yml from this repo
        ↓
┌─────────────────────────────────────────┐
│         reusable-pr-review.yml          │
│                                         │
│  Job 1: detect                          │
│  → Diff base..head                      │
│  → Detect Python/JS/HTML/CSS changes    │
│                                         │
│  Job 2: quality (parallel matrix)       │
│  → Fetch configs from configs/          │
│  → Run linters for each language        │
│  → Run tests if they exist              │
│                                         │
│  Job 3: security                        │
│  → Run GitHub CodeQL                    │
│  → Upload results to Security tab       │
│                                         │
│  Job 4: ai-summary                      │
│  → Generate git diff                    │
│  → Send to Gemini API                   │
│  → Post review as PR comment            │
│  → Email team members                   │
└─────────────────────────────────────────┘
```

---

## Supported Languages

| Language | Linter(s) | Test Runner | Security |
|---|---|---|---|
| Python | Ruff, Flake8 | Pytest | CodeQL |
| JavaScript / React | ESLint | Jest | CodeQL |
| HTML | HTMLHint | — | — |
| CSS / SCSS | Stylelint | — | — |

---

## Secrets Required

### In `central-workflows` (this repo)
| Secret | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini AI API access |
| `MAIL_USERNAME` | Gmail sender address |
| `MAIL_PASSWORD` | Gmail App Password |

### In each client repo
Same 3 secrets as above — GitHub requires secrets to be present in the repo where the workflow runs.

---

## Updating the CI Pipeline

To update linting rules, AI prompt, or any workflow logic:

1. Edit the relevant file in this repo
2. Commit and push to `main`
3. All connected repos use the new version on their next PR

No changes needed in client repos.

---

## Notification Email

Every PR review is emailed to the addresses configured in `reusable-pr-review.yml`:

```yaml
to: your-email@example.com,teammate@example.com
```

To add or remove recipients, edit this line and push to main.

---

## Technologies Used

| Tool | Purpose |
|---|---|
| GitHub Actions | CI platform — runs the pipeline |
| Google Gemini API | AI-powered code review generation |
| GitHub CodeQL | Security vulnerability scanning |
| ESLint | JavaScript/JSX linting |
| Ruff | Python linting (fast, Rust-based) |
| Flake8 | Python linting (traditional) |
| Pytest | Python test runner |
| HTMLHint | HTML validation |
| Stylelint | CSS/SCSS validation |
| dawidd6/action-send-mail | Email delivery via SMTP |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Startup failure | Enable "Allow all actions and reusable workflows" in client repo settings |
| Secrets not found | Add GEMINI_API_KEY, MAIL_USERNAME, MAIL_PASSWORD to client repo secrets |
| AI review not posting | Enable "Read and write permissions" + "Allow GitHub Actions to create pull requests" |
| Gemini 503 error | Temporary overload — workflow retries automatically up to 8 times |
| `package.json` not found | Client repo needs a package.json for JS linting to work |
| No test results | Add a `tests/` folder with test files for Pytest or a `test` script in package.json for Jest |

---

## License

MIT — free to use and adapt.
