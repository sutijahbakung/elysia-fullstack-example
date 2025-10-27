# Copilot Agent Instructions for Elysia Fullstack Example

## Repository Overview

This is a **fullstack web application** built with **Elysia** (Bun web framework), **React 19**, and **TailwindCSS v4**. The project uses **Bun runtime** (not Node.js) for both development and production. The repository is small (~200KB of source code) with a simple, flat structure.

**Key Technologies:**
- Runtime: Bun 1.3+ (required - Node.js will not work)
- Backend: Elysia latest (TypeScript-first web framework)
- Frontend: React 19.2.0 with React DOM 19.2.0
- Styling: TailwindCSS 4.1.14 with bun-plugin-tailwind
- Type Safety: TypeScript with strict mode enabled
- API Client: @elysiajs/eden with @tanstack/react-query
- API Documentation: @elysiajs/openapi

**Project Purpose:** Demonstrates a fullstack Bun/Elysia application with static file serving, React frontend pages, and API endpoints with type-safe client-server communication.

## Critical Prerequisites

### Bun Runtime Installation (REQUIRED)
This project **requires Bun runtime**. If `bun` command is not available, install it first:

```bash
curl -fsSL https://bun.sh/install | bash
source ~/.bash_profile  # or source ~/.bashrc
```

Verify installation: `bun --version` (should show 1.3.1 or higher)

**Important:** Always ensure `~/.bun/bin` is in your PATH before running any commands. If needed:
```bash
export PATH="$HOME/.bun/bin:$PATH"
```

## Build & Development Workflow

### Initial Setup (Clean Environment)
**Always run these commands in this exact order when starting fresh:**

```bash
# 1. Install dependencies (takes ~100ms with cache, ~4s without)
bun install

# Expected: "39 packages installed" with no errors
```

### Development Server
```bash
# Start dev server with hot reload
bun run dev

# Expected output: "🦊 Elysia is running at localhost:3000"
# Server starts in <1 second
# Access: http://localhost:3000/ (main page) and http://localhost:3000/other (API demo page)
```

**Note:** The dev server uses `--watch` flag for automatic reload on file changes.

### Production Build
```bash
# Compile standalone binary (takes ~120ms)
bun run build

# Expected: Creates executable file named "server" (102MB) in repo root
# Output: "[43ms] bundle 284 modules" and "[68ms] compile server"
```

The compiled `server` binary is added to `.gitignore` and should not be committed.

### Validation Commands

**TypeScript Type Checking:**
```bash
bunx tsc --noEmit

# Expected: No output = success (takes ~5-10 seconds first run, <2s cached)
# This checks all .ts and .tsx files according to tsconfig.json
```

**Code Formatting (Prettier):**
```bash
bunx prettier --check "src/**/*.ts" "public/**/*.tsx" "public/**/*.ts"

# Expected: "All matched files use Prettier code style!"
# Config: .prettierrc (tabs, no semicolons, single quotes)
```

**Format code automatically:**
```bash
bunx prettier --write "src/**/*.ts" "public/**/*.tsx" "public/**/*.ts"
```

### Testing
**Important:** This repository has NO test suite configured. The `bun test` command will report "No tests found!" and exit with code 1. This is expected behavior - do not try to fix it or add tests unless specifically requested.

### Full Clean Build Workflow
When you need to verify everything works from scratch:

```bash
# 1. Clean (optional but recommended after major changes)
rm -f server
rm -rf node_modules

# 2. Install dependencies
bun install

# 3. Type check
bunx tsc --noEmit

# 4. Format check
bunx prettier --check "src/**/*.ts" "public/**/*.tsx" "public/**/*.ts"

# 5. Build
bun run build

# Total time: ~15-20 seconds
```

## Project Structure

### Root Directory Files
```
├── .gitignore          # Excludes: node_modules, server binary, *.bun, build artifacts
├── .prettierrc         # Prettier config: tabs, no semicolons, single quotes
├── bunfig.toml         # Bun config: enables bun-plugin-tailwind for static serving
├── package.json        # Dependencies and scripts (dev, build, test)
├── tsconfig.json       # TypeScript config with path aliases (@server, @public)
├── bun.lock           # Bun lockfile
└── README.md          # Basic getting started instructions
```

### Source Code Structure

**Backend (`/src`):**
- `src/index.ts` - Main server file (22 lines)
  - Exports `app` instance for type inference
  - Configures OpenAPI documentation
  - Serves static files from `/public` directory
  - Defines API routes (e.g., `/message`)
  - Listens on port 3000

