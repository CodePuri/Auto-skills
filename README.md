```text
     _   _   _ _____ ___    ____  _  _____ _     _     ____  
    / \ | | | |_   _/ _ \  / ___|| |/ /_ _| |   | |   / ___| 
   / _ \| | | | | || | | | \___ \| ' / | || |   | |   \___ \ 
  / ___ \ |_| | | || |_| |  ___) | . \ | || |___| |___ ___) |
 /_/   \_\___/  |_| \___/  |____/|_|\_\___|_____|_____|____/ 
```

# Auto Skills

**Plug-and-play agent intelligence.** Pre-seeded with 14 core skills across frontend, backend, QA, design, architecture, and planning. Auto-discovers and installs skills from skills.sh, skillsmp.com, and community registries.

<p align="center">
  <a href="#-quick-start"><b>Quick Start</b></a> •
  <a href="#-commands"><b>Commands</b></a> •
  <a href="#-how-it-works"><b>How It Works</b></a> •
  <a href="#-trigger-system"><b>Trigger System</b></a> •
  <a href="#-agent-integration"><b>Agent Integration</b></a> •
  <a href="#%EF%B8%8F-safety--trust"><b>Safety & Trust</b></a>
</p>

---

## 🚀 Quick Start

```bash
# Install globally (recommended)
npm install -g autoskills

# Or run via npx (no install needed)
npx autoskills doctor        # Health check
npx autoskills init           # First-run setup wizard
```

```bash
# Discover skills for a task
autoskills suggest --task "build a React dashboard with Node.js and PostgreSQL"

# List all cached/bundled skills
autoskills list

# Install a skill (safety-gated)
autoskills install <candidate-id> -y
```

---

## 📦 What is Auto Skills?

Auto Skills turns your AI agent into an expert across every domain. Instead of manually adding skills, it:

1. **Ships with 14 pre-bundled skills** covering frontend, backend, QA, design, architecture, and planning
2. **Auto-discovers** relevant skills from local `~/.codex/skills`, `~/.agents/skills`, and remote registries
3. **Scores and ranks** each skill by intent matching + trust signals (install count, owner reputation)
4. **Installs safely** — only auto-installs skills that pass a confidence threshold (score ≥ 70, trusted owner or ≥1K installs)
5. **Frontloads** relevant skill instructions into your agent's context when you use the `auto skills:` prefix

The result: your agent has instant access to best practices, patterns, and workflows for whatever task you throw at it.

---

## 📋 Commands

| Command | Description |
|---------|-------------|
| `autoskills` | Splash screen + help menu |
| `autoskills init` | First-run setup wizard — detects environment, registers bundled skills |
| `autoskills doctor` | Health check — Node version, paths, cache, connectivity |
| `autoskills suggest --task "..."` | Full pipeline: search → score → rank → display |
| `autoskills suggest --task "..." --json` | JSON output (for programmatic/agent use) |
| `autoskills suggest --task "..." --offline` | Skip remote queries, use cache only |
| `autoskills refresh` | Scan all sources and update cache |
| `autoskills refresh --network` | Same + inspect remote git sources |
| `autoskills install <id> [-y]` | Safety-gated install (needs `-y` to confirm) |
| `autoskills install <id> --dry-run` | Preview install without executing |
| `autoskills hook --task "..." [--json]` | Agent trigger check — returns JSON for AI consumption |
| `autoskills list` | Show all cached/bundled skills in a formatted table |
| `autoskills seed` | Register all 14 pre-bundled skills into cache |
| `autoskills clean` | Clear all cached data |
| `autoskills config` | Show current configuration (paths, trusted owners, thresholds) |

---

## 🔌 How It Works

```
               ┌──────────────────────┐
               │  User runs suggest    │
               │  --task "your task"   │
               └──────────┬───────────┘
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
   ┌────────────┐  ┌───────────┐  ┌──────────────┐
   │ PRE-BUNDLED│  │  LOCAL     │  │  REMOTE      │
   │ skills/    │  │~/.codex/  │  │ npx skills   │
   │ 14 skills  │  │~/.agent/  │  │ find         │
   │ (shipped)  │  │installed  │  │ skills.sh    │
   └─────┬──────┘  └─────┬─────┘  └──────┬───────┘
         │               │              │
         └───────────────┼──────────────┘
                         ▼
               ┌──────────────────┐
               │  MERGE + SCORE   │
               │  ↑               │
               │  Intent match    │
               │  ↑               │
               │  Trust signals   │
               │  ↑               │
               │  Install counts  │
               └────────┬─────────┘
                        ▼
               ┌──────────────────┐
               │  RANKED RESULTS  │
               │  Score ≥ 70:     │
               │  AUTO-INSTALL    │
               │  Score ≥ 50:     │
               │  RECOMMEND       │
               └────────┬─────────┘
                        ▼
               ┌──────────────────┐
               │  FRONTLOAD INTO  │
               │  AGENT CONTEXT   │
               └──────────────────┘
```

### Deep Dive: The Suggestion Pipeline

When you run `autoskills suggest --task "build a React dashboard with Node.js"`:

