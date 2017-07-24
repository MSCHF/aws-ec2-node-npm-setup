# SO YOU WANNA SET UP A NODE-BASED EC2 INSTANCE HUH!?


## Prerequisites

* Set up an EC2 instance

* Download the Key Pair file (.pem) and keep it somewhere safe. 

* Run `chmod 400 ~/path/to/key.pem` otherwise you cannot SSH in. 


## Steps to Install Node & NPM on EC2

Here are fast steps to set up node and npm on an EC2 instance. This is targeted for Node v4.x and can be adapted for whatever version you need.

1. SSH into your EC2 instance.

2. Execute the below lines in order:

	`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash`

	`exit`

	*Log back into the server*

	`nvm install node`

3. Run `node --version && npm --version` to confirm it worked!

4. Lastly, you need to alias `sudo` since `nvm` doesn't alter the PATH for privileged users. Add this to the end of your ~/.bashrc file. 

  `alias sudo='sudo env PATH=$PATH:$NVM_BIN'`


## Installing git

If you're on Amazon Linux simply run `sudo yum install git`. If you're on Ubuntu just type `git` to see if it is installed. If it isn't, type `sudo apt-get install git`.


## VIM Setup

First install Vundle, a Vim package manager. 

Run `git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`. 

Then grab your `.vimrc` file, for example I run: `git clone https://github.com/tetreault/vimrc.git && cp vimrc/.vimrc ~/.`.

Then run `vim` and type `:PluginInstall`. 


## Caching Git credentials 

Run the following: `git config --global credential.helper 'cache --timeout=99999'`. 

This should allow you to push or pull without continuously entering your credentials. 


## HTTPS with EFF Cert Bot on Ubuntu EC2

Stop NGINX and any server running, certbot needs those ports. 


Simply run these steps in order!

```
cd /home/ubuntu
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto certonly --standalone -d xyz.YOURDOMAIN.com
```


## Setting up NGINX on Ubuntu EC2

### Installing NGINX

Download the NGINX signing key and add it to the keychain (allows us to DL NGINX from 3rd party source)

`wget https://nginx.org/keys/nginx_signing.key -O - | sudo apt-key add -`


Go to the apt directory

`cd /etc/apt`


Open sources.list to add the nginx repository: 

```
deb http://nginx.org/packages/ubuntu trusty nginx
deb-src http://nginx.org/packages/ubuntu trusty nginx
```


Install NGINX!

`sudo apt-get install nginx`


Update NGINX

`sudo apt-get update`


### Setting up NGINX

#### /etc/nginx/conf.d/default.conf file
Change the config file name `cd /etc/nginx/conf.d && mv default.conf default.conf.bak`


#### nginx.conf file
Go back a directory, remove the `nginx.conf` file and re-open it using `cd ../ && rm nginx.conf && nano nginx.conf`: 

```
#######################################
#
# File: /etc/nginx/nginx.conf
#
#######################################
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
}

http {

	##
	# Basic Settings
	##
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```


#### /etc/nginx/sites-available/default file

Go to the root of the nginx subdir `cd /etc/nginx/` and type `mkdir sites-available/` then we will create the default file there with `nano sites-available/default`

```
########################################
#
# /etc/nginx/sites-available/default
#
########################################

##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##


upstream YOUR_DOMAIN_NAME {
  server localhost:8080 fail_timeout=0;
  keepalive 60;
}

#HTTP
server {
  listen 80;
  listen [::]:80;
  server_name    YOUR_DOMAIN_NAME;
  return         301 https://$server_name;
}

# HTTPS - proxy requests on to local Node.js app:
server {
        listen 443;
        server_name localhost:443;

        ssl on;
        # Use certificate and key provided by Let's Encrypt:
        ssl_certificate /etc/letsencrypt/live/supersecretprobablyclassified.website/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/supersecretprobablyclassified.website/privkey.pem;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

        # Pass requests for / to localhost:8080:
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:8080/;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
                proxy_cache_bypass $http_upgrade;
                proxy_redirect off;
        }
}
```


#### /etc/nginx/sites-enabled/default file

Go to the root of the nginx subdir `cd /etc/nginx/` and type `mkdir sites-enabled/` then we will create the default file there with `nano sites-enabled/default`

