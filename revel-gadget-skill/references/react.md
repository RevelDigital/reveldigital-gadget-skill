# React Gadget Reference

Scaffold a Revel Digital gadget using Vite + React + TypeScript.

## Project Structure

```
{gadget-name}/
├── src/
│   ├── App.tsx
│   ├── App.css
│   ├── main.tsx
│   ├── index.css
│   └── vite-env.d.ts
├── .github/
│   └── workflows/
│       └── deploy.yml          # only if GitHub Pages hosting
├── gadget.yaml                 # primary — Gadgetizer reads from here
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
└── vite.config.ts
```

## package.json

```json
{
  "name": "{gadget-name}",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview",
    "build:gadget": "npm run build && npx gadgetizer --build-only"
  },
  "dependencies": {
    "@reveldigital/client-sdk": "latest",
    "@reveldigital/gadget-types": "latest",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@reveldigital/gadgetizer": "latest",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.4.1",
    "typescript": "~5.7.2",
    "vite": "^6.3.5"
  }
}
```

## vite.config.ts

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: './',
  build: {
    outDir: 'dist',
  },
})
```

The `base: './'` ensures relative asset paths. For production GitHub Pages, replace with the full URL: `'https://{user}.github.io/{repo}/'`.

## tsconfig.json

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

## tsconfig.app.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

Note: `isolatedModules` is intentionally omitted. The SDK's `EventType` is a `const enum` which is incompatible with `isolatedModules`.

## tsconfig.node.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2023"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["vite.config.ts"]
}
```

## src/vite-env.d.ts

```typescript
/// <reference types="vite/client" />
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
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

## src/main.tsx

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

## src/App.tsx

This demo fetches all available SDK data and displays preferences. It wraps every async SDK call in `.catch()` to handle gracefully when running outside the player.

```tsx
import { useState, useEffect } from 'react'
import { createPlayerClient } from '@reveldigital/client-sdk'
import './App.css'

interface ClientData {
  deviceKey: string | null
  deviceTime: string | null
  deviceTimeZoneName: string | null
  deviceTimeZoneID: string | null
  deviceTimeZoneOffset: number | null
  languageCode: string | null
  revelRoot: string | null
  commandMap: string | null
  isPreviewMode: boolean
  device: Record<string, unknown> | null
  width: number | null
  height: number | null
  duration: number | null
  sdkVersion: string
}

