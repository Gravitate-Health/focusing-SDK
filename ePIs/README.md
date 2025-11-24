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
Obtain ePI examples from the [Gravitate-Health FHIR Implementation Guide](https://build.fhir.org/ig/hl7-eu/gravitate-health/).

### Option 2: Europeana Medicines API
The [Europeana Medicines Portal](https://www.ema.europa.eu/en/medicines) provides structured product information.

### Option 3: Create from SmPC
Convert Summary of Product Characteristics (SmPC) documents to FHIR format.


## Usage

The FHIR emulator automatically discovers and loads all JSON files in this folder at startup:

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
3. Preprocessors are applied.
4. Lenses are applied to match patient data with product warnings/interactions
5. Results highlight relevant product information for the patient

## Focusing Relevance

ePIs are matched against patient data to:
- Identify contraindications (e.g., drug-disease interactions)
- Highlight warnings based on patient conditions
- Show relevant side effects for the patient
- Demonstrate drug-drug interactions

---

**Documentation**: FHIR Medication Resources
**Profile**: http://hl7.org/fhir/medication.html
**Version**: Latest

**Additional Resources**:
- [EMA Medicines](https://www.ema.europa.eu/en/medicines)
- [Gravitate-Health ePI Profile](https://build.fhir.org/ig/hl7-eu/gravitate-health/)
