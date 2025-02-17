# Linux Administrator - HypotheticalCorp

## **Scenario**
You are a linux Administrator at HypotheticalCorp. Your tasks include managing users, ensuring security, monitoring system performance, and handling application depolyments.

---

## **1️⃣ User and Roles Management**
### ✅  **Task**: Create Users and Assign Them to the `developers` Group

**I created **five developers (`dev1` to `dev5`)** and added them to a group.**

#### **Commands Used**
```bash
sudo useradd -m -G developers dev1
sudo useradd -m -G developers dev2
sudo useradd -m -G developers dev3
sudo useradd -m -G developers dev4
sudo useradd -m -G developers dev5
```
![Developer group](/linux_assessment/verifying_user_in_group.png)

**I then set passwords for each user**

```bash
sudo passwd dev1
sudo passwd dev2
sudo passwd dev3
sudo passwd dev4
sudo passwd dev5
```

**Issue Encountered & Solution:**
- Just creating the users was not enough for the SSH access.
- I had to configure SSH authentication (which i will soon get to)

---
#### ✅ **Task**: Set Permission for **`/var/www/project`** 

I created a project directory and set the correct permissions

#### **Command used**
```bash
sudo mkdir -p /var/www/project
sudo chown -R root:developers /var/www/project
sudo chmod -R 750 /var/www/project
ls -ld /var/www/project
```
![verify permissions](/linux_assessment/verify_permissions.png)
#### ✅  **Task**: Restrict SSH Access for **`dev4`** & **`dev5`**

i edited the SSH configuration file
```bash
sudo nano /etc/ssh/sshd_config
```
then i added the following line to the bottom of the file
```bash
DenyUsers dev4 dev5
```
Saved and restarted SSH
```bash
sudo systemctl restart sshd
ssh dev4@my-ece-public-ip
```
**Issue Encountered & Solution:**
- Applying `DenyUsers` did **not** automatically allow SSH for `dev1`, `dev2`, and `dev3`.
- I had to manually configure SSH kay-based authentication for these users.
- I made sure all neccessary devs had the **`/home/dev1/.ssh`** path
```bash
sudo mkdir -p /home/dev1/.ssh
sudo mkdir -p /home/dev2/.ssh
sudo mkdir -p /home/dev3/.ssh
```
- I copied the authorized key from the admin and used it to give the allowed devs verified access.
```bash
cat ~/.ssh/authorized_keys
echo "copied-authorized-key" | sudo tee -a /home/dev1/.ssh/authorized_keys
```
since it is just that particular key in the `admin` authorized_keys file, and the authorized_keys file for the `devs` is empyt, this could also work
```bash
sudo cp ~/.ssh/authorized_keys /home/dev1/.ssh/authorized_keys
sudo cp ~/.ssh/authorized_keys /home/dev2/.ssh/authorized_keys
sudo cp ~/.ssh/authorized_keys /home/dev3/.ssh/authorized_keys
```
- I then restricted dev access to the SSH file
```bash
sudo chmod 700 -R /home/dev1/.ssh /home/dev2/.ssh /home/dev3/.ssh
sudo chmod 600 /home/dev1/.ssh/authorized_keys /home/dev2/.ssh/authorized_keys /home/dev3/.ssh/authorized_keys
```
Saved and restarted SSH
```bash
sudo systemctl restart sshd
ssh dev4@my-ece-public-ip
```

---

## **2️⃣ System Monitoring & Performance Analysis**

### **Task:** Identify top resource-consuming processes.

I identified the top resource-consuming process with
```bash
top
```
i checked the disk usage with
```bash
df -h
du -sh /var/log/*
```
I monitored real-time logs with
```bash
sudo journalctl -f
```
**Issue encountered & Solution:**
no particular issue, but i tried to clear old logs to free up more space by running:
```bash
sudo rm -rf /var/log/*.gz
sudo journalctl --vacuum-time=3d
```

## **3️⃣ Application Management**
### **Task:** Install and configure Nginx

i installed and started Nginx with the following command
```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```
verified that Nginx was running:
```bash
sudo systemctl status nginx
```
---

## **4️⃣ Networking and Security**
### **Task:** Restrict SSH access for `dev4` and `dev5`, allowing only local logins.

```bash
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status
```
i checked the open ports using
```bash
sudo ss -tulnp
```
### **Task:** Set up SSH key-based authentication to eliminate password logins.
I generated a new SSH Key
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/dev-key
```
i then copied the public key to `dev1`, `dev2`, and `dev3`
```bash
sudo cp ~/.ssh/authorized_keys /home/dev1/.ssh/authorized_key /home/dev2/.ssh/authorized_keys /home/dev3/.ssh/authorized_keys
```
I fixed ownership and permissions
```bash
sudo chown -R dev1:dev1 /home/dev1/.ssh
sudo chown -R dev2:dev2 /home/dev2/.ssh
sudo chown -R dev3:dev3 /home/dev3/.ssh

sudo chmod 700 /home/dev1/.ssh /home/dev2/.ssh /home/dev3/.ssh
sudo chmod 600 /home/dev1/.ssh/authorized_keys /home/dev2/.ssh/authorized_keys /home/dev3/.ssh/authorized_keys
```
Restart and test
```bash
sudo systemctl restart sshd
```