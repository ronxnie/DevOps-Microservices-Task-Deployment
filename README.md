# Microservices-Task

## Overview
This document provides details on testing various services after running the `docker-compose` file. These services include User, Product, Order, and Gateway Services. Each service has its own endpoints for testing purposes.

---

## Services and Endpoints

### **User Service**
- **Base URL:** `http://localhost:3000`
- **Endpoints:**
  - **List Users:**  
    ```
    curl http://localhost:3000/users
    ```
    Or open in your browser: [http://localhost:3000/users](http://localhost:3000/users)

---

### **Product Service**
- **Base URL:** `http://localhost:3001`
- **Endpoints:**
  - **List Products:**  
    ```
    curl http://localhost:3001/products
    ```
    Or open in your browser: [http://localhost:3001/products](http://localhost:3001/products)

---

### **Order Service**
- **Base URL:** `http://localhost:3002`
- **Endpoints:**
  - **List Orders:**  
    ```
    curl http://localhost:3002/orders
    ```
    Or open in your browser: [http://localhost:3002/orders](http://localhost:3002/orders)

---

### **Gateway Service**
- **Base URL:** `http://localhost:3003/api`
- **Endpoints:**
  - **Users:**  
    ```
    curl http://localhost:3003/api/users
    ```
  - **Products:**  
    ```
    curl http://localhost:3003/api/products
    ```
  - **Orders:**  
    ```
    curl http://localhost:3003/api/orders
    ```

---

## Instructions
1. Start all services using the `docker-compose` file:
   ```
   docker-compose up
   ```
2. Once the services are running, use the above endpoints to verify the functionality.

Happy testing!

---

# Deployment Steps

## 1. Prerequisites Check

Install the following and verify the versions:

- **Git**: `git --version`
- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux): `docker --version`
- **Docker Compose** (included in Docker Desktop): `docker compose version`

![Pre-Requisites Check](/Deployment_Process/screenshots/Step1_Pre_Requisites_Check/Pre_Requisites_Check.png)
-- Pre-Requisites Check --

---

## 2. Project Structure

```
submission/
├── user-service/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── (source code)
├── product-service/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── (source code)
├── gateway-service/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── (source code)
├── order-service/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── (source code)
├── docker-compose.yml
└── README.md
```

## 3. Step-by-Step Setup

### Step 1: Look at each service

Open each service folder and note:

- the **entry file** (`app.js`)
- the **port** the app listens on
- the `scripts.start` entry in `package.json`

```bash
cat user-service/package.json
```
![Gateway-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Gateway_Service_Package_JSON.png)
-- Gateway-Service Package.json File --

![Order-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Order_Service_Package_JSON.png)
-- Order-Service Package.json File --

![Product-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Product_Service_Package_JSON.png)
-- Product-Service Package.json File --

![User-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/User_Service_Package_JSON.png)
-- User-Service Package.json File --

### Step 3: Add a `.dockerignore` to each service

Create `user-service/.dockerignore`, `product-service/.dockerignore` and `gateway-service/.dockerignore` with:

```
node_modules
npm-debug.log
.git
.env
```
![dockerignore File Creation](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/DockerIgnore_File_Generated_For_Each_Microservice.png)
-- dockerignore File Creation --

![dockerignore File Created For Each Service](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/DockerIgnore_File_Folder_Structure.png)
-- dockerignore File Created For Each Service --

This stops your local `node_modules` from being copied into the image.

### Step 4: Create the Dockerfiles

Create a `Dockerfile` inside each service folder (see below).

**`user-service/Dockerfile`**

```dockerfile
# Lightweight official Node.js image
FROM node:18-alpine

# Working directory inside the container
WORKDIR /app

# Copy package files first (better build caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the source code
COPY . .

# Document the port this service uses
EXPOSE 3000

# Start the service
CMD ["npm", "start"]
```

**`product-service/Dockerfile`**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001
CMD ["npm", "start"]
```

**`gateway-service/Dockerfile`**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3003
CMD ["npm", "start"]
```

> If a service's `package.json` has no `start` script, use `CMD ["node", "index.js"]` instead (replace `index.js` with that service's real entry file).

### Step 5: Make the gateway talk to the other services

Inside Docker, `localhost` means *the container itself*, not your computer. If the gateway code calls `http://localhost:3000`, it will fail in Docker. Change those URLs to the **service names**:

| Instead of | Use |
|---|---|
| `http://localhost:3000` | `http://user-service:3000` |
| `http://localhost:3001` | `http://product-service:3001` |

The best approach is to read them from environment variables in the gateway code:

```js
const USER_SERVICE_URL = process.env.USER_SERVICE_URL || 'http://localhost:3000';
const PRODUCT_SERVICE_URL = process.env.PRODUCT_SERVICE_URL || 'http://localhost:3001';
```

The compose file below sets these variables for you.

### Step 6: Create `docker-compose.yml`

In the project root (next to the three service folders):

```yaml
services:
  user-service:
    build: ./user-service
    container_name: user-service
    ports:
      - "3000:3000"
    networks:
      - app-network

  product-service:
    build: ./product-service
    container_name: product-service
    ports:
      - "3001:3001"
    networks:
      - app-network

  gateway-service:
    build: ./gateway-service
    container_name: gateway-service
    ports:
      - "3003:3003"
    environment:
      - USER_SERVICE_URL=http://user-service:3000
      - PRODUCT_SERVICE_URL=http://product-service:3001
    depends_on:
      - user-service
      - product-service
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

---
