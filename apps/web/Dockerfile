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
COPY .npmrc pnpm-workspace.yaml package.json pnpm-lock.yaml* ./
COPY apps/web/package.json apps/web/package.json
COPY packages/ui/package.json packages/ui/package.json
RUN pnpm config get registry \
    && pnpm install --frozen-lockfile --filter @hemayatgar/web...

FROM base AS build
COPY --from=deps /app /app
COPY . .
RUN pnpm --filter @hemayatgar/web build

FROM base AS runner
ENV NODE_ENV=production
COPY --from=build /app /app
EXPOSE 3000
CMD ["pnpm", "--filter", "@hemayatgar/web", "start"]
