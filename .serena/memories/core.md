# claude-apps-gateway: source map

Reference deployment of Anthropic's Claude apps gateway on AWS (Bedrock upstream) plus a FastAPI admin console. Three independent codebases in one repo, no shared build:

- `gateway/` — container image only (`Dockerfile`, `entrypoint.sh`, `gateway.yaml`). No app code; the server is the `claude gateway` subcommand of the pinned Claude Code binary. Config is `${ENV}`-expanded YAML; `AVAILABLE_MODELS_RAW` is substituted by `entrypoint.sh` with `sed` because the gateway's own `${VAR}` expansion cannot fill a YAML list position.
- `admin-console/` — Python FastAPI app. See `mem:admin-console/core` for auth model, routes, and which routes need AWS.
- `cdk/` — TypeScript CDK app, 7 stacks. See `mem:cdk/core`.
- `docs/01..06-*.md` — deploy/verify runbooks; they are the only user-facing docs and mirror real behaviour (gotchas there were hit in a real account).

Invariants:
- Secrets never enter the repo: `.gitignore` blocks `*.ovpn *.pem *.key SPEC.md private/ .remember/ .codegraph/`.
- Gateway requires Postgres and an OIDC issuer at boot (schema validated, crash-loops otherwise); there is no auth bypass.
- Admin writes go through the signed-in admin's gateway JWT (audits as `oidc:<sub>`), never a static admin key. Only model-catalog changes use AWS IAM (ECS service update).
- Local, no-AWS development: see `mem:local_dev`.
- Stack, commands, conventions, done-criteria: `mem:tech_stack`, `mem:suggested_commands`, `mem:conventions`, `mem:task_completion`.
