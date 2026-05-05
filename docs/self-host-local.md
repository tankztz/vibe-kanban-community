# Local self-host quickstart

This fork supports a simple local-first setup without Docker or Electric.

## Prerequisites

- Rust
- Node.js 20+
- bun or pnpm
- PostgreSQL running locally

## 1. Prepare PostgreSQL

Create a local database, for example:

```bash
createdb vibe_kanban
```

If you plan to use Electric later, PostgreSQL must run with `wal_level=logical`.
After changing it with `ALTER SYSTEM SET wal_level = 'logical';`, you must restart PostgreSQL for it to take effect.

Examples:

```bash
# macOS (Homebrew)
brew services restart postgresql

# systemd Linux
sudo systemctl restart postgresql
```

## 2. Create the remote env file

```bash
cp crates/remote/.env.remote.example crates/remote/.env.remote
```

At minimum, set:

- `SERVER_DATABASE_URL`
- `SERVER_PUBLIC_BASE_URL`
- `VIBEKANBAN_REMOTE_JWT_SECRET`
- either local auth (`SELF_HOST_LOCAL_AUTH_EMAIL` / `SELF_HOST_LOCAL_AUTH_PASSWORD`) or OAuth credentials

## 3. Start the remote API

```bash
cd crates/remote
set -a
source .env.remote
set +a
cargo run --bin remote
```

Recommended local values:

```env
SERVER_DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/vibe_kanban
SERVER_LISTEN_ADDR=0.0.0.0:4000
SERVER_PUBLIC_BASE_URL=http://localhost:4000
```

## 4. Start the local web app

From the repo root:

```bash
export VK_SHARED_API_BASE=http://localhost:4000
bun run dev
```

Or with pnpm:

```bash
export VK_SHARED_API_BASE=http://localhost:4000
pnpm run dev
```

## Current behavior notes

- If `ELECTRIC_URL` is unset, Electric proxy routes stay disabled. This is expected for local-only mode.
- Without Electric, realtime multi-tab sync is not available. Single-tab/local usage still works.
- Claude Code executor now clears inherited `CLAUDECODE` so agent-launched Claude sessions can start even if Vibe Kanban itself was launched inside Claude Code.

## Helpful corp / constrained-network workarounds

```bash
git config --global http.sslCAInfo /etc/ssl/copilot.pem
```

`~/.cargo/config.toml`:

```toml
[net]
git-fetch-with-cli = true
```

```bash
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles
```
