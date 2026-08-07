# Traefik host proxy

- To be run on a server with a public IP address.
- Takes care of routing traffic to the appropriate containers.
- Supports HTTPS via letsencrypt.
- Terminates TLS for every other stack on the host, so a service behind it needs
  no certificate of its own.

## Naming

`traefikd_host_proxy` — the **`d`** marks a **docker**-only deployment. Traefik runs
as a container, driven by docker compose. Nothing is installed on the host itself.

`_host_` marks it as host-wide infrastructure, shared by every stack on the box,
rather than part of one application.

## How a service joins the proxy

A service in another compose project joins the external network and adds labels. It
publishes no port of its own.

```yaml
services:
  my-service:
    networks:
      - default
      - traefik_host_network
    labels:
      - "traefik.enable=true"
      # HTTP -> HTTPS. force-secure is defined on the traefik container itself.
      - 'traefik.http.routers.my-service-public.rule=Host(`my-service.example.com`)'
      - "traefik.http.routers.my-service-public.entrypoints=web"
      - "traefik.http.routers.my-service-public.middlewares=force-secure@docker"
      # HTTPS.
      - 'traefik.http.routers.my-service-secure.rule=Host(`my-service.example.com`)'
      - "traefik.http.routers.my-service-secure.entrypoints=websecure"
      - "traefik.http.routers.my-service-secure.tls.certresolver=${RESOLVER_NAME}"
      - "traefik.http.services.my-service.loadbalancer.server.port=8080"
      - "traefik.docker.network=traefik_host_network"

networks:
  traefik_host_network:
    external: true
```

## Usage

1. `cp .env.example .env`
2. Make appropriate changes to the values in .env file
3. `touch acme.json`
4. `chmod 600 acme.json`
5. Create the docker network by running this command in your terminal: `docker network create traefik_host_network`
6. `docker-compose up -d`

## Notes

- The ACME resolver uses a **DNS-01** challenge, so it can issue a certificate before
  a public DNS record for the service exists. The service is not reachable until that
  record does exist.
- Traefik logs at `INFO`, so certificate issuance and renewal are visible in
  `docker logs traefik`. At `ERROR` a successful renewal logs nothing, which is
  indistinguishable from one that never ran.
- `.env` and `acme.json` are gitignored. Never commit either one.
