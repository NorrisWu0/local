# Infisical Local Dev Design

**Date:** 2026-03-07

## Summary
Add Infisical secrets management + Valkey to the existing `docker-compose.yml`.

## Spec
- Valkey: `valkey/valkey:9-alpine` on `6379` (Redis-compatible)
- Infisical: `infisical/infisical:v0.158.7` on `8080`
- Reuse existing `postgres:17` — create new `infisical` database
- All services on existing `dbnet`
- `ENCRYPTION_KEY` + `AUTH_SECRET` hardcoded for local dev
