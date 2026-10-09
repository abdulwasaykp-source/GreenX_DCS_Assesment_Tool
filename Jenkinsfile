```yaml
name: GreenX CI/CD

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: [self-hosted, greenx-vm-156]

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Verify Docker
        run: |
          docker --version
          docker compose version
          docker ps

      - name: Login to Docker Hub
        env:
          DOCKER_USERNAME: ${{ secrets.ABDULWASAYKP }}
          DOCKER_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
        run: |
          printf '%s' "$DOCKER_TOKEN" | docker login \
            --username "$DOCKER_USERNAME" \
            --password-stdin

      - name: Build Backend Image
        run: |
          docker build \
            -t abdulwasaykp/greenx-backend:${GITHUB_RUN_NUMBER} \
            ./GreenX_DCS_Assesment_Tool_Backend

      - name: Build Frontend Image
        run: |
          docker build \
            -t abdulwasaykp/greenx-frontend:${GITHUB_RUN_NUMBER} \
            ./greenX-assessment-tool-frontend

      - name: Push Backend Image
        run: |
          docker push \
            abdulwasaykp/greenx-backend:${GITHUB_RUN_NUMBER}

      - name: Push Frontend Image
        run: |
          docker push \
            abdulwasaykp/greenx-frontend:${GITHUB_RUN_NUMBER}

      - name: Deploy GreenX Application
        run: |
          set -e

          cd /home/osboxes/greenx-deployment

          export DOCKER_TAG="${GITHUB_RUN_NUMBER}"

          docker compose \
            --env-file .env \
            -f compose.deploy.yml \
            pull backend frontend

          docker compose \
            --env-file .env \
            -f compose.deploy.yml \
            up -d

      - name: Verify Deployment
        run: |
          docker ps
          docker compose \
            --env-file /home/osboxes/greenx-deployment/.env \
            -f /home/osboxes/greenx-deployment/compose.deploy.yml \
            ps
```
