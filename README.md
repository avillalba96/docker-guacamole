# Apache Guacamole Deployment with Docker Compose

This guide provides steps to deploy Apache Guacamole using Docker Compose with PostgreSQL as the database backend.

## Prerequisites

1. Docker and Docker Compose installed on your system.
2. Basic knowledge of Docker commands.

## Steps to Deploy

### 1. Generate the Database Initialization Script

Run the following command to generate the PostgreSQL schema required for Guacamole:

```bash
docker run --rm guacamole/guacamole:1.5.5 /opt/guacamole/bin/initdb.sh --postgresql > ./guacamole/db/init/initdb.sql
```

This will create an `initdb.sql` file inside the `./guacamole/db/init/` directory. Ensure this path matches the volume configuration in your `docker-compose.yml`.

### 2. Start the Services

Use the following command to start the Guacamole services:

```bash
cp env_example .env
docker-compose --compatibility up -d; docker-compose logs -ft --tail=35
```

- `--compatibility`: Ensures backward compatibility with Docker Compose V2.
- `-d`: Runs the services in detached mode.
- `logs -ft --tail=35`: Displays the last 35 logs for troubleshooting and confirms the services are running.

### 3. Access Guacamole

Once the services are up and running, you can access Guacamole in your browser:

- URL: [http://localhost:8080/guacamole](http://localhost:8080/guacamole)
- Default Credentials:
  - **Username:** `guacadmin`
  - **Password:** `guacadmin`

> **Note:** Change the default credentials after your first login for security purposes.

---

## Troubleshooting

- Ensure the `initdb.sql` file is correctly located in the directory specified in your `docker-compose.yml`.
- Check logs for issues:

  ```bash
  docker-compose logs -ft
  ```

- Verify database tables are created in PostgreSQL:

  ```bash
  docker exec -it <postgres_container_name> psql -U <db_user> -d <db_name>
  \dt
  ```

## Stopping the Services

To stop the Guacamole services, use:

```bash
docker-compose down
```

---

## Additional Notes

- Ensure the PostgreSQL database has persistent storage to avoid data loss.
- Customize the `docker-compose.yml` file to fit your environment's requirements.

Enjoy using Apache Guacamole!

