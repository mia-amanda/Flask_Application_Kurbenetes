# Flask Application on Kubernetes

This repository contains a simple Flask application that is containerized using Docker and deployed on a Kubernetes cluster. It serves as a basic example of how to set up a Flask app with Kubernetes.

## Contents

- `app.py`: The main Flask application.
- `Dockerfile`: Instructions for building the Docker image of the Flask app.
- `deployment.yaml`: Kubernetes deployment configuration for the Flask application.
- `service.yaml`: Kubernetes service configuration to expose the Flask application.

## Prerequisites

- Docker installed on your machine
- Kubernetes cluster (Minikube or any cloud-based cluster)
- `kubectl` installed for interacting with the Kubernetes cluster

### Update Docker Image in Deployment File

1. Open `deployment.yaml` in a text editor.

2. Locate the following line:

   ```yaml
   image: your-dockerhub-username/flask-app:latest

3. Replace your-dockerhub-username with your actual Docker Hub username

