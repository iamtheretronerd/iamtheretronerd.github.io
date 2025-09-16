# Linux System Administration Guide
<!-- Ctrl+shift+v to open -->
## User Administration

### Create a New Administrator User

```bash
sudo adduser administrator
```
*This command will prompt for a password and create a home directory automatically.*

### Add User to Sudoers Group

```bash
sudo usermod -aG sudo administrator
```
*This grants sudo access to the administrator user.*

To verify sudo access:
```bash
id administrator
```

### Set Home Directory Permissions

Ensure the administrator's home directory has full permissions for the user only:

```bash
cd /home
chmod 700 administrator
```
*Permission 700 = Full access for user (7), no access for group (0) and others (0)*

---

## Docker Management

### Install Docker

```bash
sudo apt install docker.io
```

### Add User to Docker Group (Optional)

```bash
sudo usermod -aG docker administrator
```
*This allows the user to run Docker commands without sudo.*

### Pull Apache Web Server Container

```bash
docker pull httpd
```

### Run Docker Container

```bash
sudo docker run -d --name apache1 -p 8080:80 httpd
```
*This runs Apache in detached mode, mapping port 8080 on host to port 80 in container.*

### Docker Container Operations

Check running containers:
```bash
sudo docker ps
```

Access container shell:
```bash
docker exec -it apache1 /bin/bash
```

Exit container shell:
```bash
exit
```

Restart container:
```bash
docker restart apache1
```

---

## SSH Configuration

### Generate SSH Keys

Create ED25519 key pair:
```bash
ssh-keygen -t ed25519
```
*This generates both private and public keys.*

### Connect via SSH

Basic SSH connection:
```bash
ssh -i demo root@ipaddress -p 22
```

### Secure Copy (SCP)

Transfer files over SSH:
```bash
scp -i demo new_key.pub ben@161.35.129.236:/home/ben/new_key.pub
```

With custom port:
```bash
scp -i demo new_key.pub ben@161.35.129.236:/home/ben/new_key.pub -P 23
```

**SCP Components:**
- `scp` - Secure copy command
- `-i demo` - Specify private key for authentication
- `new_key.pub` - Source file
- `ben@161.35.129.236` - Username and server IP
- `/home/ben/new_key.pub` - Destination path

---

## Setting Up SSH Key Authentication

### 1. Change Key Ownership

```bash
sudo chown bob:bob new_key.pub
```

### 2. Create SSH Directory Structure

```bash
mkdir .ssh
chmod 700 .ssh
```

### 3. Create Authorized Keys File

```bash
cd .ssh
touch authorized_keys
chmod 600 authorized_keys
```

### 4. Move and Configure Public Key

Move the public key:
```bash
sudo mv new_key.pub /home/bob
```

Copy key content to authorized_keys:
```bash
cat new_key.pub > .ssh/authorized_keys
```

Remove the original file:
```bash
rm new_key.pub
```

### 5. Connect Using SSH Key

```bash
ssh -i new_key bob@ipaddress -p 22
```

---

## Security Hardening

### Strengthen Password Policies

Install password quality checking library:
```bash
sudo apt install libpam-pwquality
```

Configure password requirements:
```bash
sudo nano /etc/security/pwquality.conf
```
*Modify the values in the configuration file according to your security requirements.*

### Change DNS Settings

Edit resolver configuration:
```bash
sudo nano /etc/resolv.conf
```

Example DNS server configuration:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

---

## OpenSSL Certificate Management

### Creating a Self-Signed Certificate

Generate a self-signed certificate with private key:
```bash
sudo openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes
```

**Certificate Details (Example for NEU):**
- **Country Name:** US
- **State:** MA
- **Locality:** Boston
- **Organization Name:** NEU
- **Organizational Unit:** Cybersecurity
- **Common Name:** [Your Northeastern Username]
- **Email Address:** [Your Northeastern Username]@northeastern.edu

*This command generates two files:*
- `server.key` - Private key file
- `server.crt` - Certificate file

### Setting Up OpenSSL Server

Start an OpenSSL server listening on a specific port:
```bash
sudo openssl s_server -cert server.crt -key server.key -accept [Your_Port_Number]
```
*Use the last 4 digits of your NUID as the port number (e.g., NUID 001234556 → Port 4556)*

### Connecting with OpenSSL Client

1. **Open a new terminal session:**
   ```bash
   ssh username@<your_server_ip>
   ```

2. **Start the OpenSSL client:**
   ```bash
   sudo openssl s_client -connect localhost:[Your_Port_Number]
   ```

3. **Test the connection:**
   - Type a message (e.g., "Hi") in the client terminal
   - The message should appear on the server side
   - This confirms successful SSL/TLS connection establishment

---

## Quick Reference

### Permission Values
- **7** (rwx) = Read (4) + Write (2) + Execute (1)
- **6** (rw-) = Read (4) + Write (2)
- **4** (r--) = Read only
- **0** (---) = No permissions

### Common Permission Sets
- **700** = Full access for owner, none for others
- **755** = Full for owner, read/execute for group and others
- **644** = Read/write for owner, read-only for others
- **600** = Read/write for owner only

### Key File Permissions
- `.ssh` directory: **700**
- Private keys: **600**
- `authorized_keys`: **600**
- Public keys: **644** (can be more permissive)

---

## Best Practices & Notes

### General Security
- Always backup configuration files before modifying them
- Test SSH connections before disabling password authentication
- Keep private keys secure and never share them
- Use strong passwords when creating new users
- Regularly update and patch your system

### OpenSSL Security
- Store certificates and keys in secure locations with appropriate permissions
- Use strong key sizes (4096-bit RSA or higher)
- Set appropriate certificate validity periods
- Never share private keys
- Consider using Certificate Authorities for production environments

### Docker Security
- Avoid running containers as root when possible
- Keep Docker and container images updated
- Use official images from trusted sources
- Limit container capabilities and resources
- Network isolation between containers when appropriate