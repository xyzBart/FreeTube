# FreeTube Dev Notes

## Build & Run (Ubuntu 24, Wayland)

**Prerequisites:** Node.js, yarn, pnpm (`npm install -g pnpm`). No special system packages needed (Electron bundles its own libs).

**First time setup (dev):**
```bash
yarn install
```

**Compile (keep running — serves renderer via webpack-dev-server on port 9080):**
```bash
yarn dev
```

**Launch app in a second terminal (while `yarn dev` is still running):**
```bash
ELECTRON_OZONE_PLATFORM_HINT=auto node_modules/.bin/electron --no-sandbox dist/main.js
```

> `--no-sandbox` is required because `chrome-sandbox` is not SUID-configured.
> To fix permanently instead: `sudo chown root:root node_modules/electron/dist/chrome-sandbox && sudo chmod 4755 node_modules/electron/dist/chrome-sandbox`

**Clean build output:**
```bash
yarn clean
```

---

## Production AppImage Build (Ubuntu 24, Wayland)

**Prerequisites:** pnpm must be installed and `@parcel/watcher` approved (one-time):
```bash
npm install -g pnpm
pnpm install          # first run will prompt to approve @parcel/watcher build scripts
```
Use `expect` script to approve non-interactively if needed (see session history).

**Also required for AppImage to run:**
```bash
sudo apt install libfuse2t64   # Ubuntu 24 renamed libfuse2
```

**Build (produces AppImage, deb, zip, 7z — rpm skipped, needs rpmbuild):**
```bash
pnpm run build
```
Output goes to `build/`. Note: `yarn.lock` must not be present alongside `pnpm-lock.yaml` or electron-builder fails to detect pnpm — rename it if needed:
```bash
mv yarn.lock yarn.lock.bak
pnpm run build
mv yarn.lock.bak yarn.lock
```

**Run AppImage on Wayland:**
```bash
./build/FreeTube-<version>.AppImage --no-sandbox
```
`--no-sandbox` is required (same reason as dev mode — chrome-sandbox not SUID). The filename embeds the app version from `package.json` (e.g. `FreeTube-0.25.1.AppImage`), so it changes on every version bump — check `build/` for the current filename.

**GNOME launcher:** `~/.local/share/applications/freetube-dev.desktop` ("FreeTube (dev build)") has an `Exec=` line hardcoded to a specific `build/FreeTube-<version>.AppImage` path. After every rebuild that changes the version, update that path (and run `update-desktop-database ~/.local/share/applications/`) or the launcher will fail with "Program ... not found in $PATH".

**User data locations:**
- AppImage → `~/.config/FreeTube/`
- `yarn dev` → `~/.config/Electron/`  (separate — subscriptions/settings don't interfere)

To share data between dev and AppImage:
```bash
ELECTRON_OZONE_PLATFORM_HINT=auto node_modules/.bin/electron --no-sandbox --user-data-dir=$HOME/.config/FreeTube dist/main.js
```
