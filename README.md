# nestjs-skills

A focused collection of Claude Code skills for NestJS backend development.

Drop into `~/.claude/skills/` and Claude Code auto-discovers them.

## Skills

| Skill | Description |
|-------|-------------|
| `nestjs-module` | Scaffold modules, services, controllers, DTOs |
| `typeorm-patterns` | Entities, relations, migrations, QueryBuilder |
| `bullmq-queue` | Job queues, processors, retry logic, cron jobs |
| `websocket-gateway` | Socket.io gateways, rooms, Redis pub/sub streaming |
| `nestjs-auth` | JWT auth, guards, role-based access, decorators |
| `docker-nestjs` | Production Dockerfile, docker-compose, health checks |
| `nestjs-testing` | Unit tests, e2e tests, mocking repos and queues |

## Install

```bash
git clone https://github.com/DIYA73/nestjs-skills /tmp/nestjs-skills && mkdir -p ~/.claude/skills && mv /tmp/nestjs-skills/*/ ~/.claude/skills/ && rm -rf /tmp/nestjs-skills
```

## Usage

Once installed, Claude Code picks them up automatically:
scaffold a users module with TypeORM

add a BullMQ email queue with retry logic

protect this route with JWT auth


write unit tests for UsersService
## Stack

NestJS · TypeORM · PostgreSQL · Redis · BullMQ · Socket.io · JWT

## License

MIT
