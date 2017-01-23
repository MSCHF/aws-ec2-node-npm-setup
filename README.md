# Steps to Install Node & NPM on EC2

Here are fast steps to set up node and npm on an EC2 instance. This is targeted for Node v4.x and can be adapted for whatever version you need. 

1) SSH into your EC2 instance.
2) Execute the below lines in order: 
    ```
    curl --silent --location https://rpm.nodesource.com/setup_4.x | sudo bash -
    sudo yum -y install nodejs
    sudo yum -y install gcc-c++ make
    curl --silent --location "https://www.npmjs.org/install.sh" | sudo bash -
    ```
3) Run `node --version && npm --version` to confirm it worked!

Credit: https://easyusedev.wordpress.com/2016/02/04/how-to-install-node-js-4-x-in-aws-ec2-instance/

