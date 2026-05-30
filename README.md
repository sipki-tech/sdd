# Spec-Driven Dev

A universal skill package for AI coding assistants that enforces a 6-phase development pipeline:

```
Explore → [APPROVE] → Requirements → [APPROVE] → Design → [APPROVE] → Task Plan → [APPROVE] → Implementation → [APPROVE] → Review → [APPROVE] → Done
```

No API keys. No dependencies. Works in any IDE with AI agent support.

Designed for a **single project or monorepo**. Not intended for features spanning multiple independent repositories.

## Why

AI coding assistants are great at writing code, but they often skip the thinking phase. They jump straight to implementation without understanding what needs to be built, why, or what the constraints are.

Spec-Driven Dev forces a structured workflow:
1. **Explore** — investigate the problem space, compare 2–4 approaches, recommend direction. Output: exploration document with options, risks, and scope boundaries.
2. **Requirements** — structured 4-layer interview (context → scope → constraints → verification), then generate formal requirements. Output: requirements document with WHEN/SHALL grammar.
3. **Design** — architect the solution with Mermaid diagrams, ADRs, correctness properties, and testing strategy. Output: design document with traceability to requirements.
4. **Task Plan** — decompose design into TDD tasks (RED/GREEN/CODE/VERIFY/GATE) with coverage matrix. Output: implementation plan with full requirements traceability (no code yet).
5. **Implementation** — execute the task plan: write tests, write code, run suite after each task. Output: implementation report with task completion checklist.
6. **Review** — review written code: change set, requirements traceability, design conformance, code quality, security scan. Output: review document with PASS/NEEDS_CHANGES/BLOCK verdict.

Each phase produces a document. Each transition requires explicit human approval. No skipping.

## Installation

