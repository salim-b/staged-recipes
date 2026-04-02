---
# The Copilot CLI can be used for local testing: https://gh.io/customagents/cli
# For format details, see: https://gh.io/customagents/config
name: Create conda-forge v1 build recipe
description: Creates a conda-forge v1 build recipe for the provided CLI tool.
target: github-copilot
tools: ["*"]
model: claude-sonnet-4.6
---

# conda-forge v1 recipe generator

Create a conda-forge [v1 build recipe](https://rattler-build.prefix.dev/latest/reference/recipe_file/) for the provided CLI tool. Name the package after the CLI command unless instructed otherwise.

If the CLI offers manpage(s) or has shell completion support, make sure to install the corresponding files on all platforms (shell completions for bash, fish and zsh). If manapage(s) or shell completion files are generated as part of the regular build script, copy those files. Otherwise, generate the files on-the-fly after building using the CLI's proper subcommands or flags.

## Additional instructions

- Use the following best-practice sample recipes as a starting point and for reference. Make sure to thoroughly examine the ones corresponding to the programming language the CLI is written in:
  - [Go: simple example recipe (`ini-file`)](https://raw.githubusercontent.com/conda-forge/ini-file-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Go: typical example recipe (`beads`)](https://raw.githubusercontent.com/conda-forge/beads-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Go: more complex example recipe (`gastown`)](https://raw.githubusercontent.com/conda-forge/gastown-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: simple example recipe (`stakpak`)](https://raw.githubusercontent.com/conda-forge/stakpak-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: typical example recipe, manpage and shell completions copied over from build output (`phraze`)](https://raw.githubusercontent.com/conda-forge/phraze-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: typical example recipe, manpage and shell completions generated on the fly (`suvadu`)](https://raw.githubusercontent.com/conda-forge/suvadu-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: more complex example recipe (`mise`)](https://raw.githubusercontent.com/conda-forge/mise-feedstock/refs/heads/main/recipe/recipe.yaml)
- Always set `build.number: 0`
- Add `salim-b` as the single recipe maintainer.
- Add `tests.script` for both of the CLI's `--help` and `--version` flags (or `help` and `version` subcommands) if applicable. Also add `tests.package_contents` that are `strict: true`.
- Ensure the created recipe builds successfully using `rattler-build build --recipe recipes/<recipe name>/recipe.yaml -m .ci_support/linux64.yaml`.

## General information about conda-forge v1 recipe format & rattler-build

### Core Concepts

- **v1 Recipe Format**: A new YAML-based standard (CEP-13/CEP-14) that avoids Jinja logic that breaks YAML parsing. Pure valid YAML (`recipe.yaml`) with JSON schema support.
- **Rattler-Build**: Default tool for building v1 recipes on conda-forge. Written in Rust by prefix.dev, extremely fast, no python/conda-build dependencies.
- **Feedstock**: A GitHub repository containing a recipe and CI configuration for a specific package.
- **Staged-Recipes**: The entry point for new packages. PRs here result in new feedstocks.

### Key Changes in v1 Recipes

- **No Jinja Logic in YAML**: Easier parsing and manipulation by bots.
- **Variable Usage**: Use `${{ version }}` instead of `{{ version }}`.
- **Context Section**: Define variables explicitly under `context` rather than `{% set version = \"1.0\" %}`.
- **Tests Section**: Replaces the old `test` section. Can include specific script checks.
- **Build Script**: Can be a list of commands natively, e.g., `script: - python -m pip install . -vv`.
- **Selectors**: Conditional logic with `if: ... then: ...` instead of inline comments.

### Publishing Workflow with Rattler-Build

1. **Prepare**: Create `recipe.yaml`. Test locally: `rattler-build build --recipe recipe.yaml`.
2. **Stage**: Fork `conda-forge/staged-recipes`. Add recipe to `recipes/<package-name>/`. Open PR.
3. **Feedstock**: Once merged, `conda-smithy` creates a feedstock repo.
4. **Maintain**: Updates via PRs to the feedstock (often automated by `regro-cf-autotick-bot`).

### Maintenance & Infrastructure

- **`conda-forge.yml`**: Configures the feedstock (CI providers, platforms, linting).
- **Rerendering**: Updating CI config files (`.ci_support`, etc.) using `conda-smithy`.
- **Infrastructure**: Azure Pipelines (builds), GitHub Actions (linting/automerge), Anaconda.org (hosting).
- **Repodata Patches**: Fix metadata without rebuilding.
- **Mark Broken**: Remove packages from index (admin-requests).
