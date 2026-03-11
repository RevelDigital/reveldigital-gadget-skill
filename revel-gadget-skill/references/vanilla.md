# Vanilla JS Gadget Reference

Scaffold a Revel Digital gadget using Parcel + plain HTML/CSS/JS.

Parcel is a zero-config bundler that handles module resolution, bundling, and asset optimization out of the box. The official Revel Digital vanilla JS demo uses Parcel for this reason — it requires no config files beyond `package.json`.

## Project Structure

```
{gadget-name}/
├── src/
│   ├── index.html
│   ├── app.js
│   └── styles.css
├── .github/
│   └── workflows/
│       └── deploy.yml          # only if GitHub Pages hosting
├── gadget.yaml                 # primary — Gadgetizer reads from here
└── package.json
```

Note: With Parcel, the HTML entry point is `src/index.html` (not at the project root). The `<script>` tag in the HTML points to `app.js` and Parcel resolves everything from there.

## package.json

```json
{
  "name": "{gadget-name}",
  "version": "1.0.0",
  "private": true,
  "source": "src/index.html",
  "browserslist": "> 0.5%, last 2 versions, not dead",
  "scripts": {
    "start": "parcel src/index.html --public-url=/",
    "build": "parcel build src/index.html",
    "build:gadget": "npm run build && npx gadgetizer --build-only"
  },
  "dependencies": {
    "@reveldigital/client-sdk": "latest",
    "@reveldigital/gadget-types": "latest"
  },
  "devDependencies": {
    "@reveldigital/gadgetizer": "latest",
    "parcel": "^2.15.0"
  },
  "targets": {
    "default": {
      "publicUrl": "./"
    }
  }
}
```

The `publicUrl` in `targets.default` ensures relative asset paths. For production GitHub Pages, replace with the full URL: `'https://{user}.github.io/{repo}/'`.

## src/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="./styles.css" type="text/css" rel="stylesheet" />
    <title>{gadget-name}</title>
</head>
<body>
    <div class="app-container">
        <h1>Revel Digital Client SDK Demo</h1>

        <section class="section">
            <h2>Actions</h2>
            <div class="button-group">
                <button id="refreshBtn">Refresh Values</button>
                <button id="sendCommandBtn">Send Command</button>
                <button id="finishBtn">Finish</button>
                <button id="trackBtn">Track Event</button>
            </div>
        </section>

        <section class="section">
            <h2>Device Information</h2>
            <div class="data-grid">
                <div class="data-item"><label>Device Key:</label><span id="deviceKey"></span></div>
                <div class="data-item"><label>Device Time:</label><span id="deviceTime"></span></div>
                <div class="data-item"><label>Timezone Name:</label><span id="deviceTimeZoneName"></span></div>
                <div class="data-item"><label>Timezone ID:</label><span id="deviceTimeZoneID"></span></div>
                <div class="data-item"><label>Timezone Offset:</label><span id="deviceTimeZoneOffset"></span></div>
                <div class="data-item"><label>Language:</label><span id="languageCode"></span></div>
                <div class="data-item"><label>Preview Mode:</label><span id="isPreviewMode"></span></div>
            </div>
        </section>

        <section class="section">
            <h2>Player Information</h2>
            <div class="data-grid">
                <div class="data-item"><label>Width:</label><span id="width"></span></div>
                <div class="data-item"><label>Height:</label><span id="height"></span></div>
                <div class="data-item"><label>Duration:</label><span id="duration"></span></div>
                <div class="data-item"><label>SDK Version:</label><span id="sdkVersion"></span></div>
                <div class="data-item"><label>Revel Root:</label><span id="revelRoot"></span></div>
            </div>
        </section>

        <section class="section">
            <h2>Device Details</h2>
            <pre id="deviceDetails" class="json-display"></pre>
        </section>

        <section class="section">
            <h2>Command Map</h2>
            <pre id="commandMap" class="json-display"></pre>
        </section>
    </div>

    <script type="module" src="./app.js"></script>
</body>
</html>
```

## src/app.js

```javascript
import { createPlayerClient } from '@reveldigital/client-sdk';

