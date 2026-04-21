### Task 1 — Classify and Handle PII Fields

| Field           | Type                       | Action               | Justification                                                           |
| --------------- | -------------------------- | -------------------- | ----------------------------------------------------------------------- |
| full_name       | Direct PII                 | Drop                 | Directly identifies a person. Not needed for research.                  |
| email           | Direct PII                 | Drop                 | Direct personal identifier. High privacy risk.                          |
| date_of_birth   | Indirect PII               | Mask                 | Can help re-identify individuals. Use age range instead (e.g. 20–30).   |
| zip_code        | Indirect PII               | Mask                 | Can contribute to re-identification. Use broader region or partial zip. |
| job_title       | Indirect PII               | Keep or Pseudonymize | May be identifying in rare cases. Can generalize into categories.       |
| diagnosis_notes | Sensitive Data (quasi-PII) | Pseudonymize         | Remove names/identifiers while keeping research value.                  |



Task 2 — Audit the API Script for Ethical Compliance
Violation 1: Hardcoded API Key (Security Risk)
Problem:
API_KEY = "free_tier_key_abc123"
API keys should never be exposed in source code.
If pushed to GitHub, the credential can be stolen.
Violates security best practices.
Corrected Version:
import os

API_KEY = os.getenv("API_KEY")

Set environment variable:

export API_KEY="your_actual_key"
Violation 2: Permanent Storage of Raw PII Data
Problem:
save_to_database(records)
Stores raw patient records containing PII.
No cleaning or anonymization before storage.
Violates privacy and ethical data-sharing principles.
Corrected Version:
def anonymize(record):
    return {
        "age_group": "20-30",   # example replacing DOB
        "region": record["zip_code"][:3],  # partial zip masking
        "job_category": record["job_title"],
        "diagnosis_notes": remove_identifiers(record["diagnosis_notes"])
    }

clean_records = [anonymize(r) for r in records]

save_to_database(clean_records)
Additional Improved Script
import requests
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []

for page in range(1,101):
    response = requests.get(
        API_URL,
        params={
            "page": page,
            "key": API_KEY
        }
    )

    data = response.json()

    for r in data["results"]:
        cleaned = {
            "age_group": "20-30",
            "region": r["zip_code"][:3],
            "job_category": r["job_title"]
        }

        records.append(cleaned)

save_to_database(records)

