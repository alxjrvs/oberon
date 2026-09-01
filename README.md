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

## This repository holds no plugin code, and that is the design

There is one file here that matters: `.claude-plugin/marketplace.json`. It is a **registry**,
not a library. Every entry sources straight from the repository that already uses that plugin
day to day — for `agent-friendly-repo`, that is
[`alxjrvs/dotFiles`](https://github.com/alxjrvs/dotFiles).

The obvious way to build a marketplace is to copy each skill in and maintain it here. That
creates two copies of the same file with no mechanism keeping them equal, and the copy that
gets *used* is never the copy that gets *edited*. It drifts silently, and the published version
is the stale one — the failure mode is invisible until someone installs it and gets last
month's advice.

Sourcing in place makes drift **impossible rather than unlikely**. There is nothing to sync
because there is nothing duplicated. The version people install is the version being exercised.

`strict: false` is what allows this: the marketplace entry carries the metadata, so the source
repository needs no `plugin.json` and no plugin-shaped directory layout. It stays a dotfiles
repo that happens to be installable.

## What is deliberately not here

`dotFiles` also ships two subagents, `drift-triage` and `guard-tester`. They stay unpublished:
both are written against that machine's specific setup — one reads `boom verify` output, the
other runs that repo's own guard suites. They are useful there and noise anywhere else.

A marketplace earns trust by what it declines to publish.

## License

MIT.
