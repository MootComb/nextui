# WebSocket Documentation

## Overview

WebSocket is used for real-time communication between the browser and the server. The connection is established after successful authentication.

---

## Connection

**URL:** `wss://{domain}/websocket`

**Requirements:**
- The `SID` session cookie must be set
- Origin must match the server domain
- Only HTTPS is supported (wss://)

---

## Message Format

## Request (Client → Server)

```json
{
    "endpoint": "/devices/list",
    "data": {
        "param1": "value1",
        "param2": "value2",
        "requestId": "uuid"
    }
}
```

## Response (Server → Client)

```json
{
    "type": "response_type",
    "data": { ... },
    "requestId": "uuid",
    "success": true
}
```

## Error Response

```json
{
    "success": false,
    "error": "Error message",
    "status": 404,
    "requestId": "uuid"
}
```

---

## Endpoints

## 4.1. Authentication

### `/auth/init` — Initialization

**Request:**
```json
{
    "endpoint": "/auth/init",
    "data": {}
}
```

**Response:**
```json
{
    "success": true,
    "user": {
        "id": 1,
        "username": "admin",
        "role": "admin",
        "enabled": true,
        "totp_enabled": false,
        "webauthn_enabled": false,
        "global_devices": true
    },
    "sessions": { ... },
    "api_keys": { ... },
    "policy": { ... },
    "tokens": { ... },
    "settings": { ... }
}
```

---

### `/auth/sessions` — Session Management

**List sessions:**
```json
{
    "endpoint": "/auth/sessions",
    "data": {
        "action": "list"
    }
}
```

**Rename session:**
```json
{
    "endpoint": "/auth/sessions",
    "data": {
        "action": "rename",
        "session_id": "1234567890",
        "name": "Home PC"
    }
}
```

**Terminate session:**
```json
{
    "endpoint": "/auth/sessions",
    "data": {
        "action": "terminate",
        "session_id": "1234567890"
    }
}
```

---

### `/auth/change/username` — Change Username

```json
{
    "endpoint": "/auth/change/username",
    "data": {
        "newUsername": "new_admin"
    }
}
```

---

### `/auth/change/password` — Change Password

```json
{
    "endpoint": "/auth/change/password",
    "data": {
        "currentPassword": "old_password",
        "newPassword": "new_password"
    }
}
```

---

### `/auth/totp/generate` — Generate TOTP

```json
{
    "endpoint": "/auth/totp/generate",
    "data": {}
}
```

**Response:**
```json
{
    "success": true,
    "secret": "JBSWY3DPEHPK3PXP",
    "provisioning_uri": "otpauth://totp/NextUI:admin?secret=..."
}
```

---

### `/auth/totp/verify` — Verify TOTP

```json
{
    "endpoint": "/auth/totp/verify",
    "data": {
        "code": "123456",
        "action": "enable",
        "secret": "JBSWY3DPEHPK3PXP"
    }
}
```

---

### `/auth/totp/enable` — Enable TOTP

```json
{
    "endpoint": "/auth/totp/enable",
    "data": {}
}
```

---

### `/auth/totp/disable` — Disable TOTP

```json
{
    "endpoint": "/auth/totp/disable",
    "data": {}
}
```

---

### `/auth/webauthn/register/start` — Start WebAuthn Registration

```json
{
    "endpoint": "/auth/webauthn/register/start",
    "data": {}
}
```

**Response:**
```json
{
    "success": true,
    "options": {
        "rp": { "name": "NextUI", "id": "example.com" },
        "user": { "id": "...", "name": "admin" },
        "challenge": "...",
        "pubKeyCredParams": [...]
    }
}
```

---

### `/auth/webauthn/register/finish` — Complete WebAuthn Registration

```json
{
    "endpoint": "/auth/webauthn/register/finish",
    "data": {
        "credential": {
            "id": "credential_id",
            "rawId": "...",
            "type": "public-key",
            "response": {
                "attestationObject": "...",
                "clientDataJSON": "..."
            }
        }
    }
}
```

---

### `/auth/webauthn/disable` — Disable WebAuthn

```json
{
    "endpoint": "/auth/webauthn/disable",
    "data": {}
}
```

---

### `/auth/login-as` — Login as Another User

```json
{
    "endpoint": "/auth/login-as",
    "data": {
        "userId": 2
    }
}
```

---

## 4.2. Security

### `/lockdown` — Lockdown Management

**Enable:**
```json
{
    "endpoint": "/lockdown",
    "data": {
        "action": "enable"
    }
}
```

**Disable:**
```json
{
    "endpoint": "/lockdown",
    "data": {
        "action": "disable"
    }
}
```

**Edit settings:**
```json
{
    "endpoint": "/lockdown",
    "data": {
        "action": "edit",
        "webauthn": true,
        "totp": false,
        "time": 3600
    }
}
```

**Lock session:**
```json
{
    "endpoint": "/lockdown",
    "data": {
        "action": "lock"
    }
}
```

---

## 4.3. Users

### `/users/list` — List Users

```json
{
    "endpoint": "/users/list",
    "data": {}
}
```

---

### `/users/get` — Get User

```json
{
    "endpoint": "/users/get",
    "data": {
        "userId": 1
    }
}
```

---

### `/users/save` — Save User

```json
{
    "endpoint": "/users/save",
    "data": {
        "userId": 1,
        "username": "new_admin",
        "password": "new_password",
        "role": "admin",
        "enabled": true,
        "totp_enabled": false,
        "webauthn_enabled": false,
        "global_devices": true,
        "can_create_api_keys": true,
        "can_add_devices": true,
        "max_devices": 10,
        "max_api_keys": 5
    }
}
```

---

### `/users/delete` — Delete User

```json
{
    "endpoint": "/users/delete",
    "data": {
        "userId": 2
    }
}
```

---

## 4.4. Devices

### `/devices/list` — List Devices

```json
{
    "endpoint": "/devices/list",
    "data": {}
}
```

**Response:**
```json
{
    "success": true,
    "devices": [
        {
            "id": "7LgjUE6hcgYawTjj",
            "name": "Infinix note 30",
            "system": "android",
            "online": true,
            "cpu": 14,
            "memory": 99,
            "disk": 44
        }
    ]
}
```

---

### `/devices/get` — Device Details

```json
{
    "endpoint": "/devices/get",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj"
    }
}
```

**Response:**
```json
{
    "success": true,
    "device": {
        "id": "7LgjUE6hcgYawTjj",
        "name": "Infinix note 30",
        "system": "android",
        "online": true,
        "cpu": 14,
        "memory": 63,
        "disk": 44,
        "swap": 94,
        "network": 0,
        "temp": 50,
        "hostname": "Infinix note 30",
        "uptime": "16d 5h 59m 38s",
        "battery": 54,
        "processor": "mt6789",
        "ram": "7.5GB",
        "kernel": "5.10.260",
        "architecture": "arm64",
        "charts": {
            "1min": { ... },
            "1hour": { ... }
        },
        "access": ["system", "manage", "terminal"],
        "users": [...],
        "processes": { ... },
        "logs": { ... },
        "applications": { ... }
    }
}
```

---

### `/devices/rename` — Rename Device

```json
{
    "endpoint": "/devices/rename",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "newName": "My Phone"
    }
}
```

---

### `/devices/remove` — Remove Device

```json
{
    "endpoint": "/devices/remove",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj"
    }
}
```

---

### `/devices/command` — Send Command

```json
{
    "endpoint": "/devices/command",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "command": "restart",
        "requestId": "cmd_123"
    }
}
```

**Response:**
```json
{
    "success": true,
    "message": "Command sent",
    "requestId": "cmd_123"
}
```

---

### `/devices/schedule` — Scheduler

**Add:**
```json
{
    "endpoint": "/devices/schedule",
    "data": {
        "action": "add",
        "deviceId": "7LgjUE6hcgYawTjj",
        "command": "reboot",
        "delaySeconds": 3600
    }
}
```

**Remove:**
```json
{
    "endpoint": "/devices/schedule",
    "data": {
        "action": "remove",
        "deviceId": "7LgjUE6hcgYawTjj",
        "scheduledId": "sched_123"
    }
}
```

---

### `/devices/tokens/refresh` — Refresh Tokens

```json
{
    "endpoint": "/devices/tokens/refresh",
    "data": {}
}
```

---

### `/devices/tokens/policy` — Token Policy

```json
{
    "endpoint": "/devices/tokens/policy",
    "data": {
        "action": "update",
        "allow_url_token": true,
        "allowed_platforms": {
            "linux": true,
            "windows": true,
            "android": true
        },
        "time_restrictions": {
            "enabled": true,
            "limit_minutes": 5
        },
        "usage_restrictions": {
            "enabled": true,
            "max_uses": 5
        }
    }
}
```

---

## 4.5. Terminal

### `/devices/terminal` — Terminal Management

**Connect:**
```json
{
    "endpoint": "/devices/terminal",
    "data": {
        "action": "connect",
        "deviceId": "7LgjUE6hcgYawTjj",
        "cols": 80,
        "rows": 24
    }
}
```

**Input:**
```json
{
    "endpoint": "/devices/terminal",
    "data": {
        "action": "input",
        "sessionId": "term_123",
        "data": "ls -la\n"
    }
}
```

**Resize:**
```json
{
    "endpoint": "/devices/terminal",
    "data": {
        "action": "resize",
        "sessionId": "term_123",
        "cols": 120,
        "rows": 30
    }
}
```

**Disconnect:**
```json
{
    "endpoint": "/devices/terminal",
    "data": {
        "action": "disconnect",
        "sessionId": "term_123"
    }
}
```

---

## 4.6. Files

### `/file/upload` — Upload File

```json
{
    "endpoint": "/file/upload",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "path": "/home/user/file.txt",
        "filename": "file.txt",
        "requestId": "upload_123"
    }
}
```

**Response:**
```json
{
    "success": true,
    "sessionId": "upload_456"
}
```

---

### `/file/download` — Download File

```json
{
    "endpoint": "/file/download",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "path": "/home/user/file.txt",
        "requestId": "download_123"
    }
}
```

---

### `/file/cancel` — Cancel Transfer

```json
{
    "endpoint": "/file/cancel",
    "data": {
        "sessionId": "upload_456",
        "requestId": "cancel_123"
    }
}
```

---

## 4.7. Settings

### `/settings/global` — Global Settings

```json
{
    "endpoint": "/settings/global",
    "data": {
        "global_devices": true
    }
}
```

---

### `/auth/session/longlived` — Long-Lived Session

```json
{
    "endpoint": "/auth/session/longlived",
    "data": {
        "long_lived": true
    }
}
```

---

## 4.8. Notifications

### `/notify` — Notification Settings

**Subscribe:**
```json
{
    "endpoint": "/notify",
    "data": {
        "action": "subscribe",
        "subscription": {
            "endpoint": "https://...",
            "keys": { "auth": "...", "p256dh": "..." }
        }
    }
}
```

**Unsubscribe:**
```json
{
    "endpoint": "/notify",
    "data": {
        "action": "unsubscribe"
    }
}
```

**Update settings:**
```json
{
    "endpoint": "/notify",
    "data": {
        "action": "update",
        "settings": {
            "device_online": true,
            "device_offline": false,
            "cpu_high_usage": true
        }
    }
}
```

---

## 4.9. API Keys

### `/keys` — API Key Management

**List:**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "list"
    }
}
```

**Create:**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "create",
        "name": "My API Key",
        "type": "regular",
        "expiry_days": 30,
        "permissions": {
            "device_info": true,
            "send_commands": true,
            "device_management": false
        }
    }
}
```

