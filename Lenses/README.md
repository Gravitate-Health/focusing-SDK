# Lenses Folder

This folder contains **Gravitate-Health Lenses**, which are FHIR Library components that define how patient information should be focused/highlighted in relation to product information.

## Overview

Lenses are the core of the Gravitate-Health focusing mechanism. They encapsulate domain-specific logic to:

- Transform patient data for specific contexts
- Highlight relevant product information based on patient profile
- Apply clinical reasoning to determine what information is important
- Present information in a meaningful way to end users

## File Organization

Recommended structure:

```sh
Lenses/
├── drug-disease-interactions.json  # Lens for drug-disease interactions
├── contraindications.json           # Lens for contraindications
├── drug-drug-interactions.json      # Lens for drug-drug interactions
├── adverse-events.json              # Lens for adverse event identification
├── example-lens.json                # Example complete Lens
├── Lens-in-Development              # Folder to contain a lens which is being activelly developed
|  └── mylens.json                   # Lens' metadata (content may be left empty, selector will complete with the code below)
|  └── mylens.js                     # Lens' code, the selector will automatically bundle the content of this file with the metadata above
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

Write Java Script function to:

- Query patient data
- Retrieve product information
- Apply clinical rules
- Generate results

### Step 3: Package as FHIR Library

Create a FHIR Library resource with:

- Clear metadata (title, description, version)
- Embedded logic (JS)
- Versioning information

### Step 4: Add to Folder

Place your lens JSON file in this folder with a descriptive name.

### Step 5: Verify Discovery

Check that the lens is discoverable:

```bash
curl http://localhost:8080/focusing/lenses
```

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

## Resources

### FHIR Documentation

- [FHIR Library Resource](http://hl7.org/fhir/library.html)
- [FHIR Clinical Reasoning Module](http://hl7.org/fhir/clinicalreasoning-module.html)

### Gravitate-Health

- [Lens Profile](https://build.fhir.org/ig/hl7-eu/gravitate-health/StructureDefinition-lens.html)
- [Implementation Guide](https://build.fhir.org/ig/hl7-eu/gravitate-health/)

---

**Gravitate-Health Lenses**: Domain-specific FHIR Library components
**Status**: Initial setup ready for lens development
**Last Updated**: November 2025
