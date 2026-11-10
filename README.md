# Fork to Brand: Rebrand an App Without Breaking Upstream

> Fork an open source app and make it yours without losing the merge. Renames every layer in dependency order, keeps upstream reachable, and proves the rename did not break anything.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg) ![Claude Code](https://img.shields.io/badge/Claude_Code-skill-blueviolet.svg) [![Available on Agensi](https://img.shields.io/badge/Agensi-%2414.99-ff5a5f.svg)](https://agensi.io/skills/fork-to-brand-rebrand-an-app-without-breaking-upstream)

## What it does

Fork and rebrand an upstream app into your own product: rename every layer in dependency order, keep the upstream remote reachable, and prove the runtime still boots after the rename.

Turning an upstream open-source app into your own product: fork it, rename
every layer, keep the upstream mergeable, and prove the rename did not break
runtime contracts the compiler cannot see.

## The failure it prevents

The cases that cost time are specific, and they are written down:

- **BSD sed has no `\b`.** `sed 's/\brakazo\b/x/'` silently no-ops on macOS.
- **Running a text-mode editor over binary files aborts the whole run**
- **A blanket rename mangles URLs.** `rakazo.com` → `rakazo.the target app` is wrong.
- **Renaming an asset directory breaks the path that points at it.** Move the
- **Kotlin/Java package paths must match the package declaration.** Moving
- **Swift module names must match the podspec and the Expo module config.**

## When to use it

- "Fork X and make it ours" / "rebrand X as Y" / "use X's codebase for our app"
- Renaming a workspace, package scope, product name, app id, bundle id, or
- Any repo where a mechanical rename must survive typecheck, lint, tests, AND

## What's inside

```
plugins/
└── fork-to-brand/
    ├── .claude-plugin/plugin.json
    └── skills/fork-to-brand/
        ├── SKILL.md
```

## Install

Add this repo as a Claude Code marketplace, then install the skill:

```bash
claude plugin marketplace add Verstige/fork-to-brand
claude plugin install fork-to-brand@fork-to-brand
```

Or copy the skill straight into your skills directory:

```bash
git clone https://github.com/Verstige/fork-to-brand.git
cp -R fork-to-brand/plugins/fork-to-brand/skills/fork-to-brand ~/.claude/skills/
```

## Usage

The skill activates on the work it describes. Ask for the outcome, and it
supplies the method:

```
> "fork an upstream app and make it yours"
```

## License

MIT. See [LICENSE](LICENSE).

---

**Also available as a paid skill on Agensi** — $14.99. Install it with one command, no clone required:

[agensi.io/skills/fork-to-brand-rebrand-an-app-without-breaking-upstream](https://agensi.io/skills/fork-to-brand-rebrand-an-app-without-breaking-upstream)

Part of the [Verstige](https://github.com/Verstige) skills portfolio. More at [github.com/Verstige](https://github.com/Verstige).
