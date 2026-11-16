<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="NetBird logo" width="128">
  </picture>
</p>

<h1 align="center">aac-netbird</h1>

<p align="center">Self-hosted deployment definition for NetBird, a WireGuard-based VPN mesh with identity-based access control.</p>

<p align="center">
  <a href="https://github.com/netbirdio/netbird"><img alt="Upstream" src="https://img.shields.io/badge/upstream-netbirdio%2Fnetbird-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-netbirdio%2Fdashboard%3Av2.92.0-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/netbirdio/dashboard/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-see%20upstream-green"></a>
</p>

## About

NetBird connects machines, people and services over an encrypted WireGuard mesh. Peers
connect directly where possible and fall back to a relay otherwise. Access between peers
is controlled by groups and policies that are tied to identities from an OpenID Connect
provider.

This deployment runs the NetBird web dashboard together with the combined NetBird server
(management, signal, relay and STUN in one container) and routes both through Traefik.

Key features (as provided by upstream):

- Peer-to-peer WireGuard connectivity with relay fallback
- Identity-based access policies, groups and routes
- Built-in OpenID Connect provider or an external identity provider
- Web dashboard for managing peers, users and policies
- STUN for NAT traversal

## Upstream

- Project: <https://github.com/netbirdio/netbird>
- Documentation: <https://docs.netbird.io>
- Dashboard container image: `netbirdio/dashboard:v2.92.0`
- Server container image: `netbirdio/netbird-server:0.79.0` (deployed as `aac-netbird-server`)
- License: upstream projects are licensed individually (see their repositories)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external network used for routing must exist first:

```bash
docker network create services-secured

# Provide the secret values listed under "Configuration" in stack.env / .env
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the environment files yourself (see below). Do not commit them.

## Configuration

Environment variables of the dashboard (defaults from `service.yml`; placeholder values
such as `example.com` are overridden per deployment):

| Variable | Default | Description |
|---|---|---|
| `NETBIRD_DOMAIN` | `netbird.example.com` | Public domain of the NetBird instance; all URLs are derived from it |
| `NETBIRD_MGMT_API_ENDPOINT` | `https://<NETBIRD_DOMAIN>` | Management API endpoint used by the dashboard |
| `NETBIRD_MGMT_GRPC_API_ENDPOINT` | `https://<NETBIRD_DOMAIN>` | Management gRPC endpoint used by the dashboard |
| `AUTH_AUTHORITY` | `https://<NETBIRD_DOMAIN>/oauth2` | OpenID Connect issuer |
| `AUTH_AUDIENCE` | `netbird-dashboard` | OIDC audience |
| `AUTH_CLIENT_ID` | `netbird-dashboard` | OIDC client ID |
| `AUTH_CLIENT_SECRET` | empty | OIDC client secret (empty for the public dashboard client) |
| `AUTH_SUPPORTED_SCOPES` | `openid profile email groups` | Requested OIDC scopes |
| `AUTH_REDIRECT_URI` | `/nb-auth` | Login callback path |
| `AUTH_SILENT_REDIRECT_URI` | `/nb-silent-auth` | Silent renewal callback path |
| `USE_AUTH0` | `false` | Use the Auth0 flavour of the dashboard login |
| `NGINX_SSL_PORT` | `443` | SSL port value passed to the dashboard image |
| `LETSENCRYPT_DOMAIN` | `none` | Let's Encrypt domain value passed to the dashboard image; TLS is terminated by Traefik |
| `TZ` | `Europe/Berlin` | Time zone |

Server configuration (rendered into `/etc/netbird/config.yaml`):

| Variable | Default | Description |
|---|---|---|
| `NB_TRUSTED_PROXY` | `0.0.0.0/0` | Reverse proxy addresses the server trusts for `X-Forwarded-*` headers |
| `NB_AUTH_SECRET` | `CHANGE_ME` (secret) | Server auth secret |
| `NB_STORE_ENCRYPTION_KEY` | `CHANGE_ME` (secret) | Encryption key for the server datastore |
| `NB_SESSION_COOKIE_KEY` | `CHANGE_ME` (secret) | 32-byte base64 key for reverse-proxy SSO session cookies; the setting is omitted from the config while the placeholder is set |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `80` | TCP | Dashboard and server HTTP/gRPC behind Traefik (`web`) |
| `3478` | UDP | STUN for NAT traversal |
| `33073` | TCP | gRPC listener for NetBird reverse proxies; intended for internal use only |

Volumes:

| Mount | Description |
|---|---|
| `nb_data` -> `/var/lib/netbird` | Persistent NetBird server data (datastore and keys) |
| `nb_config` -> `/etc/netbird` | Rendered `config.yaml` directory |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-netbird>.

## Credits and license

NetBird is developed by NetBird GmbH and the NetBird community. This repository only
contains the deployment definition and is not a fork of the application. Licensing of the
individual upstream components is documented in their repositories. The logo is provided
through the selfh.st icon collection; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md).
