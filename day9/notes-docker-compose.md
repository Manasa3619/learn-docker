Docker Compose Notes

What Docker Compose is
- Docker Compose is a tool for defining and running multi-container applications.
- You describe services, networks, and volumes in a `docker-compose.yaml` file.
- Compose starts containers in the right order and connects them on a shared network.

Key concepts
- Service: A container definition for one part of your app (frontend, backend, db).
- Image: A built artifact used to run a container.
- Build: Instructions to build an image from a Dockerfile.
- Volume: Persistent storage outside the container filesystem.
- Network: Virtual network that lets services communicate by service name.
- Environment: Configuration passed to the container at runtime.
- Healthcheck: A command to test if a service is ready.

Typical structure of a compose file
- `services`: Defines containers. Each service can have `build` or `image`.
- `ports`: Maps host ports to container ports.
- `env_file`: Loads environment variables from a file.
- `depends_on`: Controls startup order and can wait for healthchecks.
- `volumes`: Defines named volumes.

What Compose does for you
- Creates a network so services can talk by name (e.g., `backend` can reach `db`).
- Starts containers in dependency order.
- Applies environment variables and volume mounts consistently.
- Rebuilds images from Dockerfiles with a single command.
- Shows logs for the whole stack in one place.

Common patterns
- Database + API + UI:
  - `db` provides persistence and runs on an internal network.
  - `backend` connects to `db` using the service name as hostname.
  - `frontend` talks to `backend` using the host port or reverse proxy.
- Use a volume for the database to keep data across restarts.
- Add healthchecks for services that need to be “ready” before others start.
- Keep secrets in `.env` files, not in the compose file itself.

Environment variables
- Use `env_file` for service-specific configuration.
- Example: backend `DATABASE_URL` points to `db` service.
- Example: frontend `REACT_APP_API_BASE_URL` used at build time in React.
- `.env` at the compose root is a special file read by Compose itself.

Build vs image
- Use `build` when you have a Dockerfile and source code locally.
- Use `image` when pulling from a registry.
- You can combine `build` with `image` to tag the built image.

Networking rules
- All services are on the same default network unless configured otherwise.
- A service can reach another by using the service name as the hostname.
- The host can reach a service only through published ports.

Volumes
- Named volumes are managed by Docker.
- Bind mounts map a host directory into a container (useful for dev).
- Database containers should use named volumes to avoid data loss.

Healthchecks in practice
- A healthcheck is a command Docker runs inside the container to verify readiness.
- Example for Postgres: `pg_isready -U postgres`.
- Compose can use `depends_on` with `condition: service_healthy` to wait for DB readiness.
- Without healthchecks, dependent services can start too early and fail connection attempts.

Without Docker Compose
- You must build each image manually with `docker build`.
- You must run each container manually with `docker run`, add ports, env vars, and volumes.
- You must create a network and connect each container to it.
- You must ensure startup order and health readiness yourself.
- You must remember and coordinate multiple long commands every time.

Useful commands
- Start: `docker compose up`
- Start with rebuild: `docker compose up --build`
- Stop: `docker compose down`
- Stop and remove volumes: `docker compose down -v`
- Check status: `docker compose ps`
- Logs: `docker compose logs -f`
- Rebuild one service: `docker compose build <service>`
- Run a command in a service: `docker compose exec <service> <cmd>`

Tips for reliable stacks
- Add healthchecks for DBs and queues.
- Keep resource usage reasonable by limiting unnecessary images.
- Use explicit ports to avoid collisions.
- Avoid hard-coding secrets in repo files.
- Prefer minimal images in production (slim or alpine).
