# Quick Reference - Gravitate-Health Focusing SDK

## 🚀 Getting Started (30 seconds)

```bash
cd focusing-sdk
docker-compose up -d
open http://localhost:4200  # or use your browser
```

## 🌐 Service URLs

| Service | URL | Purpose |
|---------|-----|---------|
| **Inspector** | http://localhost:4200 | Interactive testing UI |
| **Swagger API** | http://localhost:8888 | API documentation |
| **Focusing Manager** | http://localhost:8080 | Core API |
| **FHIR Server** | http://localhost:3000/fhir | Data endpoints |

## 📦 Key Services in docker-compose.yml

```
focusing-manager          - Central orchestration
fhir-emulator            - Patient & product data
manual-preprocessor      - Manual test input (auto-discovered)
mvp2-preprocessor        - Advanced preprocessing (auto-discovered)
folder-lens-provider     - Local lens discovery (auto-discovered)
git-lens-provider        - GitHub lens discovery (commented, enable as needed)
focusing-inspector       - Web testing UI
swagger-ui              - API documentation
```

## 🛠️ Common Commands

### Management
```bash
docker-compose ps              # View service status
docker-compose up -d           # Start all services
docker-compose down            # Stop services
docker-compose down -v         # Stop + remove data
docker-compose restart <svc>   # Restart specific service
```

### Logs & Debugging
```bash
docker-compose logs -f                    # All logs (follow)
docker-compose logs -f focusing-manager   # Specific service
docker-compose logs --tail=50 fhir-emulator  # Last 50 lines
docker-compose logs <svc> | grep ERROR   # Search logs
```

### Testing
```bash
curl http://localhost:8080/focusing/lenses          # List lenses
curl http://localhost:8080/focusing/preprocessors   # List preprocessors
```

## 📁 Folder Structure

| Folder | Purpose | Files Type |
|--------|---------|-----------|
| `Patients/` | Patient summaries | FHIR IPS Bundles |
| `ePIs/` | Product information | FHIR Bundles |
| `Lenses/` | Clinical logic | FHIR Libraries |

## ➕ Add Custom Components

### Quick Example: Add a Lens Provider

 1. copy conditions-git-lens-selector and paste in docker-compose.yml
 2. Update GIT_REPOSITORY_URL
 3. Run: docker-compose up -d git-lens-selector-example
 4. Verify: curl http://localhost:8080/focusing/lenses


### Quick Example: Add FHIR Data

1. Create/download valid FHIR Bundle JSON
2. Place in Patients/, ePIs/, or Lenses/


## 🔍 Service Discovery Labels

Services auto-discover using Docker labels:

```yaml
# For Preprocessors
labels:
   eu.gravitate-health.fosps.preprocessing: True

# For Lens Providers
labels:
  eu.gravitate-health.fosps.focusing: True
```

## 📝 File Guide

| File | Size | Purpose |
|------|------|---------|
| `README.md` | ~600 lines | Full documentation |
| `QUICKSTART.md` | ~200 lines | 5-min start guide |
| `docker-compose.yml` | ~272 lines | Services config |
| `Patients/README.md` | ~150 lines | Patient data guide |
| `ePIs/README.md` | ~150 lines | Product data guide |
| `Lenses/README.md` | ~300 lines | Lens dev guide |

## 🐛 Troubleshooting Quick Fixes

| Problem | Solution |
|---------|----------|
| Port conflict | Change ports in docker-compose.yml |
| Services won't start | Check logs: `docker-compose logs` |
| Can't access Inspector | Wait 30s and refresh, check `docker-compose ps` |
| Data not loading | Check volumes: `docker inspect <container>` |
| Services not discovered | Check labels: `docker inspect <container> \| grep gravitatehealth` |