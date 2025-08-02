FROM node:20-bullseye-slim

RUN apt-get update \
 && apt-get install -y nginx dumb-init gettext-base \
 && rm -rf /var/lib/apt/lists/* \
 && rm -f /etc/nginx/sites-enabled/default \
 && rm -f /etc/nginx/conf.d/default.conf \
 && mkdir -p /run/nginx

WORKDIR /app

COPY .next/standalone ./
COPY .next ./.next
COPY public ./public



COPY docker/nginx.conf.template /etc/nginx/conf.d/default.conf.template

ENV NODE_ENV=production
ENV LISTEN_PORT=80
ENV APP_PORT=3001

EXPOSE 80

ENTRYPOINT ["/usr/bin/dumb-init","--"]
// CMD ["bash","-lc","envsubst '$LISTEN_PORT $APP_PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf && HOST=0.0.0.0 PORT=$APP_PORT node server.js & nginx -g 'daemon off;'"]
CMD ["bash","-lc","HOST=0.0.0.0 PORT=${PORT:-3000} node server.js"]
