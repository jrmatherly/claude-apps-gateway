# Conventions

- Config via env vars everywhere; same image promoted across environments. Python settings in `admin-console/app/config.py` (`pydantic-settings`, case-insensitive). Gateway env names: `GATEWAY_PUBLIC_URL OIDC_ISSUER OIDC_CLIENT_ID OIDC_CLIENT_SECRET GATEWAY_JWT_SECRET GATEWAY_POSTGRES_URL GATEWAY_ADMIN_WRITE_KEY ADMIN_OKTA_GROUP_NAME AWS_REGION AVAILABLE_MODELS_RAW`. Console env: `SESSION_SECRET_KEY GATEWAY_BASE_URL PUBLIC_URL GATEWAY_SERVICE_ARN AWS_REGION`.
- Long module docstrings and inline comments carry the *why* (trust boundaries, Okta quirks, ECS Express Mode limits). Keep that style; do not strip rationale comments when editing.
- Bandit `# nosec` markers are deliberate (`B104` bind-all, `B105` session key name). Keep them.
- Admin console: one `GatewayClient` per request, bearer token from the session; errors surface as `GatewayError` / `GatewayModelsError` with user-facing `.message`, rendered via redirects with `?error=`.
- Admin group name is duplicated: `ADMIN_GROUP_NAME` constant in `admin-console/app/auth.py` (UX gate only) and `ADMIN_OKTA_GROUP_NAME` env on the gateway (real enforcement). Change both together.
- `AVAILABLE_MODELS_RAW` format is a YAML flow sequence string `[a, b, c]`; parsed by `_parse_available_models_raw` and written by `apply_model_selection`.
- CDK: one stack per file in `cdk/lib/`, props interfaces exported alongside; ECS Express Mode URLs are patched post-create by "URL fixer" custom resources (placeholder `https://placeholder.invalid`).
