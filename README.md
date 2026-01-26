# Traefik host proxy

- To be run on a server with a public IP address.
- Takes care of routing traffic to the appropriate containers.
- Supports HTTPS via letsencrypt.

## Usage
1. `cp .env.example .env`
2. Make appropriate changes to the values in .env file
3. `touch acme.json`
4. `chmod 600 acme.json`
5. Create the docker network by runnung this command in your terminal: `docker network create traefik_host_network`
6. `docker-compose up -d`
