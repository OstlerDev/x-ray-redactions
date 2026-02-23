# x-ray: PDF Redaction Failure Analyzer

`x-ray` is a security analysis tool for detecting ineffective
redactions in PDF files. It identifies cases where text is still recoverable
even though a rectangle appears to hide it visually.

This repository is intended for ethical use by security research teams.

## Authorized Use

Use this tool only on documents you are explicitly authorized to assess.

- Internal assessments and QA workflows
- Third-party validation of documents
- Incident response

Do not use this tool for any unethical activity.

## What It Detects

`x-ray` focuses on overlay-style redaction failures where text remains embedded
in the PDF content stream and can still be extracted.

High-level pipeline:

1. Parse page drawings and detect candidate opaque rectangles.
2. Identify characters occluded by those rectangles.
3. Group characters by rectangle and apply text-based filters.
4. Render each region and keep only uniform-color overlays likely to be
   ineffective redactions.

Detailed model and caveats: [`docs/detection-model.md`](docs/detection-model.md)

## Scope and Non-Goals

In scope:

- PDF vector/overlay redaction failures where hidden text remains extractable
- Batch triage for likely redaction mistakes

Out of scope:

- OCR-only leaks in image-only documents
- Exhaustive forensic PDF reverse engineering for every possible leak pattern
- Policy decisions about the "sensitivity" of a document, and if it should be ethically published

## Installation

Using `uv`:

```text
uv add x-ray
```

Using `pip`:

```text
pip install x-ray
```

## Quickstart (Offline, Local Files)

Run against a known test file:

```bash
xray tests/assets/rectangles_yes.pdf
```

Run against a file with no expected findings:

```bash
xray tests/assets/no_bad_redactions.2.1.pdf
```

Use Python API with a local path:

```python
from pathlib import Path
import xray

findings = xray.inspect(Path("tests/assets/rectangles_yes.pdf"))
print(findings)
```

Use Python API with file bytes:

```python
import xray

with open("tests/assets/rectangles_yes.pdf", "rb") as f:
    findings = xray.inspect(f.read())
print(findings)
```

Note: `xray.inspect()` supports `https://` strings, but internal workflows
should prefer local-file processing for predictable handling and auditing.

## Output Format

Output is JSON from CLI and a Python object from API. The top-level structure:

- Dictionary keyed by page number (`int`)
- Each page maps to a list of findings
- Each finding contains:
  - `bbox`: `(x0, y0, x1, y1)` coordinates for the candidate redaction area
  - `text`: underlying recovered text occluded by that rectangle

Example shape:

```json
{
  "1": [
    {
      "bbox": [58.55, 72.19, 75.65, 739.39],
      "text": "Example recovered text"
    }
  ]
}
```

Treat recovered `text` as potentially sensitive data.

## Known Limitations

No automated detector is perfect. `x-ray` can produce false positives and false
negatives depending on PDF complexity.

Notable limitations reflected in tests:

- Complex overlapping text/layering scenarios can evade detection
- Some non-standard drawing behaviors require conservative heuristics
- Image-heavy documents and OCR edge cases are not comprehensively handled

For details, see [`docs/detection-model.md`](docs/detection-model.md).

## Integration Patterns

Batch and pipeline examples (local files, JSON post-processing):
[`docs/integration.md`](docs/integration.md)

## Development

Set up development dependencies:

```bash
uv sync
```

Run unit tests:

```bash
python -m unittest -v
```

Run tox matrix locally:

```bash
tox
```

Run pre-commit checks:

```bash
pre-commit run --all-files
```

Run type checks:

```bash
mypy .
```

## Issue Reporting

Use GitHub issues in this repository for:

- Detection quality issues
- False positive/false negative examples
- Security concerns in parser or dependency behavior
- Enhancement requests for detection workflows

## License

This repository is licensed under BSD-2-Clause. See [`LICENSE`](LICENSE).
