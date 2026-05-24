# AGENTS.md

See `CLAUDE.md` for full architecture, environment variables, conventions, and deployment details.

## Cursor Cloud specific instructions

### Services

| Service | How to run | Notes |
|---------|-----------|-------|
| **Node.js Express server** | `npm run dev` | Single monolith process serving API, dashboard, landing pages, cron, and pass generation on port 3000 |
| **PostgreSQL** | `sudo pg_ctlcluster 16 main start` | Must be running before the server starts; schema auto-applies on boot via `getDb()` |

### Database setup (one-time, already done in VM snapshot)

A local PostgreSQL database `nudj_dev` with user `nudj` / password `nudj_dev_password` is pre-provisioned. The connection string is:

```
postgresql://nudj:nudj_dev_password@localhost:5432/nudj_dev
```

### Required environment variables for dev

```bash
export DATABASE_URL="postgresql://nudj:nudj_dev_password@localhost:5432/nudj_dev"
export JWT_SECRET="dev-secret-key-12345"
export NODE_ENV="development"
export CUSTOM_DOMAIN="localhost:3000"
export PASS_TYPE_IDENTIFIER="pass.com.nudj.dev"
export TEAM_IDENTIFIER="DEVTEAM123"
```

### Starting the dev server

```bash
sudo pg_ctlcluster 16 main start   # ensure PostgreSQL is running
npm run dev                         # node --watch src/server.js on port 3000
```

### Key URLs

- Dashboard: `http://localhost:3000/dashboard/`
- Health check: `http://localhost:3000/health`
- API base: `http://localhost:3000/api/v1`
- Debug sign test: `http://localhost:3000/debug/sign-test`

### Default admin credentials (auto-seeded on first boot)

- Email: `admin@ads2wallet.com`
- Password: `Ads2Wallet2026!`

### Gotchas

- **No ESLint / no test framework**: the codebase has no lint config or automated test suite. `src/engine/test.js` is legacy (written for the old SQLite API) and does not work with the current PostgreSQL db layer.
- **Schema migrations are inline**: DDL runs inside `getDb()` in `src/db/index.js` on every boot — there is no separate migration CLI.
- **Mock signing mode**: without valid Apple certificates the app still runs; pass files are generated but not installable on real iOS devices. The repo includes cert files in `certs/` which may or may not be valid.
- **Timezone**: the process sets `TZ=Europe/Rome` at startup; all cron and date logic uses Italian time.
- **node --watch**: the dev command uses Node's built-in `--watch` flag for hot reloading, but changes to `node_modules` or new dependency installs may require a manual restart.
