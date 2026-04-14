# Traccar Data Models

This package (`org.traccar.model`) contains the entities Traccar utilizes for data representations, which correlate strongly to the LOGISTICS.SY tracking data scheme.

## LOGISTICS.SY Context

- **Position**: Maps to `tracking_events` in LOGISTICS.SY. It contains crucial contextual tracking dimensions: `latitude`, `longitude`, `speed`, and `attributes` representing internal JSON metrics.
- **Event**: Contains geofence milestones triggers.
- **Device**: Correlates to `vehicles` table in LOGISTICS.SY. The identifier links 1-to-1 with a vehicle's `traccar_device_id`.
- **Driver**: **WARNING:** Do not use the `Driver` model inside Traccar for business logic. Our source of truth for the Dispatcher-to-Driver bindings sits inside LOGISTICS.SY `drivers` table (managed via `shipment`).

## Modification Cautions
- Do not modify Traccar models directly to include LOGISTICS.SY business fields (e.g. adding Contracts to Positions). Use the `attributes` (JSON map string) or external referencing inside the LOGISTICS.SY DB to stay mergeable with upstream Traccar updates.
