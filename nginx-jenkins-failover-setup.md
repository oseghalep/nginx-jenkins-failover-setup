# Steps to Set Up NGINX for Failover with Jenkins Servers
In this setup, you will install NGINX on a dedicated third server (nginx_server). This server will act as a reverse proxy, directing traffic to your primary Jenkins servers (jenkins_server_1 and jenkins_server_2). If jenkins_server_1 becomes unreachable, NGINX will automatically failover and route traffic to the backup Jenkins server (jenkins_server_2).

# Step 1: Prepare the Third Server (nginx_server)
Provision a New Server:

Set up a new server that will host NGINX. This server should have access to both jenkins_server_1 and jenkins_server_2.
Update the Server:
Before installing any software, update your package lists:
sudo dnf update -y

# Step 2: Install NGINX on the Third Server
Install NGINX:
Install NGINX using the following command:
sudo dnf install nginx -y

Start and Enable NGINX:
Start the NGINX service and enable it to start on boot:
sudo systemctl start nginx
sudo systemctl enable nginx

# Step 3: Configure NGINX for Jenkins Failover
Create or Edit the NGINX Configuration File:
Create a new configuration file for Jenkins in the /etc/nginx/conf.d/ directory:
sudo vi /etc/nginx/conf.d/jenkins_failover.

Add the Failover Configuration:
In the configuration file, add the following configuration, replacing jenkins_server_1 and jenkins_server_2 with the actual IP addresses or hostnames of your Jenkins servers:

upstream jenkins_backend {
    # Primary Jenkins server
    server jenkins_server_1:8080 max_fails=3 fail_timeout=30s;

    # Backup Jenkins server
    server jenkins_server_2:8080 backup;
}

server {
    listen 80;
    server_name jenkins.example.com;

    location / {
        proxy_pass http://jenkins_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Increase buffer size for large Jenkins responses
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
        proxy_redirect off;
    }
}

Explanation:
upstream jenkins_backend: Defines the Jenkins servers. NGINX will try to reach jenkins_server_1 first. If jenkins_server_1 fails three times in 30 seconds, traffic is routed to jenkins_server_2.
backup: Marks jenkins_server_2 as the backup server.
proxy_pass: Directs incoming traffic to the jenkins_backend group.

Save and Exit the Configuration File.

# Step 4: Test and Apply the NGINX Configuration
Test the NGINX Configuration:
Before applying the changes, test the NGINX configuration to ensure there are no syntax errors:
sudo nginx -t
If the test is successful, you’ll see a message indicating that the syntax is okay.

Restart NGINX:
Apply the changes by restarting NGINX:
sudo systemctl restart nginx

# Step 5: Adjust Firewall Rules on NGINX Server
Allow HTTP/HTTPS Traffic:
Ensure that the NGINX server can receive traffic on HTTP (and optionally HTTPS) ports:

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload

Test Connectivity:
Open a browser and navigate to http://jenkins.example.com (or the appropriate domain). Traffic should be routed to jenkins_server_1.
Stop the Jenkins service on jenkins_server_1 to simulate a failure. Confirm that traffic is automatically routed to jenkins_server_2.

# Step 6: Optional SSL/TLS Configuration
If you want to secure Jenkins with HTTPS:
Obtain an SSL Certificate:
Use a certificate from a trusted CA or generate a self-signed certificate.
Update the NGINX Configuration:
Modify the jenkins_failover.conf to include SSL settings:

server {
    listen 443 ssl;
    server_name jenkins.example.com;

    ssl_certificate /etc/nginx/ssl/jenkins.crt;
    ssl_certificate_key /etc/nginx/ssl/jenkins.key;

    location / {
        proxy_pass http://jenkins_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Increase buffer size for large Jenkins responses
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
        proxy_redirect off;
    }
}

Restart NGINX:
Restart NGINX to apply the SSL configuration:
sudo systemctl restart nginx

# Step 7: Verify the Setup
Test Normal Operation:
Ensure that traffic is routed to jenkins_server_1 when it is online.

Test Failover:
Stop the Jenkins service on jenkins_server_1 and verify that traffic is routed to jenkins_server_2.

Restore and Monitor:
Start the Jenkins service on jenkins_server_1 again and verify that traffic reverts back to the primary server.

Conclusion
By following these steps, you will have a robust setup where NGINX on a third server manages traffic for your Jenkins servers, ensuring high availability and seamless failover between jenkins_server_1 and jenkins_server_2.
