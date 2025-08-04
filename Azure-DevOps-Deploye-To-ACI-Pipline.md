This pipeline is for a **CI/CD workflow** that builds a Docker image for a 2048 web game, pushes it to Azure Container Registry (ACR), and deploys it to Azure Container Instance (ACI).

````md
# 🚀 Azure DevOps CI/CD Pipeline: Build & Deploy to Azure Container Instance (ACI)

This document describes a complete **multi-stage Azure DevOps pipeline** that performs the following:

- Builds a Docker image from source code (2048 web game)
- Pushes the image to Azure Container Registry (ACR)
- Deploys the container to Azure Container Instance (ACI)
- Retrieves the public IP address after deployment

---

## 🔄 Trigger

```yaml
trigger:
  - main
````

> The pipeline runs automatically on every push to the `main` branch.

---

## ⚙️ Pipeline Configuration

```yaml
pool:
  vmImage: ubuntu-latest

variables:
  acrName: myadoaks
  acrLoginServer: myadoaks.azurecr.io
  imageName: web-game
  imageTag: build-$(Build.BuildId)
  resourceGroup: ADO-AKS-ACR-RG
  containerName: my-game
  location: centralindia
```

---

## 🛠️ Build Stage: Build & Push Docker Image to ACR

```yaml
stages:
- stage: Build
  displayName: 'Build and Push to ACR'
  jobs:
    - job: BuildImage
      displayName: 'Build Docker Image'
      steps:
        - task: AzureCLI@2
          displayName: 'Build and Push Image'
          inputs:
            azureSubscription: 'Azure subscription 1(c2edf3cd-5e67-41bc-9b94-2677390d2194)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              echo "🔐 Login to ACR..."
              az acr login -n $(acrName)

              echo "🐳 Build Docker image: $(acrLoginServer)/$(imageName):$(imageTag)"
              docker build -t $(acrLoginServer)/$(imageName):$(imageTag) 2048-game

              echo "🚀 Push image to ACR"
              docker push $(acrLoginServer)/$(imageName):$(imageTag)
```

> This stage builds a Docker image for the `2048-game` directory and pushes it to the Azure Container Registry with a unique tag based on the build ID.

---

## 🚀 Deploy Stage: Deploy Image to Azure Container Instance (ACI)

```yaml
- stage: Deploy
  displayName: 'Deploy to Azure Container Instance'
  dependsOn: Build
  jobs:
    - job: DeployACI
      displayName: 'Deploy to ACI'
      steps:
        - task: AzureCLI@2
          displayName: 'Update ACI with New Image'
          inputs:
            azureSubscription: 'Azure subscription 1(c2edf3cd-5e67-41bc-9b94-2677390d2194)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              acrName=$(acrName)
              acrLoginServer=$(acrLoginServer)
              imageName=$(imageName)
              imageTag=$(imageTag)
              resourceGroup=$(resourceGroup)
              containerName=$(containerName)
              location=$(location)

              echo "🔐 Get ACR credentials..."
              ACR_USERNAME=$(az acr credential show --name $acrName --query username -o tsv)
              ACR_PASSWORD=$(az acr credential show --name $acrName --query passwords[0].value -o tsv)

              echo "🗑️ Delete existing ACI (if any)..."
              az container delete --name $containerName --resource-group $resourceGroup --yes || true

              echo "🚀 Deploy ACI with image: $acrLoginServer/$imageName:$imageTag"
              az container create \
                --resource-group $resourceGroup \
                --name $containerName \
                --image $acrLoginServer/$imageName:$imageTag \
                --registry-login-server $acrLoginServer \
                --registry-username $ACR_USERNAME \
                --registry-password $ACR_PASSWORD \
                --ports 80 \
                --cpu 1 \
                --memory 1.5 \
                --os-type Linux \
                --ip-address Public \
                --location $location

              echo "✅ Deployment complete!"

              echo "🌐 Get Public IP of ACI:"
              az container show \
                --name $containerName \
                --resource-group $resourceGroup \
                --query ipAddress.ip \
                -o tsv
```

> This stage fetches ACR credentials, deletes the existing container (if any), then creates a new ACI instance using the latest pushed image, and finally outputs the public IP address of the deployed container.

---

## ✅ Final Output

* The ACI instance will host the Dockerized 2048 game and expose it on **port 80**.
* You can access it using the public IP address printed at the end of the deployment stage.
