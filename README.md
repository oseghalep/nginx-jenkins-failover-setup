# nginx-jenkins-failover-setup
# NGINX Failover Setup for Jenkins Servers
This repository provides detailed instructions for configuring NGINX as a reverse proxy with failover capabilities for Jenkins servers. The setup ensures that all user traffic is directed to a primary Jenkins server (jenkins_server_1). In case the primary server becomes unreachable, NGINX automatically reroutes the traffic to a backup Jenkins server (jenkins_server_2), ensuring high availability and minimal downtime.

Key Features:
1. Primary-Backup Configuration: Routes traffic to the primary Jenkins server (jenkins_server_1) and automatically fails over to the backup server (jenkins_server_2) if the primary server is down.
2. Seamless User Experience: Users continue accessing Jenkins services without interruption, even if the primary server fails.
3. Centralized Traffic Management: NGINX is installed on a dedicated server, efficiently handling all incoming traffic and routing based on server availability.
4. Scalability: The setup can be extended to manage additional services or Jenkins nodes as your infrastructure grows.
5. SSL/TLS Support: Optionally, configure NGINX to handle SSL termination for secure HTTPS access.
6. This repository is ideal for system administrators and DevOps engineers looking to implement a reliable and scalable Jenkins setup with built-in redundancy using NGINX.

# Recommended Setup
# Third Server with NGINX Installed:

1. NGINX Server: A separate server dedicated to running NGINX as the reverse proxy.
2. Jenkins_server_1: Primary Jenkins server.
3. Jenkins_server_2: Backup Jenkins server.

This setup ensures that traffic management is independent of the Jenkins servers themselves, providing better redundancy, load management, and easier troubleshooting. It also allows you to maintain high availability even if one of the Jenkins servers fails.

