Add this section to your README.md after the "Database Management" section:

```markdown
## 🚀 Deploy to Vercel

### Quick Deploy Guide

**Recommended:** Frontend on Vercel + Backend on Railway/Render

1. **Deploy Frontend to Vercel:**
   ```bash
   cd frontend
   npm install -g vercel
   vercel login
   vercel
   ```

2. **Deploy Backend to Railway:**
   - Visit [railway.app](https://railway.app)
   - New Project → Deploy from GitHub
   - Set Root Directory: `backend`
   - Add PostgreSQL database
   - Set environment variables

3. **Configure:**
   - Set `REACT_APP_API_URL` in Vercel (your backend URL)
   - Set `DATABASE_URL` and `JWT_SECRET` in Railway

📖 **Detailed Guides:**
- [DEPLOYMENT_QUICK_START.md](./DEPLOYMENT_QUICK_START.md) - Quick start (10 minutes)
- [VERCEL_DEPLOYMENT.md](./VERCEL_DEPLOYMENT.md) - Complete deployment guide
```
