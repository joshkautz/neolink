# Testing and Debugging

## Unit Tests

```bash
cargo test                          # Run all tests
cargo test -p neolink_core          # Test core library only
cargo test --workspace              # Test all workspace members
```

## Integration Testing

Most integration testing requires actual Reolink camera hardware. There is no mock camera server. Testing is largely manual:

1. Connect to a real camera with known credentials
2. Verify RTSP stream is accessible (e.g., with VLC or ffplay)
3. Verify MQTT messages are published correctly
4. Test subcommands (ptz, reboot, image capture, etc.)

## Logging

Control log verbosity with the `RUST_LOG` environment variable:

```bash
RUST_LOG=debug neolink rtsp --config neolink.toml          # Debug level
RUST_LOG=neolink=trace neolink rtsp --config neolink.toml  # Trace for neolink only
RUST_LOG=neolink_core=trace neolink rtsp --config neolink.toml  # Trace for core library
```

Note: release builds cap log output at debug level (`release_max_level_debug`).

## GStreamer Debugging

```bash
GST_DEBUG=3 neolink rtsp --config neolink.toml    # GStreamer warnings and above
GST_DEBUG=5 neolink rtsp --config neolink.toml    # Verbose GStreamer output
GST_DEBUG_DUMP_DOT_DIR=/tmp neolink rtsp ...       # Dump pipeline graphs
```

## Protocol Debugging with Wireshark

The `dissector/` directory contains tools for protocol-level debugging:

- **dissector/baichuan.lua** - Wireshark Lua dissector for the Baichuan protocol. Install by placing in Wireshark's plugin directory or loading via `-X lua_script:dissector/baichuan.lua`.
- **dissector/protocol.md** - General protocol structure documentation
- **dissector/messages.md** - Known message types, their IDs, and XML body formats
- **dissector/mediapacket.md** - Media stream packet structure (video/audio)
- **dissector/udp.md** - UDP transport variant documentation

## Common Issues

- **GStreamer not found**: Ensure GStreamer dev packages are installed (see 03-build-and-deploy.md)
- **Camera connection refused**: Verify camera IP, port 9000 is accessible, credentials are correct
- **Stream freezes**: May indicate network issues or camera firmware bugs; check logs for reconnection attempts
- **Cross-compilation fails**: Ensure `cross` is installed (`cargo install cross`) and Docker is running
