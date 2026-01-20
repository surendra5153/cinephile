# 🚀 Vercel Deployment Guide

This guide covers deploying the Movie Recommendation System to Vercel.

## 📋 Overview

Vercel supports:
- ✅ **Frontend**: React app (ideal)
- ✅ **Backend**: Serverless functions (API routes)
- ❌ **SQLite Database**: Not suitable for serverless (read-only at best)

## 🎯 Deployment Options

### Option 1: Full Vercel Deployment (Recommended)
- Frontend: Vercel
- Backend: Vercel Serverless Functions
- Database: External hosted database (PostgreSQL, Supabase, PlanetScale)

### Option 2: Hybrid Deployment
- Frontend: Vercel
- Backend: Railway/Render/Fly.io (separate deployment)
- Database: Same as backend or separate

---

## 🚀 Option 1: Full Vercel Deployment

### Step 1: Setup Project Structure for Vercel

Vercel expects API routes in a `api/` directory at the root or in the frontend directory.

**Recommended Structure:**
```
movie-rec-system/
├── frontend/          # React app
├── api/               # Vercel serverless functions
│   ├── auth/
│   ├── movies/
│   ├── search/
│   ├── ratings/
│   └── comments/
└── vercel.json        # Vercel configuration
```

### Step 2: Convert Backend to Serverless Functions

Create API route files in `api/` directory:

#### `api/auth/register.js`
```javascript
const database = require('../../backend/src/database/db');
const authService = require('../../backend/src/services/authService');

module.exports = async (req, res) => {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { username, password } = req.body;
    const result = await authService.register(username, password);
    res.status(201).json({
      message: 'User registered successfully',
      ...result
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};
```

#### `api/movies/[id].js` - Dynamic route
```javascript
const database = require('../../backend/src/database/db');

module.exports = async (req, res) => {
  const { id } = req.query;
  
  if (req.method === 'GET') {
    try {
      const movie = await database.queryOne(
        'SELECT * FROM movies WHERE id = ?',
        [id]
      );
      
      if (!movie) {
        return res.status(404).json({ error: 'Movie not found' });
      }
      
      // Get related data...
      res.json({ movie });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
};
```

**Better Approach**: Create a shared backend structure

### Step 3: Create Vercel Configuration

#### `vercel.json`
```json
{
  "version": 2,
  "builds": [
    {
      "src": "frontend/package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    },
    {
      "src": "api/**/*.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/frontend/$1"
    }
  ],
  "env": {
    "DATABASE_URL": "@database_url",
    "JWT_SECRET": "@jwt_secret"
  }
}
```

### Step 4: Setup Environment Variables in Vercel

1. Go to your Vercel project settings
2. Navigate to **Environment Variables**
3. Add:
   - `DATABASE_URL`: Your PostgreSQL connection string
   - `JWT_SECRET`: Your JWT secret key
   - `NODE_ENV`: `production`

### Step 5: Update Database Connection

Modify `backend/src/database/db.js` to support PostgreSQL:

```javascript
const { Pool } = require('pg');
// For local development, fallback to SQLite
const sqlite3 = require('sqlite3').verbose();

class Database {
  constructor() {
    this.pool = null;
    this.sqlite = null;
    this.isPostgres = !!process.env.DATABASE_URL;
  }

  async connect() {
    if (this.isPostgres) {
      // PostgreSQL for production
      this.pool = new Pool({
        connectionString: process.env.DATABASE_URL,
        ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false
      });
      console.log('Connected to PostgreSQL');
    } else {
      // SQLite for local development
      // ... existing SQLite connection code
    }
  }

  async query(sql, params = []) {
    if (this.isPostgres) {
      const result = await this.pool.query(sql, params);
      return result.rows;
    } else {
      // SQLite query
    }
  }
}
```

---

## 🎯 Option 2: Hybrid Deployment (Recommended for Now)

Deploy frontend to Vercel and backend separately.

### Part A: Deploy Frontend to Vercel

#### Step 1: Install Vercel CLI

```bash
npm install -g vercel
```

#### Step 2: Navigate to Frontend

```bash
cd frontend
```

#### Step 3: Create `vercel.json` in frontend directory

