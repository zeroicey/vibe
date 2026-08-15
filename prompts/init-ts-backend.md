You are a senior TypeScript backend engineer. The current directory is an empty project. Initialize a complete, production-ready backend project here. Do NOT ask any questions — make reasonable decisions and build everything.

## Fixed Tech Stack (MANDATORY — do NOT recommend or substitute alternatives)
- Language: TypeScript (strict mode)
- Runtime: Bun
- Web framework: Hono
- Database: PostgreSQL
- ORM: Drizzle ORM + drizzle-kit (migrations)
- Validation: Zod
- Request validation: @hono/zod-validator
- Testing: Bun built-in test runner (`bun test`)
- Linting & formatting: Biome
- Env management: Bun native `process.env` (no dotenv)
- Logging: pino + pino-http
- Security middleware: `hono-rate-limiter` (MemoryStore), `hono/secure-headers`, `hono/cors`, `hono/csrf`, `hono/body-limit`, `hono/timeout`

## Folder Structure (follow exactly)

```
.
├── package.json
├── tsconfig.json
├── biome.json
├── drizzle.config.ts
├── drizzle/                    # generated migrations
├── .env.example
├── .gitignore
├── Dockerfile                 # production image: oven/bun:1, non-root, HEALTHCHECK
├── .dockerignore
├── README.md
└── src/
    ├── index.ts                # entry: create app + start server
    ├── app.ts                  # build Hono app: security middleware, global onError, mount module routers
    ├── env.ts                  # typed env loading with Zod
    ├── db/
    │   ├── connection.ts       # Drizzle client
    │   └── schema.ts           # all table definitions (centralized)
    ├── middleware/
    │   ├── index.ts            # middleware aggregator
    │   └── security.ts         # rate limiter, secure-headers, cors, csrf, body-limit, timeout
    ├── shared/                 # cross-cutting utilities
    │   ├── errors.ts           # AppError (code + status + details) + error codes
    │   ├── error-handler.ts    # onError: AppError/ZodError/SyntaxError → unified response
    │   ├── validator.ts        # zod-validator wrapper: body/query/params, ZodError → AppError(VALIDATION)
    │   ├── response.ts         # Res builder: ok/error + shortcuts, omits unset fields
    │   ├── messages.ts         # Msg: predefined message constants
    │   └── logger.ts           # pino instance
    └── modules/                # business modules (one folder per domain)
        ├── health/
        │   ├── health.router.ts
        │   └── health.handler.ts
        └── todos/
            ├── index.ts        # re-export router
            ├── todos.router.ts # RESTful routes + zValidator middleware → handler
            ├── todos.handler.ts# reads c.req.valid('json'|'query'|'param'), no try/catch, throws AppError
            ├── todos.service.ts# business logic
            ├── todos.schema.ts # Zod schemas (body/query/params)
            ├── todos.types.ts  # inferred types
            ├── todos.mappers.ts# DB ↔ domain mapping
            └── todos.service.test.ts
```

## Architecture Rules (MANDATORY)
1. **Validation at route layer**: every route declares `zValidator('json'|'query'|'param', schema)` via the shared `validator` wrapper — never parse manually inside handlers
2. **Handlers are thin**: they read typed data via `c.req.valid(...)`, call the service, return a response. NO try/catch in handlers — throw `AppError` and let the global onError map it to the unified response format
3. **Unified error contract**: `shared/validator.ts` converts ZodError → AppError(VALIDATION, 400, issues); `shared/error-handler.ts` exports onError (wired in app.ts) mapping AppError → status from the error, SyntaxError → 400, unknown errors → 500 (logged, never leak stack)
4. **Global middleware first**: security middleware + pino-http logging registered in app.ts before routers
5. **Unified response contract** (every API response follows this shape):
   - `{ success: boolean, message: string, code?: ErrorCode, data?: T, error?: unknown }`
   - `success` + `message` always present; `code` / `data` / `error` are OMITTED when unset — use `undefined`, never `null`
   - Success: `return Res.ok(msg, data).build(c)`; error: `throw new AppError(code, msg, status)` → handled by onError
   - HTTP status is the transport signal (semantic: 201 created, 404 not found, 409 conflict, 429 rate limited); `success` is the app-level signal; `code` is the business error identifier for programmatic handling — do not duplicate status as a body field
   - Messages: literal string or predefined `Msg` constant from shared/messages.ts — no scattered hardcoded strings
   - `Res` shortcuts: ok / created / noContent / badRequest / unauthorized / forbidden / notFound / conflict / internalError
6. **Docker / deployment**: the image runs the app only — migrations are NEVER auto-run at container start; run `bun run db:migrate` separately in the deploy flow. No entrypoint script: DATABASE_URL and all secrets are injected as environment variables at deploy time. No docker-compose

## Deliverables
1. package.json with scripts: `dev`, `build`, `start`, `test`, `db:generate`, `db:migrate`
2. tsconfig.json (strict, with `@/*` → `./src/*` path alias), biome.json, drizzle.config.ts
3. .env.example with all env vars (DATABASE_URL, PORT, etc.), .gitignore
4. Initial schema: `users` and `todos` tables in src/db/schema.ts + first migration in drizzle/
5. `GET /health` endpoint (modules/health)
6. One complete example module (todos) wired through the full layers: router (zValidator) → handler (c.req.valid) → service → db, covering json body, query params and a `:id` param
7. Global onError (shared/error-handler.ts) mapping AppError/ZodError/SyntaxError → unified response format, wired in app.ts
8. Security middleware (rate limiter, secure-headers, cors, csrf, body-limit, timeout) wired in app.ts
9. pino + pino-http request logging (shared/logger.ts + request logging middleware)
10. Unified response layer per the response contract: Res builder (shared/response.ts) + AppError (shared/errors.ts) + Msg constants (shared/messages.ts), demonstrated in the todos module
11. Dockerfile: single-stage `oven/bun:1`, TZ=Asia/Shanghai (+ tzdata), non-root user UID 10001, `bun install --omit=dev`, HEALTHCHECK on /health, `CMD ["bun", "run", "start"]` — no entrypoint script, no compose
12. .dockerignore: exclude node_modules, .env, .git, coverage — keep `drizzle/` in the image (needed for db:migrate at deploy time)
13. README.md with setup & run instructions: dev (`bun run dev`), docker build & run (`docker run -p 3000:3000 -e DATABASE_URL=... <image>`), deploy flow (build image → run db:migrate against the target DB → start container). Note: docker build behind a restricted network may need proxy build-args for bun install

## Constraints
- Use ONLY the fixed stack. No Express/Fastify, no Prisma, no eslint+prettier
- Do NOT pin dependency versions — always install the latest stable release of every package, including new major versions (check for updates if the installed version is outdated)
- Everything must run with: `bun install && bun run dev`
- Use environment variables for all configuration
- Output the complete file tree first, then every file's full content in ready-to-copy code blocks
- Keep code concise but production-quality (proper error handling, fully typed)
