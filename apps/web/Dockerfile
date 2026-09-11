FROM ghcr.io/pnpm/pnpm:11.19.0 AS pnpm-src

FROM node:24-slim AS base
COPY --from=pnpm-src /opt/pnpm/pnpm /usr/local/bin/pnpm
ARG NPM_REGISTRY=https://registry.npmmirror.com
ENV npm_config_registry=${NPM_REGISTRY}
WORKDIR /app

FROM base AS deps
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml* .npmrc ./
COPY apps/web/package.json apps/web/package.json
COPY packages/ui/package.json packages/ui/package.json
RUN pnpm install --frozen-lockfile --filter @hemayatgar/web...

FROM base AS build
COPY --from=deps /app /app
COPY . .
RUN pnpm --filter @hemayatgar/web build

FROM base AS runner
ENV NODE_ENV=production
COPY --from=build /app /app
EXPOSE 3000
CMD ["pnpm", "--filter", "@hemayatgar/web", "start"]