**Frontend (`/public`):**
- `public/index.html` - Root page HTML shell
- `public/index.tsx` - Root page React app (counter demo)
- `public/other/index.html` - Second page HTML shell
- `public/other/index.tsx` - Second page React app (API call demo with @tanstack/react-query)
- `public/layouts/index.tsx` - Shared layout component with QueryClientProvider
- `public/libs/api.ts` - Type-safe API client using @elysiajs/eden and treaty
- `public/styles/global.css` - TailwindCSS import (`@import 'tailwindcss';`)
- `public/images/` - Static images (WebP format)

### TypeScript Path Aliases
Configured in `tsconfig.json` - use these imports:
- `@server` → `./src/index.ts`
- `@server/*` → `./src/*`
- `@public/*` → `./public/*`

Example: `import { api } from '@public/libs/api'` or `import type { app } from '@server'`

## Key Architectural Patterns

### Type-Safe Client-Server Communication
The project uses **@elysiajs/eden** with the `treaty` client for end-to-end type safety:

1. Server exports `app` instance in `src/index.ts`
2. Client imports the app type in `public/libs/api.ts`
3. `treaty<typeof app>('localhost:3000')` provides fully typed API client
4. React Query hooks use the typed client for data fetching

**Example:** `api.message.get()` is fully typed based on server route definition.

### Static File Serving
- `@elysiajs/static` plugin serves all files from `/public` directory
- HTML files are entry points (e.g., `/index.html`, `/other/index.html`)
- TypeScript/TSX files are transpiled on-the-fly by Bun
- Tailwind CSS is processed via `bun-plugin-tailwind` (configured in `bunfig.toml`)

### React Pages
Each page has its own HTML entry point that loads a corresponding `.tsx` file with `<script type="module">`. Bun handles TSX transpilation and module loading automatically.

## Common Issues & Solutions

### Build Command Failures
- **Error:** `bun: command not found`
  - **Solution:** Install Bun runtime (see Prerequisites section)

- **Error:** Build hangs or times out
  - **Solution:** Not a known issue - builds complete in <200ms. If this occurs, check system resources.

### Development Server Issues
- **Port 3000 already in use**
  - **Solution:** Kill existing process or change port in `src/index.ts` `.listen()` call

### TypeScript Errors
- **Cannot find module '@server'**
  - **Solution:** Ensure you're running commands from repo root where `tsconfig.json` exists
  
- **Type errors after adding new routes**
  - **Solution:** The API client in `public/libs/api.ts` auto-updates types from server. Restart TypeScript server if using IDE.

### Formatting Issues
- **Files don't match Prettier style**
  - **Solution:** Run `bunx prettier --write` on the affected files
  - **Config:** Use tabs (width 4), no semicolons, single quotes, no trailing commas

## Making Code Changes

### Adding New API Endpoints
1. Add route in `src/index.ts` after existing routes
2. Export responses as `const` for type inference (e.g., `{ data: 'value' } as const`)
3. TypeScript types automatically propagate to client via `treaty`
4. Use `api.yourRoute.get()` or `.post()` in React components

### Adding New Pages
1. Create directory under `/public` (e.g., `/public/newpage`)
2. Add `index.html` (copy from existing page, update title)
3. Add `index.tsx` with React component
4. Access at `http://localhost:3000/newpage/`

### Modifying Existing Code
1. Make changes to source files
2. Run `bunx tsc --noEmit` to verify types
3. Run `bunx prettier --check` (and `--write` if needed)
4. Test with `bun run dev` and verify in browser
5. Build with `bun run build` to ensure production binary compiles

## Performance Notes

- **Dependency installation:** ~100ms (cached) to ~4s (first time)
- **Development server startup:** <1 second
- **Production build:** ~120ms for bundling + compilation
- **TypeScript checking:** ~2-10 seconds depending on cache
- **Prettier checking:** ~1-2 seconds

## Additional Validation

- **No CI/CD:** This repository has no GitHub Actions workflows or automated checks
- **No linting:** Only Prettier for formatting, no ESLint or other linters
- **No pre-commit hooks:** No Git hooks configured

## When Instructions Are Incomplete

These instructions cover the complete build, development, and validation workflow for this repository. Only search for additional information or explore the codebase if:
1. You encounter errors not documented here
2. You need to understand specific implementation details for a feature change
3. The instructions contain incorrect information (please report this to the user)

For routine tasks (building, testing, adding features, formatting), **trust these instructions** and follow them exactly to minimize exploration time and avoid command failures.
