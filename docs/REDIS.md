# Redis Documentation

### Overview

Redis is used as the primary data store. All data is stored as Hash, Set, String.

---

### Key Schema

#### 2.1. Users (`user:*`)

**Key:** `user:{id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `id` | string | User ID |
| `username` | string | Username |
| `password_hash` | string | Argon2id password hash |
| `role` | string | Role: `admin` or `user` |
| `enabled` | string | User enabled |
| `totp_enabled` | string | TOTP enabled |
| `webauthn_enabled` | string | WebAuthn enabled |
| `totp_secret` | string | TOTP secret |
| `webauthn_credentials` | string | JSON array of WebAuthn data |
| `multiple_token` | string | Token for multiple devices |
| `single_token` | string | Token for single devices |
| `global_devices` | string | Global access to devices |
| `created_at` | string | Unix timestamp (ms) |
| `can_create_api_keys` | string | Can create API keys |
| `can_add_devices` | string | Can add devices |
| `max_devices` | string | Maximum devices (-1 = unlimited) |
| `max_api_keys` | string | Maximum API keys |

**Example:**
```redis
HSET user:1 id "1" username "admin" password_hash "$argon2id$..." role "admin" enabled "1"
```

---

#### 2.2. User Indexes

**Key:** `user_index:username:{username}` (String)
**Value:** `{user_id}`

```redis
SET user_index:username:admin "1"
```

**Key:** `user_index:token:{token}` (String)
**Value:** `{user_id}:{type}`

```redis
SET user_index:token:b6V-nW_PGTc5ndpfUKoRiMx5 "0:multiple"
SET user_index:token:ZJ0pjo5nblla8xE4SeTyPAqY "0:single"
```

---

#### 2.3. Sessions (`session:*`)

**Key:** `session:{id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `user_id` | string | User ID |
| `admin_user_id` | string | Admin user ID (when login-as) |
| `device_info` | string | JSON with device information |
| `last_activity` | string | Unix timestamp (ms) |
| `created_at` | string | Unix timestamp (ms) |
| `expires_at` | string | Unix timestamp (ms) |
| `long_lived` | string | Session long-term |
| `active` | string | Session online |
| `lockdown_enabled` | string | Lockdown enabled |
| `lockdown` | string | Session locked |
| `lockdown_webauthn` | string | WebAuthn required to unlock |
| `lockdown_totp` | string | TOTP required to unlock |
| `lockdown_time` | string | Lockdown time (sec) |

**Index:** `session_index:user:{user_id}` (Set)
```redis
SADD session_index:user:1 "session_abc123"
```

---

#### 2.4. Devices (`device:*`)

**Key:** `device:{id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `id` | string | Device ID |
| `owner` | string | Owner ID |
| `device_token_type` | string | Token type (Multiple/Single) |
| `device_token_value` | string | Token value |
| `mqtt_username` | string | MQTT username (usually = device_id) |
| `mqtt_password` | string | MQTT password |
| `created_at` | string | Unix timestamp (ms) |
| `last_update` | string | Unix timestamp (ms) |
| `location_updated` | string | Unix timestamp (ms) |
| `last_offline` | string | Unix timestamp (ms) |

**Simple fields:**

| Field | Type | Description |
|------|-----|----------|
| `simple_name` | string | Device name |
| `simple_system` | string | OS (linux, android, windows) |
| `simple_online` | string | Device online |
| `simple_cpu` | string | CPU usage (%) |
| `simple_memory` | string | Memory usage (%) |
| `simple_disk` | string | Disk usage (%) |
| `simple_swap` | string | Swap usage (%) |
| `simple_network` | string | Network activity (MB/s) |
| `simple_temp` | string | Temperature (°C) |

**Advanced fields:**

| Field | Type | Description |
|------|-----|----------|
| `advanced_hostname` | string | Hostname |
| `advanced_uptime` | string | Uptime |
| `advanced_battery` | string | Battery level (%) |
| `advanced_time` | string | Current time |
| `advanced_timezone` | string | Timezone |
| `advanced_location` | string | Geolocation |
| `advanced_localIp` | string | Local IPv4 |
| `advanced_publicIp` | string | Public IP |
| `advanced_macAddress` | string | MAC address |
| `advanced_localIpv6` | string | Local IPv6 |
| `advanced_dns` | string | DNS servers |
| `advanced_ping` | string | Ping to server |
| `advanced_processor` | string | Processor |
| `advanced_video` | string | Graphics card |
| `advanced_ram` | string | RAM (GB) |
| `advanced_swaplist` | string | Swap list |
| `advanced_kernel` | string | Kernel version |
| `advanced_architecture` | string | Architecture |
| `advanced_homeDirectory` | string | Home directory |
| `advanced_user` | string | Current user |
| `advanced_brightness` | string | Screen brightness |
| `advanced_volume` | string | Volume level |
| `advanced_mediaControl` | string | Media control state |
| `advanced_mediaPlayPause` | string | Media play/pause state |
| `advanced_clipboard` | string | Clipboard content |
| `advanced_explorerDirectory` | string | Explorer current directory |
| `advanced_explorerFiles` | string | Explorer file list |
| `advanced_applications` | string | Applications list |
| `advanced_logs` | string | System logs |
| `advanced_processes` | string | Processes list |
| `advanced_permissions` | string | Device permissions |
| `advanced_intervals` | string | JSON with update intervals |
| `advanced_updatePolicy` | string | Update policy |
| `advanced_commands` | string | JSON array of queued commands |
| `advanced_scheduled` | string | JSON array of scheduled commands |
| `advanced_webhooks` | string | JSON array of webhooks |
| `advanced_sendings_simple` | string | Send simple updates enabled |
| `advanced_sendings_advanced` | string | Send advanced updates enabled |
| `advanced_sendings_stop` | string | Send stop updates enabled |
| `advanced_policy_*` | string | Policy settings (see below) |

**Policy fields (`advanced_policy_*`):**

| Field | Type | Description |
|------|-----|----------|
| `advanced_policy_signals` | string | Signals policy |
| `advanced_policy_autostart` | string | Autostart policy |
| `advanced_policy_hiding` | string | Hiding policy |
| `advanced_policy_block` | string | Block policy |
| `advanced_policy_critical` | string | Critical policy |
| `advanced_policy_mouse` | string | Mouse policy |
| `advanced_policy_keyboard` | string | Keyboard policy |
| `advanced_policy_sound` | string | Sound policy |
| `advanced_policy_screen` | string | Screen policy |
| `advanced_policy_family` | string | Family policy |

**Family control fields (`advanced_family_*`):**

| Field | Type | Description |
|------|-----|----------|
| `advanced_family_lock` | string | Family lock state |
| `advanced_family_screen_time` | string | Screen time (min) |
| `advanced_family_daily_limit` | string | Daily limit (min) |
| `advanced_family_schedules` | string | Schedules |
| `advanced_family_blocking_apps` | string | Blocked apps list |
| `advanced_family_pause_minutes` | string | Pause remaining (min) |
| `advanced_family_update_interval` | string | Update interval (min) |
| `advanced_family_last_update` | string | Last update time |
| `advanced_family_current_apps` | string | Current active app |
| `advanced_family_app_usage` | string | App usage stats |
| `advanced_family_options` | string | Family control options |

**Access control:**

**Field:** `users` (String - JSON)
JSON array of `UserAccess` objects

```json
[
    {
        "userId": 2,
        "username": "user2",
        "permissions": ["system", "manage"],
        "grantedByID": 1,
        "grantedByName": "admin",
        "grantedAt": 1234567890
    }
]
```

> Note: The field is stored as `users` (without `advanced_` prefix).

---

#### 2.5. Charts (`charts:*`)

**Key:** `charts:{device_id}` (String - JSON)

```json
{
    "1min": {
        "timestamps": [1234567890, 1234567891, ...],
        "cpu": [14, 15, 13, ...],
        "memory": [99, 98, 99, ...],
        "disk": [44, 44, 45, ...],
        "swap": [94, 93, 94, ...],
        "battery": [54, 53, 54, ...],
        "temp": [50, 51, 50, ...]
    },
    "1hour": {
        "timestamps": [1234567890, ...],
        "cpu": [14, 15, ...]
    }
}
```

---

#### 2.6. API Keys (`api_key:*`)

**Key:** `api_key:{id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `user_id` | string | Owner ID |
| `name` | string | Key name |
| `type` | string | Type key: `ssh` or `regular` |
| `public_key` | string | Public key (for SSH) |
| `active` | string | Key enabled |
| `permissions` | string | JSON with permissions |
| `created_at` | string | Unix timestamp (ms) |
| `expires_at` | string | Unix timestamp (ms) |
| `last_used` | string | Unix timestamp (ms) |
| `usage_count` | string | Usage count |

**Index:** `api_keys:user:{user_id}` (Set)

```redis
SADD api_keys:user:1 "api_key_123"
```

---

#### 2.7. Webhook Indexes (`webhook_index:*`)

**Key:** `webhook_index:{webhook_id}` (String)
**Value:** `{device_id}`

```redis
SET webhook_index:webhook_123 "device_456"
```

---

#### 2.8. Scheduled Commands (`scheduled:*`)

**Key:** `scheduled:{id}` (String - JSON)

```json
{
    "id": "sched_123",
    "device_id": "device_456",
    "command": "reboot",
    "schedule_at": 1234567890,
    "created_at": 1234567800,
    "request_id": "req_123",
    "user_id": 1
}
```

---

#### 2.9. Notifications (`notify:*`)

**Key:** `notify:{session_id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `subscription` | string | Web Push subscription JSON |
| `device_online` | string | Device online notification |
| `device_offline` | string | Device offline notification |
| `new_device_added` | string | New device added |
| `device_renamed` | string | Device renamed |
| `device_deleted` | string | Device deleted |
| `login_successful` | string | Successful login |
| `login_failed` | string | Failed login |
| `password_changed` | string | Password changed |
| `username_changed` | string | Username changed |
| `2fa_changed` | string | 2FA settings changed |
| `cpu_high_usage` | string | High CPU usage |
| `memory_high_usage` | string | High memory usage |
| `disk_almost_full` | string | Disk almost full |
| `temperature_high` | string | High temperature |
| `battery_low` | string | Low battery |
| `api_key_created_delete` | string | API key created/deleted |
| `api_key_updated` | string | API key updated |
| `api_key_used` | string | API key used |
| `device_token_updated` | string | Device token updated |
| `access_policy_updated` | string | Access policy updated |

---

#### 2.10. Logs (`user_logs:*`)

**Key:** `user_logs:{user_id}` (Hash)
**Field:** `{timestamp}` → JSON log

**Key:** `user_logs:{user_id}:ids` (List)
List of timestamps for size limiting

```redis
HSET user_logs:1 "1234567890" '{"timestamp":1234567890,"type":"commands",...}'
LPUSH user_logs:1:ids "1234567890"
LTRIM user_logs:1:ids 0 99
```

**Limits:** Max 100 logs per user

**Log entry structure:**

```json
{
    "timestamp": 1234567890,
    "type": "commands",
    "ip": "192.168.1.100",
    "success": true,
    "username": "admin",
    "data": { ... }
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

---

#### 2.11. Quick Login (`quick:*`)

**Key:** `quick:code:{user_code}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `ws_token` | string | WebSocket token |
| `status` | string | Status: `pending`, `approved`, `rejected` |
| `user_id` | string | User ID (after approval) |
| `device_info` | string | JSON with device information |
| `ip` | string | Client IP |
| `created_at` | string | Unix timestamp (ms) |
| `expires_at` | string | Unix timestamp (ms) |

**TTL:** `config.QuickLogin.TTLSeconds` seconds (default: 300)

**Key:** `quick:token:{ws_token}` (String)
**Value:** `{user_code}`

```redis
HSET quick:code:ABC123 ws_token "..." status "pending" user_id "0" ...
SET quick:token:ws_token_abc123 "ABC123"
```

---

#### 2.12. Token Policy (`user_token_policy:*`)

**Key:** `user_token_policy:{user_id}` (Hash)

| Field | Type | Description |
|------|-----|----------|
| `allow_url_token` | string | Allow URL tokens |
| `allowed_platforms` | string | JSON with allowed platforms |
| `time_restrictions` | string | JSON with time restrictions |
| `usage_restrictions` | string | JSON with usage restrictions |
| `current_use_count` | string | Current usage count |

**Example:**

```json
{
    "allow_url_token": "1",
    "allowed_platforms": "{\"linux\":true,\"windows\":true,\"android\":true}",
    "time_restrictions": "{\"enabled\":false,\"limit_minutes\":5}",
    "usage_restrictions": "{\"enabled\":false,\"max_uses\":1}",
    "current_use_count": "0"
}
```

---

#### 2.13. Groups (`groups:*`)

**Key:** `groups:{user_id}` (String - JSON)

```json
{
    "groups": [
        {
            "id": "grp_1234567890_abc12",
            "name": "My Devices",
            "color": "#bb86fc",
            "icon": "fa-folder",
            "createdAt": 1234567890
        }
    ],
    "deviceGroups": {
        "device_id_1": ["grp_1", "grp_2"]
    },
    "order": ["grp_1", "grp_2"],
    "updatedAt": 1234567890
}
```

**Limits:**
- Max 50 groups per user
- Group name max 32 characters
- Max 100 devices per assign request
- Max 10 groups per assign request

---

#### 2.14. System (`initialized`)

**Key:** `initialized` (String)
**Value:** `"1"`

Flag indicating that the database has been initialized with default users and devices.

```redis
SET initialized "1"
```
