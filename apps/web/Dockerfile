FROM node:24-slim AS base
RUN corepack enable
WORKDIR /app

# -----------------------------
# deps stage
# -----------------------------
FROM base AS deps

# کپی فایل‌های اصلی workspace
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./

# کپی package.json های مربوط به web و ui
COPY apps/web/package.json ./apps/web/package.json
COPY packages/ui/package.json ./packages/ui/package.json

# نصب فقط وابستگی‌های web
RUN pnpm install --frozen-lockfile --filter @hemayatgar/web...

# -----------------------------
# build stage
# -----------------------------
FROM base AS build

COPY --from=deps /app /app
COPY . .

# build پروژه web
RUN pnpm --filter @hemayatgar/web build

# -----------------------------
# runner stage
# -----------------------------
FROM base AS runner
ENV NODE_ENV=production

WORKDIR /app

COPY --from=build /app /app

EXPOSE 3000
CMD ["pnpm", "--filter", "@hemayatgar/web", "start"]
