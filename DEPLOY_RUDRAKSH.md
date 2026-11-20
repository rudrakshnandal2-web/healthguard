# Deploy Rudraksh: Railway + Netlify

## 1. Upload to GitHub
Unzip this archive, open the Rudraksh directory in Terminal, then run:
```bash
git init
git add .
git commit -m "Prepare Rudraksh for deployment"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/rudraksh.git
git push -u origin main
```
Create the empty GitHub repository first. Do not commit real `.env` files or secrets.

## 2. Railway PostgreSQL
In Railway, create a project and add a **PostgreSQL** service. Copy the PostgreSQL connection URL into the backend service's `DATABASE_URL` variable using Railway variable references where available (for example `${{Postgres.DATABASE_URL}}`). Railway's URL may begin with `postgresql://`; both this and `postgres://` are handled by the config. Use the **private** URL for services in the same Railway project.

## 3. Railway backend
Add a service from the GitHub repo. Set **Root Directory** to `/backend`, choose the Dockerfile build, and configure:
```env
ENVIRONMENT=production
DATABASE_URL=<Railway Postgres private DATABASE_URL>
SECRET_KEY=<random 64+ character secret>
WHATSAPP_VERIFY_TOKEN=<random token>
BACKEND_CORS_ORIGINS=https://YOUR-SITE.netlify.app
```
Generate secrets locally with `python3 -c "import secrets; print(secrets.token_urlsafe(48))"`. Do not use mock secrets.

Deploy the backend and generate a public Railway domain. The Dockerfile starts Uvicorn using Railway's `$PORT`. Run database migrations using Railway's service shell or a one-off job with `alembic upgrade head` from `/workspace`, after verifying DB connectivity. Startup currently also calls `Base.metadata.create_all`; review schema/migration consistency before using real patient data.

Check `https://YOUR-BACKEND.up.railway.app/health`, `/ready`, and `/docs`.

## 4. Netlify frontend
Add a new site from the same GitHub repo. Set **Base directory** to `frontend`, **Build command** `npm run build`, **Publish directory** `dist` (relative to base), and environment variable:
```env
VITE_API_URL=https://YOUR-BACKEND.up.railway.app
```
Deploy. `frontend/netlify.toml` provides the SPA redirect for direct page loads.

## 5. CORS and verification
Update Railway `BACKEND_CORS_ORIGINS` with your **exact** Netlify URL (no trailing slash); redeploy the backend. Verify signup, login, chatbot response, vaccination CRUD, admin authorization, and page refresh on nested routes. Test on mobile. If a request fails, inspect browser Network tab and Railway logs.

## Important safeguards
- This is a public-health application. Do not enter real patient records before reviewing privacy, authentication, authorization, medical content, and data retention.
- The chatbot's existing NLP/translation integrations may depend on external Rasa/translation services; hosting this ZIP alone does not train a medical classifier or make those services operational.
- SMS/WhatsApp require real provider credentials and provider setup; mock defaults are not working integrations.
- Keep third-party copyright notices and licenses; changing the product's branding does not transfer original authorship or ownership.
- Deploying from this archive is not the same as a verified successful live deployment.
