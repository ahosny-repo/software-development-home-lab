# Software Development Home Lab
## Platform for Java Development and DevOps

   > ##### Docker, Docker-Compose, Bind9 DNS, Nginx Reverse Proxy, MailCow, Git, Gerrit, OpenLDAP, Jenkins, GitLab, MySQL, SonarQube, JFrog Artifactory.

----------------------
Here - in this guide, I am going to share my knowledge and explain how to setup an integrated software development lab at home using a laptop or a workstation computer and a home internet connection, focusing mainly on the software setup, configuration and integration of the software tools. I will explain the system architecture, procedure and go step-by-step on how to setup and configure the software development lab using a couple of virtual machines representing multi-nodes or servers running docker containers for the open source software tools that we are going to use wherein the main goal is to be able to mimic the entire software development delivery process in an automated fashion.

## Objective: 

- Setup a local software development lab using a laptop or workstation
- Setup infrastructure virtual machines using Oracle VirtualBox and Linux (CentOS)
- Install and configure containerized open source software tools for the integrated software development lab
- Setup environment and workflow process for team collaboration and source code review mechanism using Gerrit
- Create Jenkins pipelines for continuous integration and delivery for simple Java application (DevOps Automation Plateform)

----------------------

## Series:

1. Part-1: Setup virtual machines, linux and infrastructure software
2. Part-2: Install, configure and integrate software development tools
3. Part-3: Run and verify the entire software development cycle in our new lab ...

## Software Stack:
Open source software stack:
- **_Git:_** Distributed version control system
- **_Gerrit:_** Code review and team collaboration tool
- **_Jenkins:_** Automation server for code build, test and deloyment
- **_OpenLDAP:_** Open source implementation of LDAP (Lightweight Directory Access Protocol)
- **_SonarQube:_** Code quality and continous inspection for software development automation
- **_JFrog Artifactory:_** Repository and binaries manager for software artifacts
- **_GitLab (Gerrit Replication):_** GitLab works as an automation server but will be used her for repository replication purpose only.
- **_Infrastructure Software:_**
  - BIND9 (DNS Server)
  - MailCow (Mail Server)
  - Docker & Docker-Compose
  - NGINX (Reverse Proxy Server)
  - Portainer (Container Management)
  - Webmin (Linux Administration Tool)
  
## System Architecture:

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/000-system-architecture-overview.jpg" width="800">
</p>

## Software Development Process:

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/001-software-development-process.jpg" width="800">
</p>

## Exposed Lab Services:

| Service      | Service Endpoint                | IP Address   | Port  |
| --------     | ------------------------------  | ------------ | ----- |
| DNS Webmin   | http://dns.swlab.local          | 192.168.1.36 | 10000 |
| Portainer    | http://portainer.swlab.local    | 192.168.1.28 | 9000  |
| Mailcow      | http://mail.swlab.local         | 192.168.1.20 | 80    |
| Webmail      | http://mail.swlab.local/SOGo    | 192.168.1.20 | 80    |
| Gerrit       | http://gerrit.swlab.local       | 192.168.1.24 | 8080  |
| LDAP Web     | https://ldapweb.swlab.local/    | 192.168.1.24 | 8082  |
| Jenkins      | http://jenkins.swlab.local      | 192.168.1.26 | 8080  |
| SonarQube    | http://sonarqube.swlab.local    | 192.168.1.26 | 9000  |
| Artifactory  | http://artifactory.swlab.local  | 192.168.1.26 | 8082  |
| GitLab       | http://gitlab.swlab.local       | 192.168.1.26 | 8084  |

## Part-1: Setup virtual machines, linux and infrastructure software

### **1. Setup Environment and Tools**

   1. Download and install VirtualBox -> https://www.virtualbox.org/  
   2. Download Linux distribution (CentOS) -> https://www.centos.org/
   3. Create virtual machines / servers with the below configurations:

      | VM Name                 | IP Address   | Memory | Description                                                                              |
      | ----------------------- | ------------ | ------ | ---------------------------------------------------------------------------------------- |
      | dns.swlab.local-srv     | 192.168.1.36 |  2Gb   | DNS server                                                                               |
      | proxy.swlab.local-srv   | 192.168.1.32 |  2Gb   | Proxy server                                                                             |
      | mail.swlab.local-srv    | 192.168.1.20 |  2Gb   | Mail server                                                                              |
      | dbms.swlab.local-srv    | 192.168.1.22 |  2Gb   | Database server for software development                                                 |
      | scms.swlab.local-srv    | 192.168.1.24 |  6Gb   | Source code management systeem (Gerrit + OpenLDAP)                                       |
      | cicd.swlab.local-srv    | 192.168.1.26 |  6Gb   | CICD automation server (Jenkins, Sonarqube, Artifactory and GitLab)                      |
      | util.swlab.local-srv    | 192.168.1.28 |  2Gb   | Utility server (Portainer for container management and monitoring)                       |
    
       - Processor: 1 or 2
       - Network: Bridged Adaptor
       - Storage: Dynamically Allocated

   4. Set static IPs for the virtual machine
      1. **_Method#1:_** Modifying the interface configuration file manually.

         > For each network interface created and managed by NetworkManager, a configuration file is created inside /etc/sysconfig/network-scripts directory, the name of the file should start with ifcfg- then the network interface name for example 'enp0s3'. Login as a root user and modify the interface configuration file as following by setting your IP Address, Prefix, Gateway and DNS configurations:
         >> $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
         >> 
         >> - BOOTPROTO=none_**
         >> - IPADDR=192.168.1.X_**
         >> - PREFIX=24_**
         >> - GATEWAY=192.168.1.1_**
         >> - DNS1=192.168.1.36_** // Local DNS Server IP
         >> - DNS2=X.X.X.X_** // Home network DNS Server IP
      2. **_Method#2:_** Using NetworkManager user interface to edit the connection as following:
         1. Login as a root user
         2. Run command "$ nmtui"
         3. NetworkManager TUI -> Edit a connection
         4. Ethernet -> Select Interface i.e. enp0s3 -> Then Edit
         5. Set IP Address, Gateway and DNS servers as below screenshot
         6. Save changes and restart NetworkManager "$ systemctl restart NetworkManager"
         <p align="center">
          <img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/002-networkmanager-edit-connection.jpg">
         </p>
     
