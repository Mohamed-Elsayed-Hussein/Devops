# -------- Stage 1: Build & install dependencies --------
FROM  oven/bun:canary-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./

RUN bun install



# -------- Stage 2: Runtime --------
FROM oven/bun:1 AS runtime
WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules

COPY src ./src
COPY tests  ./tests
COPY public ./public

COPY package.json ./

EXPOSE 3000
CMD ["bun", "run", "start"]