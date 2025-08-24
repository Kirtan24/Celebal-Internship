# 🚀 Azure DevOps CI/CD Pipeline: Deploy Web App to Azure Virtual Machine (VM)

This multi-stage pipeline automates deployment of the **2048 Web Game** to an **Azure Linux VM**.  

It performs:  
- 📦 Archiving game files into a `.zip`  
- 📤 Publishing the archive as a pipeline artifact  
- 🔐 Transferring the artifact to the VM via SSH  
- 🛠️ Unzipping and restarting **Nginx** on the VM  

---

## 🏗️ Architecture Overview

> The pipeline automates packaging and deployment of a static web app to an Azure VM running **Nginx**.  

<img width="1572" height="952" alt="image" src="https://github.com/user-attachments/assets/cf0e2456-d245-44a1-bb38-2ee345676059" />

**Flow:**  
**Source Code → Azure DevOps Pipeline → Build Artifact → Transfer to VM via SSH → Extract & Serve via Nginx**

---

## 🔄 Pipeline Trigger

```yaml
trigger:
  - main
```

✅ The pipeline automatically runs when changes are pushed to the `main` branch.

---

## ⚙️ Pipeline Configuration

```yaml
pool:
  vmImage: 'ubuntu-latest'

variables:
  artifactName: 'game-dist'
  deployPath: '/var/www/2048-game'
  zipFileName: '2048.zip'
```

These variables define the **artifact name, target deploy path on VM, and archive filename**.

---

## 🛠️ Stage 1: Build & Archive Web App Files

```yaml
stages:
- stage: Build
  jobs:
    - job: BuildAndArchive
      steps:
        - task: ArchiveFiles@2
          inputs:
            rootFolderOrFile: '2048-game'
            includeRootFolder: false
            archiveType: 'zip'
            archiveFile: '$(Build.ArtifactStagingDirectory)/$(zipFileName)'
            replaceExistingArchive: true
          displayName: '📦 Archive Game Files'

        - task: PublishBuildArtifacts@1
          inputs:
            PathtoPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: '$(artifactName)'
            publishLocation: 'Container'
          displayName: '📤 Publish Artifact'
```

👉 **What happens here?**  
1. Compresses the `2048-game` directory into `2048.zip`.  
2. Publishes it as a **pipeline artifact** for the next stage.  

✅ Result: A ready-to-deploy `.zip` is stored in the pipeline.  

---

## 🚀 Stage 2: Deploy to Azure VM via SSH

```yaml
- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
    - job: DeployToVM
      steps:
        # Download Artifact
        - task: DownloadPipelineArtifact@2
          inputs:
            artifactName: '$(artifactName)'
            targetPath: '$(Pipeline.Workspace)/$(artifactName)'
          displayName: '⬇️ Download Artifact'

        # Clean Deployment Directory
        - task: SSH@0
          inputs:
            sshEndpoint: 'vm-ssh-service-connection'
            runOptions: 'inline'
            inline: |
              echo "🧹 Cleaning up deploy folder..."
              rm -rf $(deployPath)/*
          displayName: '🧹 Clean Target Folder'

        # Copy ZIP to VM
        - task: CopyFilesOverSSH@0
          inputs:
            sshEndpoint: 'vm-ssh-service-connection'
            sourceFolder: '$(Pipeline.Workspace)/$(artifactName)'
            contents: '$(zipFileName)'
            targetFolder: '$(deployPath)'
          displayName: '📁 Copy ZIP to VM'

        # Unzip and Restart Web Server
        - task: SSH@0
          inputs:
            sshEndpoint: 'vm-ssh-service-connection'
            runOptions: 'inline'
            inline: |
              echo "📂 Checking contents of $(deployPath)..."
              ls -l $(deployPath)

              cd $(deployPath)

              echo "📦 Unzipping..."
              unzip $(zipFileName)

              echo "🗑️ Removing ZIP..."
              rm $(zipFileName)

              echo "🔁 Restarting nginx..."
              sudo systemctl restart nginx
          displayName: '🛠️ Unzip & Restart Nginx'
```

👉 **What happens here?**  
1. **Download Artifact** – Retrieves the `2048.zip` from the pipeline.  
2. **Clean Target Directory** – Clears old files from `/var/www/2048-game`.  
3. **Copy ZIP to VM** – Transfers the archive via SSH/SCP.  
4. **Unzip & Restart Nginx** – Extracts files, removes the zip, restarts web server.  

✅ Result: The **2048 game** is deployed and served by **Nginx** on the VM.  

---

## 🔐 SSH Configuration Note

Ensure that your service connection `vm-ssh-service-connection` is:  
- Configured with the **Azure Linux VM public IP**  
- Authenticated with a **valid private key or password**  
- Has required permissions for `scp`, `unzip`, and `nginx`  

---

## 📁 Folder Structure (Expected)

```bash
.
├── azure-pipelines.yml            # This pipeline file
└── 2048-game/                     # Game source directory
    ├── index.html
    ├── style.css
    ├── ...
```

---

## ✅ Final Output

After successful execution:  
- 📦 `2048.zip` is generated and published.  
- 📤 The archive is transferred to the VM.  
- 🧹 Old files are cleared.  
- 🛠️ Game files are unzipped & deployed.  
- 🌐 App is served via **Nginx** on the VM’s **public IP / domain**.  

---

## 🔍 How the Pipeline Works (Stage-by-Stage Summary)

1. **Code Commit (Trigger)**  
   - Any push to `main` branch starts the pipeline.  

2. **Build Stage**  
   - Archives the game files into `2048.zip`.  
   - Publishes it as an artifact.  

3. **Deploy Stage**  
   - Downloads the artifact.  
   - Cleans the target directory on the VM.  
   - Transfers the `.zip` via SSH.  
   - Extracts contents & restarts Nginx.  

4. **User Access**  
   - Open the VM’s public IP in a browser → 🎮 Play the 2048 game!  

---
