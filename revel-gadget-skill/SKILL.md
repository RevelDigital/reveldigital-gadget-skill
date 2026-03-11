---
name: revel-gadget
description: Scaffold and build Revel Digital digital signage gadgets using the @reveldigital/client-sdk. Use this skill whenever the user wants to create a gadget, widget, or interactive content for Revel Digital digital signage, or mentions Revel Digital gadgets, the Revel Digital client SDK, gadget preferences/prefs, the Gadgetizer tool, or deploying signage content to GitHub Pages. Also trigger when the user asks about building custom digital signage components, creating gadget XML definitions, or interfacing with a Revel Digital player. Covers Angular, React, Vue, and Vanilla JS scaffolding.
---

# Revel Digital Gadget Skill

This skill scaffolds a complete Revel Digital gadget project — a self-contained web application that runs inside the Revel Digital digital signage player. Gadgets communicate with the player via the `@reveldigital/client-sdk` and expose customizable preferences to signage designers through a `gadget.yaml` definition file.

## Workflow

### 1. Ask the user which framework they want

Present the user with a choice of framework before scaffolding:

- **React** — Vite + React + TypeScript
- **Angular** — Angular CLI + `@reveldigital/player-client` (Angular-native library with DI & RxJS)
- **Vue** — Vite + Vue 3 (JavaScript by default, TypeScript optional)
- **Vanilla JS** — Parcel + plain HTML/CSS/JS

Also ask:
- **Gadget name** — used for the project directory, `gadget.yaml` title, and package.json name
- **Will they host on GitHub Pages?** — if yes, include Gadgetizer integration and a GitHub Actions workflow

### 2. Scaffold the project

Read the appropriate framework reference file **before generating any code**:

| Framework  | Reference file                          |
|------------|-----------------------------------------|
| React      | `references/react.md`                   |
| Angular    | `references/angular.md`                 |
| Vue        | `references/vue.md`                     |
| Vanilla JS | `references/vanilla.md`                 |

Follow the reference file instructions precisely — they contain the exact file structure, dependencies, and sample code for each framework.

### 3. Common dependencies

**React, Vue, and Vanilla JS** projects use:

```json
"@reveldigital/client-sdk": "latest",
"@reveldigital/gadget-types": "latest"
```

The `client-sdk` is the runtime library. The `gadget-types` package provides TypeScript type definitions for the OpenSocial Gadgets API (the `gadgets.Prefs` class and related types). Include it even in JS projects — it powers IDE autocompletion.

**Angular** projects use a different, Angular-native library instead:

```json
"@reveldigital/player-client": "latest",
"@reveldigital/gadget-types": "latest"
```

The `player-client` library provides an injectable `PlayerClientService` with RxJS observables for player lifecycle events (`onReady$`, `onStart$`, `onStop$`, `onCommand$`). It also includes an `ng add` schematic for auto-configuring Angular projects. See the Angular reference file for details.

Every project should also include a `build:gadget` convenience script:

```json
"build:gadget": "npm run build && npx gadgetizer --build-only"
```

### 4. Always include `gadget.yaml`

This is the gadget definition file. It must be placed in the **project root** directory. The Gadgetizer CLI reads this file from the root when generating the gadget XML.

```yaml
title: My Gadget
title_url: https://mysupporturl.org
description: Describe the purpose of your gadget here
author: My Organization
background: transparent

requirements:
  - reveldigital
  - offline
  - webfont

locales:
  - messages: https://reveldigital.github.io/reveldigital-gadgets/ALL_ALL.xml

# Preferences provide customization options accessible at design time and runtime.
# Supported datatypes: string, enum, hidden, bool, style, list, color
prefs:
  - name: myStringPref
    display_name: Sample string preference
    datatype: string
    default_value: test string
    required: true

  - name: myBoolPref
    display_name: Sample boolean preference
    datatype: bool
    default_value: true
    required: true
    depends:
      - name: myEnumPref
        any_of:
          - values:
            - fast

  - name: myStylePref
    display_name: Sample style preference
    datatype: style
    default_value: font-family:Verdana;color:rgb(255, 255, 255);font-size:18px;
    required: true

  - name: myEnumPref
    display_name: Sample enum preference
    datatype: enum
    default_value: fast
    required: true
    multiple: false
    options:
      - value: fastest
        display_value: Fastest
      - value: fast
        display_value: Fast
      - value: medium
        display_value: Medium
```

The `depends` field makes a preference conditionally visible based on other preference values. Condition types: `any_of`, `all_of`, `none_of`.

### 5. Initial Gadgetizer setup (after GitHub repo creation)

After scaffolding the project and creating a GitHub repository, the developer must run the Gadgetizer **without** the `--build-only` flag for the initial setup:

```bash
npx gadgetizer
```

This interactive command:
- Detects the project type (React, Vue, Vanilla JS, or Angular)
- Prompts for the GitHub Pages deployment URL (e.g. `https://{user}.github.io/{repo}/`)
- Installs required SDK dependencies (`@reveldigital/client-sdk`, `@reveldigital/gadget-types`)
- Generates the initial gadget XML in the build output directory

**Important:** The GitHub repo should be created first so the correct GitHub Pages URL is known. This URL is embedded into the generated gadget XML and is how the Revel Digital CMS locates the gadget assets.

After this initial setup, all subsequent builds use the `--build-only` flag (which is what the `build:gadget` script does). The GitHub Actions workflow also uses `--build-only` since the deployment URL is already configured.

