---
name: nestjs-testing
description: "Unit and e2e testing patterns for NestJS services, controllers, and gateways. Use when writing unit tests for a NestJS service or controller, writing e2e tests for API endpoints, or mocking TypeORM repositories or external services."
---

# nestjs-testing

Unit and e2e testing patterns for NestJS services, controllers, and gateways.

## Trigger

Use this skill when asked to:
- Write unit tests for a NestJS service or controller
- Write e2e tests for API endpoints
- Mock TypeORM repositories or external services

## Unit Test — Service

```typescript
const mockRepo = {
  create: jest.fn(), save: jest.fn(), find: jest.fn(),
  findOne: jest.fn(), remove: jest.fn(), update: jest.fn(),
};

describe('UsersService', () => {
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepo },
      ],
    }).compile();
    service = module.get<UsersService>(UsersService);
    jest.clearAllMocks();
  });

  it('returns user when found', async () => {
    const user = { id: '1', email: 'test@test.com' } as User;
    mockRepo.findOne.mockResolvedValue(user);
    expect(await service.findOne('1')).toEqual(user);
  });

  it('throws NotFoundException when not found', async () => {
    mockRepo.findOne.mockResolvedValue(null);
    await expect(service.findOne('x')).rejects.toThrow(NotFoundException);
  });
});
```

## Unit Test — Controller

```typescript
const mockService = { findAll: jest.fn(), findOne: jest.fn(), create: jest.fn() };

describe('UsersController', () => {
  let controller: UsersController;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [{ provide: UsersService, useValue: mockService }],
    }).compile();
    controller = module.get<UsersController>(UsersController);
    jest.clearAllMocks();
  });

  it('findAll returns array', async () => {
    mockService.findAll.mockResolvedValue([{ id: '1' }]);
    expect(await controller.findAll()).toHaveLength(1);
  });
});
```

## E2E Test

```typescript
describe('UsersController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({ imports: [AppModule] }).compile();
    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    await app.init();
  });

  afterAll(async () => { await app.close(); });

  it('GET /users returns 401 without token', () => {
    return request(app.getHttpServer()).get('/api/v1/users').expect(401);
  });
});
```

## Mock Patterns

```typescript
// Redis
const mockRedis = {
  get: jest.fn().mockResolvedValue(null),
  set: jest.fn().mockResolvedValue('OK'),
  del: jest.fn().mockResolvedValue(1),
};

// Bull Queue
const mockQueue = { add: jest.fn().mockResolvedValue({ id: 'job-1' }) };
{ provide: getQueueToken('my-queue'), useValue: mockQueue }
```

## Rules

- Always `jest.clearAllMocks()` in `beforeEach`
- Unit tests: mock ALL external dependencies
- E2E tests: use real DB with `.env.test`
- Test behavior, not implementation
- Naming: `it('returns 404 when user not found', ...)`
