# Quick Start Guide - Gravitate-Health Focusing SDK

This guide gets you from zero to running the Focusing SDK in 5 minutes.

## Prerequisites Check

```bash
# Check Docker is installed
docker --version
# Should show: Docker version XX.XX.XX

# Check Docker Compose is installed
docker-compose --version
# Should show: Docker Compose version XX.XX.XX

# Check you have at least 8GB RAM available
# Windows: Task Manager → Performance → Memory
# Mac: Activity Monitor → Memory
# Linux: free -h
```

## 1. Navigate to Repository

```bash
cd path/to/focusing-sdk
```

## 2. Start All Services

```bash
docker-compose up -d
```

This will:
- Pull all Docker images (first run only, ~2-3 minutes)
- Create the focusing network
- Start all 8 services
- Enable auto-discovery of preprocessors and lens providers

## 3. Wait for Services to Start

Check service health:
```bash
docker-compose ps
```

Expected output (all should be "Up"):
```
NAME                    STATUS
focusing-manager        Up (healthy)
fhir-emulator          Up (healthy)
manual-preprocessor     Up
mvp2-preprocessor       Up
folder-lens-provider    Up
focusing-inspector      Up (healthy)
swagger-ui             Up
```

First-time startup typically takes:
- ~30 seconds for basic services
- ~60-90 seconds until all services are healthy

## 4. Access the Web Interfaces

Open your browser to these URLs:

### Main Testing Tool - Focusing Inspector
```
http://localhost:4200
```
- Interactive UI for testing the focusing process
- Load patients, products, apply lenses, see results

### API Documentation - Swagger UI
```
http://localhost:8888
```
- Complete REST API documentation
- Try-it-out interface for testing endpoints
- Request/response examples

### FHIR Data Server
```
http://localhost:3000/fhir/metadata
```
- FHIR capability statement
- Direct API access to patient and product data

### Focusing Manager API
```
http://localhost:8080/api/lenses
```
- List available lenses
- View preprocessor status
- Direct API access

## 5. Verify Everything Works

### Test FHIR Emulator
```bash
curl http://localhost:3000/fhir/metadata
```

### Test Focusing Manager
```bash
curl http://localhost:8080/api/lenses
```

### Test Services are Discoverable
```bash
docker-compose exec focusing-manager curl http://focusing-manager:8080/api/preprocessors
```

## 6. Common First Steps

### Add Test Patient Data
```bash
# Create a sample FHIR IPS bundle
cp sample-patient.json Patients/
docker-compose restart fhir-emulator
```

### Add Product Information
```bash
# Create a sample FHIR bundle with medication
cp sample-epi.json ePIs/
docker-compose restart fhir-emulator
```

### Add a Lens
```bash
# Create a FHIR Library lens
cp my-lens.json Lenses/
docker-compose restart folder-lens-provider
```

## 7. Stop the Environment

```bash
# Stop all services (data persists)
docker-compose down

# Stop and remove all data
docker-compose down -v
```

## 8. View Logs

```bash
# View all logs in real-time
docker-compose logs -f

# View specific service logs
docker-compose logs -f focusing-manager

# View last 100 lines
docker-compose logs --tail=100 fhir-emulator
```

## Troubleshooting

### Port Already in Use
If you get "port already in use" error:
```bash
# Check what's using port 8080
netstat -ano | findstr :8080

# Either stop that service or change ports in docker-compose.yml
```

### Services Won't Start
```bash
# Check Docker daemon is running
docker ps

# View detailed error logs
docker-compose logs

# Try rebuilding
docker-compose build --no-cache
```

### Can't Access Services
```bash
# Verify services are running
docker-compose ps

# Test connectivity
docker-compose exec focusing-manager curl http://fhir-emulator:3000/fhir/metadata
```

## Next Steps

1. **Read the full README**: `README.md` in the root directory
2. **Explore data folders**: `Patients/`, `ePIs/`, `Lenses/`
3. **Review docker-compose.yml** to understand service configuration
4. **Add your own lenses** following `Lenses/README.md`
5. **Develop preprocessors** following the pattern in docker-compose.yml

## Key Concepts

- **Focusing Manager**: Central service that orchestrates everything
- **FHIR Emulator**: Provides FHIR API with your data
- **Preprocessors**: Transform/enrich FHIR data (auto-discovered)
- **Lens Providers**: Discover FHIR Library lenses (auto-discovered)
- **Inspector**: Web UI for interactive testing
- **Service Discovery**: Uses Docker labels for automatic registration

## Getting Help

1. Check the main `README.md`
2. View folder-specific guides: `Patients/README.md`, `ePIs/README.md`, `Lenses/README.md`
3. Check service logs: `docker-compose logs -f <service-name>`
4. Review `docker-compose.yml` comments for configuration details

---

**You're all set! Start with http://localhost:4200 in your browser.**