> **_NOTE: About CentOS_**  
> I am using CentOS (Linux) version 8.x from osboxes.org that offers Linux/Unix virtual machines (VDIs) for Oracle VirtualBox with ready to use virtual images that you can download and attach to a virtual machine however; you can refer to official Linux (CentOS) distribution website that you can download in different format and setup as per your convinent.
> 
> Please refer to https://www.centos.org/centos-stream/ for converting from CentOS Linux 8 to CentOS Stream 8 since CentOS Linux 8 reached EOL in 31st December, 2021 and replaced by the new CentOS Stream.
    
### **2. Setup DNS Server (BIND9)**

1. Install DNS

        1. Login as a root user
        $ ssh root@192.168.1.36
   
        2. Set the hostname for the DNS server
        $ hostnamectl set-hostname dns.swlab.local
   
        3. Update OS software packages
        $ yum update -y

        4. Install bind and configure named service
        $ yum install bind bind-utils -y
        $ systemctl start named
        $ systemctl enable named
        $ systemctl status named

2. Configure DNS manually

        1. Configure name service

        # edit named.conf and do the following configuration
        # Change or Add lines between dotted-lines below ---- in the named.conf file
        $ vi /etc/named.conf

        # Configure BIND to listen to server IP (dns.swlab.local:192.168.1.36); by default BIND listens to (localhost:127.0.0.1) only
        # Configure BIND to allow to all IP addresses or hosts in the network to query DNS server for IP to name translation
        -------------------------------------------------------
               listen-on port 53 { 127.0.0.1;192.168.1.36; };
               allow-query     { localhost;192.168.1.1/24; };
        -------------------------------------------------------
        # Create Forward zone "swlab.local"
        -------------------------------------------------------
         zone "swlab.local" IN {                       // Domain name
               type master;                            // Primary DNS
               file "/var/named/forward.swlab.local";  // Forward lookup file
               allow-update { none; };                 // allow-update – Since this is the primary DNS, it should be none
         };
        -------------------------------------------------------
        # Create Reverse Zone "1.168.192.in-addr.arpa"
        -------------------------------------------------------
         zone "1.168.192.in-addr.arpa" IN {            // 1.168.192.in-addr.arpa – Reverse lookup name
               type master;                            // master – Primary DNS
               file "/var/named/reverse.swlab.local";  // Reverse lookup file
               allow-update { none; };                 // Since this is the primary DNS, it should be none
         };
        -------------------------------------------------------

        2. Create file Foward "forward.swlab.local"
        $ vi /var/named/forward.swlab.local
        ​-------------------------------------------------------
        @       IN      SOA     dns.swlab.local dns.swlab.local. (
                        1033
                        3H
                        15M
                        1W
                        1D )
         ;Name Server Information
         @                        IN      NS      dns.swlab.local.    ; NS : Name Server
         dns.swlab.local.         IN      A       192.168.1.36        ; Pointer 192.168.1.36 to dns.swlab.local
         scms.swlab.local.        IN      A       192.168.1.24        ; A - A Record ( SCMS Server )
         cidcd.swlab.local.       IN      A       192.168.1.26        ; A - A Record ( CICD Server )
         dbms.swlab.local.        IN      A       192.168.1.22     
         mail.swlab.local.        IN      A       192.168.1.20     
         swlab.local.             IN      MX      1 mail.swlab.local. ; MX - Mail for Exchange
         ftp                      IN      CNAME   www.swlab.local.    ; CNAME – Canonical Name
         gerrit.swlab.local.      IN      A       192.168.1.24
         ldapweb.swlab.local.     IN      A       192.168.1.24
         ldap.swlab.local.        IN      A       192.168.1.24
         jenkins.swlab.local.     IN      A       192.168.1.26
         artifactory.swlab.local. IN      A       192.168.1.26
         sonarqube.swlab.local.   IN      A       192.168.1.26
         gitlab.swlab.local.      IN      A       192.168.1.26
         proxy.swlab.local.       IN      A       192.168.1.32
         util.swlab.local.        IN      A       192.168.1.28
        -------------------------------------------------------

        3. Create file Reveser "reverse.swlab.local"
        $ vi /var/named/reverse.swlab.local
        -------------------------------------------------------
        @       IN      SOA     dns.swlab.local. root.swlab.local. (
                        1029
                        3H
                        15M
                        1W
                        1D )
         ;Name Server Information
         1.168.192.in-addr.arpa.         IN      NS      dns.swlab.local.
         ;PTR Record IP address to Hostname
         36.1.168.192.in-addr.arpa.      IN      PTR     dns.swlab.local.       
         24.1.168.192.in-addr.arpa.      IN      PTR     scms.swlab.local.      
         26.1.168.192.in-addr.arpa.      IN      PTR     cicd.swlab.local.       
         22.1.168.192.in-addr.arpa.      IN      PTR     dbms.swlab.local.
         20.1.168.192.in-addr.arpa.      IN      PTR     mail.swlab.local.
         24.1.168.192.in-addr.arpa.      IN      PTR     gerrit.swlab.local.
         24.1.168.192.in-addr.arpa.      IN      PTR     ldapweb.swlab.local.
         24.1.168.192.in-addr.arpa.      IN      PTR     ldap.swlab.local.
         26.1.168.192.in-addr.arpa.      IN      PTR     jenkins.swlab.local.
         26.1.168.192.in-addr.arpa.      IN      PTR     artifactory.swlab.local.
         26.1.168.192.in-addr.arpa.      IN      PTR     sonarqube.swlab.local.
         26.1.168.192.in-addr.arpa.      IN      PTR     gitlab.swlab.local.
         32.1.168.192.in-addr.arpa.      IN      PTR     proxy.swlab.local.
         28.1.168.192.in-addr.arpa.      IN      PTR     util.swlab.local.
        -------------------------------------------------------

