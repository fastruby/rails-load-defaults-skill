# Releasing

This skill uses [semantic versioning](https://semver.org/spec/v2.0.0.html) with git
tags and a [CHANGELOG.md](CHANGELOG.md).

## The invariant

Three places carry the version. They move together, or the release is broken:

1. the git tag, `vX.Y.Z`
2. `version` in `.claude-plugin/plugin.json`
3. `ref` in this plugin's entry in
   [`ombulabs/claude-skills`](https://github.com/ombulabs/claude-skills)
   `.claude-plugin/marketplace.json`

## What counts as which bump

The skill has no API, so the contract is how it is invoked and what it generates.

| Bump  | Meaning | Examples |
| ----- | ------- | -------- |
| MAJOR | Breaking change to how the skill is invoked or structured: file layout, `SKILL.md` workflow step semantics, `configs/*.yml` schema | moving files into `rails-load-defaults/` (0.5.0) |
| MINOR | New Rails version support, new capability, new tier or template | Rails 8.0 and 8.1 configs (1.1.0) |
| PATCH | Config accuracy fixes, corrected defaults or risk tiers, docs, wording | the `old_default` literal fixes (1.1.0) |

A config accuracy fix is a PATCH even though it changes what lands in a user's
`config/application.rb`. It corrects the skill's output toward what Rails actually
does; it does not change how the skill is used.

## Steps

1. Land the changes on `main`.
2. Pick the bump from the table above.
3. Move the `## [Unreleased]` entries in `CHANGELOG.md` under a new
   `## [X.Y.Z] - YYYY-MM-DD` heading, and add the compare link at the bottom.
4. Bump `version` in `.claude-plugin/plugin.json` to exactly `X.Y.Z`, no `v`.
5. Commit as `chore(release): vX.Y.Z`.
6. Tag and push:

   ```bash
   git tag -a vX.Y.Z -m "vX.Y.Z"
   git push origin main --tags
   ```

7. Create the release, using the changelog section as the notes:

   ```bash
   gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."
   ```

8. Open a PR against `ombulabs/claude-skills` updating this plugin's `source.ref`
   to `vX.Y.Z`.

## Why step 4 is not optional

`plugin.json`'s `version` is what Claude Code uses to decide whether an install is
stale. From the [marketplace
docs](https://code.claude.com/docs/en/plugin-marketplaces):

> Setting `version` pins the plugin for every source type [...] If you declare
> `"version": "1.0.0"` in `plugin.json` and push new commits without changing that
> string, existing users of those sources keep the cached copy, because Claude Code
> sees the same version.

This repo shipped in exactly that state from April to August 2026: the manifest
said `1.0.0` the whole time while the marketplace SHA moved, so existing installs
never saw the Rails 8.x work. Skipping the bump silently ships nothing.

Related gotcha from the same docs: do not also set `version` in the marketplace
entry. `plugin.json` always wins, without a warning, so a version in both places
means the marketplace one is a decoy.

## Verifying a release actually shipped

In a scratch directory:

```
/plugin marketplace update
/plugin install rails-load-defaults@ombulabs-ai
```

Confirm the installed version matches the tag and that whatever the release added
is present on disk. An install still reporting the previous version means the cache
did not invalidate, which almost always means step 4 was missed.
