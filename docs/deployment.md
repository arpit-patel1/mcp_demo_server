# MCP Server Deployment Guide

This guide provides instructions for deploying the MCP Server in various environments, from development to production.

## Prerequisites

Before deploying the MCP Server, ensure you have the following:

- Python 3.8 or higher
- pip package manager
- Access to target Kubernetes clusters (for Kubernetes module)
- Network connectivity to target network devices (for Network module)
- Sufficient permissions to install packages and create services

## Installation Options

The MCP Server can be deployed in several ways:

1. **Direct Installation**: Install directly on a host
2. **Docker Container**: Deploy as a containerized application
3. **Kubernetes Deployment**: Deploy on a Kubernetes cluster
4. **High-Availability Setup**: Deploy in a redundant configuration

## Direct Installation

### System Requirements

- Operating System: Linux (recommended), macOS, or Windows
- CPU: 2+ cores
- Memory: 4GB+ RAM
- Disk: 20GB+ free space

### Installation Steps

1. **Clone the repository**:

```bash
git clone https://github.com/your-org/mcp-server.git
cd mcp-server
```

2. **Create a virtual environment**:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**:

```bash
pip install -r requirements.txt
```

4. **Configure the server**:

```bash
cp config/config.example.yaml config/config.yaml
# Edit config/config.yaml with your settings
```

5. **Initialize the database**:

```bash
python -m mcp_server.db.init
```

6. **Start the server**:

```bash
python -m mcp_server.server
```

### Running as a Service

To run the MCP Server as a system service on Linux (systemd):

1. **Create a service file**:

```bash
sudo nano /etc/systemd/system/mcp-server.service
```

2. **Add the following content**:

```ini
[Unit]
Description=MCP Server
After=network.target

[Service]
User=mcp
WorkingDirectory=/path/to/mcp-server
ExecStart=/path/to/mcp-server/venv/bin/python -m mcp_server.server
Restart=on-failure
RestartSec=5
Environment=MCP_CONFIG_FILE=/path/to/mcp-server/config/config.yaml

[Install]
WantedBy=multi-user.target
```

3. **Enable and start the service**:

```bash
sudo systemctl daemon-reload
sudo systemctl enable mcp-server
sudo systemctl start mcp-server
```

4. **Check the service status**:

```bash
sudo systemctl status mcp-server
```

## Docker Deployment

### Prerequisites

- Docker Engine 19.03 or higher
- Docker Compose (optional, for multi-container deployment)

### Building the Docker Image

1. **Build the image**:

```bash
docker build -t mcp-server:latest .
```

2. **Run the container**:

```bash
docker run -d \
  --name mcp-server \
  -p 8000:8000 \
  -v /path/to/config.yaml:/app/config/config.yaml \
  -v /path/to/data:/app/data \
  mcp-server:latest
```

### Using Docker Compose

For a more complex setup with additional services (like a database), use Docker Compose:

1. **Create a docker-compose.yml file**:

```yaml
version: '3'

services:
  mcp-server:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - ./config:/app/config
      - ./data:/app/data
    environment:
      - MCP_CONFIG_FILE=/app/config/config.yaml
      - MCP_LOG_LEVEL=INFO
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secure_password
      - POSTGRES_USER=mcp
      - POSTGRES_DB=mcp
    restart: unless-stopped

volumes:
  postgres_data:
```

2. **Start the services**:

```bash
docker-compose up -d
```

3. **Check the logs**:

```bash
docker-compose logs -f mcp-server
```

## Kubernetes Deployment

### Prerequisites

- Kubernetes cluster 1.19 or higher
- kubectl configured to access the cluster
- Helm 3 (optional, for Helm chart deployment)

### Deployment with kubectl

1. **Create a namespace**:

```bash
kubectl create namespace mcp-server
```

2. **Create a ConfigMap for configuration**:

```bash
kubectl create configmap mcp-config --from-file=config.yaml=./config/config.yaml -n mcp-server
```

3. **Create a Secret for sensitive data**:

```bash
kubectl create secret generic mcp-secrets \
  --from-literal=db-password=secure_password \
  --from-literal=jwt-secret=your_jwt_secret \
  -n mcp-server
```

4. **Apply the deployment manifests**:

```bash
kubectl apply -f kubernetes/deployment.yaml -n mcp-server
kubectl apply -f kubernetes/service.yaml -n mcp-server
kubectl apply -f kubernetes/ingress.yaml -n mcp-server
```

5. **Check the deployment status**:

```bash
kubectl get pods -n mcp-server
kubectl get services -n mcp-server
kubectl get ingress -n mcp-server
```

### Deployment with Helm

