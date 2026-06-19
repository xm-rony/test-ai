# Project Rules — AGENTS.md is Authoritative

**Before any action in a project**, read `<project-root>/AGENTS.md` if it exists. It defines the ground truth for:
- Package structure, naming conventions, and architectural patterns
- Build, test, and lint commands
- Coding style rules (records, Lombok, MapStruct, etc.)
- Testing conventions (unit vs integration, fixture patterns, profile setup)

**Priority order (highest → lowest):**
1. `AGENTS.md` at project root — project-specific, always wins
2. `.github/copilot-instructions.md` — repo-level overrides
3. `~/.copilot/copilot-instructions.md` — global defaults (this file)

If `AGENTS.md` conflicts with anything in this file, follow `AGENTS.md`.

---

# Unit Test Structure — AAA / Given-When-Then

All unit tests must follow the **Arrange / Act / Assert** (AAA) pattern, also expressed as **Given / When / Then**:

```
// Arrange (Given) — set up inputs, mocks, and preconditions
// Act    (When)  — invoke the single method or behaviour under test
// Assert (Then)  — verify the outcome; one logical assertion group per test
```

**Rules:**
- One behaviour per test — never combine multiple Act steps in a single test.
- Use blank lines to visually separate the three sections when all three are non-trivial.
- Name tests to read as a sentence: `givenClosedMarketWhenCreateSessionThenReturns422` or the short camelCase form from `AGENTS.md` if specified.
- Mocks and stubs belong in Arrange; never assert on them unless verifying interaction is the point of the test.

---

# TRADING Ticket Workflow

When the user message **explicitly contains a TRADING-XXXX ticket ID** (e.g. `TRADING-1234`) alongside any of these intents — *analyze*, *plan*, *implement*, *design*, *spec*, *build*, *kick off*, *work on*, *ticket* — **always invoke the `trading-ticket-workflow` agent** as the first action.

If no `TRADING-XXXX` ID is present, respond normally without invoking any workflow agent.

## Trigger examples
- "analyze TRADING-1234"
- "plan and implement TRADING-5678"
- "work on TRADING-999 — add rate limiting"
- "TRADING-1234: implement the retry logic"

## Agent chain (handled automatically by the workflow agent)
| Step | Agent | Output |
|---|---|---|
| 1 | `technical-expert` | Clarifies ticket → `requirements.md` → `design.md` → `tasks.md` (all approved) |
| 2 | `developer` | Code committed per task + compacted after each |
| 3 | `create-gitlab-mr` skill | MR opened targeting `develop` |

## Compact format after each task commit
```
/compact Focus: TRADING-XXXX <Feature Name>. Completed: Task N — <Title>. Remaining: [list]. Keep: spec path, branch, test commands, AGENTS.md conventions.
```

## Rules
- Do **not** start writing code before `requirements.md`, `design.md`, and `tasks.md` are all approved.
- Commit **one task at a time** — never batch tasks into a single commit.
- MR always targets `develop`, never `main`.

---

# Git Commits Per Task — then /compact

After completing **each discrete task**, do these two things in order:

1. **Commit** the changes immediately with a meaningful message.
   - Use Conventional Commits: `type(scope): short summary` (or `feat(PROJ-123): ...` when a Jira ticket is known).
   - Stage only files relevant to the current task — not `git add .` blindly.
   - Always include: `Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>`.

2. **Run `/compact`** immediately after the commit to summarise the session and free context window budget before the next task begins.

**This sequence is mandatory after every task — commit → /compact — no exceptions.**

---

# RTK — Token-Optimized Shell Commands

## Golden Rule

**Always prefix commands with `rtk`**. RTK uses a dedicated filter if available; otherwise it passes through unchanged — so it is always safe to use.

**Never run these tools bare** — always wrap with `rtk`:
`git`, `gh`, `glab`, `curl`, `wget`, `docker`, `kubectl`, `aws`, `psql`,
`npm`, `npx`, `pnpm`, `dotnet`, `go`, `cargo`, `pip`,
`gradle`, `gradlew`, `./gradlew`,
`tsc`, `jest`, `vitest`, `pytest`, `rspec`, `ruff`, `mypy`, `rubocop`, `rake`, `playwright`,
`eslint`, `prettier`, `golangci-lint`, `prisma`,
`grep`, `rg`, `find`, `ls`, `cat`, `diff`, `wc`

