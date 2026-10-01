# MQTT Documentation

### Overview

MQTT is used for data exchange between the server and devices. SSL/TLS connection is used on port 8883.

---

### Connection

**Connection parameters:**
```
Broker: ssl://{mqtt_host}:8883
Client ID: nextui_server_{timestamp}
Username: {mqtt_username}
Password: {mqtt_password}
TLS: Enabled
CA Certificate: /path/to/ca.crt
```

---

### Topics

#### 3.1. Commands to Device

**Topic:** `devices/{device_id}/command`
**QoS:** 0
**Direction:** Server → Device

**Message format:**
```json
{
    "type": "command",
    "command": "reboot",
    "requestId": "cmd_123"
}
```

**Example:**
```bash
mosquitto_pub -t "devices/7LgjUE6hcgYawTjj/command" \
  -m '{"type":"command","command":"restart","requestId":"cmd_456"}'
```

---

#### 3.2. Updates from Device

**Topic:** `devices/{device_id}/update`
**QoS:** 0
**Direction:** Device → Server

**Message format:**
```json
{
    "type": "device_update",
    "data": {
        "simple": {
            "cpu": 14,
            "memory": 99,
            "disk": 44,
            "swap": 94,
            "network": 0,
            "temp": 50,
            "online": true,
            "name": "Redmi 9T",
            "system": "android"
        },
        "advanced": {
            "hostname": "Redmi 9T",
            "uptime": "16d 5h 59m 38s",
            "battery": "54.0",
            "brightness": 23,
            "volume": 0,
            "processor": "bengal (ro.board.platform)",
            "video": "Android GPU (qcom)",
            "ram": "3.6GB",
            "kernel": "4.19.325",
            "architecture": "arm"
        }
    }
}
```

---

#### 3.3. Command Response

**Topic:** `devices/{device_id}/update`
**Type:** `command_response`

```json
{
    "type": "command_response",
    "data": {
        "requestId": "cmd_123",
        "command": "restart",
        "success": true
    }
}
```

**Error:**
```json
{
    "type": "command_response",
    "data": {
        "requestId": "cmd_123",
        "command": "unknown_command",
        "success": false
    }
}
```

---

#### 3.4. Terminal Output

**Topic:** `devices/{device_id}/update`
**Type:** `terminal_output`

**New format (preferred):**
```json
{
    "type": "terminal_output",
    "data": {
        "command": "terminal_output|term_123|uid=0(root) gid=0(root)"
    }
}
```

**Old format (backward compatibility):**
```json
{
    "type": "terminal_output",
    "data": {
        "session_id": "term_123",
        "output_data": "uid=0(root) gid=0(root)"
    }
}
```

---

#### 3.5. Device Status (Offline)

**Topic:** `devices/{device_id}/update`
**Type:** `device_offline`

```json
{
    "type": "device_offline"
}
```

**LWT (Last Will and Testament):**
Automatically sent when the connection is lost
```json
{
    "type": "device_offline"
}
```

---

#### 3.6. MQTT Client Management

**Topic:** `$CONTROL/dynamic-security/v1`
**QoS:** 1
**Direction:** Server → MQTT Broker

**Create client:**
```json
{
    "commands": [{
        "command": "createClient",
        "username": "device_id",
        "password": "mqtt_password",
        "roles": [{
            "rolename": "device",
            "priority": 1
        }]
    }]
}
```

**Delete client:**
```json
{
    "commands": [{
        "command": "deleteClient",
        "username": "device_id"
    }]
}
```

---

#### 3.7. Terminal Commands

**Topic:** `devices/{device_id}/command`
**Direction:** Server → Device

**Start terminal:**
```json
{
    "type": "command",
    "command": "terminal|term_123|start|80|24"
}
```

**Input:**
```json
{
    "type": "command",
    "command": "terminal|term_123|input|ls -la\n"
}
```

**Resize:**
```json
{
    "type": "command",
    "command": "terminal|term_123|resize|120|30"
}
```

**Stop:**
```json
{
    "type": "command",
    "command": "terminal|term_123|stop"
}
```

**Format:**
```
terminal|{session_id}|start|{cols}|{rows}
terminal|{session_id}|input|{data}
terminal|{session_id}|resize|{cols}|{rows}
terminal|{session_id}|stop
```

---

#### 3.8. Stream Commands

**Topic:** `devices/{device_id}/command`
**Direction:** Server → Device

