# FreeTAKHub Node-RED
This document describes how to install and secure Node-RED.
- Install Using Script (simple)
- Install using APT (advanced Users)
- Secure Node-RED

# Install Using Script (simple)

The following script will do Installing and Upgrading Node-RED to install Node.js, npm and Node-RED onto a Ubuntu Raspberry Pi. The script can also be used to upgrade an existing install when a new release is available.

Running the following command will download and run the script. If you want to review the contents of the script first, you can view it on Github.
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

Enable service so that starts automagically:
```
sudo systemctl enable nodered.service
```
Run the Node-RED service:
```
node-red-start
```
Navigate to your Node-RED:
```
http://<IP>:1880
```

# Install Using APT (advanced users)
To install Node-RED with APT you will need to:
- Satisfy the requirements
- Create a Non root user
- Install Node JS

## Prerequisites
This guide assumes that you are using Ubuntu 20.04 on cloud installation (we use digital Ocean). 


##  Create a Non-root user
First create a special user and dedicated group:

```
sudo useradd -m nodered -G nodered
```

## Install Node JS
 you can use the apt package manager. Refresh your local package index first by typing:

```
sudo apt update
```

Then install Node.js:

```
sudo apt install nodejs
 ```
 
Check that the install was successful by querying node for its version number:
```
node -v
 ```
Output something like this
```
v10.19.0
```

If the package in the repositories suits your needs, this is all you need to do to get set up with Node.js. In most cases, you’ll also want to also install npm, the Node.js package manager. You can do this by installing the npm package with apt:

```
sudo apt install npm
```
This will allow you to install modules and packages to use with Node.js.

## Install Node-RED
Use npm to install node-red and a helper utility called node-red-admin.

```
sudo npm install -g --unsafe-perm node-red node-red-admin
```

npm normally installs its packages into your current directory. Here, we use the -g flag to install packages ‘globally’ so they’re placed in standard system locations such as /usr/local/bin. The --unsafe-perm flag helps us avoid some errors that can pop up when npm tries to compile native modules (modules written in a compiled language such as C or C++ vs. JavaScript).

After a bit of downloading and file shuffling, you’ll be returned to the normal command line prompt. Let’s test our install.

First, we’ll need to open up a port on our firewall. Node-RED defaults to using port 1880, so let’s allow that.
```
sudo ufw allow 1880
``` 
And now launch Node-RED itself. No sudo is necessary, as port 1880 is high enough to not require root privileges.

```
node-red
```
## Create the Service
In order to start Node-RED automatically on startup, we’ll need to install a node-red.service file: 

Let's creates the service using the Tee command:
```
sudo tee /etc/systemd/system/node-red.service >/dev/null << EOF
Description=Node-RED
After=syslog.target network.target

[Service]
ExecStart=/usr/local/bin/node-red-pi --max-old-space-size=128 -v
Restart=on-failure
KillSignal=SIGINT

# log output to syslog as 'node-red'
SyslogIdentifier=node-red
StandardOutput=syslog

# non-root user to run as
WorkingDirectory=/home/nodered/
User=nodered
Group=nodered

[Install]
WantedBy=multi-user.target
EOF
```
 
Now that our service file is installed , we need to enable it. This will enable it to execute on startup.
```
sudo systemctl enable node-red
```

Let’s manually start the service now to test that it’s still working.
```
sudo systemctl start node-red
 ```
To shut it back down you can use:

```
sudo systemctl stop node-red
```
## Test Your Installation 
Point a browser back at the server’s port 1880 and verify that Node-RED is back up. e.g. if your server is installed under the IP 143.198.39.135
``` browser
http://143.198.39.135:1880/
```
You will see the welcome screen of Node-RED with the tutorial.
Now, to install the flows, click on the hamburger menu and then import:

![image](https://user-images.githubusercontent.com/60719165/143110628-d5e1d2b9-15e8-4b34-b977-abdc99c205f9.png)

Download one of the FreeTAKHub integrations (json files).
Import into your Node-RED. 

Resolve any conflicts by importing additional nodes into your palette. 

You can find "Manage palette" from top right corner, "setting" > "Manage palette"

![Image 089](https://user-images.githubusercontent.com/11376362/219843959-b494a03c-6da9-4714-bece-f521ae1c2f4c.png)

By clicking "Manage palette" a "User setting" page will pop up. 

go to "Palette" > "install" tab and type "node-red-contrib-config" in the search bar

![Image 090](https://user-images.githubusercontent.com/11376362/219844290-b6a5d06d-ce16-4f44-928d-b316e7cc6c5d.png)

Success!!!!

![image](https://user-images.githubusercontent.com/60719165/143122002-35f25669-17c3-4dfa-9655-14b52612bd04.png)

# Secure Node-RED

By default, Node-RED is installed with no authentication to the user interface (UI), also called the "Editor".  This means that anyone with your IP address can make changes to any of your flows, or even add malicious flows to your Node-RED instance.  Once your Node-RED instance is up-and-running, you should **immediately** secure the editor with a password, at a minimum.

The following will get you started on securing the Node-RED editor with a password.  
For more details on how to secure Node-RED, see the Node-RED docs here:
https://nodered.org/docs/user-guide/runtime/securing-node-red

Switch to the Node-RED folder:
```
cd ~/.node-red
```

Edit the settings.js file:

```
sudo nano settings.js
```

Uncomment the adminAuth section 
![image](https://user-images.githubusercontent.com/60719165/197415369-020905e2-1cb2-4ab5-aa5d-cfe3cf3626ef.png)


 save the file and Exit.

```
 CTRL+o
 ENTER
 CRTL+X
```

You can now login to your Node-RED editor [YOUR-IP-ADDRESS]:1880.  The default user name is admin, and the default password is password.
This isn't enough.  
You'll want to create a unique password, which will require creating a password hash.  
This is detailed in the Node-RED docs here:
https://nodered.org/docs/user-guide/runtime/securing-node-red#editor--admin-api-security
