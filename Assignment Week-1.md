# Assignment - Week 1: Linux Basics

## 1. File and Directory Permissions

### Create a File and Assign Permissions

```bash
touch myfile.txt
```

![image](https://github.com/user-attachments/assets/66535a45-59e9-4400-80d3-b587180de5ac)

### Set Permissions Using `chmod`

```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission from group
chmod g-w file.txt

# Set read-only for others
chmod o=r file.txt

# Give execute permission to everyone
chmod a+x run.sh

# Set: Owner (read/write), Group (read), Others (read)
chmod u=rw,go=r notes.txt
```

### Numeric Representation of Permissions

| Permission  | Binary | Value |
| ----------- | ------ | ----- |
| Read (r)    | 100    | 4     |
| Write (w)   | 010    | 2     |
| Execute (x) | 001    | 1     |

Example:
`chmod 755 file.sh` means:

* Owner: `rwx` (7 = 4+2+1)
* Group: `r-x` (5 = 4+0+1)
* Others: `r-x` (5 = 4+0+1)

---

![image](https://github.com/user-attachments/assets/c503a14a-94bb-453b-8d8b-0013c647fac2)
![image](https://github.com/user-attachments/assets/4ede868d-beea-4f7e-946d-94edefd66ab8)
![image](https://github.com/user-attachments/assets/e205996a-afa2-4293-b777-fb51dca511d0)
![image](https://github.com/user-attachments/assets/f9b93304-d3b8-4927-8d0f-e820359c82df)
![image](https://github.com/user-attachments/assets/943f4dcf-acd5-4429-b0cb-a71257b2510a)

## 2. Basic Linux Commands

### Manage Files and Directories

```bash
mkdir mydir            # Create a directory
cd mydir               # Navigate into a directory
cat file.txt           # Display contents of a file
```

### Copy and Rename Files

```bash
mv file1.txt file2.txt         # Rename file
mv file.txt /path/to/dir/      # Move file to another directory
rm file.txt                    # Delete a file
rm -r mydir/                   # Delete directory and its contents
rmdir emptydir/                # Delete an empty directory
cp source.txt dest.txt         # Copy file
```

![image](https://github.com/user-attachments/assets/cf07b31d-0009-4db9-8d30-36364ceab482)

### Find a File in Directory

```bash
find . -name "filename.txt"    # Search for file from current directory
```

![image](https://github.com/user-attachments/assets/fc018ae7-6d4b-46bc-97d2-9207166e7c74)

---

## 3. Navigating and Moving Files

```bash
pwd                            # Show present working directory
cd /path/to/directory          # Change directory
ls                             # List files and directories
ls -la                         # List with details including hidden files
mv myfile.txt ../              # Move file to parent directory
```

![image](https://github.com/user-attachments/assets/ac203922-9afb-4180-af8d-4bc14ff54131)

### Manage File/Directory Permission and Ownership
![image](https://github.com/user-attachments/assets/d60e3f68-e36a-42c8-88d4-46c5d2578d04)
![image](https://github.com/user-attachments/assets/f5c5317f-5dbf-43ae-b988-abf27962264f)
![image](https://github.com/user-attachments/assets/e52fba72-54dc-4c7d-910f-983ba708c56b)
![image](https://github.com/user-attachments/assets/6f90e495-b189-409b-aa53-f7b377a30cc6)

---

## 4. User and Group Management

### Create a User and Group

```bash
sudo useradd newuser           # Add new user
sudo addgroup newgroup         # Add new group
sudo usermod -aG newgroup newuser  # Add user to group
```
![image](https://github.com/user-attachments/assets/b92fe65b-21ac-4c91-aa56-c5b74aa4947f)

### Change User Details

```bash
sudo usermod -l newname oldname       # Change username
sudo usermod -s /bin/bash newuser     # Change default shell
```
![image](https://github.com/user-attachments/assets/344352e1-0370-4c1d-b0ed-8770215bce0c)

### Delete User and Group

```bash
sudo userdel newuser              # Delete user
sudo delgroup newgroup            # Delete group
```

### Change User Password

```bash
sudo passwd newuser
```

### Switch Users

```bash
su - newuser                      # Switch to another user
```
![image](https://github.com/user-attachments/assets/cadd3984-500d-48ab-9d15-16a2186282da)
#### Sudo file for User
![image](https://github.com/user-attachments/assets/ea73509c-e84d-4056-ba74-41042b83b044)
#### Sudo file for Group
![image](https://github.com/user-attachments/assets/57c9f8a2-4336-43c2-990e-f1c9b41890ca)

---

## 5. Advanced Linux Commands

### Zip Files

```bash
zip -r archive.zip directory/                     # Zip entire directory
zip -r archive.zip directory/ -x "*.log"          # Exclude .log files
```
![image](https://github.com/user-attachments/assets/fdb506a7-1909-48a0-a566-4ec1661ef4c7)

# Linux System Management Commands

## Disk Space Management

### `df` – Disk Free Space

```bash
df -h        # Human-readable disk usage
df -Th       # Show filesystem types along with usage
```
![image](https://github.com/user-attachments/assets/3132d547-b32f-4838-b6b0-e103c26c511d)

## Directory/Folder Usage

### `du` – Disk Usage

```bash
du -sh /var/log   # Display the total size of /var/log directory
du -sh *          # Show size of each item in the current directory
```
![image](https://github.com/user-attachments/assets/598cbaeb-a181-4d71-8103-3c44b0bf6176)

## Process Priority Management

### `nice` and `renice`

```bash
nice -n 10 command      # Start a command with lower priority
renice -n 5 -p PID      # Change priority of a running process by PID
```

## Application Management

### Using `apt` for Package Management

```bash
sudo apt update           # Update the list of available packages
sudo apt install nginx    # Install the Nginx web server
sudo apt remove nginx     # Remove Nginx from the system
sudo apt upgrade          # Upgrade all installed packages
```

## Network Management

### Common Network Commands

```bash
ip a                     # Show IP addresses and network interfaces
ping google.com          # Test network connectivity to a remote host
ss -tuln                 # Show listening ports and active connections
netstat -tulnp           # Show network statistics (legacy, may need to install)
nslookup google.com      # Perform a DNS lookup for a domain name
```
