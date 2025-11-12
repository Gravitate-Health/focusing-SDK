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
Obtain IPS examples from the [Gravitate-Health FHIR Implementation Guide](https://www.gravitatehealth.eu/deliverables/).

### Option 2: Create Test Data
Use the FHIR specification to create sample IPS bundles with realistic patient data.

### Option 3: Use Tools
Tools like:
- [FHIR Validator](http://hl7.org/fhir/validator/)
- [FHIR Forge](https://fire.appliedis.com/)
- [Postman FHIR collections](https://www.postman.com/gravitatehealth/)

## Validation

Ensure all files are valid FHIR bundles:

```bash
# Inside the FHIR emulator container
docker-compose exec fhir-emulator npm run validate -- /data/patients
```

## Usage

The FHIR emulator automatically discovers and loads all JSON files in this folder at startup:

1. Start the environment:
   ```bash
   docker-compose up -d
   ```

2. Verify patients are loaded:
   ```bash
   curl http://localhost:3000/fhir/Patient
   ```

3. Access specific patient:
   ```bash
   curl http://localhost:3000/fhir/Patient/patient-001
   ```

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
3. Preprocessors enrich patient data
4. Lenses are applied based on patient context

## Next Steps

- [ ] Download or create patient sample data
- [ ] Validate FHIR bundle structure
- [ ] Add files to this folder
- [ ] Restart FHIR emulator: `docker-compose restart fhir-emulator`
- [ ] Test access via http://localhost:3000/fhir/Patient

---

**Documentation**: FHIR International Patient Summary (IPS)
**Profile**: http://hl7.org/fhir/uv/ips/
**Version**: Latest
