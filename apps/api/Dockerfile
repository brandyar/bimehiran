FROM ghcr.io/pnpm/pnpm:11.19.0 AS pnpm-src

FROM node:24-slim AS base
RUN apt-get update \
    && apt-get install -y --no-install-recommends libatomic1 \
    && rm -rf /var/lib/apt/lists/*
COPY --from=pnpm-src /opt/pnpm/ /usr/local/bin/
ENV NODE_OPTIONS=--dns-result-order=ipv4first
WORKDIR /app

FROM base AS deps
COPY .npmrc pnpm-workspace.yaml package.json pnpm-lock.yaml* ./
COPY apps/api/package.json apps/api/package.json
COPY packages/database/package.json packages/database/package.json
RUN pnpm config get registry \
    && pnpm install --frozen-lockfile --filter @hemayatgar/api...

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
