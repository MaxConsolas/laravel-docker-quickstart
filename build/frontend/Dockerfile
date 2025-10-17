# Multi-stage build for optimization
FROM node:20.9-alpine AS base

# Install only necessary packages
RUN apk add --no-cache dumb-init && \
    addgroup -g 1001 nodejs && \
    adduser -D nextjs -u 1001 -G nodejs

WORKDIR /app

# Stage 1: Dependencies
FROM base AS deps

COPY frontend/package*.json frontend/yarn.lock ./
RUN yarn install --frozen-lockfile --production --no-audit --no-fund && \
    yarn cache clean

# Stage 2: Build
FROM base AS builder

COPY frontend/package*.json frontend/yarn.lock ./
RUN yarn install --frozen-lockfile --no-audit --no-fund

COPY frontend ./

ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_HOST
ARG SENTRY_AUTH_TOKEN
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_HOST=$NEXT_PUBLIC_HOST
ENV SENTRY_AUTH_TOKEN=$SENTRY_AUTH_TOKEN

RUN yarn build && \
    yarn cache clean

# Stage 3: Production
FROM base AS runner

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Create user
RUN addgroup -g 1001 nodejs || true && \
    adduser -D nextjs -u 1001 -G nodejs || true

# Copy only necessary files
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules

USER nextjs

EXPOSE 3000

ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["dumb-init", "yarn", "start"]
