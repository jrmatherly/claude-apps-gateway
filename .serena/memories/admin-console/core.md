# admin-console

FastAPI app, entry `app/main.py` (`python -m app.main`, uvicorn on `settings.host:port`, default 8080).

- `app/config.py`: `Settings` (pydantic-settings). Required env: `SESSION_SECRET_KEY`, `GATEWAY_BASE_URL`. Optional: `PUBLIC_URL` (https prefix turns on secure cookies), `GATEWAY_SERVICE_ARN`, `AWS_REGION`, `PORT`, `HOST`.
- `app/auth.py`: Starlette `SessionMiddleware` session holds the gateway JWT; payload decoded unverified for display only. `ADMIN_GROUP_NAME` constant gates UI pages; the gateway enforces for real.
- `app/gateway_client.py`: `GatewayClient` over the gateway HTTP API: device flow (`/oauth/device_authorization`, `/oauth/token`), spend limits CRUD, `/effective`, `/audit`, `/v1/models`.
- `app/gateway_models.py`: the only AWS-dependent module (boto3 `ecs` + `bedrock`). Reads/writes `AVAILABLE_MODELS_RAW` on the ECS Express Gateway Service; lists Bedrock Anthropic catalog. Errors are `GatewayModelsError`, caught by the `/models*` routes.
- Routes: `/healthz`, `/signin*`, `/signout`, `/not-authorized`, `/`, `/spend/dashboard`, `/spend/limits*`, `/spend/audit` (gateway only); `/models`, `/models/apply`, `/models/status` (AWS).
- `_pending_devices` is an in-process dict; single instance assumed.
- Templates `app/templates/*.html` (Jinja2, `base.html` nav), static CSS under `app/static/css`.
