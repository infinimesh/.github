# infinimesh

**The device layer for your IoT and AI stack.** Apache-2.0, written in Go, and you run it yourself.

![The inf CLI creates a namespace, a device and a rule, reports a hot reading, lists the raised alarm and acks it](inf-cli-demo.gif)

Every connected product ends up building the same three things: a list of devices, rules about who may see them, and a record of what each device last reported. infinimesh gives you those three things as software you run yourself. It runs on one machine with Docker Compose, or in Kubernetes.

## What it does

- **Device registry.** Accounts, namespaces and devices, stored as a graph. Access can stop at a single device.
- **Device shadow.** Reported and desired state for every device, over MQTT 3.1.1 (MQTT 5 in part), HTTP ingest, or CoAP on UDP 5683 for small devices on a private network.
- **Device auth.** Each device has a client certificate. TLS 1.2 and 1.3 on port 8883, or a fingerprint login on 1883 for local tests. Scoped tokens for the API.
- **One API.** REST, gRPC and Connect from the same service.
- **Console and CLI.** A web console and the `inf` command, both on the same API.
- **Rules and alarms (3.4).** Conditions on device state such as `reported.temp > 70` raise alarms you ack and clear in the console or the API.
- **OTA contract (3.4).** Core stores the target firmware version per device and the status the device reports back. Updates switch on only after an OTA addon registers through the plugins API. OTA is the only addon kind that registers. The hawkBit, Mender, and signed webhook addons are in the addons repo. You build them. They are not published images. The core v3.4.1 image does not serve the namespace webhook URL to the webhook addon.
- **Operations (3.4).** An audit log for alarm, plugin and OTA actions. Quotas per namespace for devices and message rates. MQTT bridge sessions in Redis with `MQTT_SESSION_STORE=redis`. Prometheus metrics on every service. A media service for files per namespace.
- **AI (coming soon).** An MCP server for agents and an LLM addon for any OpenAI-compatible API, scoped by the same permissions. Not built yet.

The v1 API is stable. New fields may be added, but breaking changes go into a new version. See the [versioning policy](https://github.com/infinimesh/core/blob/main/docs/api-versioning.md).

## Quick start

You need Docker with Compose.

```bash
git clone https://github.com/infinimesh/core.git
cd core
cp .env.example .env
docker compose up -d
```

Open http://console.127.0.0.1.nip.io

Log in as `infinimesh` / `infinimesh`.

Change the password right away.

Get the CLI with Homebrew:

```bash
brew tap infinimesh/inf-cli https://github.com/infinimesh/inf-cli
brew install inf
```

If Homebrew says the tap is untrusted, run `brew trust infinimesh/inf-cli` and try again. Other ways to install are in the [inf-cli README](https://github.com/infinimesh/inf-cli). Use a 3.4 CLI with a 3.4 core. Log in on port 443:

```bash
inf login api.127.0.0.1.nip.io:443 infinimesh infinimesh --insecure-skip-verify
```

The local certificate is self-signed, which is why the flag is there. The [CLI guide](https://github.com/infinimesh/docs/blob/main/guides/cli-getting-started.md) walks through quotas, rules, OTA addon registration and audit.

Connect a device: pick an example from [infinimesh/docs](https://github.com/infinimesh/docs/tree/main/examples) (ESP32, Raspberry Pi Pico W or curl) and follow its README.

## Docs and examples

The [infinimesh/docs](https://github.com/infinimesh/docs) repo has small device examples and practical guides. Each example is one sketch or script with a short README.

- [esp32-mqtt](https://github.com/infinimesh/docs/tree/main/examples/esp32-mqtt): ESP32 (Arduino), MQTT over TLS on port 8883 with a device certificate
- [esp32-http](https://github.com/infinimesh/docs/tree/main/examples/esp32-http): ESP32 (Arduino), HTTPS with a device token
- [picow-mqtt](https://github.com/infinimesh/docs/tree/main/examples/picow-mqtt): Raspberry Pi Pico W (MicroPython), MQTT over TLS with a device certificate
- [curl](https://github.com/infinimesh/docs/tree/main/examples/curl): a shell script that sends and reads device state over the REST API
- [CLI getting started](https://github.com/infinimesh/docs/blob/main/guides/cli-getting-started.md): login, quotas, rules and alarms, OTA addon registration, bulk assign, audit and metrics
- [TLS on small devices](https://github.com/infinimesh/docs/blob/main/guides/small-devices-tls.md): why ESP8266 class chips time out during the TLS handshake, and how to fix it

## Repositories

| Repo | What it is |
|---|---|
| [core](https://github.com/infinimesh/core) | The platform: registry, shadow, MQTT bridge, APIs, console, Docker Compose |
| [inf-cli](https://github.com/infinimesh/inf-cli) | The `inf` command |
| [proto](https://github.com/infinimesh/proto) | gRPC, Connect and OpenAPI definitions |
| [docs](https://github.com/infinimesh/docs) | Device examples for ESP32 (MQTT and HTTP), Raspberry Pi Pico W (MQTT) and curl, plus guides such as TLS on small devices |
| [addons](https://github.com/infinimesh/addons) | Connectors for the v3 API, plus OTA addons for hawkBit, Mender, and a signed webhook. No published addon images |
| [platform-operator](https://github.com/infinimesh/platform-operator) | Kubernetes operator |
| [k8s](https://github.com/infinimesh/k8s) | Plain Kubernetes manifests |
| [saas](https://github.com/infinimesh/saas) | Account service and manifests for hosting infinimesh for others |
| [website](https://github.com/infinimesh/website) | [infinimesh.org](https://infinimesh.org/) |

## Who it is for

Teams that ship connected hardware and want to own the backend. infinimesh is the server side only. Your devices bring their own MQTT client, and your data stays on your own servers.

## When not to use it

- You want a managed cloud service and do not want to run servers.
- You want dashboards in the box. infinimesh has rules and alarms, but no charts. Feed Grafana or your own tools through an addon.
- You have a few devices and one user. A plain MQTT broker is simpler.

## Get involved

Found a bug or a gap in the docs? Open an issue in the repo it belongs to. Pull requests are welcome. Report security problems as described in our [security policy](https://github.com/infinimesh/.github/blob/main/SECURITY.md), not in public issues.

Start with the [core quick start](https://github.com/infinimesh/core#quickstart). If it runs on your machine in ten minutes, it is worth a closer look.