function App() {
  const [clientData, setClientData] = useState<ClientData | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const loadClientData = async () => {
      try {
        const client = createPlayerClient()

        const [
          deviceKey, deviceTime, deviceTimeZoneName, deviceTimeZoneID,
          deviceTimeZoneOffset, languageCode, revelRoot, commandMap,
          isPreviewMode, device, width, height, duration, sdkVersion
        ] = await Promise.all([
          client.getDeviceKey().catch(() => null),
          client.getDeviceTime().catch(() => null),
          client.getDeviceTimeZoneName().catch(() => null),
          client.getDeviceTimeZoneID().catch(() => null),
          client.getDeviceTimeZoneOffset().catch(() => null),
          client.getLanguageCode().catch(() => null),
          client.getRevelRoot().catch(() => null),
          client.getCommandMap().catch(() => null),
          client.isPreviewMode().catch(() => false),
          client.getDevice().catch(() => null),
          client.getWidth().catch(() => null),
          client.getHeight().catch(() => null),
          client.getDuration().catch(() => null),
          client.getSdkVersion().catch(() => 'Unknown')
        ])

        setClientData({
          deviceKey: deviceKey as string | null,
          deviceTime: deviceTime as string | null,
          deviceTimeZoneName: deviceTimeZoneName as string | null,
          deviceTimeZoneID: deviceTimeZoneID as string | null,
          deviceTimeZoneOffset: deviceTimeZoneOffset as number | null,
          languageCode: languageCode as string | null,
          revelRoot: revelRoot as string | null,
          commandMap: commandMap as string | null,
          isPreviewMode: Boolean(isPreviewMode),
          device: (device && typeof device === 'object') ? device as Record<string, unknown> : null,
          width: width as number | null,
          height: height as number | null,
          duration: duration as number | null,
          sdkVersion: (sdkVersion as string) || 'Unknown',
        })
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error')
      } finally {
        setLoading(false)
      }
    }
    loadClientData()
  }, [])

  const handleSendCommand = () => {
    try {
      const client = createPlayerClient()
      client.sendCommand('test', 'Hello from React gadget!')
      alert('Command sent!')
    } catch (err) {
      alert('Error: ' + (err instanceof Error ? err.message : 'Unknown'))
    }
  }

  const handleTrackEvent = () => {
    try {
      const client = createPlayerClient()
      client.track('user_interaction', {
        action: 'button_click',
        timestamp: new Date().toISOString()
      })
      alert('Event tracked!')
    } catch (err) {
      alert('Error: ' + (err instanceof Error ? err.message : 'Unknown'))
    }
  }

  const handleFinish = () => {
    try {
      const client = createPlayerClient()
      client.finish()
      alert('Finish signal sent!')
    } catch (err) {
      alert('Error: ' + (err instanceof Error ? err.message : 'Unknown'))
    }
  }

  if (loading) return <div className="app-container"><h1>Revel Digital Client SDK Demo</h1><p>Loading...</p></div>
  if (error) return <div className="app-container"><h1>Revel Digital Client SDK Demo</h1><p className="error">Error: {error}</p></div>

  // Read preferences
  let prefs: ReturnType<ReturnType<typeof createPlayerClient>['getPrefs']> | null = null
  try { prefs = createPlayerClient().getPrefs() } catch { prefs = null }

  return (
    <div className="app-container">
      <h1>Revel Digital Client SDK Demo</h1>

      <section className="section">
        <h2>Actions</h2>
        <div className="button-group">
          <button onClick={handleSendCommand}>Send Command</button>
          <button onClick={handleTrackEvent}>Track Event</button>
          <button onClick={handleFinish}>Finish</button>
        </div>
      </section>

      <section className="section">
        <h2>Device Information</h2>
        <div className="data-grid">
          <div className="data-item"><label>Device Key:</label><span>{clientData?.deviceKey ?? 'N/A'}</span></div>
          <div className="data-item"><label>Device Time:</label><span>{clientData?.deviceTime ?? 'N/A'}</span></div>
          <div className="data-item"><label>Timezone:</label><span>{clientData?.deviceTimeZoneName ?? 'N/A'}</span></div>
          <div className="data-item"><label>Timezone ID:</label><span>{clientData?.deviceTimeZoneID ?? 'N/A'}</span></div>
          <div className="data-item"><label>Timezone Offset:</label><span>{clientData?.deviceTimeZoneOffset ?? 'N/A'}</span></div>
          <div className="data-item"><label>Language:</label><span>{clientData?.languageCode ?? 'N/A'}</span></div>
          <div className="data-item"><label>Preview Mode:</label><span>{clientData?.isPreviewMode ? 'Yes' : 'No'}</span></div>
        </div>
      </section>

      <section className="section">
        <h2>Player Information</h2>
        <div className="data-grid">
          <div className="data-item"><label>Width:</label><span>{clientData?.width != null ? `${clientData.width}px` : 'N/A'}</span></div>
          <div className="data-item"><label>Height:</label><span>{clientData?.height != null ? `${clientData.height}px` : 'N/A'}</span></div>
          <div className="data-item"><label>Duration:</label><span>{clientData?.duration != null ? `${clientData.duration}ms` : 'N/A'}</span></div>
          <div className="data-item"><label>SDK Version:</label><span>{clientData?.sdkVersion ?? 'N/A'}</span></div>
          <div className="data-item"><label>Revel Root:</label><span>{clientData?.revelRoot ?? 'N/A'}</span></div>
        </div>
      </section>

      {clientData?.device && Object.keys(clientData.device).length > 0 && (
        <section className="section">
          <h2>Device Details</h2>
          <pre className="json-display">{JSON.stringify(clientData.device, null, 2)}</pre>
        </section>
      )}

      {prefs && (
        <section className="section">
          <h2>Preferences (from gadget.yaml)</h2>
          <div className="data-grid">
            <div className="data-item"><label>myStringPref:</label><span>{prefs.getString('myStringPref') ?? 'N/A'}</span></div>
            <div className="data-item"><label>myBoolPref:</label><span>{String(prefs.getBool('myBoolPref') ?? 'N/A')}</span></div>
            <div className="data-item"><label>myEnumPref:</label><span>{prefs.getString('myEnumPref') ?? 'N/A'}</span></div>
            <div className="data-item"><label>myStylePref:</label><span>{prefs.getString('myStylePref') ?? 'N/A'}</span></div>
          </div>
        </section>
      )}
    </div>
  )
}

export default App
```

## src/App.css

```css
.app-container {
  max-width: 700px;
  margin: 0 auto;
  padding: 2rem;
  font-family: system-ui, -apple-system, sans-serif;
}

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

.json-display { background: #f5f5f5; padding: 1rem; border-radius: 4px; font-size: 0.8rem; overflow-x: auto; }
.error { color: #d32f2f; }
```

## src/index.css

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { margin: 0; background: transparent; }
```

## GitHub Actions

The deploy workflow `publish_dir` should be `./dist` for Vite React projects.

## Build & Development

```bash
npm install           # install dependencies
npm run dev           # start dev server
npm run build         # production build to ./dist
npm run build:gadget  # build + generate gadget XML
```
