# ⚡ Quick Start: Deploy to Vercel

## 🚀 Fastest Way to Deploy

### Step 1: Deploy Frontend to Vercel (5 minutes)

1. **Install Vercel CLI:**
   ```bash
   npm install -g vercel
   ```

2. **Navigate to frontend:**
   ```bash
   cd frontend
   ```

3. **Login to Vercel:**
   ```bash
   vercel login
   ```

4. **Deploy:**
   ```bash
   vercel
   ```
   - Follow prompts
   - For production: `vercel --prod`

5. **Set Environment Variable:**
   - Go to [vercel.com/dashboard](https://vercel.com/dashboard)
   - Select your project
   - Go to Settings → Environment Variables
   - Add: `REACT_APP_API_URL` = `https://your-backend-url.railway.app/api`

### Step 2: Deploy Backend to Railway (5 minutes)

1. **Go to Railway:**
   - Visit [railway.app](https://railway.app)
   - Sign up with GitHub

2. **Create New Project:**
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Choose your repository
   - Set Root Directory: `backend`

3. **Add PostgreSQL:**
   - Click "New" → "Database" → "Add PostgreSQL"
   - Railway auto-provides `DATABASE_URL`

4. **Set Environment Variables:**
   - Click on your backend service
   - Go to Variables tab
   - Add:
     - `JWT_SECRET`: (generate a random secret)
     - `PORT`: `5000`
     - `DATABASE_URL`: (auto-set by Railway)

5. **Deploy:**
   - Railway auto-deploys on git push
   - Or click "Deploy" button

6. **Get Backend URL:**
   - Railway provides a public URL
   - Copy this URL (e.g., `https://your-app.railway.app`)

### Step 3: Update Frontend API URL

1. **Update Vercel Environment Variable:**
   - Go to Vercel dashboard
   - Settings → Environment Variables
   - Update `REACT_APP_API_URL` with your Railway URL: `https://your-app.railway.app/api`

2. **Redeploy Frontend:**
   ```bash
   cd frontend
   vercel --prod
   ```

### Step 4: Initialize Database

1. **Connect to Railway Database:**
   ```bash
   # Install Railway CLI
   npm install -g @railway/cli
   
   # Login
   railway login
   
   # Link to your project
   railway link
   ```

2. **Run Database Setup:**
   ```bash
   cd backend
   
   # Initialize database (from init.sql)
   railway run node -e "const db = require('./src/database/db'); db.connect().then(() => { const fs = require('fs'); const sql = fs.readFileSync('./database/init.sql', 'utf8'); db.db.exec(sql); console.log('Database initialized'); process.exit(0); })"
   
   # Seed movies
   railway run npm run seed
   ```

   **Or manually via PostgreSQL client:**
   ```bash
   # Get connection string from Railway
   railway variables
   
   # Connect with psql
   psql $DATABASE_URL
   
   # Run init.sql
   \i backend/database/init.sql
   
   # Exit and seed
   \q
   railway run npm run seed
   ```

---

## 🎯 Alternative: Deploy Backend to Render

### Render Setup:

1. **Go to Render:**
   - Visit [render.com](https://render.com)
   - Sign up with GitHub

2. **Create Web Service:**
   - New → Web Service
   - Connect GitHub repository
   - Settings:
     - **Name**: `movie-rec-backend`
     - **Root Directory**: `backend`
     - **Environment**: `Node`
     - **Build Command**: `npm install`
     - **Start Command**: `npm start`
     - **Instance Type**: Free tier (or paid for production)

3. **Add PostgreSQL:**
   - New → PostgreSQL
   - Copy connection string

4. **Set Environment Variables:**
   - `DATABASE_URL`: (from Render PostgreSQL)
   - `JWT_SECRET`: (your secret)
   - `PORT`: `5000`
   - `NODE_ENV`: `production`

5. **Initialize Database:**
   - SSH into Render instance or use Render Shell
   - Run: `npm run init-db && npm run seed`

---

## 🔧 Environment Variables Summary

### Frontend (Vercel):
```
REACT_APP_API_URL=https://your-backend.railway.app/api
```

### Backend (Railway/Render):
```
DATABASE_URL=postgresql://... (auto-provided)
JWT_SECRET=your-random-secret-key
PORT=5000
NODE_ENV=production
```

---

## ✅ Testing Your Deployment

1. **Frontend:**
   - Visit your Vercel URL
   - Should load the React app

2. **Backend API:**
   ```bash
   curl https://your-backend.railway.app/health
   ```
   Should return: `{"status":"OK","message":"Movie Recommendation API is running"}`

3. **Full Integration:**
   - Try registering a user
   - Search for movies
   - Check browser console for errors

---

## 🐛 Troubleshooting

### CORS Errors:
Update `backend/src/app.js`:
```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL || 'https://your-app.vercel.app',
  credentials: true
}));
```

### Database Connection Issues:
- Check `DATABASE_URL` is correct
- Ensure database is accessible
- Check PostgreSQL connection limits

### Build Failures:
- Check build logs in Vercel/Railway dashboard
- Ensure all dependencies in `package.json`
- Check Node.js version compatibility

---

## 📚 Full Documentation

See [VERCEL_DEPLOYMENT.md](./VERCEL_DEPLOYMENT.md) for detailed deployment guide.

---

**That's it! Your app should now be live! 🎉**
