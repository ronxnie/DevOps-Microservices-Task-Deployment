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

-- Pre-Requisites Check --

![Pre-Requisites Check](/Deployment_Process/screenshots/Step1_Pre_Requisites_Check/Pre_Requisites_Check.png)

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
-- Gateway-Service Package.json File --

![Gateway-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Gateway_Service_Package_JSON.png)

-- Order-Service Package.json File --

![Order-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Order_Service_Package_JSON.png)

-- Product-Service Package.json File --

![Product-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/Product_Service_Package_JSON.png)

-- User-Service Package.json File --

![User-Service Package.json File](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/User_Service_Package_JSON.png)

### Step 2: Add a `.dockerignore` to each service

Create `user-service/.dockerignore`, `product-service/.dockerignore` and `gateway-service/.dockerignore` with:

```
node_modules
npm-debug.log
.git
.env
```
-- dockerignore File Creation --

![dockerignore File Creation](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/DockerIgnore_File_Generated_For_Each_Microservice.png)

-- dockerignore File Created For Each Service --

![dockerignore File Created For Each Service](/Deployment_Process/screenshots/Step2_Step-By-Step_Service_Map/DockerIgnore_File_Folder_Structure.png)

This stops your local `node_modules` from being copied into the image.

### Step 3: Create the Dockerfiles

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

# Start the service (If the file preset starts with a different file name, then replace app.js with the file name present for the service)
CMD ["node", "app.js"]
```
-- User Service Docker File --

![User Service Docker File](/Deployment_Process/screenshots/Step3_Docker_File_Creation/User_Service_Docker_File.png)

**`product-service/Dockerfile`**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001
CMD ["node", "app.js"]
```
-- Product Service Docker File --

![Product Service Docker File](/Deployment_Process/screenshots/Step3_Docker_File_Creation/Product_Service_Docker_File.png)

**`order-service/Dockerfile`**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3002
CMD ["node", "app.js"]
```
-- Order Service Docker File -- 

![Order Service Docker File](/Deployment_Process/screenshots/Step3_Docker_File_Creation/Order_Service_Docker_File.png)

**`gateway-service/Dockerfile`**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3003
CMD ["node", "app.js"]
```
-- Gateway Service Docker File --

![Gateway Service Docker File](/Deployment_Process/screenshots/Step3_Docker_File_Creation/Gateway_Service_Docker_File.png)

> If a service's `package.json` has `start` script, use `CMD ["npm", "start"]` instead.

