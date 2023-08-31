---
title: Deploy Application with ACA with Dockerfile from GitHub to achieve CICD
categories: Azure
description: Deploy Application with ACA with Dockerfile from GitHub to achieve CICD
tags: Azure-container-apps
abbrlink: 39432
date: 2023-08-25 16:16:00
password:
message:

---

---

**There is a demo how to deploy Application with ACA with Dockerfile from GitHub to achieve CICD:**

To deploy a Docker container using a Dockerfile from GitHub to an Azure Container App,  you can follow these steps. Make sure you have the Azure CLI installed and you're logged in to your Azure account.

Deploying an application using Azure Container Apps (ACA) with a Dockerfile from GitHub and implementing Continuous Integration and Continuous Deployment (CI/CD) involves several steps. 

1. **Create an Azure Resource Group (if not already created)**

```bash
az group create --name MyResourceGroup --location eastus
```

2. **Create an Azure Container Registry (ACR)**

```bash
az acr create --resource-group MyResourceGroup --name myacr --sku Basic --admin-enabled true
```

3. **Create a GitHub Repository**

Create a GitHub repository where you'll store your application code and Dockerfile.

4. **Set Up GitHub Actions for CI/CD**

In your GitHub repository, set up GitHub Actions to automate the CI/CD process. Create a `.github/workflows/main.yml` file with the following content:

```yaml
name: CI/CD to Azure Container Apps

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to Azure Container Registry
      uses: azure/docker-login@v1
      with:
        login-server: myacr.azurecr.io
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}

    - name: Build and push Docker image
      run: |
        docker build -t myacr.azurecr.io/myapp:$GITHUB_SHA .
        docker push myacr.azurecr.io/myapp:$GITHUB_SHA

    - name: Deploy to Azure Container Apps
      uses: Azure/container-apps-deploy@v1
      with:
        name: mycontainerapp
        resource-group: MyResourceGroup
        image: myacr.azurecr.io/myapp:$GITHUB_SHA
```

In this example, `ACR_USERNAME` and `ACR_PASSWORD` are GitHub repository secrets that you'll need to create in your repository settings to securely store your ACR credentials.

5. **Configure Your Dockerfile**

Create a `Dockerfile` in your GitHub repository that defines how to build your application image. Make sure your application code and required dependencies are included.

```Dockerfile
# Dockerfile
# Use latest jboss/base-jdk:11 image as the base
FROM jboss/base-jdk:11

# Set the WILDFLY_VERSION env variable
ENV WILDFLY_VERSION 24.0.0.Final
ENV WILDFLY_SHA1 391346c9ed2772647ff07aeae39deb838ee11dcf
ENV JBOSS_HOME /opt/jboss/wildfly

USER root

# Add the WildFly distribution to /opt, and make wildfly the owner of the extracted tar content
# Make sure the distribution is available from a well-known place
RUN cd $HOME \
    && curl -O https://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz \
    && sha1sum wildfly-$WILDFLY_VERSION.tar.gz | grep $WILDFLY_SHA1 \
    && tar xf wildfly-$WILDFLY_VERSION.tar.gz \
    && mv $HOME/wildfly-$WILDFLY_VERSION $JBOSS_HOME \
    && rm wildfly-$WILDFLY_VERSION.tar.gz \
    && chown -R jboss:0 ${JBOSS_HOME} \
    && chmod -R g+rw ${JBOSS_HOME}

# Ensure signals are forwarded to the JVM process correctly for graceful shutdown
ENV LAUNCH_JBOSS_IN_BACKGROUND true

USER jboss

# Expose the ports in which we're interested
EXPOSE 8080

# Set the default command to run on boot
# This will boot WildFly in standalone mode and bind to all interfaces
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0"]
```

6. **Create an Azure Container App**

```bash
az containerapp create --name mycontainerapp \
--resource-group MyResourceGroup \
--location eastus \
--image myacr.azurecr.io/myapp:$GITHUB_SHA \
--registry-login-server myacr.azurecr.io \
--registry-username <your-username> \
--registry-password <your-password> \
--dns-name-label mycontainerapp
```

Replace `<your-username>` and `<your-password>` with your ACR username and password.

With these steps, GitHub Actions workflow will automatically trigger whenever you push to the `main` branch. It will build your Docker image, push it to your Azure Container Registry, and deploy the updated image to your Azure Container App.

Please note that the code examples provided are meant for guidance and may need adjustments based on your specific project structure, configurations, and security considerations. Always ensure that you follow best practices for security and CI/CD processes.
