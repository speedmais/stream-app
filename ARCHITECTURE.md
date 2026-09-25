# Stream App Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    STREAM APP SYSTEM                         │
└─────────────────────────────────────────────────────────────────┘

┌────────────────────┐         ┌────────────────────┐
│   Frontend   │◄───────────────┤   Backend    │
│  (Next.js)   │  REST   │  (NestJS)    │
└────────────────────┘  Socket │  TypeORM     │
     │                   └────────────────┬────────────────┘
     │                          │
     │              ┌─────────────────────────────────────┬──────────────┐
     │              │           │           │
  Video Player   Postgres    Redis      RTMP Server
  HLS/WebRTC     (DB)    (Cache/Pub)   (Nginx)
                               │          Sub)
                               │
                   ┌─────────────────────────────┬──────────────────┐
                   │                        │
              Chat Messages           Stream State
              Notifications           Live Events
```

## Module Responsibilities

### 1. Auth Module
- User registration & login
- JWT token generation & validation
- Password hashing with bcrypt
- Refresh token logic

**Files:**
```
backend/src/auth/
├── auth.controller.ts
├── auth.service.ts
├── auth.module.ts
├── jwt.strategy.ts
├── dtos/
│   ├── register.dto.ts
│   ├── login.dto.ts
│   └── login-response.dto.ts
└── guards/
    └── jwt.guard.ts
```

### 2. Users Module
- User profiles
- Follow/Subscribe system
- User statistics

**Files:**
```
backend/src/users/
├── users.controller.ts
├── users.service.ts
├── users.module.ts
├── entities/
│   └── user.entity.ts
└── dtos/
    ├── create-user.dto.ts
    └── update-user.dto.ts
```

### 3. Streams Module
- Stream CRUD operations
- Stream discovery
- Stream metadata (title, description, thumbnail)
- Start/End streaming

**Files:**
```
backend/src/streams/
├── streams.controller.ts
├── streams.service.ts
├── streams.module.ts
├── entities/
│   └── stream.entity.ts
└── dtos/
    ├── create-stream.dto.ts
    ├── update-stream.dto.ts
    └── stream.dto.ts
```

### 4. Chat Module
- WebSocket connections
- Real-time messaging
- Chat history
- Moderation (mute, ban)

**Files:**
```
backend/src/chat/
├── chat.gateway.ts
├── chat.service.ts
├── chat.module.ts
├── entities/
│   └── message.entity.ts
└── dtos/
    ├── send-message.dto.ts
    └── message.dto.ts
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  display_name VARCHAR(100),
  avatar_url VARCHAR(255),
  bio TEXT,
  followers_count INT DEFAULT 0,
  following_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Streams Table
```sql
CREATE TABLE streams (
  id SERIAL PRIMARY KEY,
  streamer_id INT NOT NULL REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  stream_key VARCHAR(50) UNIQUE NOT NULL,
  status VARCHAR(20) DEFAULT 'offline', -- 'offline', 'live', 'ended'
  thumbnail_url VARCHAR(255),
  viewers_count INT DEFAULT 0,
  start_time TIMESTAMP,
  end_time TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Messages Table
```sql
CREATE TABLE messages (
  id SERIAL PRIMARY KEY,
  stream_id INT NOT NULL REFERENCES streams(id),
  user_id INT NOT NULL REFERENCES users(id),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Follows Table
```sql
CREATE TABLE follows (
  follower_id INT NOT NULL REFERENCES users(id),
  following_id INT NOT NULL REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (follower_id, following_id)
);
```

## Data Flow

### Stream Start Flow
```
1. Streamer opens OBS
2. OBS connects to RTMP Server (Nginx)
3. Backend notifies via WebSocket: stream_started
4. Frontend updates dashboard
5. RTMP Server converts stream to HLS
6. Viewers can access via HLS URL
```

### Chat Flow
```
1. Viewer types message
2. WebSocket sends to Backend (Chat Gateway)
3. Chat Service saves to DB
4. Redis Pub/Sub broadcasts to all viewers
5. All connected clients receive message
```

### Authentication Flow
```
1. User submits login credentials
2. Backend hashes password, checks DB
3. JWT token generated
4. Token stored in browser (localStorage/cookies)
5. Token sent in Authorization header for API requests
6. JwtGuard validates token on protected routes
```

## API Response Format

```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: {
    code: string;
    message: string;
  };
  timestamp: string;
}
```

## WebSocket Events

### Chat Events
```typescript
// Client → Server
'message:send' {
  streamId: number;
  content: string;
}

// Server → Client
'message:new' {
  id: number;
  user: { id, username, avatar };
  content: string;
  createdAt: string;
}

'user:joined' {
  username: string;
  totalViewers: number;
}

'user:left' {
  username: string;
  totalViewers: number;
}
```

### Stream Events
```typescript
'stream:started' {
  id: number;
  streamer: { id, username, avatar };
  title: string;
  viewers: number;
}

'stream:ended' {
  id: number;
  duration: number;
  totalViewers: number;
}

'viewers:updated' {
  streamId: number;
  count: number;
}
```

## Scalability Considerations

### Horizontal Scaling
- **Backend:** Use load balancer (nginx/HAProxy) with multiple NestJS instances
- **Database:** PostgreSQL read replicas for reporting
- **Redis:** Redis Cluster for session management
- **WebSockets:** Use adapter (socket.io-redis) for multiple instances

### Performance Optimizations
- CDN for frontend assets
- HLS caching for video segments
- Database indexing on frequently queried fields
- Redis caching for user profiles, stream metadata
- Message compression in WebSockets

### Security
- JWT token expiration & refresh
- HTTPS for all communications
- Rate limiting on API endpoints
- SQL injection prevention via ORM
- CORS configuration
- Input validation & sanitization

## Deployment

### Docker Compose (Development)
- Backend, Frontend, PostgreSQL, Redis, Nginx RTMP
- Single command startup

### Kubernetes (Production)
- Separate deployments for each service
- Persistent volumes for data
- Ingress for routing
- ConfigMaps for configuration
- Secrets for sensitive data

## Monitoring

- Application logs: ELK Stack (Elasticsearch, Logstash, Kibana)
- Performance: Prometheus + Grafana
- Error tracking: Sentry
- Stream health: Custom metrics dashboard
