# Publishing Webamp Updates

Webamp is a static Progressive Web App deployed to GitHub Pages. Publish a new version by committing changes and pushing them to `main`; the Pages workflow builds and deploys the app automatically.

## Release Steps

1. Make and locally test the change. For service-worker testing, serve the project on `localhost` or HTTPS; opening `index.html` with `file://` does not support PWA installation or service workers.
2. Commit the intended files and push to `main`:

   ```sh
   git add <files>
   git commit -m "Describe the change"
   git push origin main
   ```

3. Check the **Deploy webamp to GitHub Pages** workflow in the repository's Actions tab. Wait for both the build and deploy jobs to succeed.
4. Open the Pages URL and confirm the footer shows the new short commit ID. Reopen or foreground an installed copy and confirm it shows that same ID.

The workflow is `.github/workflows/deploy-pages.yml`. It copies the static app into the Pages artifact, writes the workflow commit SHA into `version.json`, substitutes that SHA into `service-worker.js`, then deploys the artifact. No package install, manual version bump, or generated `_site/` commit is required. `_site/` is build output and is ignored by Git.

## How Installed Apps Update

- Keep the service-worker URL stable as `service-worker.js`. The browser checks that script for changes when the app opens; Webamp also requests an update when the app returns to the foreground.
- Each deployment changes the worker's commit-stamped cache name. The new worker installs a fresh app-shell cache, activates immediately, and removes older `webamp-shell-*` caches.
- The page reloads after the new worker takes control if no track is playing. While audio is playing, Webamp defers the reload until playback is paused, avoiding an update interrupting a song. A long playlist can therefore delay the visible version change until playback is stopped.
- The user's music library is stored in IndexedDB for that exact origin and browser profile. The deployment cache does not contain or clear uploaded tracks. A move from localhost to GitHub Pages is a different origin and does not migrate the local library.
- The footer's short SHA is the deployed app version. A `local` value is expected when running from the source tree without the Pages workflow-generated `version.json`.

## First-Time Pages Setup

The repository's Pages build source must be **GitHub Actions**. The workflow uses `actions/configure-pages` and the official Pages artifact/deploy actions. If deployment reports that Pages is disabled or does not use Actions, enable **Settings → Pages → Build and deployment → Source → GitHub Actions**, then rerun the workflow.

## Playback and Platform Notes

Webamp uses the native audio element for normal playback and registers Media Session metadata and supported transport actions for lock-screen controls. Background playback and lock-screen UI are ultimately controlled by each browser and operating system; validate on real iOS and Android devices before promising identical behavior. The equalizer opts into Web Audio only when an EQ band is changed.

## Keep These Invariants

- Keep launch, scope, icon, and app-shell URLs relative so the PWA works at both localhost root and the GitHub Pages `/webamp/` project path. Keep the manifest `id` explicitly set to `/webamp/`; `./` resolves to the GitHub Pages origin root and can collide with other installed apps on `timelessp.github.io`.
- During service-worker installation, fetch app-shell assets with a build-specific query before storing them under their stable URLs. This bypasses stale browser/CDN HTTP-cache entries so the manifest ID and version JSON match the new worker.
- Add every required static shell asset to `APP_SHELL` in `service-worker.js` and copy it into `_site` in the Pages workflow.
- Do not cache uploaded audio blobs in the Cache API; they belong in IndexedDB and must remain separate from app releases.
- Preserve the update deferral while audio is active and the initial controller-claim handling in `registerPwa()`.