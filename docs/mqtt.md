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

### 3.7. System Topics

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
