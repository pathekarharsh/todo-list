# ⚙️ Task 2 — Full-Stack App (Node.js + Express)
## Deploy on: AWS EC2

A Task Manager application with a REST API backend and a browser frontend.

---

## 📁 Project Structure
```
fullstack-app/
├── server.js          ← Express app (entry point)
├── package.json       ← Node.js dependencies
├── routes/
│   └── tasks.js       ← CRUD API routes
└── public/
    ├── index.html     ← Frontend UI
    ├── style.css      ← Styles
    └── app.js         ← Frontend JavaScript
```

---

## 🔌 API Endpoints
| Method | Route | Description |
|---|---|---|
| GET | `/health` | Server health check |
| GET | `/api/tasks` | Get all tasks |
| POST | `/api/tasks` | Create a task |
| PUT | `/api/tasks/:id` | Update a task |
| DELETE | `/api/tasks/:id` | Delete a task |

---

## 🚀 Deployment Steps — AWS EC2

### Step 1: Launch an EC2 Instance
1. Go to **AWS Console → EC2 → Launch Instance**
2. Name: `task-manager-server`
3. AMI: **Ubuntu Server 22.04 LTS** (Free tier eligible)
4. Instance type: **t2.micro** (Free tier)
5. Key pair: Create new → `my-key` → Download `.pem` file ⬇️
6. Network settings → **Allow HTTP traffic from the internet** ✅
7. Click **Launch Instance**

### Step 2: Configure Security Group
1. Go to your instance → **Security** tab → click the Security Group
2. Click **Edit inbound rules** → **Add rule**:
   - Type: **Custom TCP**, Port: `3000`, Source: `0.0.0.0/0`
3. Click **Save rules**

### Step 3: Connect to Your Instance
```bash
# On your local machine (fix key permissions first)
chmod 400 my-key.pem

# Connect via SSH (replace YOUR-PUBLIC-IP)
ssh -i my-key.pem ubuntu@YOUR-PUBLIC-IP
```

### Step 4: Install Node.js on EC2
```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install Node.js 18 (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version   # should show v18.x.x
npm --version
```

### Step 5: Upload Your App to EC2

**Option A — Using SCP (from your local machine):**
```bash
# Upload the entire folder
scp -i my-key.pem -r ./fullstack-app ubuntu@YOUR-PUBLIC-IP:~/
```

**Option B — Using Git (recommended):**
```bash
# On EC2, clone from GitHub
sudo apt install -y git
git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git
cd YOUR-REPO/fullstack-app
```

### Step 6: Run the App
```bash
cd ~/fullstack-app

# Install dependencies
npm install

# Start the server
npm start
```

You should see:
```
🚀  Task Manager API Running
Port : 3000
```

### Step 7: Access Your App
Open your browser and go to:
```
http://YOUR-EC2-PUBLIC-IP:3000
```

Your app is live! 🎉

---

## 🔄 Keep App Running (PM2)
By default, the app stops when you close SSH. Use PM2:

```bash
# Install PM2 globally
sudo npm install -g pm2

# Start app with PM2
pm2 start server.js --name "task-manager"

# Auto-start on reboot
pm2 startup
pm2 save
```

---

## 🌐 Optional: Use Port 80 with Nginx (Reverse Proxy)
```bash
# Install Nginx
sudo apt install -y nginx

# Create config
sudo nano /etc/nginx/sites-available/task-manager
```

Paste this config:
```nginx
server {
    listen 80;
    server_name YOUR-PUBLIC-IP;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable and restart
sudo ln -s /etc/nginx/sites-available/task-manager /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Now access via `http://YOUR-PUBLIC-IP` (no port needed!)

---

## 💡 Key AWS Concepts Used
| Concept | Description |
|---|---|
| **EC2 Instance** | Virtual machine to run your Node.js server |
| **AMI** | Amazon Machine Image — the OS template |
| **Security Group** | Firewall rules for your instance |
| **Key Pair** | SSH credentials to connect to your server |
| **Public IP** | The address to access your app from the internet |
| **Nginx** | Reverse proxy to forward port 80 → 3000 |
| **PM2** | Process manager to keep Node.js running |




## Steps to resolve login issue in github 

1. Download publish profile from azure (web app overview)
2. Copy the key from it.
3. Go to settings of repo in github at bottom find secret and variables
4. Create new repository secret paste the key there and save it.
sample yml file that works:

        # Docs for the Azure Web Apps Deploy action: https://github.com/Azure/webapps-deploy
        # More GitHub Actions for Azure: https://github.com/Azure/actions
        
        name: Build and deploy Node.js app to Azure Web App - deploy-parth
        
        on:
          push:
            branches:
              - main
          workflow_dispatch:
        
        jobs:
          build:
            runs-on: ubuntu-latest
            permissions:
              contents: read #This is required for actions/checkout
        
            steps:
              - uses: actions/checkout@v4
        
              - name: Set up Node.js version
                uses: actions/setup-node@v3
                with:
                  node-version: '22.x'
        
              - name: npm install, build, and test
                run: |
                  npm install
                  npm run build --if-present
                  npm run test --if-present
        
              - name: Upload artifact for deployment job
                uses: actions/upload-artifact@v4
                with:
                  name: node-app
                  path: .
        
          deploy:
            runs-on: ubuntu-latest
            needs: build
            permissions:
              id-token: write #This is required for requesting the JWT
              contents: read #This is required for actions/checkout
        
            steps:
              - name: Download artifact from build job
                uses: actions/download-artifact@v4
                with:
                  name: node-app
              
              - name: Deploy to Azure Web App
                uses: azure/webapps-deploy@v3
                with:
                  app-name: 'deploy-parth'  // change the name
                  publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
                  package: .               // make sure only one . is there


