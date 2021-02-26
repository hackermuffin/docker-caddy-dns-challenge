# caddy-local-dns

This repo contains the files for a modified caddy docker image, configured to reverse proxy a site over HTTPS using a DNS challenge, designed with either a cloudflare or duckdns DNS provider. One use case is to create an SSL connection over a local network, which is useful for services such as bitwarden, or simply to avoid browser errors.

The repo currently contains docker setup and configuration for both cloudflare and duckdns, with the following tags:
- `latest`, `cloudflare`
- `duckdns`

However, with a custom caddyfile, the duckdns version can be used for a variety of other DNS providers, as documented https://go-acme.github.io/lego/dns/.

## Basic Usage
You will need a domain configured to resolve to the (possibly local) ip address of the server where caddy will run. This can be done either with an existing domaing through [Cloudflare ](https://www.cloudflare.com/) or for free with [DuckDNS](https://www.duckdns.org/). 

The contianer uses port 80 and 443, which can be forwarded to any other host port, and should have a persisted /data folder. 

The following environment variables should be set:
- `DOMAIN`: domain on which caddy will request a certificate
- `TOKEN`: cloudflare  API token (only if using cloudflare)
- `DUCKDNS_TOKEN`: duckdns token (only if using duckdns)
- `REVERSE_PROXY`: The address and port of the service to be reverse proxied

The default container can be run with the following:

```console
$ docker run -d \
    -p 80:80 \
    -p 433:433 \
    -v caddy_data:/data \
    -v caddy_config:/config \
    -e DOMAIN=<domain> \ 
    -e TOKEN=<token> \
    -e REVERSE\_PROXY=<proxy-address> \
    hackermuffin/caddy-dns-challenge
```

Or for the duckdns module:
```console
$ docker run -d \
    -p 80:80 \
    -p 433:433 \
    -v caddy_data:/data \
    -v caddy_config:/config \
    -e DOMAIN=<domain> \ 
    -e DUCKDNS_TOKEN=<token> \
    -e REVERSE\_PROXY=<proxy-address> \
    hackermuffin/caddy-dns-challenge:duckdns
```

Or, in a docker-compose file:

```console
version: '3'
 
services: 
  caddy:
    image: hackermuffin/caddy-duckdns-docker
    container_name: caddy
    hostname: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - DOMAIN=<domain>
      - TOKEN=<token>
      - REVERSE_PROXY=<proxy-address>
    volumes:
      - caddy_data:/data

volumes:
  caddy_data:
```

And for duckdns:
```console
version: '3'
 
services: 
  caddy:
    image: hackermuffin/caddy-duckdns-docker:duckdns
    container_name: caddy
    hostname: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - DOMAIN=<domain>
      - DUCKDNS_TOKEN=<token>
      - REVERSE_PROXY=<proxy-address>
    volumes:
      - caddy_data:/data

volumes:
  caddy_data:

## Further documentation

Full documetation for the base caddy container can be found at https://hub.docker.com/\_/caddy/.

For further documentation on the lego-deprecated module, see https://github.com/caddy-dns/lego-deprecated, or for the cloudflare module at https://github.com/caddy-dns/cloudflare.