**Edit:**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "edit",
        "id": "api_key_123",
        "name": "New Name",
        "permissions": { ... }
    }
}
```

**Toggle (enable/disable):**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "toggle",
        "id": "api_key_123"
    }
}
```

**Delete:**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "delete",
        "id": "api_key_123"
    }
}
```

---

## 4.10. Logs

### `/logs` — Log Management

**List:**
```json
{
    "endpoint": "/logs",
    "data": {
        "action": "list",
        "limit": 50,
        "offset": 0,
        "type": "commands"
    }
}
```

**Delete:**
```json
{
    "endpoint": "/logs",
    "data": {
        "action": "remove",
        "user_id": 1,
        "timestamp": 1234567890
    }
}
```

---

## 4.11. Server → Client Messages

**Response Types:**

| Type | Description |
|------|-------------|
| `connected` | Successful connection |
| `devices-list` | Device list update |
| `devices-get` | Device details update |
| `device-remove` | Device removed |
| `notification` | Notification |
| `terminal_output` | Terminal output |
| `file/upload/ready` | Ready for upload |
| `file/upload/complete` | Upload completed |
| `file/upload/error` | Upload error |
| `file/download/ready` | Ready for download |
| `file/download/complete` | Download completed |
| `file/download/error` | Download error |
| `file/cancel/complete` | Cancellation completed |

**Example notification:**
```json
{
    "type": "notification",
    "notificationType": "success",
    "notificationSubtype": "command_success",
    "message": "Command 'reboot' executed successfully",
    "requestId": "cmd_123",
    "command": "reboot",
    "deviceId": "7LgjUE6hcgYawTjj"
}
```

---

## WebSocket Error Handling

| Code | Reason |
|------|--------|
| 1000 | Normal closure |
| 1006 | Abnormal closure (connection lost) |
| 1008 | Policy violation (Account disabled, Session locked) |
| 1011 | Internal server error |

---

## WebSocket Connection Flow

```
1. Client: HTTP Login → Receives SID cookie
2. Client: WebSocket Upgrade Request (with SID cookie)
3. Server: Validate session, user, limits
4. Server: WebSocket Accept
5. Server: Sends "connected" message
6. Client: Sends "/auth/init" request
7. Server: Sends user data
8. Client: Begins operation
```

---

## Browser Usage Example

```javascript
const ws = new WebSocket('wss://example.com/websocket');

ws.onopen = () => {
    // Send request
    ws.send(JSON.stringify({
        endpoint: '/devices/list',
        data: { requestId: crypto.randomUUID() }
    }));
};

ws.onmessage = (event) => {
    const response = JSON.parse(event.data);
    
    if (response.type === 'devices-list') {
        console.log('Devices:', response.devices);
    }
    
    if (response.requestId && pendingRequests.has(response.requestId)) {
        const { resolve } = pendingRequests.get(response.requestId);
        resolve(response);
    }
};
```