### Step 4: Create `docker-compose.yml`

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

  order-service:
    build: ./order-service
    container_name: product-service
    ports:
      - "3002:3002"
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
      - ORDER_SERVICE_URL=http://order-service:3002
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```
-- docker-compose.yaml File Creation Complete --

![docker-compose.yaml File Creation Completed](/Deployment_Process/screenshots/Step4_Docker-Compose_File_Creation/docker-compose_File_Creation_1.png)

![docker-compose.yaml File Creation Completed](/Deployment_Process/screenshots/Step4_Docker-Compose_File_Creation/docker-compose_File_Creation_2.png)

---

## 4. Run the Application

From the project root:

```bash
docker compose up --build
```
-- Docker Build Services Mapped To Ports --

![Docker Build Completed and 4 Services Mapped to 4 Different Ports](/Deployment_Process/screenshots/Step5_Docker_Build/Docker-Build_Services_Mapped_Ports.png)

(Older installs: `docker-compose up --build`.)

Wait until you see log lines showing each service listening on its port. To run in the background instead:

```bash
docker compose up --build -d
```
-- Deploy on the Background -- 

![Services Stopped For Deploying in the Background](/Deployment_Process/screenshots/Step5_Docker_Build/Services_Stopped_For_Deploying_In_Background.png)

![Deploy the Services in the Background](/Deployment_Process/screenshots/Step5_Docker_Build/Docker_Services_Deployed_On_Background.png)

Check that all containers are running:

```bash
docker compose ps
```
-- List of Deployed Services in Background --

![List of Deployed Services in Background](/Deployment_Process/screenshots/Step5_Docker_Build/List_of_Deployed_Services.png)

You should see `user-service`, `product-service`, `order-service` and `gateway-service` with status **Up**.

To stop everything:

```bash
docker compose down
```
-- To Stop All Services -- 

![Stopping All Deployed Services](/Deployment_Process/screenshots/Step5_Docker_Build/Stopping_All_Deployed_Services.png)
---


## 5. How to Test Each Service

> The exact routes depend on the provided source code. Check each service's route definitions (for example `app.get('/...')`) and adjust the paths below. A browser works for GET requests; `curl` works in the terminal.

### User Service: http://localhost:3000

```bash
curl http://localhost:3000
curl http://localhost:3000/users
```
-- Local Host 3000 - User Service --

![User Service Running Successfully](/Deployment_Process/screenshots/Step6_Deployment_Testing/User-Service_Health.png)

### Product Service: http://localhost:3001

```bash
curl http://localhost:3001
curl http://localhost:3001/products
```
-- Local Host 3001 - Product Service --

![Product Service Running Successfully](/Deployment_Process/screenshots/Step6_Deployment_Testing/Product-Service_Health.png)

### Order Service: http://localhost:3002

```bash
curl http://localhost:3002
curl http://localhost:3002/orders
```
-- Local Host 3002 - Order Service --

![Order Service Running Successfully](/Deployment_Process/screenshots/Step6_Deployment_Testing/Order-Service_Health.png)

### Gateway Service: http://localhost:3003

```bash
curl http://localhost:3003
curl http://localhost:3003/users
curl http://localhost:3003/products
curl http://localhost:3003/orders
```
-- Local Host 3003 - Gateway Service --

![Gateway Service Running Successfully](/Deployment_Process/screenshots/Step6_Deployment_Testing/Gateway-Service_Health.png)

-- Gateway Service to User Service Mapping -- 

![Gateway Service to User Service Mapping](/Deployment_Process/screenshots/Step6_Deployment_Testing/Gateway_to_User_Mapping.png)

-- Gateway Service to Product Service Mapping --

![Gateway Service to Product Service Mapping](/Deployment_Process/screenshots/Step6_Deployment_Testing/Gateway_to_Product_Mapping.png)

-- Gateway Service to Order Service Mapping --

![Gateway Service to Order Service Mapping](/Deployment_Process/screenshots/Step6_Deployment_Testing/Gateway_to_Order_Mapping.png)

If the gateway returns data from the user and product services, the shared network is working.

### View logs

```bash
docker compose logs -f
docker compose logs gateway-service
```
-- Docker Compose Logs -- 

![Docker Compose Logs Regarding the Working](/Deployment_Process/screenshots/Step6_Deployment_Testing/Logs_of_Docker_Compose.png)


### Expected result

| Check | Expected |
|---|---|
| `docker compose ps` | All 4 services **Up** |
| `localhost:3000` | User Service responds |
| `localhost:3001` | Product Service responds |
| `localhost:3002` | Order Service responds |
| `localhost:3003` | Gateway responds and can reach both services |

---


## 6. Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `Cannot connect to the Docker daemon` | Docker isn't running | Start Docker Desktop (or `sudo systemctl start docker` on Linux) |
| `port is already allocated` / `address already in use` | Another program uses 3000, 3001 or 3003 | Stop it, or change the left side of the mapping, e.g. `"4000:3000"` |
| `npm start` fails / `Missing script: start` | No `start` script in `package.json` | Use `CMD ["node", "app.js"]` with the correct file name |
| `Cannot find module ...` | Dependencies not installed or `node_modules` copied from host | Confirm `.dockerignore` lists `node_modules`, then `docker compose build --no-cache` |
| Gateway gets `ECONNREFUSED 127.0.0.1:3000` | Gateway code still uses `localhost` | Use `http://user-service:3000` and `http://product-service:3001` |
| Service opens but the browser can't connect | App listens only on `127.0.0.1` | Make it listen on all interfaces: `app.listen(PORT, '0.0.0.0')` |
| Changes in code don't appear | Old image is cached | Run `docker compose up --build` |
| Container exits immediately | App crashed on startup | Run `docker compose logs <service-name>` and read the error |
| `version is obsolete` warning | Newer Compose ignores `version:` | Safe to ignore (this file doesn't use it) |

Full reset (removes containers, networks and built images for this project):

```bash
docker compose down --rmi local
docker compose up --build
```
-- Undeploying and Stopping the Container --

![Undeploying and Stopping the Containers Deployed on Docker](/Deployment_Process/screenshots/Step7_Undeploy_The_Background_Deployed_App/Undeploy_The_Deployed_Containers.png)

![Undeploying and Stopping the Non-Background Application](/Deployment_Process/screenshots/Step7_Undeploy_The_Background_Deployed_App/Stopping_Currently_Deployed_Application.png)

---

## 7. Useful Commands

```bash
docker compose up --build        # build and start
docker compose up -d             # start in background
docker compose down              # stop and remove containers/network
docker compose ps                # list services
docker compose logs -f           # follow logs
docker compose restart gateway-service
docker images                    # list images
docker network ls                # list networks
```

---
