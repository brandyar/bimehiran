FROM node:24-slim AS base
ARG NPM_REGISTRY=https://registry.npmmirror.com
ENV npm_config_registry=${NPM_REGISTRY}
RUN npm install -g pnpm@11.19.0
WORKDIR /app

FROM base AS deps
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml* ./
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
