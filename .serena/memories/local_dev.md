# Local development without AWS (validated 2026-10-24 on macOS, Claude Code 2.1.310)

The gateway runs natively on macOS (`claude gateway --config`); no gateway image build needed locally. Docs: "macOS works for local development". Needs three things, none of them AWS:

1. Postgres: `docker run -d --rm --name cag-pg -e POSTGRES_USER=gw -e POSTGRES_PASSWORD=pw -e POSTGRES_DB=gateway -p 15432:5432 postgres:18.3`
2. A loopback OIDC issuer. Dex (`ghcr.io/dexidp/dex`) with `connectors: [{type: mockCallback}]` and a `staticClients` entry whose `redirectURIs` is `http://127.0.0.1:<gw-port>/oauth/callback`. Mock user is `kilgore@kilgore.trout`, groups `[authors]`. Gateway env must include `CLAUDE_GATEWAY_ALLOW_LOOPBACK=1` or the SSRF guard rejects the loopback issuer.
3. Upstream that never calls AWS: append `load_test_mode: {enabled: true, reply_seconds: 0}` to the resolved config (needs gateway >= 2.1.282). The `bedrock` upstream in `gateway/gateway.yaml` still resolves credentials per request, so set placeholder `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`; without them each request spends 60 s probing IMDS and returns 502. Alternative: swap `upstreams` to `provider: anthropic` with a real `ANTHROPIC_API_KEY` for real inference.

Resolve the repo's own `gateway/gateway.yaml` the way `entrypoint.sh` does (`sed` on `${AVAILABLE_MODELS_RAW}`), set `listen.host: 127.0.0.1` (then `public_url` is optional and plain http is accepted by the CLI for loopback hosts), export the env names listed in `mem:conventions`, and run. Schema gotchas hit: `admin.write_keys[].key` >= 32 chars; `session.jwt_secret` >= 32 bytes. Rotating `GATEWAY_JWT_SECRET` invalidates every issued token.

Admin console locally: `cd admin-console && SESSION_SECRET_KEY=$(openssl rand -hex 32) GATEWAY_BASE_URL=http://127.0.0.1:18080 PUBLIC_URL=http://127.0.0.1:18090 PORT=18090 uv run --python 3.12 --with-requirements requirements.txt python -m app.main`. Sign-in, spend limits, audit work with no AWS. `/models` needs ECS + Bedrock and shows the caught `GatewayModelsError` instead.

Known gap: `ADMIN_GROUP_NAME` in `admin-console/app/auth.py` is hardcoded `claude-gateway-admins`; the Dex mock user carries `authors`, so the console routes to `/not-authorized` even though the gateway (`admin_groups: [authors]`) accepts the token. Make the constant env-driven to use the console as admin locally.

Worked scratch files from the validation run (dex.yaml, gateway-resolved.yaml) lived in the session scratchpad; nothing was committed.
