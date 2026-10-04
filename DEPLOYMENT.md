# Deployment Guide

## Local Development with Docker

### Prerequisites
- Docker installed and running
- At least 2GB of available disk space (for base image + dependencies)
- 20 minutes for first-time build

### Quick Start

1. **Build the Docker image:**
   ```bash
   docker build -t label-check .
   ```

2. **Run the container:**
   ```bash
   docker run -p 8501:8501 label-check
   ```

3. **Access the application:**
   Open your browser and navigate to: **http://localhost:8501**

### Docker Commands Reference

**View running containers:**
```bash
docker ps
```

**Stop the container:**
```bash
docker stop <container_id>
```

**Remove the image:**
```bash
docker rmi label-check
```

**Run with volume mount (for development):**
```bash
docker run -p 8501:8501 -v $(pwd):/app label-check
```

---

## Azure Container Apps Deployment

### Prerequisites
- Azure subscription with Container Apps enabled
- Azure CLI installed
- Logged into Azure (`az login`)

### Steps

1. **Create a resource group:**
   ```bash
   az group create \
     --name label-check-rg \
     --location eastus
   ```

2. **Push to Azure Container Registry (ACR):**
   ```bash
   az acr build --registry <your-acr-name> \
     --image label-check:latest \
     .
   ```

3. **Deploy to Container Apps:**
   ```bash
   az containerapp create \
     --name label-check \
     --resource-group label-check-rg \
     --image <your-acr-name>.azurecr.io/label-check:latest \
     --environment <your-environment> \
     --target-port 8501 \
     --ingress external \
     --cpu 0.5 \
     --memory 1.0Gi
   ```

4. **Get the application URL:**
   ```bash
   az containerapp show \
     --name label-check \
     --resource-group label-check-rg \
     --query properties.configuration.ingress.fqdn
   ```

---

## Render Deployment

### Prerequisites
- Render account (render.com)
- GitHub repository connected to Render
- Project pushed to GitHub

### Steps

1. **Create a new Web Service on Render:**
   - Go to Render Dashboard
   - Click "New +"
   - Select "Web Service"
   - Connect your GitHub repository

2. **Configure the service:**
   - **Name:** `label-check` (or preferred name)
   - **Environment:** Docker
   - **Build Command:** `docker build -t label-check .`
   - **Start Command:** `streamlit run app.py --server.port=8501`
   - **Plan:** Start with Free or Starter plan

3. **Add environment variables (if needed):**
   - No environment variables required for MVP

4. **Deploy:**
   - Click "Create Web Service"
   - Render will build and deploy automatically
   - Access via the provided Render URL

---

## Network & Firewall Considerations

### Why This Solution Works in Restricted Environments

✅ **No Outbound Calls:** The application makes NO external network requests
✅ **Self-Contained:** All processing happens locally within the container
✅ **No Cloud API Dependencies:** Uses open-source libraries (Tesseract, OpenCV)
✅ **Data Privacy:** Labels never leave the local environment
✅ **Firewall-Friendly:** Compatible with strict enterprise firewalls

### What's NOT Required
- ❌ API keys or credentials
- ❌ External image processing services
- ❌ Cloud ML services
- ❌ Outbound HTTPS connections
- ❌ Database connections

---

## Troubleshooting

### Container fails to start
- Check Docker logs: `docker logs <container_id>`
- Ensure port 8501 is not already in use
- Verify Docker daemon is running

### Slow OCR processing
- This is normal for the first image (Tesseract initialization)
- Subsequent images process faster
- Larger images take longer to process

### Image extraction produces no text
- Try a higher resolution image
- Ensure the label text is clearly visible
- Avoid extremely tilted or obscured labels

### Permission denied errors
- On Linux/Mac: may need `sudo` for docker commands
- Consider adding user to docker group: `sudo usermod -aG docker $USER`

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Container Startup Time | ~5-10 seconds |
| Image Upload Processing | 3-8 seconds |
| OCR Text Extraction | 2-5 seconds per label |
| Memory Usage | ~400-600MB |
| Disk Space Required | ~2GB (image + dependencies) |

---

## Production Considerations

For production deployment to Azure Container Apps:

1. **Add authentication** to restrict access to authorized agents
2. **Implement audit logging** for compliance tracking
3. **Add persistent storage** for label archives
4. **Scale instances** based on demand (Container Apps auto-scaling)
5. **Set up monitoring** with Application Insights
6. **Configure network policies** for restricted firewall environments

These enhancements are out of scope for the MVP but recommended for production.
