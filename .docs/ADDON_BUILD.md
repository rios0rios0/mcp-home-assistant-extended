# GitHub Actions Setup

This repository includes automated builds via GitHub Actions that build and push Docker images to Docker Hub.

## Configuration

- **Registry**: `rios0rios0`
- **Image Name**: `mcp-home-assistant-extended`
- **Image Pattern**: `rios0rios0/{arch}-addon-mcp-home-assistant-extended`

Where `{arch}` is one of: `aarch64`, `armv7`, `amd64`

## Workflow

The workflow (`.github/workflows/build.yaml`) automatically:

1. **Triggers on**:
   - Push to `main` or `master` branch
   - Creation of version tags (e.g., `v0.1.0`)
   - Pull requests (builds only, no push)
   - Manual dispatch via GitHub UI

2. **Builds for all architectures**:
   - `aarch64` (ARM 64-bit)
   - `armv7` (ARM 32-bit)
   - `amd64` (x86_64)

3. **Tags images**:
   - `latest` (only on default branch)
   - Branch name (e.g., `main`)
   - Semantic version tags (e.g., `0.1.0`, `0.1`)

## Setup Instructions

### 1. Configure GitHub Secrets

Before the workflow can push images, you need to set up Docker Hub credentials:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add the following secrets:
   - **Name**: `DOCKER_USERNAME`
     - **Value**: Your Docker Hub username (`rios0rios0`)
   - **Name**: `DOCKER_PASSWORD`
     - **Value**: Your Docker Hub access token (create one at https://hub.docker.com/settings/security)

### 2. Push to GitHub

Once secrets are configured, push your code to trigger the build:

```bash
git add .
git commit -m "Add GitHub Actions workflow"
git push origin main
```

**Note**: The GitHub repository is `rios0rios0/mcp-home-assistant-extended`
```

### 3. Monitor Builds

- View build progress: **Actions** tab in your GitHub repository
- Check build logs for any issues
- Images will be pushed to Docker Hub automatically on successful builds

## Image URLs

After a successful build, images will be available at:

- `rios0rios0/aarch64-addon-mcp-home-assistant-extended:latest`
- `rios0rios0/armv7-addon-mcp-home-assistant-extended:latest`
- `rios0rios0/amd64-addon-mcp-home-assistant-extended:latest`

## Version Tags

To create a versioned release:

```bash
git tag -a v0.1.0 -m "Release version 0.1.0"
git push origin v0.1.0
```

This will build and tag images with:
- `rios0rios0/{arch}-addon-mcp-home-assistant-extended:0.1.0`
- `rios0rios0/{arch}-addon-mcp-home-assistant-extended:0.1`

## Troubleshooting

### Build fails with authentication error

- Verify `DOCKER_USERNAME` and `DOCKER_PASSWORD` secrets are set correctly
- Ensure the Docker Hub access token has write permissions
- Check that the token hasn't expired

### Build fails with "platform not supported"

- The workflow uses QEMU for cross-platform builds
- This is handled automatically by the `docker/setup-qemu-action`

### Images not appearing on Docker Hub

- Check the Actions tab for build errors
- Verify the workflow completed successfully
- Ensure you're looking at the correct Docker Hub repository

## Local Testing

To test builds locally before pushing:

```bash
./build.sh
```

This will build images locally (without pushing) for testing.

## Related Documentation

- [Addon Installation](ADDON_INSTALLATION.md) - Installing the addon
- [Addon Structure](ADDON_STRUCTURE.md) - Understanding the addon architecture
- [Addon Setup Summary](ADDON_SETUP_SUMMARY.md) - Overview of addon components
- [Documentation Index](SUMMARY.md) - Complete documentation navigation
