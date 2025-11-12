# Lenses Folder

This folder contains **Gravitate-Health Lenses**, which are FHIR Library components that define how patient information should be focused/highlighted in relation to product information.

## Overview

Lenses are the core of the Gravitate-Health focusing mechanism. They encapsulate domain-specific logic to:
- Transform patient data for specific contexts
- Highlight relevant product information based on patient profile
- Apply clinical reasoning to determine what information is important
- Present information in a meaningful way to end users

## What are Lenses?

A Lens is a FHIR Library resource (an extension of the Gravitate-Health Lens profile) that contains:
- **Logic**: CQL (Clinical Quality Language) or FHIRPath expressions
- **Metadata**: Title, description, version, clinical domain
- **Purpose**: Target use case (e.g., "Drug-Disease Interaction Checking")
- **Inputs**: Required FHIR resources (e.g., Patient, Medication)
- **Outputs**: Focused/highlighted information relevant to the use case

## File Organization

Recommended structure:
```
Lenses/
├── drug-disease-interactions.json  # Lens for drug-disease interactions
├── contraindications.json           # Lens for contraindications
├── drug-drug-interactions.json      # Lens for drug-drug interactions
├── adverse-events.json              # Lens for adverse event identification
├── example-lens.json                # Example Lens template
└── README.md                        # This file
```

## Format

Each lens should be a valid FHIR Library resource:

```json
{
  "resourceType": "Library",
  "id": "drug-disease-interactions",
  "name": "DrugDiseaseInteractionsLens",
  "title": "Drug-Disease Interaction Lens",
  "description": "Focuses on identifying contraindications and drug-disease interactions",
  "version": "1.0.0",
  "status": "active",
  "type": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/library-type",
      "code": "logic-library"
    }]
  },
  "url": "http://gravitate-health.eu/Library/drug-disease-interactions",
  "topic": [{
    "text": "Drug-Disease Interactions"
  }],
  "author": [{
    "name": "Gravitate-Health"
  }],
  "content": [{
    "contentType": "text/cql",
    "data": "base64-encoded-cql-logic"
  }],
  "relatedArtifact": [{
    "type": "depends-on",
    "resource": "http://hl7.org/fhir/Library/FHIRHelpers"
  }]
}
```

## Creating a Lens

### Step 1: Define Purpose
What clinical problem does your lens solve?
- Drug-disease interactions
- Adverse event identification
- Dosage recommendations
- Patient-specific highlighting

### Step 2: Define Logic
Write CQL (Clinical Quality Language) or FHIRPath to:
- Query patient data
- Retrieve product information
- Apply clinical rules
- Generate results

### Step 3: Package as FHIR Library
Create a FHIR Library resource with:
- Clear metadata (title, description, version)
- Embedded logic (CQL, FHIRPath)
- Input/output definitions
- Versioning information

### Step 4: Add to Folder
Place your lens JSON file in this folder with a descriptive name.

### Step 5: Restart Provider
Restart the folder-based lens provider:
```bash
docker-compose restart folder-lens-provider
```

### Step 6: Verify Discovery
Check that the lens is discoverable:
```bash
curl http://localhost:8080/api/lenses
```

## Example Lens Template

```json
{
  "resourceType": "Library",
  "id": "my-custom-lens",
  "name": "MyCustomLens",
  "title": "My Custom Lens",
  "description": "A custom lens for specific clinical reasoning",
  "version": "1.0.0",
  "status": "active",
  "type": {
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/library-type",
      "code": "logic-library"
    }]
  },
  "url": "http://gravitate-health.eu/Library/my-custom-lens",
  "publisher": "Your Organization",
  "topic": [{
    "text": "Your Topic"
  }],
  "author": [{
    "name": "Your Name"
  }],
  "content": [{
    "contentType": "text/cql",
    "data": "/* CQL Logic Here */"
  }],
  "relatedArtifact": [
    {
      "type": "depends-on",
      "resource": "http://hl7.org/fhir/Library/FHIRHelpers"
    }
  ],
  "parameter": [
    {
      "name": "Patient",
      "use": "in",
      "type": "Patient"
    },
    {
      "name": "Medication",
      "use": "in",
      "type": "Medication"
    },
    {
      "name": "FocusedInformation",
      "use": "out",
      "type": "Bundle"
    }
  ]
}
```

