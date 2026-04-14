# Traccar API Integration Guide

This package (`org.traccar.api`) includes the core REST resources and the `AsyncSocketServlet` for WebSocket integration.

## LOGISTICS.SY Context

- **WebSocket Stream**: `AsyncSocketServlet.java` is the most important component. LOGISTICS.SY's `tracking` module will connect directly to `/api/socket` to receive real-time streams of `Position`, `Event`, and `Device` updates without relying on polling.
- **Device Management**: `DeviceResource.java` will be accessed by LOGISTICS.SY backend via HTTP POST when a carrier registers a vehicle, mapping the Traccar ID to the `vehicles` database table.
- **Position History**: `PositionResource.java` is utilized by the dispute and reporting modules in LOGISTICS.SY to retrieve the historical track (`GET /api/positions`) in case of delivery anomalies or missing tracking context.
- **Geofence Resources**: `GeofenceResource.java` enables dynamic boundary creation for shipments (origin, pickup, checkpoints).

## Key Development Rules 
- Do not expose these API endpoints publicly. They sit behind the inner Docker network (`transport-net`).
- Only use Keycloak/API Key bindings from LOGISTICS.SY for authentication.
