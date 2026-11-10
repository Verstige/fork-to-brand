---
name: fork-to-brand
description: "Fork and rebrand an upstream app into your own product: rename every layer in dependency order, keep the upstream remote reachable, and prove the runtime still boots after the rename."
version: 1.0.0
license: MIT
author: "the maintainers"
---
# Forked App Rebrand

Turning an upstream open-source app into your own product: fork it, rename
every layer, keep the upstream mergeable, and prove the rename did not break
runtime contracts the compiler cannot see.

## When to Use

- "Fork X and make it ours" / "rebrand X as Y" / "use X's codebase for our app"
- Renaming a workspace, package scope, product name, app id, bundle id, or
  env-var prefix across a whole repo
- Any repo where a mechanical rename must survive typecheck, lint, tests, AND
  a running app

## Procedure

### 1. Check the license before anything else

Read the upstream LICENSE. You need rights to rebrand and redistribute.

| License | Rebrand + redistribute? |
|---|---|
| Apache-2.0 | Yes — keep LICENSE, add NOTICE, state changes, retain attribution |
| MIT / BSD | Yes — keep the copyright notice |
| GPL-2.0 | Yes — derivative must also be GPL |
| AGPL-3.0 | Yes — but network use triggers source disclosure |
| No license / "all rights reserved" | **No.** Ask for permission. |

Apache-2.0 additionally grants patent rights and requires a NOTICE file
naming the upstream project plus a statement of your changes. Rename the
product, not the copyright attribution.

### 2. Fork and wire remotes so upstream stays reachable

```bash
gh repo fork <owner>/<upstream> --clone=false --remote=false --fork-name <name>
git clone --origin upstream https://github.com/<owner>/<upstream>.git <dir>
cd <dir> && git remote add origin https://github.com/<you>/<name>.git
git checkout -b <your-branch>
```

Keep `upstream` pointing at the source repo. A fork you cannot pull from is
a copy. Fetch both periodically:

```bash
git fetch upstream && git fetch origin
```

### 3. Establish the baseline BEFORE renaming anything

You cannot tell what you broke if you never knew what worked.

```bash
<install>          # expect clean exit
<typecheck>        # expect 0 errors
<lint>             # record the warning count
<boot the app>     # expect it to serve
```

Record the lint warning count. Later, proving "the fork introduces no new
warnings" needs the baseline. A pristine upstream worktree is the cleanest
baseline:

```bash
git worktree add /tmp/<name>-baseline upstream/main --detach
```

### 4. Rename in dependency order, largest blast radius first

Order matters because later renames depend on earlier ones.

1. **npm/pnpm package scope** (`@upstream/*` → `@ours/*`) + package names
2. **Env-var prefix** (`UPSTREAM_*` → `OURS_*`) — read at runtime, so a
   partial rename silently changes behavior
3. **Product identity** — productName, appId, bundle id, slug, scheme,
   app-group id, Android package
4. **Source identifiers** — exported symbols, types, interfaces
5. **Runtime contracts** — window globals, postMessage keys, cookie names,
   IPC channel names, native class names
6. **User-facing copy** — UI strings, locales, marketing site, docs
7. **Legal** — NOTICE, LICENSE retention, upstream docs preserved as
   `*.upstream.md` rather than deleted

### 5. Rename producers AND consumers together

The single most common way a rebrand breaks at runtime.

A name that crosses a process boundary is a contract with two ends. Renaming
one side leaves the compiler happy and the app broken. Before renaming, list
every pair:

| Contract | Producer | Consumer |
|---|---|---|
| Electron preload global | `contextBridge.exposeInMainWorld("x", …)` | `window.x` in the renderer |
| postMessage key | `postMessage({ x: … })` | `event.data.x` |
| Cookie name | `Set-Cookie: x=…` | `document.cookie` / server read |
| Native class name | `RakazoPdfViewModule` in Swift | `expo-module.config.json` + podspec |
| Environment variable | `env.X` | the environment file + its example copy |
| DB identifier | `CREATE SCHEMA x` in a migration | `x.table` in code |

