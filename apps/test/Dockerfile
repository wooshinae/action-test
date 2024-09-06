FROM node:20-alpine AS base

FROM base AS builder
RUN apk update
RUN apk add --no-cache libc6-compat

WORKDIR /app
RUN yarn global add turbo
COPY . .
RUN turbo prune test --docker

FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
RUN yarn install
    
COPY --from=builder /app/out/full/ .
RUN yarn turbo run build --filter=test...

FROM base AS runner
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/test/next.config.mjs .
COPY --from=installer /app/apps/test/package.json .

COPY --from=installer --chown=nextjs:nodejs /app/apps/test/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/test/.next/static ./apps/test/.next/static

CMD ["node", "./apps/test/server.js"]