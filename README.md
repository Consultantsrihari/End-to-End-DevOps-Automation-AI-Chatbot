# End-to-End DevOps Automation for a Scalable and Secure AI Chatbot

This repository contains the complete source code and configuration for deploying a scalable and secure AI-powered chatbot using modern DevOps practices. The project is based on the "End-to-End DevOps Automation" guide by DevOps Shack.

## Features
- **CI/CD Automation**: Automated build, test, and deployment pipeline using GitHub Actions.
- **Containerization**: Application containerized with Docker for consistency.
- **Orchestration**: Deployed on Kubernetes for scalability and high availability.
- **Infrastructure as Code (IaC)**: Kubernetes manifests define the desired state of the application.
- **Security (DevSecOps)**:
  - Code scanning with CodeQL.
  - Container image scanning with Trivy.
  - Secure pod configurations (non-root user, read-only filesystem).
  - Secrets management via Kubernetes Secrets.
- **Monitoring & Observability**:
  - Metrics endpoint for Prometheus scraping.
  - Centralized logging support.
- **Performance & Scalability**:
  - Horizontal Pod Autoscaler (HPA) for automatic scaling.
  - Redis caching to reduce latency and database load.
  - Performance testing with k6.

## Tech Stack
- **Backend**: Python with FastAPI
- **AI**: OpenAI GPT API
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **CI/CD**: GitHub Actions
- **Caching**: Redis
- **Security Scanning**: CodeQL, Trivy
- **Monitoring**: Prometheus
- **Performance Testing**: k6

## Setup and Usage

### Prerequisites
1.  **Docker Desktop**: To build and run containers locally.
2.  **kubectl**: To interact with a Kubernetes cluster.
3.  **A Kubernetes Cluster**: Minikube, Kind, or a cloud provider (EKS, GKE, AKS).
4.  **OpenAI API Key**: Get one from the [OpenAI Platform](https://platform.openai.com/).
5.  **k6**: For running performance tests.

### Local Development
1.  **Clone the repository:**
    ```bash
    git clone <your-repo-url>
    cd <repo-name>
    ```

2.  **Set up environment variables:**
    Create a `.env` file in the root directory and add your OpenAI API key:
    ```
    OPENAI_API_KEY="your-openai-api-key"
    ```

3.  **Create a virtual environment and install dependencies:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    pip install -r requirements.txt
    ```

4.  **Run the application locally:**
    The application will be available at `http://127.0.0.1:8000`.
    ```bash
    uvicorn app.main:app --reload
    ```

### Docker
1.  **Build the Docker image:**
    ```bash
    docker build -t chatbot-api:latest .
    ```

2.  **Run the Docker container:**
    ```bash
    docker run -d -p 8000:8000 --env-file .env --name chatbot-container chatbot-api:latest
    ```

### Kubernetes Deployment
1.  **Create a Kubernetes Secret** for your API key. Remember to base64 encode it.
    ```bash
    echo -n "your-openai-api-key" | base64
    ```
    Create a `secret.yaml` file (DO NOT commit this file if it contains real secrets):
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: chatbot-secrets
    type: Opaque
    data:
      OPENAI_API_KEY: "your-base64-encoded-key"
    ```
    Apply the secret: `kubectl apply -f secret.yaml`

2.  **Deploy the application:**
    The `kustomization.yaml` file groups all our Kubernetes manifests.
    ```bash
    kubectl apply -k ./kubernetes
    ```

3.  **Access the service:**
    If using Minikube, you can expose the service:
    ```bash
    minikube service chatbot-service
    ```
    If using a cloud provider, the `LoadBalancer` will provision a public IP address.

## CI/CD Pipeline
The pipeline is defined in `.github/workflows/ci-cd.yml` and automates the following steps on every push to `main`:
1.  **Test & Lint**: Runs `pytest`.
2.  **Security Scan**: Scans the code with GitHub CodeQL.
3.  **Build & Push**: Builds the Docker image and pushes it to Docker Hub.
4.  **Scan Image**: Scans the pushed Docker image for vulnerabilities with Trivy.
5.  **Deploy**: Deploys the new image to the Kubernetes cluster.

### Required GitHub Secrets
To make the CI/CD pipeline work, you must configure the following secrets in your GitHub repository settings:
- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub password or access token.
- `KUBE_CONFIG`: Base64 encoded content of your `kubeconfig` file to allow GitHub Actions to access your cluster.
- `SLACK_WEBHOOK`: The webhook URL for sending Slack notifications.

## Performance Testing
To run the load test script:
```bash
# Make sure your service is accessible at the URL in the script
k6 run k6/load-test.js
