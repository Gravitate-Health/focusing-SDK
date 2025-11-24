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
   - **Focusing Manager** (Core API): http://localhost:8080/focusing

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
- **Role**: Central orchestration - coordinates between preprocessors and lens providers
- **Discovery**: Automatically detects services using Docker labels

#### FHIR Emulator
- **Container**: `fhir-emulator`
- **Role**: Provides FHIR API endpoints
- **Data Sources**: 
  - `/Patients` folder → mounted as `/data/patients`
  - `/ePIs` folder → mounted as `/data/epis`

### Preprocessor Services

Preprocessors are data transformation components that enrich FHIR resources. They're automatically discovered by the focusing manager through Docker labels

#### MVP2 Preprocessor
- **Container**: `mvp2-preprocessor`
- **Purpose**: Production-grade data processing and enrichment
- **Auto-discovery**: Enabled via labels

### Lens Selector Services

Lens selectors discover and expose FHIR Library components (Lenses). The focusing manager queries these to find lenses.

#### Folder-Based Lens selector
- **Container**: `lens-selector-file`
- **Purpose**: Reads FHIR Library resources from mounted volume. Active development can happen of lenses in this volume
- **Auto-discovery**: Enabled via labels
- **Data Sources**: 
  - `/Lenses` folder → mounted as `/app/lenses`
- **Additional Instances**: you may add additional instances of this service to serve the folder where you are developing lenses.

#### Git-Based Lens Providers
- **Containers**: `lens-selector-pregnancy`, `lens-selector-conditions`
- **Purpose**: Reads FHIR Library resources from mounted volume. Active development can happen of lenses in this volume
- **Auto-discovery**: Enabled via labels
- **Purpose**: Discover and serve lenses from GitHub repositories
- **Additional Instances**: you may add additional instances of this service to serve the git repository where you are developing lenses. Remember to change `CACHE_TTL_MINUTES=0` to disable caching for active development.

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

1. **copy another git lens selector** from `docker-compose.yml`:
   ```yaml
   git-lens-provider-example:
     container_name: git-lens-provider-example
     image: ghcr.io/gravitate-health/lens-selector-git:latest
     # ... rest of configuration
   ```

2. **Update the configuration** with your repository:
   ```yaml
   environment:
     GIT_REPO_URL: https://github.com/your-org/your-lens-repo.git
     GIT_BRANCH: main
     CACHE_TTL_MINUTES: 0 # disabled cache for active development
    networks:
      - focusing-network
    restart: unless-stopped
    labels:
       eu.gravitate-health.fosps.focusing: True
   ```

3. **Start the new service**:
   ```bash
   docker-compose up -d git-lens-provider-example
   ```

4. **Verify discovery**:
   - Check Swagger UI: http://localhost:8888
   - Check Inspector: http://localhost:4200
   - Check focusing-manager: http://localhost:8080/focusing/lenses
   - View logs: `docker-compose logs -f git-lens-provider-example`

### Adding a New Preprocessor

To add a custom preprocessor:

1. **Add service definition** in `docker-compose.yml`:
   ```yaml
   my-custom-preprocessor:
     container_name: my-custom-preprocessor
     image: my-org/my-preprocessor:latest
     environment:
       # Add your custom environment variables
     networks:
       - focusing-network
     depends_on:
       - focusing-manager
     labels:
      eu.gravitate-health.fosps.preprocessing: True
   ```

2. **Build and run**:
   ```bash
   docker-compose up -d my-custom-preprocessor
   ```

## 🔍 Service Discovery Mechanism

The Gravitate-Health focusing manager automatically discovers services using **Docker labels**. Each service is labeled to identify its type and capabilities:
 - **Preprocessors:**: `eu.gravitate-health.fosps.preprocessing: True`
 - **Lens selectors:**: `eu.gravitate-health.fosps.focusing: True`


## 📊 Typical Development Workflow

### 1. Developing a New Lens

 1. Install FHIR lens bundler tool.

 2. Create lens

 3. `lens-selector-file` will automatically verify lens and bundle with valid sibling js file 

 4. Develop lens (either json or js), update ePIs with needed manual annotations

 4. Test focusing.
```bash
# 1. Create a FHIR Library resource and add to Lenses/ folder
cp my-lens.json Lenses/


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

**Verify internal connectivity:**
For example check that `focusing-manager` can access `fhir.emulator`
```bash
docker-compose exec focusing-manager curl http://fhir-emulator:3000/
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
docker inspect <container-name> | grep gravitate-health
```

**Check discovery logs:**
```bash
docker-compose logs -f focusing-manager
```

**Re-check labels:**
- Ensure label prefix is exactly: `eu.gravitate-health.fosps`
- Ensure service is running and healthy

## 📚 API Examples

### Discover Available Lenses
```bash
curl http://localhost:8080/focusing/lenses
```

### Get Preprocessor Status
```bash
curl http://localhost:8080/focusing/preprocessors
```

### Submit Data for Focusing
```bash
curl -X POST http://localhost:8080/focusing/focus \
  -H "Content-Type: application/fhir+json" \
  -d @patient-bundle.json
```

See the **Swagger UI** (http://localhost:8888) for complete API documentation.

## 🚢 Production Considerations

This setup is designed for **development only**. For production deployments:

- Use Kubernetes instead of Docker Compose
- Implement proper authentication and authorization
- Use environment-specific configurations
- Set up persistent storage volumes
- Implement monitoring and logging infrastructure
- Use proper secret management for credentials
- Set up CI/CD pipelines for image building
- Do not use `fhir-emulator`or `nginx` these components are specific for this SDK.

## 📖 Additional Resources

- [FHIR Specification](https://www.hl7.org/fhir/)
- [Gravitate-Health Documentation](https://www.gravitatehealth.eu/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [OpenAPI/Swagger Documentation](https://spec.openapis.org/)

## 🤝 Contributing

To contribute to the Focusing SDK:

1. Create a fork
2. Test thoroughly in this development environment
3. Document your changes
4. Submit a pull request with a clear description

## 📝 License

   Copyright 2025 Universidad Politécnica de Madrid

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

## 📧 Support & Questions

For questions about:
- **Lenses**: See the Lenses folder documentation
- **Preprocessors**: See individual preprocessor documentation
- **FHIR Data**: See Patients and ePIs folder guidelines
- **Infrastructure**: See Docker Compose configuration comments

---

**Last Updated**: November 2025
**Gravitate-Health FOSPS SDK v1.0**
