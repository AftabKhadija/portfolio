## Appendix: 

A Detailed Guide for Using Docker in Your Projects
Docker is a powerful tool that simplifies the process of building, deploying, and running applications by using containers. Containers allow you to package your application and its dependencies together, ensuring it runs the same across different environments, from development to production. This guide provides step-by-step instructions on how to use Docker to containerize a project and organize services like the frontend, backend, and database, making it easier to deploy and maintain.
This guide assumes a scenario similar to the project structure we shared, where the project consists of a React.js frontend, a Node.js backend, and a MySQL database.
Part 1: Understanding the basic concepts of Docker
Docker Image: A template that includes your application and its dependencies. It’s a snapshot of everything needed to run your app.
Docker Container: A running instance of a Docker image. It isolates the application from the host system.
Dockerfile: A script that defines how the Docker image is built. It includes instructions for installing dependencies and running the application.
docker-compose.yml: A configuration file that defines and runs multi-container Docker applications. It helps manage multiple containers (like your frontend, backend, and database) as a single service.
Volumes: Storage spaces that allow containers to persist data (e.g., for a database).
Networks: Virtual networks that allow containers to communicate with each other.
Part 2: Setting Up Docker
1. Install Docker
Windows / macOS: Download and install Docker Desktop from docker.com.
2. Verify Docker Installation
Run the following command to verify Docker is installed and running correctly:
docker --version
Part 3: Setting Up Your Project Structure
For this guide, we’ll assume the following project structure:
/your-project
   /frontend
      /project-frontend
         - Dockerfile
         - package.json
         - src/ (React.js files)
   /backend
      /project-backend
         - Dockerfile
         - package.json
         - server.js (Node.js API)
   - schema.sql
   - docker-compose.yml

Here, we have separate directories for the frontend, backend, and a root-level file schema.sql to initialize the MySQL database. The docker-compose.yml is used for the entire project.
Part 4: Creating Dockerfiles
1. Dockerfile for Frontend (React.js)
A Dockerfile defines how to build your Docker image for the React frontend.
Navigate to the frontend directory:
cd your-project/frontend/project-frontend

Create a file named Dockerfile with the following code:


# Use an official Node.js image as the base
FROM node:18

# Set the working directory inside the container
WORKDIR /app

# Copy the package.json file and install dependencies
COPY package.json ./
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the port the app will run on
EXPOSE 3000

# Start the React app
CMD ["npm", "run", "dev"]
2. Dockerfile for Backend (Node.js + Express)
Navigate to the backend directory:
cd your-project/backend/project-backend

Create a file named Dockerfile with the following code:
# Use the official Node.js image
FROM node:18

# Set the working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json ./
RUN npm install

# Copy the rest of the app files
COPY . .

# Expose the port for the API
EXPOSE 5000

# Start the Node.js app
CMD ["node", "server.js"]
3. Database Initialization Script (schema.sql)
In the root of your project, create a schema.sql file to define your database schema. This script will automatically initialize the database when the container starts.
Part 5: Setting Up docker-compose.yml
docker-compose.yml is used to orchestrate your entire application, managing the frontend, backend, and database containers.
In the root of your project (/your-project), create a docker-compose.yml file:
version: '3.9'

services:
  frontend:
    build: 
      context: ./frontend/project-frontend
      dockerfile: Dockerfile
    container_name: frontend
    restart: always
    ports:
      - "3000:3000"
    environment:
      VITE_HOST: localhost
      VITE_PORT: 5000
    networks:
     - network
    depends_on:
      - backend

  backend:
    build:
      context: ./backend/project-backend
      dockerfile: Dockerfile
    container_name: backend
    ports:
      - "5000:5000"
    environment:
      - MYSQL_HOST=db
      - MYSQL_USER= // insert here
      - MYSQL_PASSWORD= // insert here
      - MYSQL_DATABASE= // insert here
      - PORT=5000
      - JWT_SECRET=jwtSecret
      - JWT_LIFETIME=1d
      - NODE_ENV=production
    depends_on:
      - db
    networks:
     - network

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: // insert here
      MYSQL_DATABASE: // insert here
    volumes:
      - db_data:/var/lib/mysql
      - ./schema.sql:/docker-entrypoint-initdb.d/schema.sql
    ports:
      - "3306:3306"
    networks:
     - network

networks:
  network:

volumes:
  db_data:

Explanation of the docker-compose.yml
Frontend Service:
build: Defines the path to the frontend directory and Dockerfile.
ports: Exposes port 3000 for the React app.
environment: Passes environment variables to configure Vite’s host and backend connection.
depends_on: Ensures that the backend starts before the frontend.
Backend Service:
build: Defines the path to the backend directory and Dockerfile.
ports: Exposes port 5000 for the Node.js API.
environment: Sets database credentials, JWT settings, and other necessary backend configuration.
depends_on: Ensures the database starts before the backend.
Database Service:
MySQL image is used.
Sets up environment variables like MYSQL_ROOT_PASSWORD and MYSQL_DATABASE.
Volumes:
db_data: Persists the database data.
schema.sql: Initializes the database with the schema file.
Networks: All services are connected to a shared network for internal communication.
Part 6: Building and Running the Docker Containers
Build and Start Containers
Navigate to the root of your project (/your-project).
Run the following command to build and start the containers: docker-compose up --build
This will build Docker images for your frontend, backend, and MySQL services. 
Accessing the Application
Frontend: Visit http://localhost:3000 to access the React frontend.
Backend API: The API will be available at http://localhost:5000.
Database: MySQL will be running on port 3306, and you can connect using a MySQL client.
Stop the Containers
To stop the running containers, use: docker-compose down
This stops and removes the containers.
Part 7: Debugging and Troubleshooting
Check Logs: If something isn’t working, check the logs for errors: docker-compose logs
Port Conflicts: If you encounter port conflicts (e.g., MySQL is already running on port 3306), you can modify the ports in the docker-compose.yml: 
ports:
  - "3307:3306"
Rebuilding Containers: If you make changes to the Dockerfile, rebuild the images:
docker-compose up --build
Database Persistence: The db_data volume ensures the database persists between restarts. If you want a fresh database, remove the volume:
docker-compose down --volumes
Part 8: Deploying to the Cloud (Optional)
Once Dockerized, you can deploy your application to cloud platforms like:
Amazon ECS (Elastic Container Service)
Google Cloud Run
Microsoft Azure App Services
You can either build your images locally and push them to DockerHub or use the cloud provider’s CI/CD pipelines.
Conclusion
Docker simplifies the process of building, running, and deploying applications by containerizing them. This setup ensures that your project can be run consistently across different environments, making it easier to collaborate, develop, and deploy.