Rename both ends in the same commit, then grep for the old name to confirm
zero survivors.

### 6. Leave checksummed migrations and other upstream-pinned names alone

Prisma verifies migration checksums — editing a migration breaks deploys.
More generally: **any identifier an upstream migration creates must keep its
upstream name in code.** Renaming the DB role or schema in application code
while the migration still creates the old name fails at runtime, not at
compile time.

```bash
grep -rhoiE 'CREATE (SCHEMA|TABLE|ROLE|FUNCTION) (IF NOT EXISTS )?[a-z_][a-z0-9_.]*' \
  packages/db/prisma/migrations/*/migration.sql | sort -u
```

Cross-check every one against your renamed code. Upstream keeps adding
migrations that reference these names — renaming breaks every future merge.

### 7. Run the linter's formatter after the scope rename

Renaming an npm scope changes alphabetical import order, so a previously
sorted import block is now wrong. Expect hundreds of import-order errors
that are not real bugs:

```bash
<lint>            # e.g. 443 import-order errors
<format>          # auto-fixes mechanically
<lint>            # 0 errors
```

### 8. Prove the app still works — don't trust the compiler

Typecheck and lint do not exercise runtime contracts, database identifiers,
or boot-time wiring. Boot it:

```bash
<start the app>   # background
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://127.0.0.1:<port>/
```

Then exercise one real user path end to end (sign up, create a record, check
the DB row). A green typecheck with a broken boot is the classic outcome of a
rename that missed a contract.

### 9. Commit in logical layers, not one giant commit

| Commit | Contents |
|---|---|
| 1 | Mechanical rename (scope, identity, env prefix, symbols, copy) |
| 2 | Runtime-contract fixes the first commit exposed |
| 3 | Lint/format normalization |

Explain WHY in the body, especially any deliberate non-rename. A reviewer
needs to know that `upstream_app` was left alone on purpose.

## Pitfalls

- **BSD sed has no `\b`.** `sed 's/\brakazo\b/x/'` silently no-ops on macOS.
  Use Python `re` for word-boundary renames, or the `patch` tool.
- **Running a text-mode editor over binary files aborts the whole run**
  (`RE error: illegal byte sequence`). Select text files first with
  `grep -Il`.
- **A blanket rename mangles URLs.** `rakazo.com` → `rakazo.the target app` is wrong.
  Grep for URL-shaped hits and fix them individually.
- **Renaming an asset directory breaks the path that points at it.** Move the
  asset first, then update every reference, then verify the path resolves.
- **Kotlin/Java package paths must match the package declaration.** Moving
  `com/upstream/x` to `com/ours/x` without updating `package com.ours.x`
  fails to compile.
- **Swift module names must match the podspec and the Expo module config.**
  Three files, one name.
- **Upstream tooling may diff against `origin/main`.** A fresh fork with only
  a feature branch lacks it, and the tool throws "Could not read origin/main".
  `git fetch origin main`.
- **The environment example file ships placeholder secrets.** A literal `***` in
  `DATABASE_URL` makes the app try to connect and fail instead of
  provisioning its own container. Generate real values, or unset the var.
- **Check what the tool reports vs. what it means.** "30 warnings" is not a
  failure; compare against the baseline before claiming you introduced them.

## Verification checklist

```bash
<install>                 # clean exit
<typecheck>               # 0 errors
<lint>                    # 0 errors, warnings <= baseline
<boot>                    # app serves
curl <health endpoint>    # {"ok":true}
grep -r <old-scope> .     # 0 hits in source
grep -r <OLD_PREFIX>_ .   # 0 hits
grep -ri <old brand> <rendered html>   # 0 hits
```

Then exercise one real user path and confirm the persisted row.

## Related

- `the target app-architecture` — a worked example of this workflow (Rakazo → the target app)
- `snapshot-diff` — comparing two versions of the same codebase
- `github` — gh CLI operations including forks
