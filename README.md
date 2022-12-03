# 2420_assign2
##Description
Create VPC
create a Node Balancer and how that Node balancer will forward http requests to two different droplets. 
Install Volta and then Nodejs and npm on server
Create proxy server in Caddy


## Step-One: Create VPC, droplets, Load balancer and firewall
Create a VPC, Two droplets(Server1 and server2), Load balancer and firewall: <a href="https://vimeo.com/775412708/4a219b37e7" target="_blank">refer the video</a>



## Step-Two:  Create regular users on Droplets
 
 Perform the following steps on created droplets
1. Create a new user using the following command
```useradd -s /usr/bin/bash -m <username>```
* Name the user server
* Give the new user a password with ``` passwd <username> ```

Switch the user to the new user using  ``` su <username> ```

Add the public ssh key from the root user to new user:
* Inside new user home directory ```mkdir .ssh```
* Switch to your root user and copy the authorized_keys file to the new users .ssh with
``` cp ~/.ssh/authorized_keys /home/<username>/.ssh/ ```


## Step-Three: Install Caddy
 To install Caddy 
* Switch to root user and run the following commands
``` 
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```


## Step Four - Write Your Web App
* Create a new directory on your local machine 2420-assign-two
* Inside this new directory create 2 new directories: src & html
* Inside the html directory, create an index.html file with appropriate html content
```
<!DOCTYPE html>
<html lang="en">
<html>
    <head>
                <meta charset="UTF-8" />
        <title>Example Site for 2420</title>
    </head>
    <body>
        <h1>Success! Caddy This is Server 1 </h1>
        <h2 style="color: red;">All your internets are belong to us!</h2>
    </body>
</html>
```

* Inside src directory, create a new node project, you will need to install fastify
Follow these commands step by step. Run these in your src directory:
``` npm init ```
```  npm install fastify ```


* Inside src create an index.js file
```
// Require the framework and instantiate it
const fastify = require('fastify')({ logger: true })

// Declare a route
fastify.get('/', async (request, reply) => {
  return { hello: 'Server x' }
})

// Run the server!
const start = async () => {
  try {
    await fastify.listen({ port: 3000 })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()

```

* Once complete, use sftp to move the html and src directories to both server droplets.
```
sftp -i ~/.ssh/{key_file_name} {user}@{droplet_ip}
put -r src
put -r html
```
## Step Five - Caddyfile

On your local machine, you will write a Caddyfile, which will be your configuration file for caddy
```
http:// {
	root * /var/www/html
	reverse_proxy /api localhost:5050
	file_server
}

```



## Step Six - Installing Node and Npm with Volta

To do so follow the commands:

``` curl https://get.volta.sh | bash ```
```  source ~/.bashrc ```
```  volta install node ```
``` npm install fastify ```

## Step Seven - Node App Service File
 write a service file on your local machine to start your node application
 * On your local machine create file called hello_web.service and add the following content:
 ```
 [Unit]
Description=run node app on localhost:5050
After=network.target

[Service]
Type=simple
User=server
Group=server
ExecStart=/home/server/.volta/bin/node /var/www/src/index.js
Restart=on-failure
RestartSec=5
SyslogIdentifier=hello_web

[Install]
WantedBy=multi-user.target
 
 ```
* Copy this file to both your droplets
* Within your droplets move this file to the /etc/systemd/system/ directory: sudo mv hello_web.service /etc/systemd/system


* To test the service, run the following commands:
```
sudo systemctl daemon-reload
sudo systemctl enable hello_web.service
sudo systemctl restart hello_web.service
systemctl status hello_web.service
```

## Step-Eight: Test Your Load Balancer
 you should be able to access your Load Balancer's IP address and see the HTML content from both droplets, you should also be able to see the node app of both droplets by visiting your Load Balancer's API route.
 
 To test Your Load Balancer follow the steps below:
 visit http://24.199.71.46/
 
