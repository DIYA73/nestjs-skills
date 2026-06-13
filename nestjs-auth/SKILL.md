# nestjs-auth

JWT authentication with guards, decorators, and role-based access in NestJS.

## Trigger

Use this skill when asked to:
- Add authentication to a NestJS app
- Protect routes with JWT
- Add role-based access control
- Get the current user in a controller

## Setup

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcryptjs
npm install -D @types/passport-jwt @types/bcryptjs
```

## JWT Payload

```typescript
export interface JwtPayload {
  sub: string;
  email: string;
  role: string;
}
```

## Auth Service

```typescript
@Injectable()
export class AuthService {
  constructor(
    @InjectRepository(User) private readonly users: Repository<User>,
    private readonly jwt: JwtService,
  ) {}

  async register(email: string, password: string): Promise<{ token: string }> {
    const existing = await this.users.findOne({ where: { email } });
    if (existing) throw new ConflictException('Email already in use');
    const hash = await bcrypt.hash(password, 12);
    const user = await this.users.save(this.users.create({ email, password: hash }));
    return { token: this.sign(user) };
  }

  async login(email: string, password: string): Promise<{ token: string }> {
    const user = await this.users.findOne({
      where: { email },
      select: ['id', 'email', 'password', 'role'],
    });
    if (!user || !(await bcrypt.compare(password, user.password))) {
      throw new UnauthorizedException('Invalid credentials');
    }
    return { token: this.sign(user) };
  }

  private sign(user: User): string {
    const payload: JwtPayload = { sub: user.id, email: user.email, role: user.role };
    return this.jwt.sign(payload);
  }
}
```

## JWT Strategy

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService, @InjectRepository(User) private readonly users: Repository<User>) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: config.getOrThrow<string>('JWT_SECRET'),
    });
  }

  async validate(payload: JwtPayload): Promise<User> {
    const user = await this.users.findOne({ where: { id: payload.sub } });
    if (!user) throw new UnauthorizedException();
    return user;
  }
}
```

## Guards

```typescript
// jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

// roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(ctx: ExecutionContext): boolean {
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [ctx.getHandler(), ctx.getClass()]);
    if (!roles?.length) return true;
    const { user } = ctx.switchToHttp().getRequest<{ user: User }>();
    return roles.includes(user.role);
  }
}
```

## Decorators

```typescript
// current-user.decorator.ts
export const CurrentUser = createParamDecorator(
  (_data: unknown, ctx: ExecutionContext): User =>
    ctx.switchToHttp().getRequest<{ user: User }>().user,
);

// roles.decorator.ts
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

## Controller Usage

```typescript
@Get('me')
@UseGuards(JwtAuthGuard)
me(@CurrentUser() user: User) { return user; }

@Delete(':id')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin')
delete(@Param('id') id: string) { return this.service.remove(id); }
```
