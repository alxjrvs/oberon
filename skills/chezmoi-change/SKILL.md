---
name: chezmoi-change
description: The worktree → apply → PR ritual for a chezmoi dotfiles repo (alxjrvs/dotFiles). The source checkout stays on main; a change is made in a worktree and applied from there with --source "$PWD". Use for "change my dotfiles", "edit a dotfile", "add a chezmoi template", "test a dotfiles change before merging", "apply from a worktree", "delete a dotfile", "why didn't chezmoi remove that file", "chezmoi run script".
---

# chezmoi-change

`~/Code/dotFiles` is chezmoi's source directory. It **stays on `main`** — only `chezmoi update`
moves it. Every change is made in a worktree, applied from that worktree, and lands through a PR.

The point of the ritual: `chezmoi apply` renders whatever branch the source checkout is on, and
says nothing about which one that was. Testing a change by checking a branch out in the source
directory leaves the machine silently running an unmerged branch until someone notices. A worktree
plus `--source "$PWD"` keeps the tested tree and the live tree separate and explicit.

**Four traps, each of which has cost a day, are in [`references/traps.md`](references/traps.md).**
The two that bite during an ordinary edit are inlined below; read the reference before deleting a
source file or touching anything under `.chezmoiscripts/`.

## Procedure

1. **Confirm the source checkout is on `main` and clean.** If it is not, stop and say so — a dirty
   or branched source directory means an earlier ritual was abandoned partway, and applying from
   it renders that abandoned state.
   ```sh
   git -C ~/Code/dotFiles status --short --branch
   ```

2. **Make the worktree.** Claude Code's native worktrees are the default — dotFiles' `.gitignore`
   already names `.claude/worktrees/`, so they cost the repo nothing:
   ```sh
   claude --worktree            # or: git -C ~/Code/dotFiles worktree add ../dotFiles-<topic> -b <topic>
   ```
   Either flavour is safe to apply from. What makes a worktree path dangerous is `--init`, not the
   path itself — see trap 1.

3. **Edit the source files there.** Ordinary chezmoi source naming (`dot_`, `private_`, `run_`,
   `.tmpl`); the worktree is an ordinary checkout.

4. **Preview, then apply, from the worktree.** `$PWD` is the worktree root:
   ```sh
   chezmoi diff  --source "$PWD"
   chezmoi apply --source "$PWD"
   ```
   **Never add `--init`.** Trap 1 — it is the one that deletes the worktree's own directory out
   from under the live config, and re-running does not recover it.

5. **Verify against the real target**, not against the diff you just read. Open the file, run the
   thing it configures, start the shell.

6. **PR, merge, then sync the source checkout.** `chezmoi update` is what moves `main`:
   ```sh
   chezmoi update              # pull in the source dir + apply, in one step
   ```
   Do not `git pull` the source directory by hand and skip the apply — that leaves `main` ahead of
   the machine with nothing signalling that it is.

7. **Remove the worktree** once merged. The branch is gone server-side if the repo deletes on
   merge; the local worktree is not.
   ```sh
   git -C ~/Code/dotFiles worktree remove <path>
   ```

## Guardrails

- **Never `chezmoi init` with `--source` from a worktree.** The config template pins `sourceDir`
  to the working tree it was rendered from. Point that at a worktree and the live config names a
  directory that gets deleted along with the worktree — chezmoi is then left with no source at
  all. Trap 1.
- **Never leave the source checkout on a branch.** `apply` renders the checked-out branch and
  reports nothing unusual, so the failure is silent and outlives the session that caused it.
  Trap 2.
- **A deleted source file does not delete its target.** `chezmoi apply` never removes a target
  whose source went away. The PR that deletes a source states the `rm` to run on each machine, in
  the PR body — otherwise the file lives on every machine forever. Trap 3.
- **A `run_` script cannot call `chezmoi`.** The outer `apply` holds the persistent-state lock, so
  the inner call blocks until it times out. Script the underlying command directly. Trap 4.
- **Apply before the PR, not after.** The worktree apply *is* the test. A PR whose change was
  never applied from its own worktree has been reviewed, not verified.
