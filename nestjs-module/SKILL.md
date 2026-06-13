# nestjs-module

Scaffold any NestJS module with correct structure, imports, and wiring.

## Trigger

Use this skill when asked to:
- Create a new NestJS module, service, controller, or feature
- Scaffold a resource (CRUD module)
- Wire a module into AppModule

## Rules

- Always create: `feature.module.ts`, `feature.service.ts`, `feature.controller.ts`
- DTOs go in `dto/` subfolder: `create-feature.dto.ts`, `update-feature.dto.ts`
- Entity goes in same folder: `feature.entity.ts`
- Export service if other modules need it
- Use `@Injectable()`, `@Controller()`, `@Module()` decorators correctly
- Always add `ValidationPipe` compatible class-validator decorators to DTOs
- Never use `any` — use explicit types everywhere
- Use `async/await`, never `.then()`

## Module Template

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { FeatureEntity } from './feature.entity';
import { FeatureService } from './feature.service';
import { FeatureController } from './feature.controller';

@Module({
  imports: [TypeOrmModule.forFeature([FeatureEntity])],
  providers: [FeatureService],
  controllers: [FeatureController],
  exports: [FeatureService],
})
export class FeatureModule {}
```

## Service Template

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { FeatureEntity } from './feature.entity';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';

@Injectable()
export class FeatureService {
  constructor(
    @InjectRepository(FeatureEntity)
    private readonly repo: Repository<FeatureEntity>,
  ) {}

  async create(dto: CreateFeatureDto): Promise<FeatureEntity> {
    return this.repo.save(this.repo.create(dto));
  }

  async findAll(): Promise<FeatureEntity[]> {
    return this.repo.find({ order: { createdAt: 'DESC' } });
  }

  async findOne(id: string): Promise<FeatureEntity> {
    const entity = await this.repo.findOne({ where: { id } });
    if (!entity) throw new NotFoundException(`${id} not found`);
    return entity;
  }

  async update(id: string, dto: UpdateFeatureDto): Promise<FeatureEntity> {
    const entity = await this.findOne(id);
    Object.assign(entity, dto);
    return this.repo.save(entity);
  }

  async remove(id: string): Promise<void> {
    const entity = await this.findOne(id);
    await this.repo.remove(entity);
  }
}
```

## Controller Template

```typescript
import {
  Controller, Get, Post, Patch, Delete,
  Param, Body, HttpCode, HttpStatus,
} from '@nestjs/common';
import { FeatureService } from './feature.service';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';

@Controller('features')
export class FeatureController {
  constructor(private readonly service: FeatureService) {}

  @Post()
  create(@Body() dto: CreateFeatureDto) {
    return this.service.create(dto);
  }

  @Get()
  findAll() {
    return this.service.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.service.findOne(id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() dto: UpdateFeatureDto) {
    return this.service.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string) {
    return this.service.remove(id);
  }
}
```

## Register in AppModule

```typescript
import { FeatureModule } from './feature/feature.module';

@Module({
  imports: [FeatureModule],
})
export class AppModule {}
```
