# The four traps

Reference for the `chezmoi-change` skill. Each of these has cost a day. They are recorded in
`alxjrvs/dotFiles`' `docs/GOTCHAS.md`; this file is the operational form — what not to type, and
what it does if you type it.

## 1. `--init` with `--source` from a worktree deletes chezmoi's source directory

**Never run `chezmoi init --source <worktree>`, in any form.**

`init` renders the config template, and the rendered config pins `sourceDir` to whatever working
tree it was rendered from. Hand it a worktree path and the *live* config — the one every later
`chezmoi apply` on that machine reads — now names the worktree. Remove the worktree when the PR
merges, as step 7 of the ritual says to, and the app deletes that directory: chezmoi is left
pointed at a path that no longer exists.

Re-running `apply` does not recover it. The fix is another `init` against the real source
directory, which is a second render of the config template.

**`--source "$PWD"` on its own is safe from any worktree path**, native (`.claude/worktrees/`) or
plain `git worktree add`. `--source` alone selects a source directory for that one command;
nothing is written to the config, so nothing outlives the process. The worktree flavour is not
what makes the difference — `--init` is. This follows from the mechanism above rather than from a
run on the machine; one apply from a native worktree path, checking `chezmoi source-path` before
and after, would confirm it directly.

## 2. `apply` renders the branch the source checkout happens to be on

`chezmoi apply` reads the source directory's working tree. There is no "you are not on main"
warning, no branch in the output, nothing in `chezmoi diff` that names a ref.

So a source directory left on a topic branch keeps applying that branch — after the session that
checked it out is gone, after the PR is closed unmerged, after the branch is deleted on the
remote. The machine diverges from `main` and reports nothing.

This is the whole reason the ritual exists: **the source checkout stays on `main`, and only
`chezmoi update` moves it.** Worktrees are how a change gets tested without ever violating that.

## 3. `apply` never removes a target whose source was deleted

Deleting a file from the source directory removes it from *future* machines. It does nothing to
the machines that already have the target — `chezmoi apply` adds and updates, and never removes a
target it no longer manages.

**So the PR that deletes a source file names the `rm` for each machine, in its body.** Something
like:

```sh
rm ~/.config/foo/bar.toml    # source deleted in this PR; apply will not remove it
```

Without that line the file lives on every provisioned machine indefinitely, still doing whatever
it did, with nothing in the repo left to explain it.

`chezmoi` has `remove_` / `.chezmoiremove` for managing removals declaratively; if a deletion is
worth doing on every machine automatically, that is the tool, and it belongs in the same PR
instead of the `rm` line.

## 4. A `run_` script cannot call `chezmoi`

Anything under `.chezmoiscripts/` (or any `run_` source file) executes *inside* an outer
`chezmoi apply`, and that outer process holds the persistent-state lock for its whole run. A
nested `chezmoi` call — `chezmoi apply`, `chezmoi source-path`, anything touching state — blocks
on that lock until it times out.

Script the underlying command directly instead. Where a script needs a value chezmoi knows, take
it from the template context or the environment chezmoi exports, not from a nested invocation.
