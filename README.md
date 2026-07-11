# Envpilot — GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Envpilot-purple?logo=github)](https://github.com/marketplace/actions/envpilot)
[![GitHub release](https://img.shields.io/github/v/release/rafay99-epic/envpilot-action)](https://github.com/rafay99-epic/envpilot-action/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

**Stop copy-pasting secrets into repository settings.** Envpilot pulls environment variables from your [Envpilot](https://www.envpilot.dev) project straight into your GitHub Actions workflow at runtime — no static secrets in your repo, no stale `.env` files, no drift between environments.

## Quick start

1. Create a service token in [Envpilot](https://www.envpilot.dev) → Project Settings → CI/CD Tokens.
2. Add it as a repository secret (e.g. `ENVPILOT_TOKEN`) in your GitHub repo.
3. Drop the action into your workflow:

```yaml
- uses: rafay99-epic/envpilot-action@v1
  with:
    token: ${{ secrets.ENVPILOT_TOKEN }}
    environment: production
```

Every variable from that environment is now available as `$VAR_NAME` in subsequent steps.

## Why not just use GitHub Secrets?

| GitHub Secrets                              | Envpilot Action                                                     |
| ------------------------------------------- | ------------------------------------------------------------------- |
| Copy-paste each variable individually       | Pull them all in one step                                           |
| Change a value → update in every repo       | Change it once in Envpilot                                          |
| Teams share secrets via copy-paste or Slack | Tokens are scoped per project and environment                       |
| No concept of "environments" per secret set | `production`, `staging`, `development` — one token per environment |

## Inputs

| Input         | Required | Default                    | Description                                                                                      |
| ------------- | -------- | -------------------------- | ------------------------------------------------------------------------------------------------ |
| `token`       | Yes      | —                          | Envpilot service token (`envpk_…`), scoped to a single project and environment.                  |
| `environment` | Yes      | —                          | Environment to pull, e.g. `production`, `staging`, `development`.                                |
| `api-url`     | No       | `https://www.envpilot.dev` | Envpilot API base URL. Override only for self-hosted or non-production instances.                |
| `export-env`  | No       | `"true"`                   | When `true`, exports every pulled variable to `$GITHUB_ENV` so later steps can read it directly. |
| `env-file`    | No       | —                          | Path to write the pulled variables as a dotenv file (`KEY='value'` per line). Skipped if unset.  |

## Outputs

| Output  | Description                              |
| ------- | ---------------------------------------- |
| `count` | Number of variables pulled from Envpilot |

## Recipes

### Multi-environment workflow

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: rafay99-epic/envpilot-action@v1
        with:
          token: ${{ secrets.ENVPILOT_TOKEN }}
          environment: staging
      - run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: rafay99-epic/envpilot-action@v1
        with:
          token: ${{ secrets.ENVPILOT_TOKEN }}
          environment: production
      - run: ./deploy.sh
```

### Write a dotenv file instead of exporting to env

```yaml
- uses: rafay99-epic/envpilot-action@v1
  with:
    token: ${{ secrets.ENVPILOT_TOKEN }}
    environment: production
    export-env: "false"
    env-file: .env
- run: docker compose up -d
```

## Security

- **Values are masked in logs.** Every pulled value is registered with `core.setSecret` — accidental logging shows `***`, not the real value.
- **Scoped tokens.** A service token can only read variables from the project and environment(s) it was minted for.
- **Read-only.** Tokens cannot create, modify, or delete variables.
- **Revocable.** Revoke a token from Envpilot at any time — it's rejected on the next pull with no grace period.
- **Rate-limited.** Envpilot rate-limits pulls per token; abuse triggers `429` responses.

## License

MIT © [envpilot.dev](https://www.envpilot.dev)
