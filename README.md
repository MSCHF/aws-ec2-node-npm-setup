# SO YOU WANNA SET UP A NODE-BASED EC2 INSTANCE HUH!?

## Prerequisites

* Set up an EC2 instance

* Download the Key Pair file (.pem) and keep it somewhere safe. 

* Run `chmod 400 ~/path/to/key.pem` otherwise you cannot SSH in. 

## Steps to Install Node & NPM on EC2

Here are fast steps to set up node and npm on an EC2 instance. This is targeted for Node v4.x and can be adapted for whatever version you need.

1. SSH into your EC2 instance.

2. Execute the below lines in order:

	`curl --silent --location https://rpm.nodesource.com/setup_4.x | sudo bash -`

	`sudo yum -y install nodejs`

	`sudo yum -y install gcc-c++ make`

	`curl --silent --location "https://www.npmjs.org/install.sh" | sudo bash -`

3. Run `node --version && npm --version` to confirm it worked!

## Installing git

Simply run `sudo yum install git`.

## Steps to Add SSL to your EC2 Instance

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
          ca: fs.readFileSync('/etc/ssl/certs/internetpillowfight_com.ca-bundle', 'utf-8'),
          key: fs.readFileSync('/etc/ssl/private/privatekey.pem', 'utf-8'),
          cert: fs.readFileSync('/etc/ssl/certs/sitename_com.crt', 'utf-8')
    }, app);

