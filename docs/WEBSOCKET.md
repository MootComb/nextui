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
        "global_devices": true,
        "isAdmin": true
    },
    "sessions": { ... },
    "api_keys": { ... },
    "policy": { ... },
    "tokens": { ... },
    "settings": { ... },
    "notify": { ... },
    "lockdown": { ... },
    "groups": { ... },
    "users": [ ... ]
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

### `/auth/quick/info` — Get Quick Login Info

```json
{
    "endpoint": "/auth/quick/info",
    "data": {
        "user_code": "ABC123"
    }
}
```

**Response:**
```json
{
    "success": true,
    "info": {
        "screenResolution": "1920x1080",
        "timezone": "Europe/Moscow",
        "userAgent": "Mozilla/5.0 ...",
        "platform": "Windows",
        "ip": "192.168.1.100",
        "location": "RU, Moscow"
    },
    "expires_at": 1234567890
}
```

**Errors:**
- `400` — user_code required
- `429` — Too many attempts
- `404` — Invalid or expired code
- `409` — Code already processed / Code is already being watched in another session

---

### `/auth/quick/accept` — Accept Quick Login

```json
{
    "endpoint": "/auth/quick/accept",
    "data": {
        "user_code": "ABC123"
    }
}
```

**Errors:**
- `400` — user_code required
- `429` — Too many attempts
- `404` — Invalid or expired code
- `409` — Code already processed

---

### `/auth/quick/reject` — Reject Quick Login

```json
{
    "endpoint": "/auth/quick/reject",
    "data": {
        "user_code": "ABC123"
    }
}
```

**Errors:**
- `400` — user_code required
- `404` — Invalid or expired code
- `409` — Code already processed

---

### `/auth/quick/unsubscribe` — Unsubscribe from Quick Code

```json
{
    "endpoint": "/auth/quick/unsubscribe",
    "data": {
        "user_code": "ABC123"
    }
}
```

**Errors:**
- `400` — user_code required

---

## 4.2. Groups

### `/groups/create` — Create Group

```json
{
    "endpoint": "/groups/create",
    "data": {
        "name": "My Devices",
        "color": "#bb86fc",
        "icon": "fa-folder"
    }
}
```

**Limits:**
- Group name: max 32 characters
- Max 50 groups per user
- Group name must be unique per user

**Errors:**
- `400` — Group name is required / Group name max N characters / Invalid color format / Icon name too long / Maximum N groups reached / Group name already exists
- `500` — Failed to save group

---

### `/groups/update` — Update Group

```json
{
    "endpoint": "/groups/update",
    "data": {
        "groupId": "grp_1234567890_abc12",
        "name": "New Name",
        "color": "#ff5722",
        "icon": "fa-star"
    }
}
```

**Errors:**
- `400` — groupId is required / Group name cannot be empty / Group name max N characters / Group name already exists / Invalid color format / Icon name too long
- `404` — Group not found
- `500` — Failed to save group

---

### `/groups/delete` — Delete Group

```json
{
    "endpoint": "/groups/delete",
    "data": {
        "groupId": "grp_1234567890_abc12"
    }
}
```

**Response:**
```json
{
    "success": true,
    "groupId": "grp_1234567890_abc12",
    "affectedDevices": ["device1", "device2"]
}
```

**Errors:**
- `400` — groupId is required
- `404` — Group not found
- `500` — Failed to save groups

---

### `/groups/assign` — Assign Devices to Groups

```json
{
    "endpoint": "/groups/assign",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "groupId": "grp_1234567890_abc12"
    }
}
```

**Multiple assignment:**
```json
{
    "endpoint": "/groups/assign",
    "data": {
        "deviceIds": ["device1", "device2"],
        "groupIds": ["grp_1", "grp_2"]
    }
}
```

**Limits:**
- Max 100 devices per request
- Max 10 groups per request

