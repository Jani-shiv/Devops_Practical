# Practical 12: Full Stack CI/CD Pipeline using React + Node.js + MongoDB + Docker + GitHub Actions + AWS EC2

## Objective
Build and deploy a complete, fully working Full-Stack application (React frontend + Node.js/Express backend + MongoDB database) orchestrated via Docker Compose and deployed automatically to AWS EC2 using GitHub Actions.

---

## Architecture & Security Principles
1. **Isolated Database**: MongoDB is kept internal to the Docker network and does **not** expose port `27017` to the public internet.
2. **Reverse Proxy Architecture**: Nginx inside the React container serves static production files and proxies `/api/*` calls directly to the internal Node.js backend container (`server:5000`).
3. **Resilient Backend**: The Node.js application implements retry logic with exponential backoff when connecting to MongoDB, and Docker Compose utilizes healthchecks to ensure service readiness before starting dependents.
4. **Modern Docker Compose**: Uses the standard `docker compose` plugin rather than outdated standalone binaries.
5. **Immutable Image Tags**: Every build produces images tagged with the short Git commit SHA for reliable rollbacks.

---

## Directory Structure
```text
q12-fullstack-app/
├── client/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   ├── public/
│   │   └── index.html
│   └── src/
│       ├── App.js
│       └── index.js
├── server/
│   ├── Dockerfile
│   ├── package.json
│   └── index.js
├── docker-compose.yml
└── .github/
    └── workflows/
        └── fullstack-deploy.yml
```

---

## Application Source Code

### 1. Server Application (`server/`)

#### `server/package.json`
```json
{
  "name": "fullstack-server",
  "version": "1.0.0",
  "description": "Node.js Express MongoDB API",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.19.2",
    "mongoose": "^8.3.1"
  }
}
```

#### `server/index.js`
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 5000;
const MONGO_URI = process.env.MONGO_URI || 'mongodb://mongodb:27017/appdb';

app.use(cors());
app.use(express.json());

// Define Mongoose Schema
const ItemSchema = new mongoose.Schema({
  name: { type: String, required: true },
  createdAt: { type: Date, default: Date.now }
});
const Item = mongoose.model('Item', ItemSchema);

// Connection with Retry Logic
const connectWithRetry = () => {
  console.log('Attempting MongoDB connection...');
  mongoose.connect(MONGO_URI, { serverSelectionTimeoutMS: 5000 })
    .then(() => console.log('Successfully connected to MongoDB!'))
    .catch((err) => {
      console.error('MongoDB connection error, retrying in 5 seconds...', err.message);
      setTimeout(connectWithRetry, 5000);
    });
};
connectWithRetry();

// Health Check Endpoint
app.get('/api/health', (req, res) => {
  const dbStatus = mongoose.connection.readyState === 1 ? 'Connected' : 'Disconnected';
  res.status(200).json({ status: 'OK', database: dbStatus, timestamp: new Date() });
});

