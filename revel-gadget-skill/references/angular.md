# Angular Gadget Reference

Scaffold a Revel Digital gadget using Angular + the **`@reveldigital/player-client`** library.

**Important:** Angular gadgets do NOT use `@reveldigital/client-sdk`. Instead, they use `@reveldigital/player-client`, an Angular-native library that provides an injectable `PlayerClientService` with RxJS observables for player lifecycle events. It also includes an `ng add` schematic that auto-configures the project.

## Scaffolding Approach

The recommended approach for Angular is to use the library's schematic, which configures everything automatically:

```bash
ng new my-gadget --directory ./ --standalone
ng add @reveldigital/player-client@latest
```

The `--standalone` flag tells Angular 17 to generate a standalone component app (no NgModule). The schematic adds the library, creates the `assets/gadget.yaml`, and configures build scripts (`build:gadget`, `deploy:gadget`).

**Critical — fix the builder after `ng new`:** Angular 17's `ng new` generates `angular.json` with the `application` builder (`@angular-devkit/build-angular:application`), which outputs to `dist/browser/` instead of `dist/{gadget-name}/`. This breaks the deploy script and Gadgetizer. After scaffolding, you **must** replace the generated `angular.json` with the one shown below, which uses the `browser` builder (`@angular-devkit/build-angular:browser`) so the output goes to `dist/{gadget-name}/`.

If scaffolding files manually (e.g. from this skill), create the structure below.

## Project Structure

```
{gadget-name}/
├── src/
│   ├── app/
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   ├── app.component.css
│   │   └── app.config.ts
│   ├── assets/
│   │   └── gadget.yaml         # schematic places it here
│   ├── index.html
│   ├── main.ts
│   └── styles.css
├── .github/
│   └── workflows/
│       └── deploy.yml          # only if GitHub Pages hosting
├── gadget.yaml                 # project root — Gadgetizer reads from here
├── .browserslistrc
├── angular.json
├── package.json
└── tsconfig.json
```

Note: The `gadget.yaml` must exist in the project root for the Gadgetizer, AND in `src/assets/` for the Angular build assets pipeline.

## package.json

```json
{
  "name": "{gadget-name}",
  "version": "1.0.0",
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "build:gadget": "npm run build && npx gadgetizer --build-only",
    "deploy:gadget": "npm run build:gadget && npx angular-cli-ghpages --dir=dist/{gadget-name}"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "^17.0.0",
    "@angular/common": "^17.0.0",
    "@angular/compiler": "^17.0.0",
    "@angular/core": "^17.0.0",
    "@angular/platform-browser": "^17.0.0",
    "@angular/platform-browser-dynamic": "^17.0.0",
    "@reveldigital/player-client": "latest",
    "@reveldigital/gadget-types": "latest",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.14.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^17.0.0",
    "@angular/cli": "^17.0.0",
    "@angular/compiler-cli": "^17.0.0",
    "@reveldigital/gadgetizer": "latest",
    "angular-cli-ghpages": "^2.0.0",
    "typescript": "~5.2.0"
  }
}
```

Key differences from React/Vue/Vanilla:
- Uses `@reveldigital/player-client` instead of `@reveldigital/client-sdk`
- Includes `angular-cli-ghpages` for the `deploy:gadget` script
- Does NOT need `@reveldigital/client-sdk`

## angular.json

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "{gadget-name}": {
      "projectType": "application",
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "outputPath": "dist/{gadget-name}",
            "index": "src/index.html",
            "main": "src/main.ts",
            "tsConfig": "tsconfig.json",
            "baseHref": "./",
            "assets": [
              { "glob": "**/*", "input": "src/assets", "output": "/assets" },
              { "glob": "gadget.yaml", "input": "src/assets", "output": "/" }
            ],
            "styles": ["src/styles.css"],
            "scripts": []
          },
          "configurations": {
            "production": {
              "budgets": [
                { "type": "initial", "maximumWarning": "500kB", "maximumError": "1MB" }
              ],
              "outputHashing": "all"
            },
            "development": {
              "optimization": false,
              "extractLicenses": false,
              "sourceMap": true
            }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": { "buildTarget": "{gadget-name}:build:production" },
            "development": { "buildTarget": "{gadget-name}:build:development" }
          },
          "defaultConfiguration": "development"
        }
      }
    }
  }
}
```

The `gadget.yaml` in `src/assets/` is copied to the build output root via the assets config so the Gadgetizer can find it.

**Important:** The `browser` builder outputs directly to `dist/{gadget-name}/` (no `/browser/` subfolder). Do NOT use the `application` builder — it outputs to `dist/browser/` which breaks the deploy script and Gadgetizer. Always use `@angular-devkit/build-angular:browser` as shown above.

## tsconfig.json

```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/out-tsc",
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "node",
    "importHelpers": true,
    "sourceMap": true,
    "declaration": false,
    "experimentalDecorators": true,
    "lib": ["ES2022", "dom"],
    "skipLibCheck": true,
    "useDefineForClassFields": false
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