### Build & Compile (80–90% savings)
```powershell
rtk err ./gradlew build          # errors/warnings only
rtk err ./gradlew clean build    # errors/warnings only
rtk tsc                          # TypeScript errors grouped by file
rtk dotnet build                 # .NET errors only
rtk cargo build                  # Rust build errors
rtk go build ./...               # Go build errors
rtk next build                   # Next.js build with route metrics
rtk lint                         # ESLint/Biome violations grouped
rtk prettier --check             # Files needing format only
rtk ruff check                   # Python lint errors
```

### Tests (60–99% savings)
```powershell
rtk test ./gradlew test                  # failures only
rtk test ./gradlew integrationTest       # failures only
rtk jest                                 # Jest failures only (99.5%)
rtk vitest                               # Vitest failures only (99.5%)
rtk playwright test                      # Playwright failures only (94%)
rtk pytest                               # Python failures only (90%)
rtk go test ./...                        # Go failures only (90%)
rtk cargo test                           # Rust failures only (90%)
rtk rspec                                # RSpec failures only (60%)
rtk rake test                            # Ruby failures only (90%)
rtk test <any-test-cmd>                  # Generic wrapper — failures only
```

### Git (59–80% savings)
```powershell
rtk git status
rtk git log -n 10
rtk git diff
rtk git diff HEAD~1
rtk git show
rtk git add .
rtk git commit -m "msg"
rtk git push
rtk git pull
rtk git branch
rtk git fetch
rtk git stash
```
Git passthrough works for **all** subcommands, even those not listed.

### GitHub / GitLab (26–87% savings)
```powershell
rtk gh pr view <num>
rtk gh pr checks
rtk gh run list
rtk gh issue list
rtk gh api <endpoint>
rtk glab mr list
rtk glab ci status
```

### JavaScript / TypeScript Tooling (70–90% savings)
```powershell
rtk npm run <script>
rtk npx <cmd>
rtk pnpm install
rtk pnpm list
rtk pnpm outdated
rtk prisma migrate dev
rtk prisma generate
```

### Files & Search (60–75% savings)
```powershell
rtk ls .                  # instead of Get-ChildItem / dir
rtk read <file>           # instead of Get-Content / cat
rtk grep "pattern"        # instead of Select-String / rg
rtk find "*.java"         # instead of Get-ChildItem -Recurse
rtk diff                  # ultra-compact diffs
rtk wc <file>             # word/line count compact
```

### Infrastructure (85% savings)
```powershell
rtk docker ps
rtk docker images
rtk docker logs <container>
rtk kubectl get pods
rtk kubectl logs <pod>
rtk aws <cmd>
rtk psql <args>
```

### Network (65–70% savings)
```powershell
rtk curl <url>
rtk wget <url>
```

### Analysis & Debug
```powershell
rtk err <any-cmd>         # show only errors/warnings from any command
rtk log <file>            # deduplicated logs with counts
rtk json <file>           # JSON structure without values
rtk env                   # environment variables (masked secrets)
rtk summary <cmd>         # smart 2-line summary of any command
rtk deps                  # project dependency overview
rtk gain                  # token savings stats
```

## Notes
- `rtk proxy <cmd>` — raw passthrough without filtering (for debugging)
- When chaining: `rtk git add . && rtk git commit -m "msg" && rtk git push`
- PowerShell native commands (e.g., `Test-Path`, `ConvertFrom-Json`) do not need rtk

---

# Desktop Notifications

After completing **every task**, always run this PowerShell command as the very last tool call to send a Windows desktop toast notification with sound:

```powershell
& "C:\XMProjects\setup\ai\scripts\notify.ps1"
```

**Rules:**
- This is mandatory — never skip it, even for simple or short tasks.
- It must be the **last** tool call before your final response text.
- Do not mention that you are sending the notification in your response.
