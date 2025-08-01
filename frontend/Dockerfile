FROM node:20-bullseye-slim

RUN apt-get update \
 && apt-get install -y nginx dumb-init gettext-base \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p /run/nginx

WORKDIR /app

COPY .next/standalone ./
COPY .next ./.next
COPY public ./public

COPY docker/nginx.conf.template /etc/nginx/conf.d/default.conf.template

ENV NODE_ENV=production

EXPOSE 3000
EXPOSE 80

ENTRYPOINT ["/usr/bin/dumb-init","--"]
CMD ["bash","-lc","envsubst '$PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf && PORT=3001 node server.js & nginx -g 'daemon off;'"]
