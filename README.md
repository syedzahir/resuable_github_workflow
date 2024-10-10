# GitHub Workflow

This repository contains a reusable GitHub Actions workflow designed to build, tag, and push Docker images, and update Helm charts based on the specified environment. The workflow is highly configurable and can be used across multiple repositories with minimal setup.

## Workflow Overview

### Workflow File: `.github/workflows/main.yaml`

This workflow performs the following steps:

1. **Checkout Main Repository**: Clones the primary repository.
2. **Set up Docker Buildx**: Prepares the environment for Docker builds using Buildx.
3. **Log in to Docker Hub**: Authenticates with Docker Hub using credentials.
4. **Determine Image Name**: Sets the Docker image name based on the branch (`main` or others).
5. **Build Docker Image**: Builds the Docker image and tags it with a version.
6. **Push Docker Image**: Pushes the built Docker image to Docker Hub.
7. **Clone Helm Repository**: Clones the Helm chart repository to update the chart with the new Docker image version.
8. **Determine Environment and Directory**: Sets the environment (`production` or `staging`) and the corresponding Helm directory based on the branch.
9. **Update Helm Chart**: Updates the `values.yaml` file with the new Docker image tag.
10. **Commit and Push Changes**: Commits and pushes the updated Helm chart back to the repository.

## Inputs

- **`docker_repo`**: The Docker repository name. *(Required)*
- **`helm_repo`**: The Helm chart repository URL. *(Required)*
- **`deploy_on_cluster`**: To deploy on the cluster. By default true. *(optional)*

## Secrets

The workflow requires the following secrets, which should be defined in the target project repository:

- **`DOCKER_USERNAME`**: The Docker Hub username. *(Required)*
- **`DOCKER_PASSWORD`**: The Docker Hub password. *(Required)*
- **`PERSONAL_ACCESS_TOKEN`**: A GitHub personal access token with permission to push changes to the Helm chart repository. *(Required)*

### Secret Management

To centralize secret management, these secrets should be configured in the repository. Follow these steps to set up the secrets:

1. **Navigate to the Central Repository**: Go to the project repository.
2. **Go to Settings**: Click on the "Settings" tab.
3. **Access Secrets**: In the sidebar, find "Secrets and variables" and select "Actions".
4. **Add Secrets**:
   - **DOCKER_PASSWORD**: Add a new secret with the name `DOCKER_PASSWORD` and the value as your Docker Hub password.
   - **PERSONAL_ACCESS_TOKEN**: Add a new secret named `PERSONAL_ACCESS_TOKEN` and the value as a GitHub personal access token. This token should have `repo` scope to push changes to the Helm chart repository.

### Usage

To utilize this reusable workflow in other repositories, you need to create a workflow file in each target repository that calls the reusable workflow and passes the necessary inputs.

#### Example Usage

Here’s how you can set up the workflow in a target repository:

1. **Create a Workflow File**: In the target repository, create a new workflow file at `.github/workflows/docker-build-and-push.yml`.

2. **Define the Workflow**: Use the following YAML configuration to call the reusable workflow:

   ```yaml
   # .github/workflows/main.yaml
   name: Build and Push Docker Image
   on:
     push:
       branches:
         - main
         - staging  
   jobs:
     build-and-push:
       uses: <your-company-github project name>/resuable-workflow/.github/workflows/main.yml@main
       with:
         docker_repo: <write your-project-repository name>
         helm_repo: <write your-project-helm-repository name>
         deploy_on_cluster: <default true | false>
       secrets:  inherit # Passing secrets DOCKER_USERNAME, DOCKER_PASSWORD, PERSONAL_ACCESS_TOKEN to resuable workflow
   ```
