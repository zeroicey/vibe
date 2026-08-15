You are a senior React frontend engineer. The current directory is an empty project. Initialize a complete, production-ready React frontend project here. Do NOT ask any questions — make reasonable decisions and build everything.

## Fixed Tech Stack (MANDATORY — do NOT recommend or substitute alternatives)
- Language: TypeScript (strict mode)
- Package manager: Bun
- Build tool: Vite + React 19
- Router: react-router
- Data fetching: TanStack React Query v5 + ky (HTTP client)
- Forms: react-hook-form + @hookform/resolvers
- Validation: Zod (shared by forms and API types)
- Styling: Tailwind CSS 4 (@tailwindcss/vite) + shadcn (base-nova style) + @base-ui/react + class-variance-authority + clsx + tailwind-merge + tw-animate-css
- UI helpers: lucide-react (icons), sonner (toasts), next-themes (dark mode)
- Dates: date-fns
- Client state: zustand (non-server state only)
- Testing: vitest + @testing-library/react + jsdom
- Linting & formatting: Biome
- Env management: Vite `import.meta.env` validated with Zod

## Folder Structure (follow exactly)

```
.
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── biome.json
├── components.json              # shadcn config
├── .env.example
├── .gitignore
├── README.md
└── src/
    ├── main.tsx                 # entry: mount App with providers
    ├── App.tsx                  # root: RouterProvider
    ├── api/                     # HTTP layer (the ONLY place that talks HTTP)
    │   ├── client.ts            # ky instance + apiUrl/resolveApiPath helpers
    │   ├── errors.ts            # ApiError (message/status/detail) + toDisplayError
    │   └── unwrap.ts            # parse unified envelope { success, message, data, error }, throw ApiError on failure
    ├── app/                     # app-level composition
    │   ├── router.tsx           # react-router route config (all routes declared here)
    │   ├── providers.tsx        # QueryClientProvider, theme provider, etc.
    │   ├── layout/              # app-navbar, app-layout, page-loading
    │   └── pages/               # welcome, not-found
    ├── components/
    │   ├── common/              # shared presentational components
    │   └── ui/                  # shadcn components (button, input, dialog, ...)
    ├── config/
    │   └── env.ts               # typed env with Zod (import.meta.env) — components never read import.meta.env directly
    ├── features/                # feature modules (one folder per domain)
    │   └── todos/
    │       ├── index.ts         # re-export
    │       ├── api.ts           # API calls (ky + unwrap)
    │       ├── queries.ts       # React Query hooks + query keys
    │       ├── schemas.ts       # Zod schemas (+ schemas.test.ts)
    │       ├── components/      # feature components
    │       └── pages/           # feature pages
    ├── hooks/                   # shared hooks
    ├── lib/                     # utilities (cn, etc.)
    ├── stores/                  # zustand stores (client-only state)
    ├── styles/
    │   └── globals.css          # Tailwind 4 global styles
    ├── test/                    # vitest setup.ts + helpers.tsx
    └── types/                   # shared types
```

## Architecture Rules (MANDATORY)
1. **Feature-first**: all business UI lives in `features/<name>/` — api.ts, queries.ts, schemas.ts, components/, pages/ colocated in one folder. NO cross-feature imports: shared code goes to components/common, hooks, lib
2. **Single HTTP layer**: only `src/api/` talks to the network. ky instance uses `throwHttpErrors: false` + `credentials: 'include'`; `unwrap()` parses the backend's unified envelope `{ success, message, data, error }` and throws `ApiError(message, status, detail)` on failure — feature api.ts files never see raw Response objects
3. **Server state only via React Query**: each feature's queries.ts exports query keys + hooks (`useQuery` / `useMutation`); mutations invalidate their keys and surface errors via sonner toasts. Components never call api.ts directly
4. **Validation**: Zod schemas in the feature's schemas.ts are the single source of truth — shared by react-hook-form resolvers and API request/response types
5. **Routing**: every route is declared in `app/router.tsx`; feature pages are mounted there; unknown paths fall back to the not-found page
6. **Styling**: Tailwind 4 utility classes + shadcn ui components only; `cn()` from lib/utils; global styles only in styles/globals.css — no per-component CSS files
7. **State split**: server state → React Query; client-only UI state (theme, sidebar, dialogs) → zustand stores; ephemeral local state → useState
8. **Tests**: vitest + @testing-library/react; component/page/schema tests colocated as `*.test.tsx` next to the code; shared setup in src/test/
9. **Env**: every env var is declared and validated in config/env.ts via Zod; components read from `env` only

## Deliverables
1. package.json with scripts: `dev`, `build`, `preview`, `typecheck`, `test`, `lint`, `format`
2. vite.config.ts (@vitejs/plugin-react, @tailwindcss/vite, `@` → `./src` alias), tsconfig.json (strict, paths), biome.json, components.json
3. index.html, .env.example (VITE_API_BASE_URL, etc.), .gitignore
4. Tailwind 4 globals.css + shadcn setup (base-nova style) with a few base ui components (button, input, dialog, sonner)
5. API layer (src/api/): client.ts, errors.ts, unwrap.ts — matching the backend unified response contract `{ success, message, data, error }`
6. app/: router.tsx (welcome page, /todos route, not-found), providers.tsx (QueryClient + theme), layout (app-layout + app-navbar)
7. One complete example feature (todos) wired through the full layers: schemas.ts → api.ts → queries.ts → components → pages, mounted in the router
8. Test setup (src/test/setup.ts + helpers.tsx) + one example component test for the todos feature
9. README.md with setup & run instructions

## Constraints
- Use ONLY the fixed stack. No Next.js, no axios, no Redux, no CSS modules, no plain fetch in components, no eslint+prettier
- Do NOT pin dependency versions — always install the latest stable release of every package, including new major versions (check for updates if the installed version is outdated)
- Everything must run with: `bun install && bun run dev`
- Output the complete file tree first, then every file's full content in ready-to-copy code blocks
- Keep code concise but production-quality (fully typed, proper error handling)
