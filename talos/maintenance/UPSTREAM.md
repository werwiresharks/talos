# Hermes upstream maintenance

Talos keeps `NousResearch/hermes-agent` as its upstream foundation. Upstream
updates are history operations, not ordinary dependency upgrades. Preserve the
current state before moving a branch, and do not replace divergent history.

## Remote layout

```text
origin    https://github.com/werwiresharks/talos.git
upstream  https://github.com/NousResearch/hermes-agent.git
```

Verify these URLs instead of assuming that a local remote name is correct:

```bash
git remote get-url origin
git remote get-url upstream
```

## Manual update procedure

1. Confirm that the worktree is clean and inventory every worktree, stash and
   local branch.
2. Create a recovery tag at the current Talos `main` tip. Use a name such as
   `recovery/m0-pre-hermes-sync-YYYYMMDD-<short-sha>`.
3. Fetch complete upstream history and tags:

   ```bash
   git fetch upstream main --tags
   ```

4. Calculate divergence. The first number is Talos-only history and the second
   number is upstream-only history:

   ```bash
   git rev-list --left-right --count main...upstream/main
   git merge-base --is-ancestor main upstream/main
   ```

5. If Talos has no unique commits and the ancestry check succeeds, prepare a
   fast-forward-only update:

   ```bash
   git switch main
   git merge --ff-only upstream/main
   ```

6. If Talos has unique commits or ancestry cannot be proved, stop. Create a
   reviewed integration branch and resolve the history explicitly. Never reset,
   force-push or discard Talos history to make the graph appear clean.
7. Use isolated `HERMES_HOME` and `HERMES_RUNTIME_DIR` values. Run the supported
   activation path, `python scripts/check`, and `scripts/run_tests.sh`. Record
   the exact commands, platform, results and incomplete checks in the ledger.
8. Verify the CLI, Desktop, gateway, sessions, model setup, tool execution,
   memory and delegation with tests or running smoke checks. Documentation is
   not proof that a surface works.
9. Merge only reviewed history. Re-fetch upstream immediately before landing,
   then record the landed SHA and rollback reference in
   [`upstream-ledger.md`](upstream-ledger.md).

## GitHub workflow

Run **Talos - Prepare Hermes upstream update** from the default branch. The
workflow fetches full history, proves that Talos `main` is an ancestor of the
requested upstream revision, pushes a namespaced update branch and opens a pull
request. It refuses divergent history. It does not merge, run releases or alter
branch protection.

The workflow intentionally does not invent a green validation result or edit
the ledger. The operator must validate the proposed SHA and add a ledger entry
to the pull request before it lands.

## Recovery

A recovery tag is a pointer to the pre-update commit. To inspect or restore it,
create a new branch; do not reset a shared branch:

```bash
git switch --create recovery/inspect-YYYYMMDD <recovery-tag>
```

If a published update must be reversed, use a reviewed revert or a replacement
branch appropriate to the graph. Never rewrite shared `main` without an
explicit incident decision.
