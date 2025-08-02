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

# start.sh 재작성 (heredoc)
RUN cat > /usr/local/bin/start.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

echo "ENV: PORT=${PORT:-unset} APP_PORT=${APP_PORT:-unset}"

# 1) nginx conf 생성
envsubst '$PORT $APP_PORT' < /etc/nginx/conf.d/default.conf.template > /etc/nginx/conf.d/default.conf
echo "----- /etc/nginx/conf.d/default.conf -----"
sed -n '1,120p' /etc/nginx/conf.d/default.conf

# 2) nginx 설정 테스트 (실패해도 종료하지 않음)
if ! nginx -t; then 
  echo "WARN: nginx -t failed, but continuing to start for diagnostics"
fi

# 3) Next(standalone) 먼저 백그라운드로 시작
echo "→ start Next on APP_PORT=$APP_PORT (HOSTNAME = 127.0.0.1)"
HOSTNAME=127.0.0.1 PORT=${APP_PORT:-3001} node server.js &

sleep 0.5
ss -ltnp | grep ":${APP_PORT:-3001}" || true

# 4) Next 준비 대기 (최대 20초). 실패해도 종료하지 않음
echo "→ waiting Next readiness on 127.0.0.1:${APP_PORT:-3001} ..."
i=0
until curl -sS -o /dev/null --max-time 1 "http://127.0.0.1:${APP_PORT:-3001}/"; do
  i=$((i+1))
  echo "  waiting... try ${i}/40"
  if [ "$i" -ge 40 ]; then
    echo "WARN: Next not ready yet, proceeding to start nginx anyway"
    break
  fi
  sleep 0.5
done

# 5) nginx 포그라운드 실행 (메인 프로세스)
echo "→ start nginx (listen on PORT=$PORT)"
exec nginx -g 'daemon off;'
SH
RUN chmod +x /usr/local/bin/start.sh

