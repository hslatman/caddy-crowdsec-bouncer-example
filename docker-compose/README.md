# Example with docker-compose

## Setup

1. Update variables in the `.env` file and define your `10.x.x.` subnet values in `docker-compose.yml`. Edit all `~/path` as relevant to your setup.
2. Add the Caddy bouncer, generating an API key with `docker-compose exec crowdsec cscli bouncers add caddy-bouncer`. Store that key in the `.env` file.
3. Run with `docker-compose up --build --force-recreate`.

## Run

## Background

This example shows how to use CrowdSec with Caddy with docker-compose and a common network topology (similar to [this](https://www.crowdsec.net/blog/secure-docker-compose-stacks-with-crowdsec)):

```
┌────────────┐   ┌──────────┐    ┌───────────┐
│ Cloudflare ├──►│   Caddy  ├───►│  Web-app  │
└────────────┘   └─────┬────┘    └─────┬─────┘
                       │               │
                 ┌─────┴────┐    ┌─────┴─────┐
                 │ Crowdsec │    │    db     │
                 └──────────┘    └───────────┘
```

This setups considers the following:

- Cloudflare manages the DNS records of your domain, redirecting (and obfuscating with the orange cloud) the traffic to your target ip where the caddy instance lives
- SSL certificates are automatically managed by caddy with the [caddy-dns/cloudflare](github.com/caddy-dns/cloudflare) module - you need to create an API token Zone Read and DNS Edit access ((see here)[https://github.com/libdns/cloudflare])
- The [crowdsec bouncer](github.com/hslatman/caddy-crowdsec-bouncer/http) module acts as a firewall for those domains that have the `crowdsec` directive in the Caddyfile, inspecting visitors ip and allowing or denying access.
- To increase security, the `docker-compose` setup uses different networks. On host machine only ports `80/443` are open. All other services live in a separate `frontend` network, accessed by caddy to proxy requests. If you have additional "proxy" services (e.g. cloudflared tunnel) you can add them to the `proxy` network.
- To monitor all access for a domain, set a `log {}` directive in the Caddyfile. This log file is mounted on the crowdsec docker instance and inspected with the `acquis.d/caddy.yaml` config. Alternatively, if you are using services with specific [collections](https://app.crowdsec.net/hub/collections), you might decide to use that as opposed to enabling access logging.
