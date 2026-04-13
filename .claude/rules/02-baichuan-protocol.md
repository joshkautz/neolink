# Baichuan Protocol

## Overview

Baichuan is the proprietary binary protocol used by Reolink IP cameras for all communication. It operates on TCP port 9000, with a UDP variant for P2P and relay connections.

## Message Structure

Each Baichuan message consists of:

1. **Magic bytes** - Protocol identifier at the start of each message
2. **Message ID** - Identifies the message type (Login, VideoStream, PTZ, etc.)
3. **Body length** - Length of the message payload
4. **XML body** - Most messages carry an XML payload with structured data

## Encryption

- Initial handshake is unencrypted
- After the handshake, all communication uses **AES-128 CFB** encryption
- Encryption keys are derived during the handshake process
- The encryption covers message bodies, not headers

## Key Message Types

- **Login** - Authentication with username/password
- **Video stream request** - Initiates mainStream or subStream
- **PTZ control** - Pan/tilt/zoom commands
- **LED control** - Status LED on/off
- **PIR control** - Motion detection settings
- **Battery status** - Battery level queries (battery-powered cameras)
- **Reboot** - Camera restart command
- **User management** - Add/remove/modify camera users

## Media Packets

Media data is sent as a separate stream within the connection:

- **Video**: H264 or H265 NAL units, depending on camera model and configuration
- **Audio**: AAC audio frames
- Media packets have their own header format distinct from control messages
- The `bcmedia` module in neolink_core handles parsing these packets

## UDP Variant

Some cameras (especially newer models) use UDP for P2P or relay connections:

- Implemented in `neolink_core/bcudp/`
- Used when direct TCP on port 9000 is not available
- Supports relay through Reolink's servers

## Debugging

- **Wireshark dissector**: `dissector/baichuan.lua` - Lua-based dissector for decoding Baichuan traffic in Wireshark
- **Protocol documentation**: `dissector/protocol.md` - General protocol structure
- **Message reference**: `dissector/messages.md` - Known message types and their XML bodies
- **Media packet format**: `dissector/mediapacket.md` - Media stream packet structure
- **UDP protocol**: `dissector/udp.md` - UDP transport variant documentation
