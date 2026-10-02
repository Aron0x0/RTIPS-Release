<div align="center">

# RTIPS — Real-Time ID Printing System

**A kiosk-enabled, self-service identification card issuance platform for schools and enterprises.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](#-licensing)
[![Qt](https://img.shields.io/badge/Qt-6.10%2B-41CD52?logo=qt)](https://www.qt.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%2B-336791?logo=postgresql)](https://www.postgresql.org/)
[![Server v1.0.28](https://img.shields.io/badge/Server-v1.0.28-informational)](https://github.com/Aron0x0/RTIPS-Release/releases)
[![Client v1.0.28](https://img.shields.io/badge/Kiosk-v1.0.28-informational)](https://github.com/Aron0x0/RTIPS-Release/releases)

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
10. [Contributing](#-contributing)
11. [Reference](#-reference)
12. [Licensing](#-licensing)

---

## 🧭 Overview

RTIPS is a **two-component system** designed to modernize ID card issuance in schools and enterprise environments. It replaces manual, staff-mediated processes with a secure, auditable, and self-service kiosk workflow.

| Component | Package | Description |
|---|---|---|
| **RTIPSServer** | `RTIPSServer_*.rar` | Desktop admin application — manages records, templates, QR tokens, print queues, audit logs, and system settings. |
| **RTIPSKiosk** | `RTIPSKiosk_*.rar` | Kiosk client application — handles student/employee authentication, photo/signature capture, ID preview, and physical printing. |

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
┌─────────────────────────┐        PostgreSQL (shared DB)        ┌─────────────────────────┐
│     RTIPSServer          │ ◄───────────────────────────────►   │     RTIPSKiosk           │
│  (Administrator App)     │                                      │  (Self-Service Kiosk)    │
│                          │                                      │                          │
│  • User Directory        │                                      │  • QR / PIN Login        │
│  • Template Designer     │                                      │  • Photo Capture         │
│  • QR Token Issuance     │                                      │  • Signature Capture     │
│  • Print Queue Monitor   │                                      │  • ID Preview            │
│  • Audit / Print Logs    │                                      │  • Print & Mark Printed  │
│  • Settings & Policies   │                                      │  • Auto-Update Support   │
└─────────────────────────┘                                      └─────────────────────────┘
```

Both applications connect to a **shared PostgreSQL database**. The server runs on an administrator workstation; the kiosk runs on a dedicated self-service machine.

---

## 📦 Installation

### Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| **Windows** | 10 (64-bit) | Windows 11 recommended |
| **PostgreSQL** | 16 | 17 or 18 also supported; must be running before launch |
| **Visual C++ Redistributable** | 2015–2022 x64 | Bundled Qt runtime requires it |
| **Printer driver** | — | Any Windows-compatible printer for ID output |
| **Webcam** (Kiosk only) | — | Required for photo/signature capture |

> [!NOTE]
> Both packages are **self-contained** — Qt libraries and SQL drivers are bundled inside the `.rar` archive. No separate Qt installation is needed to run the pre-built packages.

---

### Quick Install (Pre-built Packages)

The latest releases are listed on the [GitHub Releases page](https://github.com/Aron0x0/RTIPS-Release/releases).

Download the correct package for your machine:

| Package | Download | Size | SHA-256 |
|---|---|---|---|
| RTIPSServer v1.0.28 | [RTIPSServer_vv1.0.28-build001_2026-10-02.rar](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSServer-vv1.0.28/RTIPSServer_vv1.0.28-build001_2026-10-02_23-12-36.rar) | 55.31 MB | `3c28c742...` |
| RTIPSKiosk v1.0.28 | [RTIPSKiosk_v1.0.28-build001_2026-09-24.rar](https://github.com/Aron0x0/RTIPS-Release/releases/download/RTIPSKiosk-v1.0.28/RTIPSKiosk_v1.0.28-build001_2026-09-24_19-01-15.rar) | 89.83 MB | `ab075484...` |

---

### RTIPSServer (Admin)

1. Download `RTIPSServer_*.rar` from the releases page.
2. Extract the archive to a permanent location, e.g.:
   ```
   C:\RTIPS\Server\
   ```
3. Open the extracted folder. You should see:
   ```
   RTIPSAdmin.exe
   data\
   sqldrivers\
   *.dll   (Qt and PostgreSQL runtime libraries)
   ```
4. Make sure **PostgreSQL is running** and you have created a database for RTIPS (see [Database Setup](#database-setup)).
5. Launch `RTIPSAdmin.exe`.
6. On first run, the **Database Setup** screen will appear — enter your PostgreSQL connection details.

> [!IMPORTANT]
> Do **not** move `RTIPSAdmin.exe` out of its extracted folder. The application expects its DLLs and `data/` subdirectory to be in the same directory.

---

### RTIPSKiosk (Client)

1. Download `RTIPSKiosk_*.rar` from the releases page.
2. Extract to a permanent location, e.g.:
   ```
   C:\RTIPS\Kiosk\
   ```
3. The extracted folder contains:
   ```
   RTIPSKiosk.exe
   UpdateHelper.exe
   data\
   sqldrivers\
   *.dll
   ```
4. Launch `RTIPSKiosk.exe`.
5. On first run, enter the same PostgreSQL connection details as configured in RTIPSServer.

> [!TIP]
> For a dedicated kiosk machine, configure Windows to auto-launch `RTIPSKiosk.exe` at startup via **Task Scheduler** or placing a shortcut in the `Shell:Startup` folder.

---

## ⚙️ Setup

### Database Setup

RTIPS uses **PostgreSQL** as its centralized database. Both applications must connect to the same PostgreSQL instance.

#### 1. Create the database

Open a PostgreSQL shell (`psql`) as a superuser and run:

```sql
CREATE DATABASE rtips;
CREATE USER rtips_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE rtips TO rtips_user;
```

#### 2. Schema initialization

RTIPS automatically creates all required tables on first run. No manual SQL migration is needed. The auto-created schema includes:

| Table | Purpose |
|---|---|
| `admins` | Admin and faculty login accounts |
| `students` | Student identity records |
| `employees` | Employee identity records |
| `addresses` | Shared address records |
| `courses` / `sections` | Academic classification |
| `emergency_contacts` | Guardian / emergency contact data |
| `qr_tokens` | One-time QR authentication tokens |
| `print_requests` | Print job requests and status |
| `id_cards` | Issued card records (`is_printed`, `printed_date`, `expiration_date`) |
| `print_logs` | Audit log of every print event |
| `settings` | System-wide configuration key/values |
| `id_templates` | Custom ID card template definitions |

#### 3. Connection string reference

Both apps present a GUI setup screen on first launch. Typical values:

| Field | Example |
|---|---|
| Host | `localhost` or IP of your PostgreSQL server |
| Port | `5432` |
| Database | `rtips` |
| Username | `rtips_user` |
| Password | `your_secure_password` |

---

### First-Run Configuration

After connecting to the database, complete the following in **RTIPSAdmin**:

1. **Create the first admin account** — prompted automatically on first launch.
2. **Configure school/organization info** — go to **Settings → Organization**.
3. **Set up ID card templates** — go to **ID Management → Templates** and design or import a template for students, employees, and admins.
4. **Issue QR tokens** — go to **User Directory → [Student/Employee] → Issue QR Token** to generate a one-time login token for each user.
5. **Test a kiosk login** — launch `RTIPSKiosk.exe`, scan a QR token, and verify the ID preview and print flow.

---

### Network Configuration

If the kiosk machine is on a **separate device** from the PostgreSQL server:

1. Allow inbound connections on PostgreSQL's port (default: `5432`) in **Windows Firewall**.
2. Edit `pg_hba.conf` on the PostgreSQL server to allow the kiosk machine's IP:
   ```
   host    rtips    rtips_user    192.168.1.0/24    scram-sha-256
   ```
3. In `postgresql.conf`, set:
   ```
   listen_addresses = '*'
   ```
4. Restart the PostgreSQL service.

> [!WARNING]
> Never expose the PostgreSQL port directly to the public internet. Use a private LAN or VPN between the server and kiosk machines.

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

Each release folder (`release/server/` and `release/client/`) contains a `manifest.json` describing the artifact:

```jsonc
// release/server/manifest.json
{
  "product":      "RTIPSServer",
  "version":      "v1.0.28",
  "build":        1,
  "buildString":  "001",
  "releaseTag":   "RTIPSServer-vv1.0.28",
  "fileName":     "RTIPSServer_vv1.0.28-build001_2026-10-02_23-12-36.rar",
  "downloadUrl":  "https://github.com/Aron0x0/RTIPS-Release/releases/download/...",
  "sha256":       "3c28c74220bfb335af60ed3b8d14f949a31deaa2fec823e716da79faa5f4a3eb",
  "sizeBytes":    57999077,
  "sizeMB":       55.31,
  "publishedAt":  "2026-10-02T23:12:49+08:00"
}
```

The manifest is consumed by the **auto-update mechanism** in both applications to detect and download new releases.

---

### Versioning Scheme

RTIPS follows **Semantic Versioning** (`MAJOR.MINOR.PATCH`) with an additional build counter:

```
v1.0.28-build001
 │ │  │       └── Build number within the same version (001, 002, ...)
 │ │  └────────── Patch — bug fixes and small improvements
 │ └───────────── Minor — new features, backward-compatible
 └─────────────── Major — breaking changes or major redesigns
```

---

### Auto-Update Mechanism

RTIPSKiosk ships an `UpdateHelper.exe` companion process. On startup, the kiosk checks the `manifest.json` hosted in this repository for a newer version. If found:

1. The user sees an **"Update Available"** notification banner.
2. Clicking **Update** invokes `UpdateHelper.exe`, which:
   - Downloads the new `.rar` package.
   - Verifies the SHA-256 checksum.
   - Extracts and replaces the old files.
   - Restarts the application.

To host your own update endpoint, fork this repository and update the `downloadUrl` field in `manifest.json` to point to your server.

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
| `backend/data/DatabaseManager.*` | Schema creation, migration, and connection pool |
| `ui/pages/UserDirectory.qml` | Add/edit students and employees |
| `ui/pages/IDManagement.qml` | ID template designer and assignment |
| `ui/pages/PrintQueue.qml` | Monitor active print jobs |
| `ui/components/InputField.qml` | Reusable validated input component |
| `tools/` | Import utilities (CSV batch import) |

#### RTIPSKiosk (Client) — Source Modules

| Path | Module |
|---|---|
| `backend/services/KioskService.*` | Authentication, print marking, `id_cards` upsert |
| `ui/pages/ClientLogin.qml` | QR scan + PIN login screen |
| `ui/pages/ClientInformationPreview.qml` | ID card preview and print trigger |
| `ui/pages/MediaCapturePhoto.qml` | Live webcam photo capture |
| `ui/pages/MediaCaptureSignature.qml` | Digital signature pad |
| `ui/pages/DatabaseSetup.qml` | First-run database configuration |
| `IdData/` | Shared ID card data model library |
| `UpdateHelper/` | Auto-update extraction helper |

---

## 🔧 Troubleshooting

### Application won't start — missing DLL

**Symptom:** `The program can't start because libpq.dll is missing`

**Fix:** Copy the PostgreSQL runtime DLLs from your PostgreSQL installation's `bin/` directory into the same folder as `RTIPSAdmin.exe` / `RTIPSKiosk.exe`:
```
libpq.dll
libssl-*.dll
libcrypto-*.dll
```
These are located at `C:\Program Files\PostgreSQL\<version>\bin\`.

---

### Cannot connect to database

**Symptom:** "Connection refused" or "could not connect to server" on the setup screen.

**Checklist:**
- [ ] PostgreSQL service is running (`services.msc` → `postgresql-x64-*` → **Started**)
- [ ] Host and port are correct (`localhost` / `5432` by default)
- [ ] The database `rtips` exists (`psql -U postgres -l`)
- [ ] The user has `CONNECT` privileges
- [ ] `pg_hba.conf` allows the connecting IP
- [ ] Windows Firewall allows port `5432` (for cross-machine setups)

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

To import many students or employees at once:
1. Prepare a CSV file matching the import template (see `RTIPSServer/student.csv` or `employee.csv` for the column format).
2. In RTIPSAdmin, go to **User Directory → Import CSV**.
3. Select your CSV file and confirm the import.
4. Review the import results — any rows with errors are shown with the specific validation failure.

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

Made with ❤️ for Science City of Muñoz Senior High School and beyond.

[🐛 Report a Bug](https://github.com/Aron0x0/RTIPS-Release/issues) · [💡 Request a Feature](https://github.com/Aron0x0/RTIPS-Release/issues) · [📦 Latest Release](https://github.com/Aron0x0/RTIPS-Release/releases)

</div>