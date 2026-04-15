# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`constellat-broadcast-service` is a lightweight Python library implementing the publish/subscribe (broadcast) pattern. It is published to PyPI as `constellat_broadcast_service`.

## Architecture

The library is minimal — two files do all the work:

- `constellat_broadcast_service/_core.py` — contains `BroadcastService` class and the module-level singleton `broadcast_service`. Handles topic registration, listener subscription, and dispatch (optionally via a `ThreadPoolExecutor` with 5 workers).
- `constellat_broadcast_service/__init__.py` — re-exports only `broadcast_service` from `_core.py`.

Users import the singleton directly: `from broadcast_service import broadcast_service` (note: the import alias drops the `constellat_` prefix via package naming).

The three public methods on `BroadcastService`:
- `listen(topic_name, callback)` — subscribe a callback to a topic
- `broadcast(topic_name, *args, **kwargs)` — dispatch to all subscribers, threaded by default
- `stop_listen(topic_name, callback)` — unsubscribe a callback

`enable_multithread = True` by default; set to `False` on the singleton to run callbacks synchronously.

## Packaging & Publishing

Version is set in `setup.py` → `version` field. Current version: `1.0.3`.

```bash
# Build source distribution
python setup.py sdist

# Upload to PyPI
twine upload dist/*
# or upload a specific version
twine upload dist/constellat_broadcast_service-<version>.tar.gz
```

> Prefer `python -m build` over `python setup.py sdist` for modern builds (generates both sdist and wheel). See `summary.md` for the full PyPI publishing workflow.

When bumping the version, update only the `version` field in `setup.py`. PyPI does not allow re-uploading the same version.
