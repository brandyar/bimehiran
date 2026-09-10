FROM node:24-slim AS base
RUN corepack enable
WORKDIR /app

# -----------------------------
# deps stage
# -----------------------------
FROM base AS deps

# کپی فایل‌های اصلی workspace
COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./

# کپی package.json های مربوط به api و database
COPY apps/api/package.json ./apps/api/package.json
COPY packages/database/package.json ./packages/database/package.json

# نصب فقط وابستگی‌های api
RUN pnpm install --frozen-lockfile --filter @hemayatgar/api...

# -----------------------------
# build stage
# -----------------------------
FROM base AS build

COPY --from=deps /app /app
COPY . .

# prisma generate برای پکیج database
RUN pnpm --filter @hemayatgar/database prisma:generate

# build api
RUN pnpm --filter @hemayatgar/api build

# -----------------------------
# runner stage
# -----------------------------
FROM base AS runner
ENV NODE_ENV=production

WORKDIR /app

COPY --from=build /app /app

EXPOSE 4000
CMD ["node", "apps/api/dist/index.js"]
