# Patients Folder

This folder contains **FHIR International Patient Summaries (IPS)** examples from the Gravitate-Health FHIR implementation guide.

## Overview

Patient data is used by the FHIR emulator to provide realistic patient information to the focusing process. These are FHIR Bundle resources that contain patient demographics, clinical observations, and relevant medical history.

## Format

Each file should be a valid FHIR Bundle containing:
- Patient resource
- Observation resources (clinical data)
- MedicationStatement resources (medications)
- Condition resources (diagnoses)
- Other relevant FHIR resources

## File Organization

Recommended structure:
```
Patients/
├── patient-001.json         # IPS Bundle for patient 1
├── patient-002.json         # IPS Bundle for patient 2
├── example-ips-bundle.json  # Example IPS Bundle template
└── README.md                # This file
```

## Adding Patient Data

### Option 1: From Gravitate-Health Samples
Obtain IPS examples from the [Gravitate-Health FHIR Implementation Guide](https://build.fhir.org/ig/hl7-eu/gravitate-health/).

### Option 2: Create Test Data
Use the FHIR specification to create sample IPS bundles with realistic patient data.

### Option 3: Use Tools
Tools like:
- [FHIR Validator](http://hl7.org/fhir/validator/)
- [FHIR Forge](https://fire.appliedis.com/)

## Usage

The FHIR emulator automatically discovers and dynamically loads all JSON files in this folder at runtime.


## IPS Bundle Structure Example

```json
{
  "resourceType": "Bundle",
  "type": "document",
  "entry": [
    {
      "resource": {
        "resourceType": "Composition",
        "type": {
          "coding": [{
            "system": "http://loinc.org",
            "code": "60591-5",
            "display": "Patient Summary Document"
          }]
        }
      }
    },
    {
      "resource": {
        "resourceType": "Patient",
        "id": "patient-001",
        "name": [{
          "given": ["John"],
          "family": "Doe"
        }],
        "birthDate": "1980-01-01"
      }
    }
  ]
}
```

## Integration with Focusing

1. Patient data flows from this folder → FHIR Emulator
2. Focusing Manager queries patient data when processing
3. Preprocessors enrich ePIs
4. Lenses are applied based on patient context

---

**Documentation**: FHIR International Patient Summary (IPS)
**Profile**: http://hl7.org/fhir/uv/ips/
**Version**: Latest
