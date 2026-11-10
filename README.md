# gridreserve-config

Central runtime configuration for [GridReserve](https://github.com/chhavinath-bit/GridReserve), served to every service by the Spring Cloud Config Server.

> **Never put secrets here.** This repository is public. Passwords, private keys and tokens come from environment variables; files here may only reference them as `${VARIABLE_NAME}`.

## How it is read

- The Config Server reads this repository from `CONFIG_REPO_URI`:
  - `dev`: the local clone next to the GridReserve folder, mounted into the config-server container (only **committed** changes are served).
  - `prod`: `https://github.com/chhavinath-bit/gridreserve-config`, branch `main`.
- A service named `booking-service` running with profile `dev` receives, merged in this order (later wins): `application.yml` → `application-dev.yml` → `booking-service.yml` → `booking-service-dev.yml`.

## Files (added from milestone M1.1)

| File | Contents |
|---|---|
| `application.yml`, `application-{dev,prod}.yml` | Shared defaults: actuator, logging, resilience defaults; the JWT **public** keys per profile (not secret) |
| `api-gateway.yml` + `-dev` / `-prod` | Routes, CORS, rate limits (Phase 4) |
| `identity-service.yml` + `-dev` / `-prod` | Token lifetimes, cookie settings; the signing `key-id` per profile |
| `booking-service.yml` + `-dev` / `-prod` | Job intervals, Feign client timeouts and resilience policies, event subscriptions (Phase 5) |

## Keys per profile

The JWT public keys (`gridreserve.jwt.public-keys`) live in `application-dev.yml` and `application-prod.yml`, never in the shared `application.yml`: Spring merges map entries across profile files, so a key in the shared file would be trusted in every profile. Prod trusts only its own key; its private key exists only on the deploy host.

## What can change without a restart

Grouped `@ConfigurationProperties` / `@RefreshScope` settings are applied to running instances with `scripts/refresh-config.sh` in the GridReserve repo (it finds every instance through Eureka and calls `POST /actuator/refresh`):

- Refreshable: resilience thresholds (timeouts, circuit breaker, retry, bulkhead, rate limiter), rate-limit settings, CORS origins, job intervals, event subscriptions, feature toggles.
- Not refreshable (restart needed): datasource settings, ports, signing keys.
- Not configuration at all: booking rules (slot length, cutoff, limits) are data in the booking database.

## Changing configuration

1. Open a pull request here (link it from the related GridReserve milestone).
2. After merge, run the refresh script, or restart the affected service for non-refreshable settings.
3. Invalid values are rejected by validated properties and logged; revert with a new commit.

## License

[MIT](LICENSE)
