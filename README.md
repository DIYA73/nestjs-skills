# 🛠️ nestjs-skills

8 Claude Code skills for NestJS backend development — drop them in and Claude Code auto-discovers them.

![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat&logo=nestjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)
![Claude Code](https://img.shields.io/badge/Claude%20Code-skills-orange?style=flat)

---

## Install

```bash
git clone https://github.com/DIYA73/nestjs-skills.git ~/.claude/skills/nestjs-skills
```

Claude Code picks up all skills automatically on next start.

## Skills

| Skill | Trigger phrases | What it generates |
|-------|----------------|-------------------|
| `nestjs-module` | "scaffold module", "create service" | Module, service, controller, DTOs with validation |
| `typeorm-patterns` | "create entity", "add migration" | Entities, relations, migrations, QueryBuilder examples |
| `bullmq-queue` | "add queue", "create job processor" | Queue module, processor, retry config, cron jobs |
| `websocket-gateway` | "add websocket", "create gateway" | Socket.io gateway, rooms, Redis pub/sub streaming |
| `nestjs-auth` | "add JWT auth", "protect route" | JWT strategy, guards, decorators, role-based access |
| `docker-nestjs` | "dockerize", "add docker" | Dockerfile (multi-stage), docker-compose, env handling |
| `nestjs-testing` | "write tests", "add unit test" | Jest unit tests, e2e tests, mock factories |
| `typeorm-patterns` | "optimize query", "add index" | QueryBuilder patterns, N+1 fixes, relation loading |

## Usage examples

In Claude Code, just describe what you need naturally:

```
> scaffold a users module with email and password fields
> add a BullMQ queue for sending emails with retry on failure
> protect this endpoint with JWT and require the admin role
> write unit tests for the UsersService
> dockerize this NestJS app for production
```

Claude Code reads the relevant skill file and generates production-quality code that matches your existing project structure.

## What's inside each skill

Each skill file (`SKILL.md`) contains:
- The exact NestJS patterns to follow
- Ready-to-use code templates
- Common gotchas and how to avoid them
- Integration tips (e.g. how BullMQ connects to Redis, how TypeORM entities map to Prisma-style patterns)

## Contributing

PRs welcome — new skills, improvements to existing ones, or fixes for NestJS version changes.

## Related

- [contextpulse](https://github.com/DIYA73/contextpulse) — MCP server built with these patterns
- [agentflow](https://github.com/DIYA73/agentflow) — visual agent builder using NestJS + BullMQ

## License

MIT