# PostgreSQL host port conflict

**Context:** Local FlashCart development on Windows with Docker Desktop, WSL 2, and PostgreSQL. This entry captures the port conflict discussed during setup; exact command output and process IDs have not been recorded.

## Symptom

The application or database client using `localhost:5432` did not reliably target the intended Docker PostgreSQL. Publishing the container on host port 5432 can also fail if another process already owns it.

## Mental model and likely cause

The PostgreSQL process inside the container listens on 5432. The host can have a separate PostgreSQL listener on 5432. `localhost:5432` selects the listener on the caller's machine; it does not select a container by name. The remedy for a host port conflict is a free host port such as `5433`, mapped to the container's `5432`, plus an updated connection URL.

## Evidence to collect next time

1. Record `docker ps` port mappings and the PostgreSQL container logs.
2. On Windows, inspect port 5432 and 5433 listeners and their owning processes.
3. Record the app's effective JDBC host, port, and database name without exposing the password.
4. Query `SELECT current_database(), inet_server_port(), version();` against the connected database to identify the endpoint. Note that `inet_server_port()` reports the server-side port (typically 5432), even when the host connection used 5433.

## Fix and verification

Publish `127.0.0.1:5433:5432` for host-only development, point a host-run app at `jdbc:postgresql://localhost:5433/flashcart`, and confirm a known FlashCart table or database identity. If both app and database run on the same Compose network, connect to `postgres:5432` instead. Keep real credentials outside committed examples.

## Prevention

Document which process owns each host port; keep the host and container connection URLs distinct; verify the connected database before running migrations or destructive SQL.

**Lesson:** `localhost:5432` means “whatever process is listening on port 5432” from that caller's network environment.