**Tell the developer:** After creating their GitHub repo, run `npx gadgetizer` once to complete the initial configuration, then use `npm run build:gadget` for all future builds.

### 6. GitHub Actions workflow (only if user wants GitHub Pages hosting)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Gadget to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Gadgetizer
        run: npx gadgetizer --build-only

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

Adjust `publish_dir` based on framework — see each reference file for the correct build output path.

### 7. Sample UX that demonstrates the SDK

The scaffolded app should include a working demo that:

1. Initializes the client SDK with `createPlayerClient()`
2. Fetches all available device/player data (device key, timezone, dimensions, SDK version, preview mode, etc.)
3. Reads gadget preferences using `client.getPrefs()` and displays them
4. Provides action buttons for `sendCommand()`, `track()`, and `finish()`
5. Shows loading/error states

This gives the developer a working starting point they can see running immediately.

## SDK API Quick Reference (React, Vue, Vanilla JS)

The following applies to `@reveldigital/client-sdk`. Angular uses `@reveldigital/player-client` which has the same methods but uses RxJS observables for events — see the Angular reference file.

```typescript
import { createPlayerClient, EventType } from "@reveldigital/client-sdk";

const client = createPlayerClient();

// Events
client.on(EventType.START, () => { /* player started */ });
client.on(EventType.STOP, () => { /* player stopped */ });
client.on(EventType.COMMAND, (data) => { /* command received */ });
client.off(EventType.START); // remove listener

// Device info
await client.getDeviceKey();            // unique device identifier
await client.getDeviceTime();           // device time in ISO8601
await client.getDeviceTimeZoneName();   // e.g. "America/Chicago"
await client.getDeviceTimeZoneID();     // timezone ID
await client.getDeviceTimeZoneOffset(); // offset in minutes
await client.getLanguageCode();         // e.g. "en"
await client.getDevice();               // full device details object
await client.isPreviewMode();           // true if running in CMS preview

// Player/content info
await client.getWidth();                // gadget zone width in pixels
await client.getHeight();               // gadget zone height in pixels
await client.getDuration();             // scheduled duration in ms
await client.getRevelRoot();            // Revel Digital root URL
await client.getCommandMap();           // available command mappings
await client.getSdkVersion();           // SDK version string

// Communication
client.callback(...args);                                  // send callback to player
client.sendCommand(name, arg);                              // send command to player
client.sendRemoteCommand(deviceKeys, name, arg);            // send to remote devices
client.finish();                                            // signal gadget is done

// Analytics
client.track(eventName, properties);    // track analytics event
client.timeEvent(eventName);            // start timing an event

// Preferences (Gadgets API — types from @reveldigital/gadget-types)
const prefs = client.getPrefs();
prefs.getString('myStringPref');
prefs.getBool('myBoolPref');
prefs.getFloat('myFloatPref');
prefs.getInt('myIntPref');
prefs.getArray('myListPref');
```

## Gadgetizer

The Gadgetizer CLI tool transforms a standard web app build into a Revel Digital gadget package. It reads `gadget.yaml` from the project root and generates the gadget XML definition in the build output directory.

### Initial setup (run once, after creating the GitHub repo):
```bash
npx gadgetizer
```

This interactive mode prompts for the GitHub Pages deployment URL and configures the project. It should be run **after** the GitHub repository has been created so the correct public URL (e.g. `https://{user}.github.io/{repo}/`) can be provided. The URL is embedded into the generated gadget XML.

### Subsequent builds (non-interactive):
```bash
npx gadgetizer --build-only
```

The `--build-only` flag skips interactive prompts and just regenerates the XML using the previously configured deployment URL. This is what the `build:gadget` script uses, and what runs in CI/CD pipelines.

The `build:gadget` convenience script in package.json chains the framework build with the non-interactive Gadgetizer step: `npm run build && npx gadgetizer --build-only`.

## `base` / `publicUrl` Configuration

For GitHub Pages hosting, the `base` (Vite) or `publicUrl` (Parcel) must be set so asset paths resolve correctly. The reference files use relative paths (`'./'`) by default, which works for local development.

When the developer runs `npx gadgetizer` (the initial interactive setup), the tool prompts for the GitHub Pages deployment URL and embeds it into the generated gadget XML. The `base`/`publicUrl` in the bundler config should match this URL for production builds. Relative paths (`'./'`) work in most cases, but for production GitHub Pages deployments the full URL (e.g. `'https://{user}.github.io/{repo}/'`) is more explicit and recommended.

## Important Notes

- After scaffolding, the developer must run `npx gadgetizer` (without `--build-only`) once to complete the initial interactive setup. This should happen after the GitHub repo is created so the deployment URL can be configured.
- Subsequent builds use `npm run build:gadget` which calls `npx gadgetizer --build-only` (non-interactive).
- The `gadget.yaml` file must be in the **project root** — the Gadgetizer reads it from there.
- The gadget XML is generated automatically by the Gadgetizer — never hand-write XML.
- Preferences defined in `gadget.yaml` become available at runtime via `client.getPrefs()`.
- The SDK works both inside and outside the Revel Digital player — when running standalone (during development), methods return sensible defaults or null.
- Always wrap SDK calls in `.catch()` for graceful error handling — some methods may throw when running outside the player.
- Always clean up event listeners on component unmount to avoid memory leaks.
- Use TypeScript types from the SDK: `import type { PlayerClient, EventType, IEventProperties, IOptions } from '@reveldigital/client-sdk';` (for Angular, import from `@reveldigital/player-client` instead)
