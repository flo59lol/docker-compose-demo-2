FROM node:22-alpine AS builder

WORKDIR /app

# RUN apk add --no-cache openssl

COPY package*.json ./
RUN npm ci
COPY . .

# COPY prisma ./prisma
# COPY prisma.config.ts tsconfig.json ./
# COPY src ./src

RUN DATABASE_URL="mysql://task_manager:build_only@mysql:3306/task_manager" npm run build

FROM node:22-alpine AS production

WORKDIR /app

# RUN apk add --no-cache openssl

COPY package*.json ./

RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/prisma.config.ts ./

EXPOSE 3000

CMD ["node", "dist/server.js"]
