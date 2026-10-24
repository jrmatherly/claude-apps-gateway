# Commands

Local no-AWS stack: full recipe in `mem:local_dev`.

- Gateway: `CLAUDE_GATEWAY_ALLOW_LOOPBACK=1 claude gateway --config <resolved.yaml>`; `CLAUDE_GATEWAY_LOG_LEVEL=debug` for id_token claim names. Probes: `GET /healthz` (`ok`), `GET /readyz`, `GET /.well-known/oauth-authorization-server`, `POST /oauth/device_authorization`.
- Admin API without a browser: `curl -H "x-api-key: $GATEWAY_ADMIN_WRITE_KEY" <gw>/v1/organizations/spend_limits`.
- Admin console: `cd admin-console && uv run --python 3.12 --with-requirements requirements.txt python -m app.main` (env in `mem:local_dev`). No venv is checked in; `.venv/` is gitignored.
- CDK: `cd cdk && npm install && npx cdk synth --all -c oidcIssuer=... -c oidcClientId=... -c oidcClientSecret=... -c adminOktaGroupName=...`. Deploy only on explicit request; it bills NAT, Aurora, 2 ALBs.
- Docker on this Mac is OrbStack; build gateway image with `docker build --platform linux/amd64 gateway/` (binary is linux-x64 only).
- Darwin specifics: `sed -i ''` (BSD sed), no `psql` installed (use `docker exec cag-pg psql -U gw gateway`), `timeout` exists via coreutils.