Install via [skills.sh](https://skills.sh):

```bash
npx skills add sipki-tech/sdd
```

The CLI auto-detects your installed agents (GitHub Copilot, Claude Code, Cursor, Codex, Windsurf, Cline, and [40+ others](https://skills.sh)) and symlinks the skill into their config directories.

### Install Options

```bash
# Install globally (available across all projects)
npx skills add sipki-tech/sdd -g

# Install to a specific agent
npx skills add sipki-tech/sdd -a github-copilot
npx skills add sipki-tech/sdd -a claude-code
npx skills add sipki-tech/sdd -a cursor

# CI-friendly (no prompts)
npx skills add sipki-tech/sdd --all -y

# Full GitHub URL also works
npx skills add https://github.com/sipki-tech/sdd
```

### Manual Installation

If you prefer not to use `npx`, clone the repo directly into your project:

```bash
git clone https://github.com/sipki-tech/sdd.git /tmp/sdd
cp -r /tmp/sdd/skills/sdd skills/sdd
rm -rf /tmp/sdd
```

### Verify

```bash
npx skills list
```

## Quick Start

Tell your AI assistant:

> "I want to add user authentication with OAuth2"

The agent will automatically pick up the pipeline and start with the exploration phase.

## How It Works

### Project Configuration

Customize AI behavior for your project by creating `.spec/config.yaml`:

> **Note:** The parser supports **flat `key: value` pairs only** — nested YAML, multi-line blocks (`|`, `>`), and arrays (`- item`) are NOT supported. See [`.spec/config.yaml.example`](.spec/config.yaml.example) for a complete template.

```yaml
# .spec/config.yaml

# Project context (single line)
context: Go 1.23 monorepo, PostgreSQL, gRPC, go test + testify, make build

# Phase-specific rules (flat key: value)
rules.explore: Focus on existing API surface before proposing new endpoints
rules.requirements: All REQ should use gRPC error codes, not HTTP statuses
rules.design: ADR must consider protobuf backward compatibility
rules.task-plan: Test command: go test ./...; Build command: make build
rules.implementation: Run tests after each task before marking it done
rules.review: Always check protobuf backward compatibility
rules.docs: Always include Mermaid diagrams in ARCHITECTURE.md

# Optional: test style cascade overrides
test_skill: my-test-skill        # Tier 1: delegate test generation to this skill
test_reference: "**/*_test.go"   # Tier 2: use these files as test style reference

# Optional: project documentation directory
docs_dir: .spec                  # Default. Change to customize (e.g., .docs, docs/)
doc_freshness_days: 30           # Days before a generated doc is considered stale (default: 30)
```

- **`context`** is injected into ALL phases — the agent knows your stack before asking questions.
- **`rules.<phase>`** adds phase-specific rules on top of the template defaults.
- **`test_skill`** (optional) — name of an installed skill for test generation. When set, Design and Implementation phases delegate test specification to this skill instead of writing test tasks directly.
- **`test_reference`** (optional) — glob or file paths pointing to representative test files. When set, the agent uses these as the style reference for all generated tests. When omitted, the agent auto-discovers adjacent tests.
- **`docs_dir`** (optional) — directory for project documentation, default: `.spec`. The agent reads and writes project docs here.
- **`doc_freshness_days`** (optional) — number of days after which a generated doc is considered stale, default: `30`. Used by `docs-check` to flag outdated files.
- **`rules.docs`** (optional) — rules for documentation generation (e.g., skip irrelevant docs, require diagrams).

Phase-specific rule keys: `rules.explore`, `rules.requirements`, `rules.design`, `rules.task-plan`, `rules.implementation`, `rules.review`, `rules.docs`.

### File Structure

```
skills/sdd/              ← skill package (installed by skills.sh)
├── SKILL.md                         ← orchestrator (skills.sh entry point)
├── templates/
│   ├── explore.md                   ← phase 1 prompt
│   ├── requirements.md              ← phase 2 prompt
│   ├── design.md                    ← phase 3 prompt
│   ├── task-plan.md                 ← phase 4 prompt
│   ├── implementation.md            ← phase 5 prompt
│   ├── review.md                    ← phase 6 prompt
│   └── docs/                        ← documentation generation templates
│       ├── README.md                ← manifest (lists all doc templates)
│       ├── bootstrap.md             ← generates README.md, agent-rules.md
│       ├── agents-index.md          ← generates AGENTS.md
│       ├── core.md                  ← generates ARCHITECTURE, PACKAGES, DOMAIN, CODE_STYLE
│       ├── development.md           ← generates TOOLS, TESTING, FILES
│       ├── errors.md                ← generates ERRORS.md
│       ├── auth.md                  ← generates AUTH.md / OAUTH.md
│       ├── database.md              ← generates DATABASE.md
│       ├── api.md                   ← generates API.md
│       ├── deployment.md            ← generates DEPLOYMENT.md
│       ├── infrastructure.md        ← generates per-component infra docs
│       ├── clients.md               ← generates CLIENTS.md + per-client docs
│       ├── security.md              ← generates SECURITY.md
│       ├── feature-flags.md         ← generates FEATURE_FLAGS.md
│       ├── background-jobs.md       ← generates BACKGROUND_JOBS.md
│       ├── cli.md                   ← generates CLI.md
│       ├── state-management.md      ← generates STATE.md
│       ├── events.md                ← generates EVENTS.md
│       ├── components.md            ← generates COMPONENTS.md
│       └── routing.md               ← generates ROUTING.md
└── scripts/
    └── pipeline.sh                  ← state machine (POSIX sh, zero deps)

.spec/                               ← project-local (committed to git)
├── config.yaml                      ← project context & rules (opt-in)
├── features/                        ← per-feature pipeline artifacts
│   ├── grpc-streaming/              ← example completed feature
│   │   ├── pipeline.json            ← pipeline state (for agents)
│   │   ├── pipeline.kv              ← internal KV store
│   │   ├── explore.md               ← phase 1 artifact
│   │   ├── requirements.md          ← phase 2 artifact
│   │   ├── design.md                ← phase 3 artifact
│   │   ├── task-plan.md             ← phase 4 artifact
│   │   ├── implementation.md        ← phase 5 artifact
│   │   ├── review.md                ← phase 6 artifact
│   │   ├── revisions/               ← artifact revision history
│   │   └── approved/                ← approved snapshots
│   └── oauth-login/                 ← another feature (in progress or done)
│       └── ...
├── README.md                        ← documentation index
├── ARCHITECTURE.md                  ← architecture overview
├── PACKAGES.md                      ← package reference
├── DOMAIN.md                        ← domain model
├── CODE_STYLE.md                    ← code conventions
├── ERRORS.md                        ← error handling & error catalog
├── TOOLS.md                         ← commands & tooling
├── TESTING.md                       ← testing conventions
├── DATABASE.md                      ← database schema & migrations
├── API.md                           ← API endpoints & conventions
├── DEPLOYMENT.md                    ← deployment pipeline & environments
├── SECURITY.md                      ← security audit & OWASP mapping
└── ...                              ← domain-specific docs (AUTH, CLIENTS, etc.)
```

### Pipeline Commands

```bash
P="skills/sdd/scripts/pipeline.sh"

# Core flow
sh $P help                            # Show full usage
sh $P init my-feature                 # Start a new pipeline
sh $P status                          # Show current phase, artifacts, progress
sh $P artifact [path]                 # Register output artifact for current phase
sh $P approve                         # Advance to next phase (after user approval)

# Implementation tasks (implementation phase)
sh $P task-init T-1 T-2 ...           # Register task set + create tasks/T-N.md evidence stubs
sh $P task T-3 [done|wip|blocked]     # Set per-task status (default: done)
sh $P tasks                           # List tasks and status
sh $P task-next                       # Print next actionable task (id + evidence path)
sh $P task-reset                      # Clear the task set (keeps evidence files)

# Inspect / manage
sh $P history                         # Show all features and their status
sh $P revisions [ph]                  # Show revision history (current or specified phase)
sh $P abandon [feature]               # Abandon an active pipeline
sh $P inject <phase> <path>           # Inject a pre-written artifact, skip to phase

# Config & docs
sh $P config-check                    # Validate .spec/config.yaml keys and types
sh $P doctor                          # Diagnose environment (tools, templates, .spec, git)
sh $P docs-check                      # Check project documentation status (JSON)
sh $P docs-init [--all|--update|<t>]  # Build a standalone docs queue
sh $P docs-next                       # Print next pending docs template
sh $P docs-done <name>                # Mark a docs template complete
sh $P docs-status                     # Docs queue progress (JSON)
sh $P docs-reset                      # Clear the docs queue
sh $P version                         # Show version

# When multiple pipelines are active simultaneously:
sh $P --feature auth-flow status      # Operate on a specific feature
sh $P --feature payment approve
```

### Typical Session

```
You: "I want to add gRPC streaming to my service"

Agent: [reads SKILL.md → runs pipeline.sh init grpc-streaming]
Agent: [reads templates/explore.md → investigates codebase]
Agent: "Let me explore the problem space first. I see you currently use
        unary gRPC calls in service X. Here are 3 approaches:
        1. Server streaming  2. Bidirectional  3. SSE fallback
        I recommend #1 because..."
Agent: "Exploration document ready. Approve?"

You: "Approve"

Agent: [runs pipeline.sh approve → advances to requirements]
Agent: [reads templates/requirements.md → starts interview]
Agent: "Now let me capture the formal requirements.
        1. Which service is this for?
        2. What's the current communication pattern?
        3. Who are the consumers of this API?"

You: [answers questions]

Agent: [generates requirements document with WHEN/SHALL grammar]
Agent: "Requirements document is ready. Approve?"

You: "Approve"

Agent: [→ design → task plan, same pattern]
Agent: "Task plan approved. Now I'll implement the code."

Agent: [reads templates/implementation.md → executes the TDD plan]
Agent: [writes tests, writes code, marks tasks done with [x]]

Agent: "All tasks completed. Implementation report ready. Approve?"

You: "Approve"

Agent: [→ review phase]
Agent: "Code review complete. Verdict: NEEDS_CHANGES.
        Found 2 issues: F-1 (major) missing test for REQ-1.2,
        F-2 (minor) naming inconsistency.
        What would you like me to do?"

You: "Fix both"

Agent: [fixes code, re-reviews]
Agent: "All findings resolved. Verdict: PASS. Approve?"

You: "Approve"

Agent: "Pipeline complete!"
Agent: "This feature added new packages. Update .spec/PACKAGES.md?"

You: "Update docs"

Agent: [regenerates affected documentation]
```

## Phase Details

### Phase 1: Explore

The agent investigates the problem space before committing to requirements:
- Reads existing codebase to understand current state
- Identifies constraints, risks, and dependencies
- Compares 2–4 realistic approaches with trade-offs
- Recommends a direction with suggested scope boundaries

Output: exploration document with Intent, Investigation, Build Tooling, Options Considered, Constraints & Risks, Recommended Direction, Scope Boundaries, and Assumptions & Open Questions.

### Phase 2: Requirements

The agent conducts a structured interview through 4 layers:
1. **Context & Motivation** — what, why, who's affected
2. **Scope Boundaries** — what changes, what must NOT change
3. **Constraints & Edge Cases** — errors, defaults, conflicts
4. **Verification** — how to prove it works

Output: formal requirements document with overview, glossary, requirements using **WHEN/SHALL grammar**, verification commands, and open design questions.

### Phase 3: Design

Takes the requirements document and produces:
- **Architecture** with Mermaid diagrams (color-coded: new/modified/existing)
- **Components & Interfaces** — affected files + files NOT requiring changes
- **Key Decisions (ADR)** — choices between alternatives with rationale
- **Correctness Properties** — formal "For all X, Y must hold" statements
- **Testing Strategy** — test style source, project commands, unit and property-based test specifications

### Phase 4: Task Plan

Takes both documents and produces a **TDD implementation plan**:
- Exploration tests (RED) — prove the problem exists
- Preservation tests (GREEN) — lock in correct behavior
- Implementation tasks — atomic, one file per subtask
- Re-tests — confirm fix, no regressions
- Checkpoints — integration verification

Every task is traceable: `Requirements X.Y → Task N → Correctness Property K`.

### Phase 5: Implementation

The agent executes the approved task plan:
- Writes real tests and real production code for **every** task
- Runs the test suite after each task; iterates until green
- Marks each completed task with `[x]` in the implementation report
- Does NOT create new tasks or modify the plan — only executes
- Final verification: all tests pass, build succeeds, lint is clean

Output: implementation report with task checklist showing what was done.

### Phase 6: Review

After the agent implements the TDD plan, it reviews the **written code**:
- **Change Set Discovery** — git diff from the base commit, cross-reference with the plan
- **Requirements Traceability** — every requirement mapped to test and code
- **Design Conformance** — architectural boundaries, data models, API contracts, correctness properties
- **Code Quality** — naming, dead code, scope creep, test quality
- **Security Scan** — input validation, auth, injection, secrets (scoped to changed files)

Verdict: `PASS` (no critical/major findings), `NEEDS_CHANGES` (major findings), or `BLOCK` (critical findings).
If not `PASS`, the agent presents findings and recommendations to the user. The user decides which findings to fix. This follows the same human-in-the-loop model as all other phases.

## Self-Documenting Mechanic

The skill includes a self-documenting mechanic that keeps project documentation (`.spec/`) in sync with code changes.

### How it works

**Before the pipeline** (soft gate):
- When starting a new feature, the agent checks if `.spec/` exists
- If missing, it suggests generating documentation first: *"Project docs not found. Say 'generate docs' or 'skip'."*
- If present, it reads the docs as context for all phases, reducing codebase scan time
- This is a suggestion, not a blocker — the pipeline works without `.spec/`

**After the pipeline** (targeted update):
- When the feature pipeline completes, the agent analyzes which files were changed
- It suggests updating only the affected documentation (e.g., new package → update `PACKAGES.md`)
- You can accept or skip the update

### Documentation templates

Templates for generating docs live in `skills/sdd/templates/docs/`. Each template generates specific doc files:

| Template | Stage | Generates |
|----------|-------|----------|
| `bootstrap.md` | Bootstrap | `README.md`, `agent-rules.md` — project index and agent rules |
| `agents-index.md` | Bootstrap | `AGENTS.md` — entry point for agents |
| `core.md` | Core | `ARCHITECTURE.md`, `PACKAGES.md`, `DOMAIN.md`, `CODE_STYLE.md` |
| `development.md` | Core | `TOOLS.md`, `TESTING.md`, `FILES.md` |
| `errors.md` | Core | `ERRORS.md` — error architecture & business error catalog |
| `auth.md` | Domain | `AUTH.md` / `OAUTH.md` — authentication & authorization |
| `database.md` | Domain | `DATABASE.md` — schema, migrations, query patterns |
| `api.md` | Domain | `API.md` — endpoint reference, middleware, error format |
| `deployment.md` | Domain | `DEPLOYMENT.md` — environments, CI/CD, rollout, health checks |
| `infrastructure.md` | Domain | Per-component infra docs (`OBSERVABILITY.md`, `REDIS.md`, etc.) |
| `clients.md` | Domain | `CLIENTS.md` + per-client docs (`FRONTEND.md`, `TELEGRAM.md`, etc.) |
| `security.md` | Domain | `SECURITY.md` — security audit, OWASP mapping, secrets management |
| `feature-flags.md` | Domain | `FEATURE_FLAGS.md` — flag inventory, lifecycle, rollout, cleanup |
| `background-jobs.md` | Domain | `BACKGROUND_JOBS.md` — job inventory, retry/DLQ, concurrency, scaling |
| `cli.md` | Domain | `CLI.md` — commands, flags, config, exit codes |
| `state-management.md` | Domain | `STATE.md` — state stores (Redux, Zustand, BLoC, Pinia, MobX) |
| `events.md` | Domain | `EVENTS.md` — event-driven patterns (Kafka, NATS, pub/sub) |
| `components.md` | Domain | `COMPONENTS.md` — UI component library / design system |
| `routing.md` | Domain | `ROUTING.md` — client-side routing, navigation guards |

To add a new documentation type, create a template file in `templates/docs/` and add it to the manifest (`templates/docs/README.md`).

## Architecture Decisions

1. **POSIX sh over Python** — guaranteed to be available everywhere, even in minimal containers
2. **KV file + JSON mirror** — KV for simple shell manipulation, JSON for agents to parse
3. **skills.sh distribution** — standard skill packaging, no custom installer needed
4. **No auto-approve** — human in the loop is the whole point
5. **Persistent artifacts in `.spec/features/`** — committed to git, creating a permanent record of decisions
6. **Per-feature directories** — each feature gets its own directory with all artifacts, revisions, and state
7. **Code review as a phase** — code is verified against specs before the pipeline completes
8. **Review as information** — review presents findings and verdict to the user for decision, consistent with the human-in-the-loop philosophy across all 6 phases
9. **Task Plan / Implementation split** — planning (Phase 4) and execution (Phase 5) are separate phases with separate approval gates, ensuring the plan is reviewed before any code is written

## Requirements

- POSIX-compatible shell (`sh`, `bash`, `zsh`, `dash`)
- `git`, `date`, `grep`, `sed`, `mkdir` — standard Unix utilities
- No Python, Node.js, or other runtime required

## License

MIT
