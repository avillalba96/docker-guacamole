# Apache Guacamole Deployment with Docker Compose

This guide explains how to deploy Apache Guacamole using Docker Compose with PostgreSQL, and set proper permissions based on user ID mappings when **userns-remap** is enabled.

---

## Prerequisites

1. Docker and Docker Compose installed.
2. Basic knowledge of Docker commands.
3. `userns-remap` enabled in Docker (`/etc/docker/daemon.json`).

---

## Steps to Deploy

### 1. Generate the Database Schema

Run the following command to generate the PostgreSQL schema for Guacamole:

```bash
docker run --rm guacamole/guacamole:1.5.5 /opt/guacamole/bin/initdb.sh --postgresql > ./guacamole/db/init/initdb.sql
```

Place the `initdb.sql` file in the directory specified in your `docker-compose.yml`.

---

### 2. Start the Containers

Start the containers so their user IDs can be checked:

```bash
docker-compose --compatibility up -d
```

---

### 3. Check Container User IDs

Run these commands to check the user IDs used by the containers:

1. For **guacd**:
   ```bash
   docker exec -it guacd_guacamole id
   ```
   Example output:
   ```plaintext
   uid=1000(guacd) gid=1000(guacd)
   ```

2. For **guacamole**:
   ```bash
   docker exec -it guacamole id
   ```
   Example output:
   ```plaintext
   uid=1001(guacamole) gid=1001(guacamole)
   ```

---

### 4. Calculate Host User IDs (Mapped IDs)

If **userns-remap** is enabled, the container IDs (e.g., `1000` for **guacd**, `1001` for **guacamole**) are mapped to host IDs using the **base ID** from `/etc/subuid`. To find the base ID:

```bash
cat /etc/subuid
```

Example output:
```plaintext
dockremap:165536:65536
```

Here:
- Base ID = `165536`
- Mapped IDs:
  - **guacd**: `165536 + 1000 = 166536`
  - **guacamole**: `165536 + 1001 = 166537`

---

### 5. Adjust File Ownership and Permissions

Based on the calculated host IDs, update the ownership and permissions of the required volumes:

```bash
chown -R 166536:166537 guacamole/guacd
chmod -R 2750 guacamole/guacd
```

---

### 6. Access Guacamole

Once permissions are correctly set, access Guacamole at:

- **URL:** [http://localhost:8080/guacamole](http://localhost:8080/guacamole)  
- **Default Login:**
  - **Username:** `guacadmin`  
  - **Password:** `guacadmin`

**Important:** Change the default credentials immediately.

---

### 7. Stop the Containers

To stop the services:

```bash
docker-compose down
```

---

## Summary

1. **Generate Schema:** Create `initdb.sql` and place it correctly.
2. **Start Containers:** Run them to check user IDs.
3. **Calculate Host IDs:** Add container IDs to the base ID in `/etc/subuid`.
4. **Set Permissions:** Adjust volume ownership using the mapped IDs.

Enjoy using Guacamole!
