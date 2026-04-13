# Architecture

## Workspace Layout

Neolink is a Cargo workspace with a main binary crate and several library crates:

- **neolink** (root) - Main binary. CLI entry point, subcommand dispatch, RTSP server, MQTT client.
- **neolink_core** (crates/core) - Core protocol library (~21k LOC). All Baichuan protocol logic lives here.
- **neolink_decoder** (crates/decoder) - Media decoding utilities.
- **pushnoti** (crates/pushnoti) - FCM push notification listener.
- **mailnoti** (crates/mailnoti) - Email notification support.

## neolink_core Internals

### bc_protocol/
High-level camera API. The `BcCamera` struct is the primary interface for interacting with a camera: connecting, authenticating, requesting streams, sending commands (PTZ, LED, reboot, etc.).

### bc/
Low-level protocol structures. Defines message types, header formats, XML body structures. All Baichuan message serialization and deserialization happens here. XML bodies use quick-xml with serde derives.

### bcmedia/
Media stream parsing. Extracts H264/H265 NAL units and AAC audio frames from Baichuan media packets. These are then repackaged for RTSP or saved as images.

### bcudp/
UDP transport layer for P2P and relay connections. Some cameras use UDP rather than direct TCP.

## Main Binary Modules

### src/rtsp/
GStreamer-based RTSP server. Creates per-camera stream factories. When an RTSP client connects, it triggers a Baichuan connection to the camera and streams video/audio. Uses gstreamer-rtsp-server bindings.

### src/mqtt/
MQTT client using rumqttc. Publishes camera status, motion detection events, and other telemetry. Supports Home Assistant MQTT auto-discovery for seamless integration. Per-camera topic handlers manage subscriptions and commands.

### src/common/
Central coordination layer:
- **NeoReactor** - Central coordinator that manages all camera connections and shared state.
- **CamThread** - Per-camera connection manager. Handles connection lifecycle, reconnection on failure, and multiplexing requests from RTSP/MQTT/subcommands.
- Instance management for ensuring single connections per camera.

### Subcommand Modules
`battery/`, `pir/`, `ptz/`, `talk/`, `statusled/`, `reboot/`, `users/`, `services/`, `image/` - Each implements a CLI subcommand that connects to a camera and performs a specific action.

## Async Architecture

The entire application uses Tokio as its async runtime. All I/O (camera connections, RTSP serving, MQTT) is async. The binary uses `#[tokio::main]` with the multi-thread runtime.

## Connection Lifecycle

1. RTSP client connects to Neolink's GStreamer RTSP server
2. Neolink initiates a Baichuan TCP connection to the camera on port 9000
3. Initial handshake exchanges encryption parameters
4. AES-128 CFB encryption is negotiated and activated
5. Neolink authenticates with camera username/password
6. Video stream is requested (mainStream or subStream)
7. Camera sends media packets containing H264/H265 NAL units + AAC audio
8. Neolink extracts NAL units and repackages them as an RTSP stream
9. RTSP client receives standard H264/H265 video stream
