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
curl http://localhost:8080/api/lenses          # List lenses
curl http://localhost:8080/api/preprocessors   # List preprocessors
curl http://localhost:3000/fhir/metadata       # FHIR capability
```

## 📁 Folder Structure

| Folder | Purpose | Files Type |
|--------|---------|-----------|
| `Patients/` | Patient summaries | FHIR IPS Bundles |
| `ePIs/` | Product information | FHIR Bundles |
| `Lenses/` | Clinical logic | FHIR Libraries |

## ➕ Add Custom Components

### Quick Example: Add a Lens Provider

```bash
# 1. Uncomment git-lens-provider-example in docker-compose.yml
# 2. Update GIT_REPOSITORY_URL
# 3. Run: docker-compose up -d git-lens-provider-example
# 4. Verify: curl http://localhost:8080/api/lenses
```

### Quick Example: Add FHIR Data

```bash
# 1. Create valid FHIR Bundle JSON
# 2. Place in Patients/, ePIs/, or Lenses/
# 3. Restart provider: docker-compose restart fhir-emulator
# 4. Test: curl http://localhost:3000/fhir/<ResourceType>
```

## 🔍 Service Discovery Labels

Services auto-discover using Docker labels:

```yaml
# For Preprocessors
labels:
  com.gravitatehealth.type: preprocessor
  com.gravitatehealth.preprocessor.name: my-preprocessor
  com.gravitatehealth.preprocessor.capabilities: capability1,capability2

# For Lens Providers
labels:
  com.gravitatehealth.type: lens-provider
  com.gravitatehealth.lens-provider.name: my-lenses
  com.gravitatehealth.lens-provider.source: file-system
```

## 📝 File Guide

| File | Size | Purpose |
|------|------|---------|
| `README.md` | ~600 lines | Full documentation |
| `QUICKSTART.md` | ~200 lines | 5-min start guide |
| `DEVELOPMENT.md` | ~600 lines | Dev guidelines |
| `docker-compose.yml` | ~272 lines | Services config |
| `Patients/README.md` | ~150 lines | Patient data guide |
| `ePIs/README.md` | ~150 lines | Product data guide |
| `Lenses/README.md` | ~300 lines | Lens dev guide |

## ⚙️ Configuration

### Environment Variables
Create `.env` from `.env.example`:
```bash
cp .env.example .env
# Edit as needed
docker-compose --env-file .env up -d
```

### Common Settings
```bash
LOG_LEVEL=INFO                              # DEBUG, INFO, WARN, ERROR
FOCUSING_MANAGER_DISCOVERY_INTERVAL=30s    # Service scan frequency
INSPECTOR_FOCUSING_MANAGER_URL=http://localhost:8080
```

## 🐛 Troubleshooting Quick Fixes

| Problem | Solution |
|---------|----------|
| Port conflict | Change ports in docker-compose.yml |
| Services won't start | Check logs: `docker-compose logs` |
| Can't access Inspector | Wait 30s and refresh, check `docker-compose ps` |
| Data not loading | Check volumes: `docker inspect <container>` |
| Services not discovered | Check labels: `docker inspect <container> \| grep gravitatehealth` |

## 🚀 Development Workflow

```
1. Start environment
   docker-compose up -d

2. Add/modify FHIR data
   - Copy files to Patients/, ePIs/, Lenses/

3. Restart relevant service
   docker-compose restart fhir-emulator
   docker-compose restart folder-lens-provider

4. Test in Inspector
   http://localhost:4200

5. Review logs for errors
   docker-compose logs -f
```

## 📊 API Quick Reference

### List Resources
```bash
curl http://localhost:8080/api/lenses
curl http://localhost:8080/api/preprocessors
curl http://localhost:3000/fhir/Patient
curl http://localhost:3000/fhir/Medication
```

### Submit for Focusing
```bash
curl -X POST http://localhost:8080/api/focus \
  -H "Content-Type: application/fhir+json" \
  -d @data.json
```

### Full API Documentation
→ http://localhost:8888

## 🔐 Security Notes

- ⚠️ Development environment only
- ⚠️ No authentication configured
- ⚠️ Don't expose to internet
- ⚠️ Use `.env` for secrets (in `.gitignore`)

## 📚 Documentation Map

```
You are here
    ↓
├─ QUICKSTART.md ........................... Start here (5 min)
├─ README.md .............................. Full documentation
├─ DEVELOPMENT.md ......................... Dev guidelines
└─ Folder Guides:
   ├─ Patients/README.md .................. Patient data
   ├─ ePIs/README.md ..................... Product data
   └─ Lenses/README.md ................... Lens development
```

## ✅ Pre-flight Checklist

- [ ] Docker installed (`docker --version`)
- [ ] Docker Compose installed (`docker-compose --version`)
- [ ] 8GB+ RAM available
- [ ] Ports 3000, 4200, 8080, 8888 available
- [ ] Internet for initial image pull

## 🎯 Next Steps

1. **5 min**: Read QUICKSTART.md
2. **10 min**: Start services
3. **15 min**: Access Inspector at http://localhost:4200
4. **30 min**: Add sample data to folders
5. **60 min**: Read full README.md
6. **2 hrs**: Read DEVELOPMENT.md and add custom components

---

**Last Updated**: November 2025
**Keep this handy during development!** 📌
