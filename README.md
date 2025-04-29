1.Continuous Integration and Deployment of Node.js Application with Jenkins, AWS EC2, and Docker

This project demonstrates how to set up continuous integration and deployment (CI/CD) for a Node.js application using Jenkins, Docker, and AWS EC2.

## Overview

The goal of this project is to build a Node.js application, containerize it using Docker, and automate its deployment using Jenkins. The steps include:

- Setting up an EC2 instance in AWS.
- Installing Docker on the EC2 instance.
- Installing and configuring Jenkins on the EC2 instance.
- Creating a Jenkins pipeline to build and deploy a Node.js application.
- Building a Docker image of the Node.js application and pushing it to Docker Hub.
- Deploying the Docker image to an EC2 instance.
- Verifying the deployment by accessing the application via a web browser.

## Project Structure

The project is structured as follows:

```
.
├── Dockerfile
├── index.js
├── Jenkinsfile
├── package.json
└── README.md
```

- `Dockerfile`: The Dockerfile used to containerize the Node.js application.
- `index.js`: The main file of the Node.js application, serving the "Hello World" message.
- `Jenkinsfile`: The Jenkins pipeline script for building and deploying the Docker image.
- `package.json`: The Node.js application dependencies file.
- `README.md`: This file.

## Prerequisites

Before you begin, make sure you have the following:

- An AWS account and EC2 instance with public IP.
- Docker installed on both the EC2 instance and the local machine (if you're running the application locally).
- Jenkins installed on the EC2 instance or any other server.
- A Docker Hub account (for pushing the Docker image).
  
## Steps to Set Up the Project

### 1. Set Up AWS EC2 Instance

- Launch an EC2 instance with Ubuntu.
- Install Docker and Jenkins on the instance.
- Create a new security group that allows incoming traffic on ports 22 (SSH) and 80 (HTTP).

  ![image](https://github.com/user-attachments/assets/047a0449-74a4-4b9a-bc76-6d35b9f87928)
  ![image](https://github.com/user-attachments/assets/4ea952c4-2aca-4451-8632-27afb49f0290)


  
### 2. Set Up Jenkins

- Install Jenkins on your EC2 instance.
- Configure Jenkins to work with Docker.
- Install necessary Jenkins plugins like Docker, Git, NodeJS, and Pipeline.

  ![image](https://github.com/user-attachments/assets/85adb363-e8c9-4705-9e57-652af7a96de5)


### 3. Create the Node.js Application

Create a simple Node.js application to serve a "Hello World" message:

```js
// index.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### 4. Create the `Dockerfile`

Create a `Dockerfile` to containerize the Node.js application:

```Dockerfile
# Use official Node.js image as base
FROM node:16

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application
COPY . .

# Expose port 3000
EXPOSE 3000

# Command to run the application
CMD ["node", "index.js"]
```

### 5. Create `package.json`

Create a `package.json` file to manage dependencies:

```json
{
  "name": "nodejs-hello-world",
  "version": "1.0.0",
  "description": "A simple Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

### 6. Jenkins Pipeline Setup

Create a `Jenkinsfile` for your Jenkins pipeline that automates the process of building, pushing, and deploying the Docker image.

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'tamilselvanbt/nodejs-hello-world'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE:latest .'
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@<EC2_PUBLIC_IP> "docker pull $DOCKER_IMAGE:latest && docker run -d -p 80:3000 $DOCKER_IMAGE:latest"'
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -af'
        }
    }
}
```

### 7. Build and Push the Docker Image

- In Jenkins, the pipeline will build a Docker image using the `Dockerfile`.
- The image is then pushed to Docker Hub.

![image](https://github.com/user-attachments/assets/adf2dd13-b420-4c81-9e49-30abd775110d)


### 8. Deploy the Docker Image

- The Jenkins pipeline will SSH into your EC2 instance, pull the Docker image from Docker Hub, and run it.

  ![image](https://github.com/user-attachments/assets/ad382745-8c8e-42b1-bf7d-d82241b58e84)


### 9. Verify the Deployment

- Access your application using the EC2 instance’s public IP in a web browser (`http://<EC2_PUBLIC_IP>`). You should see the "Hello World!" message.

  ![image](https://github.com/user-attachments/assets/c0345117-8d13-4c2d-be57-b40de8e01f00)


## Troubleshooting

- Ensure that your EC2 instance’s security group allows inbound traffic on port 80.
- Verify that Jenkins has the correct permissions to interact with Docker and access the repository.
- Ensure Docker is running on your EC2 instance and there are no firewall rules blocking access.

## Conclusion

By following these steps, you have successfully set up a CI/CD pipeline for a Node.js application, using Jenkins for continuous integration, Docker for containerization, and AWS EC2 for deployment.
