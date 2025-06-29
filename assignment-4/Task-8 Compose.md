# 🐳 Week 4 – Docker Compose: Task 8

## 📌 Task: Deploy a Three-Tier Architecture using Docker Compose with Security Best Practices

## 🎯 Objective

In this task, I created a **three-tier architecture** using **Docker Compose** for deploying a **MERN stack application**. The goal was to deploy MongoDB, Node.js backend API, and React frontend in separate containers while ensuring **network isolation** and **persistent storage** using Docker best practices. The architecture includes:

- **MongoDB** as the database, running in a container with persistent storage.
- **Node.js backend API** developed with Express, which connects to MongoDB.
- **Frontend UI** built with React that interacts with the backend API. and displays it in a table.
- **Dockerfiles** for both backend and frontend.
- The backend API image was pushed to **Azure Container Registry (ACR)**, while the frontend React app was pushed to **Docker Hub**.
- All services are managed via **Docker Compose** for easy orchestration.

---

I covered all major Docker concepts and best practices, including:

  - **Dockerfile Creation** – Wrote Dockerfiles for both backend and frontend.
  - **Multi-Stage Builds** – Used multi-stage Docker builds for a lightweight and optimized frontend image.
  - **Image Optimization** – Ensured production-ready images with smaller size and efficient structure.
  - **Secure Networking** – Used custom user-defined networks (csi_net) for internal communication and network isolation.
  - **Service Orchestration** – Managed multiple services (MongoDB, backend, frontend) efficiently using Docker Compose.
  - **Environment Variable Management** – Passed sensitive data securely through environment variables.
  - **Persistent Storage** – Mounted Docker volumes to retain MongoDB data across container restarts.
  - **Image Publishing** – Pushed backend image to Azure Container Registry and frontend image to Docker Hub.

--- 

## Step 1. Developed Backend API with Express and MongoDB

I started by creating a simple Node.js backend using Express. The API connects to MongoDB to fetch employee data and serves it to the frontend.

The basic structure includes:

- Express app setup
- MongoDB connection using mongodb
- API endpoint `/` to return employee records