// Get all items
app.get('/api/items', async (req, res) => {
  try {
    const items = await Item.find().sort({ createdAt: -1 });
    res.json(items);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Add new item
app.post('/api/items', async (req, res) => {
  try {
    const newItem = new Item({ name: req.body.name || 'Sample DevOps Item' });
    const saved = await newItem.save();
    res.status(201).json(saved);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server listening on port ${PORT}`);
});
```

#### `server/Dockerfile`
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 5000
HEALTHCHECK --interval=15s --timeout=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:5000/api/health || exit 1
CMD ["node", "index.js"]
```

---

### 2. Client Application (`client/`)

#### `client/package.json`
```json
{
  "name": "fullstack-client",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

#### `client/public/index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>DevOps Full-Stack CI/CD</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

#### `client/src/index.js`
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

#### `client/src/App.js`
```javascript
import React, { useState, useEffect } from 'react';

function App() {
  const [items, setItems] = useState([]);
  const [name, setName] = useState('');
  const [health, setHealth] = useState({ status: 'Loading...', database: 'Checking...' });

  const fetchItems = () => {
    fetch('/api/items')
      .then(res => res.json())
      .then(data => setItems(data))
      .catch(err => console.error('Error fetching items:', err));
  };

  const checkHealth = () => {
    fetch('/api/health')
      .then(res => res.json())
      .then(data => setHealth(data))
      .catch(() => setHealth({ status: 'Error', database: 'Unreachable' }));
  };

  useEffect(() => {
    fetchItems();
    checkHealth();
  }, []);

  const handleAddItem = (e) => {
    e.preventDefault();
    if (!name.trim()) return;

    fetch('/api/items', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name })
    })
      .then(res => res.json())
      .then(() => {
        setName('');
        fetchItems();
      });
  };

  return (
    <div style={{ fontFamily: 'Arial, sans-serif', maxWidth: 650, margin: '40px auto', padding: '0 20px' }}>
      <h2>Full-Stack DevOps Deployment (React + Node.js + MongoDB)</h2>
      <div style={{ background: '#f1f5f9', padding: '15px', borderRadius: '8px', marginBottom: '20px' }}>
        <strong>Backend Status:</strong> {health.status} | <strong>MongoDB:</strong> {health.database}
      </div>

      <form onSubmit={handleAddItem} style={{ display: 'flex', gap: '10px', marginBottom: '20px' }}>
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Enter item name..."
          style={{ flex: 1, padding: '10px', borderRadius: '4px', border: '1px solid #ccc' }}
        />
        <button type="submit" style={{ padding: '10px 20px', background: '#0284c7', color: 'white', border: 'none', borderRadius: '4px', cursor: 'pointer' }}>
          Add Item
        </button>
      </form>

      <h3>Registered Items:</h3>
      <ul>
        {items.map(item => (
          <li key={item._id} style={{ padding: '6px 0' }}>
            {item.name} <small style={{ color: '#64748b' }}>({new Date(item.createdAt).toLocaleTimeString()})</small>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

#### `client/nginx.conf`
*(Nginx reverse proxy for frontend and API routes)*
```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    # Serve React client app
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Reverse proxy API requests to backend container
    location /api/ {
        proxy_pass http://server:5000/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### `client/Dockerfile`
```dockerfile
# Build Stage
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production Stage with Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### 3. Docker Compose Configuration (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    restart: always
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - fullstack-net

  server:
    image: ${DOCKERHUB_USERNAME}/fullstack-server:${IMAGE_TAG:-latest}
    restart: always
    environment:
      - PORT=5000
      - MONGO_URI=mongodb://mongodb:27017/appdb
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - fullstack-net

  client:
    image: ${DOCKERHUB_USERNAME}/fullstack-client:${IMAGE_TAG:-latest}
    restart: always
    ports:
      - "80:80"
    depends_on:
      - server
    networks:
      - fullstack-net

networks:
  fullstack-net:
    driver: bridge

volumes:
  mongo-data:
```

---

## How To Set Up and Deploy (Step-by-Step)

### Step 1: EC2 Host Preparation
SSH into your Ubuntu EC2 instance:
```bash
ssh -i "my-key.pem" ubuntu@<EC2-PUBLIC-IP>
```
Install Docker with Compose v2 plugin:
```bash
sudo apt update -y
sudo apt install -y docker.io docker-compose-v2
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
sudo systemctl restart docker
mkdir -p /home/ubuntu/app
```

### Step 2: Configure GitHub Repository Secrets
Under **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `DOCKERHUB_USERNAME`: Your Docker Hub account.
- `DOCKERHUB_TOKEN`: Personal Access Token.
- `EC2_HOST`: EC2 Public IP address.
- `EC2_USER`: `ubuntu`
- `EC2_SSH_KEY`: Content of `.pem` private key.

### Step 3: Create GitHub Actions Workflow (`.github/workflows/fullstack-deploy.yml`)
```yaml
name: Full Stack CI/CD Deployment

on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.vars.outputs.sha_short }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Compute Short Git SHA
        id: vars
        run: echo "sha_short=$(echo ${{ github.sha }} | cut -c1-7)" >> $GITHUB_OUTPUT

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Server Image
        uses: docker/build-push-action@v5
        with:
          context: ./server
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-server:${{ steps.vars.outputs.sha_short }}
            ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-server:latest

      - name: Build and Push Client Image
        uses: docker/build-push-action@v5
        with:
          context: ./client
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-client:${{ steps.vars.outputs.sha_short }}
            ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-client:latest

  deploy-ec2:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code for docker-compose file
        uses: actions/checkout@v4

      - name: Transfer docker-compose.yml to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "docker-compose.yml"
          target: "/home/ubuntu/app"

      - name: Pull and Deploy Stack on EC2
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            set -e
            cd /home/ubuntu/app
            export DOCKERHUB_USERNAME="${{ secrets.DOCKERHUB_USERNAME }}"
            export IMAGE_TAG="${{ needs.build-and-push.outputs.image_tag }}"

            echo "Deploying Full-Stack release: ${IMAGE_TAG}"
            sudo docker compose pull
            sudo docker compose down
            sudo docker compose up -d

            echo "Waiting for services to settle..."
            sleep 10
            sudo docker compose ps
```

### Step 4: Verify Full Stack Deployment
1. Open your browser and navigate to `http://<EC2-PUBLIC-IP>`.
2. Verify that **Backend Status** says `OK` and **MongoDB** says `Connected`.
3. Add a new item in the input box and submit to confirm end-to-end database writes and reads work seamlessly through the Nginx reverse proxy.
