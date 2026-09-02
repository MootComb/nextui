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
| `enabled` | string | `"1"` or `"0"` |
| `totp_enabled` | string | TOTP enabled |
| `webauthn_enabled` | string | WebAuthn enabled |
| `totp_secret` | string | TOTP secret (base32) |
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
| `long_lived` | string | `"1"` or `"0"` |
| `active` | string | `"1"` or `"0"` |
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
| `simple_online` | string | `"1"` or `"0"` |
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
| `advanced_localIp` | string | Local IPv4 |
| `advanced_publicIp` | string | Public IP |
| `advanced_macAddress` | string | MAC address |
| `advanced_location` | string | Geolocation |
| `advanced_processor` | string | Processor |
| `advanced_video` | string | Graphics card |
| `advanced_ram` | string | RAM (GB) |
| `advanced_kernel` | string | Kernel version |
| `advanced_architecture` | string | Architecture |
| `advanced_updatePolicy` | string | Update policy |
| `advanced_intervals` | string | JSON with intervals |
| `advanced_commands` | string | JSON array of commands |
| `advanced_scheduled` | string | JSON array of scheduled commands |
| `advanced_webhooks` | string | JSON array of webhooks |
| `advanced_users` | string | JSON array of access rights |
| `advanced_sendings_*` | string | Sending settings |

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
| `type` | string | `regular` or `ssh` |
| `public_key` | string | Public key (for SSH) |
| `active` | string | `"1"` or `"0"` |
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
| `cpu_high_usage` | string | High CPU usage |
| `memory_high_usage` | string | High memory usage |
| `disk_almost_full` | string | Disk almost full |
| `battery_low` | string | Low battery |
| `login_successful` | string | Successful login |
| `login_failed` | string | Failed login |
| `password_changed` | string | Password changed |
| `api_key_created_delete` | string | API key created/deleted |

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