3. Configure DNS using Webmin tool

        # Download webmin rpm
        1. wget http://prdownloads.sourceforge.net/webadmin/webmin-1.984-1.noarch.rpm
   
        2. Install optional dependencies
        $ yum -y install perl perl-Net-SSLeay openssl perl-Encode-Detect
   
        3. Install webmin rpm
        $ rpm -U webmin-1.984-1.noarch.rpm

        4. Open firewall for webmin service port 10000
        $ firewall-cmd --add-port=10000/tcp --zone=public --permanent
        $ firewall-cmd --add-port=53/tcp --zone=public --permanent
        $ firewall-cmd --add-port=53/udp --zone=public --permanent
        $ firewall-cmd --reload

        5. Edit webmin configuration and disable ssl for https and set entry to "0" so it will be "ssl=0"
        $ vi /etc/webmin/miniserv.conf 

        6. Start and enable webmin service 
        $ systemctl start webmin
        $ systemctl enable webmin
        $ systemctl status webmin

        7. Test webmin tool using root user login -> http://dns.swlab.local:10000/
        8. Configure BIND DNS Server from Servers menu

4. DNS domain enteries in Webmin tool
- Forward "forward.swlab.local" domain entries

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/003-dns-swlab.local-forward-webmin.jpg" width="800">
</p>

- Reveser "reverse.swlab.local" domain entries
<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/004-dns-swlab.local-reverser-webmin.jpg" width="800">
</p>
> **_NOTE: About BIND9 DNS_**  
> - Webmin is a web control panel that allows you to administer linux servers like DNS, SSH ...etc. through a simple interface so instead of creating and configuring dns domain names manually you can simply using Webmin to create master/forward/reverser zone and configure name entries.
> - You will need also to edit "hosts" file in Windows (host operating system) located under the path "C:\Windows\System32\drivers\etc" to add the name entries to it in order to be able to access the lab services from Windows (Host OS).

### **3. Setup Infrastructure Software**
This step needs to be done for the following environments below:

- Mail Server (192.168.1.20:mail.swlab.local)
- Database Management Server (192.168.1.22:dbms.swlab.local)
- Souce Code Management Server (192.168.1.24:scms.swlab.local)
- CI/CD Automation Server & Tools (192.168.1.26:cicd.swlab.local)
- Utilities and Containers Monitoring Server (192.168.1.28:util.swlab.local)

        1. Login as root@SERVER_IP
        $ ssh root@192.168.1.20

        2. Set the hostname for each VM server
        $ hostnamectl set-hostname i.e. scms.swlab.local
        # check hostname updated successfully
        $ hostnamectl status
    
        1. Update software packages
        $ yum update -y
    
        1. Create docker user
        # create docker user 'docker'
        $ useradd docker

        # set user password
        $ passwd docker
        
        # add docker user to sudoers
        $ usermod -aG wheel docker
         
        # switch to docker user to install docker
        $ su docker

        1. Install docker
        # remove following docker packages / dependencies; if exists
        $ sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
       
        # install yum-utils to be able to add docker repo
        $ sudo yum install -y yum-utils

        $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
       
        # install docker engine and related packages
        $ sudo yum install docker-ce docker-ce-cli containerd.io
       
        # install and add docker to docker group so you can run docker command with no need for sudo <<you will need to logout and login again or restart the virtual machine so that your group membership is re-evaluated>>
        $ sudo groupadd docker
        $ sudo usermod -aG docker docker

        # start and enable docker services   
        $ sudo systemctl start docker
        $ sudo systemctl enable docker.service
        $ sudo systemctl enable containerd.service

        # test docker installation
        $ docker run hello-world

        1. Install docker-compose

        # download the current stable release of docker compose
        $ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

        # give executable permissions to the binary 
        $ sudo chmod +x /usr/local/bin/docker-compose

        # check installed docker-compose version
        $ docker-compose --version

        # if docker-compose command does not work after installation, then you may need to create a symbolic link to /usr/bin
        $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    