![Screenshot 2025-06-29 191348](https://github.com/user-attachments/assets/f6b0050e-4608-4470-9360-b85ed128e72d)



### 🔹 Dockerizing the Backend API

I dockerize the backend API (Node.js). I already had a Node.js application ready, and now I needed to create a Dockerfile to build the container image.

### 🔹 Backend Dockerfile

In the backend directory, I created a Dockerfile that defines the container image for my Node.js app:

```bash
FROM node:16-alpine
WORKDIR /app

COPY package*.json ./
RUN npm install
COPY . .

EXPOSE 8080

CMD ["node", "app.js"]
```

![Screenshot 2025-06-29 191401](https://github.com/user-attachments/assets/1c3eda42-c8a2-408a-bd7e-f6e46fece446)


### 🔹 Build Backend Docker Image

I built the Docker image locally with:

```bash
docker build -t csi-backend:v1 .
```

![Screenshot 2025-06-29 191412](https://github.com/user-attachments/assets/b6768a6b-56bb-42e3-b71a-1b1cfa23078a)


### 🔹 Created Azure Container Registry and Login

First, I created an Azure Container Registry named `csiecr` in my Azure subscription to host the Docker images.

![Screenshot 2025-06-29 191432](https://github.com/user-attachments/assets/8c326b42-b1a9-46b5-b757-bdb65a7e1749)


Next, I logged in to my Azure account using:

```bash
az login
```

![Screenshot 2025-06-29 191443](https://github.com/user-attachments/assets/e20a0465-db55-4b6b-970e-42e9202d8817)


### 🔹 Tag and Push the Backend Image to ACR

After successful login, I logged into the Azure Container Registry:

```bash
az acr login --name csiecr
```

I tagged the locally built backend image with the ACR registry login server:

```bash
docker tag csi-backend:v1 csiecr.azurecr.io/csi-backend:v1
```

I pushed the tagged image to the Azure Container Registry:

```bash
docker push csiecr.azurecr.io/csi-backend:v1
```

![Screenshot 2025-06-29 191455](https://github.com/user-attachments/assets/7160944a-3406-4b5d-9515-6fb67cae5de9)


Finally I verified pushed Image in Container Registry Repositories

![Screenshot 2025-06-29 191509](https://github.com/user-attachments/assets/18ab3241-7473-4b51-b835-452ae73104be)


---

## Step 2: Developed the React app for Frontend

I developed the React frontend application to display CSI intern data fetched from the backend API in a clean tabular format.

The React app was built to call the backend API and show employee details dynamically.

![Screenshot 2025-06-29 191525](https://github.com/user-attachments/assets/4579df5c-9685-4621-ab06-102db2b1da9c)


### 🔹 Created Multi-Stage Dockerfile with Nginx Reverse Proxy

To optimize the image size and serve the React app efficiently, I created a multi-stage Dockerfile:

- Stage 1 builds the React app using the official Node.js image.
- Stage 2 serves the static build files using Nginx.
- Nginx is configured to act as a reverse proxy to forward API requests to the backend.
- The final image is smaller and more efficient for production use.

![Screenshot 2025-06-29 191541](https://github.com/user-attachments/assets/f2fa2449-e355-4282-85c5-f839cca3738d)


### 🔹 Build and Tag the Frontend Image

I built the Docker image locally with the following command:

![Screenshot 2025-06-29 191556](https://github.com/user-attachments/assets/83a1ca00-b0be-4b51-9d0e-c7c0f1f32c92)


After building, I tagged the image for pushing to Docker Hub:

![Screenshot 2025-06-29 191607](https://github.com/user-attachments/assets/22f2b526-faee-439a-9026-94e856fe3a51)


### 🔹 Pushed the Frontend Image to Docker Hub

I logged into Docker Hub and pushed the tagged image:

```bash
docker login
docker push mydockerhub/csi-frontend:v1
```

![Screenshot 2025-06-29 191617](https://github.com/user-attachments/assets/c8a91a69-1917-44ca-8e54-93266d403f46)



### 🔹 Verified in DockerHub

Finally, I verified the uploaded image appeared in my Docker Hub repository.

![Screenshot 2025-06-29 191632](https://github.com/user-attachments/assets/d5b04830-3ec8-4038-a6e4-f0f84e2b79db)


---

## Step 3: Setting Up the Docker Compose File

Now that both the backend and frontend images were ready, I created a docker-compose.yml file to orchestrate the entire stack. `[docker-compose-file](./docker-compose.yml)`

![Screenshot 2025-06-29 191645](https://github.com/user-attachments/assets/127f89b2-db97-4bf7-aa41-1b459f68b6d0)


### 🔹 Key Details

#### React Frontend Service:

- The React app is built and served using Nginx inside the container, serving static files on port 80.  
- It fetches employee data from the Node.js backend API at `http://backend:8080/` via the internal Docker network (`csi_net`).  
- The frontend depends on the backend to be running to retrieve and display data in a clean tabular format.

#### Backend Service:

- The **Node.js backend API** connects to **MongoDB**, and database credentials are passed via environment variables.
- It serves data from the **MongoDB database** via an API endpoint (e.g., `GET /`).
- The **backend service** depends on **MongoDB** being up and running first to ensure a successful connection to the database when it starts.

#### MongoDB Service:

- **MongoDB** runs as a service with persistent storage using the `mongo_data` volume.
- The database is securely connected to the **backend** using a custom network (**csi_net**), isolated from the external world.
- **MongoDB** uses the `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` environment variables for authentication.

#### Networks:

- A **custom network** (`csi_net`) is created to isolate all services, ensuring secure communication between the **React frontend**, **backend**, and **MongoDB**.
- This custom network ensures that these services can communicate with each other internally, while not being exposed to the external network unless explicitly required.

#### Volumes:

- The **mongo_data** volume is created to persist **MongoDB** data. This ensures that the data survives even if the MongoDB container is restarted or destroyed.
- This persistent volume ensures the integrity and availability of the database even across container lifecycles.


### 🔹 Security Best Practices

To ensure the security of the application, I implemented the following best practices during the setup and deployment process:

#### Network Isolation

I created a **custom network** (`csi_net`) for all the services to communicate over. This ensures that only the necessary services can interact with each other and that they are not exposed to the external network unless explicitly required. By isolating the services in this manner, I reduced the risk of accidental exposure or unauthorized access to internal services.

#### Environment Variables

To keep sensitive information secure, I passed database credentials and other sensitive data as **environment variables**. This approach avoids hardcoding any sensitive information directly in the codebase, making the project more secure and easier to maintain. Environment variables also allow for easier configuration changes without modifying the code.

#### No default bridge network: 

Created a custom Docker network to control communication between containers and prevent accidental exposure.

#### Persistent Volume for MongoDB

For **MongoDB**, I used Docker volumes to ensure that the database data is **persisted** across container restarts. The `mongo_data` volume ensures that even if the MongoDB container is stopped or removed, the data will not be lost. This is crucial for maintaining the integrity and availability of the database across container lifecycles.

---

## Step 4. Deployed MERN Stack using Docker-Compose

After configuring all the services and ensuring everything is properly set up, deploying the entire stack is straightforward using **Docker Compose**. Here's how I deployed the application:

### 🔹 Deployed with Docker Compose

I used the following command to build and start all the services in the background:

```bash
docker-compose up -d
```

This command Starts all services defined in the docker-compose.yml file in detached mode (-d), so it runs in the background.

![Screenshot 2025-06-29 191702](https://github.com/user-attachments/assets/b9945b27-5714-4e9c-8dd1-41ecd52e7a70)


### 🔹 Confirming All Containers Are Running

 I verified that all three essential containers MongoDB, Backend API, and Frontend were up and running smoothly by executing:

 ```bash
 docker ps
 ```

 The output confirmed that the containers named `csi-mongo`, `csi-backend`, and `csi-frontend` were running without any issues. Each container was correctly assigned and exposed on the intended ports, which reassured me that the initial deployment was successful.

 ![Screenshot 2025-06-29 191713](https://github.com/user-attachments/assets/c251692c-8b9b-4cd0-b952-fdefaa4b6d1a)



### 🔹 Verifying the Custom Docker Network

Next, I ensured the network isolation and inter-container communication were set up properly. I listed all Docker networks:

```bash
docker network ls
```

Here I've seen the custom network csi_net and inspected its details with:

```bash
docker network inspect csi_net
```

This inspection showed that all three containers were connected to the csi_net bridge network, ensuring secure and isolated communication between services.

![Screenshot 2025-06-29 191724](https://github.com/user-attachments/assets/e52c554e-f4ae-44ed-a99c-e12da23345a9)


### 🔹 Checking Persistent Volume for MongoDB

To confirm data persistence, I listed the Docker volumes:

```bash
docker volume ls
```

I found the mongo_data volume created as expected. Inspecting the volume provided details confirming it was properly configured and mounted to the MongoDB container’s data directory, ensuring data durability beyond container lifecycle events.

![Screenshot 2025-06-29 191734](https://github.com/user-attachments/assets/4433a227-61ed-4187-bd0f-899f0de0ac6a)


### 🔹 Ensuring MongoDB Started Successfully

I reviewed the MongoDB container logs:

```bash
docker logs csi-mongo
```
The logs showed a clean startup without errors, confirming that the database service was operational and ready to accept connections.

![Screenshot 2025-06-29 191745](https://github.com/user-attachments/assets/a0698f8e-a9f8-4718-ac0a-990a18ba2f89)


### 🔹 Connecting to MongoDB

To interact with the database, I accessed the MongoDB shell inside the container using the configured credentials:

```bash
docker exec -it csi-mongo mongosh -u admin -p 'csi@123'
```

![Screenshot 2025-06-29 191755](https://github.com/user-attachments/assets/32ac0f34-1e9d-4912-a443-945a9d7f3869)



### 🔹 Validating Backend API Functionality

After confirming MongoDB, I checked the backend container logs:

```bash
docker logs csi-backend
```
The logs indicated the backend API was running without errors successfully running on port 8080.

![Screenshot 2025-06-29 191803](https://github.com/user-attachments/assets/c2cd282f-3678-4598-9aae-4ce0bed2773f)


#### **Internal-Only API Access**
It's important to note that the backend container is not exposed to the host machine or the internet. It does not publish any ports externally, which is intentional for security reasons.

Instead, it communicates only within the internal Docker network `(csi_net)`, where it can be accessed by other services like the frontend container. 

### 🔹 Verifying Frontend Container

I opened the frontend UI in my browser with my machine IP `http://4.240.88.24:80/`

![Screenshot 2025-06-29 191813](https://github.com/user-attachments/assets/09d7f8ec-5c95-4d6a-8902-72c3526c83d8)


At this initial stage, since no data existed in the MongoDB database, the React app loaded successfully but displayed an empty table with no employee records. This confirmed the frontend was correctly connected to the backend API but was reflecting the current empty state of the database.

### 🔹 Adding Data to MongoDB

I connected to the MongoDB container and added employee records as follows:

```bash
docker exec -it csi-mongo mongosh -u admin -p 'csi@123'
```

Inside the MongoDB shell, I created and switched to the csidb database, then inserted multiple employee records:

```bash
db.employees.insertMany([
  { name: "Vikas", csiintern: true, domain: "DevOps", location: "Kakinada, Visakhapatnam" },
  { name: "Lala's Hymavati", csiintern: true, domain: "Cloud Infrastructure", location: "Visakhapatnam" },
  { name: "Prasanthy", csiintern: true, domain: "DevOps", location: "Visakhapatnam" },
  { name: "Greeshma", csiintern: true, domain: "DevOps", location: "Kakinada" },
  { name: "Venkat", csiintern: true, domain: "Python Backend", location: "Tirupati" },
  { name: "Suresh", csiintern: true, domain: "Frontend - React", location: "Kurnool" }
]);
```

To ensure the records were successfully added.

![Screenshot 2025-06-29 191825](https://github.com/user-attachments/assets/e1ca6b27-641e-47bc-b5fb-007aa0ae9b56)


### 🔹 Frontend Displays the Newly Added Data

With data now present in MongoDB, I refreshed the frontend browser `http://4.240.88.24/`

![Screenshot 2025-06-29 191840](https://github.com/user-attachments/assets/f6c2bd8d-a595-40c3-91da-3127b21e7ea6)


The React frontend immediately displayed a neatly formatted table containing the employee records fetched via the backend API.


### 🔹 Updating Employee Records in MongoDB

After initially inserting the first set of employee records, I decided to update the dataset by adding new employees and removing that ensure everything works smoothly.

```bash
db.employees.deleteOne({ name: "Venkat" })
```

This command removed the employee named Venkat from the collection.

Next, I added new employee records to reflect the latest team structure and roles:

```bash
db.employees.insertMany([
  { name: "Anjali", csiintern: false, domain: "QA Testing", location: "Hyderabad" },
  { name: "Ravi Kumar", csiintern: false, domain: "Backend - Node.js", location: "Vizag" },
  { name: "Meena", csiintern: true, domain: "UI/UX Design", location: "Bangalore" },
  { name: "Sanjay", csiintern: false, domain: "Cloud Infrastructure", location: "Hyderabad" },
  { name: "Sneha", csiintern: true, domain: "Frontend - React", location: "Chennai" },
  { name: "Rahul", csiintern: false, domain: "DevOps", location: "Delhi" }
])
```

![Screenshot 2025-06-29 191853](https://github.com/user-attachments/assets/3515f85a-cb20-40d4-9fb0-769aa0bd51b9)


### 🔹 Verifying Frontend UI After Data Update

With the MongoDB records updated, I returned to the browser and refreshed the frontend UI at `http://4.240.88.24:80/`

![Screenshot 2025-06-29 191906](https://github.com/user-attachments/assets/1d2c1edc-1523-47d6-8390-3956b5485d1c)


The React application reloaded successfully and displayed the newly added employee records in a clean tabular layout. The previously removed entry for Venkat no longer appeared, confirming that the frontend was correctly reflecting the updated database state via the backend API.


**This confirmed the entire data flow—from database to backend to frontend—was functioning perfectly and data changes in MongoDB were reflected in real-time on the UI.**

---

## Step 5: Cleanup Docker Environment

After finishing the deployment and testing, I cleaned up my Docker environment to free up system resources and remove the containers, networks, and volumes created during this task.

### 🔹 Stopped and Removed All Containers, Networks, and Volumes
I ran the following command to stop all the running services and remove the associated containers, the custom network, and the persistent MongoDB volume:

```bash
docker-compose down --volumes
```

This ensured that:
 - All the containers (csi-mongo, csi-backend, and csi-frontend) were stopped and removed.
 - The custom network csi_net created for service isolation was deleted.
 - The persistent volume mongo_data used by MongoDB was removed, cleaning up the stored database data.

![Screenshot 2025-06-29 191917](https://github.com/user-attachments/assets/74d0df1c-3c05-42f2-8c97-6b9d91362452)


---

## Conclusion

This project successfully demonstrated deploying a secure, scalable MERN stack using Docker Compose with best practices like network isolation and persistent storage. The seamless integration between MongoDB, backend API, and React frontend ensured smooth data flow and easy management. Cleaning up the environment afterward kept the system ready for future use. Overall, this setup provides a strong foundation for containerized full-stack applications.

---
