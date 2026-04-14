# Traccar Protocol Decoders

This package (`org.traccar.protocol`) houses over 262 parsers (TCP/UDP).

## LOGISTICS.SY Context

- **OsmAndProtocol**: This is the most crucial protocol decoder for the MVP phase. It interprets the REST-like socket messages sent by drivers using the mobile OsmAnd app.
- **Scaling Hardware**: Protocols like `TeltonikaProtocol` and `Gl200Protocol` (Queclink) are primary targets when scaling fleet adoption to hardwired trackers in Phase 2.
- **No Modifications Required**: The parsers adhere strictly to device manufacturing formats. Any additions needed for Syrian context parsing should be dealt with either up-stream to the open source project, or via `PATCHES.md`. DO NOT append or rewrite decoders locally unless highly specific and necessary (which avoids broken compatibility on software auto-updating).
