---
name: rails-load-defaults
description: >
  Incrementally upgrade Rails `config.load_defaults` by walking through each
  new_framework_defaults config one at a time. Use this skill whenever the user
  mentions load_defaults upgrade, new_framework_defaults, framework defaults,
  or wants to bring their Rails app's default configuration up to match their
  Rails version. Also trigger when the user mentions a gap between their Rails
  version and their load_defaults version (e.g., "app is on Rails 7.2 but
  load_defaults is 6.1"). This skill handles the iterative uncomment-test-commit
  workflow and the final consolidation into config/application.rb.
---

# Rails load_defaults Upgrade Skill

This skill guides the incremental upgrade of `config.load_defaults` in a Rails
application. The process walks through each config introduced in a new Rails
version, one at a time, so the user can test each change in isolation.

## Overview

When a Rails app has a `load_defaults` version lower than its Rails version,
there are framework default configs that need to be adopted. This skill:

1. Detects the current `load_defaults` version
2. Determines the next target version
3. Generates the `new_framework_defaults_X_Y.rb` initializer with all configs commented
4. Walks through each config one by one, analyzing the codebase to recommend values
5. Waits for the user to test/commit each change before moving to the next
6. Consolidates into `config/application.rb` when all configs are done

## Step 1: Detect Current State

Read `config/application.rb` and look for:

```ruby
config.load_defaults X.Y
```

Also check if a `config/initializers/new_framework_defaults_*.rb` file already
exists (the user may be mid-upgrade).

Report to the user:

- Current `load_defaults` version
- Current Rails version (from Gemfile.lock)
- Which version transitions are needed (e.g., 6.1 → 7.0 → 7.1 → 7.2)

## Step 2: Load Version Config Reference

Read the appropriate config reference file for the target version:

- **7.0**: `configs/7_0.yml`
- **7.1**: `configs/7_1.yml` (when available)
- **7.2**: `configs/7_2.yml` (when available)

Each config file contains entries organized into tiers with lookup patterns
and decision trees for each config.

## Step 3: Create the Initializer File

If no `new_framework_defaults_X_Y.rb` exists yet, copy the template from
the `templates/` directory into the app's `config/initializers/`:

- **7.0**: Copy `templates/new_framework_defaults_7_0.rb` → `config/initializers/new_framework_defaults_7_0.rb`
- **7.1**: Copy `templates/new_framework_defaults_7_1.rb` (when available)
- **7.2**: Copy `templates/new_framework_defaults_7_2.rb` (when available)

These templates contain the exact canonical Rails initializer with all configs
commented out, matching what `rails app:update` would generate. Always use
the template rather than generating from scratch — this ensures the comments,
formatting, and config ordering match the Rails source.

## Step 4: Iterative Config Walkthrough

Process configs in order from safest (Tier 1) to those needing analysis (Tier 2).

For each config:

### 4a. Analyze the Codebase

Run the lookup patterns defined in the config reference:

- Use `grep -r` or `find` to search for the patterns listed
- Check the specific files/directories indicated
- Apply the decision tree to determine the recommended value

### 4b. Present Recommendation

Tell the user:

- What the config does (old behavior → new behavior)
- What you found in their codebase
- Your recommended value and why
- Risk level (low/medium/high)

### 4c. Apply the Change

Once the user agrees:

- Uncomment the config line in the initializer file
- Set the value (either the new default or the override value)

### 4d. Wait for User

Stop and wait. The user will:

- Commit and push the change
- Run CI or manually test
- Come back to confirm success or report failure

If the change broke something:

- Re-comment the config line or set it to the old value
- Note it as needing investigation
- Move to the next config

### 4e. Track Progress

Keep track of which configs have been processed by reading the initializer
file. Commented lines = pending. Uncommented lines = done.

Configs that are explicitly noted in the file with comments like
`# NOTE: must be configured in config/application.rb` should be handled
separately during consolidation.

## Step 5: Consolidation

When all configs in the initializer have been uncommented and tested:

1. Delete the `config/initializers/new_framework_defaults_X_Y.rb` file
2. Update `config.load_defaults` in `config/application.rb` to the new version
3. **CRITICAL**: For any configs where you kept the OLD behavior/value (i.e., did
   not adopt the new default), you MUST add explicit overrides in
   `config/application.rb` after the `load_defaults` line. If you skip this,
   setting `load_defaults` to the new version will silently switch those configs
   to their new default values — undoing your deliberate decision to keep the
   old behavior.

   Each config entry in the version reference YAML includes an `old_default`
   field. Use that value for the override.

```ruby
config.load_defaults 7.0

# Override: kept old behavior because CSS targets input[type=submit] from button_to
config.action_view.button_to_generates_button_tag = false
```

4. Also handle any configs that were flagged as "must go in config/application.rb"
   (like `cache_format_version` and `disable_to_s_conversion`) — these were
   never in the initializer file, so set them here if adopting the new default,
   or omit them to keep old behavior.
5. Remind the user to commit and do a final round of testing.

## Step 6: Next Version

If more version transitions are needed, start again at Step 2 with the next
version.

## Important Notes

- Never rush through configs. Each one gets its own commit and test cycle.
- Some configs have notes saying they must go in `config/application.rb` (not
  the initializer). Flag these clearly and handle them during consolidation.
- When the decision tree says "if unsure, keep the old default" — do that.
  It's always safer to be conservative.
- The user's testing (CI, manual QA) is the ultimate arbiter. The codebase
  analysis is guidance, not gospel.
