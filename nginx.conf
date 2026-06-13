````markdown
# Nginx Configuration for Roboshop Frontend

This Nginx configuration serves as the frontend web server and reverse proxy for the Roboshop microservices application.

The configuration performs:

- Serves static frontend content
- Routes API requests to backend microservices
- Exposes health check endpoint
- Generates access and error logs
- Handles custom error pages
- Caches images
- Provides Nginx metrics endpoint

This is a common production architecture where Nginx acts as an entry point for users and routes traffic to internal services.

---

# Architecture Overview

```text
                Users
                  │
                  ▼
             Nginx (80)
                  │
      ┌───────────┼───────────┐
      │           │           │
      ▼           ▼           ▼
 Catalogue      User        Cart
   :8080       :8080       :8080
      │
      ├──────────────► Shipping :8080
      │
      └──────────────► Payment  :8080
```

---

# Nginx Configuration

```nginx
# Run Nginx worker processes as nginx user
user nginx;

# Automatically determine number of worker processes
worker_processes auto;

# Error log file location
error_log /var/log/nginx/error.log notice;

# Process ID file
pid /run/nginx.pid;

# Load dynamic modules
include /usr/share/nginx/modules/*.conf;

# ---------------------------------------------------
# EVENTS BLOCK
# ---------------------------------------------------

events {

    # Maximum simultaneous connections per worker
    worker_connections 1024;
}

# ---------------------------------------------------
# HTTP BLOCK
# ---------------------------------------------------

http {

    # Custom access log format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    # Access log location
    access_log /var/log/nginx/access.log main;

    # Enable efficient file transfers
    sendfile on;

    # Optimize packet transmission
    tcp_nopush on;

    # Keep connection open for reuse
    keepalive_timeout 65;

    # Hash table size for MIME types
    types_hash_max_size 4096;

    # MIME types configuration
    include /etc/nginx/mime.types;

    default_type application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {

        # Listen on IPv4
        listen 80;

        # Listen on IPv6
        listen [::]:80;

        # Default hostname
        server_name _;

        # Static website root
        root /usr/share/nginx/html;

        include /etc/nginx/default.d/*.conf;

        # ---------------------------------------------------
        # CUSTOM ERROR PAGES
        # ---------------------------------------------------

        error_page 404 /404.html;

        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;

        location = /50x.html {
        }

        # ---------------------------------------------------
        # IMAGE CACHING
        # ---------------------------------------------------

        location /images/ {

          # Cache images for 5 seconds
          expires 5s;

          root /usr/share/nginx/html;

          # Return placeholder image if image doesn't exist
          try_files $uri /images/placeholder.jpg;
        }

        # ---------------------------------------------------
        # MICROSERVICE ROUTING
        # ---------------------------------------------------

        location /api/catalogue/ {
          proxy_pass http://catalogue:8080/;
        }

        location /api/user/ {
          proxy_pass http://user:8080/;
        }

        location /api/cart/ {
          proxy_pass http://cart:8080/;
        }

        location /api/shipping/ {
          proxy_pass http://shipping:8080/;
        }

        location /api/payment/ {
          proxy_pass http://payment:8080/;
        }

        # ---------------------------------------------------
        # HEALTH CHECK ENDPOINT
        # ---------------------------------------------------

        location /health {

          # Expose Nginx statistics
          stub_status on;

          # Disable access logging
          access_log off;
        }
    }
}
```

---

# User Directive

```nginx
user nginx;
```

Runs worker processes using:

```text
nginx user
```

Benefits:

- Better security
- Reduced privileges
- Least privilege principle

---

# Worker Processes

```nginx
worker_processes auto;
```

Automatically determines:

```text
Number of CPU Cores
```

Example:

```text
4 CPU Cores
     ↓
4 Worker Processes
```

Benefits:

- Better performance
- Automatic scaling

---

# Error Logs

```nginx
error_log /var/log/nginx/error.log notice;
```

Stores:

```text
Application Errors
Backend Failures
Configuration Issues
```

Example:

```text
502 Bad Gateway
Upstream Timeout
Connection Refused
```

---

# Worker Connections

```nginx
worker_connections 1024;
```

Maximum simultaneous connections per worker.

Example:

```text
4 Workers
      ×
1024 Connections
      =
4096 Connections
```