> **_NOTE: About Docker_**  
> - You may need to remove existing container packages that pre-installed with CentOS distribution we are using that are conflicting with docker packages like podman, buildah ...etc. , run "sudo yum remove podman" and "sudo yum remove buildah" to be able to install docker binaries successfully.
> - Another option is to add "--allowerasing" to docker install command that will check the conflicting packages and recommend actions accordingly "$ sudo yum install --allowerasing docker-ce docker-ce-cli containerd.io".

### **4. Setup Mail Server (Mailcow)**

1. Login as root@(192.168.1.20:mail.swlab.local) to docker host virtual machine
2. Clone mailcow docker-compose file from github "https://github.com/mailcow/mailcow-dockerized"

        # install git first
        $ yum install -y git

        # navigate to /opt directory to clone the repo as per the mailcow guide
        $ cd /opt
        
        # Clone mailcow repo from github to /opt directory
        $ clone https://github.com/mailcow/mailcow-dockerized.git

        # cd to cloned mailcow-dockerized directory
        $ cd mailcow-dockerized

        # generate mailcow configurations
        # set hostname to "mail.swlab.local" and timezone when prompt
        $ ./generate_config.sh

3. Edit mailcow configurations

        # skip SSL certificate validation by skiping lets encrypt option in the mailcow.conf file "no https" 
        # Edit mailcow.conf file and set the skip lets encrypt entry to yes -> "SKIP_LETS_ENCRYPT=y"
        $ vi /opt/mailcow-dockerized/mailcow.conf

        # change posftfix configurations to make jenkins access mailcow on tcp port 25
        # change following entries under the "# inter-mx with postscreen on 25/tcp" section in the master.cf file by:
        1- updating existing entry and set to blank: -> "-o smtpd_helo_restrictions="
        2- adding new entry by inserting a new line after the above entry: -> "-o smtpd_helo_required=no"
      
        $ cd /opt/mailcow-dockerized/data/conf/postfix
        $ vi master.cf

4. Start mailcow server using docker-compose

        # switch to docker user
        $ su docker

        $ cd /opt/mailcow-dockerized

        # start mailcow docker-compose
        $ sudo docker-compose up -d

5. Login to mailcow "http://mail.swlab.local/" using admin user

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/005-mailcow-login-page.JPG" width="800">
</p>

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/006-mailcow-home-page.JPG" width="800">
</p>
6. Create "swlab.local" domain and required mailboxes "emails"
   - From "Configurations" menu on the top from the home page, click "Mail Setup"
   - Then click "+ Add domain" button to create new domain "swlab.local" for our lab
   - Enter "swlab.local" in the "Domain" field and leave other fields with the default values
   - Finally, click button "Add domain and restart SOGo" and wait for the domain to be created
   - Now, our domain is added successfully
<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/007-mailcow-add-domain-pag.JPG" width="800">
</p>
7. Create mailboxes "emails" for our software lab
   - From "Configurations" menu on the top from the home page, click "Mail Setup"
   - Then select "Mailboxes" tab and start adding mailboxes by clicking "+ Add mailbox"
   - Add "gerrit" and "jenkins" mailboxes to our domain "swlab.local"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/008-mailcow-mailbox-page.JPG" width="800">
</p>

8. Login to webmail "http://mail.swlab.local/SOGo/"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/009-mailcow-sogo-login-page.JPG" width="800">
</p>

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/010-mailcow-sogo-gerrit-page.JPG" width="800">
</p>

> **_NOTE: About Mailcow_**  
> - Login using "admin" as a user name and "moohoo" the default admin password; you can check also the documentation for default admin password on https://github.com/mailcow/mailcow-dockerized
> - You may be asked to change default admin password after login for the first time to mailcow server

### **5. Setup Reverse Proxy Server (NGINX)**

1. Install NGINX Server

        1. Login as root user
        $ ssh root@192.168.1.32

        2. Set the hostname for the server
        $ hostnamectl set-hostname proxy.swlab.local
        # check hostname updated successfully
        $ hostnamectl status
    
        3. Update software packages
        $ yum update -y
    
        1. Create nginx user
        $ useradd nginx

        # set user password
        $ passwd nginx
        
        # add nginx user to sudoers
        $ usermod -aG wheel nginx
         
        # switch to nginx user to install nginx server
        $ su nginx
        
        5. Install nginx
        $ sudo yum install nginx
        
        # Open http and https firewall ports
        $ sudo firewall-cmd --permanent --zone=public --add-service=https --add-service=http
        $ sudo firewall-cmd --reload

        # Enable and start nginx service
        $ sudo systemctl enable nginx
        $ sudo systemctl start nginx

        # Check server status should be active and running
        $ sudo systemctl status nginx