```
########################################
#
# /etc/nginx/sites-enabled/default
#
########################################

##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##


upstream YOUR_DOMAIN_NAME {
  server localhost:8080 fail_timeout=0;
  keepalive 60;
}

#HTTP
server {
  listen 80;
  listen [::]:80;
  server_name    YOUR_DOMAIN_NAME;
  return         301 https://$server_name;
}

# HTTPS - proxy requests on to local Node.js app:
server {
        listen 443;
        server_name localhost:443;

        ssl on;
        # Use certificate and key provided by Let's Encrypt:
        ssl_certificate /etc/letsencrypt/live/supersecretprobablyclassified.website/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/supersecretprobablyclassified.website/privkey.pem;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

        # Pass requests for / to localhost:8080:
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:8080/;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
                proxy_cache_bypass $http_upgrade;
                proxy_redirect off;
        }
}
```


### Moment Of Truth

- Restart NGINX by typing `sudo service nginx restart`
- Then when you are ready to see if this failed or not, go to your project folder and start the node server, then go to the URL in your browser. 
- If you get a 50* error in the browser then something went wrong. Buckle up - time to troubleshoot. Go over the above steps to isolate the issue. 


## Steps to Add SSL to your EC2 Instance (non-Cert-bot method)

Either use EFF's [CertBot](https://certbot.eff.org/) or follow the below: 

1. In the security group for your EC2 instance, make sure you add the https rule (port 443 to '0.0.0.0/0').

2. Give your EC2 instance an elastic IP.

3. Buy a domain! (I suggest NameCheap)

4. Create an A record for the domain in Route53.

5. Run `which openssl` to see if you have OpenSSL installed already on the EC2 instance.

6. Generate a private key! Go to where you want to keep the key (mine's /etc/ssl/private) and run `openssl genrsa 2048 > privatekey.pem.

7. Generate your certificate signing request in the directory you want to keep them in (mine's /etc/ssl/certs) `openssl req -new -key privatekey.pem -out csr.pem`.

8. You will have to fill out contact info and stuff. One important note: for the common name make sure you enter it like example.com, not http://example.com or www.example.com nor should you use the IP address.

9. Buy an SSL cert! I suggest NameCheap, the Komodo basic SSL cert is $9. I suggest you use CNAME host record form of verifying that you own the domain. 

10. You will eventually receive a ZIP file in your email, this will contain the bundle file and crt.

11. You need to Secure Copy the ZIP file to your EC2 server. Then you need to put them where you put your private key and cert in the prior step (again, mine is /etc/ssl/certs and /etc/ssl/private). Here's the example syntax: `scp -i ~/path/to/pem/file ec2-user@XXXXXXXXXXXXXXXXXXXXXXXXXXXX:~/some/dir/where/upload/goes`. Once the files are on the server move them to that key/cert location. 

12. You then need to configure your server script so it works for HTTPS: 

    ```javascript
    const fs=require('fs');

    const https = require('https');

    const PORT = 443; 

    const app = express();

    const server = https.createServer({
          ca: fs.readFileSync('/etc/ssl/certs/sitename_com.ca-bundle', 'utf-8'),
          key: fs.readFileSync('/etc/ssl/private/privatekey.pem', 'utf-8'),
          cert: fs.readFileSync('/etc/ssl/certs/sitename_com.crt', 'utf-8')
    }, app);
    ```

## Adding A Public Key to Grant SSH Access for A User

1. Get the user's public key

2. Secure copy the user's public key to the EC2 instance: 

    `scp -i ~/path/to/key.pem new_user_key.pub (ubuntu or ec2-user)@ec2-your-instance-name.compute.amazonaws.com:/tmp`

3. SSH into the server yourself as root to create the user account with the following commands: 

    `sudo su && useradd -c "firstname lastname" user -m`

4. Now you have to place their key into their SSH authorized keys file:

    `cd ~user`

    `mkdir .ssh`
    
    `chmod 700 .ssh`
    
    `chown user:user .ssh`
    
    `cat /tmp/new_user.pub >> .ssh/authorized_keys`
    
    `chmod 600 .ssh/authorized_keys`
    
    `chown user:user .ssh/authorized_keys`

5. Now have the user test logging in via SSH and they should be ready to go! 

    `ssh -i ~/path/to/their/key.pem user_name@ec2-your-instance-name.compute.amazonaws.com`

Source: https://aws.amazon.com/articles/1233/