const client = createPlayerClient();

async function updateValues() {
    document.getElementById('deviceTime').textContent = await client.getDeviceTime().catch(() => 'N/A');
    document.getElementById('deviceTimeZoneName').textContent = await client.getDeviceTimeZoneName().catch(() => 'N/A');
    document.getElementById('deviceTimeZoneID').textContent = await client.getDeviceTimeZoneID().catch(() => 'N/A');
    document.getElementById('deviceTimeZoneOffset').textContent = await client.getDeviceTimeZoneOffset().catch(() => 'N/A');
    document.getElementById('languageCode').textContent = await client.getLanguageCode().catch(() => 'N/A');
    document.getElementById('deviceKey').textContent = await client.getDeviceKey().catch(() => 'N/A');
    document.getElementById('revelRoot').textContent = await client.getRevelRoot().catch(() => 'N/A');
    document.getElementById('sdkVersion').textContent = await client.getSdkVersion().catch(() => 'N/A');
    document.getElementById('isPreviewMode').textContent = await client.isPreviewMode().catch(() => 'N/A');

    const device = await client.getDevice().catch(() => null);
    document.getElementById('deviceDetails').textContent = device ? JSON.stringify(device, null, 2) : 'N/A';

    document.getElementById('width').textContent = await client.getWidth().catch(() => 'N/A');
    document.getElementById('height').textContent = await client.getHeight().catch(() => 'N/A');
    document.getElementById('duration').textContent = await client.getDuration().catch(() => 'N/A');
    document.getElementById('commandMap').textContent = await client.getCommandMap().catch(() => 'N/A');
}

document.addEventListener('DOMContentLoaded', () => {
    updateValues();

    document.getElementById('refreshBtn').addEventListener('click', updateValues);

    document.getElementById('sendCommandBtn').addEventListener('click', () => {
        client.sendCommand('TestCommand', 'TestArg');
        alert('Command sent!');
    });

    document.getElementById('finishBtn').addEventListener('click', () => {
        client.finish();
        alert('Finish signal sent!');
    });

    document.getElementById('trackBtn').addEventListener('click', () => {
        client.track('TestEvent', { foo: 'bar' });
        alert('Event tracked!');
    });
});
```

## src/styles.css

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { margin: 0; background: transparent; }

.app-container { max-width: 700px; margin: 0 auto; padding: 2rem; font-family: system-ui, -apple-system, sans-serif; }
h1 { font-size: 1.6rem; margin-bottom: 1.5rem; border-bottom: 2px solid #333; padding-bottom: 0.5rem; }
h2 { font-size: 1.1rem; margin-bottom: 0.75rem; color: #555; }
.section { margin-bottom: 2rem; }
.button-group { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.button-group button {
  padding: 0.5rem 1rem; background: #0066cc; color: #fff;
  border: none; border-radius: 4px; cursor: pointer; font-size: 0.9rem;
}
.button-group button:hover { background: #0052a3; }
.data-grid { display: grid; gap: 0.5rem; }
.data-item { display: flex; justify-content: space-between; padding: 0.4rem 0; border-bottom: 1px solid #eee; }
.data-item label { font-weight: 600; }
.data-item span { font-family: monospace; color: #444; }
.json-display { background: #f5f5f5; padding: 1rem; border-radius: 4px; font-size: 0.8rem; overflow-x: auto; white-space: pre-wrap; }
```

## GitHub Actions

For Parcel projects, the default build output is `./dist`. The deploy workflow `publish_dir` should be `./dist`.

## Build & Development

```bash
npm install           # install dependencies
npm start             # start dev server (Parcel)
npm run build         # production build to ./dist
npx gadgetizer        # FIRST TIME ONLY: interactive setup (configure deployment URL)
npm run build:gadget  # subsequent builds: build + generate gadget XML (non-interactive)
```

Run `npx gadgetizer` (without `--build-only`) once after creating the GitHub repo to configure the deployment URL. After that, use `npm run build:gadget` for all subsequent builds.
