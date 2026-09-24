# Docker Foundations: Images, Containers, Ports, and PostgreSQL

## 1. Core Idea

An **image** is a packaged filesystem and startup instructions. A **container** is a running or stopped instance of an image. One image can produce many containers, each with its own runtime state and configuration.

Docker helps FlashCart run PostgreSQL locally in a predictable environment. A published port lets the application on the host reach PostgreSQL inside the container; a volume keeps database files across container replacement.

## 2. Mental Model

```text
PostgreSQL image ── create/run ──> PostgreSQL container
                                      listens on container port 5432
Host app ── localhost:5433 ──> host port 5433 ──> container port 5432
                                                |
                                       database files in a volume
```

`-p 5433:5432` reads **host port : container port**. It does not change PostgreSQL's internal listening port. `localhost` always refers to the network environment of the caller: the Windows host, a container, and a remote server each have their own `localhost`.

## 3. How It Fits In The Backend

During local development, a Spring Boot service on the host opens a JDBC connection to `localhost:5433`. Docker forwards it to PostgreSQL's port `5432` in the container. A service running in another container on the same Compose network should normally use the database service name and internal port, for example `postgres:5432`, rather than its own `localhost` or the host's published port.

## 4. Important Concepts

- **Image versus container:** rebuild or pull an image to change the packaged software; restart a container to restart its process. A container's writable layer is not a safe place to keep durable database data.
- **Port mapping:** `5433:5432` binds port 5433 on the host to port 5432 in the container. The host port must be free. Publishing a port makes the service reachable through that host binding; choose the binding deliberately.
- **Volume:** mount a named volume at PostgreSQL's data directory. The volume can outlive a removed container. A volume is persistence, not a backup or a guarantee against corrupt or accidentally deleted data.
- **Docker Desktop + WSL 2:** on Windows, Docker Desktop can use a WSL 2 based Linux engine. The Docker CLI talks to the engine; containers run in that Linux environment. A stopped Docker Desktop or unavailable WSL engine prevents containers from running. Docker Desktop and a local Windows PostgreSQL installation are separate processes that may compete for a host port.
- **Readiness:** a container marked running may not yet have a database ready to accept connections. Inspect its logs or run a readiness check before blaming the application.

## 5. Engineering Decisions

- **Why use it:** keep local dependencies reproducible and reduce setup differences across machines.
- **When:** local development, integration testing, and controlled deployment environments where containers fit operational needs.
- **When not:** avoid adding Docker just for a one-off program with no environment dependency. Do not assume Docker alone handles backups, security, or orchestration.
- **Tradeoffs:** isolation and repeatability cost memory, disk, startup time, and another networking layer to debug.
- **Alternatives:** install PostgreSQL directly on the host; use an existing managed/shared database where suitable. Direct installation has fewer layers but can cause version and port conflicts.
- **Host port choice:** use `5433:5432` when host port 5432 is occupied and local tooling needs access. If only other containers connect, a host port may be unnecessary.

## 6. Real Example

For a FlashCart `user-service` running on the Windows host, a local Compose service could be:

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: flashcart
      POSTGRES_USER: flashcart
      POSTGRES_PASSWORD: change-me-locally
    ports:
      - "127.0.0.1:5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

The password shown is only a placeholder; use local secrets and never commit a real password. From the host, the Spring datasource URL is `jdbc:postgresql://localhost:5433/flashcart`. If `user-service` moves into the same Compose project, use `jdbc:postgresql://postgres:5432/flashcart` from that container. Match the volume mount path to the chosen image version's documented `PGDATA` layout before relying on persistence.

## 7. Common Mistakes

- Pointing the host app at `localhost:5432` while the container is published as `5433:5432`. That reaches whatever process is listening on host port 5432, possibly a separately installed PostgreSQL.
- Pointing a containerized app at `localhost:5433`. That means the app container itself, not the Windows host.
- Swapping port order and assuming `5433:5432` means container 5433.
- Deleting the container and expecting its unmounted data to remain; deleting a volume and expecting its contents to be recoverable.
- Assuming a successful TCP connection proves it reached the intended database. Credentials, database name, server version, and data can identify the wrong instance.

## 8. Debugging Lessons

Our PostgreSQL port conflict illustrates the rule: **`localhost:5432` means “whatever process is listening on port 5432” in the caller's environment.** It is not shorthand for “my Docker PostgreSQL.” If host port 5432 is already used, publishing `5432:5432` fails or the application reaches the host's other PostgreSQL. Publish `5433:5432`, then update the host application's JDBC URL to 5433.

Check each boundary in order:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
docker logs <postgres-container>
docker exec <postgres-container> pg_isready -U flashcart -d flashcart
```

On Windows PowerShell, inspect listeners with `Get-NetTCPConnection -LocalPort 5432,5433 -State Listen` and match each `OwningProcess` with `Get-Process -Id <pid>`. Then verify the actual JDBC URL, credentials, and target database. If the service cannot connect, distinguish port occupied, connection refused, authentication failure, and database missing; each implies a different layer. See the [case journal](../11-debugging-journal/postgresql-port-conflict.md).

## 9. Interview Discussion

- Explain why a container may be running while the host cannot connect to its database.
- Explain the difference between `localhost:5433` from the host and `postgres:5432` from a peer container.
- Discuss whether you would publish a production database port at all and how you would protect the data.
- Explain how you would move a container to another machine while retaining or migrating database data.

## 10. 5-Minute Revision

- Image = package; container = instance/process.
- `5433:5432` = host 5433 → container 5432.
- `localhost` depends on **where the client runs**.
- Named volume keeps database files outside the container lifecycle; it is not a backup.
- Docker Desktop + WSL 2 runs a Linux container engine on Windows.
- Check port listeners, mapping, readiness, URL, and database identity in that order.

## 11. Active Recall Questions

1. If Windows PostgreSQL already listens on 5432, what happens when Docker tries `5432:5432`?
2. What JDBC URL does a host-run Spring Boot app use with `5433:5432`? What URL does a peer container use?
3. Why might `localhost:5432` connect successfully but show unexpected tables?
4. What does a named volume survive, and what does it not protect against?
5. How would you distinguish a port conflict from PostgreSQL startup delay or incorrect credentials?
