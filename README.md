# Deploying a React App on Ubuntu Server with Nginx (AWS EC2)

A step-by-step guide to deploying a production React application on an AWS Ubuntu EC2 instance, served via Nginx.

---

## Prerequisites

- An active AWS account
- Basic familiarity with the AWS Console
- A terminal (Mac/Linux: built-in Terminal, Windows: Git Bash or WSL)

---

## Part 1 — Launch an Ubuntu EC2 Instance on AWS

1. Log in to your AWS Console at https://aws.amazon.com
2. In the search bar, type EC2 and click on it.
3. Click Launch instances.
4. Fill in the following settings:

   - Name: react-nginx-server
   - AMI: Ubuntu Server 22.04 LTS
   - Instance type: t2.micro (Free Tier eligible)
   - Key pair: Create a new key pair or select an existing one.
     If creating new: give it a name, leave type as RSA, format as .pem, and click Download.
     Save the .pem file somewhere safe — you cannot download it again.

5. Under Network settings, make sure these are enabled:
   - Allow SSH traffic from: Anywhere (0.0.0.0/0)
   - Allow HTTP traffic from the internet: checked

6. Leave storage as default (8 GB).
7. Click Launch instance.
8. Wait for the instance Status check to show: 2/2 checks passed before proceeding.

---

## Part 2 — Connect to Your Instance via SSH

1. Open your terminal.

2. Navigate to the folder where your .pem key file is saved:
```
   cd ~/Downloads
```

3. Set the correct permissions on your key file (required — SSH will reject it otherwise):
```
   chmod 400 your-key-name.pem
```

4. Go to your AWS Console → EC2 → Instances, click your instance, and copy the Public IPv4 address.

5. SSH into your instance:
```
   ssh -i your-key-name.pem ubuntu@YOUR_PUBLIC_IP
```

6. Type yes when prompted about the fingerprint.
   You should now see a prompt like:
```
   ubuntu@ip-xxx-xxx-xxx-xxx:~$
```
   You are now inside your server.

---

## Part 3 — Install Node.js and npm

Node.js is the engine that runs JavaScript on the server.
npm is the package manager that installs app dependencies.

1. Update the package list:
```
   sudo apt update
```

2. Install Node.js and npm:
```
   sudo apt install -y nodejs npm
```

3. Verify the installation:
```
   node -v && npm -v
```
   You should see version numbers printed for both.

---

## Part 4 — Install and Start Nginx

Nginx is the web server that will serve your React app to the internet.

1. Install Nginx:
```
   sudo apt install -y nginx
```

2. Start Nginx:
```
   sudo systemctl start nginx
```

3. Enable Nginx to auto-start on server reboot:
```
   sudo systemctl enable nginx
```

4. Confirm it is running:
```
   sudo systemctl status nginx
```
   You should see: active (running). Press Q to exit.

---

## Part 5 — Clone the React App

1. Install Git (it may not be pre-installed):
```
   sudo apt install -y git
```

2. Clone the React app repository:
```
   git clone https://github.com/pravinmishraaws/my-react-app.git
```

3. Move into the project folder:
```
   cd my-react-app
```

---

## Part 6 — Personalize the App

1. Navigate to the source folder:
```
   cd src
```

2. Open the App.js file in a text editor:
```
   nano App.js
```

3. Find these two lines and update them with your name and today's date:
```
   <h2>Deployed by: <strong>Your Full Name</strong></h2>
   <p>Date: <strong>DD/MM/YYYY</strong></p>
```

4. Save and exit:
   - Press Ctrl + O to save
   - Press Enter to confirm
   - Press Ctrl + X to exit

5. Go back to the project root:
```
   cd ..
```

---

## Part 7 — Install Dependencies and Build the App

React source code needs to be compiled into plain HTML, CSS, and JavaScript
before it can be served. This is called a production build.

1. Install all the app dependencies:
```
   npm install
```

2. Build the app for production:
```
   npm run build
```
   This creates a /build folder containing the compiled, browser-ready version of your app.

---

## Part 8 — Serve the React App via Nginx

Now you will move the compiled app into Nginx's serving directory
so Nginx can deliver it to anyone who visits your server's IP address.

1. Clear the default Nginx content:
```
   sudo rm -rf /usr/share/nginx/html/*
```

2. Copy your React build files into Nginx's serving directory:
```
   sudo cp -r build/* /usr/share/nginx/html/
```

3. Give Nginx ownership of the files:
```
   sudo chown -R www-data:www-data /usr/share/nginx/html
```

4. Set the correct file permissions:
```
   sudo chmod -R 755 /usr/share/nginx/html
```

---

## Part 9 — Configure Nginx for React Routing

By default Nginx serves files as-is. React apps handle their own routing
in the browser, so Nginx needs to be told: if a file path does not exist,
just serve index.html and let React handle it.

Without this config, refreshing any page other than the homepage
will return a 404 error.

1. Write the Nginx configuration:
```
   echo 'server {
     listen 80;
     server_name _;
     root /usr/share/nginx/html;
     index index.html;

     location / {
       try_files $uri /index.html;
     }

     error_page 404 /index.html;
   }' | sudo tee /etc/nginx/conf.d/react-app.conf > /dev/null
```

2. Test the Nginx configuration for errors:
```
   sudo nginx -t
```
   You should see: syntax is ok and test is successful.

3. Restart Nginx to apply the new configuration:
```
   sudo systemctl restart nginx
```

---

## Part 10 — Access Your Deployed App

1. Get your server's public IP:
```
   curl ifconfig.me
```

2. Open your browser and visit:
```
   http://YOUR_PUBLIC_IP
```

You should see your React app live, with your name and date displayed.

---

## Troubleshooting

**Page not loading in browser**
- Make sure port 80 is open in your EC2 Security Group (HTTP rule allows 0.0.0.0/0).
- Check Nginx is running: sudo systemctl status nginx
- Always use http:// not https:// — no SSL certificate was set up in this guide.

**Permission denied errors**
- Make sure you used chmod 400 on your .pem file before SSHing.
- Use sudo before commands that write to system directories.

**npm install or npm run build fails**
- Run sudo apt update first, then retry.
- If Node.js version is too old, upgrade it:
```
  sudo npm install -g n
  sudo n stable
```
  Then close and reopen your terminal and retry.

**Nginx shows default page instead of React app**
- Confirm your build files are in /usr/share/nginx/html/:
```
  ls /usr/share/nginx/html/
```
  You should see index.html and other files there.
- Restart Nginx: sudo systemctl restart nginx

---

## Clean Up (Important)

AWS charges for running EC2 instances. When you are done:

1. Go to AWS Console → EC2 → Instances.
2. Select your instance.
3. Click Instance state → Terminate instance.
4. Confirm termination.

---

*End of guide.*