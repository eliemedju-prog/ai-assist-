# AI-Powered IT Network Admin Assistant

A production-ready full-stack application for IT network administration featuring an AI chat assistant, incident management, and real-time network monitoring.

## Project Structure

```
ai-assist-/
├── frontend/                 # React TypeScript frontend
│   ├── src/
│   │   ├── components/      # React components
│   │   ├── hooks/           # Custom React hooks
│   │   ├── types/           # TypeScript interfaces
│   │   ├── api/             # API client
│   │   ├── styles/          # Tailwind CSS
│   │   └── App.tsx          # Main app component
│   ├── package.json
│   ├── tsconfig.json
│   └── tailwind.config.js
├── backend/                  # Node.js Express backend
│   ├── src/
│   │   ├── routes/          # API endpoints
│   │   ├── controllers/     # Business logic
│   │   ├── models/          # Data models
│   │   ├── middleware/      # Express middleware
│   │   └── server.ts        # Express server
│   ├── package.json
│   └── tsconfig.json
└── README.md
```

## Features

✅ **Dashboard UI**
- Dark/Light mode toggle
- Responsive design (desktop & tablet)
- Sidebar navigation
- Real-time notifications

✅ **Network Management**
- Network health monitoring
- VPN pool usage tracking
- Device status alerts
- Uptime metrics

✅ **Incident Management**
- Ticket creation and tracking
- Quick actions (Acknowledge, Close, Assign)
- Filter and search
- Status updates

✅ **AI Chat Assistant**
- Real-time messaging
- File upload support
- Voice input
- Context-aware responses

✅ **Backend API**
- 15+ RESTful endpoints
- Mock database
- Error handling
- Environment configuration

## Tech Stack

**Frontend:**
- React 18+
- TypeScript
- Tailwind CSS
- Framer Motion
- Axios
- shadcn/ui components

**Backend:**
- Node.js
- Express.js
- TypeScript
- Cors
- Body-parser

## Getting Started

### Prerequisites
- Node.js 16+
- npm or yarn
- Git

### Installation

1. **Clone the repository:**
```bash
git clone https://github.com/eliemedju-prog/ai-assist-.git
cd ai-assist-
```

2. **Install backend dependencies:**
```bash
cd backend
npm install
```

3. **Install frontend dependencies:**
```bash
cd ../frontend
npm install
```

### Running Locally

**Terminal 1 - Start Backend:**
```bash
cd backend
npm start
# Runs on http://localhost:3001
```

**Terminal 2 - Start Frontend:**
```bash
cd frontend
npm start
# Runs on http://localhost:3000
```

Open your browser to `http://localhost:3000`

## API Endpoints

### Incidents
- `GET /api/incidents` - List all incidents
- `POST /api/incidents` - Create new incident
- `GET /api/incidents/:id` - Get incident details
- `PUT /api/incidents/:id` - Update incident
- `DELETE /api/incidents/:id` - Delete incident
- `POST /api/incidents/:id/acknowledge` - Acknowledge incident
- `POST /api/incidents/:id/close` - Close incident

### Network
- `GET /api/network/status` - Get network health
- `GET /api/network/devices` - List devices
- `GET /api/network/vpn` - Get VPN metrics
- `GET /api/network/uptime` - Get uptime stats

### Chat
- `POST /api/chat` - Send message to AI
- `GET /api/chat/history` - Get chat history

## Deployment

### Option 1: Vercel (Frontend) + Heroku (Backend)

**Frontend on Vercel:**
1. Push code to GitHub
2. Connect GitHub to Vercel
3. Select `frontend` as root directory
4. Deploy

**Backend on Heroku:**
1. Push code to GitHub
2. Connect GitHub to Heroku
3. Select `backend` as root directory
4. Add environment variables
5. Deploy

### Option 2: Docker

**Build and run with Docker:**
```bash
docker-compose up
```

### Option 3: Traditional Hosting

1. Deploy backend to any Node.js hosting
2. Deploy frontend to any static hosting
3. Update API URL in frontend environment

## Environment Variables

### Backend (.env)
```
PORT=3001
NODE_ENV=production
API_URL=http://localhost:3001
```

### Frontend (.env)
```
REACT_APP_API_URL=http://localhost:3001
REACT_APP_ENV=development
```

## Usage

1. **Login Dashboard** - Access the main dashboard
2. **Monitor Network** - View real-time network status
3. **Manage Incidents** - Create, update, and close tickets
4. **AI Chat** - Ask questions or get assistance
5. **Settings** - Configure preferences and API keys

## Development

### Running Tests
```bash
cd backend
npm test

cd ../frontend
npm test
```

### Building for Production

**Frontend:**
```bash
cd frontend
npm run build
```

**Backend:**
```bash
cd backend
npm run build
```

## File Sizes & Performance

- Frontend Bundle: ~150KB (gzipped)
- Backend: ~5MB
- Total: ~155MB with node_modules

## License

MIT License - see LICENSE file for details

## Support

For issues and questions, please open a GitHub issue.

## Last Updated

Tuesday, November 11, 2025