1. Configure NGINX Server
      1. NGINX Reverse Proxy Configurations

        1. Create a new configuration file
        $ vi /etc/nginx/conf.d/default.conf

        1. Add custom log format in the first line of the file
         #-------------------LOG FORMAT-----------------------
         log_format custom_log '"Request: $request\n Status: $status\n Request_URI: $request_uri\n Host: $host\n Client_IP: $remote_addr\n Proxy_IP(s): $proxy_add_x_forwarded_for\n Proxy_Hostname: $proxy_host\n Real_IP: $http_x_real_ip\n User_Client: $http_user_agent"';

        2. Add following configurations to the file
         #-------------------WEBMIN--------------------------
         server {
         listen       80;
         server_name  dns.swlab.local;
            
            access_log /var/log/nginx/dns-access-logs.log custom_log;
            error_log /var/log/nginx/dns-error-logs.log;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  
         location / {
            proxy_pass http://192.168.1.36:10000;
            }
         }
         #-------------------PORTAINER-----------------------
         server {
         listen       80;
         server_name  portainer.swlab.local;

            access_log /var/log/nginx/portainer-access-logs.log custom_log;
            error_log /var/log/nginx/portainer-error-logs.log;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

         location / {
            proxy_pass http://192.168.1.28:9000;
            }
         }
         #---------------------MAILCOW----------------------- 
         server {
         listen       80;
         server_name  mail.swlab.local;

            access_log /var/log/nginx/mail-access-logs.log custom_log;
            error_log /var/log/nginx/mail-error-logs.log;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

         location / {
            proxy_pass http://192.168.1.20:80;
            }
         }
         #---------------------LDAPWEB----------------------- 
         server {
         listen       80;
         server_name  ldapweb.swlab.local;

            access_log /var/log/nginx/ldapweb-access-logs.log custom_log;
            error_log /var/log/nginx/ldapweb-error-logs.log;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            location / {
            proxy_pass http://192.168.1.24:8082;
            }
         }

      2. NGINX Reverse Proxy for Jenkins
         - Please refere to https://www.jenkins.io/doc/book/system-administration/reverse-proxy-configuration-nginx/ 
      3. NGINX Reverse Proxy for Gerrit
         - Please refere to https://gerrit-review.googlesource.com/Documentation/config-reverseproxy.html 
         - However; following configurations will use in our lab:
  
               1. Create a new configuration file for gerrit
               $ vi /etc/nginx/conf.d/gerrit.conf

               2. Add following configurations to the file
               #---------------------GERRIT-----------------------            
               server {
               listen 80;
               server_name gerrit.swlab.local;

                  access_log      /var/log/nginx/gerrit.access.log;
                  error_log       /var/log/nginx/gerrit.error.log;

                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                  location / {
                  proxy_pass        http://192.168.1.24:8080;
                  }
               }
      4. NGINX Reverse Proxy for JFrog Artifactory
         - Please refere to https://www.jfrog.com/confluence/display/JFROG/HTTP+Settings#:~:text=To%20configure%20a%20reverse%20proxy,Using%20NGINX%3F
         - However; following configurations will use in our lab:
  
               1. Create a new configuration file for artifactory
               $ vi /etc/nginx/conf.d/artifactory.conf

               2. Add following configurations to the file
               #------------------ARTIFACTORY-------------------            
               server {
               listen 80;
               server_name artifactory.swlab.local;

                  access_log      /var/log/nginx/artifactory.access.log;
                  error_log       /var/log/nginx/artifactory.error.log;

                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                  location / {
                  proxy_pass        http://192.168.1.26:8082;
                  }
               }
      5. NGINX Reverse Proxy for SonarQube
         - Please refere to https://docs.sonarqube.org/7.2/SecuringtheServerBehindaProxy.html#src-11639522_SecuringtheServerBehindaProxy-UsingNginx
         - However; following configurations will use in our lab:
  
               1. Create a new configuration file for sonarqube
               $ vi /etc/nginx/conf.d/sonarqube.conf

               2. Add following configurations to the file
               #------------------SONARQUBE-------------------            
               server {
               listen 80;
               server_name sonarqube.swlab.local;

                  access_log      /var/log/nginx/sonarqube.access.log;
                  error_log       /var/log/nginx/sonarqube.error.log;

                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                  location / {
                  proxy_pass        http://192.168.1.26:9000;
                  }
               }
      6. NGINX Reverse Proxy for GitLab
         - Please refere to https://gitlab.com/gitlab-org/gitlab-recipes/blob/master/web-server/nginx/gitlab-omnibus-nginx.conf