## Validation

Validate your lens FHIR syntax:

```bash
# Inside the FHIR emulator container
docker-compose exec fhir-emulator npm run validate -- Lenses/my-lens.json
```

Validate CQL logic:
- Use the [CQL Testing Framework](http://cql.hl7.org/)
- Test against sample patient data

## Integration with Focusing

### Discovery Process
1. Folder-based Lens Provider scans `Lenses/` folder
2. Reads FHIR Library resources
3. Reports lenses to Focusing Manager
4. Focusing Manager indexes lens capabilities

### Execution Process
1. Patient data arrives at Focusing Manager
2. Manager queries available lenses based on context
3. Selected lenses are applied to patient/product data
4. Results are focused/highlighted per lens logic
5. Focused information is returned to Inspector

### Access via APIs
```bash
# List all lenses
curl http://localhost:8080/api/lenses

# Get specific lens details
curl http://localhost:8080/api/lenses/drug-disease-interactions

# Apply lens to data
curl -X POST http://localhost:8080/api/focus \
  -H "Content-Type: application/fhir+json" \
  -d @request.json
```

## Testing Lenses

### Manual Testing via Inspector
1. Navigate to http://localhost:4200
2. Load sample patient data
3. Load sample product information
4. Select lenses to apply
5. Review focused results

### Automated Testing
```bash
# Example using curl and test data
curl -X POST http://localhost:8080/api/focus \
  -H "Content-Type: application/fhir+json" \
  -H "X-Lens-ID: drug-disease-interactions" \
  -d @test-patient.json
```

### Debugging Lens Execution
Enable detailed logging:
```bash
# Update environment in docker-compose.yml
docker-compose up -d focusing-manager
docker-compose logs -f focusing-manager
```

## Contributing Lenses

### Share Your Lens
1. Create a FHIR-compliant Library resource
2. Document purpose and logic
3. Include example use cases
4. Submit to Gravitate-Health repository

### Lens Best Practices
- Use clear, descriptive names
- Document clinical rationale
- Include comprehensive metadata
- Version your lenses semantically
- Test thoroughly before deployment
- Provide example inputs/outputs
- Consider performance implications
- Handle edge cases gracefully

## Advanced: Git-Based Lenses

For lenses stored in GitHub repositories, the Git-based Lens Provider:
1. Clones the repository
2. Discovers FHIR Library resources
3. Reports them to Focusing Manager
4. Updates on schedule or webhook

To enable (uncomment in docker-compose.yml):
```yaml
git-lens-provider-example:
  image: gravitatehealth/git-lens-provider:latest
  environment:
    GIT_REPOSITORY_URL: https://github.com/gravitate-health/lens-repo.git
  labels:
    com.gravitatehealth.type: lens-provider
    # ... other labels
```

## Resources

### FHIR Documentation
- [FHIR Library Resource](http://hl7.org/fhir/library.html)
- [FHIR Clinical Reasoning Module](http://hl7.org/fhir/clinicalreasoning-module.html)

### CQL Resources
- [CQL Specification](https://cql.hl7.org/)
- [CQL Introduction](https://www.hl7.org/implement/standards/CQL.cfm)
- [CQL Testing Guide](https://cql.hl7.org/)

### Gravitate-Health
- [Lens Profile](https://www.gravitatehealth.eu/deliverables/)
- [Implementation Guide](https://www.gravitatehealth.eu/deliverables/)

## Next Steps

- [ ] Study existing Gravitate-Health lenses
- [ ] Understand CQL basics
- [ ] Design your first lens
- [ ] Create FHIR Library resource
- [ ] Add to this folder
- [ ] Test with sample data
- [ ] Contribute to Gravitate-Health

---

**Gravitate-Health Lenses**: Domain-specific FHIR Library components
**Status**: Initial setup ready for lens development
**Last Updated**: November 2025
