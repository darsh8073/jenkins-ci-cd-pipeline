# Jenkins CI/CD Pipeline with Docker

A complete CI/CD pipeline for a Python Flask application using Jenkins, GitHub, Docker, and Docker Hub.

The pipeline automatically tests, builds, publishes, deploys, and health-checks the application whenever code is pushed to the GitHub main branch.

## Architecture

```text
Developer
    |
    | git push
    v
GitHub
    |
    | Webhook
    v
Jenkins
    |
    +--> Checkout Source
    |
    +--> Install Dependencies
    |
    +--> Run Pytest
    |
    +--> Build Docker Image
    |
    +--> Push Image to Docker Hub
    |
    +--> Pull Versioned Image
    |
    +--> Deploy Docker Container
    |
    +--> Health Check
    |
    v
Running Flask Application
