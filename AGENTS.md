# Console Whisperer Deployment - Main Branch

## Branch Purpose

The **main** branch is the packaging and release infrastructure for **Console Whisperer**. It uses [Hydraulic Conveyor](https://hydraulic.dev) to create native installers (Windows/macOS) from a JAR artifact built in a private repository.

**Related Branch:** The `gh-pages` branch contains the marketing website and download portal. See that branch's AGENTS.md for website-specific documentation.

---

## Architecture Overview

### Release Flow
1. **Source Build** → Private repo (`LiveNathan/console-whisperer-desktop`) builds the Spring Boot JAR
2. **Dispatch Event** → Private repo triggers this repo via `repository_dispatch` event
3. **Package** → This repo downloads the JAR and runs Conveyor to create installers
4. **Publish** → Conveyor uploads installers to GitHub Releases and updates the download page on `gh-pages`

### Key Components

- **`conveyor.conf`**: Conveyor configuration for packaging the Spring Boot app
- **`.github/workflows/package.yml`**: GitHub Action that orchestrates the packaging process
- **`app-icon.svg`**: Application icon used for installers and app branding

---

## Conveyor Configuration (`conveyor.conf`)

### Application Metadata
- **Display Name**: "Console Whisperer"
- **Reverse DNS**: `dev.nathanlively`
- **Filesystem Name**: `console-whisperer-desktop`
- **Platforms**: macOS (Apple Silicon & Intel), Windows (AMD64)

### JVM Configuration
- **Java Version**: 25 (OpenJDK)
- **Main Class**: `org.springframework.boot.loader.launch.JarLauncher` (Spring Boot launcher)
- **GUI Mode**: No console window (GUI only)
- **Memory**: Adaptive (6% initial, 25% max of system RAM)
- **Garbage Collector**: G1GC
- **Virtual Threads**: Enabled (`spring.threads.virtual.enabled=true`)

### Spring Boot Properties
```hocon
server.port = 22019
spring.application.name = console-whisperer-desktop
vaadin.launch-browser = false
embabel.models.default-llm = gemini-2.5-flash
spring.servlet.multipart.max-file-size = 25MB
```

### Special JVM Flags
- Multiple `--add-opens` directives for Spring Boot 3.x + Java 25 compatibility
- `--enable-native-access=ALL-UNNAMED` for native library access
- macOS: `-XstartOnFirstThread` for UI frameworks

### Update Strategy
- **Updates**: `aggressive` (auto-update enabled)
- **Compression**: `high` (minimize download size)

### GitHub Pages Integration
```hocon
app.site.github {
  oauth-token = ${env.GITHUB_TOKEN}
  pages-branch = "gh-pages"
}
```
Conveyor automatically publishes installers to GitHub Releases and updates the download page on `gh-pages`.

---

## GitHub Actions Workflow (`.github/workflows/package.yml`)

### Triggers
1. **`repository_dispatch`** with type `release-agent` (triggered from private repo)
2. **`workflow_dispatch`** (manual trigger with version and run_id inputs)

### Workflow Steps

#### 1. System Diagnostics
Captures initial disk and memory state for debugging

#### 2. Download JAR Artifact
```yaml
gh run download "$RUN_ID" \
  --repo "$SOURCE_REPO" \
  --name jar-artifact \
  --dir target/
```
Downloads the JAR from the private repo's workflow run using `SOURCE_REPO_TOKEN` secret.

#### 3. Verify JAR
Checks existence, size, and MD5 checksum of the downloaded JAR.

#### 4. Run Conveyor
```yaml
uses: hydraulic-software/conveyor/actions/build@v21.1
with:
  command: make copied-site
  signing_key: ${{ secrets.SIGNING_KEY }}
```
Runs Conveyor to:
- Create native installers (`.exe`, `.dmg`, `.pkg`)
- Sign the installers (requires `SIGNING_KEY` secret)
- Upload to GitHub Releases
- Update the download page on `gh-pages`

#### 5. Post-build Diagnostics
Captures final system state and artifact sizes (runs even on failure).

### Required Secrets
- **`SOURCE_REPO_TOKEN`**: GitHub PAT with access to the private repo (for downloading JAR and pushing to gh-pages)
- **`SIGNING_KEY`**: Conveyor signing key (for code signing installers)
- **`VKT`**: Conveyor license token

---

## Development Workflows

### Testing Changes to Conveyor Config
1. Modify `conveyor.conf`
2. Commit changes to `main`
3. Trigger manually via `workflow_dispatch` with a test version
4. Check GitHub Releases for the generated installers

### Adding New JVM Options
Edit the `jvm.options` array in `conveyor.conf`:
```hocon
jvm {
  options += "-XX:NewFlag=value"
}
```

### Changing Target Platforms
Edit the `machines` array:
```hocon
app {
  machines = [ "mac", "windows.amd64", "linux.amd64.glibc" ]
}
```

### Updating Java Version
1. Update `include required` statement for new JDK version
2. Update `jvm.feature-version` and `jvm.version`
3. Test thoroughly (especially `--add-opens` flags)

---

## Troubleshooting

### Common Issues

**JAR Download Fails**
- Check `SOURCE_REPO_TOKEN` has access to private repo
- Verify `run_id` is valid and artifact exists
- Check artifact name is exactly `jar-artifact`

**Conveyor Build Fails**
- Check `SIGNING_KEY` is valid
- Ensure JAR is properly built (Spring Boot fat JAR)
- Review Conveyor compatibility level (`conveyor.compatibility-level = 21`)

**Installers Not Uploaded**
- Verify `GITHUB_TOKEN` (via `SOURCE_REPO_TOKEN`) has write access to Releases
- Check `app.site.github.pages-branch` points to correct branch

**Out of Disk Space**
- Conveyor builds can be large (native binaries + JDK)
- Check diagnostics output for disk usage
- Consider reducing `jvm.modules` to only required modules

### Debug Mode
Add to `conveyor.conf` for more verbose output:
```hocon
app {
  logging = "verbose"
}
```

---

## File Structure

```
main branch/
├── .github/
│   └── workflows/
│       └── package.yml          # Packaging workflow
├── conveyor.conf                # Conveyor configuration
├── app-icon.svg                 # Application icon
├── .gitignore                   # Ignore .idea, .DS_Store, etc.
└── AGENTS.md (→ CLAUDE.md)      # This file
```

---

## Best Practices

### When Modifying Conveyor Config
- Always test with a pre-release version first
- Check both Windows and macOS installers
- Verify auto-update works after deployment
- Monitor installer download sizes (compression settings)

### When Updating Dependencies
- Keep Java version in sync with private repo
- Update `jvm.modules` if adding dependencies with new module requirements
- Test `--add-opens` flags if updating Spring Boot version

### Release Versioning
- Use semantic versioning (e.g., `1.2.3`)
- Version is passed from private repo via `APP_VERSION` env var
- Conveyor uses version for installer names and update checks

---

## Integration with gh-pages Branch

The `gh-pages` branch is automatically updated by Conveyor:
- Download page is regenerated with new version info
- Installers are linked from GitHub Releases
- No manual intervention needed

To update the marketing website (landing page), work directly on the `gh-pages` branch. See that branch's AGENTS.md for details.