1. **Install the Helm chart**:

```bash
helm install mcp-server ./charts/mcp-server \
  --namespace mcp-server \
  --create-namespace \
  --set config.logLevel=INFO \
  --set persistence.enabled=true \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=mcp.example.com
```

2. **Upgrade the deployment**:

```bash
helm upgrade mcp-server ./charts/mcp-server \
  --namespace mcp-server \
  --set config.logLevel=DEBUG
```

## High-Availability Setup

For production environments, a high-availability setup is recommended:

### Architecture

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    └────────┬────────┘
                             │
             ┌───────────────┴───────────────┐
             │                               │
      ┌──────▼─────┐                  ┌──────▼─────┐
      │ MCP Server │                  │ MCP Server │
      │ Instance 1 │                  │ Instance 2 │
      └──────┬─────┘                  └──────┬─────┘
             │                               │
             └───────────────┬───────────────┘
                             │
                    ┌────────▼────────┐
                    │  Database Cluster│
                    └─────────────────┘
```

### Implementation with Kubernetes

1. **Deploy multiple replicas**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcp-server
  namespace: mcp-server
spec:
  replicas: 3  # Multiple replicas for HA
  selector:
    matchLabels:
      app: mcp-server
  template:
    metadata:
      labels:
        app: mcp-server
    spec:
      containers:
      - name: mcp-server
        image: mcp-server:latest
        # ... other configuration ...
```

2. **Use a database cluster**:

For PostgreSQL, consider using a managed service or an operator like:
- AWS RDS
- Google Cloud SQL
- Azure Database for PostgreSQL
- Postgres Operator for Kubernetes

3. **Implement session persistence**:

If using JWT tokens, no special configuration is needed as they are stateless.
For API keys or other stateful sessions, use a distributed cache like Redis.

## Configuration

### Configuration File

The MCP Server uses a YAML configuration file. Here's an example:

```yaml
server:
  host: 0.0.0.0
  port: 8000
  workers: 4
  log_level: INFO
  debug: false

auth:
  jwt_secret: your_jwt_secret
  token_expiration: 3600  # seconds
  refresh_token_expiration: 86400  # seconds

database:
  type: postgresql
  host: localhost
  port: 5432
  name: mcp
  user: mcp
  password: secure_password
  pool_size: 10

kubernetes:
  connection_timeout: 30
  watch_timeout: 300
  max_connections_per_cluster: 5

network:
  connection_timeout: 60
  command_timeout: 30
  max_connections_per_device: 3
  session_log_dir: /app/data/sessions
```

### Environment Variables

Configuration can also be provided through environment variables:

```
MCP_SERVER_HOST=0.0.0.0
MCP_SERVER_PORT=8000
MCP_SERVER_WORKERS=4
MCP_SERVER_LOG_LEVEL=INFO
MCP_SERVER_DEBUG=false

MCP_AUTH_JWT_SECRET=your_jwt_secret
MCP_AUTH_TOKEN_EXPIRATION=3600
MCP_AUTH_REFRESH_TOKEN_EXPIRATION=86400

MCP_DATABASE_TYPE=postgresql
MCP_DATABASE_HOST=localhost
MCP_DATABASE_PORT=5432
MCP_DATABASE_NAME=mcp
MCP_DATABASE_USER=mcp
MCP_DATABASE_PASSWORD=secure_password
MCP_DATABASE_POOL_SIZE=10

MCP_KUBERNETES_CONNECTION_TIMEOUT=30
MCP_KUBERNETES_WATCH_TIMEOUT=300
MCP_KUBERNETES_MAX_CONNECTIONS_PER_CLUSTER=5

MCP_NETWORK_CONNECTION_TIMEOUT=60
MCP_NETWORK_COMMAND_TIMEOUT=30
MCP_NETWORK_MAX_CONNECTIONS_PER_DEVICE=3
MCP_NETWORK_SESSION_LOG_DIR=/app/data/sessions
```

## Security Considerations

### TLS Configuration

For production deployments, TLS should be enabled:

1. **Generate certificates**:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout server.key -out server.crt
```

2. **Configure TLS in the server**:

```yaml
server:
  # ... other settings ...
  tls:
    enabled: true
    cert_file: /path/to/server.crt
    key_file: /path/to/server.key
```

3. **Or use a reverse proxy** (recommended for production):

- Nginx
- Apache
- Traefik
- Kubernetes Ingress with cert-manager

### Secure Credential Storage

For storing sensitive credentials:

1. **Use environment variables** (preferred for container deployments)
2. **Use a secrets manager**:
   - HashiCorp Vault
   - AWS Secrets Manager
   - Google Secret Manager
   - Azure Key Vault

Example configuration for HashiCorp Vault:

```yaml
secrets:
  provider: vault
  vault:
    url: https://vault.example.com
    token: s.your-vault-token
    mount_path: secret
    path_prefix: mcp-server
