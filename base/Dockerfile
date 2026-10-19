# ghcr.io/visiarch-sites/static-site-base: Nginx set up to serve one static site.
# Each site's image is this plus its build output (see .github/workflows/deploy.yml).
FROM nginx:1.29-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
RUN rm -f /usr/share/nginx/html/*
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1/healthz || exit 1
EXPOSE 80
