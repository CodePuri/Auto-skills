---
name: auto-skills
description: Use when the user asks to discover, refresh, recommend, install, or automate agent skills; identify missing capabilities for a task; or decide whether a specialized skill would improve current or future work. Uses the Auto Skills CLI with strict trust checks and mandatory user approval for installation.
---

# Auto Skills

Use the Auto Skills CLI to aggregate local and remote agent skills, infer task intent, rank candidates, and safely recommend skills that may improve a task.

## Locate the CLI

Before running commands, locate the CLI in this order:

1. If `AUTO_SKILLS_CLI` is set, run `node "$AUTO_SKILLS_CLI" ...`.
2. If `skill-aggregator` is on `PATH`, run `skill-aggregator ...`.
3. If a local checkout exists, run `node <repo>/dist/cli.js ...`.
   Common checkout paths include `~/Code/auto-skills`, `~/Desktop/Code/auto-skills`, and the current repository root.

If the CLI is missing, tell the user to install it:

```bash
git clone https://github.com/CodePuri/Auto-skills.git ~/Code/auto-skills
cd ~/Code/auto-skills
npm install
npm run build
```

Use SSH instead if the user prefers it:

```bash
git clone git@github.com:CodePuri/Auto-skills.git ~/Code/auto-skills
```

## Commands

```bash
skill-aggregator refresh
skill-aggregator refresh --network
skill-aggregator refresh --dry-run
skill-aggregator suggest --task "<task>" --json
skill-aggregator suggest --task "<task>" --json --offline
skill-aggregator install <candidate-id> --dry-run
skill-aggregator install <candidate-id>
skill-aggregator hook --task "<task>" --json --offline
```

If using a direct script path, replace `skill-aggregator` with `node <repo>/dist/cli.js`.

## Workflow

### Discover Skills

When the user asks to find a skill, extend capabilities, improve output, or check whether a skill exists for a task:

```bash
skill-aggregator suggest --task "<task>" --json
```

For offline-only discovery, use:

```bash
skill-aggregator suggest --task "<task>" --json --offline
```

Review the JSON results. Prefer candidates with strong task fit, clear descriptions, reputable sources, high install counts when available, and `canAutoInstall: true` only when all trust checks pass.

### Present Recommendations

Before suggesting installation, present the user with:

- skill name and candidate id
- description
- source URL or local source path
- install count if available
- score, reason, and `canAutoInstall`
- the install command if provided
- a short rationale for why it fits the task

Use this approval question:

```text
I found the '<skill-name>' skill, which helps with <task>. Would you like me to install it?
```

### Install Only After Approval

Installation requires explicit user approval for the specific skill. After approval, install by candidate id:

```bash
skill-aggregator install <candidate-id>
```

For a preflight check:

```bash
skill-aggregator install <candidate-id> --dry-run
```

Do not install weak, unknown-source, or non-auto-installable candidates. The CLI should refuse unsafe candidates, but the agent must still apply its own approval and trust review.

## Safety Rules

- Discovery and cache refresh may run automatically when relevant.
- Never install a skill in the background.
- Never install multiple skills from one vague approval.
- Never treat `canAutoInstall: true` as user consent.
- Never bypass confirmation flags unless the user has already approved that exact installation.
- Do not recommend a skill solely because it exists; recommend it only when it likely improves the task.
- For trivial requests such as greetings, simple shell commands, or short factual answers, do not run a hook or recommend skills.

## Contextual End-of-Task Hook

After meaningful coding, design, automation, research, debugging, or workflow tasks, check whether a skill would improve future work:

```bash
skill-aggregator hook --task "<task summary>" --json --offline
```

If `shouldSuggest` is false, do not mention skills. If true, briefly ask whether the user wants to add or use one of the recommended skills.

## Refresh and Automation

Refresh the metadata index before serious discovery, on a schedule, or when the user asks for newer sources:

```bash
skill-aggregator refresh --network
```

Weekly automation prompt:

```text
Run the Auto Skills CLI refresh command from the Auto Skills repository or installed skill-aggregator binary. Prefer `skill-aggregator refresh --network`; if the binary is not on PATH, run `node dist/cli.js refresh --network` from the repository root. Report the cache path, whether local skills were found, whether configured git sources were inspected, and any notable new or changed skills visible from the output. Do not install skills, do not run `skill-aggregator install`, and do not modify anything except the aggregator metadata cache created by the refresh command.
```

## Sources and Configuration

The CLI can aggregate from:

- installed local skill folders such as `~/.codex/skills` and `~/.agents/skills`
- shared local folders such as `~/Code/Skills`
- configured GitHub or git repositories
- Skills CLI / skills.sh results when network discovery is available

User-specific sources can be configured at:

```text
~/.config/skill-aggregator/sources.json
```

Keep cached data metadata-first: name, description, source, install command, install count, hash, score, and refresh time. Fetch or inspect full `SKILL.md` content only when evaluating top candidates or installing.
