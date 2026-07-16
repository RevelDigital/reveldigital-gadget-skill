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

Also read these shared references:

| Always read | Purpose |
|-------------|---------|
| `references/signage.md` | Theme tokens, accessibility (508 / WCAG), and distance readability — apply to every scaffold |
| `references/datatable.md` | **When the gadget displays CMS data-table content** — live `createDataTable()` reads, filtering/sorting, and real-time row events |

**Apply `references/signage.md` to every scaffold** (not optional): ship `theme.css` and import it
globally, recolor `--brand` to the user's brand (and honor any `color`/`style` gadget preference),
use semantic landmarks with `aria-live` for dynamic content (clocks, tickers), zone-relative readable
type, and `prefers-reduced-motion` guards.

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

**Use the Client SDK, not browser equivalents.** The player exposes device context through the SDK
(`@reveldigital/client-sdk`; Angular: `@reveldigital/player-client`). Always prefer it over the
browser API, and fall back to the browser only when off-device. Off-device, the async methods below
already degrade to a mock and resolve `null` — the one call that needs guarding is `getPrefs()`,
which throws synchronously and must be wrapped in `try`/`catch` (see Important Notes):

| Need | Use (SDK) | Instead of |
|------|-----------|------------|
| Time / clock | `getDeviceTime()` + `getDeviceTimeZoneName()` (format in the device timezone) | `new Date()` rendered in the browser's timezone |
| Timezone | `getDeviceTimeZoneName()` / `getDeviceTimeZoneID()` / `getDeviceTimeZoneOffset()` | `Intl…resolvedOptions().timeZone` |
| Language / locale | `getLanguageCode()` (set `<html lang>`) | `navigator.language` |
| Location (geo) | `getDevice().location` (lat/long, city/state) | `navigator.geolocation` |
| Zone size | `getWidth()` / `getHeight()` | `window.innerWidth/innerHeight` |
| Scheduled duration | `getDuration()` | — |
| Preview vs. live | `isPreviewMode()` | assuming live |
| Live CMS data | `createDataTable()` (see `references/datatable.md`) | hard-coded data |
| Commands | `on(EventType.COMMAND, …)` / `sendCommand()` | — |
| Analytics | `track()` / `timeEvent()` | — |

> **For a clock gadget**, don't render a bare `new Date()`. Call `getDeviceTime()` once to compute
> the device↔local offset, tick locally each second from that offset (re-sync every few minutes for
> drift), and format with `getDeviceTimeZoneName()` so it shows the device's wall-clock time
> regardless of where the browser runs. Off-device, the offset is 0 → local time.

The scaffolded app should include a working demo that:

1. Initializes the client SDK with `createPlayerClient()`
2. Fetches all available device/player data (device key, timezone, dimensions, SDK version, preview mode, etc.) — via SDK methods, not browser APIs
3. Reads gadget preferences via a guarded `getPrefs()` (`try`/`catch`, each read falling back to its
   `gadget.yaml` default) and displays them — **all four scaffolds must do this**, including Vanilla JS
4. Provides action buttons for `sendCommand()`, `track()`, and `finish()`
5. Shows loading/error states
6. Handles the lifecycle: pauses on `STOP` and resumes on `START` (see **Lifecycle** below)

This gives the developer a working starting point they can see running immediately.

## Lifecycle: pausing on `STOP`, resuming on `START`

`START` and `STOP` are not just notifications — they bracket the periods when the gadget's zone is
actually **visible**. A gadget that ignores them keeps animating and keeps burning through timers
while hidden, then jumps or cascades the moment it reappears. Any gadget with animation, rotation, or
timed content must handle them.

**`START` fires every time the zone is shown, not just once.** Treating it as "initialize" restarts
the gadget from its first item on every re-show. It should resume:

```ts
let started = false;

client.on(EventType.START, () => {
  if (started) { setPaused(false); return; }   // resume — don't restart
  started = true;
  void start();
});

client.on(EventType.STOP, () => setPaused(true));
```

**Author anything pausable as a keyframe animation, not a transition.** The obvious freeze —

```css
.is-paused * { animation-play-state: paused !important; }
```

— pauses `@keyframes` animations and **silently does nothing to CSS transitions**. A progress bar
built as a `width` transition keeps sliding while the zone is hidden. This is a design constraint to
decide up front; it's painful to retrofit.

