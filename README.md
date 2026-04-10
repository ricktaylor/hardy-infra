# Hardy Deployment

Docker Compose deployment for Hardy BPA with observability.

## Quick start

```bash
docker compose up
```

This starts the BPA with embedded TCPCLv4, PostgreSQL, MinIO (S3), and the
full observability stack (OpenTelemetry Collector, Prometheus, Grafana).

## Compose files

| File              | Storage                      | Services                              |
| ----------------- | ---------------------------- | ------------------------------------- |
| `compose.yml`     | PostgreSQL + S3 (persistent) | hardy, postgres, minio, observability |
| `compose.mem.yml` | In-memory (ephemeral)        | hardy, observability                  |

```bash
# Persistent storage (default)
docker compose up

# In-memory storage
docker compose -f compose.mem.yml up
```

## Standalone TCPCL

Both compose files support `--profile tcpcl` to add a standalone TCPCLv4
server as a second CLA alongside the embedded one.

```bash
docker compose --profile tcpcl up
docker compose -f compose.mem.yml --profile tcpcl up
```

The embedded TCPCL (`tcp-cla-1`) listens on port **4556**.
The standalone TCPCL (`tcp-cla-2`) listens on port **4557**.

> ⚠️ When using standalone TCPCL, you **must** do both of the following
> or the services will conflict on port 4556:
>
> 1. Comment out the `[[clas]]` section in the BPA config (`configs/hardy/hardy.toml`
>    or `configs/hardy/hardy-mem.toml`)
> 2. Comment out the `4556:4556` port mapping on the `hardy` service in the compose file
>
> Alternatively, change the standalone TCPCL to a different port (e.g. `4557:4556`)
> to run both CLAs side by side.

## Ports

| Port  | Service                                   |
| ----- | ----------------------------------------- |
| 50051 | BPA gRPC API                              |
| 4556  | TCPCLv4 (embedded or standalone)          |
| 3000  | Grafana                                   |
| 4317  | OTLP gRPC (OpenTelemetry Collector)       |
| 9000  | MinIO S3 API (`compose.yml` only)         |
| 9001  | MinIO Console (`compose.yml` only)        |

## Observability

All deployments include:

- **OpenTelemetry Collector** -- receives OTLP metrics/traces/logs from Hardy
- **Prometheus** -- scrapes metrics from the collector
- **Grafana** -- pre-provisioned with a Prometheus datasource and a
  Hardy BPA dashboard at [http://localhost:3000](http://localhost:3000)
  (login: `admin` / `admin`)

The Hardy BPA dashboard shows bundle throughput, pipeline status, drop
reasons, filter activity, cache performance, and infrastructure gauges.

## Configuration

```
configs/
  hardy/
    hardy.toml       # BPA + embedded TCPCL + S3/PostgreSQL
    hardy-mem.toml   # BPA + embedded TCPCL + in-memory
  tcpcl/
    tcpcl.toml       # Standalone TCPCLv4 server
  otel/
    otel-collector.yml
  prometheus/
    prometheus.yml
```

## Volumes

`compose.yml` persists data across restarts:

| Volume            | Purpose                        |
| ----------------- | ------------------------------ |
| `bundles`         | Bundle data (local disk cache) |
| `postgres-data`   | PostgreSQL metadata            |
| `minio-data`      | S3 bundle storage              |
| `prometheus-data` | Prometheus metrics             |
| `grafana-data`    | Grafana state                  |

`compose.mem.yml` only persists `prometheus-data` and `grafana-data`.

To reset all data:

```bash
docker compose down -v
```
