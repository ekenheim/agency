# https://sko.ai/blog/how-to-actually-build-hugo-containers/
FROM --platform=$BUILDPLATFORM docker.io/library/alpine:3.19@sha256:e5d0aea7f7d2954678a9a6269ca2d06e06591881161961ea59e974dff3f12377 as build
RUN \
    apk add --no-cache \
      git \
      hugo \
      tzdata \
 && apk add --no-cache dart-sass --repository=http://dl-cdn.alpinelinux.org/alpine/edge/testing/
WORKDIR /src
COPY . .
RUN --mount=type=cache,target=/tmp/hugo_cache \
    hugo --minify

FROM docker.io/library/nginx:1.26@sha256:41b194461e4bae16f9b25d68b0976ed4735b89ca625c89aad88e1c1c3b7e8860
# implement changes required to run NGINX as an less-privileged user
RUN \
    sed -i 's,/var/run/nginx.pid,/tmp/nginx.pid,' /etc/nginx/nginx.conf \
 && sed -i "/^http {/a \    proxy_temp_path /tmp/proxy_temp;\n    client_body_temp_path /tmp/client_temp;\n    fastcgi_temp_path /tmp/fastcgi_temp;\n    uwsgi_temp_path /tmp/uwsgi_temp;\n    scgi_temp_path /tmp/scgi_temp;\n" /etc/nginx/nginx.conf \
 # nginx user must own the cache and etc directory to write cache and tweak the nginx config
 && chown -R 101:0 /var/cache/nginx \
 && chmod -R g+w /var/cache/nginx \
 && chown -R 101:0 /etc/nginx \
 && chmod -R g+w /etc/nginx

COPY --chown=nginx:nginx --from=build /src/public /site
COPY --chown=nginx:nginx ./docker/default.conf /etc/nginx/conf.d/

USER nginx
