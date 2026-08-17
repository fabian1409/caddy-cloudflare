# caddy-cloudflare

A custom [Caddy](https://caddyserver.com/) Docker image built with the
[`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare) plugin, used to
reverse-proxy home services (Home Assistant, internal sites, etc.) with automatic
HTTPS via Cloudflare's DNS-01 challenge — no public inbound ports required.

Built automatically on every push via GitHub Actions and published to
[GitHub Container Registry](https://ghcr.io). Dependabot keeps the base Caddy
version and Actions dependencies up to date.


## Usage

### Docker Compose

```yaml
services:
  caddy:
    image: ghcr.io/fabian1409/caddy-cloudflare:latest
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - CF_API_TOKEN=${CF_API_TOKEN}
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
```

### Caddyfile example

```
{
    email you@example.com
}

*.yourdomain.com {
    tls {
        dns cloudflare {env.CF_API_TOKEN}
    }

    @foo host foo.yourdomain.com
    handle @foo {
        reverse_proxy localhost:8123
    }
}
```

`CF_API_TOKEN` needs both `Zone.Zone:Read` (to look up the zone) and
`Zone.DNS:Edit` (to create/remove the DNS-01 challenge TXT record) permissions,
scoped to the relevant zone, in the Cloudflare dashboard.
 
The `email` in the global options block isn't required for certs to work, but
Let's Encrypt uses it to notify you if renewals ever start failing — worth
setting to an address you actually check.

