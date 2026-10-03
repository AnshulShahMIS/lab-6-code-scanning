# Answers to Part 3

Add your answers to the questions in Part 3, Step 2 below.

## Vulnerability Remediation:
### Vulnerability 1:
1. Which package or library are you addressing?

**Answer:** Pillow 9.4.0 (Python imaging library), declared on line 20 of `requirements.txt`. Flagged as CRITICAL by the Trivy filesystem scan.

2. Which CVE is linked to this vulnerability?

**Answer:** CVE-2023-50447. Pillow through version 10.1.0 allows arbitrary code execution through the `environment` parameter of `PIL.ImageMath.eval`. An attacker who can influence that parameter can get Python code executed on the server. This is a separate flaw from CVE-2022-22817, which affected the `expression` parameter of the same function.

3. What remediation steps do you suggest?

**Answer:**
- Upgrade Pillow to 10.2.0 or later (the fixed version reported by Trivy) by changing line 20 of `requirements.txt` to `Pillow>=10.2.0`, then rebuild the container image.
- Re-run the pipeline to confirm Trivy no longer reports the CVE.
- As defense in depth, never pass user-controlled input into `ImageMath.eval`.

### Vulnerability 2:
1. Which vulnerability are you addressing?

**Answer:** PyYAML 5.1, declared on line 29 of `requirements.txt`. Flagged as CRITICAL by the Trivy filesystem scan, which reported three CRITICAL CVEs against this version (CVE-2019-20477, CVE-2020-1747, and CVE-2020-14343).

2. Which CVE is linked to this vulnerability?

**Answer:** CVE-2020-14343. PyYAML versions before 5.4 allow arbitrary code execution when processing untrusted YAML with `yaml.full_load()` or `FullLoader`. A crafted YAML document can use the `python/object/new` constructor to make the parser create Python objects and run code. This is an insecure deserialization flaw, and it exists because the earlier fix for CVE-2020-1747 was incomplete.

3. What remediation steps do you suggest?

**Answer:**
- Upgrade PyYAML to 6.0.1 or later. Trivy reports 5.4 as the minimum fixed version, but 6.0.1+ is recommended because older versions have build issues on Python 3.11, which PyGoat's base image uses. Update line 29 of `requirements.txt` and rebuild the image.
- Any code that parses user-supplied YAML should use `yaml.safe_load()` (or `SafeLoader`), which only constructs basic data types and cannot instantiate arbitrary Python objects.
- Re-run the pipeline to confirm all three PyYAML CVEs are cleared.