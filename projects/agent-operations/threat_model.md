# Agent Operations security audit scope

## Project status and licensing

Agent Operations is an early-stage service for named external agent actions,
immutable approval, evidence-based verification and recovery after uncertain
writes. The server and dashboard are source-available under Elastic License 2.0;
the TypeScript and Python SDKs are Apache-2.0. The core is not represented as
OSI-approved open source. No independent adoption or production-scale claim is
made. Please assess program eligibility case by case before scheduling a scan.

## Intended trust boundaries

Read `SECURITY.md`, `docs/execution.md`, `docs/api.md`, `docs/packages.md` and
`docs/deployment.md`. Workspace administrators may publish operation definitions,
set credentials and grant roles. Provider responses, browser pages, model output,
API payloads, artifact identifiers, path segments and operation input are
untrusted. API-key roles and workspace scope must hold across all API routes and
artifact reads. The stdio MCP bridge may submit/read operations, not approve them
or administer a workspace. Approval must bind to the exact resolved mutation.

Credentials must not leak through responses, logs, action evidence, dashboard
rendering, screenshots or artifacts beyond the documented reader access. Connector
secret headers use the platform master key. A compromised host/database/platform
administrator is outside the confidentiality guarantee. The event hash chain is
not externally anchored or tamper-proof against database administrators.

## Priority code and attack surfaces

- `src/app.ts`, `src/store.ts`, `src/db.ts`: authorization, tenant isolation,
  sessions, API keys, SQL construction and persisted data.
- `src/network.ts`, `src/executors.ts`: exact-origin validation, DNS/address
  pinning, redirects, SSRF, response limits, browser restrictions and read-only
  verification. Configured `ALLOW_PRIVATE_NETWORK=true` is an explicit development
  exception; test default production restrictions separately.
- `src/engine.ts`, `src/domain.ts`: stable business identity, payload conflicts,
  leases/heartbeats, delayed workers, cancellation and uncertain writes.
- `src/github.ts`, `src/packages.ts`: fixed repository/provider scope, marker and
  receipt matching, hostile issue content, and missing/ambiguous verification.
- `src/artifacts.ts`, `web/`, SDKs and `src/mcp.ts`: artifact traversal/isolation,
  HTML injection, role boundaries, credential exposure and client validation.

Arbitrary shell/Python execution is not an operation feature. Model steps produce
schema-validated data. A Chromium context is not a VM/container boundary; the
documented deployment requires independent isolation and enforced egress rules.
Report concrete bypasses of implemented restrictions without assuming that a
trusted workspace administrator or a hostile browser engine is contained by them.

## Offline exercises

The image retains development dependencies, built outputs, Chromium headless
shell, the Python SDK and all source under `/src`. Tests use embedded PGlite by
default and local synthetic providers. No cloud/model/provider credentials are
needed for the following commands:

```sh
cd /src
pnpm typecheck
pnpm build
pnpm test
pnpm test:browser
pnpm test:e2e
```

`pnpm test:e2e` starts local demo servers on ports 4310 and 4311 and writes
synthetic `.data/e2e` state; clear that state between independent runs if needed.
Network PostgreSQL tests require a separately provisioned local server via
`TEST_DATABASE_URL`; none is supplied by this image, so independent-session
PostgreSQL behavior is not covered by its default test execution. Optional Python
framework integration suites have extra requirements and are not preinstalled;
the core service, SDKs and local browser workflows are the intended first scope.
Do not send requests to GitHub, real suppliers, live installations or paid models.

## Reports and severity

Provide source locations, attacker privileges, configuration prerequisites,
violated boundaries, an offline reproducer, and a minimal patch/regression test.
Cross-workspace secret access or unauthorized mutations can be high/critical when
demonstrated; explain the actual impact. Distinguish trusted-admin configuration,
documented demo/development behavior and missing deployment isolation from an
application vulnerability. Unknown outcomes must not silently become success or
permission for blind redispatch. Liveness/resource exhaustion should be assessed
according to realistic attacker access and affected deployment scope.
