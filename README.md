# t0k0sh1-skills

A plugin marketplace for Claude Code, Codex CLI, and Grok Build.

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json      # Claude Code marketplace
├── .agents/plugins/
│   └── marketplace.json      # Codex marketplace; same plugin directories
├── .grok-plugin/
│   ├── marketplace.json      # Grok Build marketplace
│   └── plugin-index.json     # skill and agent names for the marketplace browser
├── plugins/
│   └── example-plugin/       # one directory per plugin
│       ├── .claude-plugin/
│       │   └── plugin.json   # Claude metadata
│       ├── .codex-plugin/
│       │   └── plugin.json   # Codex metadata
│       ├── .grok-plugin/
│       │   └── plugin.json   # Grok metadata
│       ├── skills/           # one shared <name>/SKILL.md for every client
│       ├── agents/           # subagents (*.md)
│       └── output-styles/    # Claude output styles (*.md)
└── README.md
```

## Plugins

The descriptions below describe Claude Code behavior. Codex and Grok differences are in the sections below.

| Name | What it does |
| --- | --- |
| `action-first` | Always-on action-first output: lead with the answer or next action, number multi-step work, show what now works, suppress tangents, no preamble, recap or closers. Ships as an output style and is applied automatically when the plugin is enabled |
| `devils-advocate` | Adversarial review of a finished plan or design. Shared review skill, with a Claude subagent entry point |
| `interview-me` | Interview mode started with `/interview-me <requirement or design>`. One question at a time, no recommendations, no multiple choice, ends on approval. After approval it lists ADR candidates and files only the ones you pick into docs/adr via `adrs` |
| `babysit-pr` | One iteration of PR babysitting, built to run under `/loop` (e.g. `/loop /babysit-pr 123`). Works from the PR author's side: reads PR state, addresses reviewers' open threads and failing CI, replies, pushes, and ends the loop when every thread is resolved and every check has passed. Stops as BLOCKED after 2 failed fix attempts or when a human decision is needed. User-invoked only |

## Install with Claude Code

From a terminal:

```bash
# 1. Register this marketplace (first time only)
claude plugin marketplace add t0k0sh1/skills

# 2. Install a plugin (<plugin-name> is a name from the table above)
claude plugin install <plugin-name>@t0k0sh1-skills

# Example
claude plugin install interview-me@t0k0sh1-skills
```

Inside a Claude Code session, the `/plugin` command does the same:

```
/plugin marketplace add t0k0sh1/skills
/plugin install interview-me@t0k0sh1-skills
```

After installing, restart Claude Code or open a new session for the plugin to take effect. `claude plugin list` shows what is installed.

To develop against a local checkout instead, add its path in place of `t0k0sh1/skills`
(`claude plugin marketplace add /path/to/skills`).

### Getting updates

Changes pushed to this repository do not reach installed plugins automatically. Pull them in with:

```bash
claude plugin marketplace update t0k0sh1-skills
claude plugin update <plugin-name>
```

## Uninstall from Claude Code

```bash
# Remove a plugin
claude plugin uninstall <plugin-name>

# Example
claude plugin uninstall interview-me
```

To pause a plugin without removing it, disable it:

```bash
claude plugin disable <plugin-name>
claude plugin enable <plugin-name>    # re-enable
```

To remove the marketplace registration itself:

```bash
claude plugin marketplace remove t0k0sh1-skills
```

All of these also work inside a session through `/plugin`, for example `/plugin uninstall <plugin-name>`.

## Adding a plugin

1. Create `plugins/<plugin-name>/` with `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, and `.grok-plugin/plugin.json`.
2. Put the shared instructions in `skills/<skill-name>/SKILL.md`.
3. Add the same plugin directory to `.claude-plugin/marketplace.json`, `.agents/plugins/marketplace.json`, and `.grok-plugin/marketplace.json`. Add its skills and agents to `.grok-plugin/plugin-index.json`.
4. Keep any client-specific entry points limited to loading those shared instructions.


## Codex CLI

