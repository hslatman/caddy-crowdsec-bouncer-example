FROM caddy:builder AS builder

RUN xcaddy build \
	# Use Cloudflare for ACME DNS challenge
	--with github.com/caddy-dns/cloudflare \
	# for crowdsec bouncer
    --with github.com/hslatman/caddy-crowdsec-bouncer/http

FROM caddy:latest

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
