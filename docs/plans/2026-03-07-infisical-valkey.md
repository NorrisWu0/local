# Infisical + Valkey Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add Valkey and Infisical services to the existing `docker-compose.yml` for local secrets management.

**Architecture:** Extend existing compose file with a Valkey service (Redis-compatible) and Infisical app service, both joining the existing `dbnet`. Infisical reuses the existing `postgres:17` instance with a dedicated `infisical` database.

**Tech Stack:** Docker Compose v2, valkey/valkey:9-alpine, infisical/infisical:v0.158.7, postgres:17 (existing)

---

### Task 1: Add Valkey and Infisical to docker-compose.yml

**Files:**
- Modify: `docker-compose.yml`

**Step 1: Add Valkey service**

Add to `services:` in `/home/norris/git/local/docker-compose.yml`:

```yaml
  valkey:
    image: valkey/valkey:9-alpine
    container_name: valkey
    ports:
      - "6379:6379"
    volumes:
      - valkey_data:/data
    networks:
      - dbnet
```

**Step 2: Add Infisical service**

Add to `services:` after valkey:

```yaml
  infisical:
    image: infisical/infisical:v0.158.7
    container_name: infisical
    environment:
      NODE_ENV: production
      DB_CONNECTION_URI: postgresql://postgres:postgres@postgres:5432/infisical
      REDIS_URL: redis://valkey:6379
      ENCRYPTION_KEY: a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
      AUTH_SECRET: a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1
      SITE_URL: http://localhost:8080
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - valkey
    networks:
      - dbnet
```

**Step 3: Add valkey_data to top-level volumes**

```yaml
volumes:
  pgdata:
  pgadmin_data:
  valkey_data:
```

**Step 4: Validate the compose file**

```bash
docker compose -f /home/norris/git/local/docker-compose.yml config
```

Expected: resolves cleanly with no errors

**Step 5: Create the infisical database**

```bash
docker exec postgres psql -U postgres -c "CREATE DATABASE infisical;"
```

Expected: `CREATE DATABASE`

**Step 6: Start new services**

```bash
docker compose up -d valkey infisical
docker compose ps
```

Expected: `valkey` and `infisical` show as `running`

**Step 7: Verify Infisical is reachable**

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/api/status
```

Expected: `200`

**Step 8: Commit**

```bash
git add docker-compose.yml
git commit -m "feat: add valkey and infisical services to docker compose"
```