**Response:**
```json
{
    "success": true,
    "assigned": [
        { "deviceId": "device1", "groupId": "grp_1" }
    ],
    "failed": [
        { "deviceId": "device2", "groupId": "grp_2", "error": "Device not found" }
    ]
}
```

**Errors:**
- `400` — deviceId or deviceIds is required / groupId or groupIds is required / Max N devices per request / Max N groups per request
- `500` — Failed to save groups

---

### `/groups/unassign` — Unassign Devices from Groups

```json
{
    "endpoint": "/groups/unassign",
    "data": {
        "deviceId": "7LgjUE6hcgYawTjj",
        "groupId": "grp_1234567890_abc12"
    }
}
```

**Multiple unassignment:**
```json
{
    "endpoint": "/groups/unassign",
    "data": {
        "deviceIds": ["device1", "device2"],
        "groupIds": ["grp_1", "grp_2"]
    }
}
```

**Response:**
```json
{
    "success": true,
    "unassigned": [
        { "deviceId": "device1", "groupId": "grp_1" }
    ]
}
```

**Errors:**
- `400` — deviceId or deviceIds is required / groupId or groupIds is required / Max N devices per request / Max N groups per request
- `500` — Failed to save groups

---

### `/groups/reorder` — Reorder Groups

```json
{
    "endpoint": "/groups/reorder",
    "data": {
        "groupIds": ["grp_1", "grp_2", "grp_3"]
    }
}
```

**Response:**
```json
{
    "success": true,
    "order": ["grp_1", "grp_2", "grp_3"]
}
```

**Errors:**
- `400` — groupIds array is required / groupIds must contain all user's groups / Unknown groupId: N
- `500` — Failed to save order

---

## 4.3. Security

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

**Errors:**
- `400` — Cannot enable lockdown: no 2FA methods configured for this user / WebAuthn is not enabled for this user / TOTP is not enabled for this user / Cannot disable all 2FA methods while lockdown is active / Lockdown mode is not enabled for this session / Cannot lock session: no 2FA methods configured / No valid settings to update / Unknown action
- `401` — No active session
- `404` — User not found
- `500` — Failed to enable/disable/update lockdown

---

## 4.4. Users

### `/users/list` — List Users

```json
{
    "endpoint": "/users/list",
    "data": {}
}
```

**Response:**
```json
{
    "success": true,
    "users": [
        {
            "id": 1,
            "username": "admin",
            "role": "admin",
            "enabled": true,
            "totp_enabled": false,
            "webauthn_enabled": false,
            "global_devices": true,
            "created_at": 1234567890,
            "can_create_api_keys": true,
            "can_add_devices": true,
            "max_devices": -1,
            "max_api_keys": -1
        }
    ]
}
```

**Errors:**
- `403` — Admin privileges required
- `404` — User not found
- `500` — Failed to get users list

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

**Errors:**
- `400` — User ID is required
- `403` — Admin privileges required
- `404` — User not found / Current user not found

---

### `/users/save` — Save User

**Create new user (userId: 0):**
```json
{
    "endpoint": "/users/save",
    "data": {
        "userId": 0,
        "username": "new_user",
        "password": "password123",
        "role": "user",
        "enabled": true,
        "totp_enabled": false,
        "webauthn_enabled": false,
        "global_devices": false,
        "can_create_api_keys": true,
        "can_add_devices": true,
        "max_devices": 10,
        "max_api_keys": 5
    }
}
```

**Update existing user:**
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

**Errors:**
- `400` — Username and password are required / invalid username/password / Username already exists / Failed to save user
- `403` — Admin privileges required / Cannot modify host user
- `404` — Current user not found / User not found
- `500` — Failed to get updated user data

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

**Errors:**
- `400` — User ID is required / Cannot delete yourself
- `403` — Admin privileges required / Cannot delete host user
- `500` — Failed to delete user

---

## 4.5. Devices

