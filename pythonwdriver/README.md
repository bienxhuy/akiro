# pythonwdriver

This directory contains Dockerfiles for building Python base images with a pre-installed browser driver. These images are used as the base for the test framework container in the pipeline, providing a ready-to-use environment for E2E test execution without requiring driver installation at runtime.

---

## Available Images

| Dockerfile | Browser | Driver |
|------------|---------|--------|
| `chrome.Dockerfile` | Google Chrome (stable) | ChromeDriver (version-matched automatically) |

Additional browser images (e.g., Firefox) can be added as separate Dockerfiles following the same naming convention.

---

## Building an Image

### Chrome

```bash
cd pythonwdriver
docker build -f chrome.Dockerfile -t pythonwdriver:chrome .
```

The build process:
1. Starts from `python:3.12`.
2. Installs Google Chrome stable from the official Google repository.
3. Detects the installed Chrome major version and downloads the matching ChromeDriver automatically.
4. Installs `pytest` and `selenium` as base packages.

> The image does not pin a specific Chrome or ChromeDriver version. It resolves the latest stable version at build time. Rebuild the image periodically or when the Chrome version on the host drifts significantly from the image.

---

## Using the Image in the Pipeline

The `Jenkinsfile` references this image by name when creating the test framework container. Ensure the image is built on the Jenkins host before running the pipeline for the first time.

The test framework container installs its full Python dependencies at runtime from the local Devpi mirror, so this image only needs to provide the base Python runtime and the browser driver — not the full framework stack.

---

## Tagging Convention

| Tag | Meaning |
|-----|---------|
| `pythonwdriver:chrome` | Latest build for Chrome |
| `pythonwdriver:chrome-<date>` | Dated snapshot, e.g. `chrome-20260601` |

Using a dated tag is recommended when you want to pin a specific Chrome version for a period of time before upgrading.