# 🚀 Azure DevOps CI/CD Pipeline: Build & Deploy to Azure Container Instance (ACI)

This document describes a complete **multi-stage Azure DevOps pipeline** that:

- 🐳 Builds a Docker image for the **2048 web game**
- 📦 Pushes the image to **Azure Container Registry (ACR)**
- ☁️ Deploys the image to **Azure Container Instance (ACI)**
- 🌐 Retrieves the **public IP address** after deployment

---

## 🏗️ Architecture Overview  

> The pipeline automates building, storing, and deploying the 2048 web game container image.  

<img width="1801" height="995" alt="image" src="https://github.com/user-attachments/assets/f5622502-bd39-452c-aa7e-5b70e93838c3" />

The flow looks like this:  

**Source Code → Azure DevOps Pipeline → Build & Push to ACR → Deploy on ACI → Public Access**

---

## 🔄 Pipeline Trigger

```yaml
trigger:
  - main
```

✅ The pipeline runs automatically on every push to the `main` branch.

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

These variables define the **ACR name, image details, resource group, container instance, and deployment region**.

---

## 🛠️ Stage 1: Build & Push Docker Image to ACR

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

👉 **What happens here?**
1. **Login to ACR** – The pipeline authenticates with Azure Container Registry.  
2. **Build Docker Image** – A new Docker image for the 2048 web game is created.  
3. **Push Image to ACR** – The image is tagged with a unique build ID and stored in ACR.  

Result → The image is now available in ACR for deployment.  

---

## 🚀 Stage 2: Deploy to Azure Container Instance (ACI)

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

👉 **What happens here?**
1. **Fetch ACR Credentials** – Required for ACI to pull the image.  
2. **Delete Old Container (if any)** – Ensures a clean environment.  
3. **Deploy New Container** – Creates a new Azure Container Instance with the latest Docker image.  
4. **Expose Public IP** – The game becomes available on the internet (Port 80).  

Result → The latest version of the game is deployed and accessible.  

---

## ✅ Final Output

- The **ACI instance** hosts the **Dockerized 2048 game**  
- Available on **port 80**  
- Access using the **public IP address** printed at the end of the pipeline 🎉  

---

## 🔍 How the Pipeline Works (Stage-by-Stage Summary)

1. **Code Commit (Trigger)**  
   - Any push to the `main` branch triggers the pipeline.  

2. **Build Stage**  
   - Logs in to ACR  
   - Builds Docker image for the 2048 game  
   - Pushes image to ACR with a unique tag  

3. **Deploy Stage**  
   - Fetches ACR credentials  
   - Deletes old container instance (if running)  
   - Deploys the **new image** on Azure Container Instance (ACI)  
   - Retrieves the **public IP address**  

4. **User Access**  
   - Developers/users can open the public IP in a browser and play the **2048 game** 🎮  

---

✨ In short:  
**Every commit → Triggers pipeline → Builds image → Pushes to ACR → Deploys to ACI → Instantly available via public IP.**
