 
# Flask App with MySQL Docker Setups

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

## Prerequisites

Before you begin, make sure you have the following installed:

- Docker
- Git (optional, for cloning the repository)

## Setup

1. Clone this repository (if you haven't already):

   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   ```

2. Navigate to the project directory:

   ```bash
   cd your-repo-name
   ```

3. Create a `.env` file in the project directory to store your MySQL environment variables:

   ```bash
   touch .env
   ```

4. Open the `.env` file and add your MySQL configuration:

   ```
   MYSQL_HOST=mysql
   MYSQL_USER=your_username
   MYSQL_PASSWORD=your_password
   MYSQL_DB=your_database
   ```

## Usage

1. Start the containers using Docker Compose:

   ```bash
   docker-compose up --build
   ```

2. Access the Flask app in your web browser:

   - Frontend: http://localhost
   - Backend: http://localhost:5000

3. Create the `messages` table in your MySQL database:

   - Use a MySQL client or tool (e.g., phpMyAdmin) to execute the following SQL commands:
   
     ```sql
     CREATE TABLE messages (
         id INT AUTO_INCREMENT PRIMARY KEY,
         message TEXT
     );
     ```

4. Interact with the app:

   - Visit http://localhost to see the frontend. You can submit new messages using the form.
   - Visit http://localhost:5000/insert_sql to insert a message directly into the `messages` table via an SQL query.

## Cleaning Up

To stop and remove the Docker containers, press `Ctrl+C` in the terminal where the containers are running, or use the following command:

```bash
docker-compose down
```

## To run this two-tier application using  without docker-compose

- First create a docker image from Dockerfile
```bash
docker build -t flaskapp .
```

- Now, make sure that you have created a network using following command
```bash
docker network create twotier
```

- Attach both the containers in the same network, so that they can communicate with each other

i) MySQL container 
```bash
docker run -d \
    --name mysql \
    -v mysql-data:/var/lib/mysql \
    --network=twotier \
    -e MYSQL_DATABASE=mydb \
    -e MYSQL_ROOT_PASSWORD=admin \
    -p 3306:3306 \
    mysql:5.7

```
ii) Backend container
```bash
docker run -d \
    --name flaskapp \
    --network=twotier \
    -e MYSQL_HOST=mysql \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=admin \
    -e MYSQL_DB=mydb \
    -p 5000:5000 \
    flaskapp:latest

```

## Notes

- Make sure to replace placeholders (e.g., `your_username`, `your_password`, `your_database`) with your actual MySQL configuration.

- This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

- Be cautious when executing SQL queries directly. Validate and sanitize user inputs to prevent vulnerabilities like SQL injection.

- If you encounter issues, check Docker logs and error messages for troubleshooting.


# 2-Tier Application Deployment with Kubernetes

This guide explains how to deploy a 2-tier Flask application with MySQL using Kubernetes (Minikube).

## Prerequisites

Ensure that you have the following installed on your Ubuntu machine:
- Docker
- kubectl (Kubernetes CLI)
- Minikube

## Step 1: Install Docker

Update your package list and install Docker:

```bash
sudo apt update
sudo apt install docker.io
```

## Step 2: Install `kubectl`

Download the latest stable release of `kubectl`:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Verify the download:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

Install `kubectl`:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify the installation:

```bash
kubectl version --client
```

## Step 3: Install Minikube

Download and install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

## Step 4: Setup Docker Permissions

Give the Docker permissions to the current user:

```bash
sudo chmod 777 /
sudo usermod -aG docker $USER
```

Log out and log back in to apply the Docker group changes.

## Step 5: Start Minikube

Start Minikube and check its status:

```bash
minikube start
minikube status
kubectl cluster-info
```

## Step 6: Deploy Kubernetes Manifests

Once Minikube is running, apply the Kubernetes manifests one by one to deploy the application and MySQL.

### Deploy the Flask Application

```bash
kubectl apply -f two-tier-app-pod.yml
kubectl apply -f two-tier-app-deployment.yml
kubectl apply -f two-tier-app-svc.yml
```

### Deploy MySQL

```bash
kubectl apply -f mysql-deployment.yml
kubectl apply -f mysql-svc.yml
kubectl apply -f mysql-pv.yml
kubectl apply -f mysql-pvc.yml
```

## Step 7: Update Environment Variables

You need to configure the MySQL host in your `two-tier-app-deployment.yml` file.

First, retrieve the MySQL ClusterIP by running:

```bash
kubectl get svc
```

Locate the `ClusterIP` of the MySQL service and update the `two-tier-app-deployment.yml`:

```yaml
- name: MYSQL_HOST
  value: "your-sql-cluster-ip"
```

Once updated, reapply the deployment:

```bash
kubectl apply -f two-tier-app-deployment.yml
```

## Conclusion

You have successfully deployed a 2-tier Flask application with MySQL using Kubernetes. You can now access the application using the NodePort service created for the Flask app.

```bash
kubectl get svc
```

Look for the `NodePort` to access the app via your Minikube instance's IP and the assigned port.



## Helm Commands for Deployment 2 tier app

### 1. Create a Helm Chart
To create a Helm chart for your application, use the following command:
```bash
helm create <app-name>
```

### 2. View All Manifest Files
To display the generated Kubernetes manifest files for the Helm chart:
```bash
helm template <chart-name>
```

### 3. Package the Helm Chart
To package the Helm chart into a `.tgz` file:
```bash
helm package <chart-name>
```

### 4. Deploy the Helm Chart
To deploy the packaged Helm chart:
```bash
helm install <release-name> ./<chart-name>
```

### 5. List Deployed Releases
To view all deployed releases in Helm:
```bash
helm list
```

### Example:
```bash
helm create my-app
helm template my-app
helm package mysql-chart
helm install mysql-chart ./mysql-chart
helm list
```


