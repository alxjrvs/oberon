# Oberon

A Claude Code plugin marketplace.

```
/plugin marketplace add alxjrvs/oberon
```

Named for Mister Miracle's assistant — the one who handles the setup so the escape looks
effortless.

## What is here

| Plugin | What it does |
|---|---|
| `agent-friendly-repo` | Configures a GitHub repo so the agent completion path — commit, push, PR, `gh pr merge --auto --squash` — actually lands. Squash-only merges, a branch-protection ruleset that keeps CI required *without* a human review gate an agent cannot satisfy, stacked PRs, and optional Dependabot auto-merge or a merge queue. |
| `1password-mcp` | The 1Password desktop app's own MCP server, by absolute path. Installing the plugin is the whole setup: nothing is written to `~/.claude.json`, so a dotfiles repo can converge it through `enabledPlugins` like any other plugin. |

## The gate

`main` takes squash-merged pull requests with the `lint` check green: the marketplace
manifest and every skill validated by `claude plugin validate`. The ruleset is
[`.github/gate.sh`](https://github.com/alxjrvs/dotFiles/blob/main/.github/gate.sh) in
dotFiles, run once against this repo.

## One copy, and it lives here

A plugin exists in exactly one place. Two copies of a skill with no mechanism keeping them
equal drift silently, and the copy that gets *used* is never the copy that gets *edited* — the
published version ends up stale, invisibly, until someone installs it and gets last month's
advice.

So each plugin is held here rather than pointed at. A skill that configures **other people's**
repositories is not machine state and has no business in a dotfiles repo; this is its home, and
there is no second copy anywhere to fall behind it.

`strict: false` keeps that cheap: the marketplace entry carries the metadata, so a plugin needs
no `plugin.json` and no plugin-shaped directory layout — a skill directory is enough.

## License

MIT.