### `/devices/tokens/refresh` — Refresh Tokens

```json
{
    "endpoint": "/devices/tokens/refresh",
    "data": {}
}
```

**Errors:**
- `404` — User not found
- `500` — Failed to refresh tokens

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

**Errors:**
- `400` — No valid fields to update / Invalid action. Use: get or update
- `403` — Token policy management is disabled for this user
- `404` — User not found
- `500` — Failed to update policy / Failed to get updated policy

---

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

**Errors:**
- `403` — User account is disabled / Invalid admin session
- `404` — User not found

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

**Errors:**
- `400` — Device ID is required
- `404` — Device not found

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

**Errors:**
- `400` — Device ID is required / New name is required / invalid device name
- `403` — Permission denied: need 'danger' right to rename device
- `404` — Device not found
- `500` — Failed to rename device

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

**Errors:**
- `400` — Device ID is required
- `403` — Permission denied: need 'danger' right to remove device
- `500` — Failed to remove device

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

**Special command prefixes:**
- `webhook:add|command` — add webhook
- `webhook:remove|webhook_id` — remove webhook
- `stream:type:on|off` — stream control
- `policy:setting|true|false|toggle` — policy management
- `interval:type|value` — interval management
- `users:add|userID|perms|transfer` — user access
- `users:remove|userID` — remove user access
- `users:update|userID|perms` — update user access
- `users:transfer|userID` — transfer ownership
- `sending:type|true|false` — sending settings
- `delete-command:command` — delete queued command
- `upgrade:architecture` — upgrade client binary
- `initial-state:policy` — set initial state policy

**Errors:**
- `400` — Device ID is required / Command is required / invalid command format / Unknown command
- `403` — Permission denied / need 'danger' right required to queue commands
- `404` — Device not found
- `500` — Error with MQTT / Failed to marshal commands / Failed to update device
- `503` — Device is offline and command could not be sent

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

