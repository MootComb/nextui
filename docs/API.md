# API Documentation

## Overview

NextUI provides a REST API for managing devices, users, and the system. All API requests use a single endpoint with authentication via API keys.

## Basic Settings

### Endpoint

```
POST /api
```

### Authentication

All requests require an **API key**, which is passed in the request body:

```json
{
    "token": "your_api_key",
    "action": "action_name",
    "data": { ... }
}
```

### Response Format

All responses are returned in JSON format:

**Successful response:**
```json
{
    "success": true,
    "action": "action_name",
    "...": "additional_data"
}
```

**Error:**
```json
{
    "success": false,
    "error": "Error message",
    "status": 400
}
```

---

## API Methods

### 1. Device List

Retrieves a list of all user devices.

**Action:** `devices`

**Required permission:** `device_info`

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "devices"
}
```

**Example response:**
```json
{
    "success": true,
    "action": "devices",
    "devices": [
        {
            "id": "7LgjUE6hcgYawTjj",
            "name": "Redmi 9T",
            "system": "android",
            "online": true,
            "cpu": 14,
            "memory": 99,
            "disk": 44,
            "swap": 94,
            "network": 0,
            "temp": 50,
            "access": ["*"]
        }
    ],
    "count": 1
}
```

---

### 2. Sending a Command

Sends a command to a device.

**Action:** `command`

**Required permission:** `send_commands`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deviceId` | string | Yes | Device ID |
| `command` | string | Yes | Command to execute |
| `requestId` | string | No | Identifier for tracking |

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "command",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "command": "restart",
        "requestId": "cmd-123"
    }
}
```

**Example response:**
```json
{
    "success": true,
    "action": "command",
    "message": "Command sent",
    "requestId": "cmd-123"
}
```

**Available commands:**

#### Power Management
| Command | Description |
|---------|-------------|
| `shutdown` | Power off the device |
| `reboot` | Restart the device |
| `lock` | Lock the device |
| `sleep` | Sleep mode |
| `recovery` | Boot into Recovery |
| `fastboot` | Boot into Fastboot |
| `odin` | Boot into Odin |
| `safe` | Safe mode |

#### Updates
| Command | Description |
|---------|-------------|
| `update-simple` | Update basic data (CPU, Memory) |
| `update-advanced` | Update advanced data |
| `update-all` | Update all data |
| `update-stop` | Stop updates |
| `restart` | Restart the client |
| `reconnect` | Reconnect |
| `stop` | Stop the client |
| `suicide` | Client self-destruct |

#### Resource Management
| Command | Description |
|---------|-------------|
| `volume:0-100` | Set volume |
| `brightness:0-100` | Set brightness |
| `clipboard:text` | Set clipboard |
| `kill-pid:PID` | Kill process by PID |

#### Media
| Command | Description |
|---------|-------------|
| `media:play` | Play |
| `media:pause` | Pause |
| `media:next` | Next track |
| `media:prev` | Previous track |
| `media-seek:0-100` | Seek (in percent) |

#### Terminal
| Command | Description |
|---------|-------------|
| `terminal:start\|session_id\|cols\|rows` | Start terminal |
| `terminal:input\|session_id\|data` | Input to terminal |
| `terminal:resize\|session_id\|cols\|rows` | Resize terminal |
| `terminal:stop\|session_id` | Stop terminal |

#### Remote Control
| Command | Description |
|---------|-------------|
| `remote:power` | Power button |
| `remote:hdmi` | Switch HDMI |
| `remote:play` | Play |
| `remote:pause` | Pause |
| `remote:previous` | Previous |
| `remote:next` | Next |
| `remote:mute` | Mute |
| `remote:volume-down` | Volume down |
| `remote:volume-up` | Volume up |
| `remote:up` | Up |
| `remote:down` | Down |
| `remote:left` | Left |
| `remote:right` | Right |
| `remote:ok` | OK |
| `remote:back` | Back |
| `remote:home` | Home |
| `remote:voice` | Voice assistant |

#### Policies
| Command | Description |
|---------|-------------|
| `policy:signals\|true/false/toggle` | Signal blocking |
| `policy:autostart\|true/false/toggle` | Autostart |
| `policy:hiding\|true/false/toggle` | Process hiding |
| `policy:block\|true/false/toggle` | USB blocking |

#### Family Controls
| Command | Description |
|---------|-------------|
| `family:enable:true/false` | Enable/disable controls |
| `family:lock:true/false` | Lock/unlock |
| `family:limit:minutes` | Set daily limit (minutes) |
| `family:pause:minutes` | Pause for (minutes) |
| `family:resume` | Resume from pause |
| `family:interval:minutes` | Update interval |

#### Intervals
| Command | Description |
|---------|-------------|
| `interval:quick\|seconds` | Quick update interval |
| `interval:medium\|seconds` | Medium update interval |
| `interval:long\|seconds` | Long update interval |
| `interval:send\|seconds` | Send interval |

#### Sending Settings
| Command | Description |
|---------|-------------|
| `sending:simple\|true/false` | Enable sending basic data |
| `sending:advanced\|true/false` | Enable sending advanced data |
| `sending:stop\|true/false` | Enable sending stop commands |

#### Webhooks
| Command | Description |
|---------|-------------|
| `webhook:add\|command` | Create webhook with command |
| `webhook:remove\|webhook_id` | Remove webhook |

#### Miscellaneous
| Command | Description |
|---------|-------------|
| `start-ssh` | Start SSH tunnel |
| `stop-ssh` | Stop SSH tunnel |
| `upgrade:architecture` | Upgrade client (arm64, amd64, arm) |
| `command-script_name` | Execute custom script |
| `open-site:url` | Open website on device |

---

### 3. Renaming a Device

**Action:** `rename`

**Required permission:** `device_management`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deviceId` | string | Yes | Device ID |
| `newName` | string | Yes | New name (max. 16 characters) |

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "rename",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "newName": "My Phone"
    }
}
```

**Example response:**
```json
{
    "success": true,
    "action": "rename",
    "message": "Device renamed successfully",
    "device": {
        "id": "7LgjUE6hcgYawTjj",
        "name": "My Phone"
    }
}
```

---

### 4. Removing a Device

**Action:** `remove`

**Required permission:** `device_management`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deviceId` | string | Yes | Device ID |

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "remove",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj"
    }
}
```

**Example response:**
```json
{
    "success": true,
    "action": "remove",
    "message": "Device removed successfully",
    "deviceId": "7LgjUE6hcgYawTjj"
}
```

---

### 5. Retrieving Device Tokens

**Action:** `tokens`

**Required permission:** `device_tokens`

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "tokens"
}
```

