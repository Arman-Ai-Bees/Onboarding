# Onboarding
## Step 0: Get Your VPN
At first, talk to the Admin to get your VPN account, set it up and go to the next step. 
*Note*: Most steps require a proper VPN to work, and not all VPNs are compatible.

## Step 1: Login to Google Cloud
1. Navigate to [Google Cloud Console](https://console.cloud.google.com/).
2. Login with the company email and select the project.

## Step 2: Generate SSH Key Pair
Run the following command on your local machine:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

## Step 3: Add the SSH Public Key
1. Go to **Compute Engine > Metadata > SSH Keys**.
2. Click **Edit** and paste your public key.

## Step 4: Create a VM
1. Navigate to **Compute Engine > VM Instances**.
2. Click **Create VM** and configure the instance as required.
    - Make sure you select duffieicent amount of CPU cores, RAM, and Storage.
3. Use the `SSH` option in the **Connect** column to access the server.

---

## Step 5: Configure SSH for Local Access
1. Open or create `~/.ssh/config`:
   ```text
   Host gcloud-instance
       HostName <External_IP>
       User <USER_NAME>
       IdentityFile <PRIVATE_KEY_PATH>
   ```

2. Connect via SSH:
   ```bash
   ssh -i "<PRIVATE_KEY_PATH>" <USER_NAME>@<External_IP>
   ```

---

## Step 6: Manually Add the Public Key (If Needed)
1. Click **SSH** in the VM's **Connect** column to open the terminal.
2. Add your public key manually:
   ```bash
   nano ~/.ssh/authorized_keys
   ```
   Paste the key and save.

3. Set permissions:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

4. Retry the connection.


----------
# Install Docker, Homebrew, and Vespa CLI


## **Step 1: Install Docker**

1. **Add Docker’s GPG Key**:
   ```bash
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

2. **Set Up the Docker Repository**:
   ```bash
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

3. **Update Package Lists**:
   ```bash
   sudo apt-get update
   ```

4. **Install Docker**:
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. **Verify Docker Installation**:
   - Run the test container:
     ```bash
     sudo docker run hello-world
     ```
   - Check system information:
     ```bash
     sudo docker info | grep "Total Memory"
     ```

---

## **Step 2: Install Homebrew**

1. **Install Homebrew**:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Set Up Environment Variables**:
   ```bash
   test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
   test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

   echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bashrc

   ```

3. **Verify Homebrew Installation**:
   ```bash
   brew install hello
   ```

---

## **Step 3: Install and Configure Vespa CLI**

1. **Install Vespa CLI**:
   ```bash
   brew install vespa-cli
   ```

2. **Set Vespa CLI Home Directory**:
   ```bash
   export VESPA_CLI_HOME=/tmp
   ```

3. **Set Target to Local**:
   ```bash
   vespa config set target local
   ```

---

## **Step 4: Deploy Vespa in a Docker Container**

1. **Run Vespa Container**:
   ```bash
   sudo docker run --detach --name vespa --hostname vespa-container \
   --publish 8080:8080 --publish 19071:19071 vespaengine/vespa
   ```

2. **Clone and Deploy an Application**:
   - Clone example application:
     ```bash
     vespa clone album-recommendation myapp && cd myapp
     ```
   - Deploy the application:
     ```bash
     vespa deploy --wait 300
     ```

3. **Feed Data**:
   ```bash
   vespa feed ext/documents.jsonl
   ```

---

## **Step 5: Perform Queries and Operations**

1. **Run Queries**:
   - Search for albums containing a specific word:
     ```bash
     vespa query "select * from music where album contains 'head'" language=en-US
     ```

   - Use ranking and input preferences:
     ```bash
     vespa query "select * from music where true" \
     "ranking=rank_albums" \
     "input.query(user_profile)={pop:0.8,rock:0.2,jazz:0.1}"
     ```

   - Include tensor formatting:
     ```bash
     vespa query "select * from music where true" \
     "ranking=rank_albums" \
     "input.query(user_profile)={pop:0.8,rock:0.2,jazz:0.1}" \
     "presentation.format.tensors=short-value"
     ```

2. **Retrieve a Document**:
   ```bash
   vespa document get id:mynamespace:music::a-head-full-of-dreams
   ```

3. **Visit Documents**:
   ```bash
   vespa visit
   ```

---



# Set Up Python Environment with Pyvespa
---

## **Step 1: Install Python and pip**

1. **Install pip**:
   ```bash
   sudo apt-get install python3-pip
   ```

2. **Install Python Virtual Environment**:
   ```bash
   sudo apt install python3.11-venv
   ```

---

## **Step 2: Create and Activate a Virtual Environment**

1. **Create a Virtual Environment**:
   ```bash
   python3 -m venv myenv
   ```

2. **Activate the Virtual Environment**:
   ```bash
   source myenv/bin/activate
   ```

---

## **Step 3: Install Pyvespa**

1. **Install Pyvespa**:
   ```bash
   pip install pyvespa
   ```

2. **Verify Installation**:
   Run the following command to ensure `pyvespa` is installed:
   ```bash
   pip show pyvespa
   ```

---

### Fixing Docker Permission Denied Issue:

1. **Check Docker Socket Permissions**:  
   ```bash
   ls -l /var/run/docker.sock
   ```
   - Confirm the group is `docker` and permissions are `srw-rw----`.

2. **Add User to Docker Group**:  
   ```bash
   sudo usermod -aG docker $USER
   ```
   - Replace `$USER` with your username (e.g., `remote-server`).

3. **Apply Changes**:  
   - Log out and back in, or reload your session:  
     ```bash
     newgrp docker
     ```

4. **Verify Group Membership**:  
   ```bash
   groups $USER
   ```
   - Ensure `docker` is listed.

5. **Test Docker Access**:  
   ```bash
   docker version
   ```
   - Confirm no permission error.

6. **Restart Docker (if needed)**:  
   ```bash
   sudo systemctl restart docker
   ```

7. **Temporary Workaround** (if needed):  
   - Run commands with `sudo`:  
     ```bash
     sudo docker version
     ```

If the above did not worked, follow this one:
# Solution 1: Create a custom Jupyter kernel with inherited environment
# Create a new directory for the custom kernel
```bash
mkdir -p ~/.local/share/jupyter/kernels/python3docker
```
# Create the kernel.json file
```bash
cat > ~/.local/share/jupyter/kernels/python3docker/kernel.json << 'EOL'
{
 "argv": [
    "python3",
    "-m",
    "ipykernel_launcher",
    "-f",
    "{connection_file}"
 ],
 "display_name": "Python 3 (with Docker)",
 "language": "python",
 "env": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/var/run/docker.sock"
 }
}
EOL
```
---
# Solution 2: Create a wrapper script for VSCode
```bash
cat > ~/vscode-docker-jupyter.sh << 'EOL'
#!/bin/bash
# Source the user's profile to get updated group memberships
if [ -f "$HOME/.profile" ]; then
    source "$HOME/.profile"
fi

# Ensure docker group membership is active
if ! groups | grep -q docker; then
    newgrp docker << EONG
    code .
EONG
else
    code .
fi
EOL

# Make the wrapper script executable
chmod +x ~/vscode-docker-jupyter.sh
```

---
# Solution 3: Create sudoers entry for passwordless docker access
```bash
# Run this command to create the sudoers entry
echo '# Create this file as: /etc/sudoers.d/jupyter-docker'
echo 'yourusername ALL=(ALL) NOPASSWD: /usr/bin/docker' 

# Solution 4: Update VSCode settings
echo '# Add to VSCode settings.json:'
echo '{
    "jupyter.kernelSpec.windowsUser": "'$(whoami)'",
    "terminal.integrated.inheritEnv": true,
    "jupyter.terminals.environmentVars": {
        "PATH": "${env:PATH}:/var/run/docker.sock"
    }
}'
```