---

# Access Log Format

```nginx
log_format main
```

Captures:

```text
Client IP
Timestamp
HTTP Request
Status Code
User Agent
Referrer
```

Example:

```text
192.168.1.10 GET /products 200
```

Useful for:

- Troubleshooting
- Monitoring
- Analytics

---

# Static Content Hosting

```nginx
root /usr/share/nginx/html;
```

Serves:

```text
index.html
css files
javascript files
images
```

Frontend requests:

```text
http://server-ip/
```

are served from:

```text
/usr/share/nginx/html
```

---

# Custom Error Pages

## 404 Page

```nginx
error_page 404 /404.html;
```

Shown when:

```text
Requested page not found
```

Example:

```text
/products123
```

---

## 50x Errors

```nginx
error_page 500 502 503 504 /50x.html;
```

Shown during:

```text
Application Failure
Backend Down
Gateway Timeout
```

---

# Image Caching

```nginx
location /images/
```

Purpose:

```text
Improve Performance
Reduce Bandwidth Usage
```

---

## Cache Duration

```nginx
expires 5s;
```

Browser caches images for:

```text
5 Seconds
```

---

## Placeholder Images

```nginx
try_files $uri /images/placeholder.jpg;
```

If image doesn't exist:

```text
Requested Image
        ↓
Not Found
        ↓
placeholder.jpg
```

Benefits:

- Better user experience
- No broken images

---

# Reverse Proxy Configuration

Nginx forwards API requests to backend microservices.

---

## Catalogue Service

```nginx
location /api/catalogue/
```

Routes to:

```text
catalogue:8080
```

Example:

```text
/api/catalogue/products
```

---

## User Service

```nginx
location /api/user/
```

Routes to:

```text
user:8080
```

Handles:

```text
Login
Registration
Authentication
```

---

## Cart Service

```nginx
location /api/cart/
```

Routes to:

```text
cart:8080
```

Handles:

```text
Shopping Cart Operations
```

---

## Shipping Service

```nginx
location /api/shipping/
```

Routes to:

```text
shipping:8080
```

Handles:

```text
Shipping Charges
Address Validation
```

---

## Payment Service

```nginx
location /api/payment/
```

Routes to:

```text
payment:8080
```

Handles:

```text
Payment Processing
Transaction Validation
```

---

# Health Endpoint

```nginx
location /health
```

Provides:

```text
Nginx Health Information
```

Uses:

```nginx
stub_status on;
```

Example Output:

```text
Active connections: 5
server accepts handled requests
1000 1000 5000
```

Useful for:

- Monitoring
- Prometheus Exporters
- Load Balancer Health Checks

---

# Disable Logging for Health Checks

```nginx
access_log off;
```

Prevents:

```text
Health Check Spam
```

from filling log files.

---

# Request Flow Example

User accesses:

```text
http://shop.example.com/api/cart/items
```

Flow:

```text
User
   │
   ▼
Nginx
   │
   ▼
location /api/cart/
   │
   ▼
cart:8080
   │
   ▼
Response
```

---

# Real DevOps Use Cases

## Microservices Gateway

Single entry point for:

```text
Catalogue
User
Cart
Shipping
Payment
```

---

## Kubernetes Ingress Alternative

Nginx can act as:

```text
Reverse Proxy
API Gateway
Load Balancer
```

---

## Monitoring

Expose:

```text
/health
```

for:

- Prometheus
- Grafana
- Load Balancers

---

# Best Practices

✅ Use reverse proxy for microservices

✅ Enable access and error logs

✅ Create health check endpoints

✅ Cache static content

✅ Use custom error pages

✅ Separate frontend and backend services

---

# Benefits

- Centralized Routing
- Improved Performance
- Better Security
- Easier Monitoring
- Simplified Microservice Access
- Production-Ready Architecture

---

# Why This Configuration Is Important

This Nginx configuration demonstrates several production-grade concepts:

- Static Content Hosting
- Reverse Proxying
- Microservice Routing
- Health Monitoring
- Logging
- Caching
- Error Handling

These concepts are commonly used in:

- DevOps Engineering
- Kubernetes Platforms
- Cloud-Native Applications
- E-Commerce Platforms
- Production Web Infrastructure
````
