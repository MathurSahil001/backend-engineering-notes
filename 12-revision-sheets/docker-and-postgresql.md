# Docker and PostgreSQL: Quick Revision

| Question | Answer |
| --- | --- |
| Image or container? | Image is a reusable package; container is its runtime instance. |
| `5433:5432`? | Host port 5433 forwards to container port 5432. |
| Host-run FlashCart JDBC URL? | `jdbc:postgresql://localhost:5433/flashcart` for this mapping. |
| Compose peer JDBC URL? | `jdbc:postgresql://postgres:5432/flashcart` if the service is named `postgres`. |
| Why a volume? | Preserve database files across container replacement. It is not a backup. |
| What is WSL 2 doing? | Docker Desktop uses a Linux engine on Windows to run Linux containers. |
| Why wrong data at `localhost:5432`? | Another local PostgreSQL may own that host port. Check the listener and database identity. |

**Debug sequence:** port listener → Docker mapping → container readiness → effective app URL → credentials and database identity.

**Recall:** Why does `localhost` mean something different inside a container? What survives `docker rm`? When should you avoid publishing the database port?

Full explanation: [Docker foundations](../06-docker/docker-foundations-and-postgresql-ports.md). Real case: [port conflict journal](../11-debugging-journal/postgresql-port-conflict.md).
