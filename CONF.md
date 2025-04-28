<a href="#">
    <img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=9745f5&height=120&section=header" />
</a>

![Typing SVG](https://readme-typing-svg.herokuapp.com/?color=9745f5&size=35&center=true&vCenter=true&width=1000&lines=Meet+the+Team;Ismail+Enniou;Malika+El+Abderrahmani;Fatima+El+Asri;Welcome+to+Our+Project!+:)

---

# Deploy Spring Boot Application on a VPS with Nginx Reverse Proxy

## Prerequisites
Ensure the following software is installed on your VPS:

1. **Java Development Kit (JDK)**
   - Install Oracle Java 17:
     ```bash
     sudo add-apt-repository ppa:linuxuprising/java -y
     sudo apt update
     sudo apt-get install oracle-java17-installer
     sudo apt-get install oracle-java17-set-default
     ```

2. **Maven**
   - Install Maven to build your Spring Boot project:
     ```bash
     sudo apt install maven
     ```

3. **Nginx**
   - Install Nginx web server:
     ```bash
     sudo apt update
     sudo apt install nginx
     sudo ufw allow 'Nginx Full'
     sudo systemctl status nginx
     ```

## Setting Up the Project on the Server

1. **Clone the Project**
   - Install Git and clone your project repository:
     ```bash
     sudo apt install git
     git clone https://github.com/username/your-app.git
     ```

2. **Build the JAR File**
   - Navigate to your project directory and build the JAR file using Maven:
     ```bash
     cd your-app
     mvn clean package
     ```

## Nginx Configuration for Reverse Proxy

### 1. **Initial Nginx Configuration**

   - Create a simple Nginx configuration file for your domain in `/etc/nginx/sites-available/yourdomain`:

     ```bash
     sudo vi /etc/nginx/sites-available/yourdomain
     ```

   - Add the following configuration for reverse proxying to your Spring Boot app:

     ```nginx
     server {
         server_name yourdomain.com;  # Replace with your domain name
         index index.html index.htm;

         access_log /var/log/nginx/yourdomain-access.log;
         error_log /var/log/nginx/yourdomain-error.log error;

         location / {
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header Host $http_host;
             proxy_pass http://127.0.0.1:8080;  # Ensure this matches your Spring Boot port
             proxy_redirect off;
         }
     }
     ```

   - Enable the configuration by creating a symbolic link:
     ```bash
     sudo ln -s /etc/nginx/sites-available/yourdomain /etc/nginx/sites-enabled/
     ```

   - Test the Nginx configuration:
     ```bash
     sudo nginx -t
     ```

   - Restart Nginx:
     ```bash
     sudo systemctl restart nginx
     ```

### 2. **Enhanced Nginx Configuration with HTTPS**

   - Now, create another configuration file for enhanced functionality with SSL in `/etc/nginx/sites-available/yourdomain.conf`:

     ```bash
     sudo vi /etc/nginx/sites-available/yourdomain.conf
     ```

   - Add the following content, which includes SSL and file caching:

     ```nginx
     server {
         listen 80;
         listen [::]:80;

         listen 443 ssl http2;
         listen [::]:443 ssl http2;

         server_name yourdomain.com;  # Replace with your domain
         root /var/www/html;  # Update as needed

         access_log /var/log/nginx/yourdomain-access.log;
         error_log /var/log/nginx/yourdomain-error.log;

         # Redirect all HTTP traffic to HTTPS
         if ($scheme != "https") {
             return 301 https://$host$request_uri;
         }

         location / {
             try_files $uri @proxy;
         }

         location @proxy {
             proxy_pass http://localhost:8080;  # Your Spring Boot app port
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }

         location ~ /.well-known {
             allow all;
         }

         location ~* \.(css|js|jpg|jpeg|gif|png|ico|svg)$ {
             expires 1y;
             add_header Cache-Control "public, must-revalidate, proxy-revalidate";
         }

         location ~* \.(gz|svgz|ttf|otf|woff|woff2|eot|mp4|ogg|ogv|webm|webp|zip|swf)$ {
             expires 1y;
             add_header Cache-Control "public, must-revalidate, proxy-revalidate";
         }
     }
     ```

   - Enable the configuration by creating a symbolic link:
     ```bash
     sudo ln -s /etc/nginx/sites-available/yourdomain.conf /etc/nginx/sites-enabled/
     ```

   - Test the configuration again:
     ```bash
     sudo nginx -t
     ```

   - Restart Nginx:
     ```bash
     sudo systemctl restart nginx
     ```

## Making the Spring Boot Application Always Running

1. **Create a Systemd Service**
   - Create a service file to manage your Spring Boot application:
     ```bash
     sudo nano /etc/systemd/system/springboot.service
     ```

2. **Add the following content:**

   ```ini
   [Unit]
   Description=Spring Boot Application
   After=syslog.target

   [Service]
   User=your-user  # Update with the username running the app
   ExecStart=/usr/bin/java -jar /path/to/your-application.jar
   SuccessExitStatus=143
   Restart=always
   RestartSec=10

   [Install]
   WantedBy=multi-user.target
   ```

3. **Reload Systemd and Start the Service:**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start springboot
   sudo systemctl enable springboot
   ```

## Enable HTTPS with Let’s Encrypt

1. **Install Certbot**
   - Install Certbot for SSL certificates:
     ```bash
     sudo apt install snapd
     sudo snap install --classic certbot
     ```

2. **Obtain SSL Certificate**
   - Run Certbot with the Nginx plugin to generate the certificate:
     ```bash
     sudo certbot --nginx -d yourdomain.com
     ```

3. **Automatic Renewal**
   - Certbot automatically handles renewal, but you can test the renewal process:
     ```bash
     sudo certbot renew --dry-run
     ```

## Conclusion

This guide outlines the steps to deploy a Spring Boot application on a VPS with Nginx as a reverse proxy, setting up both basic and enhanced configurations, making your application run as a service, and enabling HTTPS with Let’s Encrypt.