> **_NOTE: About NGINX_**  
> - We are using HTTP for the services and software tools used in this lab for simplicity however; in real production environment you may need to secure your services access using HTTPS/SSL connection that would require openning HTTPS port 443 and installing SSL certificate for NGINX service proxy configurations over HTTPS.

### **6. Setup Containers Management and Monitoring (Portainer)**
1. Login as root@(192.168.1.32:util.swlab.local) to docker host virtual machine
2. Copy docker-compose contents below to docker-compose.yaml file

         version: '3'
         services:
         portainer-svc:
            image: portainer/portainer-ce:latest
            hostname: portainer
            container_name: portainer
            networks:
               default:
                  aliases:
                  - portainer
                  - portainer.swlab
                  - portainer.swlab.local
            ports:
               - 9000:9000
               - 8000:8000
            restart: unless-stopped
            volumes:
               - /etc/localtime:/etc/localtime:ro
               - /var/run/docker.sock:/var/run/docker.sock:ro
               - /data/portainer-data:/data portainer/portainer-ce

3. Start portainer by running "docker-compose up -d"
4. Test and verify Portainer installation
- Launch jenkins "http://portainer.swlab.local/"
- Create user and login to portainer 

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/portainer-login-page.JPG" width="800">
</p>

- Add environment to portainer
  - Click on Environments menu
  - Select "Agent Portianer Agent" and "Docker Swarm"
  - Copy Docker Swarm command "curl -L https://downloads.portainer.io/agent-stack-ce211.yml -o agent-stack.yml && docker stack deploy --compose-file=agent-stack.yml portainer-agent" and run the command to install the portainer agent container in the target environment to be monitored and managed by portainer i.e. "mail.swlab.local".
  - Enter environment details below
    - Name
    - Environment URL
    - Public IP
  - And "Add Environment"
  - Repeat the above steps for each environment to add corresponding environment

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/portainer-add-environment-page.JPG" width="800">
</p>

- Portainer Home after adding all environment

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/portainer-home-page.JPG" width="800">
</p>

- Show environment containers

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/portainer-show-environment-page.JPG" width="800">
</p>

- Show containers list

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/portainer-environment-containers-list-page.JPG" width="800">
</p>

## Part-2: Install, configure and integrate software development tools

### **1. Setup Gerrit with MySQL and OpenLDAP using Docker-Compose**

1. Login as docker@(192.168.1.24:scms.swlab.local) to docker host virtual machine
2. Create the required directories for the docker containers in the docker host virtual machine

        # create a base directory for persisting containers data
        $ sudo mkdir /data/

        # create directory(s) for docker-compose.yaml file
        $ sudo mkdir -p /data/docker/scms.swlab.local

        # create docker-compose.yaml file
        $ sudo touch /data/docker/scms.swlab.local/docker-compose.yaml

3. Copy docker-compose contents below to docker-compose.yaml file

        version: "3"
        services:
          gerrit-db-svc:
            image: "mysql:5.7"
            hostname: gerritdb
            container_name: gerritdb-container
            networks:
               default:
                  aliases:
                     - gerritdb.swlab.local
            ports:
               - "3306:3306"
            restart: unless-stopped
            volumes:
               - /data/mysql/db:/var/lib/mysql
            environment:
               - MYSQL_ROOT_PASSWORD=dbrootpassword
          ldap-svc:
            image: osixia/openldap
            hostname: ldap
            container_name: ldap-container
            networks:
               default:
                  aliases:
                     - ldap.swlab.local
            ports:
               - "389:389"
               - "636:636"
            restart: unless-stopped
            environment:
               - LDAP_ADMIN_PASSWORD=ldappassword
            volumes:
               - /data/ldap/var:/var/lib/ldap
               - /data/ldap/etc:/etc/ldap/slapd.d
          ldap-web-svc:
            image: osixia/phpldapadmin
            hostname: ldapweb
            container_name: ldapweb-container
            networks:
               default:
                  aliases:
                     - ldapweb.swlab.local
            ports:
               - "8082:80"
            restart: unless-stopped
            environment:
               - PHPLDAPADMIN_LDAP_HOSTS=ldap
               - PHPLDAPADMIN_HTTPS=false
          gerrit-svc:
            depends_on:
               - ldap-svc
               - gerrit-db-svc
            image: gerritcodereview/gerrit
            hostname: gerrit
            container_name: gerrit-container
            networks:
               default:
                  aliases:
                     - gerrit.swlab.local
            links:
               - gerrit-db-svc
            ports:
               - "29418:29418"
               - "8080:8080"
            restart: unless-stopped
            user: root
            volumes:
               - /data/gerrit/etc:/var/gerrit/etc
               - /data/gerrit/git:/var/gerrit/git
               - /data/gerrit/index:/var/gerrit/index
               - /data/gerrit/cache:/var/gerrit/cache
            environment:
               - DATABASE_TYPE=mysql
               - DATABASE_HOSTNAME=gerritdb
               - DATABASE_PORT=3306
               - DATABASE_DATABASE=gerrit
               - DATABASE_USERNAME=gerrit
               - DATABASE_PASSWORD=gerrit
               - CANONICAL_WEB_URL=http://gerrit.swlab.local

