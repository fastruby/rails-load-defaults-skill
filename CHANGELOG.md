# Changelog

All notable changes to this skill are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
See [RELEASING.md](RELEASING.md) for what counts as a major, minor, or patch
change for a skill.

Versions before 1.0.0 predate the Claude Code plugin manifest and were never
published to a marketplace. Their tags were backfilled from git history.

## [Unreleased]

## [1.1.0] - 2026-08-20

### Added

- Rails 8.0 support: `configs/8_0.yml` and `templates/new_framework_defaults_8_0.rb`
  covering `active_support.to_time_preserves_timezone`,
  `action_dispatch.strict_freshness`, and `Regexp.timeout`.
- Rails 8.1 support: `configs/8_1.yml` and `templates/new_framework_defaults_8_1.rb`
  covering `action_controller.escape_json_responses`,
  `action_controller.action_on_path_relative_redirect`,
  `active_record.raise_on_missing_required_finder_order_columns`,
  `active_support.escape_js_separators_in_json`, `action_view.render_tracker`,
  and `action_view.remove_hidden_field_autocomplete`.
- A `yjit` entry in `configs/8_1.yml`. `load_defaults 8.1` changes it from `true`
  to `!Rails.env.local?`, disabling YJIT in development and test while leaving
  production unchanged. Rails does not expose this one in the
  `new_framework_defaults_8_1.rb` template, so the entry carries a `note`
  explaining how to give it its own test cycle.

### Fixed

- `old_default` in two entries was prose rather than a Ruby literal. Step 5 of the
  skill writes that field verbatim into `config/application.rb` when a user keeps
  the old behavior, so the previous values emitted broken config:
  `active_support.to_time_preserves_timezone` in `8_0.yml` is now `:offset`
  (was `":offset (previously `true`)"`), and `action_view.render_tracker` in
  `8_1.yml` is now `:regex` (was `"regex-based tracker"`).

### Changed

- README and `SKILL.md` list 8.0 and 8.1 in the supported version tables.

## [1.0.1] - 2026-05-24

### Added

- Social preview image for the repository.
- `.gitignore`.

### Changed

- README documents installing from inside the Claude Code CLI with
  `/plugin marketplace add` and `/plugin install`.

## [1.0.0] - 2026-04-17

First release published as a Claude Code plugin.

### Added

- `.claude-plugin/plugin.json` manifest, making the skill installable through the
  `ombulabs-ai` marketplace.

## [0.5.0] - 2026-04-08

### Changed

- **Breaking:** moved `SKILL.md`, `configs/`, and `templates/` into a
  `rails-load-defaults/` subdirectory so the repo can be consumed as a plugin.
  Any path referencing the old locations needs updating.

## [0.4.0] - 2026-03-24

### Added

- Rails 5.0, 5.1, 5.2, 6.0, and 6.1 support: config references and initializer
  templates for each, extending the skill back to the oldest `load_defaults`
  versions.
- `cookie_rotator.rb` template for SHA1 to SHA256 cookie rotation.

## [0.3.1] - 2026-02-27

### Fixed

- `application_rb_only` configs are now handled explicitly. These cannot live in
  the generated initializer, so `SKILL.md` gained step 4e for testing them
  directly in `config/application.rb` and rules for removing or keeping them
  during consolidation.

## [0.3.0] - 2026-02-20

### Added

- Rails 7.2 support: `configs/7_2.yml` and `templates/new_framework_defaults_7_2.rb`.

## [0.2.0] - 2026-02-20

### Added

- Rails 7.1 support: `configs/7_1.yml` and `templates/new_framework_defaults_7_1.rb`.

## [0.1.0] - 2026-02-18

First working version of the skill.

### Added

- `SKILL.md` describing the detect, generate, walk through, consolidate workflow.
- Rails 7.0 support: `configs/7_0.yml` and `templates/new_framework_defaults_7_0.rb`.
- Tiered risk model (tier 1 safe, tier 2 needs a codebase check, tier 3 needs
  human review) with per-config lookup patterns and decision trees.

[Unreleased]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v1.0.1...v1.1.0
[1.0.1]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.5.0...v1.0.0
[0.5.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.4.0...v0.5.0
[0.4.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.3.1...v0.4.0
[0.3.1]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.3.0...v0.3.1
[0.3.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/ombulabs/claude-code_rails-load-defaults-skill/releases/tag/v0.1.0
