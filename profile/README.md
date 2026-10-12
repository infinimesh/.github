# infinimesh

**Run your own IoT device backend. Apache-2.0, written in Go.**

Every connected product ends up building the same three things: a list of devices, rules about who may see them, and a record of what each device last reported. infinimesh gives you those three things as software you run yourself. It runs on one machine with Docker Compose, or in Kubernetes.

## What it does

- **Device registry.** Accounts, namespaces and devices, stored as a graph. Access can stop at a single device.
- **Device shadow.** Reported and desired state for every device, over MQTT 3.1.1 (MQTT 5 in part).
- **Device auth.** Each device has a client certificate. TLS 1.2 and 1.3 on port 8883, or a fingerprint login on 1883 for local tests. Scoped tokens for the API.
- **One API.** REST, gRPC and Connect from the same service.
- **Console and CLI.** A web console and the `inf` command, both on the same API.

The APIs are beta. They may still change between releases.

## Quick start

You need Docker with Compose.

```bash
git clone https://github.com/infinimesh/core.git && cd core
cp .env.example .env
sed -i 's/^BASE_DOMAIN=.*/BASE_DOMAIN=127.0.0.1.nip.io/' .env
docker compose up -d
```

Open `http://console.127.0.0.1.nip.io` and log in as `infinimesh` with password `infinimesh`. That is a local default. Change it before anyone else can reach the machine.

Get the CLI with Homebrew:

```bash
brew tap infinimesh/inf-cli https://github.com/infinimesh/inf-cli
brew install inf
```

If Homebrew says the tap is untrusted, run `brew trust infinimesh/inf-cli` and try again. Other ways to install are in the [inf-cli README](https://github.com/infinimesh/inf-cli).

## Repositories

| Repo | What it is |
|---|---|
| [core](https://github.com/infinimesh/core) | The platform: registry, shadow, MQTT bridge, APIs, console, Docker Compose |
| [inf-cli](https://github.com/infinimesh/inf-cli) | The `inf` command |
| [proto](https://github.com/infinimesh/proto) | gRPC, Connect and OpenAPI definitions |
| [docs](https://github.com/infinimesh/docs) | Device examples (ESP32 MQTT and HTTP, Pico W, curl) and guides, including TLS on small devices |
| [addons](https://github.com/infinimesh/addons) | Connectors for the v3 API: Kafka, webhook, Postgres/TimescaleDB, InfluxDB, MQTT bridge, RedisTimeSeries, Elasticsearch, Snowflake, SAP HANA, S3, GCS and Azure Blob |
| [platform-operator](https://github.com/infinimesh/platform-operator) | Kubernetes operator |
| [k8s](https://github.com/infinimesh/k8s) | Plain Kubernetes manifests |
| [saas](https://github.com/infinimesh/saas) | Account service and manifests for hosting infinimesh for others |
| [website](https://github.com/infinimesh/website) | [infinimesh.org](https://infinimesh.org/) |

## Who it is for

Teams that ship connected hardware and want to own the backend. infinimesh is the server side only. Your devices bring their own MQTT client, and your data stays on your own servers.

## When not to use it

- You want a managed cloud service and do not want to run servers.
- You want dashboards and a rule engine in the box. infinimesh has neither. The infinimesh-timeseries-app repo is a Grafana viewer and not part of the platform.
- You need a frozen API today.
- You have a few devices and one user. A plain MQTT broker is simpler.

## Get involved

Found a bug or a gap in the docs? Open an issue in the repo it belongs to. Pull requests are welcome. Report security problems as described in our [security policy](https://github.com/infinimesh/.github/blob/main/SECURITY.md), not in public issues.

Start with the [core quick start](https://github.com/infinimesh/core#quickstart). If it runs on your machine in ten minutes, it is worth a closer look.
