name: GreenX CI/CD

on:
  push:
    branches:
      - main

  workflow_dispatch:

permissions:
  contents: read

jobs:

  build-and-push:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.ABDULWASAYKP }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Backend Image
        run: |
          docker build \
            -t abdulwasaykp/greenx-backend:${{ github.run_number }} \
            ./GreenX_DCS_Assesment_Tool_Backend

      - name: Build Frontend Image
        run: |
          docker build \
            -t abdulwasaykp/greenx-frontend:${{ github.run_number }} \
          ./greenX-assessment-tool-frontend

      - name: Push Backend Image
        run: |
          docker push \
            abdulwasaykp/greenx-backend:${{ github.run_number }}

      - name: Push Frontend Image
        run: |
          docker push \
            abdulwasaykp/greenx-frontend:${{ github.run_number }}
