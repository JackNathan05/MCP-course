
Date: 2025-04-10

Tags: #Courses #AI #Software

Backlinks: [[_Lesson 4]]

---

## Practical Exercise 2.4

**Objective**: Deploy and secure an MCP server

### Instructions

1. Containerize your MCP server using Docker:
   - Create a Dockerfile for your MCP server
   - Build a Docker image
   - Run the container locally
   - Test that the server works correctly in the container

2. Implement OAuth authentication:
   - Set up a simple OAuth provider (you can use a mock for testing)
   - Configure your MCP server to use OAuth for authentication
   - Implement token validation
   - Test authentication with valid and invalid tokens

3. Set up monitoring using a logging framework:
   - Add structured logging to your server
   - Log key events (requests, responses, errors)
   - Implement basic metrics collection
   - Create a simple dashboard or log viewer

4. Create a deployment script or configuration:
   - Write a script to automate deployment
   - Include environment configuration
   - Add health checks
   - Document the deployment process

5. Test the deployment in a local or cloud environment:
   - Deploy the server
   - Verify it's running correctly
   - Test authentication
   - Check that logs and metrics are working

### Expected Outcome

A containerized MCP server with:
- Docker configuration
- OAuth authentication
- Logging and monitoring
- Deployment automation
- Documentation

## Code Example (Docker Configuration)

Here's a starter Dockerfile for your MCP server:

```dockerfile
# Dockerfile for MCP server
FROM node:18-alpine

# Create app directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Build TypeScript (if applicable)
RUN npm run build

# Expose the MCP server port
EXPOSE 8000

# Set environment variables
ENV NODE_ENV=production
ENV MCP_PORT=8000
ENV MCP_AUTH_ENABLED=true

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget -qO- http://localhost:8000/health || exit 1

# Run the server
CMD ["node", "dist/server.js"]
```

And a simple deployment script:

```bash
#!/bin/bash

# MCP Server Deployment Script

# Configuration
IMAGE_NAME="mcp-server"
CONTAINER_NAME="mcp-server"
PORT=8000
ENV_FILE=".env.production"

# Build the Docker image
echo "Building Docker image..."
docker build -t $IMAGE_NAME .

# Stop and remove existing container if it exists
echo "Stopping existing container if running..."
docker stop $CONTAINER_NAME 2>/dev/null || true
docker rm $CONTAINER_NAME 2>/dev/null || true

# Run the new container
echo "Starting new container..."
docker run -d \
  --name $CONTAINER_NAME \
  --restart unless-stopped \
  --env-file $ENV_FILE \
  -p $PORT:8000 \
  $IMAGE_NAME

# Check if container is running
echo "Checking container status..."
CONTAINER_RUNNING=$(docker ps --filter "name=$CONTAINER_NAME" --format "{{.Names}}")

if [ "$CONTAINER_RUNNING" == "$CONTAINER_NAME" ]; then
  echo "Container $CONTAINER_NAME is running."
  
  # Wait for health check
  echo "Waiting for health check to pass..."
  for i in {1..10}; do
    HEALTH=$(docker inspect --format "{{.State.Health.Status}}" $CONTAINER_NAME)
    if [ "$HEALTH" == "healthy" ]; then
      echo "Container is healthy!"
      break
    fi
    
    if [ $i -eq 10 ]; then
      echo "Health check failed after 10 attempts."
      exit 1
    fi
    
    echo "Waiting for container to become healthy (attempt $i)..."
    sleep 3
  done
  
  echo "Deployment completed successfully."
else
  echo "Failed to start container."
  exit 1
fi
```

## Extension Activities

1. **Kubernetes Deployment**:
   - Create Kubernetes configuration files for your MCP server
   - Deploy on a Kubernetes cluster (local or cloud)
   - Set up auto-scaling
   - Configure liveness and readiness probes

2. **CI/CD Pipeline**:
   - Set up a GitHub Actions or GitLab CI pipeline
   - Automate testing and deployment
   - Implement environment promotion (dev, staging, prod)
   - Add security scanning

3. **Advanced Monitoring**:
   - Implement Prometheus metrics
   - Set up Grafana dashboards
   - Configure alerts for critical issues
   - Add distributed tracing

## Recommended Reading

1. [MCP Deployment Guide](https://modelcontextprotocol.io/docs/guides/deployment/)
   - Deployment best practices
   - Environment configurations
   - Security considerations

2. [Security Best Practices](https://modelcontextprotocol.io/docs/guides/security/)
   - Authentication implementation
   - Authorization patterns
   - Data protection strategies
   - Security checklist

3. [Monitoring and Observability](https://modelcontextprotocol.io/docs/guides/monitoring/)
   - Logging standards
   - Metrics collection
   - Performance analysis
   - Alerting systems

## Self-Assessment Questions

Test your understanding by answering these questions:

1. What deployment options are available for MCP servers?
2. How should OAuth 2.1 be implemented with MCP?
3. What performance optimization techniques can improve MCP server efficiency?
4. What metrics should be monitored for MCP servers?
5. What maintenance procedures should be established for production MCP servers?
6. What are the advantages and disadvantages of containerized deployment?
7. How can you ensure high availability for MCP servers?
8. What security considerations are important when deploying MCP servers?

## Deployment Planning Exercise

Create a deployment plan for a production MCP server that includes:

1. **Infrastructure Requirements**:
   - Server specifications
   - Network configuration
   - Storage requirements
   - Scaling strategy

2. **Security Plan**:
   - Authentication configuration
   - Authorization model
   - Data protection measures
   - Access controls

3. **Monitoring Strategy**:
   - Key metrics to track
   - Logging configuration
   - Alerting thresholds
   - Incident response process

4. **Maintenance Schedule**:
   - Update procedures
   - Backup strategy
   - Performance tuning
   - Disaster recovery plan

5. **Documentation**:
   - Deployment instructions
   - Configuration reference
   - Troubleshooting guide
   - Security protocols

## Cloud Deployment Comparison

Research and compare deploying MCP servers on different cloud platforms:

1. **AWS**:
   - Deployment options (EC2, ECS, EKS)
   - Authentication integration (Cognito)
   - Monitoring solutions (CloudWatch)
   - Cost considerations

2. **Azure**:
   - Deployment options (App Service, AKS)
   - Authentication integration (Azure AD)
   - Monitoring solutions (Azure Monitor)
   - Cost considerations

3. **Google Cloud**:
   - Deployment options (GCE, GKE)
   - Authentication integration (Identity Platform)
   - Monitoring solutions (Cloud Monitoring)
   - Cost considerations

Create a comparison matrix highlighting the pros and cons of each platform for MCP deployment.

## Additional Resources

### Deployment Guides
- [Docker Deployment Best Practices](https://modelcontextprotocol.io/docs/guides/deployment/docker)
- [Kubernetes Deployment](https://modelcontextprotocol.io/docs/guides/deployment/kubernetes)
- [Serverless Deployment](https://modelcontextprotocol.io/docs/guides/deployment/serverless)

### Security Resources
- [OAuth 2.1 Implementation Guide](https://modelcontextprotocol.io/docs/guides/security/oauth)
- [API Security Checklist](https://modelcontextprotocol.io/docs/guides/security/checklist)
- [Security Auditing](https://modelcontextprotocol.io/docs/guides/security/auditing)

### Monitoring Solutions
- [Prometheus for MCP](https://modelcontextprotocol.io/docs/guides/monitoring/prometheus)
- [Grafana Dashboards](https://modelcontextprotocol.io/docs/guides/monitoring/grafana)
- [Log Analysis](https://modelcontextprotocol.io/docs/guides/monitoring/logging)
