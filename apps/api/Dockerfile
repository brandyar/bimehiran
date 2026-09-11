FROM ghcr.io/pnpm/pnpm:11.19.0 AS pnpm-src

FROM node:24-slim AS base
RUN apt-get update \
    && apt-get install -y --no-install-recommends libatomic1 openssl \
    && rm -rf /var/lib/apt/lists/*
COPY --from=pnpm-src /opt/pnpm/ /usr/local/bin/
ENV NODE_OPTIONS=--dns-result-order=ipv4first
RUN printf '%s\n' 'precedence ::ffff:0:0/96  100' >> /etc/gai.conf
WORKDIR /app

FROM base AS deps
COPY .npmrc pnpm-workspace.yaml package.json pnpm-lock.yaml* ./
COPY apps/api/package.json apps/api/package.json
COPY packages/database/package.json packages/database/package.json
RUN pnpm config get registry \
    && pnpm install --frozen-lockfile --ignore-scripts --filter @hemayatgar/api... \
    && pnpm approve-builds --all \
    && pnpm rebuild --pending

FROM base AS build
COPY --from=deps /app /app
COPY . .
RUN attempts=0; \
    until cd packages/database && ./node_modules/.bin/prisma generate; do \
      attempts=$((attempts + 1)); \
      if [ "$attempts" -ge 5 ]; then exit 1; fi; \
      echo "Prisma engine download failed; retrying ($attempts/5)"; \
      cd /app; \
      sleep 10; \
    done
RUN cd apps/api && ./node_modules/.bin/tsc -p tsconfig.json

FROM base AS runner
ENV NODE_ENV=production
COPY --from=build /app /app
EXPOSE 4000
CMD ["node", "apps/api/dist/index.js"]
