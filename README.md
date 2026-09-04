# Azure DevOps CI/CD Pipeline

A practical DevOps portfolio project demonstrating an automated CI/CD pipeline using GitHub Actions, Docker and Microsoft Azure.

The pipeline automatically tests a Python application, builds a Docker image, publishes the image to Azure Container Registry and deploys the new version to Azure Container Apps.

## Architecture

```text
Developer
   |
   | git push
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   +--> Run automated Python tests
   |
   +--> Build Docker image
   |
   +--> Authenticate to Azure using OIDC
   |
   +--> Push image to Azure Container Registry
   |
   v
Azure Container Apps
   |
   v
Public HTTPS Application
```

## Technologies Used

* Python
* Flask
* Pytest
* Gunicorn
* Docker
* Git
* GitHub
* GitHub Actions
* Microsoft Azure
* Azure CLI
* Azure Container Registry (ACR)
* Azure Container Apps
* Microsoft Entra ID
* OpenID Connect (OIDC)

## CI/CD Workflow

The GitHub Actions workflow is triggered automatically when code is pushed to the `main` branch.

The pipeline performs the following stages:

1. Checks out the source code.
2. Creates a Python environment.
3. Installs the application dependencies.
4. Runs automated tests using Pytest.
5. Authenticates to Azure using OpenID Connect.
6. Authenticates to Azure Container Registry.
7. Builds the Docker image.
8. Tags the image using the Git commit SHA.
9. Pushes the Docker image to Azure Container Registry.
10. Updates the Azure Container App with the new image.

A deployment therefore follows this process:

```text
Code Change
    |
    v
Git Push
    |
    v
Automated Tests
    |
    v
Docker Build
    |
    v
Container Registry
    |
    v
Azure Deployment
```

A successful push to `main` can therefore result in a new application version being deployed without a manual Azure deployment step.

## Continuous Integration

The CI stage validates the application before deployment.

Automated tests verify both the main application endpoint and the health endpoint.

Tests are executed with:

```bash
pytest -v
```

If the tests fail, the GitHub Actions workflow fails and the remaining deployment process does not proceed successfully.

This provides a basic quality gate before deployment.

## Docker

The Python application is packaged as a Docker image.

The image contains the application and the dependencies required to run it consistently across environments.

The application uses Gunicorn as its application server and listens on port `5000`.

A local image can be built using:

```bash
docker build -t azure-devops-cicd .
```

## Azure Container Registry

Successful application builds are published to Azure Container Registry.

Images are tagged using the GitHub commit SHA, providing traceability between a deployed container image and the source-code revision that produced it.

A `latest` tag is also published by the workflow.

Conceptually:

```text
Git commit
    |
    v
Docker image
    |
    v
Commit SHA tag
    |
    v
Azure Container Registry
```

## Azure Container Apps

The application is hosted using Azure Container Apps.

Azure Container Apps retrieves the container image from Azure Container Registry and creates a new application revision when the deployment pipeline updates the image.

External ingress exposes the application over HTTPS.

The application provides two endpoints:

```text
/
```

Main application endpoint.

```text
/health
```

Health-check endpoint returning:

```json
{
  "status": "healthy"
}
```

## Secure Azure Authentication

The GitHub Actions pipeline authenticates to Microsoft Azure using OpenID Connect (OIDC) and workload identity federation.

This avoids storing a long-lived Azure client secret in the GitHub repository.

The authentication flow is:

```text
GitHub Actions
      |
      | OIDC token
      v
Microsoft Entra ID
      |
      | Federated identity validation
      v
Azure
```

Repository secrets provide the identifiers required by the Azure login action:

```text
AZURE_CLIENT_ID
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
```

No Azure passwords, access tokens or client secrets are stored in the source code.

## Managed Identity and RBAC

Azure identities and role-based access control are used to limit access between components.

The deployment identity is granted the permissions required to manage the project's Azure resources and push images to Azure Container Registry.

The Azure Container App uses a managed identity with permission to pull images from the registry.

This demonstrates the principle of using identities and scoped permissions instead of embedding credentials in application code.

## Project Structure

```text
azure-devops-cicd/
|
|-- .github/
|   `-- workflows/
|       `-- ci-cd.yml
|
|-- tests/
|   `-- test_app.py
|
|-- app.py
|-- Dockerfile
|-- requirements.txt
|-- README.md
`-- .gitignore
```

## Example Development Workflow

A developer makes a change locally and pushes it to the `main` branch:

```bash
git add .
git commit -m "Update application"
git push origin main
```

GitHub Actions then automatically performs the configured CI/CD workflow.

The developer can inspect the workflow execution and individual stages through the GitHub Actions interface.

## What This Project Demonstrates

This project demonstrates practical experience with:

* Source control using Git and GitHub
* Automated CI/CD workflows
* GitHub Actions
* Automated application testing
* Python and Flask
* Docker image creation
* Container registries
* Containerised application deployment
* Azure Container Apps
* Azure CLI
* Azure RBAC
* Managed identities
* Microsoft Entra ID
* OIDC workload identity federation
* Secure cloud authentication
* Application health endpoints
* Git SHA-based image versioning
* Cloud resource lifecycle management

## Key Learning

The project demonstrates the difference between manually deploying an application and implementing an automated deployment pipeline.

Instead of manually building and deploying each application version, a source-code push triggers a repeatable process that tests, packages and deploys the application.

This provides a foundation for more advanced DevOps practices including deployment environments, approval gates, infrastructure as code, monitoring, rollback strategies and automated security scanning.

## Resource Management

Azure resources used for this portfolio project can be removed after deployment testing to avoid leaving unnecessary cloud resources running.

The source code, Git history and GitHub Actions workflow remain available in GitHub as evidence of the implementation.
