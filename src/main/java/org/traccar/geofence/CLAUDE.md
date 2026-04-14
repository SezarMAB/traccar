# Traccar Geofence Logic

This package (`org.traccar.geofence`) provides bounds-checking classes (Polygons, Circles, and Polylines). 

## LOGISTICS.SY Context

- Building geofences is crucial for detecting business lifecycle milestones, mainly `PickupConfirmed`, `DeliveryConfirmed`, `AtCustoms`, and route deviations.
- In LOGISTICS.SY, when a transport order is formed, its origin/destination zones should be synchronized dynamically into Traccar as polygon/circle geofences via `api/resource/GeofenceResource.java`. 
- **Milestone Detection**: Upon boundary intercept, Traccar generates events (mapped through `api/socket`) consumed by the `tracking` module to auto-update the shipment state machine in LOGISTICS.SY (e.g., `IN_TRANSIT` to `NEAR_DESTINATION`).

## Performance Notice
- Ensure complex boundaries (e.g., huge polylines representing cross-border borders) are accurately optimized per geofence vertex counts, as heavy intersect calculations on continuous TCP streaming can bottleneck GPS tracking.
