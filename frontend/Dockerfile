FROM node:20-bullseye-slim

RUN apt-get update \
 && apt-get install -y nginx dumb-init gettext-base curl \
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
ENV APP_PORT=3001

RUN mkdir -p /usr/local/bin \
 && cat > /usr/local/bin/start.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail
echo "ENV: PORT=${PORT:-unset} APP_PORT=${APP_PORT:-unset}"
echo "→ /app files:"; ls -la /app | head -n 200
echo "→ .next structure:"; ls -la .next || true; ls -la .next/standalone || true; ls -la .next/static || true

echo "→ write nginx conf from template"
envsubst '$PORT $APP_PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf
echo "----- /etc/nginx/conf.d/default.conf -----"
sed -n '1,160p' /etc/nginx/conf.d/default.conf

echo "→ nginx -t"; nginx -t
echo "→ node --version"; node --version

echo "→ start Next on APP_PORT=$APP_PORT"
HOST=0.0.0.0 PORT=${APP_PORT:-3001} node server.js &

sleep 1
echo "→ check Next upstream (http://127.0.0.1:${APP_PORT:-3001}/)"
curl -I --max-time 2 http://127.0.0.1:${APP_PORT:-3001}/ || true

echo "→ start nginx (listen on PORT=$PORT)"
exec nginx -g 'daemon off;'
SH

# 실행권한 부여 (heredoc과 별도 RUN)
RUN chmod +x /usr/local/bin/start.sh

ENTRYPOINT ["/usr/bin/dumb-init","--"]
CMD ["/usr/local/bin/start.sh"]
