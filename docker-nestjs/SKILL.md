---
name: docker-nestjs
description: "Production Dockerfile and docker-compose for NestJS + PostgreSQL + Redis."
---

# docker-nestjs

Production Dockerfile and docker-compose for NestJS + PostgreSQL + Redis.

## Trigger

Use this skill when asked to:
- Dockerize a NestJS app
- Create a docker-compose for local dev
- Set up production containers
- Add health checks

## Dockerfile

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## docker-compose.yml

```yaml
version: '3.9'

services:
  app:
    build: .
    ports:
      - '3000:3000'
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: ${DB_NAME}
    ports:
      - '5432:5432'
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${DB_USER}']
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'
    volumes:
      - redisdata:/data
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

volumes:
  pgdata:
  redisdata:
```

## .dockerignore
node_modules

dist

.env

*.log

coverage

.git
## Rules

- Never `synchronize: true` in production
- Never commit `.env`
- Use multi-stage build
- Pin image versions
- Use `service_healthy` depends_on
