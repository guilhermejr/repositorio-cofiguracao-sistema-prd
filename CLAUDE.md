# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The configuration repository served by **`config-server`** to every microservice in the system. Editing a file here changes the configuration of a running service the next time it fetches — no rebuild.

## Layout

```
application.yml               # shared by every service
<service-name>/
  <service-name>.yml          # that service's own settings
```

`application.yml` carries what is common: actuator exposure and health settings, mail, AWS, Eureka coordinates. The per-service files carry port, `context-path`, datasource and anything specific.

## Secrets are not here

Values like `${autenticacaoDBPass}` and `${JWTSecret}` are **placeholders**, resolved by each service against **Vault** — not by this repo and not by the config server. Never replace one with a literal value.

## Things worth knowing

- `management.endpoint.health.show-details` is `never` on purpose: `/actuator/health` is public, so it must not leak database, disk or Vault detail.
- `management.endpoints.web.exposure.include` is `"*"`, so every actuator endpoint exists. Only `/actuator/health` is public; the rest is protected by each service's security chain.
- `config-server`, `eureka-server` and `gateway-server` do **not** read this repo — they configure themselves from Vault. Changes here do not reach them.

## Applying a change

```bash
git push origin main
```

The config server reads from GitHub per request, so a push is enough for services that restart afterwards. Services already running keep the values they fetched at startup until restarted.
