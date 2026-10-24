# Task completion checks

No CI, no test suite, no lint config exists in the repo. Minimum before calling a change done:

- Python (`admin-console/`): `cd admin-console && uv run --with-requirements requirements.txt python -c "import app.main"` (import-time settings validation needs `SESSION_SECRET_KEY` and `GATEWAY_BASE_URL` env set). Start it and hit `GET /healthz`.
- CDK (`cdk/`): `cd cdk && npm install && npx tsc --noEmit && npx cdk synth --all -c oidcIssuer=https://x.okta.com -c oidcClientId=x -c oidcClientSecret=x -c adminOktaGroupName=g` (synth needs no AWS credentials; `cdk.context.json` is gitignored).
- Gateway config (`gateway/gateway.yaml`): validate by booting `claude gateway --config` against the resolved file (see `mem:local_dev`); the gateway validates the full schema at boot and exits non-zero on error.
- Docs changed? Keep `docs/0N-*.md` consistent with `README.md` tables and env names in `cdk/lib/*-stack.ts`.
