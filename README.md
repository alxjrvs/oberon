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

## Staying current

Removing the `version` pins means an install resolves to HEAD of `main` — the copy this repo
edits. That only reaches a machine that refreshes, and **Claude Code disables auto-update for
third-party marketplaces by default**, so an installed plugin sits at the commit it was
installed from until something moves it.

Turn it on once, per marketplace: `/plugin` → **Marketplaces** → **oberon** → **Enable
auto-update**. Claude Code then refreshes the catalogue and updates installed plugins in the
background after startup, on a random delay of up to ten minutes so the running session keeps
the version it launched with. When a plugin does update it prompts for `/reload-plugins`;
otherwise the new version loads on the next launch.

A machine that converges from a dotfiles repo can declare it instead of toggling it by hand:

```json
{
  "extraKnownMarketplaces": {
    "oberon": {
      "source": { "source": "github", "repo": "alxjrvs/oberon" },
      "autoUpdate": true
    }
  }
}
```

This is a reconcile, not an upgrade step: the setting is declared once and Claude Code does the
refreshing, so a provisioning script still installs a plugin only when it is absent. Note that
`autoUpdate` on an `extraKnownMarketplaces` entry is documented as the *managed-settings* path;
if a user-scope `settings.json` is what you have, confirm the marketplace reads as auto-updating
in `/plugin` and fall back to the toggle above if it does not.

## License

The [MIT License](LICENSE).
