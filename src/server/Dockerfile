##### DEPENDENCIES

FROM --platform=linux/amd64 node:20-alpine AS deps
RUN apk add --no-cache libc6-compat openssl
WORKDIR /app

RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:$PATH"

COPY package.json bun.lock ./
COPY src/server/package.json ./src/server/

RUN bun install --frozen-lockfile

##### BUILDER

FROM --platform=linux/amd64 node:20-alpine AS builder
ARG AUTH_SECRET
ARG AUTH_GITHUB_ID
ARG AUTH_GITHUB_SECRET
ARG DATABASE_URL
ARG GITHUB_TOKEN
ARG AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY
ARG AWS_BUCKET_NAME
ARG AWS_REGION
ARG ECS_TASK_DEFINITION_MEET
ARG ECS_TASK_DEFINITION_TEAMS
ARG ECS_TASK_DEFINITION_ZOOM
ARG ECS_CLUSTER_NAME
ARG ECS_SUBNETS
ARG ECS_SECURITY_GROUPS

WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/src/server/node_modules ./src/server/node_modules
COPY src/server/ ./src/server/

RUN cd src/server && SKIP_ENV_VALIDATION=1 npx next build

##### RUNNER

FROM --platform=linux/amd64 gcr.io/distroless/nodejs20-debian12 AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/src/server/next.config.js ./
COPY --from=builder /app/src/server/public ./public
COPY --from=builder /app/src/server/package.json ./package.json
COPY --from=builder /app/src/server/.next/standalone ./
COPY --from=builder /app/src/server/.next/static ./.next/static

EXPOSE 3000
ENV PORT=3000
CMD ["server.js"]
