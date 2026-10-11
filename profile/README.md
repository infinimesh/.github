# Infinimesh

Open IoT platform. Apache-2.0. Device registry, auth, digital-twin shadow, MQTT bridge, and REST, gRPC, and Connect APIs. One tenant graph, no vendor lock-in.

The public site is [infinimesh.org](https://infinimesh.org/).

This page is the map. Install and day-to-day operation are below. Each repository's own README is the detail for that repo.

## Repositories

| Repo | Module or path | Role |
|---|---|---|
| [core](https://github.com/infinimesh/core) | `github.com/infinimesh/core` | Platform: APIs, shadow, MQTT bridge, console, Docker Compose |
| [proto](https://github.com/infinimesh/proto) | `github.com/infinimesh/proto` | Schemas and generated stubs. Edit protos here, then `buf generate` |
| [inf-cli](https://github.com/infinimesh/inf-cli) | `github.com/infinimesh/inf-cli` | `inf` command |
| [addons](https://github.com/infinimesh/addons) | `github.com/infinimesh/addons` | Connectors. Not part of the platform process |
| [platform-operator](https://github.com/infinimesh/platform-operator) | `github.com/infinimesh/enterprise-operator` | Kubernetes operator. The name `operator` is retired on this org |
| [k8s](https://github.com/infinimesh/k8s) | manifests | Cluster install of the core images. ArangoDB, RabbitMQ, Redis. No Kafka |
| [saas](https://github.com/infinimesh/saas) | `github.com/infinitedevices/platform-saas` | Account service and hosting manifests. No kubeconfigs; credentials in the manifests are placeholders |
| [website](https://github.com/infinimesh/website) | static files | [infinimesh.org](https://infinimesh.org/) |

`core` depends on `proto`. `inf-cli` depends on `core` and `proto`. Addons talk to the platform API. They do not import `core`.

## Install on one machine

Requirements: Docker, Docker Compose, Go 1.27 for the CLI.

```bash
git clone https://github.com/infinimesh/core.git
cd core
cp .env.example .env
docker compose up -d
go install github.com/infinimesh/inf-cli@main
```

`.env.example` sets `BASE_DOMAIN=infinimesh.local`. For that default, point these names at the host: `console`, `api`, `traefik`, `db`, `rbmq`, `media`. Or set `BASE_DOMAIN` to a name that already resolves, such as `127.0.0.1.nip.io`.

Local URLs, with that domain:

| Surface | URL |
|---|---|
| Console | `http://console.${BASE_DOMAIN}` |
| API (gRPC, Connect, REST) | `http://api.${BASE_DOMAIN}` |
| Traefik | `http://traefik.${BASE_DOMAIN}` |
| ArangoDB | `http://db.${BASE_DOMAIN}` |
| RabbitMQ management | `http://rbmq.${BASE_DOMAIN}` |
| MQTT | `1883` plain, `8883` TLS (`hack/server.crt`, `hack/server.key`) |

Local-only passwords, from `.env.example` and `docker-compose.yaml`. Change them before any shared use.

- Platform root: user `root`, password `infinimesh` (`INF_DEFAULT_ROOT_PASS`)
- ArangoDB: `root` / `openSesame`
- RabbitMQ: `infinimesh` / `infinimesh`

```bash
API=http://api.${BASE_DOMAIN}:8000
inf login --api "$API" --username root --password infinimesh --insecure
inf namespaces create demo
inf namespaces use demo
inf devices create demo-sensor
```

More of the same flow is in [core/docs/installation.md](https://github.com/infinimesh/core/blob/main/docs/installation.md) and [core/docs/device-auth.md](https://github.com/infinimesh/core/blob/main/docs/device-auth.md). OpenAPI is `api.swagger.json` in core, generated from [proto](https://github.com/infinimesh/proto).

## Public domain

Traefik is the proxy in core's `docker-compose.yaml`. On a real `BASE_DOMAIN`, with TCP 80 and 443 open and the ACME email set in `traefik.yml`, it gets Let's Encrypt certificates for `console`, `api`, `traefik`, `rbmq`, `db`, and `media`.

MQTT on port 8883 is terminated by the bridge, not by the proxy. `docker-compose.public.yml` asks Let's Encrypt for `mqtt.$BASE_DOMAIN` and points the bridge at Traefik's `acme.json` (`ACME`). The bridge uses the certificate whose name starts with `mqtt.`.

Caddy is the other proxy (`docker-compose.caddy.yml` and `Caddyfile`). The bridge cannot read Caddy's storage. That overlay mounts Caddy's PEM files as `TLS_CERT_FILE` and `TLS_KEY_FILE`.

The device registry in the Compose stack is the `api` service. Inside the Docker network it is plaintext gRPC at `api:8000`. HTTP/JSON is the `gateway` service. Pairing is the `pairing` service; its public path stays `/infinimesh.handsfree.`. Public clients use `api.$BASE_DOMAIN` port 443:

```bash
inf login --api api.example.com:443 --username root --password 'your-root-password'
```

Devices use `mqtt.$BASE_DOMAIN:8883` with a client certificate. Steps, DNS, and which ports to leave unpublished are in [core/docs/public-tls.md](https://github.com/infinimesh/core/blob/main/docs/public-tls.md).

The [platform-operator](https://github.com/infinimesh/platform-operator) reconciles the same images. MQTT TLS is `spec.mqtt.secretName`, mounted at `/cert` on the bridge (`tls.crt`, `tls.key`).

## Install on Kubernetes

The operator installs a platform into a cluster. Static manifests for the same processes are in [k8s](https://github.com/infinimesh/k8s) (`kubectl apply -k base`, namespace `infinimesh`).

```bash
kubectl apply -f https://raw.githubusercontent.com/infinimesh/platform-operator/main/manifests/crd.yaml
kubectl apply -f https://raw.githubusercontent.com/infinimesh/platform-operator/main/manifests/operator.yaml
```

Then apply a Platform manifest. The root password is written to a Secret.

Docker Compose in `core` is the single-host stack. The operator and the `k8s` base are the cluster install.

## Operate

- Console manages accounts, namespaces, and devices.
- CLI does the same against the API. `inf login` first. Pass `--insecure` only for local h2c.
- Device MQTT uses the token or certificate issued for that device. See `docs/device-auth.md` in core.
- Shadow is the device twin. `api` and `shadow` are separate processes under `cmd/` in core.
- Logs and health for a Compose install: `docker compose ps` and `docker compose logs` in the core checkout.
- Images publish to `ghcr.io/infinimesh/core/<service>` when a `vX.Y.Z` tag is pushed on core.

## Addons

Clone [addons](https://github.com/infinimesh/addons). One Go module, one directory per connector.

| Directory | Needs |
|---|---|
| `redisstream` | `USERNAME`, `PASSWORD`, `API_URL`, `REDIS_ADDR` |
| `elasticsearch`, `snowflake`, `saphana` | the Redis stream from `redisstream` |
| `timeseries` | the platform API, RedisTimeSeries |
| `cloudconnect` | the platform API, then S3, GCS, or Azure Blob |

```bash
docker compose -f redisstream/docker-compose.yml up --build
docker compose -f elasticsearch/docker-compose.yml up --build
```

Run those from the addons repository. Placeholder credentials in the compose files are `CHANGEME`.

## Checks

Go repositories run, on push and pull request: `gofmt`, `go vet`, `go test`, `govulncheck`, and CodeQL. Dependabot opens weekly update PRs. Core also runs the console test suite.

## License

Apache-2.0. The `LICENSE` file in each repository is the grant.