1. **Tokenization**: The task is split into keywords → `[react, dashboard, node, js, build]`
2. **Bundled search**: Scans all 14 pre-bundled SKILL.md files for keyword matches → finds react-patterns, node-api-design, css-mastery, database-patterns, system-design
3. **Local search**: Scans `~/.codex/skills/` and `~/.agents/skills/` for any already-installed skills
4. **Remote search (online only)**: Runs `npx skills find react dashboard node js build` to discover skills from the community registry
5. **Scoring**: Each candidate gets a score (0-100) based on:
   - Baseline: 50 (bundled), 42 (local), 25 (remote)
   - +16 per keyword match between task and skill name/description
   - +20 if from a trusted owner (vercel-labs, anthropics, microsoft, openai, codepuri)
   - +18 if ≥1000 installs, +10 if ≥100, +4 if >0
   - +12 if skill name appears verbatim in task text
6. **Ranking**: Top 12 candidates shown, sorted by score descending
7. **Output**: Beautiful vaporwave-styled terminal output or machine-readable JSON

---

## 🎯 Trigger System (AI Agent Integration)

Auto Skills supports a **three-tier trigger system** that lets AI agents automatically discover and load relevant skills.

### Tier 1: Prompt Prefix (Zero Config)

```
USER: auto skills: build a fullstack dashboard with React frontend, 
Node.js backend, PostgreSQL, and WebSocket real-time updates

AGENT: ✓ Detected "auto skills:" prefix
       ✓ Runs: autoskills suggest --task "build a fullstack dashboard..."
       ✓ Found 8 relevant skills
       ✓ Frontloading react-patterns, node-api-design, postgres-optimization...
       ✓ Let's build this!
```

Just start your prompt with `auto skills:` and your agent will automatically:
1. Run `autoskills hook --task "<rest>" --json` to discover relevant skills
2. Auto-install high-confidence matches
3. Load all relevant skill instructions into context

### Tier 2: Configuration File

Create `~/.config/autoskills/trigger.json`:

```json
{
  "alwaysSuggest": true,
  "minScore": 55,
  "allowedCategories": ["frontend", "backend", "architecture"]
}
```

When this file exists, the agent runs `autoskills hook` on every substantial prompt to proactively suggest relevant skills.

### Tier 3: Environment Variable

```bash
export AUTO_SKILLS=true
```

Same as Tier 2 but set globally. The agent detects this and runs `autoskills hook` proactively.

---

## ⚙️ Configuration

Default configuration is in `config/sources.json`:

```json
{
  "localPaths": [
    "~/.codex/skills",
    "~/.agents/skills"
  ],
  "gitSources": [
    "https://github.com/vercel-labs/skills"
  ],
  "trustedOwners": [
    "vercel-labs", "anthropics", "microsoft", "openai", "codepuri"
  ],
  "autoInstall": {
    "targetAgent": "codex",
    "minimumScore": 70,
    "minimumInstallsForPublic": 1000
  }
}
```

Override any setting by creating `~/.config/autoskills/sources.json` with your preferences.

---

## 🛡️ Safety & Trust

### Scoring Model

| Factor | Points |
|--------|--------|
| Bundled (shipped with package) | 50 base |
| Local (already installed) | 42 base |
| Remote (skills CLI, registry) | 25 base |
| Per intent keyword match | +16 |
| Trusted owner | +20 |
| ≥1000 installs | +18 |
| ≥100 installs | +10 |
| >0 installs | +4 |
| Skill name in task text | +12 |
| **Maximum** | **100** |

### Auto-Install Guardrails

- **Scores ≥ 70**: Candidate is eligible for auto-install IF also from trusted owner OR has ≥1000 installs
- **All other candidates**: Recommendation-only — user must approve explicitly
- **Install command validation**: Only `npx skills add <repo> --skill <name> -g -a codex -y` is accepted
- **`refresh` never installs** — only scans and caches
- **`hook` never installs** — only suggests
- **`-y` flag**: Only available for installs that pass the safety threshold

---

## 💻 Development

```bash
# Clone
git clone git@github.com:CodePuri/Auto-skills.git
cd Auto-skills

# Install dependencies
npm install

# Build
npm run build

# Test
node dist/cli.js doctor
node dist/cli.js seed
node dist/cli.js suggest --task "test" --json --offline
```

### Project Structure

```
src/
  cli.ts              # Entry point, command dispatch
  types.ts             # Shared TypeScript interfaces
  core/
    config.ts          # Configuration loading
    cache.ts           # Cache read/write
    scanner.ts         # SKILL.md file discovery
    ranker.ts          # Scoring, dedup, reranking
    registrar.ts       # Remote registry queries
    installer.ts       # Safe install execution
  ui/
    splash.ts          # Vaporwave ASCII art, help menu
    box.ts             # Boxen wrappers
    table.ts           # CLI table rendering
skills/
  catalog.json         # Index of all pre-bundled skills
  frontend/            # react-patterns, css-mastery, tailwind-architecture
  backend/             # node-api-design, database-patterns, auth-systems
  qa/                  # testing-strategies, code-review-excellence
  design/              # ui-ux-patterns, accessibility-first
  architecture/        # system-design, microservices-patterns
  planning/            # project-planning, technical-writing
config/
  sources.json         # Default configuration
```

---

## 🔗 Links

- **GitHub**: [github.com/CodePuri/Auto-skills](https://github.com/CodePuri/Auto-skills)
- **npm**: [autoskills](https://www.npmjs.com/package/autoskills)
- **Skills Registry**: [skills.sh](https://skills.sh)
- **Skills Marketplace**: [skillsmp.com](https://skillsmp.com)

---

## 📄 License

MIT © [CodePuri](https://github.com/CodePuri)
