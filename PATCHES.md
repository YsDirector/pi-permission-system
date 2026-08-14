# Fork Patches

This repository is a **single-package fork** of
[`@gotgenes/pi-permission-system`](https://github.com/gotgenes/pi-packages/tree/main/packages/pi-permission-system)
(v25.2.0), vendored from the npm tarball so `pi install git:` can load it
directly (the upstream monorepo root carries no `pi` manifest).

## Motivation

The user requires a command-review gate in pi-web with a dead-man's switch:
dangerous shell commands (`rm` etc., even non-sudo) must open a review dialog,
and **if the user does not confirm within 60 seconds, the request must be
denied by default**.

Upstream prompts interactively without any timeout — a silent user leaves the
request pending forever. This fork adds a 60s fail-closed deadline.

## Patch 1 — Interactive prompt timeout (fail-closed)

File: `src/authority/permission-dialog.ts`

- Added `PROMPT_TIMEOUT_MS = 60_000`.
- `requestPermissionDecisionFromUi` now passes `{ timeout: PROMPT_TIMEOUT_MS }`
  to all three UI calls:
  1. the main approve/session/deny `select`,
  2. the forwarded-ask session-scope `select`,
  3. the deny-reason `input`.
- Extended the local `PermissionDecisionUi` interface so `select`/`input`
  accept the optional `opts` argument (keeps `tsc --noEmit` clean; the real
  SDK `ExtensionUIContext` already supports `opts.timeout`).

Behavior: in pi-web (non-TUI / RPC sessions) the dialog auto-dismisses after
60s and the request resolves as **denied** (a cancelled/`undefined` select is
mapped to `createDeniedPermissionDecision()`). The TUI inline keybind dialog is
unaffected.

## Syncing with upstream

1. Re-download the upstream npm tarball and replace the package files:
   ```bash
   npm pack @gotgenes/pi-permission-system
   tar xzf gotgenes-pi-permission-system-*.tgz
   ```
2. Re-apply Patch 1 (the `permission-dialog.ts` edits above).
3. Commit with a note of the upstream version you vendored.

Upstream changelog: https://github.com/gotgenes/pi-packages/blob/main/packages/pi-permission-system/CHANGELOG.md
