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

RUN cat > /usr/local/bin/start.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

echo "ENV: PORT=${PORT:-unset} APP_PORT=${APP_PORT:-unset}"

# 1) nginx conf 생성
envsubst '$PORT $APP_PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf
echo "----- /etc/nginx/conf.d/default.conf -----"
sed -n '1,120p' /etc/nginx/conf.d/default.conf
nginx -t

# 2) Next(standalone) 먼저 백그라운드로 시작
echo "→ start Next on APP_PORT=$APP_PORT"
HOST=0.0.0.0 PORT=${APP_PORT:-3001} node server.js &

# 3) Next가 리슨될 때까지 대기 (최대 20초, 0.5s 간격)
for i in $(seq 1 40); do
  if curl -sS -o /dev/null --max-time 1 http://127.0.0.1:${APP_PORT:-3001}/; then
    echo "Next is up (checked at try ${i})"
    break
  fi
  echo "Waiting for Next... (${i}/40)"
  sleep 0.5
done

# 최종 확인 (여전히 실패면 종료)
if ! curl -sS -o /dev/null --max-time 1 http://127.0.0.1:${APP_PORT:-3001}/; then
  echo "ERROR: Next never became ready on 127.0.0.1:${APP_PORT:-3001}"
  exit 1
fi

# 4) nginx 실행
echo "→ start nginx (listen on PORT=$PORT)"
exec nginx -g 'daemon off;'
SH

RUN chmod +x /usr/local/bin/start.sh
