# Installation Guide — AI-Empowered Urban Governance System

Complete step-by-step installation instructions for **Ubuntu** and **Windows**.

---

## Prerequisites (Both Platforms)

Before installing, you need free accounts on these services:

| Service | URL | What You Get |
|---------|-----|-------------|
| **Clerk** | https://clerk.com | Authentication (free tier) |
| **NeonDB** | https://neon.tech | PostgreSQL database (free tier) |
| **Google AI Studio** | https://aistudio.google.com | Gemini API key (free tier) |

---

---

# UBUNTU / LINUX INSTALLATION

---

## Step 1 — Install System Dependencies

Open a terminal and run:

```bash
# Update package list
sudo apt update && sudo apt upgrade -y

# Install curl and git
sudo apt install -y curl git

# Install Node.js 20 (LTS) using NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify Node.js installation
node --version     # Should show v20.x.x
npm --version      # Should show 10.x.x

# Install Python 3.11 and pip
sudo apt install -y python3 python3-venv python3-pip python3-dev

# Verify Python installation
python3 --version   # Should show Python 3.12.x
pip3 --version
```

---

## Step 2 — Clone or Navigate to the Project

```bash
# Navigate to the project directory
cd /home/thakur/urban-governance

# OR if you're setting up fresh from a git repo:
# git clone <your-repo-url>
# cd urban-governance
```

---

## Step 3 — Set Up Environment Variables

```bash
# Copy the example env file
cp .env.example .env.local

# Open in your text editor
nano .env.local
# OR
code .env.local      # if VS Code is installed
# OR
gedit .env.local     # GUI text editor
```

Fill in the following values in `.env.local`:

```env
# --- CLERK ---
# Get from: https://dashboard.clerk.com → Your App → API Keys
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Clerk Redirects (keep these as-is)
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/citizen/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/citizen/dashboard

# --- NEONDB ---
# Get from: https://console.neon.tech → Your Project → Connection String
DATABASE_URL="postgresql://username:password@ep-xxx.us-east-1.aws.neon.tech/urban_governance?sslmode=require"

# --- GEMINI AI ---
# Get from: https://aistudio.google.com/app/apikey
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# --- APP SETTINGS ---
AI_SERVICE_URL=http://localhost:8000
NEXT_PUBLIC_APP_URL=http://localhost:3000
ADMIN_EMAIL=your-admin@email.com
```

Save and close the file (`Ctrl+X` then `Y` then `Enter` in nano).

---

## Step 4 — Install Node.js Dependencies

```bash
# Make sure you are in the project root
cd /home/thakur/urban-governance

# Install all packages
npm install --legacy-peer-deps

# Expected output: "added 536 packages"
```

---

## Step 5 — Set Up the Database (NeonDB)

```bash
# Push the Prisma schema to your NeonDB database
npx prisma db push

# Generate the Prisma TypeScript client
npx prisma generate

# Optional: Open Prisma Studio to view your database in a browser
npx prisma studio
# Opens at http://localhost:5555
```

---

## Step 6 — Set Up Python AI Service

```bash
# Navigate to the python directory
cd /home/thakur/urban-governance/python

# Create a Python virtual environment
python3 -m venv venv

# Activate the virtual environment
source venv/bin/activate
# You should see (venv) at the start of your terminal prompt

# Install Python dependencies
pip install -r requirements.txt

# Create Python environment file
cp .env.example .env
nano .env
```

Add your Gemini API key in `python/.env`:
```env
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Save and close.

---

## Step 7 — Set Up Clerk Webhooks

1. Go to [Clerk Dashboard](https://dashboard.clerk.com)
2. Select your application
3. Go to **Webhooks** in the left sidebar
4. Click **Add Endpoint**
5. For local development, use [ngrok](https://ngrok.com) to expose your local server:

```bash
# Install ngrok (in a NEW terminal, keep it running)
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null \
  && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" \
  | sudo tee /etc/apt/sources.list.d/ngrok.list \
  && sudo apt update \
  && sudo apt install ngrok

# Start ngrok (after Next.js is running on port 3000)
ngrok http 3000
```

6. Copy the ngrok URL (e.g., `https://abc123.ngrok.io`)
7. In Clerk Webhooks, set URL to: `https://abc123.ngrok.io/api/webhooks/clerk`
8. Select events: `user.created`, `user.updated`, `user.deleted`
9. Copy the **Signing Secret** → paste into `CLERK_WEBHOOK_SECRET` in `.env.local`

---

## Step 8 — Start the Development Servers

You need **two terminals** running simultaneously:

### Terminal 1 — Next.js Frontend
```bash
cd /home/thakur/urban-governance
npm run dev
```
Output should show:
```
▲ Next.js 15.1.7
- Local:        http://localhost:3000
- Ready in 2.3s
```

### Terminal 2 — Python AI Service
```bash
cd /home/thakur/urban-governance/python
source venv/bin/activate
uvicorn main:app --reload --port 8000
```
Output should show:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
INFO:     Application startup complete.
```

---

## Step 9 — Access the Application

Open your browser and go to: **http://localhost:3000**

### First-Time Setup
1. Click **Get Started** → Sign Up
2. Use the email you set as `ADMIN_EMAIL` to get admin access
3. Sign up → Clerk webhook fires → user saved to NeonDB with `ADMIN` role
4. You'll be redirected to `/citizen/dashboard` first, then to `/admin` after role is applied

---

## Ubuntu Useful Commands

```bash
# Check if ports are in use
sudo lsof -i :3000
sudo lsof -i :8000

# Kill a process on a specific port
sudo kill -9 $(sudo lsof -t -i:3000)

# View Prisma database (GUI)
npx prisma studio

# Run database migrations
npx prisma migrate dev --name init

# Reset database (WARNING: deletes all data)
npx prisma migrate reset

# View Next.js build output
npm run build

# Check Python virtual environment is active
which python   # Should show path containing venv/

# Deactivate Python virtual environment
deactivate

# View Python service API docs
# Go to: http://localhost:8000/docs
```

---

---

# WINDOWS INSTALLATION

---

## Step 1 — Install System Dependencies

### Install Node.js 20
1. Go to https://nodejs.org
2. Download **Node.js 20 LTS** (Windows Installer `.msi`)
3. Run the installer, check "Add to PATH" option
4. Open **Command Prompt** or **PowerShell** and verify:

```powershell
node --version     # Should show v20.x.x
npm --version      # Should show 10.x.x
```

### Install Python 3.11
1. Go to https://www.python.org/downloads/
2. Download **Python 3.11.x** (Windows installer)
3. Run the installer
4. **IMPORTANT**: Check **"Add Python 3.11 to PATH"** before clicking Install
5. Verify in Command Prompt:

```powershell
python --version   # Should show Python 3.11.x
pip --version
```

### Install Git (optional but recommended)
1. Download from https://git-scm.com/download/win
2. Install with default settings

---

## Step 2 — Navigate to Project Directory

Open **Command Prompt** (Win + R → type `cmd` → Enter) or **PowerShell**:

```powershell
# Navigate to your project
cd C:\Users\YourName\urban-governance

# OR if you downloaded it as a zip, extract it and navigate there
# cd C:\path\to\urban-governance
```

---

## Step 3 — Set Up Environment Variables

```powershell
# Copy the example env file
copy .env.example .env.local
```

Open `.env.local` in Notepad or VS Code:

```powershell
# Open with Notepad
notepad .env.local

# OR open with VS Code (if installed)
code .env.local
```

Fill in all the values (same as Ubuntu Step 3):

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
CLERK_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/citizen/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/citizen/dashboard
DATABASE_URL="postgresql://username:password@ep-xxx.us-east-1.aws.neon.tech/urban_governance?sslmode=require"
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AI_SERVICE_URL=http://localhost:8000
NEXT_PUBLIC_APP_URL=http://localhost:3000
ADMIN_EMAIL=your-admin@email.com
```

Save the file.

---

## Step 4 — Install Node.js Dependencies

```powershell
# Make sure you are in the project root
cd C:\Users\YourName\urban-governance

# Install all packages
npm install --legacy-peer-deps

# Expected output: "added 536 packages"
```

---

## Step 5 — Set Up the Database (NeonDB)

```powershell
# Push Prisma schema to NeonDB
npx prisma db push

# Generate Prisma client
npx prisma generate

# Optional: Open Prisma Studio
npx prisma studio
```

---

## Step 6 — Set Up Python AI Service

```powershell
# Navigate to python directory
cd C:\Users\YourName\urban-governance\python

# Create Python virtual environment
python -m venv venv

# Activate the virtual environment (Windows)
venv\Scripts\activate
# You should see (venv) in your prompt

# Install Python dependencies
pip install -r requirements.txt

# Create Python environment file
copy .env.example .env
notepad .env
```

Add your Gemini API key:
```env
GEMINI_API_KEY=AIzaSyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Save and close.

---

## Step 7 — Set Up Clerk Webhooks (Windows)

For local development, use ngrok:

```powershell
# Install ngrok via Chocolatey (recommended)
# First install Chocolatey: https://chocolatey.org/install
choco install ngrok

