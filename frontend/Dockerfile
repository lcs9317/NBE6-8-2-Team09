FROM node:20-bullseye-slim

# Nginx + dumb-init + envsubst 설치
RUN apt-get update \
 && apt-get install -y nginx dumb-init gettext-base \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p /run/nginx

WORKDIR /app

COPY .next/standalone ./
COPY public ./public
COPY .next/static ./public/_next/static

# 템플릿 복사
COPY docker/nginx.conf.template /etc/nginx/conf.d/default.conf.template

ENV NODE_ENV=production

EXPOSE 3000  
EXPOSE 80    

ENTRYPOINT ["/usr/bin/dumb-init","--"]
CMD ["bash","-lc","envsubst '$PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf && node server.js & nginx -g 'daemon off;'"]
