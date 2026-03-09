# PayPlus Enterprise v3.2 — Deployment Guide

This guide covers the full installation of PayPlus Enterprise on a single-server or clustered deployment. Follow the steps in order. Do not skip the pre-flight checklist — partial installations cause difficult-to-diagnose failures at runtime.

**Audience:** System administrators and infrastructure engineers responsible for deploying and maintaining PayPlus Enterprise.

---

## Deployment models

| Model | Description | Recommended for |
|---|---|---|
| **Single-server** | All components on one host | Non-production, evaluation, and low-volume deployments |
| **Distributed (2-tier)** | Application servers and database on separate hosts | Production deployments under 500 TPS |
| **High-availability cluster** | Load-balanced application tier; active-passive database with automatic failover | Production deployments over 500 TPS or with 99.9%+ SLA requirements |

This guide covers the single-server model. The distributed and HA models follow the same procedure with the addition of [cluster-specific steps](#cluster-installation-notes).

---

## System requirements

### Application server

| Component | Minimum | Recommended |
|---|---|---|
| OS | Red Hat Enterprise Linux 8.x or Windows Server 2019 | RHEL 9.x or Windows Server 2022 |
| CPU | 8 cores | 16 cores |
| RAM | 16 GB | 32 GB |
| Disk | 100 GB SSD (OS + application) | 200 GB SSD |
| Java | OpenJDK 17 (LTS) | OpenJDK 21 (LTS) |

### Database server

| Component | Minimum | Recommended |
|---|---|---|
| Database | Oracle 19c or Microsoft SQL Server 2019 | Oracle 21c or SQL Server 2022 |
| RAM | 32 GB | 64 GB |
| Disk | 500 GB (data files) + 200 GB (archive logs) | 1 TB + 500 GB |

### Network

- PayPlus application server must reach the database on port **1521** (Oracle) or **1433** (SQL Server)
- Outbound TCP port **443** required for ACH gateway, SWIFT connectivity, and license verification
- Inbound TCP port **8443** for the PayPlus administration console
- Inbound TCP port **8080** for internal service-to-service communication (never expose externally)

---

## Pre-flight checklist

Complete every item before starting the installation. A failed pre-flight check is far easier to diagnose before installation than after.

### Host preparation

- [ ] Target host meets minimum system requirements
- [ ] OpenJDK 17+ is installed: `java -version`
- [ ] `/opt/payplus` directory exists and the installation user owns it: `mkdir -p /opt/payplus && chown payplus:payplus /opt/payplus`
- [ ] At least 10 GB of free space on the installation partition: `df -h /opt`
- [ ] The hostname resolves correctly: `hostname -f`
- [ ] Time synchronization (NTP) is running and less than 1 second off: `timedatectl status`

### Database preparation

- [ ] Database instance is running and reachable from the application server
- [ ] A dedicated database user (`PAYPLUS_APP`) exists with the permissions in [Database user permissions](#database-user-permissions)
- [ ] The `PAYPLUS` tablespace exists with at least 50 GB of available space
- [ ] A JDBC connection test succeeds from the application server (see [Test database connectivity](#step-3-test-database-connectivity))

### Network preparation

- [ ] Ports 8443 and 8080 are open on the host firewall
- [ ] Outbound port 443 is not blocked by a corporate proxy for the license server at `license.payplus.io`
- [ ] If a network proxy is required for outbound connections, the proxy hostname and port are known

### Installation files

- [ ] PayPlus Enterprise installer package is downloaded: `payplus-enterprise-3.2.x-linux-x64.tar.gz` (or `.exe` for Windows)
- [ ] Checksum of the installer matches the value in the release notes: `sha256sum payplus-enterprise-3.2.x-linux-x64.tar.gz`
- [ ] PayPlus license file (`payplus.lic`) is available

---

## Step 1: Extract the installer

```bash
cd /opt/payplus
tar -xzf ~/payplus-enterprise-3.2.x-linux-x64.tar.gz
ls -la
# Expected: bin/  conf/  lib/  scripts/  sql/  webapps/
```

Do not move the extracted directory structure — the startup scripts use relative paths.

---

## Step 2: Configure the installation

Copy and edit the main configuration file:

```bash
cp conf/payplus-install.properties.template conf/payplus-install.properties
```

Edit `payplus-install.properties`. At minimum, set these values:

```properties
# --- Database connection ---
db.type=oracle
db.host=db-server.internal
db.port=1521
db.service_name=PAYPLUS
db.username=PAYPLUS_APP
db.password=CHANGE_THIS

# --- License ---
license.file=/opt/payplus/conf/payplus.lic

# --- Application ---
app.hostname=payplus-app.internal
app.https.port=8443
app.jvm.heap.min=4g
app.jvm.heap.max=12g
```

For a full reference of all parameters, see the [Configuration runbook](configuration-runbook.md).

---

## Step 3: Test database connectivity

Run the built-in connectivity test before running the installer:

```bash
./bin/payplus-dbtest.sh --config conf/payplus-install.properties
```

Expected output on success:

```
[INFO] Testing Oracle connection to db-server.internal:1521/PAYPLUS...
[INFO] Connection successful. Oracle version: 21.9.0.0.0
[INFO] User PAYPLUS_APP has CONNECT and RESOURCE roles: YES
[INFO] Tablespace PAYPLUS: 50.2 GB available
[INFO] All checks passed. Ready to install.
```

If the test fails, do not proceed. Common failures:

| Error message | Cause | Resolution |
|---|---|---|
| `Connection refused` | The database host or port is unreachable | Verify the database is running; check firewall rules on port 1521 |
| `ORA-01017: invalid username/password` | Wrong credentials in `payplus-install.properties` | Verify `db.username` and `db.password` |
| `ORA-01034: ORACLE not available` | The Oracle instance is down | Start the Oracle instance; contact the DBA |
| `Tablespace PAYPLUS not found` | Tablespace was not created | Run `scripts/create-tablespace-oracle.sql` as a DBA user |

---

## Step 4: Run the installer

```bash
./bin/payplus-install.sh --config conf/payplus-install.properties --noninteractive
```

The installer:

1. Creates the PayPlus database schema (approximately 200 tables, 50 stored procedures)
2. Loads reference data (NACHA return codes, ISO 20022 message types, SWIFT BIC directory)
3. Deploys the application WAR files
4. Generates the keystore for HTTPS

Installation takes 10–15 minutes. Do not interrupt it. If the installer exits with a non-zero status, see `logs/install-YYYYMMDD-HHMMSS.log` for the error.

---

## Step 5: Start PayPlus

```bash
./bin/payplus-start.sh
```

Watch the startup log until the application is ready:

```bash
tail -f logs/payplus.log
```

The application is ready when you see:

```
[INFO] PayPlus Enterprise 3.2.x started on port 8443
[INFO] License: VALID - expires 2027-03-08
[INFO] Database: CONNECTED (Oracle 21c)
[INFO] Payment rail connectors: ACH (ENABLED), Fedwire (STANDBY), SWIFT (STANDBY)
```

Startup takes 60–90 seconds on a cold start. Do not proceed to the post-install validation until you see the startup confirmation.

---

## Step 6: Post-installation validation

Complete these checks before declaring the installation successful:

### Console access

Open a browser and navigate to `https://payplus-app.internal:8443/admin`. Sign in with the default credentials:

- **Username:** `admin`
- **Password:** `ChangeMe123!`

> **Change the default password immediately.** The installer does not enforce a default password change, but your security policy requires it. Go to **Administration** → **User Management** → select `admin` → **Change Password**.

### License validation

In the console, go to **Administration** → **System** → **License**. Confirm:

- Status: **Valid**
- Module entitlements match your purchased configuration (ACH, Fedwire, SWIFT, etc.)
- Expiry date is correct

### Database schema version

```bash
./bin/payplus-dbcheck.sh
```

Expected output:

```
[INFO] Schema version: 3.2.x (expected 3.2.x) — MATCH
[INFO] Reference data: 48,231 rows in 12 lookup tables — OK
```

### Test payment submission

In the console, go to **Tools** → **Payment Tester** → select **ACH Test Debit** → submit. The payment should appear in **Monitoring** → **Payment Queue** with a status of **ACCEPTED** within 5 seconds.

---

## Rollback procedure

If the installation fails or post-install validation fails:

1. Stop PayPlus: `./bin/payplus-stop.sh`
2. Drop the database schema (if it was partially created): run `scripts/drop-schema.sql` as the DBA user
3. Remove the installation directory: `rm -rf /opt/payplus`
4. Review `logs/install-YYYYMMDD-HHMMSS.log` to identify the failure point
5. Address the root cause before attempting reinstallation

> **Do not** attempt to install over a partially failed installation. Drop the schema and start clean.

---

## Cluster installation notes

For distributed and HA deployments:

- Install on each application server node using the same `payplus-install.properties`, changing only `app.hostname` per node
- On node 2+, set `db.skip_schema_create=true` — the schema is created once from node 1
- Configure a load balancer (nginx, F5, or AWS ALB) to proxy HTTPS traffic to port 8443 on each node
- Enable sticky sessions on the load balancer — PayPlus sessions are node-local in v3.2
- For Oracle RAC or SQL Server Always On, use the virtual IP or listener address, not the physical node IP

---

## Database user permissions

Run this script as a DBA to create the `PAYPLUS_APP` user with the correct permissions (Oracle):

```sql
-- Oracle — run as DBA
CREATE USER PAYPLUS_APP IDENTIFIED BY "STRONG_PASSWORD_HERE"
  DEFAULT TABLESPACE PAYPLUS
  TEMPORARY TABLESPACE TEMP
  QUOTA UNLIMITED ON PAYPLUS;

GRANT CONNECT, RESOURCE TO PAYPLUS_APP;
GRANT CREATE VIEW, CREATE SYNONYM, CREATE SEQUENCE TO PAYPLUS_APP;
GRANT SELECT ON SYS.V_$SESSION TO PAYPLUS_APP;
```

For SQL Server, see `scripts/create-user-sqlserver.sql` in the installer package.

---

## Next steps

- [Configuration runbook](configuration-runbook.md) — Full reference for all configuration parameters
- [Upgrade procedure](upgrade-procedure.md) — Version-to-version upgrade guide
- [Troubleshooting](../troubleshooting.html) — System error codes, log locations, and diagnostic procedures

---

*This document follows Microsoft Writing Style Guide conventions.*
