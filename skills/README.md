# Skills

SpecScore skills fall into two categories: a small set of **infrastructure** skills that handle plugin-level concerns (such as installing the CLI) and a larger set of **CLI-wrapper** skills that expose the `specscore` CLI's command surface to agents. Each CLI-wrapper skill covers a single CLI resource group; per-verb detail lives under `references/` and loads on demand.

See the [agent-skills feature spec](https://github.com/synchestra-io/synchestra/blob/main/spec/features/agent-skills/README.md) for design principles and the progressive-disclosure model. See [ADR-0002](https://github.com/synchestra-io/synchestra/blob/main/spec/decisions/0002-progressive-disclosure-skills.md) and [ADR-0003](https://github.com/synchestra-io/synchestra/blob/main/spec/decisions/0003-skill-naming-plugin-namespace.md) for the structural and naming rules.

## Invocation

All skills are prefixed with the plugin's manifest name. Users invoke them as:

```
/specscore:<skill-name>
```

## Skill categories

- **Infrastructure skills** — plugin-level actions that are not backed by a `specscore` CLI command. Today this category contains only `install`, which bootstraps the CLI itself.
- **CLI-wrapper skills** — one skill per `specscore` CLI resource group (`feature`, `task`, `spec`, `code`, `idea`). Each wrapper assumes the CLI is already installed and callable; see the [Pre-flight pattern](#pre-flight-pattern) below for the shared check wrappers must include.

## Available infrastructure skills

| Skill | Purpose |
|---|---|
| [`install/`](install/SKILL.md) | Install the `specscore` CLI via the official `get-cli` installer. Runtime prerequisite for every wrapper skill. |

## Planned CLI-wrapper catalogue

The wrapper catalogue mirrors the `specscore` CLI surface. One resource-level skill per command group, per-verb `references/<verb>.md` inside each. Each row links to the authoritative CLI feature spec that defines the command's contract.

| Skill | Wraps | Verbs | CLI feature spec |
|---|---|---|---|
| `feature/` | `specscore feature ...` | `info`, `list`, `tree`, `deps`, `refs`, `new` | [`cli/feature`](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/feature/README.md) |
| `task/` | `specscore task ...` | `list`, `info`, `new` | [`cli/task`](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/task/README.md) |
| `spec/` | `specscore spec ...` | `lint` | [`cli/spec`](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/spec/README.md) |
| `code/` | `specscore code ...` | `deps` | [`cli/code`](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/code/README.md) |
| `idea/` | `specscore idea ...` | TBD — pending CLI migration to strict resource+verb pattern | [`cli/new/idea`](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/new/idea/README.md) |

The Synchestra.io CLI ecosystem (synchestra, specscore, rehearse) is standardising on a strict `resource + verb` command shape. The top-level `specscore new` command is legacy from before this convention and is not wrapped here; it will be replaced by the appropriate `<resource> new` form in the CLI. The shared exit-code contract, output-format conventions, and `--project` autodetect rules that every wrapper assumes are defined once in the [CLI parent feature](https://github.com/synchestra-io/specscore-cli/blob/main/spec/features/cli/README.md).

The exact shape of each resource skill follows the template defined in [`agent-skills` feature spec](https://github.com/synchestra-io/synchestra/blob/main/spec/features/agent-skills/README.md#skill-file-format):

```
<resource>/
  SKILL.md         ← index table: user intent → reference file
  references/
    <verb>.md      ← full instructions for one CLI invocation
```

## Pre-flight pattern

Every CLI-wrapper skill must verify that `specscore` is installed before invoking it. Copy the block below verbatim into the top of any new wrapper `SKILL.md`:

> ### Pre-flight check
>
> Before running any `specscore` command, verify the CLI is installed:
>
> ```bash
> command -v specscore >/dev/null 2>&1
> ```
>
> If this check fails (exit `127` / `command not found`), stop and tell the user exactly:
>
> > The `specscore` CLI is not installed. Either:
> > - invoke `/specscore:install` to see install options, or
> > - install directly from <https://specscore.md/install>.
> >
> > Then retry your command.
>
> Do not proceed with the original command until `specscore --version` succeeds.

When the first wrapper skill lands, this snippet may be extracted to a shared reference file and linked from each `SKILL.md`. Until then, it lives here as the single source of truth.

## Status

**Shipped:** `install/` (infrastructure).

The CLI-wrapper skills listed above are still planned; individual wrappers ship incrementally as they are authored.

## Outstanding Questions

- The final per-verb list tracks the `specscore` CLI's evolving surface. When new CLI commands land, a corresponding reference file is added here in the same release cycle.
- Whether to ship a `design/` skill in this plugin is **parked**. There is no `specscore design` CLI command today. If one is added later, a wrapping skill lands in the same release. Until then, "design" is purely a methodology concept and lives in the [`spec-driven-development`](https://github.com/synchestra-io/ai-plugin-sdd) plugin.
- Final shape of the `idea/` skill depends on the `specscore` CLI migrating fully to the resource+verb pattern (e.g., `specscore idea new <slug>` instead of the current `specscore idea <slug>`).
