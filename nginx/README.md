# Reverse proxy integration point

No application upstreams, public hostname, media delivery policy, or TLS termination location are defined. Do not publish an Nginx configuration with guessed routes. The Architect must decide edge ownership and the Builder must document API/web upstream ports and health paths before proxy configuration is added. Production must terminate TLS with managed certificates, enforce security headers and body/time limits, and avoid caching authenticated responses.
