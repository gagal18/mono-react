# Stage 1: Build the app and install dependencies
FROM node:18-alpine AS base

# Install dependencies for the build stage
RUN apk update && apk add --no-cache libc6-compat

# Install pnpm and turbo globally
RUN npm install -g pnpm
RUN yarn global add turbo@^2

WORKDIR /app

# Install all dependencies in the root
COPY package.json pnpm-lock.yaml ./
RUN pnpm install 

# Copy the rest of the application
COPY . .

# Prune the workspace to only include the `docs` app
RUN turbo prune --docker --scope=docs

# Install dependencies for the `docs` app in the pruned workspace
RUN pnpm install 

# Build the Next.js app in the `docs` app (should generate .next/standalone)
RUN pnpm turbo run build --filter=docs...

# Stage 2: Installer - Install pruned dependencies and finalize the build
FROM base AS installer

# Install dependencies for the "docs" app in the pruned workspace
RUN pnpm install 

# Build the project again (to ensure everything is correctly built)
RUN pnpm turbo run build --filter=docs...

# Stage 3: Runner - Run the app in production
FROM node:18-alpine AS runner

# Create a non-root user for security best practices
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

# Set the working directory
WORKDIR /app

# Copy the build artifacts from the installer stage
# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/static ./.next/static
# COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next ./apps/docs/.next
# COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/public ./apps/docs/public
# COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/node_modules ./apps/docs/node_modules
# COPY --from=installer --chown=nextjs:nodejs /app/node_modules ./node_modules

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/config/next-config-js/output

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"
CMD ["node", "apps/docs/server.js"]