```

## Monitoring and Logging

### Logging Configuration

The MCP Server uses structured logging. Configure the log level in the configuration file:

```yaml
server:
  log_level: INFO  # DEBUG, INFO, WARNING, ERROR, CRITICAL
  log_format: json  # json or text
  log_file: /var/log/mcp-server.log  # Optional, logs to stdout if not specified
```

### Health Checks

The server provides health check endpoints:

- `/health/liveness`: Basic server liveness check
- `/health/readiness`: Checks if the server is ready to accept requests
- `/health/database`: Checks database connectivity
- `/health/kubernetes`: Checks Kubernetes module status
- `/health/network`: Checks Network module status

### Metrics

The server exposes Prometheus metrics at `/metrics`, including:

- Request counts and latencies
- Error rates
- Connection pool statistics
- Resource usage metrics

### Integration with Monitoring Systems

1. **Prometheus and Grafana**:

```yaml
monitoring:
  prometheus:
    enabled: true
    endpoint: /metrics
```

2. **ELK Stack**:

Configure Filebeat to collect logs:

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/mcp-server.log
  json.keys_under_root: true
  json.add_error_key: true
```

## Backup and Recovery

### Database Backup

For PostgreSQL:

```bash
pg_dump -U mcp -d mcp -h localhost -F c -f mcp_backup.dump
```

### Configuration Backup

Regularly back up the configuration files:

```bash
cp /path/to/mcp-server/config/config.yaml /path/to/backups/config_$(date +%Y%m%d).yaml
```

### Recovery Procedure

1. **Restore the database**:

```bash
pg_restore -U mcp -d mcp -h localhost -c mcp_backup.dump
```

2. **Restore configuration**:

```bash
cp /path/to/backups/config_20230101.yaml /path/to/mcp-server/config/config.yaml
```

3. **Restart the server**:

```bash
sudo systemctl restart mcp-server
```

## Scaling Considerations

### Horizontal Scaling

To handle increased load:

1. **Increase the number of server instances**
2. **Use a load balancer to distribute traffic**
3. **Ensure database can handle the increased connections**

### Vertical Scaling

To improve single-instance performance:

1. **Increase the number of worker processes**:

```yaml
server:
  workers: 8  # Adjust based on available CPU cores
```

2. **Optimize database connection pooling**:

```yaml
database:
  pool_size: 20  # Adjust based on workload
```

3. **Increase resource allocation** (CPU, memory)

## Troubleshooting

### Common Issues

1. **Server won't start**:
   - Check logs for errors
   - Verify configuration file syntax
   - Ensure database connectivity
   - Check port availability

2. **Authentication failures**:
   - Verify JWT secret configuration
   - Check database user credentials
   - Ensure clock synchronization between servers

3. **Module connectivity issues**:
   - Check network connectivity to Kubernetes clusters
   - Verify network device reachability
   - Check credential validity

### Diagnostic Commands

1. **Check server status**:

```bash
sudo systemctl status mcp-server
```

2. **View logs**:

```bash
sudo journalctl -u mcp-server -f
```

3. **Test database connectivity**:

```bash
python -c "from mcp_server.db import test_connection; test_connection()"
```

4. **Verify API accessibility**:

```bash
curl -i http://localhost:8000/health/liveness
```

## Upgrade Procedure

### Standard Upgrade

1. **Backup the current state**:
   - Database backup
   - Configuration backup

2. **Stop the server**:

```bash
sudo systemctl stop mcp-server
```

3. **Update the code**:

```bash
cd /path/to/mcp-server
git pull
```

4. **Update dependencies**:

```bash
source venv/bin/activate
pip install -r requirements.txt
```

5. **Apply database migrations**:

```bash
python -m mcp_server.db.migrate
```

6. **Start the server**:

```bash
sudo systemctl start mcp-server
```

### Zero-Downtime Upgrade (Kubernetes)

Using Kubernetes rolling updates:

```bash
kubectl set image deployment/mcp-server mcp-server=mcp-server:new-version -n mcp-server
```

Or with Helm:

```bash
helm upgrade mcp-server ./charts/mcp-server \
  --namespace mcp-server \
  --set image.tag=new-version
```

## Conclusion

This deployment guide covers the essential aspects of deploying and maintaining the MCP Server in various environments. For specific requirements or advanced configurations, refer to the detailed documentation or contact the development team. 