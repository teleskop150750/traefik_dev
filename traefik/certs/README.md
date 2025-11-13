# Local certificates

Place the wildcard TLS certificate for `*.example.com` in this directory. Traefik expects the files `example.com.pem` and `example.com-key.pem` (PEM format). They can be created with `mkcert` or any other local CA tool. Keep the private key secure and do not commit it to version control.


mkcert -key-file example.com-key.pem -cert-file example.com.pem example.com "*.example.com"