4. Setup Gerrit Database (MySQL)

         # start mysql database service first
         $ docker-compose up -d gerrit-db-svc

         # login to gerritdb-container by container name
         # 'docker container ls' will show all running containers
         $ docker exec -it gerritdb-container /bin/bash

         # Enter mysql console with root user
         $ mysql -u root -p

         # create gerrit database user
         $ mysql> CREATE database gerrit DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

         # grante all privilege to gerrit database user
         $ mysql> grant all privileges on gerrit.* to gerrit@'%' identified by 'gerrit';

         # flush privileges
         $ mysql> flush privileges;

         # adjust mysql timestamp
         $ mysql> set global explicit_defaults_for_timestamp=1;

         # list databases
         $ mysql> show databases;

         # exist mysql and the container bash
         $ mysql> exit

5. Start Gerrit       

         # start remaining docker-compose services in daemon mode
         $ docker-compose up -d

         # check docker containers are up and running
         $ docker container ls

         # check gerrit container logs
         $ docker logs -f gerrit-container
         
         # Logs should indicate gerrit started successfully 
         -> "Gerrit Code Review 3.5.0.1 ready"

         # now you should have gerrit directories and files mounted to docker host virtual machine 
         # directories should be located under "/data/gerrit/..." as defined in the docker-compose file

6. Configure Gerrit

         # edit gerrit.config file
         $ sudo vi /data/gerrit/etc/gerrit.config

         # we will add/update following sections:
         1- update [auth] to make gerrit use ldap for authontication
         2- update [httpd] to listen on traffic on http://*:8080
         2- add [ldap] ldap server configuration details
         3- add [database] to make gerrit use mysql database
         4- add [email] configuration section
         
         ################################################################    
         [auth]
               type = ldap
               gitBasicAuth = true
         [ldap]
               server = ldap://ldap
               username=cn=admin,dc=example,dc=org
               accountBase = dc=example,dc=org
               accountPattern = (&(objectClass=person)(uid=${username}))
               accountFullName = displayName
               accountEmailAddress = mail       
         [database]
               type = mysql
               port = 3306
               hostname = gerritdb
               database = gerrit
               username = gerrit
         [httpd]
               listenUrl = http://*:8080/
         ################################################################
 
         # edit secure.config file to add ldap and mysql user password
         $ sudo vi /data/gerrit/etc/secure.config

         # add following lines to the file
         ################################################################
         [ldap]
               password = ldappassword
         [database]
	            password = gerrit
         ################################################################
         
         # stop and start services
         $ docker-compose down
         $ docker-compose up -d

7. Test Gerrit
- Launch gerrit "http://gerrit.swlab.local/"
   
<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/011-gerrit-home-page.JPG" width="800">
</p>
- Click "Sign in" on the right side of the gerrit home page to login to gerrit
- You will be redirected to "Sign in" page that is actually using ldap for user's authentication
- If you can see "Sign in" page below, then gerrit is configured successfully to use ldap as authentication provider.

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/012-gerrit-signin-page.JPG" width="800">
</p>
8. Configure Gerrit Users in Ldap
- Launch Ldap Web Admin "http://ldapweb.swlab.local/"
- Click "login" and enter "cn=admin,dc=example,dc=org" for the "Login DN" and then "ldappassword" password as set in docker-compose file for admin user

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/013-ldap-login-page.JPG" width="800">
</p>
- Click the domain from left side tree and then create child entry 

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/014-ldap-domain-page.JPG" width="800">
</p>
- Select "Courier Mail: Account" template to create user and enter user details

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/015-ldap-createuser-page.JPG" width="800">
</p>
- Commit to save new created user

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/016-ldap-commit-user-page.JPG" width="800">
</p>

- Now, let's try login to gerrit "http://gerrit.swlab.local/" using newly created user "ahosny"

<p align="center">
<img src="images\new\017-gerrit-signin-ahosny-page.JPG">
</p>

- After successfull login using "ahosny" user, click on the user avatar on the right side of the page and select Settings to see user's details

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/018-gerrit-login-ahosny-page.JPG" width="800">
</p>
- User settings page in gerrit showing user's details from ldap

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/019-gerrit-ahosny-settings-page.JPG" width="800">
</p>
- Check and test clone url for "ALL-Projects" repository: "$git clone "http://ahosny@gerrit.swlab.local/a/All-Projects"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/020-gerrit-clone-all-projects-page.JPG" width="800">
</p>

- Now, all gerrit is set, Congratulations!

### **2. Setup Jenkins, GitLab, SonarQube, Artifactory Docker Containers**

1. Login as docker@(192.168.1.24:cicd.swlab.local) to docker host virtual machine
2. Create the required directories for the docker containers in the docker host virtual machine

        # create a base directory for persisting containers data
        $ sudo mkdir /data/

        # create directory(s) for docker-compose.yaml file
        $ sudo mkdir -p /data/docker/cicd.swlab.local

        # create docker-compose.yaml file
        $ sudo touch /data/docker/cicd.swlab.local/docker-compose.yaml

