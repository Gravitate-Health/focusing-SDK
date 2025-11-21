# Gravitate-Health Focusing SDK - Developer Environment

Welcome to the **Gravitate-Health Federated Open Source Platform and Services (FOSPS)** developer environment! This repository contains a scaled-down local deployment designed to help developers build and test **Lenses** and **Preprocessors** - the pluggable components of the Gravitate-Health focusing mechanism.

## 📋 Overview

This development environment provides a complete, containerized setup of the Gravitate-Health FOSPS with:

- **Focusing Manager**: Central orchestration service
- **FHIR Emulator**: Local FHIR data server with patient and product information
- **Preprocessors**: Data transformation and enrichment components
- **Lens Providers**: Services that discover and expose FHIR Library Lenses
- **Developer Tools**: Web-based inspection tools and API documentation

## 🚀 Quick Start

### Prerequisites

- **Docker** and **Docker Compose** installed
- At least 8GB RAM available
- Port availability: 4200, 8080, 8888 (configurable in `docker-compose.yml`)

### Starting the Environment

1. **Clone or navigate to this repository:**
   ```bash
   cd focusing-sdk
   ```

2. **Start all services:**
   ```bash
   docker-compose up -d
   ```

3. **Verify services are running:**
   ```bash
   docker-compose ps
   ```

4. **Access the web interfaces:**
   - **Focusing Inspector** (Main testing tool): http://localhost:4200
   - **Swagger UI** (API documentation): http://localhost:8888
   - **FHIR Emulator** (FHIR data): http://localhost:3000/fhir
   - **Focusing Manager** (Core API): http://localhost:8080

### Stopping the Environment

```bash
docker-compose down
```

To also remove all data volumes:
```bash
docker-compose down -v
```

## 📁 Repository Structure

```
focusing-sdk/
├── docker-compose.yml      # Main service orchestration configuration
├── README.md               # This file
├── Patients/               # FHIR Patient Summaries
│   └── (Add FHIR IPS bundles here)
├── ePIs/                   # Electronic Product Information
│   └── (Add FHIR ePI bundles here)
└── Lenses/                 # Gravitate-Health FHIR Library Lenses
    └── (Add FHIR Library components here)
```

### Folder Descriptions

#### `Patients/`
Contains FHIR International Patient Summaries (IPS) examples from the Gravitate-Health FHIR implementation guide. These are used by the FHIR emulator to provide patient data to the focusing process.

**Format**: FHIR Bundle resources
**Status**: Ready to populate with IPS examples

#### `ePIs/`
Contains FHIR bundles representing electronic Product Information (ePI), gathered from the Gravitate-Health FHIR implementation guide. These are used alongside patient data for the focusing process.

**Format**: FHIR Bundle resources
**Status**: Ready to populate with ePI bundles

#### `Lenses/`
Contains Gravitate-Health Lenses, which are extensions of the FHIR Library profile. These define how patient information should be focused/highlighted in relation to product information.

**Format**: FHIR Library resources
**Status**: Initially empty; folder-based lens provider will discover lenses here

## 🏗️ Services Architecture

### Core Services

#### Focusing Manager
- **Container**: `focusing-manager`
- **Port**: 8080
- **Role**: Central orchestration - coordinates between preprocessors and lens providers
- **Discovery**: Automatically detects services using Docker labels
- **API**: OpenAPI/Swagger available at `/api/openapi.json`

#### FHIR Emulator
- **Container**: `fhir-emulator`
- **Port**: 3000
- **Role**: Provides FHIR API endpoints
- **Data Sources**: 
  - `/Patients` folder → mounted as `/data/patients`
  - `/ePIs` folder → mounted as `/data/epis`
- **Endpoints**:
  - `GET /fhir/metadata` - FHIR capability statement
  - `GET /fhir/Patient/{id}` - Retrieve patient
  - `GET /fhir/Bundle` - Access ePIs

### Preprocessor Services

Preprocessors are data transformation components that enrich FHIR resources. They're automatically discovered by the focusing manager through Docker labels.

#### Manual Preprocessor
- **Container**: `manual-preprocessor`
- **Purpose**: Accept manual input for testing and validation
- **Auto-discovery**: Enabled via labels
- **Capabilities**: `input_validation`, `manual_entry`

#### MVP2 Preprocessor
- **Container**: `mvp2-preprocessor`
- **Purpose**: Production-grade data processing and enrichment
- **Auto-discovery**: Enabled via labels
- **Capabilities**: `advanced_validation`, `data_enrichment`, `performance_optimization`

### Lens Provider Services

Lens providers discover and expose FHIR Library components (Lenses). The focusing manager queries these to find relevant lenses for a given context.

#### Folder-Based Lens Provider
- **Container**: `folder-lens-provider`
- **Source**: Local `Lenses/` folder
- **Status**: Structure ready (implementation in development)
- **Auto-discovery**: Enabled via labels
- **Capability**: Reads FHIR Library resources from mounted volume