```json
{
  "version": 2,
  "buildCommand": "npm run build",
  "outputDirectory": "build",
  "devCommand": "npm start",
  "installCommand": "npm install",
  "framework": "create-react-app",
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "https://your-backend-url.railway.app/api/$1"
    }
  ],
  "env": {
    "REACT_APP_API_URL": "https://your-backend-url.railway.app/api"
  }
}
```

#### Step 4: Update API Configuration

Update `frontend/src/services/api.js`:

```javascript
const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json'
  }
});
```

#### Step 5: Deploy to Vercel

```bash
# Login to Vercel
vercel login

# Deploy
vercel

# For production
vercel --prod
```

Or use Vercel Dashboard:
1. Go to [vercel.com](https://vercel.com)
2. Click "New Project"
3. Import your Git repository
4. Set root directory to `frontend`
5. Build settings:
   - Framework Preset: Create React App
   - Build Command: `npm run build`
   - Output Directory: `build`
6. Add environment variable: `REACT_APP_API_URL`

### Part B: Deploy Backend to Railway/Render

#### Option B1: Railway (Recommended)

1. Go to [railway.app](https://railway.app)
2. Click "New Project" → "Deploy from GitHub"
3. Select your repository
4. Set root directory to `backend`
5. Add PostgreSQL service
6. Set environment variables:
   - `DATABASE_URL`: Auto-provided by Railway PostgreSQL
   - `JWT_SECRET`: Your secret
   - `PORT`: 5000
7. Deploy

**Railway automatically:**
- Detects Node.js app
- Runs `npm install`
- Runs `npm start`
- Provides public URL

#### Option B2: Render

1. Go to [render.com](https://render.com)
2. Create new Web Service
3. Connect GitHub repository
4. Settings:
   - Build Command: `cd backend && npm install && npm run init-db && npm run seed`
   - Start Command: `cd backend && npm start`
   - Environment: Node
5. Add PostgreSQL database
6. Set environment variables
7. Deploy

---

## 🔧 Quick Setup Scripts

### Create Deployment Configuration Files

#### `vercel.json` (in root)
```json
{
  "version": 2,
  "builds": [
    {
      "src": "frontend/package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build",
        "buildCommand": "cd frontend && npm install && npm run build"
      }
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/frontend/$1"
    }
  ]
}
```

#### `.vercelignore`
```
backend/node_modules
backend/database/*.db
node_modules
.env
.env.local
```

#### Update `frontend/.env.production`
```
REACT_APP_API_URL=https://your-backend-url.railway.app/api
```

---

## 📝 Step-by-Step Deployment Instructions

### Method 1: Vercel CLI (Frontend Only)

```bash
# 1. Install Vercel CLI
npm install -g vercel

# 2. Navigate to frontend
cd frontend

# 3. Login to Vercel
vercel login

# 4. Initialize project (first time)
vercel

# 5. Deploy to production
vercel --prod
```

### Method 2: Vercel Dashboard (Frontend Only)

1. **Prepare Frontend:**
   ```bash
   cd frontend
   npm run build  # Test build locally
   ```

2. **Deploy via Dashboard:**
   - Go to [vercel.com/dashboard](https://vercel.com/dashboard)
   - Click "New Project"
   - Import your Git repository
   - Configure:
     - Root Directory: `frontend`
     - Framework Preset: Create React App
     - Build Command: `npm run build`
     - Output Directory: `build`
     - Install Command: `npm install`

3. **Add Environment Variables:**
   - `REACT_APP_API_URL`: Your backend URL (e.g., `https://your-app.railway.app/api`)

4. **Deploy**

### Method 3: Backend on Railway

```bash
# 1. Install Railway CLI (optional)
npm install -g @railway/cli

# 2. Login
railway login

# 3. Initialize project
cd backend
railway init

# 4. Add PostgreSQL
railway add postgresql

# 5. Link environment variables
railway variables set JWT_SECRET=your-secret-key

# 6. Deploy
railway up
```

---

## 🗄️ Database Setup for Production

### Option 1: Supabase (Recommended)

1. Go to [supabase.com](https://supabase.com)
2. Create new project
3. Get connection string from Settings → Database
4. Use connection string in your backend

**Connection String Format:**
```
postgresql://postgres:[PASSWORD]@[HOST]:[PORT]/postgres
```

### Option 2: Railway PostgreSQL

1. Add PostgreSQL service in Railway
2. Use auto-provided `DATABASE_URL`
3. Update backend to use this URL

### Option 3: Render PostgreSQL

1. Create PostgreSQL database in Render
2. Get connection string
3. Add to backend environment variables

### Database Migration

Run migrations on your hosted database:

```bash
# Connect to production database
psql $DATABASE_URL

# Or use Railway CLI
railway run psql

# Run init.sql
\i backend/database/init.sql

# Seed data
railway run node backend/scripts/seedMovies.js
```

---

## 🔐 Environment Variables Checklist

### Frontend (Vercel)
- `REACT_APP_API_URL` - Backend API URL

### Backend (Railway/Render)
- `DATABASE_URL` - PostgreSQL connection string
- `JWT_SECRET` - JWT signing secret
- `PORT` - Server port (usually auto-set)
- `NODE_ENV` - `production`

---

## 🧪 Testing Deployment

### 1. Test Frontend

```bash
# Visit your Vercel deployment URL
# e.g., https://your-app.vercel.app

# Should load React app
# Check browser console for errors
```

### 2. Test Backend API

```bash
# Health check
curl https://your-backend-url.railway.app/health

# Should return: {"status":"OK","message":"Movie Recommendation API is running"}
```

### 3. Test Integration

1. Open frontend URL
2. Try registering a user
3. Try searching for movies
4. Check browser Network tab for API calls

---

## 🐛 Troubleshooting

### Issue: CORS Errors

**Solution**: Add CORS configuration in backend:

```javascript
// backend/src/app.js
const cors = require('cors');

app.use(cors({
  origin: process.env.FRONTEND_URL || 'https://your-app.vercel.app',
  credentials: true
}));
```

### Issue: Database Connection Errors

**Solution**: 
1. Check `DATABASE_URL` is correct
2. Ensure database allows connections from your hosting IP
3. Check SSL settings for PostgreSQL

### Issue: Build Failures

**Solution**:
- Check build logs in Vercel dashboard
- Ensure all dependencies are in `package.json`
- Check Node.js version compatibility

### Issue: API Routes Not Found

**Solution**:
- Verify `REACT_APP_API_URL` is set correctly
- Check backend is running and accessible
- Verify CORS configuration

---

## 📊 Monitoring & Analytics

### Vercel Analytics

1. Enable in Vercel dashboard
2. Add to `frontend/src/index.js`:
```javascript
import { Analytics } from '@vercel/analytics/react';

function App() {
  return (
    <>
      <YourApp />
      <Analytics />
    </>
  );
}
```

### Backend Logging

Use Railway/Render logs:
```bash
# Railway
railway logs

# Render
# View in dashboard → Logs
```

---

## 🚀 Production Checklist

- [ ] Database migrated to PostgreSQL
- [ ] Environment variables set
- [ ] CORS configured
- [ ] Frontend deployed to Vercel
- [ ] Backend deployed to Railway/Render
- [ ] API URL configured in frontend
- [ ] Database seeded with movies
- [ ] SSL/HTTPS enabled (automatic on Vercel)
- [ ] Error monitoring setup
- [ ] Performance monitoring enabled

---

## 📚 Additional Resources

- [Vercel Documentation](https://vercel.com/docs)
- [Railway Documentation](https://docs.railway.app)
- [Render Documentation](https://render.com/docs)
- [Supabase Documentation](https://supabase.com/docs)

---

## 💡 Recommended Architecture

```
┌─────────────────┐
│   Vercel CDN    │  ← Frontend (React)
└────────┬────────┘
         │ HTTPS
┌────────▼────────┐
│ Railway/Render  │  ← Backend API (Express)
└────────┬────────┘
         │
┌────────▼────────┐
│  Supabase/DB    │  ← PostgreSQL Database
└─────────────────┘
```

This architecture provides:
- ✅ Fast global CDN for frontend
- ✅ Scalable serverless/hosting for backend
- ✅ Managed PostgreSQL database
- ✅ Automatic SSL certificates
- ✅ Easy scaling

---

## 🎯 Quick Deploy Commands

### One-time Setup

```bash
# Frontend
cd frontend
vercel

# Backend
cd backend
railway init
railway add postgresql
railway up
```

### Update Deployment

```bash
# Frontend
cd frontend
vercel --prod

# Backend (auto-deploys on git push if connected)
git push origin main
```

---

**Need help?** Check the troubleshooting section or refer to the hosting provider's documentation.