3. Copy docker-compose contents below to docker-compose.yaml file for installing Jenkins and SonarQube

        version: '3'
         services:
         jenkins-svc:
            image: 'jenkins/jenkins:latest'
            hostname: jenkins
            container_name: jenkins-container
            user: root
            networks:
               default:
                  aliases:
                  - jenkins
                  - jenkins.swlab
                  - jenkins.swlab.local
            ports:
               - '8080:8080'
               - '50000:50000'
            restart: unless-stopped
            volumes:
               - /data/jenkins_home:/var/jenkins_home
            environment:
            - JAVA_OPTS=-Dhudson.remoting.ClassFilter=com.google.gerrit.extensions.common.AvatarInfo,com.google.gerrit.extensions.common.ReviewerUpdateInfo
         sonarqube-svc:
            image: sonarqube:community
            hostname: sonarqube
            container_name: sonarqube-container
            networks:
               default:
                  aliases:
                  - sonarqube
                  - sonarqube.swlab
                  - sonarqube.swlab.local
            depends_on:
               - sonarqube-db-svc
            ports:
               - '9000:9000'
            restart: unless-stopped
            environment:
               SONAR_JDBC_URL: jdbc:postgresql://sonarqubedb:5432/sonar
               SONAR_JDBC_USERNAME: sonar
               SONAR_JDBC_PASSWORD: sonar
            volumes:
               - /data/sonarqube/data:/opt/sonarqube/data
               - /data/sonarqube/extensions:/opt/sonarqube/extensions
               - /data/sonarqube/logs:/opt/sonarqube/logs
         sonarqube-db-svc:
            image: postgres:12
            hostname: sonarqubedb
            container_name: sonarqubedb-container
            networks:
               default:
                  aliases:
                  - sonarqubedb
                  - sonarqubedb.swlab
                  - sonarqubedb.swlab.local
            restart: unless-stopped
            environment:
               POSTGRES_USER: sonar
               POSTGRES_PASSWORD: sonar
            volumes:
               - /data/sonarqube/postgresql:/var/lib/postgresql
               - /data/sonarqube/postgresql_data:/var/lib/postgresql/data

3. Install JFrog Artifactory using docker

        # using root user
        # set the JFrog Home environment variable
        $ export JFROG_HOME=/data/jfrog_home

        # create JFrog directories
        $ mkdir -p $JFROG_HOME/artifactory/var/etc/
        $ cd $JFROG_HOME/artifactory/var/etc/
        $ touch ./system.yaml
        $ chown -R 1030:1030 $JFROG_HOME/artifactory/var
        $ chmod -R 777 $JFROG_HOME/artifactory/var         
        # create jfrog artifactory docker container using docker user
        $ docker run --name artifactory -v $JFROG_HOME/artifactory/var/:/var/opt/jfrog/artifactory -d --restart unless-stopped -p 8081:8081 -p 8082:8082 releases-docker.jfrog.io/jfrog/artifactory-oss:latest

3. Test and verify Jenkins installation
   
- Launch jenkins "http://jenkins.swlab.local/"
- You will be asked to unlock jenkins by providing the initial admin password
- Run following command to get the password from jenkins container
  - "docker exec -it jenkins-container cat /var/jenkins_home/secrets/initialAdminPassword"
   
<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/021-jenkins-unlock-password-page.JPG" width="800">
</p>

- Customize jenkins configuration
- Select "Install suggested plugins"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/022-jenkins-customize-jenkins-page.JPG" width="800">
</p>

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/023-jenkins-suggested-plugin-install-page.JPG" width="800">
</p>
- Create admin user

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/024-jenkins-create-admin-user-page.JPG" width="800">
</p>
- Configure jenkins url

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/025-jenkins-instance-config-url-page.JPG" width="800">
</p>
- Jenkins home page

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/026-jenkins-home-page.JPG" width="800">
</p>
- After installations you may be asked to upgrade jenkins to latest version

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/027-jenkins-home-latest-page.JPG" width="800">
</p>
6. Test and verify SonarQube installation

- Launch jenkins "http://sonarqube.swlab.local/"
- Enter default user name and password "admin/admin"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/028-sonarqube-login-page.JPG" width="800">
</p>
- Update default password and save
- SonarQube home page

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/029-sonarqube-home-page.JPG" width="800">
</p>

7. Test and verify JFrog Artifactory installation
- Launch jenkins "http://artifactory.swlab.local/"
- You will be asked to update default password "admin/password"

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/030-artifactory-login-page.JPG" width="800">
</p>
- JFrog Artifactory welcome page

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/031-artifactory-welcome-page.JPG" width="800">
</p>
- JFrog Artifactory Base URL
  
<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/032-artifactory-set-url-page.JPG" width="800">
</p>
- Skip default proxy configuration

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/033-artifactory-default-proxy-skip-page.JPG" width="800">
</p>
- Finish configuration

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/034-artifactory-finish-config-page.JPG" width="800">
</p>
- JFrog Artifactory home page

<p align="center">
<img src="https://github.com/ahosny-repo/software-development-home-lab/blob/main/images/035-artifactory-home-page.JPG" width="800">
</p>
- Now, Jenkins, SonarQube and JFrog Artifactory are all set. Congratulations!
