# GitHub Actions Workflows

This directory contains GitHub Actions workflows for continuous integration, building, and releasing the Acode Live Server Backend.

## Workflows Overview

### 1. CI Workflow (`ci.yml`)
**Triggers:** Push and Pull Requests to `main`, `develop`, and `feature/**` branches

**Purpose:** Quick validation of code changes

**Jobs:**
- **Lint:** Code quality checks using flake8 and black
- **Test:** Validates Python modules and tests imports

**When to use:** Runs automatically on every push and PR to ensure code quality.

---

### 2. Python Build Workflow (`python-build.yml`)
**Triggers:** 
- Manual workflow dispatch
- Workflow call from other workflows
- Push/PR to main or develop branches

**Purpose:** Comprehensive multi-platform build and testing

**Jobs:**
1. **lint-and-format:** Code quality and formatting checks
2. **test:** Matrix testing across:
   - Operating Systems: Ubuntu, Windows, macOS
   - Python Versions: 3.9, 3.10, 3.11, 3.12
3. **build-linux:** Build Linux executable with PyInstaller
4. **build-windows:** Build Windows executable with PyInstaller
5. **build-macos:** Build macOS executable with PyInstaller
6. **docker-build:** Build Docker image
7. **create-release:** Package all artifacts and create release notes

**Artifacts produced:**
- `acode-live-server-linux-x64.tar.gz`
- `acode-live-server-windows-x64.zip`
- `acode-live-server-macos-x64.tar.gz`
- `acode-live-server-docker.tar.gz`

---

### 3. Release Workflow (`release.yml`)
**Triggers:**
- Push of version tags (e.g., `v1.0.0`)
- Manual workflow dispatch with version input

**Purpose:** Create GitHub releases with compiled binaries

**Jobs:**
1. **create-release:** Creates a GitHub release with changelog
2. **build-and-upload:** Builds for all platforms and uploads to release

**How to use:**
1. **Automated:** Push a version tag
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. **Manual:** Use GitHub UI
   - Go to Actions → Release → Run workflow
   - Enter version (e.g., v1.0.0)

---

## Workflow Features

### Inspired by RustDesk Flutter Build Workflow
This workflow structure is inspired by the [RustDesk flutter-build.yml](https://github.com/rustdesk/rustdesk/blob/master/.github/workflows/flutter-build.yml), adapted for Python projects:

- **Matrix builds** for multiple platforms and versions
- **Artifact management** for distributable packages
- **Workflow reusability** through `workflow_call`
- **Comprehensive testing** across environments
- **Docker support** for containerized deployments
- **Release automation** with artifact uploads

### Key Differences from Flutter Workflow
- Uses Python/PyInstaller instead of Flutter
- Simplified for backend server (no mobile builds)
- Docker-first approach for deployment
- Python-specific linting and testing tools

### Matrix Strategy Notes
The test matrix is optimized to balance coverage and CI time:
- Full testing on Ubuntu (fastest runner) for all Python versions
- Limited testing on Windows/macOS to reduce CI minutes
- Python 3.11 and 3.12 tested across all platforms (latest versions)
- Python 3.9 and 3.10 tested only on Ubuntu (older versions)

If your project requires comprehensive cross-platform testing for older Python versions, you can remove the exclusions in `python-build.yml`.

### Changelog Format
The release workflow reads from `changelogs.md` if it exists. For better release notes:
- Use a structured changelog format (e.g., Keep a Changelog)
- Or use GitHub's auto-generated release notes
- Or implement version-specific changelog extraction

---

## Environment Variables

### Default Versions
- `PYTHON_VERSION`: "3.11" (primary version)
- `VERSION`: "1.0.0" (application version)

### Customization
You can customize versions by editing the workflow files:
```yaml
env:
  PYTHON_VERSION: "3.11"
  VERSION: "1.0.0"
```

---

## Development Tips

### Local Testing
Before pushing, you can test locally:

```bash
# Install dependencies
pip install -r requirements.txt

# Lint
pip install flake8 black
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
black --check .

# Test imports
python -c "from main import app; print('OK')"

# Build with PyInstaller
pip install pyinstaller
pyinstaller --onefile --name acode-live-server main.py
```

### Running Specific Workflows
You can manually trigger workflows from the GitHub Actions tab:
1. Go to your repository on GitHub
2. Click "Actions" tab
3. Select the workflow
4. Click "Run workflow"

---

## Troubleshooting

### Workflow Fails on Import
- Ensure all dependencies are in `requirements.txt`
- Check Python version compatibility

### Build Fails
- Check PyInstaller compatibility with your Python version
- Ensure all non-Python files are included in the build

### Docker Build Fails
- Verify Dockerfile is created correctly
- Check if all required files are copied

---

## Contributing

When adding new workflows:
1. Validate YAML syntax: `python -c "import yaml; yaml.safe_load(open('.github/workflows/your-workflow.yml'))"`
2. Test locally when possible
3. Document the workflow in this README
4. Follow the existing naming conventions

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Python Actions Setup](https://github.com/actions/setup-python)
- [PyInstaller Documentation](https://pyinstaller.org/)
- [RustDesk Flutter Build Reference](https://github.com/rustdesk/rustdesk/blob/master/.github/workflows/flutter-build.yml)