**Example response:**
```json
{
    "success": true,
    "action": "tokens",
    "tokens": {
        "multiple_token": "b6V-nW_PGTc5ndpfUKoRiMx5",
        "single_token": "ZJ0pjo5nblla8xE4SeTyPAqY"
    }
}
```

---

### 6. Resetting Device Tokens

**Action:** `reset_tokens`

**Required permission:** `reset_tokens`

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "reset_tokens"
}
```

**Example response:**
```json
{
    "success": true,
    "action": "reset_tokens",
    "tokens": {
        "multiple_token": "new_token_123",
        "single_token": "new_token_456"
    }
}
```

---

### 7. Administrative Notification

**Action:** `admin_frontend_notification`

**Required permission:** `privileged_access` (administrators only)

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `message` | string | Yes | Notification text (max. 500 characters) |
| `user_id` | int | Conditional | User ID to send to |
| `ws_connection_id` | string | Conditional | WebSocket connection ID |
| `title` | string | No | Notification title (max. 100 characters) |
| `notification_type` | string | No | Type (info, success, warning, error) |
| `duration` | int | No | Display duration (ms, max. 30000) |
| `send_push` | bool | No | Send push notification |

**Example request:**
```json
{
    "token": "your_api_key",
    "action": "admin_frontend_notification",
    "data": {
        "message": "Server maintenance in 10 minutes",
        "user_id": 1,
        "title": "Maintenance",
        "notification_type": "warning",
        "duration": 10000,
        "send_push": true
    }
}
```

**Example response:**
```json
{
    "success": true,
    "action": "admin_frontend_notification",
    "message": "Notification sent to 1 connection(s)",
    "sent_to": 1,
    "target_type": "user"
}
```

---

## API Key Permissions

The following permissions can be assigned when creating an API key:

| Permission | Description |
|------------|-------------|
| `device_info` | View device information |
| `send_commands` | Send commands to devices |
| `device_management` | Rename and remove devices |
| `device_tokens` | View device tokens |
| `reset_tokens` | Reset device tokens |
| `access_proxy` | Access proxy functions |
| `security_user` | Manage user security |
| `privileged_access` | Privileged access (administrative functions) |

---

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Bad request (missing required parameters) |
| 401 | Invalid API key |
| 403 | Insufficient permissions to perform the action |
| 404 | Resource not found |
| 405 | Method not allowed |
| 500 | Internal server error |
| 503 | Device offline |

---

## Usage Examples

### cURL

```bash
# Get device list
curl -X POST https://your-server.com/api \
  -H "Content-Type: application/json" \
  -d '{
    "token": "your_api_key",
    "action": "devices"
  }'

# Send a command
curl -X POST https://your-server.com/api \
  -H "Content-Type: application/json" \
  -d '{
    "token": "your_api_key",
    "action": "command",
    "data": {
      "deviceId": "7LgjUE6hcgYawTjj",
      "command": "reboot"
    }
  }'
```

### Python

```python
import requests

API_URL = "https://your-server.com/api"
API_TOKEN = "your_api_key"

def send_command(device_id, command):
    response = requests.post(API_URL, json={
        "token": API_TOKEN,
        "action": "command",
        "data": {
            "deviceId": device_id,
            "command": command
        }
    })
    return response.json()

# Usage example
result = send_command("7LgjUE6hcgYawTjj", "reboot")
print(result)
```

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const API_URL = 'https://your-server.com/api';
const API_TOKEN = 'your_api_key';

async function getDevices() {
    const response = await axios.post(API_URL, {
        token: API_TOKEN,
        action: 'devices'
    });
    return response.data;
}

// Usage example
getDevices().then(data => console.log(data));
```

---

## Limitations

| Limitation | Value |
|------------|-------|
| Maximum request size | 1 MB |
| Maximum command length | 4096 characters |

---

## Security Notes

1. **API keys** must be stored in a secure location and never transmitted in plain text.
2. It is recommended to use **HTTPS** for all API requests.
3. **Privileged Access** should only be granted to trusted users.
4. Regularly **rotate API keys** and remove unused ones.
