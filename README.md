# gha-workflows

Reusable GitHub Actions workflows shared across my repositories.

Call one with `uses: josa42/gha-workflows/.github/workflows/<file>@main`.

| Workflow | Use for |
| --- | --- |
| [`ha-integration.yml`](.github/workflows/ha-integration.yml) | Home Assistant custom integrations: ruff, pytest, HACS and hassfest |
| [`ha-plugin.yml`](.github/workflows/ha-plugin.yml) | Home Assistant frontend cards: syntax check and HACS |
| [`ha-blueprints.yml`](.github/workflows/ha-blueprints.yml) | Home Assistant blueprint repositories driven by a Makefile |
| [`ha-release.yml`](.github/workflows/ha-release.yml) | Tagged releases for integrations and cards |
| [`esphome.yml`](.github/workflows/esphome.yml) | ESPHome device configs: yamllint and `esphome config` |
| [`nvim-plugin.yml`](.github/workflows/nvim-plugin.yml) | Neovim plugins: stylua and plenary busted |

Each workflow detects what the calling repository actually contains and skips
the jobs that do not apply, so most callers need no inputs at all.

## ha-integration.yml

```yaml
jobs:
  ci:
    uses: josa42/gha-workflows/.github/workflows/ha-integration.yml@main
```

Jobs: `lint` (ruff over `custom_components` and the tests directory), `test`
(pytest) and `validate` (HACS plus hassfest).

`lint` is skipped without a ruff config, `test` without a tests directory, and
`validate` without a `hacs.json`. The test requirements file is picked from
`requirements_test.txt`, `requirements-test.txt`, `requirements_dev.txt` or
`requirements-dev.txt`, whichever exists first.

| Input | Default | |
| --- | --- | --- |
| `python_version` | `"3.14"` | |
| `tests` | `tests` | Directory passed to pytest |
| `pytest_args` | `-v --tb=short` | |
| `requirements` | detected | Test requirements file |
| `hacs_category` | `integration` | |
| `hacs_ignore` | | Space separated HACS checks to ignore |
| `skip_lint` / `skip_test` / `skip_validate` | detected | Set to `"true"` to force off |

## ha-plugin.yml

```yaml
jobs:
  ci:
    uses: josa42/gha-workflows/.github/workflows/ha-plugin.yml@main
```

Runs `npm run check` when `package.json` defines that script, otherwise
`node --check` on the entry point named by `hacs.json`.

| Input | Default | |
| --- | --- | --- |
| `node_version` | `"22"` | |
| `file` | `hacs.json` `filename` | Card entry point |
| `hacs_ignore` | | Space separated HACS checks to ignore |
| `skip_lint` / `skip_validate` | | Set to `"true"` to force off |

## ha-blueprints.yml

```yaml
jobs:
  ci:
    uses: josa42/gha-workflows/.github/workflows/ha-blueprints.yml@main
```

Runs `make venv` and then each target in `targets`. Every target runs even
after a failure, so one broken blueprint does not hide an out of date README.
Targets the Makefile does not define are skipped.

| Input | Default | |
| --- | --- | --- |
| `python_version` | `"3.12"` | |
| `requirements` | `requirements-dev.txt` | Used for the pip cache key |
| `targets` | `lint validate readme-check` | |

## esphome.yml

```yaml
jobs:
  ci:
    uses: josa42/gha-workflows/.github/workflows/esphome.yml@main
```

Runs [`josa42/actions/esphome-lint`](https://github.com/josa42/actions/tree/main/esphome-lint):
yamllint over the config directory, then `esphome config` for every file in
`files` with a top-level `esphome:` key. Missing secrets are filled with dummy
values, so no `secrets.yaml` needs to be committed.

| Input | Default | |
| --- | --- | --- |
| `working_directory` | `.` | Root of the ESPHome config |
| `files` | `*.yaml` | Newline separated globs of device files |
| `esphome_version` | `latest` | |
| `secrets_file` | | Secrets file used instead of dummy secrets |
| `secrets` | | YAML mapping merged on top of the dummy secrets |

## ha-release.yml

```yaml
on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    uses: josa42/gha-workflows/.github/workflows/ha-release.yml@main
```

For an integration it checks that the tag matches the version in
`manifest.json`, zips `custom_components/<domain>` and attaches the archive.
For anything else it attaches the file named by `hacs.json`. Release notes are
generated from the commits since the previous tag.

The caller needs `permissions: contents: write`.

| Input | Default | |
| --- | --- | --- |
| `files` | detected | Release assets |
| `check_version` | `"true"` | Fail when the tag does not match the manifest |
| `draft` | `false` | |
| `prerelease` | `false` | |