The Claude and Codex catalogs are rooted at this repository. Codex's
`.agents/plugins/marketplace.json` uses the official
[repo marketplace format](https://developers.openai.com/plugins/build/plugins).
Both clients load the same `plugins/<plugin>/skills/<skill>/SKILL.md` and
supporting resources. There is no generated copy or build step.

### Install

Verified with Codex CLI 0.154.0 as of 2026-09-23 — re-check against your installed version. Use a version that provides
`codex plugin marketplace` and `codex plugin add` (check `codex plugin --help`):

```bash
codex plugin marketplace add t0k0sh1/skills
codex plugin add interview-me@t0k0sh1-skills
# Replace interview-me with any plugin from the table above.
```

`codex plugin marketplace add` also accepts an HTTPS or SSH Git URL, or a local
path to a checkout of this repository.

If you registered the earlier `codex/` directory, run
`codex plugin marketplace remove t0k0sh1-skills`, register the repository
root with the command above, and reinstall your plugins.

Start a new Codex session after installation. Invoke skills using `$`:

```text
$interview-me:interview-me Clarify this feature requirement
$babysit-pr:babysit-pr 123
$devils-advocate:devils-advocate Review docs/plan.md
$action-first:action-first
```

### Behavior differences

| Plugin | Codex behavior |
| --- | --- |
| `action-first` | A skill, rather than an automatically forced output style. Invoke `$action-first:action-first` for the session. For always-on behavior, copy the rules from `plugins/action-first/skills/action-first/SKILL.md` (without YAML frontmatter) into the target project's `AGENTS.md`, preserving its existing instructions. |
| `devils-advocate` | A skill that requests an independent subagent when available. If delegation is unavailable, it explicitly reports a same-agent review. Claude uses its named `opus` agent; Codex uses the available delegation tool. Both read the same review instructions. |
| `interview-me`, `babysit-pr` | Explicit invocation only, preserved through `agents/openai.yaml` with `allow_implicit_invocation: false`. The shared instructions read the text supplied with the invocation, without requiring `$ARGUMENTS` substitution. |

`interview-me` still needs the external `adrs` executable if you choose to
file ADRs; installation of this plugin does not install `adrs`.

### Update or remove

After changes are pushed, refresh the marketplace snapshot, reinstall the
plugin to replace its cached files, then start a new session:

```bash
codex plugin marketplace upgrade t0k0sh1-skills
codex plugin remove interview-me@t0k0sh1-skills
codex plugin add interview-me@t0k0sh1-skills
```

To uninstall, run the `remove` command above. To unregister the marketplace:

```bash
codex plugin marketplace remove t0k0sh1-skills
```

## Grok Build

When `.grok-plugin/marketplace.json` is present, Grok reads that index and
does not also read `.claude-plugin/marketplace.json`. Skills stay in
`plugins/<plugin>/skills/<skill>/SKILL.md`. Per-plugin
`.grok-plugin/plugin.json` overrides `.claude-plugin/plugin.json` for Grok.

### Install

Verified with Grok Build 1.0.41:

```bash
grok plugin marketplace add t0k0sh1/skills
grok plugin install interview-me@t0k0sh1/skills --trust
grok plugin enable interview-me
```

Replace `interview-me` with any plugin from the table above. The
`@t0k0sh1/skills` pin selects this marketplace when another source offers the
same plugin name. A bare `grok plugin install interview-me --trust` works when
it is the only source. `--trust` is required for skills and agents to load.
Plugins stay off until enabled. A local checkout is pinned as
`<plugin>@local/<directory-name>` (the directory name Grok printed when the
source was added).

`grok plugin marketplace add` also accepts an HTTPS or SSH Git URL, or a local
checkout of this repository.

Start a new session after installation, or press `r` in the Plugins tab.
Invoke skills as slash commands. If a name collides with another skill or a
built-in, Grok shows the qualified form `/<plugin>:<skill>`.

```text
/interview-me Clarify this feature requirement
/babysit-pr 123
/devils-advocate Review docs/plan.md
/action-first
```

### Behavior differences

| Plugin | Grok behavior |
| --- | --- |
| `action-first` | A skill, rather than an automatically forced output style. Grok has no output styles. Invoke `/action-first` for the session. For always-on behavior, copy the rules from `plugins/action-first/skills/action-first/SKILL.md` (without YAML frontmatter) into the target project's `AGENTS.md`, preserving its existing instructions. |
| `devils-advocate` | `/devils-advocate` runs the shared review in an independent subagent. Grok also lists the bundled agent as `devils-advocate:devils-advocate`. A spawned review does not select that agent by name. The agent file keeps `model: opus` for Claude Code. |
| `interview-me` | Explicit invocation only, via `disable-model-invocation: true`. |
| `babysit-pr` | Model-invocable (no `disable-model-invocation`), because `/loop` fires it through the model. It repeats under `/loop`. Grok's `/loop` is a fixed interval of at least 60 seconds and the task expires after 7 days; cancel a finished loop with `scheduler_delete`. |

`interview-me` still needs the external `adrs` executable if you choose to
file ADRs; installation of this plugin does not install `adrs`.

### Update or remove

```bash
grok plugin marketplace update t0k0sh1-skills
grok plugin update interview-me
```

To uninstall a plugin, or to unregister the marketplace:

```bash
grok plugin uninstall interview-me --confirm
grok plugin marketplace remove t0k0sh1-skills
```

`marketplace remove` also uninstalls plugins that came from that source.
`disable` / `enable` pause a plugin without removing it.

## Maintaining shared skills

Edit `plugins/<plugin>/skills/<skill>/SKILL.md` once for every client.
Keep supporting references in that skill's directory. No generation is needed.
Claude's output-style and agent files are small entry points that refer to
these shared instructions, rather than maintaining another copy.

For explicit-only skills, keep these settings together:

- `disable-model-invocation: true` in `SKILL.md` for Claude Code and Grok Build.
- `policy.allow_implicit_invocation: false` in `agents/openai.yaml` for Codex.

Use client-neutral wording for invocation input and tool actions. Claude and
Grok use slash commands (plugin skills may be namespaced, e.g.
`/interview-me:interview-me`); Codex uses `$interview-me:interview-me`.
See [Codex skill metadata](https://developers.openai.com/ja-JP/docs/build-skills).

When adding a plugin, create `.claude-plugin/plugin.json`,
`.codex-plugin/plugin.json`, and `.grok-plugin/plugin.json`, point them at the
same `skills/`, and add the plugin to all three marketplace catalogs. Update
versions in the manifests and the Claude and Grok catalogs together when
releasing changes. Refresh `.grok-plugin/plugin-index.json` when a skill or
agent is added or its description changes. Leave `sha` off local plugins;
Grok uses `sha` for plugins fetched from a separate repository.

## Verification

Verified with Claude Code 2.1.278 and Codex CLI 0.154.0 (as of 2026-09-23) in isolated
configuration directories: both installed all five plugins that existed at
the time from this root; Codex `skills/list` loaded all six shared skills as
enabled without errors. `karpathy-guidelines` and `show-me` were removed after
this check, and it has not been repeated for the four remaining plugins. Grok Build 1.0.41 validates every plugin, lists all
six from `.grok-plugin/marketplace.json` with the component catalog, and
installs them with `--trust`.
The general-purpose Codex scaffold validators reject Claude-specific
frontmatter (`argument-hint` and `disable-model-invocation`); the installed
Codex loader accepts it. Preserve those fields for Claude and use
`agents/openai.yaml` for Codex's invocation policy.
