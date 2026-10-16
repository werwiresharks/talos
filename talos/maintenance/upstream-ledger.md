# Hermes upstream maintenance ledger

This ledger records what was observed and tested for each Talos upstream
integration. A failed or incomplete check remains visible; later results append
evidence instead of rewriting history.

## 2026-10-15 — M0 initial synchronization

| Field | Result |
| --- | --- |
| Talos repository | `werwiresharks/talos` |
| Upstream repository | `NousResearch/hermes-agent` |
| Pre-sync Talos SHA | `e06df9c2eae6901f4c88f464e0ae6e753c0428b8` |
| Recovery reference | `origin` tag `recovery/m0-pre-hermes-sync-20261015-e06df9c` |
| Synchronized upstream SHA | `60f56a6b13c8eb9a9e088c6fc1165257c92f7e11` |
| Fresh divergence before sync | 0 Talos-only, 2,418 upstream-only commits |
| Integration method | Strict fast-forward (`git merge --ff-only upstream/main`) |
| Published result | `origin/main` and `upstream/main` both at the synchronized SHA |
| Branch rule observed | GitHub reported `main` as not protected (HTTP 404 from the branch-protection endpoint) |
| Host | Windows, Python 3.14.7, Node 26.7.0, npm 12.0.2 |
| Isolated state | `%TEMP%\talos-m0-20261015\home` |

### Preservation and repository audit

- The actual checkout was the nested `talos` repository. The outer workspace
  repository had no commits and was not modified.
- The Talos worktree was clean. There were no submodules, stashes, additional
  worktrees or local branches.
- The checkout began as a shallow partial clone (`blob:none`). Complete history
  was fetched before calculating ancestry.
- `origin` and `upstream` URLs matched the repositories in this ledger.
- The recovery tag resolves to the exact pre-sync SHA locally and on `origin`.

### Validation

- Supported PowerShell activation completed in the isolated Hermes home.
- `hermes --version` reported `v0.22.1+202.g60f56a6 (2026.9.24)` from the
  synchronized checkout.
- `python scripts/check` passed all 12 blocking checks. Code health reported 0
  blocking and 11 advisory findings.
- `scripts/run_tests.sh` started the canonical per-file suite: 6,229 files and
  approximately 47,356 tests. The run was stopped at 5.7% after 3,387 passes and
  27 failures because the M0 validation gate was already red. This was not a
  complete full-suite pass.
- The credential-boundary failure reproduced in isolation:
  `tests/agent/test_context_references.py::test_blocks_canonical_read_denylist_credential_stores`.
  The result included the fixture value `GCLOUD-SECRET` from
  `.config/gcloud/application_default_credentials.json`.
- The LSP baseline failure reproduced in isolation:
  `tests/agent/lsp/test_manager_locking.py::test_delta_baseline_is_capped_by_write_recency`.
  The expected refreshed `/repo/f0.py` baseline was evicted.
- Other failures observed in the partial run were concentrated in Windows LSP
  file locking, MCP timing/path escaping, compaction event timing and local
  streaming transport. They require isolated triage before classification.
- The focused backend matrix completed 79 passes, 3 failures and 3 skips across
  seven files. CLI first-run setup, the session runtime schema, delegation and
  worker execution behavior passed. The failures were two POSIX file-mode
  assertions in `tests/tools/test_memory_tool.py` on Windows and the inherited
  container-workspace case in
  `tests/tui_gateway/test_desktop_session_workspace_persistence.py`.
- A follow-up cross-platform matrix completed 86 tests with no failures across
  `test_gateway_cli_commands.py`, `test_protocol.py` and
  `test_safe_worker_tools.py`.
- Desktop dependencies installed from the committed npm lock. Focused Vitest
  runs for the canonical protocol, session API and delegation renderer completed
  34 tests with no failures.
- `npm ci` reported deprecated transitive packages, including old `glob`,
  `rimraf`, `@xmldom/xmldom`, `inflight` and `rcedit` versions. Dependency
  health remains follow-up work; the lock-consistent install itself completed.

### Status

The Git synchronization and recovery requirements are complete. Blocking checks
are green. The full test gate is red and incomplete, so M0 baseline validation
is not recorded as fully passing.

## 2026-10-15 — Same-day divergence and reviewed merge

After the initial fast-forward was published, Talos `main` gained commit
`4659216887ffbdf6fbda77712bccc413bb021c76`, which changes the README heading
from Hermes Agent to Talos. Hermes independently advanced by 31 commits to
`acc3cada0da86c81efc8aa6c5a291989549ca900`.

The histories had merge base `60f56a6b13c8eb9a9e088c6fc1165257c92f7e11`
with 1 Talos-only and 31 Hermes-only commits. A second fast-forward was therefore
not permitted.

| Field | Result |
| --- | --- |
| Pre-merge Talos SHA | `4659216887ffbdf6fbda77712bccc413bb021c76` |
| Hermes SHA | `acc3cada0da86c81efc8aa6c5a291989549ca900` |
| Recovery reference | `origin` tag `recovery/m0-pre-hermes-merge-20261015-4659216` |
| Integration method | Clean, non-fast-forward merge on a review branch |
| Merge SHA | `3c41c7cd144cbea3f772409513f78af87ca5b795` |
| Review | Draft pull request `werwiresharks/talos#2` |
| Preservation check | The README heading remains `Talos` |

Validation on the merge branch:

- `python scripts/check` passed all 12 checks with 0 blocking and 0 advisory
  findings across 104 changed files.
- The credential deny-list and LSP baseline-recency failures still reproduced
  in isolation at the newer Hermes SHA.
- The integration pull request remains draft. The merge has not been applied to
  `main`, and M0 is not recorded as fully passing.
