# Nginx Demo Project

A demonstration project showcasing various Nginx features using Docker containers, including static file serving, URL rewriting, file downloads, reverse proxy, and status monitoring.

## Features

- **Static File Serving** - Hosts HTML pages with custom error handling
- **Pretty URLs** - URL rewriting for clean paths (e.g., `/about` → `about.html`)
- **File Downloads** - Dedicated download zone with forced download headers
- **Reverse Proxy** - Proxies API requests to a separate backend container
- **Status Monitoring** - Nginx stub_status endpoint for health checks
- **Custom Error Pages** - Custom 404 and 50x error pages
- **Security** - Server version hiding and access restrictions

## Prerequisites

- Docker
- Docker Compose

## Project Structure

```bash
nginx-project/
├── docker-compose.yml         # Docker services configuration
├── nginx/
│   └── default.conf           # Nginx server configuration
└── html/
    ├── index.html             # Homepage
    ├── about.html             # About page
    ├── 404.html               # Custom 404 error page
    ├── 50x.html               # Custom 50x error page
    └── downloads/
        └── sample.txt         # Sample downloadable file
```

## 🛠️ Installation & Usage

1. **Clone or navigate to the project directory**

    ```bash
    cd nginx-project
    ```

2. **Start the services**

    ```bash
    docker compose up -d
    ```

3. **Access the application**

    Open your browser and navigate to: `http://localhost:8080`

4. **Stop the services**

    ```bash
    docker compose down
    ```

## Available Endpoints

| Endpoint               | Description                                |
| ---------------------- | ------------------------------------------ |
| `/`                    | Homepage with links to all features        |
| `/about`               | About page (demonstrates URL rewriting)    |
| `/download/sample.txt` | File download with forced download headers |
| `/api/hello`           | Reverse proxy to API container             |
| `/status`              | Nginx status page (localhost only)         |
| `/does-not-exist`      | Demonstrates custom 404 page               |

## Docker Services

### web (Nginx Server)

- **Image**: nginx:alpine
- **Container Name**: nginx_http_server
- **Port**: 8080:80
- **Volumes**:
  - `./html` → `/usr/share/nginx/html` (static files, read-only)
  - `./nginx` → `/etc/nginx/conf.d` (configuration, read-only)

### api (Backend API)

- **Image**: hashicorp/http-echo:1.0
- **Container Name**: demo_api
- **Port**: 5678 (internal)
- **Response**: "Hello from API behind Nginx"

## Nginx Configuration Highlights

### Security

- Server version hiding (`server_tokens off`)
- Access restrictions on status endpoint (localhost only)

### URL Rewriting

```nginx
location = /about {
    try_files /about.html =404;
}
```

### Download Zone

```nginx
location /download/ {
    alias /usr/share/nginx/html/downloads/;
    autoindex on;
    add_header Content-Disposition "attachment";
}
```

### Reverse Proxy

```nginx
location /api/ {
    proxy_pass http://api:5678/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_connect_timeout 3s;
    proxy_read_timeout 5s;
}
```

## Testing

Test the various features:

```bash
# Test homepage
curl http://localhost:8080/

# Test pretty URL
curl http://localhost:8080/about

# Test file download
curl -O http://localhost:8080/download/sample.txt

# Test API proxy
curl http://localhost:8080/api/hello

# Test custom 404
curl http://localhost:8080/does-not-exist

# Test status (from inside container)
docker exec nginx_http_server curl http://127.0.0.1/status
```

## Monitoring

View logs:

```bash
# All logs
docker-compose logs -f

# Nginx logs only
docker-compose logs -f web

# Access Nginx access logs
docker exec nginx_http_server cat /var/log/nginx/access.log

# Access Nginx error logs
docker exec nginx_http_server cat /var/log/nginx/error.log
```
