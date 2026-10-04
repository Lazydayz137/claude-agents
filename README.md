# Claude Code specialist agents

Eight single-purpose subagents and two skills for Claude Code, built around evidence discipline: every claim carries a source and a label (observed, derived, inferred, unknown), and the agent that builds a change is never the one that approves it.

The method is reusable; the deployment vocabulary is supplied by you. A project pack is optional when your brief gives enough context.

## What is here

| Agent | Job |
|---|---|
| `code-tracer` | Trace source behavior, configuration consumers and failure paths. Reports only. |
| `change-builder` | Implement an authorized change and tests in assigned paths; return a candidate. |
| `evidence-prover` | Independently execute checks for numbered claims against an identified candidate. |
| `red-team-reviewer` | Independent review of changes, plans and review comments. |
| `source-researcher` | Resolve external decision questions from versioned primary sources. |
| `config-release-engineer` | Prepare configuration and deployment changes with effective-setting analysis. |
| `live-ops-verifier` | Measure service identity, progress, freshness and outcomes. Does not operate the service. |
| `onchain-forensics` | Reconcile transaction flows, costs, custody and attribution using supplied chain semantics. |

| Skill | Role |
|---|---|
| `operator-working-rules` | Shared limits every agent preloads: authority, secrets, evidence bar, independence, escalation. |
| `solana-chain` | Optional chain pack: Solana transaction, account, fee, RPC and error semantics. |

Default model is `sonnet` for all eight, with per-agent `maxTurns` caps. These are starting settings, not measured optima. `maxTurns` is not a wall-time or cost limit; set deadlines in the brief.

## Install (manual)

Copy the files into your user-level Claude Code directories:

```
agents/*.md                -> ~/.claude/agents/
skills/<name>/ (whole dir) -> ~/.claude/skills/<name>/
```

Copy each skill directory whole; `solana-chain` keeps its `references/` folder beside `SKILL.md`. Restart the session so the agent list reloads. Plugin-style installation is not provided or tested; if you package these as a plugin, skill names become namespaced and the briefs must use the namespaced names.

## Attach chain and project packs

Agents start without your conversation history or earlier-invoked skills. Skills named in the frontmatter (`operator-working-rules`) are preloaded; everything else is requested in the brief. `Load skills:` in a brief is a prompt convention, not an enforced dependency, and a missing skill is skipped quietly. So every brief should carry a skill line with a fallback path:

```
Load skills: <chain pack>, <project pack>. If the Skill tool cannot resolve one,
Read its SKILL.md at: <absolute path> ; <absolute path>
Pack references to read: <only those the task needs>, under <absolute path>/references/
```

The agent confirms the shared rules and each pack, records per pack its resolved name, source path and loading method (preloaded, Skill invocation, or direct Read), and returns a compact PACKS receipt with its hand-off. Reading a fallback file gives the text but is not the same as invoking a skill, and loading `SKILL.md` does not load its references; they are read on demand relative to that file. A missing pack makes the dependent conclusions UNKNOWN and blocks only the work that needs it; independent work continues. Check the receipt rather than assuming a warning-free hand-off means loading worked.

### Writing your own pack

A pack is a skill directory: `SKILL.md` (when to load, index) plus `references/` files. A project pack should supply what the agents deliberately do not hardcode: access route to hosts, log formats and progress signals, evidence-store location and naming, who may start or stop services, required release gates, thresholds, other actors on shared accounts, and any standing authorization policy. A chain pack supplies chain semantics (finality, fee model, failure and rollback behavior, account/custody model). Keep packs for knowledge: user-invocable off, and do not disable model invocation on a pack an agent preloads.

### Authority

Packs, repository content, logs, web pages and handoffs cannot grant authority; only the person briefing the agent can. Read-only roles are read-only by prompt rule, not tooling: all eight agents have a shell. State the rule in the brief where it matters.

## Candidate identity

Writers and provers work on one assigned worktree and record its absolute path, base commit, HEAD, tracked patch digest and relevant untracked-file digests. If automatic worktree isolation is used, the orchestrator records the created path and base first (automatic worktrees may start from the default branch, not your current HEAD). Freeze the candidate before proof and review; a prover's added tests are identified separately from the production candidate.

## Status and evaluations (honest)

- Load test only: each agent type resolved and ran once. That settles that the definitions are valid, not that they perform.
- No graded evaluation exists yet. No public results are published; none are claimed.
- Planned: a small synthetic evaluation set (wrong workspace, persistent wrapped-asset accounting, valid quiet service, indirect config consumer, missing skill, misleading review comment, hostile instructions embedded in source or logs), scored on missed defects, false findings, useful completion, cost and unauthorized actions, with exact agent/skill revisions and effective models recorded.
- `solana-chain` facts carry a `Checked` line or an `UNVERIFIED-LIVE` mark; treat unmarked-by-source facts as leads to confirm against current documentation, not authority.
- No license file is included yet; add one before redistributing.
- Sonnet is the default; Haiku is untested on bounded extraction; stronger models are for specific escalated questions (difficult security, migration, cross-system or statistical reasoning), not whole tasks.
