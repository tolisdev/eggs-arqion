# Pelican / Pterodactyl Eggs for Nextcloud & Paperless-ngx

This folder contains a complete set of Pelican/Pterodactyl compatible service eggs to run **Nextcloud** and **Paperless-ngx** alongside their dependency databases in an isolated panel-managed container setup.

## Contained Eggs

1.  **[Nextcloud App Egg](file:///home/apostolosk/Desktop/eggs/egg-nextcloud.json)** - Nextcloud application running on Apache (supporting SQLite, MariaDB, or PostgreSQL).
2.  **[Paperless-ngx App Egg](file:///home/apostolosk/Desktop/eggs/egg-paperless-ngx.json)** - Paperless-ngx document management application (supporting SQLite, PostgreSQL, or MariaDB; requires Redis).
3.  **[MariaDB Database Egg](file:///home/apostolosk/Desktop/eggs/egg-mariadb.json)** - Relational SQL Database, highly recommended for Nextcloud.
4.  **[PostgreSQL Database Egg](file:///home/apostolosk/Desktop/eggs/egg-postgres.json)** - Highly performant SQL Database, highly recommended for Paperless-ngx.
5.  **[Redis Egg](file:///home/apostolosk/Desktop/eggs/egg-redis.json)** - In-memory key-value database, required by Paperless-ngx as a task broker.

---

## Deployment Architecture

Below is the recommended deployment layout:

```mermaid
graph TD
    subgraph Client Space
        Browser[User Web Browser]
    end

    subgraph Pelican/Pterodactyl Daemon (Wings)
        NC[Nextcloud Server Container]
        PL[Paperless-ngx Container]
        MDB[(MariaDB Container)]
        PGDB[(PostgreSQL Container)]
        R[(Redis Container)]
    end

    Browser -->|Port: Allocated| NC
    Browser -->|Port: Allocated| PL
    NC -->|Port: Allocated| MDB
    PL -->|Port: Allocated| R
    PL -->|Port: Allocated| PGDB
```

---

## How to Import and Setup

### Step 1: Import Eggs into Pelican / Pterodactyl
1.  Navigate to your panel's **Admin Panel**.
2.  Go to the **Nests** section.
3.  Choose or create a Nest (e.g. named "Applications" or "Self-Hosted").
4.  Click **Import Egg** (typically a green button) and import the JSON files in this order:
    *   [egg-redis.json](file:///home/apostolosk/Desktop/eggs/egg-redis.json)
    *   [egg-mariadb.json](file:///home/apostolosk/Desktop/eggs/egg-mariadb.json)
    *   [egg-postgres.json](file:///home/apostolosk/Desktop/eggs/egg-postgres.json)
    *   [egg-nextcloud.json](file:///home/apostolosk/Desktop/eggs/egg-nextcloud.json)
    *   [egg-paperless-ngx.json](file:///home/apostolosk/Desktop/eggs/egg-paperless-ngx.json)

### Step 2: Create Database/Cache Servers
Create the backing database and caching servers first to obtain their connection details.

1.  **Create the Redis Server:**
    *   Use the **Redis** Egg.
    *   Ensure you assign an IP and Port allocation.
    *   Set a secure **Redis Password** in the environment variables (e.g. `mysecurepassword`).
2.  **Create the PostgreSQL Server (Recommended for Paperless):**
    *   Use the **PostgreSQL** Egg.
    *   Ensure you assign an IP and Port allocation.
    *   Set a secure database user, password, and database name.
3.  **Create the MariaDB Server (Recommended for Nextcloud):**
    *   Use the **MariaDB** Egg.
    *   Ensure you assign an IP and Port allocation.
    *   Set a secure root password, database user, password, and database name.

*Note: Once created, start these servers. They will automatically initialize their data structures inside `/home/container` (which guarantees data persistence across container rebuilds/panel updates).*

### Step 3: Create App Servers and Link

#### 1. Configure Nextcloud:
*   Use the **Nextcloud** Egg.
*   Allocate a port (e.g., `8080` or any free port). Nextcloud will automatically configure Apache to listen on this port.
*   Set **Database Type** to `mysql` (for MariaDB) or `postgres`.
*   Set **Database Host** to the `<IP>:<Port>` of your MariaDB/PostgreSQL server created in Step 2.
*   Set the database name, user, and password matching the settings from Step 2.
*   Set your **Nextcloud Admin User** and **Nextcloud Admin Password**.
*   Start the server. First-time initialization may take 1-2 minutes while Nextcloud copies core system files to the persistent volume.

#### 2. Configure Paperless-ngx:
*   Use the **Paperless-ngx** Egg.
*   Allocate a port (e.g., `8000` or any free port). Paperless-ngx will run on this port.
*   Set **Redis URL** to `redis://:<password>@<IP>:<Port>` (using the details from the Redis server created in Step 2).
*   Set **Database Engine** to `postgres` (or `sqlite`/`mariadb`).
*   Set **Database Host**, **Database Port**, database name, user, and password matching your PostgreSQL server settings.
*   Set your **Paperless Admin User**, password, and email.
*   Set **Paperless URL** to the URL you'll use to access the app (required for Django CSRF protection, e.g. `http://<node-ip>:<allocated-port>`).
*   Start the server.

---

## Technical Notes

*   **Persistence:** All database clusters, file roots, media uploads, and configuration states are symlinked or configured natively to write under `/home/container/`. This ensures no data is lost when containers are updated or rebuilt.
*   **Port Compatibility:** The startup scripts dynamically re-write the web service configurations (Apache for Nextcloud, Granian/Uvicorn for Paperless-ngx) to bind to the environment variable `${SERVER_PORT}` allocated by the panel, eliminating container port collision issues.
