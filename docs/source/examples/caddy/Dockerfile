FROM caddy:builder-alpine AS builder

# see https://caddyserver.com/docs/modules/http.ip_sources.cloudflare
RUN xcaddy build \
    --with github.com/caddyserver/transform-encoder \
    --with github.com/caddy-dns/cloudflare \
    --with github.com/WeidiDeng/caddy-cloudflare-ip

FROM caddy:alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

