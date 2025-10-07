# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PocketBase is an open-source Go backend framework that includes an embedded SQLite database with realtime subscriptions, built-in files and users management, an Admin dashboard UI, and a REST-ish API. It can be used as a standalone executable or as a Go library/framework.

## Development Commands

### Go Backend Development

```bash
# Run tests
go test ./...
# or
make test

# Run tests with coverage report
make test-report

# Run linter (requires golangci-lint)
golangci-lint run -c ./golangci.yml ./...
# or
make lint

# Generate JavaScript types for jsvm plugin
make jstypes
# or
go run ./plugins/jsvm/internal/types/types.go

# Build and run the base example
cd examples/base
go build
./base serve

# Run directly with go run
cd examples/base
go run main.go serve
```

### Admin UI Development

The Admin UI is a Svelte/Vite SPA located in the `ui/` directory:

```bash
cd ui

# Install dependencies
npm install

# Start dev server (runs at http://localhost:3000)
npm run dev

# Build for production (output to ui/dist)
npm run build

# Preview production build
npm preview
```

**Important:** The Admin UI expects the backend server to run at `http://localhost:8090` by default. To change this, create `ui/.env.development.local` with `PB_BACKEND_URL=YOUR_ADDRESS`.

After making UI changes, always run `npm run build` to update the embedded UI in `ui/dist` before committing.

## Architecture

### Core Packages

- **`core/`** - The backbone of PocketBase containing:
  - `App` interface and `BaseApp` implementation
  - Database models (Collection, Record, AuthOrigin, MFA)
  - Field types (text, number, password, file, url, etc.)
  - Event system and hooks
  - Database query builders and validators
  - Collection-to-table synchronization logic

- **`apis/`** - HTTP API handlers:
  - Record CRUD operations
  - Authentication endpoints (password reset, email change, OTP, OAuth2)
  - Realtime subscriptions (WebSocket)
  - File upload/download
  - Backup management
  - Admin dashboard endpoints
  - Middlewares (CORS, rate limiting, body limit, gzip)

- **`forms/`** - Request validation forms for API endpoints

- **`plugins/`** - Official plugins:
  - `jsvm` - JavaScript VM for extending PocketBase with JavaScript hooks
  - `migratecmd` - Migration command utilities
  - `ghupdate` - GitHub auto-update functionality

- **`tools/`** - Utility packages (filesystem, mailer, cron, subscriptions, etc.)

- **`migrations/`** - System database migrations

### Application Flow

1. **Initialization**: `pocketbase.New()` or `pocketbase.NewWithConfig()` creates an app instance
2. **Bootstrap**: Called automatically by `app.Start()` or manually with `app.Bootstrap()`
   - Creates data directory (default: `./pb_data`)
   - Opens database connections (data.db and aux.db)
   - Runs migrations
   - Loads settings
3. **Event Hooks**: Register hooks using `app.OnServe()`, `app.OnRecordCreate()`, etc.
4. **Start**: `app.Start()` boots the app and starts the HTTP server

### Key Concepts

- **Collections**: Define data schemas and record types (base, auth, view)
- **Records**: Individual data entries within collections
- **Fields**: Typed schema fields with validation (text, number, file, relation, etc.)
- **Events**: Hook system for intercepting app lifecycle and CRUD operations
- **Realtime**: WebSocket subscriptions for live data updates
- **JSVM Plugin**: Allows extending PocketBase with JavaScript code in `pb_hooks/` directory

## Testing

- Uses Go's standard `testing` package
- Mix of unit and integration tests
- Run specific package tests: `go test ./core/...`
- Run single test: `go test -run TestName ./package`

## Build Configuration

- **Minimum Go version**: 1.23+ (set in go.mod as 1.24.0)
- **Build constraints**: Pure Go SQLite driver supports specific platforms (see README.md)
- **Static binary**: Build with `CGO_ENABLED=0 go build`
- **Cross-compilation**: Use `GOOS` and `GOARCH` environment variables

## Important Notes

- Prebuilt executables are based on `examples/base/main.go`
- The `examples/base` includes jsvm plugin enabled by default
- Data directory defaults to `./pb_data` or can be specified with `--dataDir` flag
- JavaScript hooks directory: `pb_hooks/` (configurable with `--hooksDir`)
- Migrations directory: `pb_migrations/` (configurable with `--migrationsDir`)
- Always build the Admin UI before committing changes to UI code
