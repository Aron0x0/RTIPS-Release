<div align="center">

# RTIPS — Real-Time ID Printing System

**A kiosk-enabled, self-service identification card issuance platform for schools and enterprises.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](#-licensing)
[![Qt](https://img.shields.io/badge/Qt-6.10%2B-41CD52?logo=qt)](https://www.qt.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%2B-336791?logo=postgresql)](https://www.postgresql.org/)
[![Server v1.0.40](https://img.shields.io/badge/Server-v1.0.40-informational)](https://github.com/Aron0x0/RTIPS-Release/releases)
[![Client v1.0.40](https://img.shields.io/badge/Kiosk-v1.0.40-informational)](https://github.com/Aron0x0/RTIPS-Release/releases)
[![Ed25519 Signed](https://img.shields.io/badge/Release_Manifest-Ed25519_Signed-blue)](#-package-distribution)

</div>

---

## 📋 Table of Contents

1. [Overview](#-overview)
2. [Architecture](#-architecture)
3. [Installation](#-installation)
   - [Prerequisites](#prerequisites)
   - [Quick Install (Pre-built Packages)](#quick-install-pre-built-packages)
   - [RTIPSServer (Admin)](#rtipsserver-admin)
   - [RTIPSKiosk (Client)](#rtipskiosk-client)
4. [Setup](#-setup)
   - [Database Setup](#database-setup)
   - [First-Run Configuration](#first-run-configuration)
   - [Network Configuration](#network-configuration)
5. [From Sources](#-from-sources)
   - [Build Requirements](#build-requirements)
   - [Building RTIPSServer](#building-rtipsserver)
   - [Building RTIPSKiosk (Client)](#building-rtipskiosk-client)
6. [Package Distribution](#-package-distribution)
   - [Release Manifest](#release-manifest)
   - [Versioning Scheme](#versioning-scheme)
   - [Auto-Update Mechanism](#auto-update-mechanism)
7. [Documentation](#-documentation)
8. [Troubleshooting](#-troubleshooting)
9. [Official Companion Guide](#-official-companion-guide)
   - [CSV Bulk Import](#csv-bulk-import)
10. [Contributing](#-contributing)
11. [Reference](#-reference)
12. [Licensing](#-licensing)

---

## 🧭 Overview

RTIPS is a **two-component system** designed to modernize ID card issuance in schools and enterprise environments. It replaces manual, staff-mediated processes with a secure, auditable, and self-service kiosk workflow.

| Component | Package | Description |
|---|---|---|
| **RTIPSServer** | `RTIPSServer_*.zip` (Primary)<br>`RTIPSServer_*.rar` (Dual) | Desktop admin application — manages identity records, templates, QR tokens, print queues, audit logs, REST API server, and centralized PostgreSQL database. |
| **RTIPSKiosk** | `RTIPSKiosk_*.zip` (Primary)<br>`RTIPSKiosk_*.rar` (Dual) | Thin-client kiosk application — handles student/employee authentication, photo/signature capture, ID preview, and physical printing via secure HTTPS REST API (Port 8080). Operates with zero direct database connections. |

**Key capabilities:**
- 🪪 Self-service ID printing with one-time token enforcement (no duplicate prints)
- 📸 Live webcam photo and digital signature capture
- 🔐 QR code authentication + optional PIN verification
- 🖨️ Real-time print preview with customizable ID card templates
- 📋 Full audit trail — every print event is logged with timestamp, operator, and status
- 👥 Supports students, employees, and admin roles
- 🏫 Designed for Science City of Muñoz Senior High School — extensible to any institution

---

## 🏗️ Architecture

```
┌─────────────────────────┐     HTTPS REST API (Port 8080)       ┌─────────────────────────┐
│     RTIPSServer          │ ◄───────────────────────────────►   │     RTIPSKiosk           │
│  (Administrator App)     │     • TLS + RFC 6125 Verification   │  (Thin-Client Kiosk)    │
│                          │     • Token-Based Auth & Anti-Replay│                          │
│  • User Directory        │     • PII Minimization Response     │  • QR / PIN Login        │
│  • Template Designer     │                                      │  • Photo Capture         │
│  • QR Token Issuance     │                                      │  • Signature Capture     │
│  • Print Queue Monitor   │                                      │  • ID Preview            │
│  • Audit / Print Logs    │                                      │  • Direct Print Engine   │
│  • Central SQLite/Postgres                                      │  • Auto-Update Support   │
└─────────────────────────┘                                      └─────────────────────────┘
```

The server hosts the central database and manages administration, while kiosk terminals operate as secure thin clients communicating strictly over the authenticated HTTPS REST API. Kiosks do not connect directly to the central database, preventing credential exposure and database surface attacks.

---

## 📦 Installation

### Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| **Windows** | 10 (64-bit) | Windows 11 recommended |
| **PostgreSQL** | 16 | Required **ONLY** on the server hosting `RTIPSServer`. RTIPSKiosk requires zero PostgreSQL installation or drivers. |
| **Visual C++ Redistributable** | 2015–2022 x64 | Bundled Qt runtime requires it |
| **Printer driver** | — | Any Windows-compatible printer for ID output |
| **Webcam** (Kiosk only) | — | Required for photo/signature capture |

> [!NOTE]
> Both application packages are distributed as **zero-dependency `.zip` archives** that extract natively in Windows Explorer without requiring third-party tools. For bandwidth-constrained environments, high-compression `.rar` archives are provided alongside `.zip`. All necessary Qt runtime libraries, multimedia plugins, and network drivers are bundled inside each package.

---

### Quick Install (Pre-built Packages)

The latest verified production releases are listed on the [GitHub Releases page](https://github.com/Aron0x0/RTIPS-Release/releases).

Download the package matching your deployment role:

| Component | Package Format | Download | Size | SHA-256 |
|---|---|---|---|---|
| **RTIPSServer v1.0.40** (Admin) | **ZIP (Primary)** | [RTIPSServer_v1.0.40-build006.zip](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSServer-v1.0.40/RTIPSServer_v1.0.40-build006_2026-11-22_11-30-58.zip) | 76.10 MB | `f9eea7585f5cb5a2a4f0e85329cd2acda639c5d1c793bd126d1bd93c682c208b` |
| RTIPSServer v1.0.40 (Admin) | RAR (Dual) | [RTIPSServer_v1.0.40-build006.rar](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSServer-v1.0.40/RTIPSServer_v1.0.40-build006_2026-11-22_11-30-58.rar) | 60.14 MB | `f79ffa4ddf6a26237fe2774c1d8a4fbbcd3d34c39a09827e0d320e3314e61678` |
| **RTIPSKiosk v1.0.40** (Client) | **ZIP (Primary)** | [RTIPSKiosk_v1.0.40-build003.zip](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSKiosk-v1.0.40/RTIPSKiosk_v1.0.40-build003_2026-11-22_09-59-26.zip) | 88.61 MB | `4b72da3f557b855563336887ed3aca6306037a478cdfa587644a54a83566e1b8` |
| RTIPSKiosk v1.0.40 (Client) | RAR (Dual) | [RTIPSKiosk_v1.0.40-build003.rar](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSKiosk-v1.0.40/RTIPSKiosk_v1.0.40-build003_2026-11-22_09-59-26.rar) | 76.85 MB | `c984ae7d844815ec02430f9559d7cc2c8af20e81d4a750e4278f7da45a4723e7` |

---

### RTIPSServer (Admin)

1. Download `RTIPSServer_v1.0.40-*.zip` (or `.rar`) from the releases page.
2. Extract the archive into a permanent directory, e.g.:
   ```
   C:\RTIPS\Server\
   ```
3. Open the extracted folder. You should see:
   ```
   RTIPSAdmin.exe
   UpdateHelper.exe
   config.ini
   data\
   sqldrivers\
   *.dll   (Qt and PostgreSQL runtime libraries)
   ```
4. Ensure **PostgreSQL is running** and you have created a database for RTIPS (see [Database Setup](#database-setup)).
5. Launch `RTIPSAdmin.exe`.
6. On first run, the **Database Setup** screen appears — enter your PostgreSQL connection credentials.

> [!IMPORTANT]
> Do **not** move `RTIPSAdmin.exe` out of its extracted directory. The application requires its companion DLLs, `sqldrivers/`, and `data/` subdirectories to reside in the application directory.

---

### RTIPSKiosk (Client)

1. Download `RTIPSKiosk_v1.0.40-*.zip` (or `.rar`) from the releases page.
2. Extract the archive into a permanent directory, e.g.:
   ```
   C:\RTIPS\Kiosk\
   ```
3. The extracted folder contains:
   ```
   RTIPSKiosk.exe
   UpdateHelper.exe
   config.ini
   data\
   *.dll
   ```
4. Launch `RTIPSKiosk.exe`.
5. On first run, enter the **HTTPS Server Connection** details:
   - **Server Host / IP:** IP address or hostname of the machine running `RTIPSServer` (e.g. `192.168.1.100` or `rtips.internal`).
   - **HTTPS Port:** `8080` (default RTIPS REST API port).
   - **Kiosk Token:** Kiosk API token generated in RTIPSAdmin (`Settings → Kiosk Terminals`).

> [!IMPORTANT]
> `RTIPSKiosk` is an isolated thin client that communicates exclusively via the encrypted HTTPS REST API on Port 8080. It **never** connects directly to PostgreSQL, requires zero PostgreSQL client drivers, and requires no database credentials on the kiosk terminal.

> [!TIP]
> For a dedicated unattended kiosk, configure Windows to launch `RTIPSKiosk.exe` on startup using **Task Scheduler** or a shortcut in `shell:startup`.

---

## ⚙️ Setup

### Database Setup (Server Host Only)

RTIPS uses **PostgreSQL** (version 16, 17, or 18) as its centralized database. Database configuration is performed **strictly on the server machine running RTIPSServer**. Kiosks do not connect to the database.

#### 1. Create the database

Open a PostgreSQL administrative shell (`psql`) as a superuser and run:

```sql
CREATE DATABASE rtips;
CREATE USER rtips_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE rtips TO rtips_user;
```

#### 2. Schema initialization and automatic migration

`RTIPSServer` automatically initializes all required tables and runs schema migrations on first launch. No manual SQL script execution is required. The managed schema includes:

| Table | Purpose |
|---|---|
| `admins` | Admin and faculty login accounts with Argon2id / bcrypt credentials |
| `students` | Student identity records with granular PII controls |
| `employees` | Employee identity records |
| `addresses` | Shared student and employee residential addresses |
| `courses` / `sections` | Academic classification entities |
| `emergency_contacts` | Guardian / emergency contact data |
| `qr_tokens` | Single-use cryptographic QR authentication tokens |
| `print_requests` | Active and historical print requests |
| `id_cards` | Issued card status (`is_printed`, `printed_date`, `expiration_date`) |
| `print_logs` | Immutable audit log of every print event |
| `settings` | System-wide configuration key/value pairs |
| `id_templates` | ID card template canvas definitions |
| `schema_migrations` | Database schema version and migration journal |

#### 3. Server database connection parameters

On first launch, `RTIPSAdmin.exe` prompts with the Database Setup dialog:

| Field | Example | Notes |
|---|---|---|
| Host | `localhost` or `127.0.0.1` | Local PostgreSQL instance |
| Port | `5432` | Standard PostgreSQL port |
| Database | `rtips` | Database created in Step 1 |
| Username | `rtips_user` | Application user |
| Password | `your_secure_password` | User password |

---

### First-Run Configuration

After connecting the database, perform the initial setup in **RTIPSAdmin**:

1. **Create the first administrator account** — prompted automatically on initial startup.
2. **Configure institution info** — go to **Settings → Organization** to configure institution name and campus branding.
3. **Configure ID templates** — go to **ID Management → Templates** to customize card layouts for students and staff.
4. **Configure Kiosk Terminals** — go to **Settings → Kiosk Terminals** to register kiosk identifiers and generate API tokens.
5. **Issue QR tokens** — navigate to **User Directory → Student/Employee → Issue QR Token** to generate authentication tokens.

---

### Network & Firewall Configuration

RTIPS uses a **zero-trust, thin-client architecture**:

```
┌───────────────────────────────────────┐
│           SERVER HOST                 │
│                                       │
│   PostgreSQL Port 5432 (Localhost)    │
│            ▲                          │
│            │ (local socket / 127.0.0.1)│
│            ▼                          │
│     RTIPSServer REST Engine           │
│     (Listening on Port 8080 HTTPS)    │
└──────────────────┬────────────────────┘
                   │
                   │ Inbound HTTPS (Port 8080 only)
                   ▼
       [ Windows Firewall Rule ]
                   │
                   │ Encrypted TLS REST Traffic
                   ▼
┌───────────────────────────────────────┐
│           KIOSK TERMINAL              │
│                                       │
│       RTIPSKiosk Client               │
│   (No PostgreSQL drivers/access)      │
└───────────────────────────────────────┘
```

#### Firewall Rules (Server Machine):
1. **Allow Inbound TCP Port 8080** for HTTPS REST API:
   ```powershell
   New-NetFirewallRule -DisplayName "RTIPS Server HTTPS API" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
   ```
2. **DO NOT expose PostgreSQL Port 5432 to the local network or internet.** Keep PostgreSQL listening on `localhost` (`127.0.0.1`) only. RTIPSKiosk never requires port 5432 access.

#### Kiosk HTTPS Connection:
In `RTIPSKiosk.exe`, configure:
- **Server URL:** `https://<server-ip>:8080` (e.g. `https://192.168.1.100:8080`)
- **API Token:** Pre-shared kiosk authentication token generated in RTIPSAdmin
- **TLS Trust:** Self-signed certificates generated by RTIPSServer can be trusted by importing `server.crt` into the kiosk's trusted root store or placing it beside `RTIPSKiosk.exe`.

---

## 🔨 From Sources

### Build Requirements

| Tool | Version | Notes |
|---|---|---|
| **Qt** | 6.10 or 6.11 | Requires: `Core Qml Quick Sql Multimedia Concurrent PrintSupport Network QuickDialogs2` |
| **CMake** | 3.16+ | Bundled with Qt Creator or install separately |
| **MinGW** | 64-bit (GCC 13+) | Or MSVC 2022 — MinGW recommended for compatibility with pre-built Qt |
| **PostgreSQL dev headers** | 16–18 | `libpq.dll` must be on the build path |
| **Git** | any | For cloning; ZXing-C++ is fetched automatically via `FetchContent` |

---

### Building RTIPSServer

```powershell
# Clone the repository
git clone https://github.com/Aron0x0/RTIPS-Release.git  # or your private fork

# Open the RTIPSServer source in Qt Creator:
#   File → Open Project → select RTIPSServer/CMakeLists.txt

# Or build from the command line:
cd RTIPSServer

cmake -B build -DCMAKE_BUILD_TYPE=Release `
      -DQt6_DIR="C:/Qt/6.11.1/mingw_64/lib/cmake/Qt6" `
      -DRTIPS_POSTGRES_BIN="C:/Program Files/PostgreSQL/17/bin"

cmake --build build --config Release --parallel
```

> [!NOTE]
> **ZXing-C++** (QR code generation library) is downloaded automatically by CMake's `FetchContent` on first configure. An internet connection is required during the first build.

The built executables land in `build/bin/`:
- `RTIPSAdmin.exe` — administrator application
- `RTIPSKiosk.exe` — kiosk client (also built from this project)

---

### Building RTIPSKiosk (Client)

The kiosk client is a **separate CMake project** located in the `RTIPSClient` repository.

```powershell
cd RTIPSClient

cmake -B build -DCMAKE_BUILD_TYPE=Release `
      -DQt6_DIR="C:/Qt/6.11.1/mingw_64/lib/cmake/Qt6" `
      -DRTIPS_POSTGRES_BIN="C:/Program Files/PostgreSQL/17/bin"

cmake --build build --config Release --parallel
```

After a successful build, deploy Qt dependencies alongside the executable:

```powershell
# From the Qt installation's bin directory:
windeployqt --qmldir ui build\RTIPSKiosk.exe
```

---

## 📤 Package Distribution

### Release Manifest

Each release folder (`release/server/` and `release/client/`) contains a cryptographically signed `manifest.json` describing the artifact:

```jsonc
// release/server/manifest.json (v1.0.40 Build 006)
{
  "product":      "RTIPSServer",
  "version":      "1.0.40",
  "build":        6,
  "buildString":  "006",
  "releaseTag":   "RTIPSServer-v1.0.40",
  "fileName":     "RTIPSServer_v1.0.40-build006_2026-11-22_11-30-58.zip",
  "downloadUrl":  "https://github.com/Aron0x0/RTIPS-Release/releases/download/...",
  "sha256":       "f9eea7585f5cb5a2a4f0e85329cd2acda639c5d1c793bd126d1bd93c682c208b",
  "sizeBytes":    79794015,
  "sizeMB":       76.10,
  "sourceBranch": "feature/admin",
  "sourceCommit": "f6990ceab9b5ea2351533276a11d5fe720001a1a",
  "signature":    "dade8eb58d6cb219...5db6a81c4c236000",
  "dualPackage": {
    "fileName":    "RTIPSServer_v1.0.40-build006_2026-11-22_11-30-58.rar",
    "downloadUrl": "https://github.com/Aron0x0/RTIPS-Release/releases/download/...",
    "sha256":      "f79ffa4ddf6a26237fe2774c1d8a4fbbcd3d34c39a09827e0d320e3314e61678",
    "sizeBytes":   63060793
  }
}
```

#### Cryptographic Signature Verification
All manifests are digitally signed using Ed25519 asymmetric cryptography. You can verify manifest authenticity before deployment using `SignManifest.exe`:
```powershell
.\SignManifest.exe -v -m release\server\manifest.json
```
The signer verifies against the official production public key:
`f896be5f26a872a23a455af9ac79d51557eac628dbb7417b1a56fffec1f12c0b`

---

### Versioning Scheme

RTIPS follows **Semantic Versioning** (`MAJOR.MINOR.PATCH`) with an automated monotonic build counter:

```
v1.0.40-build002
 │ │  │       └── Build counter within the release (001, 002, ...)
 │ │  └────────── Patch — security hardening, database migrations, bug fixes
 │ └───────────── Minor — feature additions, backward-compatible API improvements
 └─────────────── Major — breaking changes or architectural redesigns
```

---

### Auto-Update Mechanism

Both applications integrate an automated updater powered by `UpdateHelper.exe`. On startup, the application queries the remote `manifest.json`. If a new version exists:

1. The user sees an **"Update Available"** banner.
2. Clicking **Update** activates `UpdateHelper.exe`, which:
   - Downloads the primary `.zip` (or `.rar`) archive.
   - Cryptographically verifies the Ed25519 signature of the manifest.
   - Validates the SHA-256 payload checksum.
   - Performs atomic file replacement and restarts the application.

---

## 📖 Documentation

| Document | Location | Description |
|---|---|---|
| Database Schema | [`RTIPSServer/CURRENT_DATABASE_SCHEMA.md`](https://github.com/Aron0x0/RTIPS-Release) | Full table definitions, column types, and ERD |
| Project Goal & Scope | `RTIPSServer/PROJECT_GOAL.txt` | Research problem, objectives, and system scope |
| Release Manifests | `release/server/manifest.json`, `release/client/manifest.json` | Artifact metadata for the latest builds |
| Source Architecture | See [From Sources](#-from-sources) | CMake project layout, module descriptions |

### Module Overview

#### RTIPSServer (Admin) — Source Modules

| Path | Module |
|---|---|
| `backend/services/KioskService.*` | Core business logic — student/employee CRUD, print management, QR tokens |
| `backend/data/DatabaseManager.*` | PostgreSQL / SQLite connection pool, schema migrations, and backup/restore |
| `backend/services/HttpApiServer.*` | Thread-pool HTTPS REST API server for kiosk client requests |
| `backend/services/SessionManager.*` | Admin token lifecycle and session validation |
| `ui/pages/UserDirectory.qml` | Add/edit students and employees |
| `ui/pages/IDManagement.qml` | ID template designer and assignment |
| `ui/pages/PrintQueue.qml` | Monitor active print jobs |
| `tools/` | Standalone release signer (`SignManifest`) and import utilities |

#### RTIPSKiosk (Client) — Source Modules

| Path | Module |
|---|---|
| `backend/services/KioskService.*` | Thin-client REST adapter for server HTTPS API |
| `backend/printing/CardPrintService.*`| Direct card printer hardware interface |
| `backend/qr/QrScanner.*` | Webcam ZXing QR code detection engine |
| `ui/pages/ClientLogin.qml` | QR scan and PIN authentication UI |
| `ui/pages/ClientInformationPreview.qml`| Real-time ID card preview |
| `UpdateHelper/` | Companion auto-updater process |

---

## 🔧 Troubleshooting

### RTIPSServer: Missing PostgreSQL DLLs

**Symptom:** `The program can't start because libpq.dll was not found.`

**Fix:** Ensure PostgreSQL `bin/` directory is in the system PATH, or copy runtime libraries into the same directory as `RTIPSAdmin.exe`:
```
libpq.dll, libssl-*.dll, libcrypto-*.dll
```

---

### RTIPSServer: Cannot connect to PostgreSQL

**Symptom:** "Connection refused" or "could not connect to server" during Database Setup.

**Checklist:**
- [ ] PostgreSQL service is running (`services.msc` → `postgresql-x64-*` → **Running**)
- [ ] Host is `127.0.0.1` or `localhost` and Port is `5432`
- [ ] Target database `rtips` exists (`psql -U postgres -c "CREATE DATABASE rtips;"`)
- [ ] User credentials match configured values in `config.ini`

---

### RTIPSKiosk: Cannot connect to RTIPSServer

**Symptom:** Kiosk displays "Server Unreachable" or "TLS Handshake Failed".

**Checklist:**
- [ ] `RTIPSAdmin.exe` is running on the host machine and the REST API server is active
- [ ] Server machine's Windows Firewall allows inbound TCP traffic on **Port 8080**
- [ ] Kiosk is configured with `https://<server-ip>:8080` (ensure `https://` is included)
- [ ] Kiosk API token entered in the kiosk setup matches the token generated in RTIPSAdmin (`Settings → Kiosk Terminals`)
- [ ] If using self-signed TLS certificates, place `server.crt` in the `RTIPSKiosk.exe` application directory or install into Windows Trusted Root Certification Authorities

---

### Kiosk does not print — ID already marked as printed

**Symptom:** The print button is grayed out with "Already Printed" shown.

**Explanation:** RTIPS enforces **one-time print** — once an ID has been printed, the record is locked.

**Fix (Admin):** In RTIPSAdmin, go to **User Directory → [Student/Employee] → Actions → Authorize Reprint**. This resets `is_printed = 0` and clears the `id_cards` record, allowing the kiosk to print again.

---

### QR token scan fails

**Symptom:** "Invalid or expired token" error on the kiosk login screen.

**Checklist:**
- [ ] The QR token was issued in RTIPSAdmin for this user (User Directory → Issue QR Token)
- [ ] The token has not already been consumed (tokens are single-use)
- [ ] The camera has sufficient lighting and is focused on the QR code
- [ ] The system clocks on the server and kiosk machine are synchronized (token expiry is time-based)

---

### Photo/signature capture is blank or fails

**Symptom:** Black preview or "No camera found" error.

**Fix:**
- Ensure the webcam is plugged in and recognized by Windows (Device Manager → Imaging Devices)
- Check that no other application (e.g., a video call app) is currently holding the camera
- If using a USB webcam, try a different USB port

---

### Build fails — Qt module not found

**Symptom:** `Could not find a package configuration file provided by "Qt6"` during CMake configure.

**Fix:** Pass the correct Qt CMake directory:
```powershell
cmake -B build -DQt6_DIR="C:/Qt/6.11.1/mingw_64/lib/cmake/Qt6" ...
```

---

### ZXing-C++ download fails during build

**Symptom:** `FetchContent` error during CMake configure — unable to download ZXing.

**Fix:** Check your internet connection. If behind a proxy, set:
```powershell
$env:HTTPS_PROXY = "http://your.proxy:8080"
cmake -B build ...
```
Alternatively, download the [ZXing-C++ v2.2.1 zip](https://github.com/zxing-cpp/zxing-cpp/archive/refs/tags/v2.2.1.zip) manually and set `FETCHCONTENT_SOURCE_DIR_ZXING-CPP` to its extracted path.

---

## 📚 Official Companion Guide

This section walks through the complete day-to-day workflow for both administrators and kiosk users.

### For Administrators — Daily Workflow

```
1. Launch RTIPSAdmin.exe
2. Log in with your admin credentials
3. (Optional) Add new students or employees via User Directory → Add Student / Add Employee
4. Issue QR tokens: User Directory → select user → Issue QR Token → hand the printed/digital token to the user
5. Monitor the print queue: Print Queue → view pending, completed, and failed jobs
6. Review audit logs: Print Logs → filter by date, user, or status
```

### For Kiosk Users — ID Printing Workflow

```
1. Approach the kiosk screen
2. Hold your QR token in front of the camera, OR enter your ID number + PIN
3. Review the ID preview — confirm your photo, name, and details are correct
4. If your photo is missing: tap "Capture Photo" and follow the on-screen instructions
5. Tap "Print ID" — the ID card will be printed on the connected printer
6. Collect your printed ID card
```

### ID Template Management

1. In RTIPSAdmin, go to **ID Management**.
2. Select the role: **Student**, **Employee**, or **Admin**.
3. Click **New Template** to create a custom layout, or select an existing template.
4. Use **Set as Default** to make a template the institution-wide active template for that role.
5. Use **Assign** to override the default for a specific course, section, or department.

### QR Token Lifecycle

```
[Issued] ──► [Active] ──► [Consumed / Used] ──► (cannot be reused)
                │
                └──► [Revoked] (manually by admin)
```

Tokens are one-time use. Once scanned and authenticated at the kiosk, they are marked **consumed** in the database.

### CSV Bulk Import

RTIPS supports high-volume batch import of students, employees, and unified directories via standard CSV format.

#### How to Import:
1. In **RTIPSAdmin**, navigate to **User Directory**.
2. Click **Import CSV** in the action bar.
3. Select your prepared CSV file (`.csv`) and confirm.
4. The system validates required headers, formats dates (`YYYY-MM-DD` or flexible dates), hashes PINs, assigns QR tokens, and inserts records into PostgreSQL inside safe transaction savepoints.
5. A summary dialog displays the count of imported records along with detailed row error messages if any row fails validation.

---

#### 1. Student CSV Specification (`students_test.csv`)

Used for bulk importing student records.

##### Header Definition:
```csv
person_type,student_id_number,last_name,first_name,middle_name,birthday,gender,year_level,course_code,course_name,section,address_province,address_city,address_barangay,address_other_info,student_pin,student_qr_token,photo_path,signature_path,is_active,is_printed
```

##### Field Breakdown:
| Column | Required | Type / Format | Description & Example |
|---|---|---|---|
| `person_type` | Optional | String | Set to `student` |
| `student_id_number` | **Yes** | Alphanumeric | Unique student ID / LRN (e.g. `202`, `100-00-0001`) |
| `last_name` | **Yes** | Text | Student family name (e.g. `Reyes`) |
| `first_name` | **Yes** | Text | Student given name (e.g. `Paolo`) |
| `middle_name` | No | Text | Middle name (e.g. `Ramos`) |
| `birthday` | No | `YYYY-MM-DD` | Birth date (e.g. `2006-05-08`) |
| `gender` | No | Text | `Male` or `Female` |
| `year_level` | No | Integer | Academic year level (`1` to `12` or `Kinder`) |
| `course_code` | No | Text | Academic program code (e.g. `BSIT`, `BSHM`, `BSED`) |
| `course_name` | No | Text | Full course title (e.g. `Bachelor of Science in Information Technology`) |
| `section` | No | Text | Section letter or code (e.g. `A`, `B`) |
| `address_province` | No | Text | Province name (e.g. `Bulacan`) |
| `address_city` | No | Text | City or municipality (e.g. `Meycauayan City`) |
| `address_barangay` | No | Text | Barangay / district (e.g. `Liang`) |
| `address_other_info` | No | Text | Street, block, lot, purok details (e.g. `Purok 3`) |
| `student_pin` | No | 4-6 Digits | PIN used for self-service kiosk login (e.g. `9595`, default: `0000`) |
| `student_qr_token` | No | Text | Optional pre-assigned QR token (auto-generated if empty) |
| `photo_path` | No | Path / Filename | Optional initial photo file path |
| `signature_path` | No | Path / Filename | Optional initial digital signature file path |
| `is_active` | No | `0` or `1` | `1` = Active account (default), `0` = Inactive / Disabled |
| `is_printed` | No | `0` or `1` | `0` = Pending / Not printed (default), `1` = Already printed |

##### Example Rows:
```csv
person_type,student_id_number,last_name,first_name,middle_name,birthday,gender,year_level,course_code,course_name,section,address_province,address_city,address_barangay,address_other_info,student_pin,student_qr_token,photo_path,signature_path,is_active,is_printed
student,202,Reyes,Paolo,Ramos,2006-05-08,Male,1,BSHM,Bachelor of Science in Hospitality Management,B,Bulacan,Meycauayan City,Liang,,9595,,,,1,0
student,203,Garcia,Mary Joy,Dela,2005-08-16,Female,2,BSED,Bachelor of Secondary Education,A,Bulacan,Baliuag,San Agustin,Purok 3,9042,,,,1,0
student,204,Villanueva,Grace,Dela,2008-04-04,Female,3,BSCRIM,Bachelor of Science in Criminology,A,Bulacan,San Jose Del Monte,Liang,,1878,,,,1,1
```

---

#### 2. Employee CSV Specification (`employees_test.csv`)

Used for bulk importing faculty, administrative, and operations personnel.

##### Header Definition:
```csv
person_type,employee_id_number,last_name,first_name,middle_name,birthday,gender,department,position,address_province,address_city,address_barangay,address_other_info,employee_pin,photo_path,signature_path,employee_qr_token,tin,gsis,pagibig,philhealth,prclicense,blood_type,is_active,is_printed
```

##### Field Breakdown:
| Column | Required | Type / Format | Description & Example |
|---|---|---|---|
| `person_type` | Optional | String | Set to `employee` |
| `employee_id_number` | **Yes** | Alphanumeric | Unique Employee ID (e.g. `01-001`, `EMP-2026`) |
| `last_name` | **Yes** | Text | Employee family name (e.g. `Cruz`) |
| `first_name` | **Yes** | Text | Employee given name (e.g. `Eduardo`) |
| `middle_name` | No | Text | Middle name (e.g. `Garcia`) |
| `birthday` | No | `YYYY-MM-DD` | Birth date (e.g. `1999-04-08`) |
| `gender` | No | Text | `Male` or `Female` |
| `department` | No | Text | Department or unit (e.g. `Human Resources`, `IT Department`) |
| `position` | No | Text | Job title / designation (e.g. `Supervisor`, `IT Officer`) |
| `address_province` | No | Text | Province name (e.g. `Bulacan`) |
| `address_city` | No | Text | City or municipality (e.g. `Angeles City`) |
| `address_barangay` | No | Text | Barangay / district (e.g. `Namayan`) |
| `address_other_info` | No | Text | Street, block, subdivision info (e.g. `Blk 3 Lot 20`) |
| `employee_pin` | No | 4-6 Digits | PIN used for self-service kiosk login (e.g. `1234`, default: `0000`) |
| `photo_path` | No | Path / Filename | Optional initial photo file path |
| `signature_path` | No | Path / Filename | Optional initial signature file path |
| `employee_qr_token` | No | Text | Optional pre-assigned QR token (auto-generated if empty) |
| `tin` | No | Text | Taxpayer Identification Number (e.g. `617-716-127`) |
| `gsis` | No | Text | GSIS BP / ID Number (e.g. `81 4257-9928`) |
| `pagibig` | No | Text | Pag-IBIG MID Number (e.g. `7873-4611-8359`) |
| `philhealth` | No | Text | PhilHealth Identification Number (e.g. `85-398704996-1`) |
| `prclicense` | No | Text | Professional Regulation Commission license (e.g. `3678638-99`) |
| `blood_type` | No | Text | Blood type (`A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`) |
| `is_active` | No | `0` or `1` | `1` = Active employee (default), `0` = Inactive |
| `is_printed` | No | `0` or `1` | `0` = Pending / Not printed (default), `1` = Already printed |

##### Example Rows:
```csv
person_type,employee_id_number,last_name,first_name,middle_name,birthday,gender,department,position,address_province,address_city,address_barangay,address_other_info,employee_pin,photo_path,signature_path,employee_qr_token,tin,gsis,pagibig,philhealth,prclicense,blood_type,is_active,is_printed
employee,01-001,Cruz,Eduardo,Garcia,1999-04-08,Male,Human Resources,Supervisor,Bulacan,Angeles City,Namayan,,,1234,,,,617-716-127,81 4257-9928,7873-4611-8359,85-398704996-1,3678638-99,O+,1,0
employee,01-002,Ramos,Carmela,Garcia,2002-12-15,Female,Library Services,Staff,Bulacan,San Jose Del Monte,Longos,,,2345,,,,649-227-487,20 5803-6925,4150-2139-1750,94-344703907-5,4905582-22,O+,1,1
employee,01-003,Gonzales,Bianca,Reyes,1994-08-13,Female,Library Services,Accountant,Bulacan,Angeles City,San Agustin,Blk 3 Lot 20,3456,,,,376-755-804,81 4598-6313,1916-4752-1525,50-530747414-5,,AB-,1,0
```

---

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

### Getting Started

1. **Fork** the appropriate repository (RTIPSServer or RTIPSClient).
2. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Make your changes. Follow the existing code style:
   - **C++**: C++20, Qt-style naming (`camelCase` for variables, `PascalCase` for types)
   - **QML**: Component files in `PascalCase`, property names in `camelCase`
   - Keep business logic in `backend/services/` — keep UI logic in `.qml` files
4. Test your changes against a real PostgreSQL instance.
5. Commit with a descriptive message:
   ```bash
   git commit -m "feat: add reprint authorization for employees"
   ```
6. Open a **Pull Request** against `main`.

### Reporting Bugs

Open an issue with:
- Operating system and version
- RTIPS version (found in the title bar or `manifest.json`)
- PostgreSQL version
- Exact error message or screenshot
- Steps to reproduce

### Code Organization Guidelines

| Rule | Detail |
|---|---|
| No raw SQL in QML | All database calls go through `KioskService` or `DatabaseManager` in `backend/` |
| Validate in C++, display in QML | Business rules live in C++; QML only renders and emits signals |
| One transaction per logical operation | Wrap multi-step DB operations in `BEGIN`/`COMMIT` |
| Preserve comments | Do not delete existing docstrings or explanatory comments |

---

## 📐 Reference

### Supported Roles

| Role | Login Method | Can Print |
|---|---|---|
| `student` | QR token or ID + PIN | ✅ (own ID only, once) |
| `employee` | QR token or ID + PIN | ✅ (own ID only, once) |
| `admin` | Username + password (RTIPSAdmin) | ✅ (via admin override) |

### `id_cards` Table — Key Columns

| Column | Type | Description |
|---|---|---|
| `owner_type` | `TEXT` | `'student'` or `'employee'` |
| `owner_id` | `BIGINT` | FK to `students.student_id` or `employees.employee_id` |
| `is_printed` | `SMALLINT` | `0` = not printed, `1` = printed |
| `printed_date` | `DATE` | Date the ID was physically printed |
| `expiration_date` | `DATE` | Computed as `printed_date + validity_years` |
| `status` | `TEXT` | `'pending'`, `'completed'`, `'failed'` |

### Environment Variables & Config

| Setting | Location | Default | Description |
|---|---|---|---|
| `RTIPS_POSTGRES_BIN` | CMake cache | auto-detected | Path to PostgreSQL `bin/` dir containing `libpq.dll` |
| `student_pin_required` | `settings` table | `1` | Whether kiosk login requires PIN in addition to ID |

### Key File Paths (Deployed)

```
RTIPSAdmin.exe          ← Admin application
RTIPSKiosk.exe          ← Kiosk application
UpdateHelper.exe        ← Auto-update helper (kiosk only)
data/
  templates/            ← ID card template files
  overlays.json         ← Template overlay configuration
sqldrivers/
  qsqlpsql.dll          ← Qt PostgreSQL driver
```

### External Libraries

| Library | Version | License | Purpose |
|---|---|---|---|
| [Qt](https://www.qt.io/) | 6.10+ | LGPL 3.0 | UI framework, database, printing, multimedia |
| [ZXing-C++](https://github.com/zxing-cpp/zxing-cpp) | v2.2.1 | Apache 2.0 | QR code generation |
| [PostgreSQL libpq](https://www.postgresql.org/) | 16–18 | PostgreSQL License | Database client library |

---

## ⚖️ Licensing

RTIPS is released under the **MIT License**.

```
MIT License

Copyright (c) 2026 RTIPS Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Third-Party Licenses

- **Qt** is used under the [GNU LGPL v3](https://www.qt.io/licensing/). The application dynamically links to Qt and does not modify it.
- **ZXing-C++** is used under the [Apache License 2.0](https://github.com/zxing-cpp/zxing-cpp/blob/master/LICENSE).
- **PostgreSQL / libpq** is used under the [PostgreSQL License](https://www.postgresql.org/about/licence/).

---

<div align="center">

Made For Academic & Research Evaluation Only • © 2026

[🐛 Report a Bug](https://github.com/Aron0x0/RTIPS-Release/issues) · [💡 Request a Feature](https://github.com/Aron0x0/RTIPS-Release/issues) · [📦 Latest Release](https://github.com/Aron0x0/RTIPS-Release/releases)

</div>