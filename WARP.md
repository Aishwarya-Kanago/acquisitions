# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Commands

- Install deps: `npm install`
- Run in dev (auto-reload): `npm run dev` (loads env via `dotenv/config` from `src/index.js`)
- Lint: `npm run lint`
- Lint (auto-fix): `npm run lint:fix`
- Format: `npm run format`
- Format (check only): `npm run format:check`
- Database (Drizzle):
  - Generate migrations from models: `npm run db:generate`
  - Apply migrations: `npm run db:migrate`
  - Open Drizzle Studio: `npm run db:studio`
- Tests: no test runner is configured in `package.json` (add one before expecting test commands).

## Architecture overview

- Runtime and entrypoints
  - ESM Node app (see `type: "module"` in `package.json`) using subpath imports via `package.json#imports` (e.g., `#config/*`, `#services/*`).
  - `src/index.js` bootstraps env (`dotenv/config`) and imports `src/server.js`.
  - `src/server.js` binds the HTTP server (`app.listen`).

- Web layer
  - Express app (`src/app.js`) wires middleware: `helmet`, `cors`, `cookie-parser`, JSON/urlencoded parsers, and request logging (`morgan` -> `winston`).
  - Health endpoints: `/` (text), `/health` (JSON), `/api` (ping JSON).
  - Routes mounted under `/api/*`; current module: `src/routes/auth.routes.js` mounted at `/api/auth`.

- Controllers and validation
  - Example: `src/controllers/auth.controller.js#signup` validates input with Zod (`src/validations/auth.validation.js`), calls service, signs a JWT, and sets it as an HTTP-only cookie.

- Services and data access
  - Services (e.g., `src/services/auth.service.js`) encapsulate business logic and data access.
  - Database via Drizzle ORM over Neon HTTP: `src/config/database.js` exports `db` (Drizzle) and `sql` (Neon client).
  - Models defined with Drizzle pg-core (e.g., `src/models/user.model.js`).

- Utilities and config
  - JWT helpers in `src/utils/jwt.js` (sign/verify with `JWT_SECRET`).
  - Cookie helpers in `src/utils/cookies.js`.
  - Validation error formatter in `src/utils/format.js`.
  - Centralized logger in `src/config/logger.js` (Winston) with:
    - Console transport in non-production
    - File transports: `logs/error.log` and `logs/combined.log`

- Migrations
  - Drizzle configuration in `drizzle.config.js` (`schema: ./src/models/*.js`, output to `./drizzle`).
  - Generated SQL and metadata live under `drizzle/`.

## Environment

- Required/used variables (from `.env` and process env):
  - `PORT` (defaults to `3000` in `.env`)
  - `NODE_ENV` (e.g., `development` | `production`)
  - `LOG_LEVEL` (e.g., `info`)
  - `DATABASE_URL` (PostgreSQL connection string for Neon; used by Drizzle and the app)
  - `JWT_SECRET` (used by `src/utils/jwt.js`; if absent, a development default is used but should be overridden)

Notes
- Node must support ESM and `package.json#imports` (Node 18+ recommended).
- Path aliases via `package.json#imports` are resolved by Node at runtime; editors/tooling may need awareness but ESLint/Prettier are configured locally (`eslint.config.js`).
- Line endings: ESLint enforces Unix line breaks (`'linebreak-style': 'unix'`). On Windows, ensure your editor uses LF for this project or let Prettier handle normalization.
