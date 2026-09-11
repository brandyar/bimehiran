FROM ghcr.io/pnpm/pnpm:11.19.0 AS pnpm-src

FROM node:24-slim AS base
RUN apt-get update \
    && apt-get install -y --no-install-recommends libatomic1 \
    && rm -rf /var/lib/apt/lists/*
COPY --from=pnpm-src /opt/pnpm/ /usr/local/bin/
ENV npm_config_registry=https://registry.npmmirror.com
ENV npm_config_fetch_retries=5
ENV npm_config_fetch_retry_mintimeout=20000
ENV npm_config_fetch_retry_maxtimeout=120000
WORKDIR /app

FROM base AS deps
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml* ./
COPY apps/api/package.json apps/api/package.json
COPY packages/database/package.json packages/database/package.json
RUN pnpm install --frozen-lockfile --filter @hemayatgar/api...

FROM base AS build
COPY --from=deps /app /app
COPY . .
RUN pnpm --filter @hemayatgar/database prisma:generate
RUN pnpm --filter @hemayatgar/api build

FROM base AS runner
ENV NODE_ENV=production
COPY --from=build /app /app
EXPOSE 4000
CMD ["node", "apps/api/dist/index.js"]
