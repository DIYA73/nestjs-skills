# bullmq-queue

BullMQ job queues, processors, retry logic, and scheduled jobs in NestJS.

## Trigger

Use this skill when asked to:
- Add a background job or queue
- Process tasks asynchronously
- Schedule recurring jobs (cron)
- Handle retries and failed jobs

## Constants

```typescript
export const EMAIL_QUEUE = 'email';
export const EMAIL_JOBS = {
  SEND_WELCOME: 'send-welcome',
  SEND_RESET: 'send-reset',
} as const;
```

## Producer Service

```typescript
import { Injectable } from '@nestjs/common';
import { InjectQueue } from '@nestjs/bull';
import type { Queue } from 'bull';
import { EMAIL_QUEUE, EMAIL_JOBS } from './email.constants';

export interface SendWelcomeJobData {
  userId: string;
  email: string;
  name: string;
}

@Injectable()
export class EmailQueue {
  constructor(@InjectQueue(EMAIL_QUEUE) private readonly queue: Queue) {}

  async sendWelcome(data: SendWelcomeJobData): Promise<void> {
    await this.queue.add(EMAIL_JOBS.SEND_WELCOME, data, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
      removeOnComplete: 100,
      removeOnFail: 50,
    });
  }
}
```

## Processor

```typescript
import { Processor, Process, OnQueueFailed } from '@nestjs/bull';
import type { Job } from 'bull';
import { Logger } from '@nestjs/common';
import { EMAIL_QUEUE, EMAIL_JOBS } from './email.constants';
import type { SendWelcomeJobData } from './email.queue';

@Processor(EMAIL_QUEUE)
export class EmailProcessor {
  private readonly logger = new Logger(EmailProcessor.name);

  @Process(EMAIL_JOBS.SEND_WELCOME)
  async handleWelcome(job: Job<SendWelcomeJobData>): Promise<void> {
    this.logger.log(`Sending welcome email to ${job.data.email}`);
    // your logic here
  }

  @OnQueueFailed()
  onFailed(job: Job, error: Error): void {
    this.logger.error(`Job ${job.id} failed: ${error.message}`);
  }
}
```

## Recurring Jobs

```typescript
await this.queue.add('notify', {}, {
  repeat: { cron: '0 9 * * *' },
  jobId: 'daily-notification',
});
```

## Module

```typescript
@Module({
  imports: [BullModule.registerQueue({ name: EMAIL_QUEUE })],
  providers: [EmailQueue, EmailProcessor],
  exports: [EmailQueue],
})
export class EmailModule {}
```

## Job Options Reference

```typescript
{
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 },
  delay: 5000,
  priority: 1,
  timeout: 30000,
  removeOnComplete: 100,
  removeOnFail: 50,
  jobId: 'unique-id',
}
```
