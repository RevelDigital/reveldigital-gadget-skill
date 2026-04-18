# Revel Digital Gadget Skill for Claude

A [Claude AI Skill](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills) that scaffolds complete [Revel Digital](https://www.reveldigital.com/) gadget projects — self-contained web applications that run inside the Revel Digital digital signage player.

Give Claude a gadget name, pick a framework, and get a fully buildable project with the SDK wired up, a `gadget.yaml` with sample preferences, and optional GitHub Pages deployment — ready to run.

For full API and platform documentation, see the [Revel Digital Developer Portal](https://developer.reveldigital.com).

## Supported Frameworks

| Framework | Bundler | SDK Library |
|-----------|---------|-------------|
| **React** | Vite + TypeScript | `@reveldigital/client-sdk` |
| **Vue 3** | Vite + JavaScript | `@reveldigital/client-sdk` |
| **Vanilla JS** | Parcel | `@reveldigital/client-sdk` |
| **Angular** | Angular CLI | `@reveldigital/player-client` |

## What Gets Scaffolded

- Full project with `package.json`, build tooling, and framework boilerplate
- `gadget.yaml` with sample preferences (string, bool, style, enum) including conditional `depends` visibility
- SDK integration with demo UI showing device info, preferences, and player actions
- `build:gadget` script chaining the framework build with the [Gadgetizer](https://www.npmjs.com/package/@reveldigital/gadgetizer) CLI
- GitHub Actions workflow for automated deployment to GitHub Pages (optional)

## Installation

### Claude.ai (Web / Desktop / Mobile)

1. Download the latest release ZIP from the [Releases](../../releases) page, or [download the skill directly](../../raw/main/revel-gadget-skill.zip)
2. In Claude, go to **Settings → Features → Skills**
3. Click **Add Custom Skill** and upload the ZIP
4. Toggle the skill **ON**

> Requires **Code Execution** to be enabled in Settings → Capabilities.

### Claude Code (CLI)

Copy the skill folder into your personal or project skills directory:

```bash
# Personal skills (available in all projects)
cp -r revel-gadget-skill ~/.claude/skills/revel-gadget

# Or project-level skills (available only in this repo)
cp -r revel-gadget-skill .claude/skills/revel-gadget
```

### Team / Enterprise Provisioning

Organization Owners can provision this skill for all users:

1. Go to **Organization Settings → Skills**
2. Upload the ZIP to make it available org-wide

## Usage

Once installed, just ask Claude to build a gadget:

> *"Create a React gadget called weather-dashboard for Revel Digital with GitHub Pages hosting"*

> *"Scaffold a vanilla JS Revel Digital gadget named lobby-ticker"*

> *"Build an Angular gadget for digital signage that shows a clock with configurable timezone"*

Claude will ask for any missing details (framework, name, hosting preference), then generate the full project.

## Skill Contents

```
revel-gadget-skill/
├── SKILL.md                    # Main skill instructions + SDK API reference
└── references/
    ├── react.md                # Vite + React + TypeScript scaffold
    ├── vue.md                  # Vite + Vue 3 + JavaScript scaffold
    ├── vanilla.md              # Parcel + plain JS scaffold
    └── angular.md              # Angular CLI + @reveldigital/player-client scaffold
```

## Related Resources

### Documentation

- [Revel Digital Developer Portal](https://developer.reveldigital.com) — Platform docs, APIs, and integration guides
- [Gadget Development Guide](https://developer.reveldigital.com/gadgets/) — End-to-end guide for building gadgets

### NPM Packages

- [@reveldigital/client-sdk](https://www.npmjs.com/package/@reveldigital/client-sdk) — Runtime SDK for React, Vue, and Vanilla JS
- [@reveldigital/player-client](https://www.npmjs.com/package/@reveldigital/player-client) — Angular-native library with DI & RxJS
- [@reveldigital/gadget-types](https://www.npmjs.com/package/@reveldigital/gadget-types) — TypeScript types for the OpenSocial `gadgets.Prefs` API
- [@reveldigital/gadgetizer](https://www.npmjs.com/package/@reveldigital/gadgetizer) — CLI that generates gadget XML from `gadget.yaml`

### Source & Demos

- [Revel Digital Client SDK (GitHub)](https://github.com/RevelDigital/reveldigital-client-sdk)
- [Demo: React Gadget](https://github.com/RevelDigital/gadget-demo-react)
- [Demo: Vue Gadget](https://github.com/RevelDigital/gadget-demo-vue)
- [Demo: Vanilla JS Gadget](https://github.com/RevelDigital/gadget-demo-vanilla-js)

## Building the .skill ZIP

To rebuild the distributable ZIP from source:

```bash
cd revel-gadget-skill
zip -r ../revel-gadget-skill.zip .
```

Or use the included build script:

```bash
./build.sh
```

## License

MIT — see [LICENSE](LICENSE) for details.
