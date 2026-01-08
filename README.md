# Releez Version Artifact Action

Generate semver, Docker, and PEP440 artifact versions using `releez`.

This action wraps `releez version artifact` to produce version strings that
match your repo's semantic release rules (powered by `git-cliff`).

## Requirements

- `uv` on `PATH` (see https://github.com/astral-sh/uv)
- `git` on `PATH`
- A `cliff.toml` (or compatible `git-cliff` config) in your repo

## Usage

Basic usage (PR build):

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
      - id: versions
        uses: hotdog-werx/releez-version-artifact-action@v1
        with:
          prerelease-type: alpha
          prerelease-number: ${{ github.event.pull_request.number }}
      - run: echo "Docker versions: ${{ steps.versions.outputs.docker-versions }}"
```

Full release (also emits alias versions when enabled):

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
      - id: versions
        uses: hotdog-werx/releez-version-artifact-action@v1
        with:
          is-full-release: 'true'
          alias-versions: minor
      - run: echo "${{ steps.versions.outputs.semver-versions }}"
```

## Inputs

| Input               | Required | Default                    | Description                                                           |
| ------------------- | -------- | -------------------------- | --------------------------------------------------------------------- |
| `releez-version`    | no       | `0.1.3`                    | Git ref for the `releez` tool to install (tag, branch, or SHA).       |
| `prerelease-type`   | no       | `""`                       | Prerelease type: `alpha`, `beta`, or `rc`.                            |
| `prerelease-number` | no       | `0`                        | Number appended to the prerelease tag.                                |
| `is-full-release`   | no       | `false`                    | Set to `true` to compute a full release (enables alias versions).     |
| `build-number`      | no       | `${{ github.run_number }}` | Build number appended to prerelease versions.                         |
| `version-override`  | no       | `""`                       | Override the version detected from the repo.                          |
| `alias-versions`    | no       | `""`                       | Alias versions to emit on full releases: `none`, `major`, or `minor`. |
| `github-token`      | no       | `${{ github.token }}`      | Token used by `releez` for GitHub access.                             |

## Outputs

| Output            | Description                  |
| ----------------- | ---------------------------- |
| `semver-versions` | Semver-formatted version(s). |
| `docker-versions` | Docker tag version(s).       |
| `pep440-versions` | PEP440 (Python) version(s).  |

## Output format

Each output is newline-separated. When alias versions are enabled for a full
release, the output includes the primary version and `v`-prefixed aliases.
Example:

```
1.2.3
v1.2
v1
```

## Notes

- `alias-versions` is only used when `is-full-release` is `true`.
- If `prerelease-type` is not set, prerelease inputs are ignored.
- `build-number` is only passed for non-full releases.
- `releez` reads config defaults from `releez.toml` or `pyproject.toml`
  (`[tool.releez]`), with CLI flags taking precedence.
