Troubleshooting Scenarios

This document outlines real issues encountered during the implementation of the Nginx reverse proxy and load balancing architecture, along with diagnosis and resolution steps.

1️⃣ 502 Bad Gateway – SELinux Blocking Outbound Connection
Issue

Accessing the proxy returned:

502 Bad Gateway
Diagnosis

Backend service was running.

Network connectivity confirmed using curl.

Error logs showed connection failure.

Checked SELinux status:

getenforce

Result: Enforcing

Checked for AVC denials:

ausearch -m avc -ts recent
Root Cause

SELinux prevented Nginx (httpd_t domain) from initiating outbound network connections.

Resolution

Enabled required SELinux boolean:

setsebool -P httpd_can_network_connect 1

Result: Proxy successfully connected to backend.

2️⃣ 502 Bad Gateway – Firewall Blocking Backend Port
Issue

Accessing backend service on port 8080 returned:

No route to host
Diagnosis

Checked listening ports:

ss -tulnp | grep 8080

Service was running.

Checked firewall configuration:

firewall-cmd --list-all

Port 8080 was not open.

Root Cause

Firewall blocking inbound traffic to backend port 8080.

Resolution
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

Backend became reachable.

3️⃣ Address Already in Use – Port Conflict
Issue

Python server failed to start:

OSError: [Errno 98] Address already in use
Diagnosis
ss -tulnp | grep 8080

Port was already bound by an existing process.

Root Cause

Port 8080 already in use by previously started service.

Resolution

Stopped existing process or reused running instance.

4️⃣ 403 Forbidden – File Permission Issue
Issue

Backend returned:

403 Forbidden
Diagnosis

Checked file permissions:

ls -l /usr/share/nginx/html/index.html

File permission was set to 600.

Root Cause

Nginx process did not have read permission for the file.

Resolution

Adjusted permissions appropriately:

chmod 644 /usr/share/nginx/html/index.html



Lessons Learned

Reverse proxy failures can originate from multiple layers: network, firewall, SELinux, application.

502 errors often indicate connectivity failure between proxy and backend.

SELinux enforcement must be understood and configured correctly rather than disabled.

Firewall configuration is critical when exposing non-default ports.

Proper log analysis is essential for root cause identification.
