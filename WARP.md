# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a Tauri desktop application with a SvelteKit frontend. The project uses:
- **Frontend**: SvelteKit 2.x with Svelte 5.x, TypeScript, and Vite
- **Backend**: Tauri 2.x (Rust-based native backend)
- **Architecture**: Single Page Application (SPA) mode using `@sveltejs/adapter-static`

## Development Commands

### Running the Application
```bash
# Development mode (runs both Vite dev server and Tauri)
npm run tauri dev

# Frontend only (for UI development without Tauri)
npm run dev
```

### Building
```bash
# Build frontend for production
npm run build

# Build the full Tauri application bundle
npm run tauri build
```

### Type Checking and Validation
```bash
# Type check with svelte-check (run once)
npm run check

# Type check in watch mode
npm run check:watch
```

### Preview
```bash
# Preview production build locally
npm run preview
```

## Architecture

### Frontend (src/)
- **Framework**: SvelteKit in SPA mode (no SSR due to Tauri constraints)
- **Routes**: File-based routing in `src/routes/`
- **Layout**: Uses SvelteKit's layout system (`+layout.ts`, `+page.svelte`)
- **State Management**: Svelte 5 runes (`$state`, `$derived`, etc.)

### Backend (src-tauri/)
- **Entry Point**: `src-tauri/src/main.rs` (calls `lib.rs::run()`)
- **Command Handlers**: Define Tauri commands in `src-tauri/src/lib.rs`
- **Capabilities**: Security permissions defined in `src-tauri/capabilities/default.json`
- **Configuration**: `src-tauri/tauri.conf.json` controls app metadata, window settings, and bundle configuration

### Frontend-Backend Communication
- Use `invoke()` from `@tauri-apps/api/core` to call Rust commands from frontend
- Rust commands use the `#[tauri::command]` macro
- Register commands in `lib.rs` using `invoke_handler(tauri::generate_handler![...])`

### Build Process
1. **Development**: `npm run dev` starts Vite dev server on port 1420 (fixed)
2. **Production**: `npm run build` generates static files in `build/` directory
3. **Tauri Bundle**: Reads frontend from `../build` and packages with Rust binary

## Key Configuration Files

- **vite.config.js**: Configured for Tauri with fixed port (1420) and HMR
- **svelte.config.js**: Uses `adapter-static` with `fallback: "index.html"` for SPA mode
- **tsconfig.json**: Extends SvelteKit's generated config, strict mode enabled
- **tauri.conf.json**: Defines app identifier, window properties, and bundle settings
- **Cargo.toml**: Rust dependencies including `tauri`, `tauri-plugin-opener`, `serde`

## Important Notes

- **No SSR**: This is an SPA because Tauri doesn't support Node.js server-side rendering
- **Port 1420**: Vite dev server uses a fixed port required by Tauri (strictPort: true)
- **Vite ignores src-tauri**: Frontend watcher explicitly ignores Rust changes to avoid conflicts
- **Rust Library Name**: The lib name is `speedtest_menubar_app_lib` (with underscore) to avoid Windows naming conflicts
- **Package Manager**: No lock file detected - ensure consistency in your environment (npm is configured in package.json scripts)

## Adding New Features

### Adding a Rust Command
1. Define function with `#[tauri::command]` in `src-tauri/src/lib.rs`
2. Add to handler: `invoke_handler(tauri::generate_handler![your_command])`
3. Call from frontend: `await invoke("your_command", { args })`

### Adding Frontend Routes
1. Create `+page.svelte` in `src/routes/your-route/`
2. Optional: Add `+layout.ts` or `+layout.svelte` for shared layouts
3. Use SvelteKit's file-based routing conventions

### Adding Tauri Plugins
1. Add plugin to `Cargo.toml` (e.g., `tauri-plugin-*`)
2. Register in `lib.rs`: `.plugin(tauri_plugin_name::init())`
3. Add required permissions to `src-tauri/capabilities/default.json`

## TypeScript Configuration

- Uses SvelteKit's generated `tsconfig.json` as base
- Path aliases are handled by SvelteKit configuration
- `$lib` alias points to `src/lib` (standard SvelteKit convention)
