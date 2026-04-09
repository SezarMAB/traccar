# Traccar Backend (Vendored Fork — v6.12.2)

This is a **vendored fork** of [Traccar](https://www.traccar.org), the Apache-2.0 open-source GPS tracking server. We keep a local checkout so we can pin the version, patch protocols/decoders if needed, and control the Docker image. This is NOT the LOGISTICS.SY backend — see `../docs/blueprint/v2/projects/backend/CLAUDE.md` for that.

## Role in the LOGISTICS.SY platform

Traccar is our **GPS layer**:
1. Accepts position reports from drivers' OsmAnd app on `TCP:5055` (MVP) or hardware GPS on 200+ other ports
2. Stores positions in its own PostgreSQL database (separate schema from LOGISTICS.SY)
3. Exposes REST API (`:8082/api/*`) and WebSocket stream (`:8082/api/socket`) consumed by the LOGISTICS.SY `tracking` module
4. Handles geofence triggers for milestone detection (pickup zone, delivery zone, border crossing)

**Integration direction:** LOGISTICS.SY → Traccar (create devices, define geofences, subscribe to stream). Traccar → LOGISTICS.SY (webhook events on geofence triggers — optional).

Read blueprint §5.3 and §13 before touching tracking integration.

## Build & run

Gradle 8 + Java 17 (target JVM). Tests run on CI; local build:

```bash
./gradlew assemble            # build JAR → target/tracker-server.jar
./gradlew check               # checkstyle + tests
./gradlew clean assemble      # clean rebuild
```

Config file: `setup/traccar.xml` (DTD at `http://java.sun.com/dtd/properties.dtd`). Default is H2 embedded DB. For LOGISTICS.SY, override to PostgreSQL:

```xml
<entry key='database.driver'>org.postgresql.Driver</entry>
<entry key='database.url'>jdbc:postgresql://postgres:5432/traccar</entry>
<entry key='database.user'>traccar</entry>
<entry key='database.password'>${TRACCAR_DB_PASS}</entry>
```

Docker: use `docker/Dockerfile.alpine` or `docker/Dockerfile.debian`. Compose example in `../docs/blueprint/v2/LOGISTICS_SY_BLUEPRINT-v2.md` §13.3 — **do not expose `:8082` externally**, only the protocol ports (`:5055` for OsmAnd).

## Top-level package map (`src/main/java/org/traccar/`)

| Package | Purpose | LOGISTICS.SY touch? |
|---|---|---|
| `api/` | REST resources + WebSocket servlet | **YES — integration point** |
| `api/resource/` | 23 JAX-RS resources (Devices, Positions, Geofences, Users, Reports, ...) | Used via HTTP client |
| `api/AsyncSocket*` | WebSocket live position stream at `/api/socket` | **PRIMARY INGESTION** |
| `api/security/` | Basic/API-key/OAuth auth — we use API key from LOGISTICS.SY | Configure admin API key |
| `protocol/` | 220+ GPS protocol decoders/encoders (OsmAnd, Teltonika, GT06, etc.) | No — leave alone |
| `model/` | Core entities (`Device`, `Position`, `Geofence`, `User`, `Driver`, `Order`, ...) | Referenced via API schema |
| `database/` | DataManager, DB access layer | No |
| `storage/` | Storage abstraction with Query DSL | No |
| `session/` | Device connection management (channel → device mapping) | No |
| `handler/` | Pipeline handlers (filtering, geofencing, notifications) | No |
| `geofence/` | Circle/polygon/polyline geofence detection | Trigger milestone events |
| `notification/` + `notificators/` | Email/SMS/Firebase/Telegram/WhatsApp/webhook fan-out | Use webhook → RabbitMQ |
| `reports/` | Trip/stop/summary/events reports (Excel + JSON) | Embed in admin dashboard |
| `web/` | Jetty setup, static file serving, MCP auth filter | No |
| `forward/` | Position forwarding to external systems (JSON/Kafka/RabbitMQ/Redis) | **YES — Option for event stream** |

## Key classes for LOGISTICS.SY integration

- `api/AsyncSocketServlet.java` — WebSocket entry; accepts `?token=...` query param for auth; emits `{positions, events, devices}` JSON envelope on every update
- `api/resource/DeviceResource.java` — `POST /api/devices` to create device, returns device ID
- `api/resource/PositionResource.java` — `GET /api/positions?deviceId&from&to` for historical track replay and dispute evidence
- `api/resource/GeofenceResource.java` — `POST /api/geofences` for pickup/delivery zones
- `api/resource/EventResource.java` — `GET /api/events` for geofence/alert events
- `model/Position.java` — the position entity (lat, lon, speed, course, valid, attributes JSON)
- `web/WebServer.java` — Jetty wiring; `/api/*` mounted at line ~198, static files at `/`, MCP at `McpServerHolder.PATH`

## Gotchas

- **Version pinning** — Traccar's API surface is stable but attribute schemas drift between minor versions. Blueprint pins to `6.12` (current HEAD is `6.12.2`). Don't bump without verifying `model/*.java` fields used by LOGISTICS.SY tracking module.
- **Liquibase schema migrations** live in `schema/changelog-*.xml`. They run on Traccar startup, NOT on LOGISTICS.SY startup — keep Traccar's DB separate.
- **Max 665 protocol files** in `protocol/` — ignore unless adding a new device. OsmAnd (phone-based) is `OsmandProtocol.java`.
- **Built-in driver entity** (`model/Driver.java`) — Traccar already has a drivers concept. LOGISTICS.SY `drivers` table is separate and richer; use Traccar's `Driver` only for raw dispatcher-to-vehicle assignments if needed.
- **WebSocket reconnect** — Traccar expects a persistent connection. If LOGISTICS.SY tracking module reconnects, catch up via `GET /api/positions?from=lastKnownTime`.
- **`api/media/*`** — Traccar can serve uploaded media. We store media in MinIO instead; ignore this path.
- **MCP auth filter** (`web/McpAuthFilter.java`) — Traccar exposes an MCP server. Not used by LOGISTICS.SY.
- **OpenAPI spec** — `openapi.yaml` at repo root (92k lines). Reference it when stubbing LOGISTICS.SY HTTP client types.

## Do not modify unless you must

We want upstream Traccar to remain mergeable. Patches should be confined to:
- `docker/` (our Dockerfiles, not upstream's)
- `setup/traccar.xml` (our config)
- Local CI/GitLab files

If a protocol or handler needs a patch, open a PR upstream first; if urgent, document the patch in `PATCHES.md` at repo root (create if missing).

## License

Apache 2.0 — permissive for commercial use. See `LICENSE.txt`.
