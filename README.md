# workflows

Shared, reusable GitHub Actions workflows for Kotake Technology projects.

Projects keep only thin caller workflows; the build, publish and Claude logic lives here.
Updating a workflow and moving the `v1` tag rolls the change out to every project.

## Reusable workflows

| Workflow | Purpose | Inputs | Secrets |
|----------|---------|--------|---------|
| `maven-ci.yml` | Build and test. Push: `mvn test`. Pull request: `mvn verify`. Writes a JaCoCo coverage summary. | `java-version` (default `21`) | `PACKAGES_TOKEN` (optional) |
| `maven-publish.yml` | Deploy to GitHub Packages. On `release`, sets the version from the tag (`v1.2.3` → `1.2.3`). | `java-version` (default `21`) | — |
| `claude.yml` | Runs Claude Code when `@claude` is mentioned in issues, PR comments or reviews. | — | `CLAUDE_CODE_OAUTH_TOKEN` |
| `claude-code-review.yml` | Automated Claude code review on pull requests. | — | `CLAUDE_CODE_OAUTH_TOKEN` |

## Usage

Copy the caller workflows from [`templates/maven/`](templates/maven) into the project's `.github/workflows/`.
Example:

```yaml
jobs:
  build:
    uses: ismaelkotake/workflows/.github/workflows/maven-ci.yml@v1
    permissions:
      contents: read
      packages: read
```

The Maven workflows use the project's Maven Wrapper (`./mvnw`) when present, otherwise the runner's `mvn`.

A reusable workflow can never get more permissions than the caller grants, so callers must declare the
`permissions` shown in the templates.

Projects that depend on packages published from another private repository (e.g. `kotake-parent`)
must pass a `PACKAGES_TOKEN` secret (a personal access token with `read:packages`): the `GITHUB_TOKEN`
of one repository cannot read Maven packages of another private repository.

```yaml
    secrets:
      PACKAGES_TOKEN: ${{ secrets.PACKAGES_TOKEN }}
```

The Claude workflows need the `CLAUDE_CODE_OAUTH_TOKEN` repository secret
(set up by `/install-github-app` in Claude Code, or `gh secret set CLAUDE_CODE_OAUTH_TOKEN`).

## Versioning

- `vX.Y.Z` tags are immutable releases.
- `vX` is a moving tag pointing to the latest `vX.Y.Z`. Projects should reference `@v1`.
- Breaking changes (renamed inputs, new required secrets) bump the major version.

To release:

```bash
git tag vX.Y.Z && git push origin vX.Y.Z
git tag -f vX vX.Y.Z && git push -f origin vX
```

This repository is public so that both public and private repositories can use it. Never put secrets here.
