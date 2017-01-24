# SO YOU WANNA SET UP A NODE-BASED EC2 INSTANCE HUH!? 

## Steps to Install Node & NPM on EC2

<p>Here are fast steps to set up node and npm on an EC2 instance. This is targeted for Node v4.x and can be adapted for whatever version you need.</p> 

<ol>
    <li>SSH into your EC2 instance.</li>
    <li>
        <p>Execute the below lines in order:</p> 
        <p>`curl --silent --location https://rpm.nodesource.com/setup_4.x | sudo bash -`</p>
        <p>`sudo yum -y install nodejs`</p>
        <p>`sudo yum -y install gcc-c++ make</p>
        <p>`curl --silent --location "https://www.npmjs.org/install.sh" | sudo bash -`</p>
    </li>
    <li>Run `node --version && npm --version` to confirm it worked!</li>
</ol>

## Steps to Add SSL to your EC2 Instance

<ol>
    <li>In the security group for your EC2 instance, make sure you add the https rule (port 443 to '0.0.0.0/0').</li>
    <li>Give your EC2 instance an elastic IP.</li>
    <li>Buy a domain! (I suggest NameCheap)</li>
    <li>Create an A record for the domain in Route53.</li>
    <li>Run `which openssl` to see if you have OpenSSL installed already on the EC2 instance.</li>
    <li>Generate a private key!</li>

</ol>


Credit: https://easyusedev.wordpress.com/2016/02/04/how-to-install-node-js-4-x-in-aws-ec2-instance/

