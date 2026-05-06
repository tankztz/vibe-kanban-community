# Local self-host quickstart

This is the fastest known path from a fresh clone to a working local Vibe Kanban Community instance.

## Prerequisites

```bash
# macOS
brew install rust node bun postgresql@16
brew services start postgresql@16

# Debian / Ubuntu
sudo apt install -y rustup nodejs postgresql-16
curl -fsSL https://bun.sh/install | bash
sudo systemctl start postgresql
```

## 1. Clone and install

```bash
git clone https://github.com/tankztz/vibe-kanban-community.git
cd vibe-kanban-community
bun install
```

## 2. Set up the database

```bash
psql -d postgres <<'SQL'
CREATE USER remote WITH PASSWORD 'remote' SUPERUSER;
CREATE DATABASE remote OWNER remote;
SQL
```

## 3. Create the relay env file

This generates local secrets and writes them into a gitignored file.

```bash
cat > crates/remote/.env.remote <<EOF
SERVER_DATABASE_URL=postgres://remote:remote@localhost:5432/remote
SERVER_LISTEN_ADDR=127.0.0.1:4000
SERVER_PUBLIC_BASE_URL=http://localhost:4000
ELECTRIC_ROLE_PASSWORD=remote
SELF_HOST_LOCAL_AUTH_EMAIL=admin@local
RUST_LOG=info,remote=info
VIBEKANBAN_REMOTE_JWT_SECRET=$(openssl rand -base64 48)
SELF_HOST_LOCAL_AUTH_PASSWORD=$(openssl rand -base64 12)
EOF

chmod 600 crates/remote/.env.remote
```

Optional: if you want the previous Electric stub behavior, add this line to `.env.remote`:

```bash
ELECTRIC_URL=http://localhost:1
```

## 4. Start the kanban backend (relay)

In one terminal:

```bash
cd crates/remote
set -a
. ./.env.remote
set +a
cargo run --bin remote
```

First build can take several minutes.

Verify the backend is running:

```bash
curl http://localhost:4000/v1/health
```

Expected response:

```json
{"status":"ok","version":"0.2.0"}
```

## 5. Start the desktop app

In a separate terminal:

```bash
cd vibe-kanban-community
export VK_SHARED_API_BASE=http://localhost:4000
bun run dev
```

## 6. Open the app

Visit <http://localhost:3001> and sign in with:

- Email: `admin@local`
- Password: `grep SELF_HOST_LOCAL_AUTH_PASSWORD crates/remote/.env.remote`

## Daily use

After first setup, you usually only need these two commands:

```bash
# Terminal 1 - relay
cd vibe-kanban-community/crates/remote
set -a
. ./.env.remote
set +a
cargo run --bin remote
```

```bash
# Terminal 2 - desktop app
cd vibe-kanban-community
export VK_SHARED_API_BASE=http://localhost:4000
bun run dev
```

PostgreSQL normally auto-starts via brew/systemd once enabled.

## Notes

- `CLAUDECODE` is now cleared before launching Claude from agent tasks, so nested Claude Code sessions should no longer block startup.
- If `ELECTRIC_URL` is unset, Electric proxy routes remain disabled. For single-user local-first usage, that is usually fine.
- If you later enable logical replication for Electric, remember that `wal_level=logical` does not take effect until PostgreSQL is restarted.

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
