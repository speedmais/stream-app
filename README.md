# Stream App 📺

A simplified streaming platform inspired by Twitch with live video, real-time chat, and user interactions.

## Features

✅ Live streaming (RTMP ingest, HLS/WebRTC delivery)
✅ Real-time chat with WebSockets
✅ User authentication (JWT)
✅ Stream discovery & recommendations
✅ Follow/Subscribe system
✅ Mobile responsive UI
✅ Docker setup for easy deployment

## Tech Stack

### Backend
- **Node.js** + **NestJS** (TypeScript)
- **PostgreSQL** for relational data
- **Redis** for caching & pub/sub
- **Socket.IO** for real-time chat

### Frontend
- **Next.js** + **React** + **TypeScript**
- **TailwindCSS** for styling
- **Video.js** for HLS playback
- **Socket.IO Client** for chat

### Streaming
- **RTMP Server** (Nginx RTMP) for ingest
- **HLS** for viewer delivery
- **WebRTC** for ultra-low latency (optional)

## Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+ (for local development)

### With Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/speedmais/stream-app.git
cd stream-app

# Start all services
docker-compose up -d

# Access the app
# Frontend: http://localhost:3000
# Backend API: http://localhost:3001
# RTMP Server: rtmp://localhost:1935/live
```

### Local Development

```bash
# Backend
cd backend
npm install
npm run start:dev

# Frontend (in another terminal)
cd frontend
npm install
npm run dev
```

## Project Structure

```
stream-app/
├── backend/
│   ├── src/
│   │   ├── auth/              # Authentication module
│   │   ├── streams/           # Stream management
│   │   ├── chat/              # Real-time chat
│   │   ├── users/             # User profiles
│   │   ├── database/          # Database config & migrations
│   │   └── main.ts
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/
│   ├── src/
│   │   ├── app/               # Next.js pages & layouts
│   │   ├── components/        # React components
│   │   ├── hooks/             # Custom hooks
│   │   ├── utils/             # Utilities
│   │   └── styles/
│   ├── Dockerfile.dev
│   ├── package.json
│   └── next.config.js
│
├── nginx-rtmp/
│   ├── nginx.conf             # RTMP server config
│   └── Dockerfile
│
├── docker-compose.yml
└── README.md
```

## API Endpoints (Backend)

### Authentication
- `POST /auth/register` - Register new user
- `POST /auth/login` - Login user
- `POST /auth/refresh` - Refresh JWT token

### Streams
- `GET /streams` - List all active streams
- `GET /streams/:id` - Get stream details
- `POST /streams` - Create new stream (authenticated)
- `PATCH /streams/:id` - Update stream (owner only)
- `DELETE /streams/:id` - End stream (owner only)

### Users
- `GET /users/:id` - Get user profile
- `PATCH /users/:id` - Update user profile (authenticated)
- `POST /users/:id/follow` - Follow user
- `DELETE /users/:id/follow` - Unfollow user

### Chat (WebSocket)
- `/socket.io` - Real-time chat & notifications

## WebSocket Events

```javascript
// Client → Server
emit('message:send', { streamId, content })
emit('user:join', { streamId })
emit('user:leave', { streamId })

// Server → Client
on('message:new', (data) => {})
on('user:joined', (user) => {})
on('user:left', (user) => {})
on('stream:started', (stream) => {})
on('stream:ended', (stream) => {})
```

## Environment Variables

Create `.env` files in `backend/` and `frontend/` directories:

### backend/.env
```
NODE_ENV=development
PORT=3001
DATABASE_URL=postgresql://stream_user:stream_password@localhost:5432/stream_app
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_secret_key_here
JWT_EXPIRES_IN=7d
```

### frontend/.env.local
```
NEXT_PUBLIC_API_URL=http://localhost:3001
```

## Streaming Setup

### OBS Studio Configuration

1. Open OBS Studio
2. Settings → Stream
3. Service: Custom
4. Server: `rtmp://localhost:1935/live`
5. Stream Key: `your-stream-key` (or username)
6. Start streaming!

### View Stream

Open browser: `http://localhost:3000/watch/your-stream-key`

## Database Migrations

```bash
# Run migrations
npm run db:migrate

# Seed database with sample data
npm run db:seed
```

## Testing

```bash
# Backend
cd backend
npm run test
npm run test:e2e

# Frontend
cd frontend
npm run test
```

## Deployment

### Using Docker

```bash
# Build images
docker-compose build

# Push to registry (optional)
docker tag stream-app-backend your-registry/stream-app-backend:latest
docker push your-registry/stream-app-backend:latest

# Deploy to production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Using Kubernetes

See `k8s/` directory for Kubernetes manifests.

## Contributing

Contributions are welcome! Please create a feature branch and submit a pull request.

```bash
git checkout -b feature/amazing-feature
git commit -m 'Add amazing feature'
git push origin feature/amazing-feature
```

## License

MIT License - see LICENSE file for details

## Support

Have questions? Open an issue or contact us!

---

**Happy streaming! 🚀**