# OR download ngrok.exe directly from https://ngrok.com/download
# Extract and add to PATH

# After Next.js is running, expose it:
ngrok http 3000
```

Follow the same steps as Ubuntu Step 7 to configure Clerk webhooks.

---

## Step 8 — Start the Development Servers

You need **two Command Prompt / PowerShell windows**:

### Window 1 — Next.js Frontend
```powershell
cd C:\Users\YourName\urban-governance
npm run dev
```

### Window 2 — Python AI Service
```powershell
cd C:\Users\YourName\urban-governance\python
venv\Scripts\activate
uvicorn main:app --reload --port 8000
```

---

## Step 9 — Access the Application

Open browser: **http://localhost:3000**

---

## Windows Useful Commands

```powershell
# Check if port is in use
netstat -ano | findstr :3000
netstat -ano | findstr :8000

# Kill a process by PID (replace XXXX with PID from above)
taskkill /PID XXXX /F

# Activate Python venv (Windows)
venv\Scripts\activate

# Deactivate Python venv
deactivate

# Run Prisma studio
npx prisma studio

# Reset database (WARNING: deletes all data)
npx prisma migrate reset

# Check Python service API docs
# Go to: http://localhost:8000/docs

# Build for production
npm run build

# Start production server
npm start
```

---

---

# Common Issues & Fixes

### Issue: `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY missing`
**Fix:** Make sure `.env.local` exists (not `.env`) and has all Clerk keys filled in.

### Issue: `DATABASE_URL` connection error
**Fix:**
- Make sure NeonDB database is created and not paused (free tier auto-pauses)
- Copy the connection string exactly from NeonDB dashboard
- Make sure `?sslmode=require` is at the end of the URL

### Issue: Gemini API error `400 Bad Request`
**Fix:**
- Verify your Gemini API key is valid at https://aistudio.google.com
- Check you're using `gemini-1.5-flash` (free tier model)
- Make sure GEMINI_API_KEY is set in both `.env.local` AND `python/.env`

### Issue: `Cannot find module 'autoprefixer'`
**Fix:**
```bash
npm install autoprefixer --save-dev --legacy-peer-deps
```

### Issue: Python `ModuleNotFoundError`
**Fix:** Make sure virtual environment is activated before running uvicorn:
- Ubuntu: `source venv/bin/activate`
- Windows: `venv\Scripts\activate`

### Issue: Clerk webhook not firing (users not saved to DB)
**Fix:**
- Make sure ngrok is running and URL is correct in Clerk dashboard
- Check `CLERK_WEBHOOK_SECRET` matches the one in Clerk dashboard
- Ensure webhook events `user.created`, `user.updated`, `user.deleted` are selected

### Issue: Images not showing after upload
**Fix:** Make sure the `public/uploads/` directory exists:
```bash
mkdir -p public/uploads    # Ubuntu
# Windows:
mkdir public\uploads
```

### Issue: `Port 3000 already in use`
```bash
# Ubuntu
sudo kill -9 $(sudo lsof -t -i:3000)
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

### Issue: Python service not connecting from Next.js
**Fix:** Make sure `AI_SERVICE_URL=http://localhost:8000` is in `.env.local` and the Python uvicorn server is running.

---

# Production Deployment (Optional)

### Deploy Next.js to Vercel
```bash
npm install -g vercel
vercel login
vercel --prod
```
- Add all `.env.local` variables to Vercel project settings
- Update `NEXT_PUBLIC_APP_URL` to your Vercel URL
- Update Clerk webhook URL to your Vercel URL

### Deploy Python Service to Railway / Render
Both support Docker deployments for free:
```bash
# Build Docker image
cd python
docker build -t urban-governance-ai .
docker run -p 8000:8000 -e GEMINI_API_KEY=your_key urban-governance-ai
```

---

# Quick Reference

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Next.js dev server (port 3000) |
| `npm run build` | Build for production |
| `npm start` | Start production server |
| `npx prisma studio` | Open database GUI (port 5555) |
| `npx prisma db push` | Sync schema changes to database |
| `npx prisma generate` | Regenerate Prisma client after schema changes |
| `uvicorn main:app --reload --port 8000` | Start Python AI service |
| `source venv/bin/activate` (Ubuntu) | Activate Python venv |
| `venv\Scripts\activate` (Windows) | Activate Python venv |
| `deactivate` | Deactivate Python venv |

---

*For any issues, check the browser console (F12) and terminal output for error details.*