## .browserslistrc

This file must be created in the project root. It ensures the Angular build targets a broad range of devices compatible with Angular 17, which is important because gadgets run on diverse signage hardware.

```
last 2 Chrome versions
last 1 Firefox version
last 2 Edge major versions
last 2 Safari major versions
last 2 iOS major versions
Firefox ESR
not dead
```

## src/index.html

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{gadget-name}</title>
  <base href="./">
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

## src/main.ts

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

## src/app/app.config.ts

```typescript
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: []
};
```

## src/app/app.component.ts

The Angular library exposes `PlayerClientService` as an injectable. Events use RxJS Subjects (`onReady$`, `onStart$`, `onStop$`, `onCommand$`) instead of the callback-based `client.on()` pattern used by the client-sdk.

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Subscription } from 'rxjs';
import { PlayerClientService } from '@reveldigital/player-client';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit, OnDestroy {
  // Device/player data
  data: Record<string, unknown> = {};
  prefs: Record<string, unknown> = {};
  loading = true;
  error: string | null = null;
  isReady = false;
  isStarted = false;

  private subscriptions = new Subscription();

  constructor(public client: PlayerClientService) {
    // Subscribe to player lifecycle events (RxJS observables)
    this.subscriptions.add(
      this.client.onReady$.subscribe((val) => {
        this.isReady = val;
        console.log(val ? 'Ready' : 'Not ready');
      })
    );

    this.subscriptions.add(
      this.client.onStart$.subscribe(() => {
        this.isStarted = true;
        console.log('onStart');
      })
    );

    this.subscriptions.add(
      this.client.onStop$.subscribe(() => {
        this.isStarted = false;
        console.log('onStop');
      })
    );

    this.subscriptions.add(
      this.client.onCommand$.subscribe((cmd) => {
        console.log(`onCommand: ${cmd.name}, ${cmd.arg}`);
      })
    );

    // Read preferences immediately (synchronous)
    const p = this.client.getPrefs();
    if (p) {
      this.prefs = {
        myStringPref: p.getString('myStringPref') ?? 'N/A',
        myBoolPref: p.getBool('myBoolPref') ?? 'N/A',
        myStylePref: p.getString('myStylePref') ?? 'N/A',
        myEnumPref: p.getString('myEnumPref') ?? 'N/A'
      };
    }
  }

  async ngOnInit() {
    try {
      // Fetch all device/player data
      const [
        deviceKey, deviceTime, deviceTimeZoneName, deviceTimeZoneID,
        deviceTimeZoneOffset, languageCode, revelRoot, commandMap,
        isPreviewMode, device, width, height, duration, sdkVersion
      ] = await Promise.all([
        this.client.getDeviceKey().catch(() => null),
        this.client.getDeviceTime().catch(() => null),
        this.client.getDeviceTimeZoneName().catch(() => null),
        this.client.getDeviceTimeZoneID().catch(() => null),
        this.client.getDeviceTimeZoneOffset().catch(() => null),
        this.client.getLanguageCode().catch(() => null),
        this.client.getRevelRoot().catch(() => null),
        this.client.getCommandMap().catch(() => null),
        this.client.isPreviewMode().catch(() => false),
        this.client.getDevice().catch(() => null),
        this.client.getWidth().catch(() => null),
        this.client.getHeight().catch(() => null),
        this.client.getDuration().catch(() => null),
        this.client.getSdkVersion().catch(() => 'Unknown')
      ]);

      this.data = {
        deviceKey, deviceTime, deviceTimeZoneName, deviceTimeZoneID,
        deviceTimeZoneOffset, languageCode, revelRoot, commandMap,
        isPreviewMode: Boolean(isPreviewMode), device, width, height,
        duration, sdkVersion
      };
    } catch (err: unknown) {
      this.error = err instanceof Error ? err.message : 'Unknown error';
    } finally {
      this.loading = false;
    }
  }

  handleSendCommand() {
    this.client.sendCommand('test', 'Hello from Angular gadget!');
    alert('Command sent!');
  }

  handleTrack() {
    this.client.track('user_interaction', {
      action: 'button_click',
      timestamp: new Date().toISOString()
    });
    alert('Event tracked!');
  }

  handleFinish() {
    this.client.finish();
    alert('Finish signal sent!');
  }

  ngOnDestroy() {
    this.subscriptions.unsubscribe();
  }
}
```

## src/app/app.component.html

```html
<div class="app-container">
  <h1>Revel Digital Angular Gadget Demo</h1>

  <div *ngIf="loading">Loading...</div>
  <div *ngIf="error" class="error">Error: {{ error }}</div>

  <ng-container *ngIf="!loading && !error">
    <section class="section">
      <h2>Player Status</h2>
      <div class="data-grid">
        <div class="data-item"><label>Ready:</label><span>{{ isReady ? 'Yes' : 'No' }}</span></div>
        <div class="data-item"><label>Started:</label><span>{{ isStarted ? 'Yes' : 'No' }}</span></div>
        <div class="data-item"><label>Preview Mode:</label><span>{{ data['isPreviewMode'] ? 'Yes' : 'No' }}</span></div>
      </div>
    </section>

    <section class="section">
      <h2>Actions</h2>
      <div class="button-group">
        <button (click)="handleSendCommand()">Send Command</button>
        <button (click)="handleTrack()">Track Event</button>
        <button (click)="handleFinish()">Finish</button>
      </div>
    </section>

    <section class="section">
      <h2>Device Information</h2>
      <div class="data-grid">
        <div class="data-item"><label>Device Key:</label><span>{{ data['deviceKey'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>Device Time:</label><span>{{ data['deviceTime'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>Timezone:</label><span>{{ data['deviceTimeZoneName'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>Language:</label><span>{{ data['languageCode'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>SDK Version:</label><span>{{ data['sdkVersion'] ?? 'N/A' }}</span></div>
      </div>
    </section>

    <section class="section">
      <h2>Display</h2>
      <div class="data-grid">
        <div class="data-item"><label>Width:</label><span>{{ data['width'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>Height:</label><span>{{ data['height'] ?? 'N/A' }}</span></div>
        <div class="data-item"><label>Duration:</label><span>{{ data['duration'] ?? 'N/A' }}</span></div>
      </div>
    </section>

    <section *ngIf="prefs" class="section">
      <h2>Preferences (from gadget.yaml)</h2>
      <div class="data-grid">
        <div class="data-item"><label>myStringPref:</label><span>{{ prefs['myStringPref'] }}</span></div>
        <div class="data-item"><label>myBoolPref:</label><span>{{ prefs['myBoolPref'] }}</span></div>
        <div class="data-item"><label>myEnumPref:</label><span>{{ prefs['myEnumPref'] }}</span></div>
        <div class="data-item"><label>myStylePref:</label><span>{{ prefs['myStylePref'] }}</span></div>
      </div>
    </section>
  </ng-container>
</div>
```

## src/app/app.component.css

```css
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
```

## src/styles.css

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body { margin: 0; background: transparent; }
```

## PlayerClientService API (Angular-specific)

The API surface is nearly identical to the `client-sdk`, but events use RxJS:

### RxJS Observables (replaces `client.on()`)

```typescript
client.onReady$    // Subject<boolean> — player ready state
client.onStart$    // Subject<void> — player started
client.onStop$     // Subject<void> — player stopped
client.onCommand$  // Subject<ICommand> — command received ({ name, arg })
client.onConfig$   // Subject — config changes (preview mode)
```

### Additional method not in client-sdk

```typescript
await client.applyConfig(prefs)   // apply config in preview mode only
```

All other methods (`getDeviceKey()`, `getDeviceTime()`, `getPrefs()`, `sendCommand()`, `track()`, `finish()`, etc.) are identical to the client-sdk.

## GitHub Actions

With the `browser` builder, the build output lands in `dist/{gadget-name}/`. Update the workflow:

```yaml
      - name: Build
        run: npm run build -- --configuration production

      - name: Gadgetizer
        run: npx gadgetizer --build-only

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist/{gadget-name}
```

Or use the built-in deploy script: `npm run deploy:gadget`.

## Build & Development

```bash
npm install                       # install dependencies
ng serve                          # start dev server
ng build                          # production build
npx gadgetizer                    # FIRST TIME ONLY: interactive setup (configure deployment URL)
npm run build:gadget              # subsequent builds: build + generate gadget XML (non-interactive)
npm run deploy:gadget             # build + gadgetize + deploy to GitHub Pages
```

Run `npx gadgetizer` (without `--build-only`) once after creating the GitHub repo to configure the deployment URL. After that, use `npm run build:gadget` for all subsequent builds.
