# MegaLinter Custom Flavor: protogon

This custom MegaLinter aims to have an optimized Docker image size.

It is built from official MegaLinter images, but is maintained on https://github.com/Protogon-Research/megalinter-custom-flavor-protogon by Benjamin Cosman

## Embedded linters

  - [ACTION_ACTIONLINT](https://megalinter.io/latest/descriptors/action_actionlint/)
  - [ACTION_ZIZMOR](https://megalinter.io/latest/descriptors/action_zizmor/)
  - [BASH_EXEC](https://megalinter.io/latest/descriptors/bash_exec/)
  - [BASH_SHELLCHECK](https://megalinter.io/latest/descriptors/bash_shellcheck/)
  - [BASH_SHFMT](https://megalinter.io/latest/descriptors/bash_shfmt/)
  - [EDITORCONFIG_EDITORCONFIG_CHECKER](https://megalinter.io/latest/descriptors/editorconfig_editorconfig_checker/)
  - [HTML_HTMLHINT](https://megalinter.io/latest/descriptors/html_htmlhint/)
  - [JSON_JSONLINT](https://megalinter.io/latest/descriptors/json_jsonlint/)
  - [JSON_PRETTIER](https://megalinter.io/latest/descriptors/json_prettier/)
  - [JSON_V8R](https://megalinter.io/latest/descriptors/json_v8r/)
  - [MARKDOWN_MARKDOWNLINT](https://megalinter.io/latest/descriptors/markdown_markdownlint/)
  - [REPOSITORY_CHECKOV](https://megalinter.io/latest/descriptors/repository_checkov/)
  - [REPOSITORY_GIT_DIFF](https://megalinter.io/latest/descriptors/repository_git_diff/)
  - [REPOSITORY_SECRETLINT](https://megalinter.io/latest/descriptors/repository_secretlint/)
  - [REPOSITORY_TRUFFLEHOG](https://megalinter.io/latest/descriptors/repository_trufflehog/)
  - [YAML_PRETTIER](https://megalinter.io/latest/descriptors/yaml_prettier/)
  - [YAML_V8R](https://megalinter.io/latest/descriptors/yaml_v8r/)
  - [YAML_YAMLLINT](https://megalinter.io/latest/descriptors/yaml_yamllint/)

## How to use the custom flavor

Follow [MegaLinter installation guide](https://megalinter.io/latest/install-assisted/), and replace related elements in the workflow.

- **GitHub Action**: On MegaLinter step in `.github/workflows/mega-linter.yml`, define `uses: Protogon-Research/megalinter-custom-flavor-protogon@<sha or tag>` (this action pins a specific image version tag — see `action.yml`)
- **Docker image**: Replace official MegaLinter image with `ghcr.io/protogon-research/megalinter-custom-flavor-protogon/megalinter-custom-flavor:v9.6.0`

## How the flavor is generated and updated

This custom flavor is kept up to date with MegaLinter releases:

1. **Version sync**: The `check-new-megalinter-version` workflow (daily cron + manual dispatch) checks for new MegaLinter releases and creates matching releases in this repository. NOTE: we deliberately do NOT configure the `PAT_TOKEN` secret (see security warning below), so the daily run cannot trigger the builder itself — bump versions by running that workflow manually, or by creating a release by hand.

2. **Automated builds**: Each release triggers the `megalinter-custom-flavor-builder` workflow, which:
   - Builds a Docker image with only the selected linters
   - Publishes to GitHub Container Registry (ghcr.io)
   - Optionally publishes to Docker Hub (if credentials are configured)

3. **Available image tags**:
   - Release tags (e.g., `v9.6.0`): Built from MegaLinter releases
   - `beta` tag: Built from non-main branch pushes for testing
   - `latest` tag: Points to the most recent release

4. **After each new image**: bump the image tag in `action.yml` here, then bump the SHA reference in the monorepo's `.github/workflows/CI.yml`.

## Configuration requirements

### Optional: Personal Access Token (use with care)

> **Security warning**: Using a Personal Access Token (PAT) is **not recommended**. A leaked or compromised PAT can give attackers broad write access to your repository. If you do not need fully automatic daily version sync, you can skip the PAT entirely and trigger the `check-new-megalinter-version` workflow manually whenever you want to upgrade. **This repo currently skips the PAT on purpose.**

If automatic daily releases ever become worth the trade-off, configure a `PAT_TOKEN` secret as a **repository-scoped fine-grained token** with:

- **Repository access**: Only select repositories (select this repository)
- **Repository permissions**:
  - Contents: Read and write
  - Actions: Read and write

Rotate the token regularly.

### Optional: Docker Hub publishing

To publish to Docker Hub in addition to ghcr.io, configure:

- `DOCKERHUB_REPO` variable (e.g., your Docker Hub username)
- `DOCKERHUB_USERNAME` secret
- `DOCKERHUB_PASSWORD` secret

## How to generate the flavor manually

If you need to manually trigger a build:

1. **Create a GitHub release**: Creates a versioned build matching the tag name (e.g., `v9.6.0`)
2. **Push to any branch** (except main): Builds a `beta` tagged image for testing
3. **Manually run the workflow**: Go to Actions > Build & Push MegaLinter Custom Flavor > Run workflow (takes an explicit `megalinter-version`)

See [full Custom Flavors documentation](https://megalinter.io/latest/custom-flavors/).

[![MegaLinter is graciously provided by OX Security](https://raw.githubusercontent.com/oxsecurity/megalinter/main/docs/assets/images/ox-banner.png)](https://www.ox.security/?ref=megalinter)
