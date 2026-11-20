# Frame Home

Frame Home is a household launcher for Samsung The Frame, based on Nathan Clark's [CleanFrame](https://github.com/nthnclrk/CleanFrame). This foundation starts at CleanFrame 0.7.0, commit `845f332d083502fab39be49869052966103b564a`. The first change gives the browser document the Frame Home name. The TV widget name and identity, version, default settings, device adapter and remote behaviour retain their upstream values.

## Local development

Use Node.js 22.13+ on the 22 line or Node.js 24+, with Python 3.10+. The preview serves the actual app and uses the included public fonts.

```sh
npm ci
npm run dev:public
```

Open [http://127.0.0.1:4795/home/](http://127.0.0.1:4795/home/). Use the arrow keys, Enter for Select and Escape for Back. In List, move Up/Down through apps and Right to controls. In Dock, move Left/Right through apps and Up to controls. Preview preferences persist in the current browser origin. Selecting a TV-dependent action displays an error in a normal browser. The test suite separately injects synthetic device APIs.

```sh
npm run check
npm run check:tools
npm run check:public
npx playwright install chromium
CLEANFRAME_PUBLIC_FONTS=1 npm run check:browser
npm run export:public
```

The last command produces an unsigned, public-font widget offline. [Upstream development](development.md) describes signing with the official SDK and a locally selected certificate profile. [Validation](testing.md) describes the existing tests, public-source checks and visual review. The inherited GitHub Actions workflow uses an unprivileged read-only token and the public-font path. Review its result before merging a foundation PR. Keep `main` as the upstream starting point during review, and sync upstream changes through a reviewed topic branch once the fork has diverged.

## Architecture and capabilities

| Area | Implementation and evidence |
| --- | --- |
| Runtime | Static HTML/CSS, bundled assets and ES5 JavaScript. `app/index.html` loads the catalogues, `device.js` and `main.js`. Node and Playwright are development-only. No frontend framework, backend, account service or remote feed is needed. The 1920 × 1080 logical stage scales uniformly. |
| Remote and accessibility | The main controller owns directional focus, a panel stack, Select and Back, key-repeat handling and focus recovery. The stage has keyboard instructions, labelled controls and polite live announcements. Focus visibility and semantics can be checked in the browser. Sofa legibility, Samsung Voice Guide and the physical remote still need device testing. |
| Preferences | `cleanframe.home.v1` stores app membership and order, appearance, rotation and selection. The fallback migrates `cleanframe.shore.v1`. Validate updates before changing live state, and preserve saved app IDs and labels when discovery fails. The editor supports add, remove, move, undo and one to nine shortcuts. |
| Installed apps and launches | The adapter uses `tizen.application.getAppsInfo` and `launch`, with filtering for system entries, regional IDs, and fallback labels and icons. The default shortlist has fixed IDs. Verify installed IDs on the target, including any AU streaming services the household chooses. Maintainer physically confirmed Netflix and YouTube on the reference LS03D running an earlier CleanFrame build. |
| HDMI | 0.7.0 reads Tizen `VIDEOSOURCE`, checks active status where available, and calls `tvwindow.setSource` followed by Samsung's TV viewer. Adapter tests cover sleeping inputs, missing ports, denied calls and bounded cancellation. Test the actual displayed source and return path on this TV. A viewer-launch failure can follow a successful source selection. |
| Clock and background | The clock, flat and photo backgrounds, rotation and fonts render locally. Static, pre-rendered material images and cached geometry keep blur and pixel analysis out of the focus animation path. Review JPEG decode and memory use, focus latency and contrast on the actual Web engine. |
| Art Mode, Smart Hub and Settings | The Art Mode application-control contract and Samsung destination IDs are device-specific. A success callback does not establish the visible destination. Check current artwork, exit, Smart Hub escape and native Settings visually before promising these shortcuts. |
| Power and physical Home | The normal web app can be launched manually. Routing physical Home or cold boot is a separate system integration. Upstream recorded physical Home from Netflix after changing native Home/startup on its reference device. Normal power-on after streaming and Art Mode remains open even there. |

The main controller couples rendering, persistence and interaction. Retain its working seams for the next iteration. Move logic incrementally only where a concrete feature needs a clearer boundary. Device calls have bounded completion, explicit retry states and a separate adapter. Future changes should retain escaped rendering of discovered names and test missing, malformed and stale device results. The public-source scan catches known private paths and credential patterns. It is a bounded scan. Keep the [MIT licence and third-party terms](../THIRD_PARTY_NOTICES.md), including open font licences, trademark attribution and photo credits. Licensed private fonts and all signing and recovery materials stay outside Git.

[AeroTizen](https://github.com/helalaou/AeroTizen) at `59d00af` provides useful references for clearly labelled HDMI shortcuts and the placement of status controls. It has a Preact/TypeScript launcher and separate CLI and device workflows, including a watcher, native integration and debloat features. Its Continue Watching feed is outside the Frame Home brief. This branch takes no code or assets from AeroTizen.

## Target hardware and recovery

| | Household target | Upstream reference |
| --- | --- | --- |
| Model | QA75LS03DAWXXY, 75-inch Australian 2024 LS03D | QA55LS03DAWXXY, 55-inch Australian LS03D |
| Reported firmware | T-PTMDUABC-0090, 2130.0 | T-PTMDUABC-0090-2130.0 |
| OS and CPU | Awaiting verification | Tizen 9.0, 32-bit ARMv7 |
| Installation and visible behaviour | Untested | Earlier signed installation and limited observed flows in [TV status](tv-status.md) |

The match is a reason to test this family first. The manifest's Tizen 5.0 floor describes installation eligibility. Region, model, app versions and developer permissions can affect behaviour. The exact target Web engine, app IDs and Art/HDMI contracts are open.

This milestone is browser-only. For an explicitly authorised install phase, first agree on a maintenance window and a recovery checklist. Record stock Home, wake, Art Mode, picture settings, HDMI and streaming behaviour privately. Use the [official Samsung developer tools](https://developer.samsung.com/smarttv/develop/getting-started/using-sdk/tv-device.html), an SDK-generated independent app identity, an owner-controlled author certificate with a private backup, and a [Samsung distributor certificate](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/creating-certificates.html) for the target DUID. Verify the signed widget and its IDs before any upload. Install and open the app manually as a separate, observable step. Validate native destinations, wake, input switching and settings from the sofa. The rollback for an app-only installation is to return to Samsung Home, uninstall only the added widget, and verify the recorded stock behaviours. Keep private addresses, DUIDs, certificates and snapshots outside Git.

CleanFrame's reference installation scripts assume a particular SDK, shell transport and app identity. The native Home helper uses undocumented system interfaces and changes startup as well as Home. Even its query path stages files. Its restore path depends on the same developer transport and cannot be relied on if that route fails. A system-level change would need separate approval, exact original values and credible recovery evidence for this device; a factory reset cannot be assumed to recover it. AeroTizen's debloat and watcher paths were not run. Source rollback is independent of TV rollback. Use the branch's parent or the preserved `main` baseline for source recovery.

## First design direction

Use the Dock as a single, quiet horizontal row with four to six household-selected apps visible together. Start with a dark background and offer one restrained landscape as an option. Keep the clock small, give the focused app a strong shape change and a readable name, and place Inputs, Art Mode and Settings at a predictable secondary focus stop. Keep a discoverable route to Samsung Home. Use labels with icons where the meaning would otherwise be unclear. The browser concept uses existing Dock and appearance controls; it is a preview configuration.

| Priority | Implementation task | Acceptance |
| --- | --- | --- |
| P0 | Prototype the compact app row, optional clock and secondary controls using the existing focus controller. | One visible focus target, predictable four-direction movement, no launch on focus, one panel per Back, and readable app labels at 1080p and 720p. |
| P0 | Put discovered HDMI inputs one action away from Home and preserve the source adapter. | Clear active, connected, sleeping and unavailable states, with retry and focus restoration. |
| P0 | Retain the app editor and legacy preference migration while simplifying first-run defaults. | Existing saved apps and appearance survive. New users can choose their own shortcuts and reorder them entirely with the remote. |
| P1 | Move All apps into Settings and reduce appearance controls exposed by default. | Preserve discovery and editing. Remove extra first-screen actions, sample shortcuts and decorative controls once the simpler path is verified. |
| P1 | Verify the Art Mode shortcut and Smart Hub escape in the separately authorised app-only TV milestone. | Observe actual destinations, return behaviour and unchanged native picture, streaming, power and Art Mode settings. |
| P2 | Explore a curated family view and PIN-gated launcher settings. | Clearly explain its scope. Smart Hub, hardware shortcuts and other app entry points can bypass launcher restrictions. Configure real rating limits in each streaming service and use native device controls where available. |

Keep film and TV recommendations, announcements, ad placements, trailers and watch-history feeds out of Home. Preserve the adapter boundary, the app editor, explicit Select, Back and focus behaviour, preferences and offline rendering.

## Foundation verification

| Check | Result |
| --- | --- |
| Node 24.22.3, Python 3.12.14 and `npm ci` | Installed 79 locked packages. The local environment emits an npm proxy-option warning. |
| `npm run dev:public` | Served the actual app at the documented loopback URL. Verified HTTP 200 and the Frame Home browser title after the change. |
| `npm run check` | Passed ES5 lint, widget/resource validation and all 10 synthetic device-adapter tests. |
| `npm run check:tools` and `npm run check:public` | Passed 15 synthetic host-tool tests and the bounded public-source scan. |
| `CLEANFRAME_PUBLIC_FONTS=1 npm run check:browser` | 421 passed with zero failures before the app change and again afterwards. The environment blocked Playwright's pinned Chromium download with HTTP 403 and refused the stock browser's process socket. The unmodified suite ran with headless Chromium 153 from an isolated local cache. |
| Additional browser smoke check | Verified real arrow events, one focus, Enter to Settings, Escape with focus restoration, a saved Dock configuration after reload, and an honest TV-only error on Select. |
| `npm run export:public` | Produced and inspected the offline, unsigned archive. Its HTML includes the Frame Home title. |
