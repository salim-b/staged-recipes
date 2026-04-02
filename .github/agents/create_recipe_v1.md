---
# The Copilot CLI can be used for local testing: https://gh.io/customagents/cli
# For format details, see: https://gh.io/customagents/config
name: Create conda-forge v1 build recipe
description: Creates a conda-forge v1 build recipe for the provided CLI tool.
target: github-copilot
tools: ["*"]
model: claude-sonnet-5
disable-model-invocation: false
user-invocable: true
---

# conda-forge v1 recipe generator

Create a conda-forge [v1 build recipe](https://rattler-build.prefix.dev/latest/reference/recipe_file/) for the provided CLI tool. Name the package after the CLI command unless instructed otherwise.

## Additional instructions

- Use the following best-practice sample recipes as a starting point and for reference. Make sure to thoroughly examine the ones corresponding to the programming language the CLI is written in:
  - [Go: simple example recipe (`ini-file`)](https://raw.githubusercontent.com/conda-forge/ini-file-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Go: typical example recipe (`beads`)](https://raw.githubusercontent.com/conda-forge/beads-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Go: more complex example recipe (`gastown`)](https://raw.githubusercontent.com/conda-forge/gastown-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: simple example recipe (`stakpak`)](https://raw.githubusercontent.com/conda-forge/stakpak-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: typical example recipe, manpage and shell completions copied over from build output (`phraze`)](https://raw.githubusercontent.com/conda-forge/phraze-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: typical example recipe, manpage and shell completions generated on the fly (`suvadu`)](https://raw.githubusercontent.com/conda-forge/suvadu-feedstock/refs/heads/main/recipe/recipe.yaml)
  - [Rust: more complex example recipe (`mise`)](https://raw.githubusercontent.com/conda-forge/mise-feedstock/refs/heads/main/recipe/recipe.yaml)

- If a project entails more than a single CLI tool (e.g. additional client, server, or shim binaries etc.), package all of them in the recipe unless there is a good reason not to.

- If a CLI offers manpage(s) or has shell completion support, make sure to install the corresponding files on all platforms (shell completions for bash, fish and zsh). If manpage(s) or shell completion files are generated as part of the regular build script, copy those files. Otherwise, generate the files on-the-fly after building using the CLI's proper subcommands or flags.

- If a Rust CLI tool is to be installed from a subdirectory (like `./cli`), OS-independently change to that directory at the very beginning of `build.script.content`. This ensures the `cargo-bundle-licenses` invocation later on captures the right set of licenses (use `--output=../THIRDPARTY.yml` for one subdirectory level, `--output=../../THIRDPARTY.yml` for two levels etc.).

- If a CLI requires a newer glibc version than conda-forge's default 2.17, you can [opt in](https://conda-forge.org/docs/maintainer/knowledge_base/#requiring-newer-glibc-versions) to version `2.28`, `2.34` or `2.39` by setting `c_stdlib_version` in `recipes/<recipe name>/conda_build_config.yaml`:

  ```yaml
  c_stdlib_version:          # [linux]
    - "2.28"                 # [linux]
  ```

- Always set `build.number: 0`

- Add `salim-b` as the single recipe maintainer.

- Add `tests.script` for both of the CLI's `--help` and `--version` flags (or `help` and `version` subcommands) if applicable. Also add `tests.package_contents` that are `strict: true`. Do not add `build_platform == host_platform` gates.

- Make sure to add the correct `homepage` URL (different from the `repository` URL) if such an official website exists. Also add an additional `documentation` URL if a dedicated one exists (excluding `https://docs.rs/*` URLs).

- Ensure the created recipe builds successfully using `rattler-build build --recipe recipes/<recipe name>/recipe.yaml -m .ci_support/linux64.yaml`.

- Name the Git branch containing your changes according to the following pattern (sans angle brackets): `copilot/conda-forge/add-<recipe name>`

## General information about conda-forge v1 recipe format & rattler-build

[`rattler-build`'s official documentation](https://rattler-build.prefix.dev/latest/) as well as [conda-forge's official documentation](https://conda-forge.org/docs/) are the recommended resources to consult if in doubt. What follows is a super concise summary of key points.

### Core concepts

- **v1 recipe format**: A new YAML-based standard (CEP-13/CEP-14) that avoids Jinja logic that breaks YAML parsing. Pure valid YAML (`recipe.yaml`) with JSON schema support.
- **rattler-build**: Default tool for building v1 recipes on conda-forge. Written in Rust by prefix.dev, extremely fast, no python/conda-build dependencies.
- **Feedstock**: A GitHub repository containing a recipe and CI configuration for a specific package.
- **staged-recipes**: The entry point for new packages. PRs here result in new feedstocks.

### Key changes in v1 recipes

- **No Jinja logic in YAML**: Easier parsing and manipulation by bots.
- **Variable usage**: Use `${{ version }}` instead of `{{ version }}`.
- **Context section**: Define variables explicitly under `context` rather than `{% set version = \"1.0\" %}`.
- **Tests section**: Replaces the old `test` section. Can include specific script checks.
- **Build script**: Can be a list of commands natively, e.g., `script: - python -m pip install . -vv`.
- **Selectors**: Conditional logic with `if: ... then: ...` instead of inline comments.

### Publishing workflow with rattler-build

1. **Prepare**: Create `recipe.yaml`. Test locally: `rattler-build build --recipe recipe.yaml`.
2. **Stage**: Fork `conda-forge/staged-recipes`. Add recipe to `recipes/<package-name>/`. Open PR.
3. **Feedstock**: Once merged, `conda-smithy` creates a feedstock repo.
4. **Maintain**: Updates via PRs to the feedstock (often automated by `regro-cf-autotick-bot`).

### Maintenance & infrastructure

- **`conda-forge.yml`**: Configures the feedstock (CI providers, platforms, linting).
- **Rerendering**: Updating CI config files (`.ci_support`, etc.) using `conda-smithy`.
- **Infrastructure**: Azure Pipelines (builds), GitHub Actions (linting/automerge), Anaconda.org (hosting).
- **Repodata patches**: Fix metadata without rebuilding.
- **Mark broken**: Remove packages from index (admin-requests).
