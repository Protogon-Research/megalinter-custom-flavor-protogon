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

1. **Version sync**: The `check-new-megalinter-version` workflow (manual dispatch only) checks for new MegaLinter releases, creates a matching release in this repository, and dispatches the builder workflow for it. It does this with the job's built-in `GITHUB_TOKEN` (the job has `actions: write`), so no Personal Access Token is needed — see below. It is deliberately not scheduled: a scheduled run would build every upstream release the night it appears, executing the upstream builder action on upstream's cadence rather than ours, and nothing consumes the new image automatically anyway — upgrading is still the manual bump in step 4.

2. **Automated builds**: The version-sync workflow dispatches the `megalinter-custom-flavor-builder` workflow for each new release (a release created by hand also triggers it via the `release` event). The builder:
   - Builds a Docker image with only the selected linters
   - Publishes to GitHub Container Registry (ghcr.io)
   - Optionally publishes to Docker Hub (if credentials are configured)

3. **Available image tags**:
   - Release tags (e.g., `v9.6.0`): Built from MegaLinter releases
   - `beta` tag: Built from non-main branch pushes for testing
   - `latest` tag: Points to the most recent release

4. **After each new image**: bump the image tag in `action.yml` here, then bump the SHA reference in the monorepo's `.github/workflows/CI.yml`.

## Configuration requirements

### No Personal Access Token

The upstream custom-flavor template expects a `PAT_TOKEN` secret so the version-sync workflow can dispatch the builder. **This repo deliberately does not configure one**: a leaked or compromised PAT can give attackers broad write access. Instead, the version-sync job grants itself `actions: write` and dispatches the builder with the built-in `GITHUB_TOKEN`, which is scoped to this repository and expires when the job ends.

If you re-run `npx mega-linter-runner --custom-flavor-setup`, it will regenerate the workflow from the template, putting the `PAT_TOKEN` fallback and the daily `schedule:` trigger back — re-apply the `actions: write` permission and remove the schedule afterwards.

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
