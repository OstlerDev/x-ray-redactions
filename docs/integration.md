# Integration Guide

This guide covers internal integration patterns for running `x-ray` at scale
using local files and offline workflows.

## Basic CLI Usage

Single file:

```bash
xray tests/assets/rectangles_yes.pdf
```

Another corpus file expected to return no findings:

```bash
xray tests/assets/no_bad_redactions.2.1.pdf
```

## Batch Scanning Patterns

PowerShell (Windows):

```powershell
Get-ChildItem .\tests\assets\*.pdf | ForEach-Object {
  xray $_.FullName
}
```

Bash:

```bash
for f in tests/assets/*.pdf; do
  xray "$f"
done
```

## Structured Output Collection

PowerShell example writing one JSON file per PDF:

```powershell
New-Item -ItemType Directory -Path .\scan-output -Force | Out-Null
Get-ChildItem .\tests\assets\*.pdf | ForEach-Object {
  $out = Join-Path .\scan-output ($_.BaseName + ".json")
  xray $_.FullName | Set-Content $out
}
```

## Python API Integration

```python
from pathlib import Path
import json
import xray

assets = Path("tests/assets")
results = {}
for pdf in assets.glob("*.pdf"):
    findings = xray.inspect(pdf)
    if findings:
        results[str(pdf)] = findings

print(json.dumps(results, indent=2))
```

## Regression Corpus Usage

The `tests/assets/` set is useful as a baseline internal regression corpus:

- Positive cases (expected findings), for example `rectangles_yes.pdf`
- Negative cases (expected no findings), for example
  `no_bad_redactions.2.1.pdf`
- Known difficult cases, for example `hidden_text_on_visible_text.pdf`

When changing detection logic, run the test suite and compare behavior against
this corpus before release.
