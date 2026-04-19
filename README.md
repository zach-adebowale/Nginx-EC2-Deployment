# Nginx Deployment on AWS EC2 with a Custom Domain 

This project involves setting up a simple web server by launching an AWS EC2 instance, installing and configuring Nginx, and then connecting a custom domain to it using DNS. 

It serves as a hands-on demonstration of how servers are deployed and made accessible on the internet.

## Architecture diagram

            +----------------+----------------+
            |            Browser              |
            |  http://nginx.adebowale.co.uk   |
            +----------------+----------------+
                             |
                             v 
                 +-----------------------+
                 |      Cloudflare       |
                 |    DNS (A record)     |
                 |  nginx -> Public IPv4 |
                 +-----------+-----------+
                             |
                             v 
              +-----------------------------+
              |           AWS EC2           |
              |-----------------------------|
              |  Ubuntu (t3.micro)          |
              |  Nginx (running)            |
              |                             |
              |  Public IPv4                |
              |                             |
              |  Security Group:            |
              |   - 22 (SSH - My IP)        |
              |   - 80 (HTTP - in use)      |
              |   - 443 (HTTPS - optional)  |
              +--------------+--------------+
                             ^
                             |
                  +----------+----------+
                  |      SSH Client     |
                  |  (User’s Computer)  |
                  | SSH Key Pair (.pem) |
                  +---------------------+

## Technologies used

* **AWS EC2** - Cloud compute service used to host the server
* **Nginx** - Web server used to serve HTTP content
* **Cloudflare** - DNS provider used to map the domain to the server
* **Ubuntu** - Operating system running on the EC2 instance

## Getting a domain

A domain was purchased from Cloudflare.
For this setup I used `adebowale.co.uk`

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/Domain%20status.png)

## Setting up the EC2 instance

An AWS free-tier account was used to launch an EC2 instance from the dashboard.

I used Ubuntu as the AMI since I'm more familiar with it.

For the instance type, `t3.micro` was chosen as it’s free-tier eligible and cheap to run.

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/AWS%20AMI%20%26%20Instance%20type.png)

A key pair was created to allow SSH access to the instance later.

Under **Network settings**, SSH traffic (restricted to **My IP**), HTTPS, and HTTP were enabled to allow remote access and web traffic.

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/AWS%20Key%20pair%20%26%20Network%20settings.png)

### After launching:
Once the instance was running, I went to **EC2 -> Instances** and noted down the **Public IPv4 address** (e.g., `23.170.8.12`).

## Setting up DNS in Cloudflare
In Cloudflare, the domain was selected and the **DNS Records** section was used to manage records.

An **A record** was then added under **DNS management for adebowale.co.uk**.

This maps the domain to the EC2 instance’s public IPv4 address, allowing users to access the server using a domain name instead of an IP.

The record name (subdomain prefix) was set to **nginx** and pointed to the EC2 Public IPv4 address.

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/DNS%20management.png)

## Accessing the instance via SSH
The SSH connection details were accessed from the **SSH client** tab in the instance's **Connect** section of the AWS console.

Before connecting, the `.pem` file permissions were updated in my local terminal using `chmod 400 <key-pair>.pem` to ensure SSH would accept it.

The instance was then accessed using the SSH command found in the AWS console, run from the directory containing the `.pem` file.

The SSH command looks like: `ssh -i "<key-pair>.pem" ubuntu@ec2-23-170-8-12.eu-west-2.compute.amazonaws.com`

## Installing and starting Nginx

First, the package list was refreshed: `sudo apt update`

Nginx was then installed using `sudo apt install -y nginx` and configured to start automatically whenever the instance reboots using `sudo systemctl enable nginx`.

## Testing everything works

To verify everything was set up correctly, I ran `nslookup <nginx.your.domain>` and `ping <nginx.your.domain>`

* `nslookup` checks DNS and shows what IP the domain points to.
* `ping` checks if the server responds and measures latency.

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/Terminal%20nslookup%20and%20ping%20test.png)

Once DNS was working and the server was reachable, the application was accessed in the browser at: `http://nginx.adebowale.co.uk`

If configured correctly, the default Nginx landing page should appear.

![image alt](https://github.com/zach-adebowale/Nginx-EC2-Deployment/blob/87c5cfac877fc3006770884f1de1134c702a5293/Images/Ngnix%20linked%20to%20domain.png)

## Troubleshooting

1. When accessing **nginx.adebowale.co.uk**, a **Web server is down** error came up in the browser.

This happened because the browser defaulted to HTTPS. Since the server was configured for HTTP at this stage, I had to explicitly use: `http://nginx.adebowale.co.uk`

## Future improvements
* Configure HTTPS 
* Redirect HTTP to HTTPS
* Replace Public IPv4 with an Elastic IP
* Create a simple HTML landing page