**Pausing animation is only half of it — timers keep running.** `setTimeout`/`setInterval` keep
firing while hidden, so a rotating gadget burns through several items the instant it reappears.
Sleeps must *stop accruing time*, banking elapsed time across a pause rather than skipping a frame:

```ts
/** A sleep that only counts down while playing. */
async function sleep(ms: number): Promise<void> {
  let remaining = ms;
  for (;;) {
    await whenResumed();
    const startedAt = Date.now();
    if (await sleepUntilPaused(remaining) === 'done') return;
    remaining -= Date.now() - startedAt;   // bank elapsed, wait for resume
    if (remaining <= 0) return;
  }
}
```

With banking, an item 1s into a 12s rotation that is paused for 8s correctly takes ~11.6s after
resume to advance, with exactly one transition. Without it, it fires immediately on resume and
cascades through several items.

### Testing lifecycle handling without a player

`createPlayerClient()` installs `window.RevelDigital.Controller`, whose `onStart()` / `onStop()` /
`onCommand()` dispatch the same `RevelDigital.*` CustomEvents the real player fires. That means
lifecycle handling is verifiable in a plain browser console — this drives the genuine `STOP` path,
not a mock:

```js
window.RevelDigital.Controller.onStop();              // pause
window.RevelDigital.Controller.onStart();             // resume
window.RevelDigital.Controller.onCommand('foo','bar') // command
```

These hooks exist unless the client was created with `{ useLegacyEventHandling: false }`. They are
currently an implementation detail rather than a documented test API — tracked upstream as
RevelDigital/reveldigital-client-sdk#19.

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
// getPrefs() THROWS when no player is attached — always guard it, and always
// fall back to the gadget.yaml default so the gadget renders standalone in dev.
// Note: the .d.ts says `gadgets.Prefs | undefined`, but it never returns undefined —
// it throws instead, so an `if (prefs)` guard does NOT protect you.
function getPrefsSafe(): gadgets.Prefs | undefined {
  try { return client.getPrefs(); } catch { return undefined; }
}

const str  = (n: string, fallback: string)  => getPrefsSafe()?.getString(n) || fallback;
const bool = (n: string, fallback: boolean) => getPrefsSafe()?.getBool(n) ?? fallback;

str('myStringPref', 'test string');   // matches default_value in gadget.yaml
bool('myBoolPref', true);

// Other readers on the Prefs object: getFloat(), getInt(), getArray()

// Data tables (gadget-only — see references/datatable.md for the full API)
const dt = client.createDataTable('tbl_id');                  // throws if datatable feature not enabled
const rows = await dt.getRows({ sort: 'name', sortDir: 'asc' });   // rows[].data.<columnKey>
dt.on('rowUpdated', (change) => { /* real-time */ });
dt.startPolling(30000);
dt.dispose();
const cfg = client.createDataTableFromPref(prefs.getString('rdDataTable')); // from a `datatable` pref
```

> **Data tables are a gadget-only capability** — the player injects the data-table library into
> gadgets but not full-screen webapps. Requires the `datatable` feature in `gadget.yaml`
> `requirements`. See `references/datatable.md`.

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
- **`getPrefs()` throws — it does not reject.** It is synchronous (`return new window.gadgets.Prefs`), so it raises a `TypeError` the moment no player is attached. `.catch()` cannot help here; it must be wrapped in `try`/`catch`. This is the first thing most gadgets call, so an unguarded read kills the whole gadget at startup — blank screen in the dev server and CMS preview. Every pref read should fall back to its `gadget.yaml` default so the gadget still renders standalone. Applies to both `client-sdk` and Angular's `player-client` (identical implementation).
- **Async Client API methods do not throw outside the player.** `getWidth()`, `getDeviceTime()`, `getDevice()`, etc. all route through `getClient()`, which falls back to a mock (logging *"Client API not available, falling back to mock API"*) and resolves to `null` or a sensible default. Defensive `.catch()` on these is dead code — use it only to substitute a display value (e.g. `?? 'N/A'`), not for error handling.
- Always clean up event listeners on component unmount to avoid memory leaks.
- Use TypeScript types from the SDK: `import type { PlayerClient, EventType, IEventProperties, IOptions } from '@reveldigital/client-sdk';` (for Angular, import from `@reveldigital/player-client` instead)
