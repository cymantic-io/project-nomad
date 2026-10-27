# Admin runtime migration

The admin application now requires Node.js 24 or newer. Its production Docker
base uses `node:24-slim`. The framework upgrade includes AdonisJS 7, the matching
framework integrations, Inertia 3 and Vite 8.

## Compatibility changes

- Encryption uses the legacy driver with the existing `APP_KEY`, preserving
  compatibility with previously encrypted data. Keep the existing key and
  persistent storage when upgrading.
- Inertia shared props live in `InertiaMiddleware`. The assistant-name cache is
  a separate service so changing the configured name still invalidates it
  without importing HTTP middleware from application configuration.
- The client entrypoint is `inertia/app.tsx`. Vite deduplicates React because
  the Transmit React wrapper also depends on React 18.
- MapLibre 6 uses namespace imports and an explicit Vite worker URL. It requires
  WebGL 2 in the client browser.
- Drug-data ingestion uses stream-json 3's Node stream adapters. The data
  pipeline still uses Node streams and preserves records split across chunks.
- Exact-version npm lifecycle-script approvals cover the native dependencies.
  Review and refresh these approvals when updating packages that need an
  installation script. Tesseract's optional postinstall remains disabled.
- The backend compiler excludes tests that import frontend modules across the
  Inertia TypeScript project boundary. Tests must be run separately.

## Verification

From `admin/`, with Node 24 and the normal build prerequisites:

```sh
npm ci
npm run build
npm run test:dependencies
npm audit --omit=dev
npm audit
```

The dependency smoke command checks the proxy trust regression, streamed JSON
record preservation, tar round trips, and native sharp, SQLite and ZIM loading.
It uses temporary files and an in-memory SQLite database.

The original migration was exercised on Linux ARM64 with an isolated copy of
an existing MySQL database: migrations, seeding, offline map rendering, Inertia
navigation, and a complete local chat stream. The pre-migration and migrated
compiled unit runs both passed 650 plain tests and 76 of 80 booted framework
tests. The same four failures concern drug-label whitespace and mocked chat
controller expectations.

The contribution adds an assistant-name cache regression test, which passes;
its booted run is 77 of 81 with the same four existing failures.

The source/JIT test runner's bootstrap import needs further investigation.
Those comparison runs compiled the test sources to JavaScript in disposable
containers before running the existing test entrypoint. AMD64 and the source
test-runner path still need validation before this migration is merge-ready.
