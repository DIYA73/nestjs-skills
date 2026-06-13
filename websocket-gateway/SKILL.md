# websocket-gateway

Socket.io WebSocket gateways in NestJS — real-time events, rooms, Redis pub/sub streaming.

## Trigger

Use this skill when asked to:
- Add real-time features (live updates, notifications, streaming)
- Create a WebSocket gateway
- Broadcast events to connected clients
- Stream data from Redis pub/sub to clients

## Basic Gateway

```typescript
import {
  WebSocketGateway, WebSocketServer, OnGatewayInit,
  OnGatewayConnection, OnGatewayDisconnect,
  SubscribeMessage, MessageBody, ConnectedSocket,
} from '@nestjs/websockets';
import type { Server, Socket } from 'socket.io';
import { Logger } from '@nestjs/common';

@WebSocketGateway({ cors: { origin: '*' }, namespace: '/events' })
export class EventsGateway implements OnGatewayInit, OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  private server!: Server;

  private readonly logger = new Logger(EventsGateway.name);

  afterInit(): void { this.logger.log('Gateway initialized'); }
  handleConnection(client: Socket): void { this.logger.log(`Connected: ${client.id}`); }
  handleDisconnect(client: Socket): void { this.logger.log(`Disconnected: ${client.id}`); }

  @SubscribeMessage('join-room')
  handleJoinRoom(@MessageBody() data: { roomId: string }, @ConnectedSocket() client: Socket) {
    void client.join(data.roomId);
    return { event: 'joined', data: data.roomId };
  }

  broadcastToRoom(roomId: string, event: string, payload: unknown): void {
    this.server.to(roomId).emit(event, payload);
  }

  broadcast(event: string, payload: unknown): void {
    this.server.emit(event, payload);
  }
}
```

## Redis Pub/Sub Streaming

```typescript
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import type { Server } from 'socket.io';
import { InjectRedis } from '@nestjs-modules/ioredis';
import type { Redis } from 'ioredis';
import { Logger, OnModuleInit } from '@nestjs/common';

export const STREAM_CHANNEL = 'app:stream';

@WebSocketGateway({ cors: { origin: '*' }, namespace: '/stream' })
export class StreamGateway implements OnModuleInit {
  @WebSocketServer()
  private server!: Server;

  private readonly logger = new Logger(StreamGateway.name);
  private subscriber!: Redis;

  constructor(@InjectRedis() private readonly redis: Redis) {}

  onModuleInit(): void {
    this.subscriber = this.redis.duplicate();
    this.subscriber.subscribe(STREAM_CHANNEL, (err) => {
      if (err) this.logger.error('Redis subscribe error', err);
    });
    this.subscriber.on('message', (_ch: string, msg: string) => {
      this.server.emit('event', JSON.parse(msg) as unknown);
    });
  }
}

// Publish from any service:
await this.redis.publish(STREAM_CHANNEL, JSON.stringify(payload));
```

## Rules

- Always use a namespace — never default namespace
- Duplicate Redis connection for subscriber mode
- Use `void client.join(room)` to suppress floating promise warnings
- Export the gateway if other services call broadcast methods
