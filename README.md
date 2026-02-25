# Nginx Reverse Proxy with Load Balancing (CentOS)

## 📌 Objective

Designed and implemented a two-tier web architecture using Nginx as a
reverse proxy to distribute client traffic across multiple backend
services using round-robin load balancing.

------------------------------------------------------------------------

## 🏗 Architecture Overview

**Proxy Server:** 192.168.1.104\
**Backend Server:** 192.168.1.106

**Backend Services:** - Nginx (Port 80) - Python HTTP Server (Port 8080)

### Traffic Flow

Client → Proxy (192.168.1.104) → Backend Pool → Response

------------------------------------------------------------------------

## ⚙️ Reverse Proxy Configuration

File: `/etc/nginx/conf.d/reverse.conf`

    upstream backend_servers {
        server 192.168.1.106:80;
        server 192.168.1.106:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_servers;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

------------------------------------------------------------------------

## 🔁 Load Balancing Method

-   Default algorithm: Round Robin
-   Requests distributed sequentially across backend services
-   If one backend fails, Nginx automatically routes traffic to
    available servers

------------------------------------------------------------------------

## 🔐 Security & System Configuration

### SELinux

SELinux was in Enforcing mode. Reverse proxy initially returned **502
Bad Gateway** due to network restrictions.

**Resolution:**

    setsebool -P httpd_can_network_connect 1

### Firewall

Port 8080 was blocked initially, causing connection failure.

**Resolution:**

    firewall-cmd --permanent --add-port=8080/tcp
    firewall-cmd --reload

------------------------------------------------------------------------

## 🧪 Testing & Validation

-   Verified load balancing using repeated `curl http://192.168.1.104`
-   Confirmed alternating responses between backend services
-   Simulated backend failure to validate fallback behavior
-   Differentiated HTTP errors:
    -   502 → Backend unreachable
    -   403 → Permission denied
    -   504 → Backend timeout

------------------------------------------------------------------------

## 📚 Key Concepts Demonstrated

-   Reverse Proxy Architecture
-   Nginx Upstream Load Balancing
-   HTTP Error Diagnosis (502, 403, 504)
-   SELinux Boolean Configuration
-   Firewall Port Management
-   Backend Service Validation

------------------------------------------------------------------------

## 🚀 Future Improvements

-   Implement HTTPS termination
-   Configure health checks
-   Add weighted load balancing
-   Integrate monitoring and logging stack

------------------------------------------------------------------------

**Author:** Deepthi Karnatakapu\
**Role:** Technical Support Engineer \| Linux Infrastructure Enthusiast
