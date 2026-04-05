# Archivist

**Archivist** is a security-hardened, self-hosted, lightweight web service designed to act as a centralized, highly secure vault for infrastructure data. It manages information and credentials across multiple VPS providers, local home lab environments, various database types, and Docker-based applications.

### Core Responsibilities
* **Secure Credential Vault:** Acts as the central point to securely encrypt and store credentials required to access VPS providers and other infrastructure.
* **Proactive Backup Notifications:** Monitors database configurations using cron schedules and dispatches automated Discord notifications when a database requires a backup.
* **Centralized Asset Inventory:** Maintains a structured registry of all infrastructure components, making it easier to track IPs, locations, and service states across scattered environments.

---

## Deployment Strategy

The service is designed to run locally as an isolated Docker container on a home server (e.g., an x86_64 mini PC running a lightweight OS like DietPi).

* **Network Isolation:** The service will explicitly *not* be exposed to the public internet. Due to the highly sensitive nature of the stored credentials, it will only be accessible within the local network.
* **Docker Configuration:** The container setup must be lean, production-ready, easily backup-able (especially the SQLite volume), and secure by default, operating with the least privilege necessary.

---

## Technical Stack & Engineering Standards

### Core Technologies
* **Runtime:** Bun
* **Language:** TypeScript
* **Framework:** Next.js
* **Database:** SQLite (ideal for a lightweight, single-node container deployment)

### Quality Assurance & CI/CD
* **Testing:** Every core feature and critical path must have comprehensive unit tests to ensure high coverage.
* **Pre-commit Hooks:** A CI script must run before every `git commit`, enforcing code coverage thresholds and running the linter to maintain code quality.
* **Environment Parity:** The system must function flawlessly in development mode, while strictly enforcing real domain validation and production-grade constraints only in the production environment.

### Security & Configuration
* **Dynamic Security Locks:** Security restrictions (like strict CORS, rate limiting, and OTP timeouts) must be configurable, allowing less restrictive settings during local development to facilitate testing. All configurations must be thoroughly documented.
* **HTTP Headers:** Strict enforcement of security headers, including Content Security Policy (CSP), X-Frame-Options, and Strict-Transport-Security (in production).
* **Documentation:** The `README.md` must be treated as a living document, constantly updated with crucial configuration aspects, deployment instructions, and environment variables.

---

## Features & Use Cases

### 1. Authentication System (OTP)
Archivist is protected behind strict authentication using a One-Time Password (OTP) logic. There are no static passwords.
* **Production Mode:** A time-sensitive, short-lived code is generated and securely dispatched via Discord webhook to the authorized user.
* **Development Mode:** For ease of testing, the OTP code is bypassed from Discord and printed directly to the local application logs.
* **Expiration:** Codes have a strict Time-to-Live (TTL). Once the timeframe expires, the code is definitively invalidated.

### 2. Infrastructure Management (CRUD)
The system provides full Create, Read, Update, and Delete operations for managing infrastructure providers.
* **Provider Data:** Users can store granular configuration details about each provider, including Name, IP Address, Physical/Cloud Location, and Tags.
* **Credential Handling:** Any provider credentials (SSH keys, API tokens) are encrypted at rest within the SQLite database.

### 3. Database Backup Management
The system tracks databases strictly for backup scheduling and inventory purposes.
* **Database Registration:** Users can register databases by storing metadata such as the database **Name** and **Host Location** (e.g., a specific VPS or local server). *Note: To minimize risk, database access credentials are explicitly NOT stored in this system.*
* **Crontab Scheduling:** Each database entry includes a configurable backup schedule using standard cron syntax (e.g., `0 2 * * *` for daily at 2 AM).
* **Automated Discord Notifications:** The system monitors the configured cron schedules. When the specific time triggers, Archivist automatically dispatches an alert via a Discord webhook, notifying the administrator that a backup must be performed.