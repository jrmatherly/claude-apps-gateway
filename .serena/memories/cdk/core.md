# cdk

Entry `cdk/bin/app.ts`; seven stacks in `cdk/lib/*-stack.ts`, deploy order network → database → secrets → build-machine → gateway → admin-console → vpn. Context keys required at synth: `oidcIssuer`, `oidcClientId`, `oidcClientSecret`, `adminOktaGroupName`.

- `gateway-stack.ts`: `CfnExpressGatewayService` with env block (names in `mem:conventions`), secrets from Secrets Manager ARNs, default `AVAILABLE_MODELS_RAW='[claude-opus-4-8, claude-sonnet-4-6, claude-haiku-4-5]'`, and a custom resource that patches `GATEWAY_PUBLIC_URL` after the service URL exists. Includes the IAM grant for application inference profiles (docs/06).
- `admin-console-stack.ts`: same pattern; `GATEWAY_BASE_URL` wired from the gateway stack output; task role gets ECS describe/update on the gateway service and `bedrock:ListFoundationModels`.
- `build-machine-stack.ts`: temporary x86_64 EC2 that builds and pushes both images (local Apple Silicon builds are unusable on Fargate).
- `database-stack.ts`: Aurora Serverless v2 + Lambda (`lib/lambda/`) deriving the `postgres_url` secret.
- `vpn-stack.ts`: Client VPN with certs generated via `node-forge` at deploy; `.ovpn` delivered in Secrets Manager.
- No unit tests exist despite the jest devDependency.
