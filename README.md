Jenkins Automation for Docker Image Deployment

This repository contains an automation script for Jenkins to facilitate the continuous integration and deployment process. It is designed to trigger a series of steps upon receiving a push event to a GitHub repository. The automation process involves cloning the code from GitHub, building a Docker image from a Dockerfile, uploading the image to Docker Hub, and finally deploying the image for the developer to test their code changes.
