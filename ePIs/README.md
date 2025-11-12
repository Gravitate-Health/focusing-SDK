# ePIs (electronic Product Information) Folder

This folder contains **FHIR bundles representing electronic Product Information (ePI)**, gathered from the Gravitate-Health FHIR implementation guide.

## Overview

Product information is used alongside patient data in the focusing process. ePIs contain standardized pharmaceutical product information including:
- Drug identification and classification
- Contraindications and warnings
- Dosage information
- Side effects and interactions
- Active ingredients

## Format

Each file should be a valid FHIR Bundle containing:
- Medication resource (product definition)
- MedicationKnowledge resource (clinical information)
- Substance resources (active ingredients)
- Other relevant product resources

## File Organization

Recommended structure:
```
ePIs/
├── aspirin-500mg.json              # ePI for Aspirin 500mg
├── metformin-850mg.json            # ePI for Metformin 850mg
├── example-epi-bundle.json         # Example ePI Bundle template
└── README.md                        # This file
```

## Adding ePI Data

### Option 1: From Gravitate-Health Samples
Obtain ePI examples from the [Gravitate-Health FHIR Implementation Guide](https://www.gravitatehealth.eu/deliverables/).

### Option 2: Europeana Medicines API
The [Europeana Medicines Portal](https://www.ema.europa.eu/en/medicines) provides structured product information.

### Option 3: Create from SmPC
Convert Summary of Product Characteristics (SmPC) documents to FHIR format.

## Validation

Ensure all files are valid FHIR bundles:

```bash
# Inside the FHIR emulator container
docker-compose exec fhir-emulator npm run validate -- /data/epis
```

## Usage

The FHIR emulator automatically discovers and loads all JSON files in this folder at startup:

1. Start the environment:
   ```bash
   docker-compose up -d
   ```

2. Verify ePIs are loaded:
   ```bash
   curl http://localhost:3000/fhir/Medication
   ```

3. Access specific ePI:
   ```bash
   curl http://localhost:3000/fhir/Medication/aspirin-500mg
   ```

## ePI Bundle Structure Example

```json
{
  "resourceType": "Bundle",
  "type": "collection",
  "entry": [
    {
      "resource": {
        "resourceType": "Medication",
        "id": "aspirin-500mg",
        "code": {
          "coding": [{
            "system": "http://snomed.info/sct",
            "code": "319768000",
            "display": "Aspirin 500 mg oral tablet"
          }]
        },
        "manufacturer": [{
          "display": "Pharmaceutical Company"
        }]
      }
    },
    {
      "resource": {
        "resourceType": "MedicationKnowledge",
        "code": {
          "coding": [{
            "system": "http://snomed.info/sct",
            "code": "319768000"
          }]
        },
        "contraindication": [{
          "reference": "#condition1"
        }],
        "warning": [{
          "text": "Do not use in patients with..."
        }]
      }
    }
  ]
}
```

## Integration with Focusing

1. ePI data flows from this folder → FHIR Emulator
2. Focusing Manager retrieves product information during focusing
3. Lenses are applied to match patient data with product warnings/interactions
4. Results highlight relevant product information for the patient

## Focusing Relevance

ePIs are matched against patient data to:
- Identify contraindications (e.g., drug-disease interactions)
- Highlight warnings based on patient conditions
- Show relevant side effects for the patient
- Demonstrate drug-drug interactions

## Next Steps

- [ ] Download or create ePI sample data
- [ ] Validate FHIR bundle structure
- [ ] Add files to this folder
- [ ] Restart FHIR emulator: `docker-compose restart fhir-emulator`
- [ ] Test access via http://localhost:3000/fhir/Medication

---

**Documentation**: FHIR Medication Resources
**Profile**: http://hl7.org/fhir/medication.html
**Version**: Latest

**Additional Resources**:
- [EMA Medicines](https://www.ema.europa.eu/en/medicines)
- [Gravitate-Health ePI Profile](https://www.gravitatehealth.eu/deliverables/)
