# docker-compose-healthcheck

Since docker-compose [version 2.1 file format](https://docs.docker.com/compose/compose-file/compose-versioning/#version-21) the [healthcheck](https://docs.docker.com/compose/compose-file/#healthcheck) parameter has been introduced.
This allows a check to be configured in order to determine whether or not containers for a service are "healthy."

## How can I wait for container X before starting Y?

This is a common problem and in earlier versions of docker-compose requires the use of additional tools and scripts such as [wait-for-it](https://github.com/vishnubob/wait-for-it) and [dockerize](https://github.com/jwilder/dockerize).
Using the `healthcheck` parameter the use of these additional tools and scripts is often no longer necessary.

## Waiting for PostgreSQL to be "healthy"

A particularly common use case is a service that depends on a database, such as PostgreSQL.
We can configure docker-compose to wait for the PostgreSQL container to startup and be ready to accept requests before continuing.

The following healthcheck has been configured to periodically check if PostgreSQL is ready using the `pg_isready` command, see the documentation [here](https://www.postgresql.org/docs/9.4/static/app-pg-isready.html).
```
healthcheck:
  test: ["CMD-SHELL", "pg_isready"]
  interval: 30s
  timeout: 30s
  retries: 3
```
If the check is successful the container will be marked as `healthy`. Until then it will remain in an `unhealthy` state.
For more details about the healthcheck parameters `interval`, `timeout` and `retries` see the documentation [here](https://docs.docker.com/engine/reference/builder/#healthcheck).

Services that depend on PostgreSQL can then be configured with the `depends_on` parameter as follows:
```
depends_on:
  postgres-database:
    condition: service_healthy
```

## Waiting for PostgreSQL before starting Kong

In [this complete example](docker-compose.yml) docker-compose waits for the PostgreSQL service to be "healthy" before starting [Kong](https://getkong.org/), an open-source API gateway.

Test it out with:
```
docker-compose up -d
```
Wait until both services are running:
```
Starting dockercomposehealthcheck_kong-database_1
Starting dockercomposehealthcheck_kong_1
```
Test by querying Kong's admin endpoint:
```
curl http://localhost:8001/
```

## License

MIT License - see the [LICENSE](LICENSE) file for details
