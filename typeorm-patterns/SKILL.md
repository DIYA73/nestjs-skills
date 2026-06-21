---
name: typeorm-patterns
description: "TypeORM entity design, relations, migrations, and query patterns for NestJS + PostgreSQL."
---

# typeorm-patterns

TypeORM entity design, relations, migrations, and query patterns for NestJS + PostgreSQL.

## Trigger

Use this skill when asked to:
- Create a TypeORM entity
- Define relations (OneToMany, ManyToOne, ManyToMany)
- Write a TypeORM query (find, findOne, QueryBuilder)
- Create or run a migration

## Rules

- Always use `@PrimaryGeneratedColumn('uuid')` — never auto-increment int
- Always add `@CreateDateColumn()` and `@UpdateDateColumn()`
- Use `!` (definite assignment) on all columns
- Nullable columns: `@Column({ nullable: true, type: 'varchar' })` + `field!: string | null`
- Never use `any` in entities

## Base Entity

```typescript
import { PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

export abstract class BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id!: string;

  @CreateDateColumn()
  createdAt!: Date;

  @UpdateDateColumn()
  updatedAt!: Date;
}
```

## Entity Template

```typescript
import { Entity, Column, ManyToOne, JoinColumn } from 'typeorm';
import { BaseEntity } from '../common/base.entity';
import { User } from '../users/user.entity';

export const PostStatus = {
  DRAFT: 'draft',
  PUBLISHED: 'published',
} as const;
export type PostStatus = typeof PostStatus[keyof typeof PostStatus];

@Entity('posts')
export class Post extends BaseEntity {
  @Column()
  title!: string;

  @Column({ type: 'text' })
  body!: string;

  @Column({ type: 'varchar', default: PostStatus.DRAFT })
  status!: PostStatus;

  @Column({ nullable: true, type: 'varchar' })
  slug!: string | null;

  @ManyToOne(() => User, (user) => user.posts, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'userId' })
  user!: User;

  @Column()
  userId!: string;
}
```

## Relations

```typescript
// OneToMany
@OneToMany(() => Post, (post) => post.user)
posts!: Post[];

// ManyToOne
@ManyToOne(() => User, (user) => user.posts, { onDelete: 'CASCADE' })
@JoinColumn({ name: 'userId' })
user!: User;

// ManyToMany
@ManyToMany(() => Tag, (tag) => tag.posts)
@JoinTable({ name: 'post_tags' })
tags!: Tag[];
```

## Query Patterns

```typescript
// Find with relations
await this.repo.findOne({ where: { id }, relations: { user: true } });

// QueryBuilder
const results = await this.repo
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.user', 'user')
  .where('post.status = :status', { status: PostStatus.PUBLISHED })
  .orderBy('post.createdAt', 'DESC')
  .take(20)
  .getMany();
```

## Indexes

```typescript
@Entity('posts')
@Index(['userId', 'status'])
@Index(['slug'], { unique: true })
export class Post extends BaseEntity { }
```
