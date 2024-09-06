FROM node:20-alpine AS base

FROM base AS builder
RUN apk update
RUN apk add --no-cache libc6-compat
# Set working directory
WORKDIR /app
# Replace <your-major-version> with the major version installed in your repository. For example:
# RUN yarn global add turbo@^2
RUN yarn global add turbo
COPY . .

# Generate a partial monorepo with a pruned lockfile for a target workspace.
# Assuming "bao" is the name entered in the project's package.json: { name: "bao" }
RUN turbo prune test --docker

# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app

# First install the dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
RUN yarn install
    
# Build the project
COPY --from=builder /app/out/full/ .
RUN yarn turbo run build --filter=test...

FROM base AS runner
WORKDIR /app

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/test/next.config.mjs .
COPY --from=installer /app/apps/test/package.json .

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/test/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/test/.next/static ./apps/test/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/test/public ./apps/test/public

# Use next start to run the application
CMD ["yarn", "start"]