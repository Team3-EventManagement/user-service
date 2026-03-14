# User Service Microservice

A production-ready microservice for managing users in an Event Management. Built with FastAPI, SQLAlchemy, MySQL, and JWT authentication.

## Project Structure
```text
userservice/
├ app/
│   ├ main.py
│   ├ routes.py
│   ├ models.py
│   ├ schemas.py
│   └ database.py
├ requirements.txt
├ Dockerfile
├ README.md
└ k8s/
    ├ deployment.yaml
    └ service.yaml
```

## Environment Variables
- `DB_USER` (Default: `root`)
- `DB_PASSWORD` (Default: `rootpassword`)
- `DB_HOST` (Default: `localhost`)
- `DB_PORT` (Default: `3306`)
- `DB_NAME` (Default: `user_db`)
- `JWT_SECRET` (Default: `supersecretkey`)
- `JWT_EXPIRE_MINUTES` (Default: `30`)

## Local Testing Instructions

### 1. Run MySQL Locally
You can start a local MySQL instance using Docker:
```bash
docker run --name mysql-local -e MYSQL_ROOT_PASSWORD=rootpassword -e MYSQL_DATABASE=user_db -p 3306:3306 -d mysql:8.0
```

### 2. Setup Python Environment
Create a virtual environment and install dependencies:
```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

pip install -r requirements.txt
```

### 3. Run FastAPI Server
Start the development server:
```bash
uvicorn app.main:app --reload
```

## API Documentation (Testing endpoints)

### 1. Register User
```bash
curl -X 'POST' \
  'http://localhost:8000/users/register' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "Mahesh",
  "email": "mahesh@email.com",
  "password": "password123",
  "role": "attendee"
}'
```

### 2. Login
```bash
curl -X 'POST' \
  'http://localhost:8000/users/login' \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "mahesh@email.com",
  "password": "password123"
}'
```
This returns a JWT token (e.g., `eyJhbG...`).

### 3. Get Profile (Protected)
Requires the JWT token:
```bash
curl -X 'GET' \
  'http://localhost:8000/users/profile' \
  -H 'Authorization: Bearer <YOUR_JWT_TOKEN>'
```

### 4. Update Profile (Protected)
```bash
curl -X 'PUT' \
  'http://localhost:8000/users/profile' \
  -H 'Authorization: Bearer <YOUR_JWT_TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "Mahesh Updated",
  "role": "organizer"
}'
```

You can also test the APIs securely using the built-in Swagger UI at: `http://localhost:8000/docs`.

## Docker Testing Instructions

### 1. Start MySQL Database
First, establish a network so the containers can talk to each other:
```bash
docker network create my-network
docker run --name mysql-db --network my-network -e MYSQL_ROOT_PASSWORD=rootpassword -e MYSQL_DATABASE=user_db -d mysql:8.0
```

### 2. Build the Service Image
Build the Docker image for the User Service:
```bash
docker build -t userservice .
```

### 3. Run the Container
Run the application container, connecting it to the database over the network and passing the environment variable:
```bash
docker run -d --name userservice-app --network my-network -p 8000:8000 -e DB_HOST=mysql-db userservice
```

### 4. Verify Endpoints
Try fetching the root health endpoint to check if the app is alive:
```bash
curl http://localhost:8000/
```
You should get `{"status": "User Service is running!"}`. Then follow the local testing CURL snippets to test routes!

## Kubernetes Deployment Steps

Ensure you create the necessary config maps or secrets first:
```bash
kubectl create secret generic db-secrets --from-literal=username=root --from-literal=password=rootpassword
kubectl create secret generic jwt-secrets --from-literal=secret-key=my_jwt_super_secret_key
```

Then apply the deployment and service files:
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

The service will be available to other cluster resources at `http://userservice-service:80` and instances will scale up according to the replica definition!
