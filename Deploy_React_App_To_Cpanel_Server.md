# This guide explain how you can deploy a React App to cpanel using SSH

## Step 1:

### 1. Configure SSH access to your server
If you have SSH access to your Cpanal Server, you can jump to Step 2.

**i.** Login to your cpanel system, under security section, navigate to SSH > Manage SSH Key > Create New Key > Fill required fields (remember your key password) > Create.

**ii** In the list of your keys, pick the one you creates > Manage > Authorize.

Then download the private key.

### 2. Configure the key in your machine

**i.** Open Terminal, change directory to Downloads Folder
```bash
cd ~/Downloads
```

**ii.** Move your key to ssh directory
```bash
mv id_rsa ~/.ssh/
# replace id_rsa with your key name
```
**iii.** Set the key with necessary permissions
```bash
chmod 600 ~/.ssh/id_rsa
```
### 3. Login to your server using the key 
Make sure the Shell access is enabled in your system. if not, contact your provider for support.

Use the following command
```sh
ssh -i ~/.ssh/id_rsa username@server_ip

# Example
ssh -i ~/.ssh/id_rsav2 jdf@69.72.248.123
```
If the server uses different SSH port rather than default(22), use this
```sh
ssh -i ~/.ssh/id_rsa username@server_ip -p port_number
```
If everything is okay, you will be prompted to enter the passphrase for your key, enter the key password and continue.

## Step 2:

### 1. Clone Git Repo

While you are connected to server, with your terminal;

**i. Navigate to home directory**
```bash
cd ~
```

**ii. Clone your repository**
```bash
git clone https://github.com/your-username/your-repo.git
```
Or, you can use this if the repo is private
```bash
git clone git@github.com:your-username/your-repo.git
```

### 2. Install dependencies

**Check if Node is installed in your server**
```bash
node -v
npm -v
```
**If Node is installed,**

install project dependencies
```bash
npm install
# or
npm i
```

**If Node is not installed in your system, Move to Step 4.**

### 3. Build the app

**a.** Create Build files
```bash
npm run build
# This will create build files at dist/ for Vite, and build/ for CRA
```

**b.** Deploy Build files to public_html
```bash
cp -r dist/* ~/public_html/
```
For CRA, use this;
```bash
cp -r build/* ~/public_html/
```

### 4. Test Website
Finally visit your domain to test your website
```sh
https://your-domain.com
```
## Step 3: 

### Configure Git Updates
Every when new changes are deployed to your git repo, you can update your site as follow:
```bash
cd ~/your-repo
git pull
npm install
npm run build
cp -r dist/* ~/public_html/
```

#### Optional: Auto deploy with single commad
Here, I have to create a script the will do all the stuffs for fetching updates and deploy.

 **i. Create a deploy script**
 ```bash
 cd ~           # make sure you are located at home directory
nano deploy.sh
```

**ii. Write the file**
```bash
#!/bin/bash

echo "---------------------------------------"
echo "🚀 Starting Deployment for JDF Website"
echo "---------------------------------------"

# Step 1: Go to project folder
cd ~/jdfsite

# Step 2: Pull latest code
echo "📥 Pulling latest changes from Git..."
git pull origin main

# Step 3: Build React app
echo "🏗️ Building project..."
npm install
npm run build

# Or use the following in npm is not added in environmental varibale
# ~/bin/node-v24.11.0-linux-x64/bin/npm install
# ~/bin/node-v24.11.0-linux-x64/bin/npm run build

# Step 4: Safe cleanup (preserve important cPanel folders)
echo "🧹 Cleaning old files (safe mode)..."
shopt -s extglob
cd ~/public_html
rm -rf !(".well-known"|"cgi-bin")

# Step 5: Copy new build files
echo "📤 Deploying build to public_html..."
cp -r ~/jdfsite/dist/* ~/public_html/

# Step 6: Finish
echo "---------------------------------------"
echo "✅ Deployment Complete!"
echo "---------------------------------------"

```
**iv. Run deployment**
Once you have pushed changes to github, connect the server and run this
```bash
./deploy.sh
```

## Step 4:
This step only concerns to the server that has not installed Node

### 1. Check if your server supports user-level binaries (without root previleges)
```bash
cd ~
mkdir bin
# If this works without error, your server allows user-level binaries installation locally
```

### 2. Check your system architecture
```bash
uname -m
# x86_64 → normal 64-bit or aarch64 → ARM server. 
# This article does not cover ARM Server
```

### 3. Download Node.js (pick the version you preffer)
In your home directory, write this;
```bash
curl -O https://nodejs.org/dist/v24.11.0/node-v24.11.0-linux-x64.tar.gz
```

### 4. Extract files
```bash
tar -xzf node-v24.11.0-linux-x64.tar.gz
```

### 5. Move the extracted folder to ~/bin
```bash
mv node-v24.11.0-linux-x64 ~/bin/node24
```

### 6. Add Node.js to your PATH

**i.** Edit .bashrc
```bash
nano ~/.bashrc
```

**ii.** Add this at the end of the file
```bash
export PATH=$HOME/bin/node24/bin:$PATH
```

**iii.** Save file
CTRL + O
ENTER
CTRL + X

**iv.** Reload Environment
```bash
source ~/.bashrc
```

**v.** Test Node Version
```bash
node -v
npm -v
```

## Step 5:

### Writting .htaccess rules for React SPA
React apps have no server-side routing, they depend on browser + index.html.
So, if you open deep links like (https://example.com/about, https://example.com/projects, https://example.com/whatever), without .htaccess you will get 404 Not Found.
To solve this, we need to define rules (rewrite rules) that will:
- Allow React SPA routings to work properly
- Not return to /index.html when appropriate file is available
- Operate similar with Cpanel/Apache environment
- Not interact with .well-known and cgi-bin directories

#### Procedures
Inside the public_html directory
```bash
nano .htaccess
```

```bash
# ---------------------------------------------------------
# React SPA Rewrite Rules
# ---------------------------------------------------------

RewriteEngine On
RewriteBase /

# Protect system folders from rewrite
RewriteRule ^\.well-known(/.*)?$ - [L]
RewriteRule ^cgi-bin(/.*)?$ - [L]

# Allow existing files & folders
RewriteCond %{REQUEST_FILENAME} -f [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^ - [L]

# Redirect everything else to index.html
RewriteRule ^ index.html [L]
```

After then, CTRL + O, Enter, CTRL + X. Test your routes.

### Configure Browser catching to improve speed
```bash
nano .htaccess
```

Add this to the top of document
```bash
# ---------------------------------------------------------
# Browser Caching – Improve performance
# ---------------------------------------------------------
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType text/html "access plus 1 hour"
</IfModule>
```