#### Git-Based Lens Providers
- **Template**: Provided but commented out in `docker-compose.yml`
- **Purpose**: Discover and serve lenses from GitHub repositories
- **Status**: Not yet implemented
- **How to enable**: See "Adding Custom Lens Providers" section below

### Developer Tools

#### Focusing Inspector
- **Container**: `focusing-inspector`
- **Port**: 4200
- **Purpose**: Web UI for testing and validating the focusing process
- **Features**: 
  - Interactive lens discovery
  - Test focusing with sample data
  - View transformation results

#### Swagger UI
- **Container**: `swagger-ui`
- **Port**: 8888
- **Purpose**: OpenAPI/Swagger documentation and testing
- **API**: Documents the Focusing Manager's REST API
- **Features**: Try-it-out testing directly in browser

## 🛠️ Managing Services

### View Service Status
```bash
docker-compose ps
```

### View Service Logs
```bash
# View logs from all services
docker-compose logs -f

# View logs from specific service
docker-compose logs -f focusing-manager

# View last 100 lines
docker-compose logs --tail=100 focusing-manager

# View logs with timestamps
docker-compose logs -f --timestamps fhir-emulator
```

### Start/Stop Individual Services
```bash
# Start a specific service
docker-compose up -d manual-preprocessor

# Stop a specific service
docker-compose stop focusing-inspector

# Restart a service
docker-compose restart mvp2-preprocessor
```

### Rebuild Images
```bash
# Rebuild all images
docker-compose build --no-cache

# Rebuild specific service
docker-compose build --no-cache focusing-manager
```

### Reset Everything
```bash
# Stop all services and remove containers
docker-compose down

# Also remove volumes (persistent data)
docker-compose down -v

# Also remove images
docker-compose down -v --rmi all
```

## 🔧 Customization & Extension

### Adding a New Lens Provider (Git-based)

To add a lens provider that discovers lenses from a GitHub repository:

1. **Uncomment the template** in `docker-compose.yml`:
   ```yaml
   git-lens-provider-example:
     container_name: git-lens-provider-example
     image: gravitatehealth/git-lens-provider:latest
     # ... rest of configuration
   ```

2. **Update the configuration** with your repository:
   ```yaml
   environment:
     GIT_REPOSITORY_URL: https://github.com/your-org/your-lens-repo.git
     GIT_BRANCH: main
     # ...
   labels:
     com.gravitatehealth.lens-provider.name: my-lens-provider
     com.gravitatehealth.lens-provider.git-url: "https://github.com/your-org/your-lens-repo.git"
   ```

3. **Start the new service**:
   ```bash
   docker-compose up -d git-lens-provider-example
   ```

4. **Verify discovery**:
   - Check Swagger UI: http://localhost:8888
   - Check Inspector: http://localhost:4200
   - View logs: `docker-compose logs -f git-lens-provider-example`

### Adding a New Preprocessor

To add a custom preprocessor:

1. **Add service definition** in `docker-compose.yml`:
   ```yaml
   my-custom-preprocessor:
     container_name: my-custom-preprocessor
     image: my-org/my-preprocessor:latest
     environment:
       FOCUSING_MANAGER_URL: http://focusing-manager:8080
       FHIR_SERVER_URL: http://fhir-emulator:3000/fhir
       # Add your custom environment variables
     networks:
       - focusing-network
     depends_on:
       - focusing-manager
       - fhir-emulator
     labels:
       com.gravitatehealth.type: preprocessor
       com.gravitatehealth.preprocessor.name: my-preprocessor
       com.gravitatehealth.preprocessor.version: "1.0"
       com.gravitatehealth.preprocessor.description: "My custom preprocessor"
       com.gravitatehealth.preprocessor.capabilities: "capability1,capability2"
   ```

2. **Build and run**:
   ```bash
   docker-compose up -d my-custom-preprocessor
   ```

### Modifying Environment Variables

Edit `docker-compose.yml` to change service configuration:

```yaml
focusing-manager:
  environment:
    LOG_LEVEL: DEBUG  # Change from INFO to DEBUG for verbose logging
    DISCOVERY_INTERVAL: 60s  # Increase discovery polling interval
```

Then restart the service:
```bash
docker-compose up -d focusing-manager
```

### Mounting Custom Configuration Files

Add volumes to services:

```yaml
focusing-manager:
  volumes:
    - ./config:/app/config:ro  # Mount custom config directory
    - /var/run/docker.sock:/var/run/docker.sock
```

## 🔍 Service Discovery Mechanism

The Gravitate-Health focusing manager automatically discovers services using **Docker labels**. Each service is labeled to identify its type and capabilities:

### Label Scheme

**Preprocessors:**
```
com.gravitatehealth.type: preprocessor
com.gravitatehealth.preprocessor.name: <unique-name>
com.gravitatehealth.preprocessor.version: <version>
com.gravitatehealth.preprocessor.description: <human-readable-description>
com.gravitatehealth.preprocessor.capabilities: <comma-separated-capabilities>
```

