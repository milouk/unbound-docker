# milouk/unbound

[![Docker Hub](https://img.shields.io/docker/v/milouk/unbound?sort=semver&label=Docker%20Hub)](https://hub.docker.com/r/milouk/unbound)
[![Docker Pulls](https://img.shields.io/docker/pulls/milouk/unbound)](https://hub.docker.com/r/milouk/unbound)
[![GitHub Actions](https://github.com/milouk/unbound-docker/actions/workflows/docker-release.yml/badge.svg)](https://github.com/milouk/unbound-docker/actions/workflows/docker-release.yml)

Docker image for [Unbound](https://nlnetlabs.nl/projects/unbound/), a validating, recursive, and caching DNS resolver.

Built from source on each upstream release. Based on the work of [Matthew Vance](https://github.com/MatthewVance/unbound-docker).

## Features

- Built from official [NLnetLabs/unbound](https://github.com/NLnetLabs/unbound) source with SHA256 verification
- Custom OpenSSL compiled from source with hardened flags (no weak ciphers, static linking, stack protection)
- Compiled with: libevent, libnghttp2 (DoH), dnstap, TCP Fast Open, EDNS Client Subnet
- Auto-tuned threads and cache sizes based on available RAM/CPU at runtime
- DNSSEC validation enabled by default
- Config path compatible with `mvance/unbound` — drop-in replacement

## Quick Start

```bash
docker run -d \
  --name unbound \
  -p 53:53/tcp \
  -p 53:53/udp \
  -v /path/to/config:/opt/unbound/etc/unbound \
  milouk/unbound:latest
```

## Docker Compose

```yaml
services:
  unbound:
    image: milouk/unbound:latest
    container_name: unbound
    volumes:
      - /data/unbound:/opt/unbound/etc/unbound/
    restart: always
    healthcheck:
      test: ['CMD', 'drill', '@127.0.0.1', 'cloudflare.com']
      interval: 30s
      timeout: 5s
      retries: 3
```

## Configuration

Mount your config directory to `/opt/unbound/etc/unbound/`. If no `unbound.conf` is present, a default one is generated at startup with sensible security and performance defaults.

Optional config files the startup script will include if present:

| File | Purpose |
|------|---------|
| `unbound.conf` | Main config (auto-generated if missing) |
| `a-records.conf` | Local A/PTR records |
| `forward-records.conf` | Upstream forwarders |
| `srv-records.conf` | Local SRV records |

### Remote Control (for unbound-exporter)

To enable TLS remote control (required for Prometheus metrics export), add to your `unbound.conf`:

```conf
remote-control:
    control-enable: yes
    control-interface: 0.0.0.0
    control-port: 8953
    server-key-file: "/opt/unbound/etc/unbound/unbound_server.key"
    server-cert-file: "/opt/unbound/etc/unbound/unbound_server.pem"
    control-key-file: "/opt/unbound/etc/unbound/unbound_control.key"
    control-cert-file: "/opt/unbound/etc/unbound/unbound_control.pem"
```

## Tags

| Tag | Description |
|-----|-------------|
| `latest` | Latest stable release |
| `1.25.0` | Specific Unbound version |

## Automated Builds

A GitHub Actions workflow runs daily, checks [NLnetLabs/unbound](https://github.com/NLnetLabs/unbound) for new stable releases (excluding RCs), fetches the SHA256 of the tarball, and builds and pushes a new image automatically.

No manual intervention needed on new Unbound releases.

## Building Locally

```bash
docker build \
  --build-arg UNBOUND_VERSION=1.25.0 \
  --build-arg UNBOUND_SHA256=062a6eda723fe2f041bee4079b76981569f1d12e066bbd74800242fc1ebddec7 \
  -t milouk/unbound:1.25.0 .
```

## Credits

- [NLnetLabs](https://nlnetlabs.nl/) — Unbound DNS resolver
- [Matthew Vance](https://github.com/MatthewVance/unbound-docker) — Original Docker build approach
