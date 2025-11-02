# Official PDS - Local Development Setup

Simplified Docker setup for local development on Windows.

## Why Docker PDS for Local Development?

The ATProto monorepo PDS **cannot run on Windows** because it creates directories with colons (`:`) in the name (from DIDs like `did:plc:xxx`), which are illegal in Windows file paths.

The Docker PDS runs **Linux in a container**, avoiding this limitation.

## Prerequisites

- **Docker Desktop for Windows**
  - Download: https://docs.docker.com/desktop/install/windows-install/
  - Ensure it's running (check system tray)
- **Node 20** (for Bluesky App and Pinata)

## Quick Start

### 1. Create Data Directory

```bash
cd ~/bsky-private-profile/official-pds
mkdir -p data
```

### 2. Start the PDS

```bash
# Using the local development compose file
docker compose -f compose.local.yaml up

# Or run in background
docker compose -f compose.local.yaml up -d
```

### 3. Verify It's Running

```bash
curl http://localhost:2583/xrpc/_health
# Should return: {"version":"0.4.x"}
```

### 4. Create Test Account

```bash
# Create account via API
curl -X POST http://localhost:2583/xrpc/com.atproto.server.createAccount \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user1@test.local",
    "handle": "user1.test",
    "password": "password123"
  }'
```

Or use the pdsadmin tool:

```bash
docker exec -it pds-local /pdsadmin/pdsadmin.sh account create
```

## Configuration

The `compose.local.yaml` includes all necessary settings for local development:

- **Port:** 2583 (mapped from container port 3000)
- **Hostname:** localhost
- **Dev Mode:** Enabled (allows HTTP, no HTTPS needed)
- **Invites:** Disabled (anyone can create accounts)
- **Data:** Stored in `./data` directory

## Managing the PDS

### Start

```bash
docker compose -f compose.local.yaml up -d
```

### Stop

```bash
docker compose -f compose.local.yaml down
```

### View Logs

```bash
docker compose -f compose.local.yaml logs -f pds
```

### Restart

```bash
docker compose -f compose.local.yaml restart
```

### Access Container Shell

```bash
docker exec -it pds-local sh
```

## Differences from Production compose.yaml

The `compose.local.yaml` is simplified for local development:

| Production (compose.yaml) | Local (compose.local.yaml) |
|---------------------------|----------------------------|
| ✅ Caddy (TLS/HTTPS) | ❌ Removed (not needed for localhost) |
| ✅ Watchtower (auto-updates) | ❌ Removed (manual updates) |
| ✅ network_mode: host | ✅ Port mapping (2583:3000) |
| ✅ /pds paths | ✅ ./data (local directory) |

## Troubleshooting

### "Cannot connect to Docker daemon"

**Solution:** Start Docker Desktop

### "Port 2583 already in use"

**Solution:** Stop the ATProto monorepo PDS:
```bash
# Find and kill process on port 2583
netstat -ano | findstr :2583
taskkill /PID [PID] /F
```

### "Permission denied" on data directory

**Solution:** Ensure the data directory exists and Docker has access:
```bash
mkdir -p data
```

## Connecting Bluesky App

The Bluesky App `.env.local` should already point to the correct URL:

```bash
EXPO_PUBLIC_PDS_URL=http://localhost:2583
```

This works with both the Docker PDS and the ATProto monorepo PDS.

## Data Persistence

All PDS data is stored in `./data` directory:
- SQLite databases
- Blob storage
- Actor repositories

This directory is mapped as a volume, so data persists between container restarts.

To reset everything:
```bash
docker compose -f compose.local.yaml down
rm -rf data
mkdir data
docker compose -f compose.local.yaml up -d
```

## Next Steps

1. ✅ Start Docker PDS
2. ✅ Create test account
3. ✅ Open Bluesky App at http://localhost:19006
4. ✅ Log in with test credentials
5. ✅ Start developing private profile features!

## Official Documentation

For production deployment, see:
- [Official README](./README.md)
- [Production compose.yaml](./compose.yaml)
- https://github.com/bluesky-social/pds

---

**Local development is now as simple as `docker compose up`!** 🐳✨

