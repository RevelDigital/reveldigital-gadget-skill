# Vue 3 Gadget Reference

Scaffold a Revel Digital gadget using Vite + Vue 3 + JavaScript (TypeScript optional).

## Project Structure

```
{gadget-name}/
├── src/
│   ├── App.vue
│   ├── main.js
│   └── style.css
├── .github/
│   └── workflows/
│       └── deploy.yml          # only if GitHub Pages hosting
├── gadget.yaml                 # primary — Gadgetizer reads from here
├── index.html
├── jsconfig.json
├── package.json
└── vite.config.js
```

## package.json

```json
{
  "name": "{gadget-name}",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "build:gadget": "npm run build && npx gadgetizer --build-only"
  },
  "dependencies": {
    "@reveldigital/client-sdk": "latest",
    "@reveldigital/gadget-types": "latest",
    "vue": "^3.5.0"
  },
  "devDependencies": {
    "@reveldigital/gadgetizer": "latest",
    "@vitejs/plugin-vue": "^5.2.0",
    "vite": "^6.3.5"
  }
}
```

## vite.config.js

```javascript
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  base: './',
  build: {
    outDir: 'dist',
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

The `base: './'` ensures relative asset paths. For production GitHub Pages, replace with the full URL.

## jsconfig.json

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "exclude": ["node_modules", "dist"]
}
```

## index.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{gadget-name}</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

## src/main.js

```javascript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```

## src/App.vue

This demo fetches all available SDK data and displays preferences, matching the pattern from the official demo repos.

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { createPlayerClient } from '@reveldigital/client-sdk'

const sdkData = ref({
  device: {},
  location: {},
  display: {},
  system: {},
  prefs: null
})
const isLoading = ref(true)
const error = ref(null)

let clientInstance = null

onMounted(async () => {
  try {
    const client = createPlayerClient()
    clientInstance = client

    const device = await client.getDevice().catch(() => null)
    sdkData.value.device = {
      deviceDetails: device ? JSON.stringify(device, null, 2) : 'N/A',
      deviceKey: await client.getDeviceKey().catch(() => 'N/A'),
      sdkVersion: await client.getSdkVersion().catch(() => 'N/A'),
      isPreviewMode: await client.isPreviewMode().catch(() => false)
    }

    sdkData.value.location = {
      deviceTime: await client.getDeviceTime().catch(() => 'N/A'),
      timezoneName: await client.getDeviceTimeZoneName().catch(() => 'N/A'),
      timezoneId: await client.getDeviceTimeZoneID().catch(() => 'N/A'),
      timezoneOffset: await client.getDeviceTimeZoneOffset().catch(() => 'N/A'),
      languageCode: await client.getLanguageCode().catch(() => 'N/A')
    }

    sdkData.value.display = {
      width: await client.getWidth().catch(() => 'N/A'),
      height: await client.getHeight().catch(() => 'N/A'),
      duration: await client.getDuration().catch(() => 'N/A')
    }

    sdkData.value.system = {
      revelRoot: await client.getRevelRoot().catch(() => 'N/A'),
      commandMap: await client.getCommandMap().catch(() => 'N/A')
    }

    try {
      sdkData.value.prefs = client.getPrefs()
    } catch {
      sdkData.value.prefs = null
    }
  } catch (err) {
    error.value = err.message || 'Unknown error'
  } finally {
    isLoading.value = false
  }
})

function handleSendCommand() {
  try {
    clientInstance?.sendCommand('test', 'Hello from Vue gadget!')
    alert('Command sent!')
  } catch (err) {
    alert('Error: ' + err.message)
  }
}

function handleTrackEvent() {
  try {
    clientInstance?.track('user_interaction', {
      action: 'button_click',
      timestamp: new Date().toISOString()
    })
    alert('Event tracked!')
  } catch (err) {
    alert('Error: ' + err.message)
  }
}

function handleFinish() {
  try {
    clientInstance?.finish()
    alert('Finish signal sent!')
  } catch (err) {
    alert('Error: ' + err.message)
  }
}
</script>

