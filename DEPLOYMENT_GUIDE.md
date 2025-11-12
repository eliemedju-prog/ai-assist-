# Complete Deployment Guide
## AI-Powered IT Network Admin Assistant

This guide provides step-by-step instructions for deploying the frontend and backend to production.

## ✅ Current Project Status

**Repository:** https://github.com/eliemedju-prog/ai-assist-
**Date:** November 11, 2025
**Status:** Repository structured and ready for full application code

### Completed Tasks
1. ✅ GitHub repository created with initial structure
2. ✅ Frontend and backend package.json files committed
3. ✅ Comprehensive README.md documentation added
4. ✅ Project directory structure established:
   - `/frontend` - React TypeScript application
   - `/backend` - Express.js Node.js API server
5. ✅ Vercel integration configured for automatic deployment

---

## 🚀 Frontend Deployment (Vercel)

### Prerequisites
- Vercel account (free tier available)
- GitHub repository with frontend code
- Node.js 16+ and npm installed locally

### Step 1: Add Frontend Source Code

First, ensure your frontend React application is in the `/frontend` directory with:
```
frontend/
├── public/
├── src/
│   ├── components/
│   ├── hooks/
│   ├── api/
│   ├── App.tsx
│   └── index.tsx
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── .env.example
```

### Step 2: Push Code to GitHub
```bash
git add .
git commit -m "Add complete frontend React application"
git push origin main
```

### Step 3: Deploy to Vercel

1. Go to https://vercel.com/dashboard
2. Click "Add New..." → "Project"
3. Select the `ai-assist-` GitHub repository
4. Set Root Directory to: `frontend`
5. Click "Deploy"

**Deployment URL:** `https://ai-assist-<your-team>.vercel.app`

### Step 4: Configure Environment Variables

In Vercel dashboard, add environment variables:
```
REACT_APP_API_URL=https://your-backend-api.com
REACT_APP_ENV=production
```

---

## 🔧 Backend Deployment (Heroku or Railway)

### Option A: Deploy to Railway (Recommended - Free)

#### Prerequisites
- Railway account at https://railway.app
- GitHub repository connected to Railway

#### Steps

1. **Create New Project on Railway**
   - Go to https://railway.app/new
   - Select "Deploy from GitHub"
   - Connect your GitHub account
   - Select `ai-assist-` repository

2. **Configure Railway Project**
   - Select `backend` as the root directory
   - Set Environment Variables:
     ```
     NODE_ENV=production
     PORT=3001
     ```

3. **Deploy**
   - Railway automatically deploys on git push
   - View logs in Railway dashboard

**Backend URL:** `https://your-project-name.up.railway.app`

### Option B: Deploy to Heroku (Requires Credit Card)

#### Prerequisites
- Heroku account
- Heroku CLI installed

#### Steps

1. **Login to Heroku**
```bash
heroku login
```

2. **Create Heroku App**
```bash
heroku create ai-assist-backend
```

3. **Set Buildpack**
```bash
heroku buildpacks:set heroku/nodejs -a ai-assist-backend
```

4. **Add Procfile to Backend**

Create `backend/Procfile`:
```
web: npm start
```

5. **Deploy**
```bash
cd backend
git subtree push --prefix backend heroku main
```

6. **Set Environment Variables**
```bash
heroku config:set NODE_ENV=production -a ai-assist-backend
heroku config:set PORT=3001 -a ai-assist-backend
```

**Backend URL:** `https://ai-assist-backend.herokuapp.com`

---

## 🔌 Connect Frontend to Backend

### Update Frontend Environment Variables

After backend deployment, update Vercel environment variables:

**Frontend Environment Variable:**
```
REACT_APP_API_URL=https://your-backend-url.com
```

Update in `frontend/src/api/client.ts`:
```typescript
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3001';
```

### Test Connection

1. Frontend should now call backend API
2. Check browser console for network requests
3. Verify CORS headers are configured in backend

---

## 📚 Local Development Setup

### Start Backend Locally
```bash
cd backend
npm install
npm run dev
# Runs on http://localhost:3001
```

### Start Frontend Locally
```bash
cd frontend
npm install
npm start
# Runs on http://localhost:3000
```

### Test Full Stack
- Frontend: http://localhost:3000
- Backend API: http://localhost:3001/api

---

## 🐛 Troubleshooting

### Frontend Build Fails
- Check Node.js version: `node -v` (should be 16+)
- Clear cache: `rm -rf node_modules && npm install`
- Check environment variables in Vercel dashboard

### Backend Connection Issues
- Verify CORS configuration in backend
- Check API URL environment variable
- Ensure backend is running and accessible

### Deployment Takes Too Long
- Large dependencies slow build time
- Optimize bundle size
- Check for large files in repository

---

## 📊 Monitoring

### Vercel Deployment Monitoring
- Dashboard: https://vercel.com/dashboard
- View real-time logs and build status
- Auto-rollback on failed deployment

### Railway/Heroku Monitoring
- View application logs
- Monitor resource usage
- Set up alerts for crashes

---

## 🎯 Next Steps

1. **Add Complete Source Code**
   - Push React components to `/frontend`
   - Push Express routes to `/backend`
   - Update package.json scripts

2. **Set Up CI/CD**
   - Add GitHub Actions for testing
   - Automatic deployment on main branch

3. **Configure Custom Domain**
   - Add domain to Vercel
   - Update CORS in backend

4. **Add Monitoring**
   - Set up error tracking (Sentry)
   - Add performance monitoring
   - Configure uptime monitoring

---

## 📞 Support

For deployment issues:
1. Check official documentation
2. Review deployment logs
3. Test locally first
4. Open GitHub issue with logs

**Last Updated:** November 11, 2025
**Project Owner:** eliemedju-prog
