````md
# 🚀 Azure DevOps CI/CD Pipeline: Deploy Web App to Azure Virtual Machine (VM)

This multi-stage pipeline performs end-to-end automation for deploying a web application (2048 game) to an **Azure Linux VM**. The pipeline handles:

- Archiving game files into a `.zip`
- Publishing the artifact
- Transferring the artifact to the VM via SSH
- Unzipping and restarting the Nginx server on the VM

---

## 🔄 Trigger

```yaml
trigger:
  - main
````

> The pipeline automatically runs when changes are pushed to the `main` branch.

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

---

## 📦 Stage 1: Build & Archive Web App Files

This stage creates a ZIP archive of the `2048-game` directory and publishes it as a pipeline artifact.

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

> This compresses the game's source folder into a `2048.zip` and makes it available for downstream jobs.

---

## 🚀 Stage 2: Deploy to Azure VM via SSH

This stage handles VM deployment tasks including file transfer, cleanup, unzipping, and Nginx restart.

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

---

## 🔐 SSH Configuration Note

Ensure that your service connection `vm-ssh-service-connection` is:

* Properly configured with the **Azure Linux VM IP**
* Using a valid **private key or password**
* Allows `scp`, `unzip`, and `nginx` to run via SSH

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

After successful pipeline execution:

* The `2048.zip` file is transferred to the VM
* The target directory (`/var/www/2048-game`) is cleaned
* The game files are unzipped and served via **Nginx**

You can now access your deployed app via the **VM’s public IP or domain**.