<template>
  <div class="app-container">
    <h1>Revel Digital Client SDK Demo</h1>

    <div v-if="isLoading">Loading...</div>
    <div v-else-if="error" class="error">Error: {{ error }}</div>
    <template v-else>
      <section class="section">
        <h2>Actions</h2>
        <div class="button-group">
          <button @click="handleSendCommand">Send Command</button>
          <button @click="handleTrackEvent">Track Event</button>
          <button @click="handleFinish">Finish</button>
        </div>
      </section>

      <section class="section">
        <h2>Device Information</h2>
        <div class="data-grid">
          <div class="data-item"><label>Device Key:</label><span>{{ sdkData.device.deviceKey }}</span></div>
          <div class="data-item"><label>SDK Version:</label><span>{{ sdkData.device.sdkVersion }}</span></div>
          <div class="data-item"><label>Preview Mode:</label><span>{{ sdkData.device.isPreviewMode ? 'Yes' : 'No' }}</span></div>
        </div>
      </section>

      <section class="section">
        <h2>Location & Time</h2>
        <div class="data-grid">
          <div class="data-item"><label>Device Time:</label><span>{{ sdkData.location.deviceTime }}</span></div>
          <div class="data-item"><label>Timezone:</label><span>{{ sdkData.location.timezoneName }}</span></div>
          <div class="data-item"><label>Timezone ID:</label><span>{{ sdkData.location.timezoneId }}</span></div>
          <div class="data-item"><label>Offset:</label><span>{{ sdkData.location.timezoneOffset }}</span></div>
          <div class="data-item"><label>Language:</label><span>{{ sdkData.location.languageCode }}</span></div>
        </div>
      </section>

      <section class="section">
        <h2>Display</h2>
        <div class="data-grid">
          <div class="data-item"><label>Width:</label><span>{{ sdkData.display.width }}</span></div>
          <div class="data-item"><label>Height:</label><span>{{ sdkData.display.height }}</span></div>
          <div class="data-item"><label>Duration:</label><span>{{ sdkData.display.duration }}</span></div>
        </div>
      </section>

      <section v-if="sdkData.prefs" class="section">
        <h2>Preferences (from gadget.yaml)</h2>
        <div class="data-grid">
          <div class="data-item"><label>myStringPref:</label><span>{{ sdkData.prefs.getString('myStringPref') ?? 'N/A' }}</span></div>
          <div class="data-item"><label>myBoolPref:</label><span>{{ sdkData.prefs.getBool('myBoolPref') ?? 'N/A' }}</span></div>
          <div class="data-item"><label>myEnumPref:</label><span>{{ sdkData.prefs.getString('myEnumPref') ?? 'N/A' }}</span></div>
          <div class="data-item"><label>myStylePref:</label><span>{{ sdkData.prefs.getString('myStylePref') ?? 'N/A' }}</span></div>
        </div>
      </section>
    </template>
  </div>
</template>

<style scoped>
.app-container { max-width: 700px; margin: 0 auto; padding: 2rem; font-family: system-ui, -apple-system, sans-serif; }
h1 { font-size: 1.6rem; margin-bottom: 1.5rem; border-bottom: 2px solid #333; padding-bottom: 0.5rem; }
h2 { font-size: 1.1rem; margin-bottom: 0.75rem; color: #555; }
.section { margin-bottom: 2rem; }
.button-group { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.button-group button { padding: 0.5rem 1rem; background: #0066cc; color: #fff; border: none; border-radius: 4px; cursor: pointer; }
.button-group button:hover { background: #0052a3; }
.data-grid { display: grid; gap: 0.5rem; }
.data-item { display: flex; justify-content: space-between; padding: 0.4rem 0; border-bottom: 1px solid #eee; }
.data-item label { font-weight: 600; }
.data-item span { font-family: monospace; color: #444; }
.error { color: #d32f2f; }
</style>
```

## src/style.css

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { margin: 0; background: transparent; }
```

## GitHub Actions

The deploy workflow `publish_dir` should be `./dist` for Vite Vue projects.

## Build & Development

```bash
npm install           # install dependencies
npm run dev           # start dev server
npm run build         # production build to ./dist
npx gadgetizer        # FIRST TIME ONLY: interactive setup (configure deployment URL)
npm run build:gadget  # subsequent builds: build + generate gadget XML (non-interactive)
```

Run `npx gadgetizer` (without `--build-only`) once after creating the GitHub repo to configure the deployment URL. After that, use `npm run build:gadget` for all subsequent builds.