**Add with absolute timestamp:**
```json
{
    "endpoint": "/devices/schedule",
    "data": {
        "action": "add",
        "deviceId": "7LgjUE6hcgYawTjj",
        "command": "reboot",
        "scheduleAt": 1234567890000
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

**Response (add):**
```json
{
    "success": true,
    "message": "Command scheduled for 2024-01-01T12:00:00Z",
    "scheduledId": "sched_123",
    "scheduleAt": 1234567890000,
    "command": "reboot",
    "deviceId": "7LgjUE6hcgYawTjj",
    "requestId": "cmd_123",
    "executesIn": 3600000
}
```

**Limits:**
- Max schedule delay: 7 days
- Cannot schedule in the past

**Errors:**
- `400` — deviceId is required / command is required / scheduleAt or delaySeconds is required / Cannot schedule command in the past / Maximum schedule delay is 7 days / scheduledId is required / Unknown action. Use: add, remove
- `403` — Permission denied: need 'danger' right
- `404` — Device not found / No scheduled commands found / Scheduled command N not found
- `500` — Failed to parse scheduled commands

---

## 4.6. Terminal

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

**Limits:**
- Max terminal input: 4KB per message

**Errors:**
- `400` — Device ID is required / Session ID is required / Data is required / Unknown terminal action
- `403` — Permission denied: need 'terminal' right
- `404` — Device is offline / Terminal session not found
- `413` — Terminal input too large (max 4KB)
- `500` — Failed to start terminal on device / Failed to send terminal input / Failed to resize terminal

---

## 4.7. Files

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

**Errors:**
- `400` — deviceId is required / path is required / invalid file path / invalid filename
- `403` — Permission denied: need 'danger' or 'manage' right
- `404` — Device not found
- `500` — Failed to send command to device
- `503` — Device is offline

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

**Response:**
```json
{
    "success": true,
    "message": "Download session created, waiting for device",
    "sessionId": "download_456",
    "requestId": "download_123"
}
```

**Errors:**
- `400` — deviceId is required / path is required / invalid file path
- `403` — Permission denied: need 'danger', 'manage' or 'explorer' right
- `404` — Device not found
- `500` — Failed to send command to device
- `503` — Device is offline

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

**Response:**
```json
{
    "success": true,
    "message": "Transfer cancelled",
    "sessionId": "upload_456",
    "requestId": "cancel_123"
}
```

**Errors:**
- `400` — sessionId is required
- `403` — Permission denied
- `404` — Session not found

---

## 4.8. Settings

### `/settings/global` — Global Settings

```json
{
    "endpoint": "/settings/global",
    "data": {
        "global_devices": true
    }
}
```

**Errors:**
- `400` — global_devices is required
- `403` — Admin privileges required
- `404` — User not found
- `500` — Failed to update settings

---

## 4.9. Notifications

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

**Available settings keys:**
- `device_online`, `device_offline`, `new_device_added`, `device_renamed`, `device_deleted`
- `login_successful`, `login_failed`, `password_changed`, `username_changed`, `2fa_changed`
- `cpu_high_usage`, `memory_high_usage`, `disk_almost_full`, `temperature_high`, `battery_low`
- `api_key_created_delete`, `api_key_updated`, `api_key_used`
- `device_token_updated`, `access_policy_updated`

**Errors:**
- `400` — Subscription data required / Settings data required / No valid settings to update / Unknown action. Use: subscribe, unsubscribe, update
- `500` — Failed to marshal subscription / Failed to save subscription / Failed to remove subscription / Failed to update settings

---

## 4.10. API Keys

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

**Create SSH key:**
```json
{
    "endpoint": "/keys",
    "data": {
        "action": "create",
        "name": "My SSH Key",
        "type": "ssh",
        "expiry_days": 30,
        "permissions": { ... }
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

**Available permissions:**
- `device_info`, `send_commands`, `device_management`, `device_tokens`, `reset_tokens`
- `access_proxy`, `security_user`, `privileged_access`, `device_groups`

**Limits:**
- Max 10 API keys per user
- Max API keys per user (configurable, `max_api_keys`)

**Errors:**
- `400` — key type is required / invalid key type / API key with name already exists / expiry days cannot be negative / maximum 10 API keys per user / invalid API key name / API key ID is required / Invalid API key ID format / Invalid permission
- `403` — Creating API keys is disabled for this user / Access denied
- `404` — User not found / API key not found
- `500` — failed to generate SSH key pair / failed to parse public key / failed to save API key / failed to marshal permissions / Failed to toggle API key

---

## 4.11. Logs

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

**Log types:**
- `commands` — device commands
- `auth` — authentication
- `2fa` — TOTP/WebAuthn changes
- `devices` — device management
- `api_keys` — API key management
- `sessions` — session management
- `webhooks` — webhook calls

**Delete (admin only):**
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

**Errors:**
- `400` — Timestamp is required / User ID is required / Unknown action. Use: list, remove
- `403` — Admin privileges required
- `404` — User not found / log not found
- `500` — failed to get logs

---

## 4.12. Server → Client Messages

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
| `quick_login_cancelled` | Quick login cancelled |
| `stream_started` | Stream started |
| `stream_stopped` | Stream stopped |
| `stream_ready` | Stream ready |
| `stream_error` | Stream error |
| `groups:created` | Group created |
| `groups:updated` | Group updated |
| `groups:deleted` | Group deleted |
| `groups:assigned` | Devices assigned to groups |
| `groups:unassigned` | Devices unassigned from groups |
| `groups:reordered` | Groups reordered |
| `error` | Error message |

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

**Example groups event:**
```json
{
    "type": "groups:created",
    "payload": {
        "group": {
            "id": "grp_1234567890_abc12",
            "name": "My Devices",
            "color": "#bb86fc",
            "icon": "fa-folder",
            "createdAt": 1234567890
        }
    }
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
