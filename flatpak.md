# Flatpak Packaging for OBS PTZ Plugin

This document describes the steps needed outside this source repository to
publish obs-ptz as a Flatpak extension on Flathub.

## Overview

The `flatpak/` directory contains the packaging files required to build obs-ptz
as a Flatpak extension (`com.obsproject.Studio.Plugin.Ptz`) for the OBS Studio
Flatpak (`com.obsproject.Studio`). These files are kept in this repo for
reference, but they must be submitted to a separate Flathub repository to
actually publish the extension.

## Files in `flatpak/`

| File | Purpose |
|------|---------|
| `com.obsproject.Studio.Plugin.Ptz.yaml` | Flatpak build manifest |
| `com.obsproject.Studio.Plugin.Ptz.metainfo.xml` | AppStream metadata (shown in software centers) |
| `flathub.json` | Flathub build configuration (architecture list) |

## Steps to Publish on Flathub

### 1. Create a Flathub Repository

Submit a new repository request to Flathub:

1. Go to https://github.com/flathub/flathub/issues/new
2. Open a "New App Submission" issue
3. Provide the app ID: `com.obsproject.Studio.Plugin.Ptz`
4. Note that this is an OBS Studio plugin extension (not a standalone app)
5. Wait for Flathub maintainers to create the repository at
   `https://github.com/flathub/com.obsproject.Studio.Plugin.Ptz`

### 2. Populate the Flathub Repository

Once the repository is created:

1. Clone it:
   ```bash
   git clone git@github.com:flathub/com.obsproject.Studio.Plugin.Ptz.git
   cd com.obsproject.Studio.Plugin.Ptz
   ```

2. Copy the files from this repo's `flatpak/` directory into the root of the
   Flathub repo:
   ```bash
   cp /path/to/obs-ptz/flatpak/* .
   ```

3. Commit and push:
   ```bash
   git add .
   git commit -m "Initial submission of obs-ptz plugin"
   git push
   ```

### 3. Test the Build Locally

Before submitting, verify the build works:

```bash
# Install the Flatpak SDK and OBS runtime if not already present
flatpak install flathub org.kde.Sdk//6.8
flatpak install flathub com.obsproject.Studio

# Build the extension
flatpak-builder --force-clean build-dir com.obsproject.Studio.Plugin.Ptz.yaml

# Install locally for testing
flatpak-builder --user --install --force-clean build-dir com.obsproject.Studio.Plugin.Ptz.yaml

# Run OBS to verify the plugin loads
flatpak run com.obsproject.Studio
```

### 4. Updating the Extension

When a new version of obs-ptz is released:

1. Tag the release in this repository (e.g., `0.19.0`)
2. Update the Flathub repo:
   - Update the `tag` and `commit` fields in the manifest
   - Update the `<releases>` section in the metainfo XML
3. Push to the Flathub repository; the build bot will automatically rebuild

If `x-checker-data` is configured on the Flathub side (it is included in the
manifest), Flathub's automated update bot may open PRs for new tags
automatically.

### 5. Sandbox Permissions

The Flatpak sandbox restricts access to hardware by default. Users who need
serial port access (for VISCA over RS-232) will need to grant additional
permissions. Serial port support is disabled in the Flatpak build by default
since it requires `--device=all` or specific device overrides.

Users can override permissions with:

```bash
flatpak override --user --device=all com.obsproject.Studio
```

Network-based protocols (VISCA over UDP/TCP and ONVIF) work within the default
OBS Studio sandbox since it already has network access.
