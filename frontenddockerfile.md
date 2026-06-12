# Nginx Dockerfile – Detailed Explanation

This Dockerfile builds a secure Nginx container that serves static content while running as a non-root user (`nginx`). It removes default configurations, prepares required directories, sets appropriate permissions, and starts Nginx in the foreground.

---

## Base Image

```dockerfile
FROM nginx:1.27-alpine
```

Uses the lightweight Alpine-based Nginx image.

### Benefits

- Small image size
- Faster downloads
- Reduced attack surface

---

## Remove Default Configurations

```
RUN rm /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf
```

Removes the default Nginx configuration files.

### Why?

- Avoid unwanted default behavior
- Use a fully customized Nginx configuration

---

## Create Cache Directories

```dockerfile
RUN mkdir -p /var/cache/nginx/client_temp && \
        mkdir -p /var/cache/nginx/proxy_temp && \
        mkdir -p /var/cache/nginx/fastcgi_temp && \
        mkdir -p /var/cache/nginx/uwsgi_temp && \
        mkdir -p /var/cache/nginx/scgi_temp
```

Nginx uses these directories for temporary file storage.

| Directory | Purpose |
|------------|----------|
| client_temp | Client upload buffering |
| proxy_temp | Proxy response buffering |
| fastcgi_temp | FastCGI buffering |
| uwsgi_temp | uWSGI buffering |
| scgi_temp | SCGI buffering |

---

## Configure Permissions

```dockerfile
chown -R nginx:nginx /var/cache/nginx && \
chown -R nginx:nginx /etc/nginx/ && \
chmod -R 755 /etc/nginx/ && \
chown -R nginx:nginx /var/log/nginx
```

### Purpose

Allows the non-root nginx user to:

- Read Nginx configuration
- Write cache files
- Write access logs
- Write error logs

### Permission Breakdown

```text
755
├── Owner  : Read Write Execute
├── Group  : Read Execute
└── Others : Read Execute
```

---

## Create SSL Directory

```dockerfile
RUN mkdir -p /etc/nginx/ssl/ && \
    chown -R nginx:nginx /etc/nginx/ssl/ && \
    chmod -R 755 /etc/nginx/ssl/
```

Stores TLS certificates and private keys.

Example:

```text
/etc/nginx/ssl/server.crt
/etc/nginx/ssl/server.key
```

---

## Create PID File

```dockerfile
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid /run/nginx.pid
```

Nginx stores its process ID in:

```text
/var/run/nginx.pid
```

Without proper permissions, Nginx may fail with:

```text
open() "/var/run/nginx.pid" failed (13: Permission denied)
```

---

## Copy Custom Configuration

```dockerfile
COPY nginx.conf /etc/nginx/nginx.conf
```

Copies your custom Nginx configuration into the container.

---

## Copy Static Website

```dockerfile
COPY static /usr/share/nginx/html/
```

Copies website content into Nginx's document root.

Example:

```text
static/
├── index.html
├── css/
├── js/
└── images/
```

becomes:

```text
/usr/share/nginx/html/
```

---

## Run as Non-Root User

```dockerfile
USER nginx
```

Runs the container as:

```text
nginx
```

instead of:

```text
root
```

### Benefits

- Better security
- Prevents privilege escalation
- Kubernetes security best practice

---

## Start Nginx

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Equivalent to:

```bash
nginx -g "daemon off;"
```

### Why `daemon off;`?

Normally Nginx runs in the background.

Docker containers require the main process to stay in the foreground.

This keeps Nginx attached to PID 1 and prevents the container from exiting.

---

## Container Startup Flow

```text
Container Starts
       │
       ▼
Runs as nginx User
       │
       ▼
Reads nginx.conf
       │
       ▼
Loads Static Files
       │
       ▼
Creates Cache Files
       │
       ▼
Writes Logs
       │
       ▼
Serves Website
```

---

## Why So Many Permission Commands?

Because the container runs as:

```dockerfile
USER nginx
```

instead of:

```dockerfile
USER root
```

The nginx user cannot write to:

```text
/etc/nginx
/var/cache/nginx
/var/log/nginx
/var/run
```

unless permissions are explicitly granted.

---

## Interview Questions

### Why run Nginx as non-root?

- Improved security
- Reduced attack surface
- Kubernetes best practice
- Compliance requirements

---

### Why remove default nginx.conf?

To use a custom configuration and avoid default behavior.

---

### Why create cache directories manually?

When running as a non-root user, Nginx may not have permission to create them automatically.

---

### Why create nginx.pid manually?

Nginx needs a writable PID file location to manage its process.

---

### Optimization

Combine multiple RUN instructions into a single layer:

```dockerfile
RUN rm /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf && \
    mkdir -p /var/cache/nginx/{client_temp,proxy_temp,fastcgi_temp,uwsgi_temp,scgi_temp} && \
    mkdir -p /etc/nginx/ssl && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/cache/nginx /etc/nginx /var/log/nginx /var/run/nginx.pid && \
    chmod -R 755 /etc/nginx
```

Benefits:

- Fewer image layers
- Smaller image size
- Faster builds