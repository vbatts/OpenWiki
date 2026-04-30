# Flatpak Packaging

## Prerequisites

```bash
# flatpak-builder and flatpak itself
sudo apt-get install flatpak flatpak-builder

# Add the Flathub remote and the GNOME SDK
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub org.gnome.Platform//47 org.gnome.Sdk//47
flatpak install flathub org.freedesktop.Sdk.Extension.rust-stable//24.08
flatpak install flathub org.freedesktop.Sdk.Extension.node20//24.08

# Source generators (Python)
pip install flatpak-builder-tools
```

## Generating offline sources

Flatpak builds run without network access. Before building or submitting to Flathub,
generate the cargo and npm source lists from the lock files:

```bash
# From the repo root
flatpak-cargo-generator src-tauri/Cargo.lock -o flatpak/cargo-sources.json
flatpak-node-generator npm package-lock.json -o flatpak/node-sources.json
```

Re-run these commands whenever `Cargo.lock` or `package-lock.json` changes.

## Local test build

```bash
cd /path/to/OpenWiki
flatpak-builder --force-clean build-dir flatpak/com.openwiki.app.yml
flatpak-builder --run build-dir flatpak/com.openwiki.app.yml openwiki
```

To install locally for testing:

```bash
flatpak-builder --user --install --force-clean build-dir flatpak/com.openwiki.app.yml
flatpak run com.openwiki.app
```

## Flathub submission

1. Fork https://github.com/flathub/flathub
2. Create a branch `new-app/com.openwiki.app`
3. Add a directory `com.openwiki.app/` containing:
   - `com.openwiki.app.yml`  (the manifest)
   - `com.openwiki.app.metainfo.xml`
   - `com.openwiki.app.desktop`
   - `cargo-sources.json`   (generated)
   - `node-sources.json`    (generated)
4. Pin the `commit:` field in the manifest to the exact release commit SHA
5. Add at least one screenshot to `flatpak/screenshots/` and push to main
6. Open a pull request on the Flathub repo

See https://docs.flathub.org/docs/for-app-authors/submission for the full checklist.

## Notes

- Screenshots are required for Flathub. Add real screenshots to `flatpak/screenshots/`
  and update the `<screenshots>` block in `com.openwiki.app.metainfo.xml`.
- The `WEBKIT_DISABLE_DMABUF_RENDERER=1` env var prevents a known WebKitGTK crash
  inside the Flatpak sandbox on some hardware.
- System tray support inside Flatpak requires the GNOME AppIndicator extension or
  a compatible StatusNotifierItem host. The finish-args include the necessary D-Bus
  names, but some GNOME setups need the extension installed separately.
