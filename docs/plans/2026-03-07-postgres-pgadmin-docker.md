# Postgres + pgAdmin Docker Compose Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Single `docker-compose.yml` to spin up PostgreSQL 17 + pgAdmin locally with persistent storage.

**Architecture:** One compose file, two services (postgres + pgadmin) on a shared bridge network, one named volume for postgres data.

**Tech Stack:** Docker Compose v2, postgres:17, dpage/pgadmin4:latest

---

### Task 1: Create docker-compose.yml

**Files:**
- Create: `docker-compose.yml`

**Step 1: Write the file**

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - dbnet

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@local.dev
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - postgres
    networks:
      - dbnet

volumes:
  pgdata:

networks:
  dbnet:
```

**Step 2: Verify it starts**

```bash
docker compose up -d
docker compose ps
```

Expected: both `postgres` and `pgadmin` show as `running`

**Step 3: Verify postgres is reachable**

```bash
docker exec -it postgres psql -U postgres -c "\l"
```

Expected: lists databases including `postgres`

**Step 4: Verify pgAdmin is reachable**

Open browser: http://localhost:5050
Login: `admin@local.dev` / `admin`
Add server: host=`postgres`, port=`5432`, user=`postgres`, password=`postgres`

**Step 5: Verify volume persists**

```bash
docker compose down
docker compose up -d
docker exec -it postgres psql -U postgres -c "\l"
```

Expected: data still present after restart
