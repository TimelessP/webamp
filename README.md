# webamp

A single-page, local-first music player packaged as a Progressive Web App. Music files stay in the browser's IndexedDB; they are not uploaded to GitHub Pages or another server.

## Run locally

Serve this directory over HTTPS or `localhost` so the browser can register the service worker:

```sh
python3 -m http.server 8000
```

Then open `http://localhost:8000/`. The PWA shell is cached for offline launches after its first successful load. Music-library data belongs to the exact site origin and browser profile, so it does not transfer between `file://`, localhost, and the deployed site.

## Install and playback

Use the browser's install prompt on desktop or Android. On iPhone and iPad, open the site in Safari and choose **Share → Add to Home Screen**. Media Session metadata and play/pause/track actions are provided for supported lock-screen controls. Background playback depends on the browser and operating system; keep playback started by a user gesture, and do not force-quit the browser or PWA.

Playback uses the native audio element by default. The equalizer uses Web Audio only after an EQ band is changed, since mobile browsers may suspend Web Audio when backgrounded.

## Publish updates

Push to `main` to run `.github/workflows/deploy-pages.yml`. The workflow publishes the static app to GitHub Pages, stamps that commit's SHA into the deployed service worker, and displays its short ID in the app footer. On a later app launch or foreground, the browser checks for the new worker, refreshes the app shell, and reloads to the new version when playback is idle. If playback is active during an update, it is allowed to continue until playback is paused.

The app's saved tracks are separate from the deploy cache and are not removed by an app update. Browser storage can still be cleared by the user or reclaimed by the browser; the app requests persistent storage where supported.