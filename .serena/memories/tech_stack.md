# Tech stack

- Gateway: Claude Code binary, `claude gateway --config <yaml>`. Image pins `CLAUDE_CODE_VERSION=2.1.204` (`gateway/Dockerfile`, bump deliberately). Base `debian:12-slim`, linux-x64 binary, GPG + SHA256 verified at build. Server runs only on the Linux binary (docs); build with `--platform linux/amd64`.
- Admin console: Python 3.12 (image), FastAPI 0.139 + Starlette, Jinja2 templates, httpx, pydantic-settings, boto3. Pins in `admin-console/requirements.txt`. No lockfile, no pyproject, no tests, no linter config.
- CDK: TypeScript ~5.7, `aws-cdk-lib` 2.261.0 pinned, `aws-cdk` CLI 2.1130.0, Node 20+. `cdk/package.json` has `build` (tsc), `test` (jest, no tests exist), `cdk`.
- Infra targets: ECS Express Mode (`CfnExpressGatewayService` L1), Aurora Serverless v2 Postgres, Secrets Manager, Client VPN, a throwaway EC2 build machine (because Apple Silicon local builds produce non-Fargate images).
- Local machine (Darwin): Docker via OrbStack (amd64 emulation available), `postgres:18.3` image present, Claude Code CLI at `~/.local/bin/claude`, Python via mise, no local `psql`.