**Lens Providers:**
```
com.gravitatehealth.type: lens-provider
com.gravitatehealth.lens-provider.name: <unique-name>
com.gravitatehealth.lens-provider.version: <version>
com.gravitatehealth.lens-provider.description: <human-readable-description>
com.gravitatehealth.lens-provider.source: <source-type: file-system|git|etc>
```

The focusing manager queries Docker daemon for labeled containers and registers their capabilities automatically, typically every 30 seconds.

## 📊 Typical Development Workflow

### 1. Developing a New Lens

```bash
# 1. Create a FHIR Library resource and add to Lenses/ folder
cp my-lens.json Lenses/

# 2. Restart the folder lens provider
docker-compose restart folder-lens-provider

# 3. Test in Focusing Inspector
# Navigate to http://localhost:4200
# The lens should now be discoverable
```

### 2. Developing a Custom Preprocessor

```bash
# 1. Build your preprocessor with the required Docker labels
# Make sure your Dockerfile tags the image appropriately

# 2. Add service definition to docker-compose.yml

# 3. Start your service
docker-compose up -d my-preprocessor

# 4. Monitor logs
docker-compose logs -f my-preprocessor

# 5. Test via Swagger UI or Inspector
# http://localhost:8888 or http://localhost:4200
```

### 3. Testing End-to-End

```bash
# 1. Add test patient to Patients/
cp test-patient.json Patients/

# 2. Add test ePI to ePIs/
cp test-epi.json ePIs/

# 3. Start the environment
docker-compose up -d

# 4. Use Focusing Inspector to:
#    - Load the patient
#    - Apply preprocessors
#    - Discover relevant lenses
#    - View focused results

# 5. Review logs for debugging
docker-compose logs -f
```

## 🐛 Troubleshooting

### Services Won't Start

**Check Docker daemon:**
```bash
docker ps
docker-compose ps
```

**View detailed logs:**
```bash
docker-compose logs -f <service-name>
```

**Common issues:**
- Port conflicts: Change ports in `docker-compose.yml`
- Insufficient resources: Ensure 8GB+ RAM available
- Volume mount issues: Check file permissions in `Patients/`, `ePIs/`, `Lenses/`

### Services Can't Communicate

**Check network:**
```bash
docker network ls
docker network inspect focusing-network
```

**Verify connectivity:**
```bash
docker-compose exec focusing-manager curl http://fhir-emulator:3000/fhir/metadata
```

### Data Not Appearing

**Check volume mounts:**
```bash
docker-compose exec fhir-emulator ls -la /data/patients
docker-compose exec fhir-emulator ls -la /data/epis
```

**Verify FHIR format:**
- Ensure files in `Patients/` and `ePIs/` are valid FHIR Bundles
- Check logs: `docker-compose logs fhir-emulator`

### Services Not Auto-Discovering

**Verify labels are set:**
```bash
docker inspect <container-name> | grep gravitatehealth
```

**Check discovery logs:**
```bash
docker-compose logs -f focusing-manager
```

**Re-check labels:**
- Ensure label prefix is exactly: `com.gravitatehealth.type`
- Ensure service is running and healthy

## 📚 API Examples

### Discover Available Lenses
```bash
curl http://localhost:8080/api/lenses
```

### Get Preprocessor Status
```bash
curl http://localhost:8080/api/preprocessors
```

### Submit Data for Focusing
```bash
curl -X POST http://localhost:8080/api/focus \
  -H "Content-Type: application/fhir+json" \
  -d @patient-bundle.json
```

### View FHIR Patient
```bash
curl http://localhost:3000/fhir/Patient/example-patient
```

See the **Swagger UI** (http://localhost:8888) for complete API documentation.

## 🚢 Production Considerations

This setup is designed for **development only**. For production deployments:

- Use Docker Stack or Kubernetes instead of Docker Compose
- Implement proper authentication and authorization
- Use environment-specific configurations
- Set up persistent storage volumes
- Implement monitoring and logging infrastructure
- Use proper secret management for credentials
- Set up CI/CD pipelines for image building

## 📖 Additional Resources

- [FHIR Specification](https://www.hl7.org/fhir/)
- [Gravitate-Health Documentation](https://www.gravitatehealth.eu/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [OpenAPI/Swagger Documentation](https://spec.openapis.org/)

## 🤝 Contributing

To contribute to the Focusing SDK:

1. Create a new branch for your feature
2. Test thoroughly in this development environment
3. Document your changes
4. Submit a pull request with a clear description

## 📝 License

[Add appropriate license information]

## 📧 Support & Questions

For questions about:
- **Lenses**: See the Lenses folder documentation
- **Preprocessors**: See individual preprocessor documentation
- **FHIR Data**: See Patients and ePIs folder guidelines
- **Infrastructure**: See Docker Compose configuration comments

---

**Last Updated**: November 2025
**Gravitate-Health FOSPS SDK v1.0**
