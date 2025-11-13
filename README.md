# Traefik dev stack

This stack runs a local Traefik reverse proxy with HTTPS termination for subdomains of `example.com`. All project containers share the external Docker network `traefik` and expose themselves to Traefik through labels.

## Prerequisites
- Docker and Docker Compose plugin.
- [`mkcert`](https://github.com/FiloSottile/mkcert) (or any other tool capable of issuing a trusted wildcard certificate for `example.com`).

Create the shared network once:

```fish
docker network create traefik
```

## Generate the wildcard certificate
1. Install and initialise `mkcert` (`mkcert -install`).
2. Generate the certificate files: `mkcert "*.example.com" example.com`.
3. Move the generated `*.pem` and `*-key.pem` files to `traefik/certs/` and rename them to `example.com.pem` and `example.com-key.pem`.
4. Ensure the private key remains untracked (`traefik/certs` is gitignored by default).

## Start Traefik

```fish
docker compose up -d --build
```

Traefik listens on `https://traefik.example.com` (add the hostname to `/etc/hosts`). The dashboard is exposed without authentication in local mode.

## Add a new project
1. Add a hosts entry for the project, e.g. `127.0.0.1 api.example.com`.
2. Attach the service to the shared network and add Traefik labels:

```yaml
services:
	api:
		image: nginx:alpine
		labels:
			- "traefik.enable=true"
			- "traefik.http.routers.api.rule=Host(`api.example.com`)"
			- "traefik.http.routers.api.entrypoints=websecure"
			- "traefik.http.routers.api.tls=true"
			- "traefik.http.services.api.loadbalancer.server.port=80"
		networks:
			- traefik

networks:
	traefik:
		external: true
```

3. Recreate the project container (`docker compose up -d`).
4. Open `https://api.example.com` in the browser; the certificate should show as trusted (if the mkcert root is installed).

Repeat the network labels for each additional service that should be exposed.

```sh
$ mkcert -install
Created a new local CA 💥
The local CA is now installed in the system trust store! ⚡️
The local CA is now installed in the Firefox trust store (requires browser restart)! 🦊

$ mkcert example.com "*.example.com" example.test localhost 127.0.0.1 ::1

Created a new certificate valid for the following names 📜
 - "example.com"
 - "*.example.com"
 - "example.test"
 - "localhost"
 - "127.0.0.1"
 - "::1"

The certificate is at "./example.com+5.pem" and the key at "./example.com+5-key.pem" ✅
```
mkcert -cert-file traefik/certs/example.com.pem -key-file traefik/certs/example.com-key.pem '*.example.com' example.com