**Start stream:**
```json
{
    "type": "command",
    "command": "stream:share-screen:on",
    "sessionId": "stream_abc",
    "turnConfig": {
        "iceServers": [
            {
                "urls": [
                    "stun:turn.example.com:3478",
                    "turn:turn.example.com:3478",
                    "turn:turn.example.com:5349?transport=tcp"
                ],
                "username": "1234567890:stream_abc",
                "password": "base64_hmac_sha1"
            }
        ]
    },
    "requestId": "req_123"
}
```

**Stop stream:**
```json
{
    "type": "command",
    "command": "stream:share-screen:off",
    "sessionId": "stream_abc",
    "requestId": "req_123"
}
```

**Stream types:** `share-screen`, `watch-screen`, `control-screen`

---

#### 3.9. File Transfer Commands

**Topic:** `devices/{device_id}/command`
**Direction:** Server → Device

**Upload:**
```json
{
    "type": "command",
    "command": "file:upload|sess_123|https://server/upload/sess_123|/path/file.txt",
    "requestId": "req_123"
}
```

**Download:**
```json
{
    "type": "command",
    "command": "file:download|sess_123|https://server/download/sess_123|/path/file.txt",
    "requestId": "req_123"
}
```

**Cancel:**
```json
{
    "type": "command",
    "command": "file:cancel|sess_123"
}
```

**Format:**
```
file:upload|{session_id}|{upload_url}|{path}
file:download|{session_id}|{download_url}|{path}
file:cancel|{session_id}
```

---

#### 3.10. SSH Tunnel Protocol

**Topic:** `devices/{device_id}/command`
**Direction:** Server → Device

**Start SSH tunnel:**
```json
{
    "type": "command",
    "command": "start-ssh",
    "requestId": "tunnel_123"
}
```

**Device response (device → server, via device channel):**

On success:
```
CONNECTED TO {target}
```

On error:
```
ERROR:{message}
```

**Target directive (server → device, before piping):**
```
TARGET:{target}\n
```

---

#### 3.11. Ping / Pong

**Ping (server → device):**
```json
{
    "type": "command",
    "command": "ping",
    "requestId": "ping-1234567890-device_id"
}
```

**Pong (device → server):**
```json
{
    "type": "command_response",
    "data": {
        "command": "pong",
        "requestId": "ping-1234567890-device_id",
        "success": true
    }
}
```

---

#### 3.12. Upgrade Command

**Topic:** `devices/{device_id}/command`
**Direction:** Server → Device

```json
{
    "type": "command",
    "command": "upgrade:{size}:{base64_data}",
    "requestId": "req_123"
}
```

- `{size}` — binary file size in bytes
- `{base64_data}` — file content encoded as base64

On success the device sends `command_response` with `success: true`, after which the server marks the device as offline.

---

#### 3.13. System Topics

| Topic | Description |
|-------|----------|
| `$SYS/broker/version` | Broker version |
| `$SYS/broker/uptime` | Broker uptime |
| `$SYS/broker/clients/total` | Total clients |
| `$SYS/broker/clients/connected` | Connected clients |
| `$SYS/broker/messages/sent` | Messages sent |
| `$SYS/broker/messages/received` | Messages received |
| `$SYS/broker/load/messages/received/1min` | Load over 1 minute |
| `$SYS/broker/load/connections/1min` | Connection load |

> Incoming messages on `$SYS/*` and `$CONTROL/*` topics are ignored by the server.

---

### Topic Validation & Limits

**Device ID format:**
- Must match `[a-zA-Z0-9_-]{12,20}`
- Recommended: 16 characters (base64url of 12 random bytes)

**Topic whitelist:**
```
^devices/[a-zA-Z0-9_-]{12,}/update$
^devices/[a-zA-Z0-9_-]{12,}/command$
^devices/[a-zA-Z0-9_-]{12,}/status$
^\$CONTROL/dynamic-security/v1$
^\$SYS/.*$
```

**Message size limit:** 10 MB

---

### Supported Message Types

| Type | Direction | Description |
|------|-----------|-------------|
| `device_update` | Device → Server | Device metrics update |
| `command_response` | Device → Server | Response to a command |
| `terminal_output` | Device → Server | Terminal output data |
| `device_offline` | Device → Server | Device going offline |

> Any other `type` is logged as warning and ignored.

---

### MQTT Roles

The broker must have a pre-created role named `device` with the appropriate ACL for the following topics:

- `devices/{device_id}/update` (write)
- `devices/{device_id}/command` (read)
- `devices/{device_id}/status` (write)

Authentication uses username/password stored in the `device:{id}` hash (`mqtt_username`, `mqtt_password`).
