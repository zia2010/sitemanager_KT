# Site Manager KT: Full Script (45 minutes)

One continuous script for the whole session, with the **action to perform on screen** while each part is being said. Written in first person for the KT giver.

- **Say:** what I speak. Read it naturally, do not recite it.
- **Do:** what I do on screen at that moment (open a file, run a command, point at a line, open DevTools).
- Items marked **(verify)** are things I could not confirm from the repo alone. Say "I believe" or "I could not confirm" for those.
- Detail for each block, with more code and Q&A backup, is in the per-block files in this folder (`01` to `10`). This script is the run of show.
- All paths are relative to `src/app` unless they start with `.github`, `.husky`, `deploy` or are `setup.cake`.

## Timeline

| Clock | Part | Minutes |
|---|---|---|
| 0:00 to 0:02 | Opening | 2 |
| 0:02 to 0:09 | 1. Foundation and Module Federation | 7 |
| 0:09 to 0:14 | 2. Dashboard and Widgets | 5 |
| 0:14 to 0:18 | 3. Alarms | 4 |
| 0:18 to 0:24 | 4. Equipment control: Thermostats, Lights, Schedule | 6 |
| 0:24 to 0:28 | 5. Configuration, Points, Reports | 4 |
| 0:28 to 0:32 | 6. State management and real-time data | 4 |
| 0:32 to 0:35 | 7. APIs and endpoints | 3 |
| 0:35 to 0:39 | 8. Dev workflow, testing, quality gates | 4 |
| 0:39 to 0:42 | 9. Build and deployment | 3 |
| 0:42 to 0:45 | Conclusion, handover and Q&A | 3 |

## Before the session (10 minutes of setup)

- [ ] From `src/app`: `yarn host` running (port 3002) and `yarn start` running (Storybook, port 9001).
- [ ] Browser: the app open, DevTools open on the **Network** tab. Keep a second tab on filter **WS** ready. Know how to switch the browser to a mobile width.
- [ ] VS Code tabs open in this order:
  1. `webpack.common.js`
  2. `src/components/Provider/SiteManagerProvider/SiteManagerProvider.tsx`
  3. `src/components/Permissions/PermissionsWrapper.tsx`
  4. `src/hooks/useSignalRSubscription.ts`
  5. `src/components/Dashboard/DashboardApp.tsx`
  6. `src/components/Alarms/AlarmsPageContent.tsx`
  7. `src/contexts/AlarmContext.tsx`
  8. `src/hooks/useSetPointValues.ts`
  9. `src/components/Dashboard/Thermostats/Thermostats.tsx`
  10. `src/utils/scheduleConverter.ts`
  11. `src/components/Configuration/Configuration.tsx`
  12. `src/hooks/useAlarms.ts`
  13. `jest.config.ts`, `jest.setup.ts`
  14. `.husky/pre-commit`, `setup.cake`, `.github/workflows/automate-build.yml`, `Dockerfile`, `nginx/init-scripts/env-variables.sh`, `nginx/nginx.conf`, `deploy/application/content/aks-deployment.yml`
- [ ] Docs open in a second window: `KT.md` (tech debt register, section 6) and `KT/10-Module-Federation-Explained.md`.
- [ ] A terminal in `src/app` with `yarn test:fast LDFlagUtils` typed but not run.
- [ ] Git is clean, so any live edit can be reverted with `git restore`.

## Timing discipline

| Checkpoint | If I am behind | Cut |
|---|---|---|
| 0:14 | More than 1 min over | Skip the energy widget |
| 0:24 | More than 1 min over | Skip schedule precedence, only mention it |
| 0:28 | More than 1 min over | Skip Reports, name it only |
| 0:35 | More than 1 min over | Keep the APIs part to the one diagram and the three base URLs |
| 0:42 | Hard stop for content | Protect the conclusion. Deployment shrinks before the conclusion does. |

---

# Opening (0:00 to 0:02)

**Say:**
"Hello everyone. For the next 45 minutes I am handing over the **Site Manager application**: what it is, how it is built, how it behaves at runtime, and how it gets to production. I will show real code for each point. By the end I want you to be able to run it, find any feature quickly, and know which known issues to look at first."

**Do:** Show the `KT` folder in the VS Code explorer, then the timeline table above (or a slide with the same content).

**Say:**
"Structure: foundation and architecture first, because everything depends on it. Then the pages one by one: Dashboard, Alarms, equipment control, and the admin pages. Then three cross-cutting parts: state and real-time data, the APIs, and the developer workflow. Last, build and deployment. I will keep questions for the end, and I will write down anything long in a parking lot."

**Do:** Open the app in the browser on the dashboard so the audience sees the product while I talk.

**Say:**
"One sentence to anchor everything: **Site Manager is a building-management UI, delivered as a Module Federation remote, where REST gives the state and SignalR gives the live updates.**"

---

# Part 1: Foundation and Module Federation (0:02 to 0:09)

Source: `KT/01-Foundation-and-Architecture.md`, `KT/10-Module-Federation-Explained.md`.

## 0:02 to 0:03: What the app is

**Say:**
"A facility manager opens a site and sees alarms, thermostats, lights and energy use, and can control equipment, all in real time. The data model is **Organization, then Site, then Asset, then Point**. An asset is a piece of equipment, like a thermostat or a light. A point is one value on that equipment: a temperature, a setpoint, an on/off status, a mode. In code, `orgId` is `sites[0].customerId` and `siteId` is `sites[0].id`. We only ever use the first site in the array."

**Do:** Point at a thermostat card on the dashboard. Name the asset (the card), then a point (the temperature value on it).

## 0:03 to 0:05: Module Federation

**Say:**
"The most important fact about this repository: it is **not a standalone app**. It is a **Module Federation remote** called `siteManagerApp`. Another application, the **unified-shell**, loads our pages in the browser at runtime. Think of a shopping mall: the mall is the unified-shell, we are a shop. We can restock without the mall being rebuilt."

**Do:** Open `webpack.common.js`, scroll to the `ModuleFederationPlugin` block (around lines 99 to 120).

**Say:**
"Here is the contract. `name` is `siteManagerApp`. `filename` is `remoteEntry.[contenthash].js`, the front door the shell downloads first. Then **eight exposes**: Dashboard, Alarms, Thermostats, Lights, Reports, Configuration, Points, Others. The shell chooses which one to render. **There is no router in this repo.**"

**Do:** Highlight the `exposes` block, then the `shared` block.

**Say:**
"Then `shared`. React, React DOM, React Query and `@hcecbp/provider` are **singletons**, so the page loads only one copy of each. This matters. Two copies of React break hooks. And our auth token comes from `useClient`, a context the **shell** creates, so we must use the shell's copy of that library. I treat these four as a contract with the shell team. `@forge/common`, our design system, is **not shared**. It is bundled into our files."

**Do:** In the browser, open DevTools, Network, filter `remoteEntry`, and show the hashed file. Open it for two seconds to show the list of exposed modules.

**Say:**
"The shell only needs three things from us: the remote name, the exposed names, and the props. The props are `sites`, `userDetails` and an `onTriggerActions` callback. Everything else, providers, data fetching, live updates, we create inside each page."

**Do:** Open `src/index.ts` (empty) and `src/App.tsx` (placeholder).

**Say:**
"One consequence: `src/index.ts` is empty and `App.tsx` is a placeholder. The webpack entry is a stub because a remote never boots itself. The shell is the thing that starts React. So `yarn host` serves our files but you will not see a normal page. For UI work without a shell we use Storybook and tests which is not available currently."

## 0:05 to 0:06: Provider and permissions

**Say:**
"Every page wraps itself in the same component, `SiteManagerProvider`. It gives you four things in one tree: a toast provider, **one React Query client**, **one SignalR connection** and the **permissions gate**."

**Do:** Open `SiteManagerProvider.tsx`. Show the `QueryClient` defaults near line 83, then the `return` at the bottom (around lines 459 to 465): Toast, Query, Permissions, Context.

**Say:**
"Query defaults are 5 minutes stale time, 10 minutes cache, 2 retries, no refetch on window focus. Notice the nesting: the page renders only if the permissions gate passes."

**Do:** Open `PermissionsWrapper.tsx`. Scroll through the early returns that render `<AccessDenied />` (around lines 88 to 135).

**Say:**
"Before any page renders we call `/status` on the Site Manager API and the user privileges on the Buildings Manager API. The user sees an AccessDenied screen for: portfolio view, site not registered or still syncing, no subscription, or no read permission. **Three permission strings matter: `SiteManager_Point_Read`, `SiteManager_Point_Write` and `SiteManager_ConfigureWidget`. Read is required to see anything. Write is enforced **per widget** with a read-only flag, not at this gate.**"

## 0:06 to 0:08: SignalR and React Query

**Say:**
"Data arrives in two ways. **REST through React Query** for lists and initial state. **SignalR** for live updates."

**Do:** Open `useSignalRSubscription.ts`. Show lines 128 to 185 (the `invoke` calls).

**Say:**
"The hook takes a group and either asset ids, point ids or gateway mappings. It tells the server what is visible with `visible-points`, `visible-assets` or `gateway-status`, then always calls `subscribe`. **If you pass point ids, asset ids are ignored.** On cleanup it unsubscribes. So the backend only sends what is on screen."

**Do:** Switch to the browser. Network tab, filter WS, click a frame.

**Say:**
"Each of these frames is a live update. When a thermostat changes in the field, you see it here, the page merges it into local state, and the card updates without a refetch. Reconnect uses automatic retry with delays of 1, 2, 5, 10 and 30 seconds, up to 10 attempts."

## 0:08 to 0:09: Feature flags and environment

**Do:** Open `src/utils/LDFlagUtils.ts` around line 116.

**Say:**
"Feature flags use LaunchDarkly. A flag is an **array of strings**. It is true if the array contains `all`, or contains **both** the org id and the site id, or both labels. Only the org, or only the site, is false. Flags you will meet: `configureWidget`, `enableSchedules`, `enableTc500`, `tc500ModeCustomizations`, `smNaDefaultValues`, `smEnableLogs`."

**Do:** Run `yarn test:fast LDFlagUtils` in the terminal and show the tests passing.

**Say:**
"The test names read as the specification. Now environment: `utils/environment.ts` holds literal placeholders like `$SiteManagerApiBaseUrl`. In dev and test you get a stub with real QA URLs. In production a container script replaces the placeholders. I will show that in the deployment part."

**Do:** Open `src/utils/environment.ts` and show the placeholders and the last line choosing the stub in development and test.

**Say:**
"Pattern for every feature from now on: **page component, then hooks, then API or SignalR, then models**. Next, the Dashboard, where all of this comes together."

---

# Part 2: Dashboard and Widgets (0:09 to 0:14)

Source: `KT/02-Dashboard-and-Widgets.md`.

## 0:09 to 0:10: The page at a glance

**Do:** Switch to the running app on the dashboard. Point left, then right.

**Say:**
"This is `DashboardApp`. On the left, the **Summary** column: site logo, alarm counts by severity, manual overrides and energy. On the right, the **HVAC** grid and the **Lights** grid. On mobile it shows one section at a time with a segmented control."

**Do:** Switch the browser to a mobile width for 5 seconds, then back to desktop.

**Do:** Open `DashboardApp.tsx` and go to the bottom (around lines 935 to 945), the provider stack.

**Say:**
"`DashboardApp` itself is only a provider stack: LaunchDarkly, then `SiteManagerProvider`, then Alarm, ManualOverrides and Schedule providers. The real logic is in `DashboardContent` in the same file."

## 0:10 to 0:11: One subscription drives the page

**Do:** Scroll to `handleSignalRUpdates` (around lines 200 to 290) and the `useSignalRSubscription` call below it.

**Say:**
"The Dashboard makes **one** SignalR subscription, in the group `dashboard`. It subscribes to the union of the visible thermostats, lights and manual-override assets. `handleSignalRUpdates` is the message router. A point update goes to the matching thermostat or light callback. A schedule update goes the same way. An alarm summary goes into `AlarmContext`. Manual overrides go to state. Children register callbacks, so updates are pushed straight to the widget that owns the asset, not through React Query."

**Do:** Switch to the WS tab, click one frame, and match its shape to one of the type guards in `src/utils/signalRUtils.ts`.

## 0:11 to 0:12: Summary column and the deep link

**Do:** Open `src/components/Dashboard/Summary/SystemAlerts/SystemAlerts.tsx` (around lines 72 to 88).

**Say:**
"The alert badges show High, Medium and Low counts. When you click one, the Dashboard does **not** navigate itself. It calls `onTriggerActions` with the action named in the `AlarmsDashboardNavigation` variable and the severity. The shell does the navigation. That is the only coupling between Dashboard and Alarms. Manual overrides only appear if the site has a Niagara gateway. Clearing is disabled for read-only users."

**Do:** Click a badge in the running app. If the dev harness does not navigate, say so. **(verify the harness behaviour before the session.)**

## 0:12 to 0:13: Grids, reorder, banners

**Do:** Open `src/components/Dashboard/shared/SortableGrid/SortableGrid.tsx` (around lines 92 to 125). Then drag a card in the browser, and try dragging a slider.

**Say:**
"Both grids use `@dnd-kit` drag and drop. Pointer drag starts after 8 pixels, touch after a 250 millisecond hold. Anything marked `data-prevent-dnd`, like a slider, is exempt, so dragging a slider does not move the card. On drop we update the order **optimistically**, call the reorder endpoint, and revert if it fails. If everything in a widget is hidden in Configuration, the grid changes from three columns to `1fr 4fr` and a banner explains why."

## 0:13 to 0:14: Energy widget

**Do:** Open `src/components/Widgets/EnergyConsumptionWidget.tsx` (around lines 159 to 198).

**Say:**
"The energy widget shows electricity, gas and water for the current month. It **only renders if the site has a CEM bundle**: we look for a bundle name containing `cem`. It polls on an interval, 10 minutes by default, and pauses when the tab is hidden. This is the one place that does not use React Query."
---

# Part 3: Alarms (0:14 to 0:18)

Source: `KT/03-Alarms.md`.

## 0:14 to 0:15: Two jobs

**Do:** Open the Alarms page in the app (desktop).

**Say:**
"The Alarms module does **two jobs**. One: the Alarms **page**, a searchable, filterable, sortable list. Two: `AlarmContext`, the **site-wide alarm summary** that colours every equipment card. Severity is High, Medium, Low: red, yellow, blue borders. The highest severity on an asset wins. Alarms are **read-only here**. There is no acknowledge action in this module."

## 0:15 to 0:16: The page and its one request

**Do:** Open `AlarmsPageContent.tsx` and show `getAlarmFilterCriterias` (around lines 68 to 104). Then open `useAlarms.ts` (around lines 26 to 104).

**Say:**
"The list uses one endpoint: **POST** `/metrics/alarmdetails`. Paging, dates, search and sort go on the query string. The filters go in the body as a list of `fieldName` and `fieldValue`. Group Status becomes `alarmStatus`, group Equipment becomes `assetId`, with the name translated to an id. Every filter, page and sort is in the **query key**, so any change makes a new cache entry and a new request. Any filter change resets the page to 1. Search is debounced 500 milliseconds."

**Do:** In the browser, Network tab, filter `alarmdetails`. Change the status filter and show the request body change.

## 0:16 to 0:17: Live summary through AlarmContext

**Do:** Open `src/contexts/AlarmContext.tsx` (around lines 36 to 72).

**Say:**
"The summary has two sources. REST through `useSystemAlerts` for the initial load, and SignalR for pushes. The rule is one `if`: if the SignalR list is not empty, use it, otherwise use REST. The Alarms **list** itself does not use SignalR. Only the summary is live."

**Do:** Switch to the Dashboard, point at a card with a coloured border, then come back to the line that produced that class.


---

# Part 4: Equipment control (0:18 to 0:24)

Source: `KT/04-Thermostats-Lights-Schedule.md`.

## 0:18 to 0:19: The pattern

**Do:** Open the Thermostats page in the app.

**Say:**
"Thermostats and Lights follow the same pattern. A page lists assets from `/layouts/assets/HVAC` or `/layouts/assets/Light`, 10 per page with search. A widget per asset shows the state. A click sends a command, the UI updates optimistically, and SignalR confirms. The Dashboard has its own versions of these widgets, with drag and drop and the TC500."

## 0:19 to 0:21: A command from click to gateway

**Do:** Open `src/hooks/useSetPointValues.ts` (around lines 38 to 82).

**Say:**
"Every write goes through one hook: **POST** `/gateways/{gatewayId}/SetPointValues`. The body has the gateway type, a system guid, a fresh `commandId` UUID per call, and a list of point id and value pairs."

**Do:** Open `Thermostats.tsx` in the Dashboard folder, `toggleWidget` (around lines 856 to 935).

**Say:**
"Power toggle is optimistic. We flip the local state first, send the command, and on failure we revert and show a toast. The value is `'true'` or `'false'` for **Niagara** gateways and `'1'` or `'0'` for the others."

**Do:** Scroll to the debounce (around line 1529 and 1760 to 1800).

**Say:**
**"The plus and minus buttons move the setpoint in 0.5 steps, and the write is **debounced 800 milliseconds per thermostat**."**

**Do:** In the app, click the plus button on a thermostat five times quickly. In the Network tab, show that only **one** `SetPointValues` call goes out after about 800 ms. Mention this changes real equipment, so do it only on a test site.

**Say:**
"There is no unit conversion anywhere. Units are display only."

## 0:21 to 0:22: TC500

**Do:** In `Thermostats.tsx`, go to `getSliderConfigForMode` (around lines 1053 to 1250).

**Say:**
"TC500 is a thermostat model, detected when the model contains `tc500`. It gets a **slider widget**, only on the Dashboard, behind the `enableTc500` flag. The key idea: the point type is **built from the current schedule**, for example `set-occupied-cooling` or `set-standby-heating`. Limits come from config points, with fallback to the `smNaDefaultValues` flag and then to 40, 120, 2 and 1. The rule is: **UnoccHeat ≤ StbyHeat ≤ OccHeat < OccCool ≤ StbyCool ≤ UnoccCool**, and cool minus heat must be at least the deadband."

**Say (warning):**
"`getSliderConfigForMode` can trigger writes during render. That is fragile. Handle it with care."

---

# Part 5: Configuration, Points, Reports (0:24 to 0:28)

Source: `KT/05-Configuration-Points-Reports.md`.

## 0:24 to 0:25: Configuration

**Do:** Open the Configuration page in the app. Toggle one asset (on a test site), then toggle it back.

**Say:**
"Configuration is the **admin screen**. It decides which HVAC and Light assets appear on the Dashboard. Each row has a show/hide toggle: **POST** `/configurations` to show, **PUT** `/configurations/remove` to hide. Optimistic, with revert on error. You need write access to toggle."

**Do:** Open `Configuration.tsx` (around lines 47 to 54).

## 0:25 to 0:26: Configure widget (point roles)

**Do:** Open the Configure modal on a thermostat (only visible with the permission and the flag). If it is not visible on the demo site, open `ConfigurationWidget.tsx` `handleApply` (around lines 270 to 330) instead.

**Say:**
"The Configure modal is the most intricate form in the app. It maps each widget control, like 'Mode - Control', to a **point role** on the site, and can apply the same mapping to other equipment. It is available only with `SiteManager_ConfigureWidget` **and** the `configureWidget` flag. On Apply we send the current asset plus the selected equipment, success is `response.code === 200`, and then `onResubscribe` re-subscribes SignalR so the new points start receiving live updates. That is why widgets start updating after you configure them."



## 0:27 to 0:28: Reports and the common shape

**Do:** Open `src/hooks/reportDownload.ts` (around lines 25 to 44).

**Say:**
"Reports is the simplest module: list, search, sort, **download PDF**. Download is a plain async function that fetches a blob, creates an object URL and clicks a temporary link. Note that it uses `encodeURIComponent` on every path part. That is the pattern to copy. Reports has no permission check, unlike Configuration and Points."

**Say:**
"The shape repeats across modules: a desktop table, a mobile card list, React Query hooks, debounced search, optimistic mutations, and the permission check. If you understand Configuration, you understand most of the others."

---

# Part 6: State management and real-time data (0:28 to 0:32)

Source: `KT/07-State-Management-and-Real-Time-Data.md`.

## 0:28 to 0:29: The four homes for state

**Do:** Use `Ctrl+Shift+F` to search `redux|zustand|recoil|jotai` in `src`. Show no results.

**Say:**
"There is **no Redux or other global store**. Every value on screen lives in one of four places. **One:** server state in React Query, for anything fetched over REST. **Two:** live state from SignalR, in component state or small contexts. **Three:** shared UI state in React contexts: alarms, overrides, schedule view. **Four:** local component state. When you add a feature, ask which of the four it is."

**Do:** Open four files side by side: `hooks/useAlarms.ts`, `DashboardApp.tsx`, `Schedule/ScheduleProvider.tsx` and `Dashboard/Thermostats/Thermostats.tsx`. Name the home of each.

## 0:29 to 0:30: React Query conventions

**Do:** Search `queryKey: \[` in `src/hooks` and read five keys aloud.

**Say:**
"One query client, created in `SiteManagerProvider`. The key convention is **feature, then org, then site, then the specifics**, so a site switch gives a new cache entry. **Hooks own the API calls.** There is no shared API layer. Components never call `fetch` directly. Invalidation is rare: I found one real `invalidateQueries`, in `useCalendarEvents` after a schedule save. Most writes are optimistic and confirmed by SignalR. One caution: `useAlarms` and `useActiveAlarmsAssets` share the key prefix `alarmsData`, so invalidating that prefix hits both."

## 0:30 to 0:31: SignalR lifecycle

**Do:** Open `useSignalRSubscription.ts` (around lines 77 to 125).

**Say:**
"everytime the page reloads the subscriptions are checked"
"Subscriptions follow **visibility**. When the user scrolls or changes page, the visible ids change and we re-subscribe. To avoid re-subscribing too often, the hook uses three defences: a ref for the latest callback, sorted copies of the id arrays, and a serialised subscription key. The weak point is that the provider's context value is rebuilt on every render, and the hook depends on it. So it may re-subscribe more than needed. If you work here, memoise it."



---

# Part 7: APIs and endpoints (0:32 to 0:35)

Source: `KT/09-APIs-and-Endpoints.md`.

## 0:32 to 0:33: Where calls go

**Do:** Open `KT/09-APIs-and-Endpoints.md` and show the diagram in section 1.

**Say:**
"The app has no server of its own. Every call goes from the browser to one of **three places**: the **Site Manager API** for assets, points, alarms, overrides, configuration, reports, utility data and setpoint writes. The **Buildings Manager API** for permissions, subscriptions, the site logo, and the calendar. And the **SignalR hub** for live updates. The base URLs come from the environment placeholders I showed earlier."

## 0:33 to 0:34: Anatomy of one call

**Do:** Open `src/hooks/useAlarms.ts` again, or section 2 of the API doc.

**Say:**
"There is no shared client. Each hook does the same seven things: get the token provider from the shell, define a cache key, get a fresh token, build the URL with `orgId` and `siteId`, call `fetch` with the `Authorization` header, throw on a non-2xx, and return the JSON. Note the token already contains `Bearer`, so we send it as is. SignalR does the opposite and strips it. Most hooks read the base URL once at module load, so in tests you must mock the environment **before** importing the hook."

## 0:34 to 0:35: Endpoint tour

**Do:** Scroll the endpoint tables in section 3 of the API doc.

**Say:**
"Four endpoints cover most of what you will touch. **Alarm list:** POST `/metrics/alarmdetails`. **The one write:** POST `/gateways/{id}/SetPointValues`, note the capital S and P. **Show or hide:** POST `/configurations` and PUT `/configurations/remove`. **Permissions:** GET `/users/orgs/{o}/nodes/{s}/privileges?scopes=SiteManager` on the Buildings Manager API. Small oddities to remember: `layout` singular for one asset, `organisations` with British spelling for the logo, and PascalCase in the subscriptions path. Purposes marked inferred in the doc should be confirmed against the backend Swagger."

---

# Part 8: Dev workflow, testing, quality gates (0:35 to 0:39)

Source: `KT/08-Dev-Workflow-Testing-and-Quality-Gates.md`.

## 0:35 to 0:36: Daily commands

**Do:** In the terminal in `src/app`, list the commands as I say them. Do not run the slow ones.

**Say:**
"We use **Yarn**. The app is in `src/app`. Daily commands: `yarn host` on port 3002, `yarn start` for Storybook on 9001, `yarn test:fast` for a quick run, `yarn test` for the full run with coverage, `yarn lint:all`, `yarn format:all` and `yarn build`. Set up with `npm install` at the root, which installs the git hooks, then `yarn install` in `src/app`."

## 0:36 to 0:37: Conventions enforced by lint

**Do:** Open `.eslintrc.js`. Show `no-console`, the header rule and the React rules.

**Say:**
"These are **errors**, not suggestions, so the pre-commit will stop you. A copyright header on every file. No `console` calls. Arrow-function components, handlers named `handleX`, props named `onX`, no array index as key, and ternaries instead of `&&` in JSX. Prettier sorts imports and Tailwind classes. Each component has `Name.tsx`, a test, and a story, though stories exist for only a few modules today."

## 0:37 to 0:38: Testing

**Do:** Open `jest.setup.ts`, then `src/hooks/useSetPointValues.test.ts`.

**Say:**
"Jest 29 with React Testing Library, about 160 test files. Global mocks are already in `jest.setup.ts`: i18n resolves real English text, and **SignalR is mocked globally**, so no real connections in tests. The recipe for a hook test: mock `useClient`, mock `fetch`, wrap in a `QueryClient` with **retry off**, `renderHook`, `waitFor`. That works for almost every hook. Fixtures live in `__mocks__`. Reuse them."

**Do (optional, 30 seconds):** Break a rule on purpose in `LDFlagUtils.ts` (change `hasOrg && hasSite` to `hasOrg || hasSite`), run `yarn test:fast LDFlagUtils`, show the failing test, then `git restore` the file.

## 0:38 to 0:39: Quality gates

**Do:** Open `.husky/pre-commit` and `jest.config.ts`.

**Say:**
"The gates in the order you meet them. **Pre-commit:** `lint-staged`, then `yarn build`, then `yarn test:fast`. Coverage thresholds are only enforced in full runs and in CI, because fast mode turns coverage off. **CI:** lint, tests with coverage, SonarQube, then Coverity and BlackDuck on master, main and hotfix. **Pull request:** CODEOWNERS assigns reviewers. If your pre-commit build fails, it is usually a TypeScript error, because Jest does not type-check in this setup. Never use `--no-verify`. Note `HUSKY_SETUP.md` is out of date. Trust `.husky/pre-commit`."

---

# Part 9: Build and deployment (0:39 to 0:42)

Source: `KT/06-Build-and-Deployment.md`.

## 0:39 to 0:40: Build and CI

**Do:** Open `setup.cake`, then `.github/workflows/automate-build.yml`.

**Say:**
"We build with **Cake**. `setup.cake` sets the source directory `./src/app`, builds the React app into `dist`, then builds a Docker image from `dist` and the nginx folder. It also configures SonarQube, BlackDuck, which breaks the build on **high** vulnerabilities, and Coverity. Versioning is GitVersion in mainline mode, which is why the checkout uses full history. CI runs on self-hosted Linux runners on every push: build, quality, then security and open-source scans on master, main and hotfix, then publish. **Every branch publishes an image**, so you can test a feature branch build."

## 0:40 to 0:41: One image for all environments

**Do:** Open `src/app/nginx/init-scripts/env-variables.sh` (the whole file). Then the `Dockerfile` copy lines and `src/utils/environment.ts`.

**Say:**
"This is the key trick. The build outputs `env.[hash].js` as its own file with `$Token` placeholders. This script runs when the container starts, finds the newest `env.*.js`, and runs `envsubst` on it. So **one image runs in every environment**. Only the container environment variables change: the two API base URLs, the SignalR URL, LaunchDarkly keys, polling intervals, `ClearOverrideWhitelist`, `CEMBundleIdentifier`, `AlarmsDashboardNavigation`. Troubleshooting tip: if the app loads but calls a URL starting with `$`, open the served `env.*.js`. A `$Something` left in it means that variable was not set."

## 0:41 to 0:42: nginx, manifests, release

**Do:** Open `nginx/nginx.conf`, then `deploy/application/content/aks-deployment.yml` (env and probes around lines 96 to 172).

**Say:**
"nginx serves `dist`, sets security headers, a CORS allow-list, and cache rules. The container runs as non-root on **port 3000**. The Kubernetes manifest has `#{token}` placeholders replaced by the deployment tool, and probes on `/health/startup`, `/liveness` and `/readiness`. There is an OpenShift equivalent. The JSON files in `deploy/` are marker files for the AutoMate deployment platform, so do not delete them. **Feature flags are the kill switch**: turn off `enableTc500` or `configureWidget` in LaunchDarkly with no deploy."

**Say (verify list, quickly):**
"Four things I would check first in a new environment. `nginx.conf` has no `/health/*` location, so the probes probably pass only through the SPA fallback. The no-cache rule targets `manager-bundle.js`, not `remoteEntry.[hash].js`, and I do not know how the shell resolves the hashed file name. Security headers may be missing on static asset locations, because `add_header` in a nested location replaces the parent headers. And the release and rollback runbook in `KT.md` section 5.8 still needs to be filled in."

---

# Conclusion, handover and Q&A (0:42 to 0:45)

## 0:42 to 0:43: Summary

**Do:** Show a closing slide, or keep `KT/00-KT-Agenda-and-Run-of-Show.md` open. Stop sharing code.

**Say:**
"To summarise in five lines.
**One:** this is a federated React remote, not a standalone app. The unified-shell loads our eight pages and passes `sites`, `userDetails` and `onTriggerActions`.
**Two:** data comes from REST through React Query and live updates through SignalR, always keyed by organisation and site.
**Three:** permissions and feature flags decide what a user sees. Read is required at the gate, write is enforced per widget.
**Four:** writes are optimistic with revert, and the one write endpoint is `SetPointValues`.
**Five:** the same build runs in every environment, because configuration is injected into `env.*.js` when the container starts."

## 0:43 to 0:44: What I would do first, and what I would fix first

**Say:**
"If you are the new owner, in your first week: run `yarn host` and `yarn test:fast`. Read `KT.md`. Then trace one flow end to end. My suggestion is the thermostat power toggle, from the click in `Thermostats.tsx`, through `useSetPointValues`, to the SignalR frame that confirms it."

**Say:**
"The issues I would look at first:
1. `Configuration.tsx` spreads `{...sites}` into the page.
2. `useUpdateSchedule` hardcodes `°C` and blanks the end value.
3. The schedule edit gate can leave editing enabled for a user with no permissions.
4. The Dashboard reads `sites[0].siteId` while the others read `sites[0].id`.
5. The provider's context value is rebuilt every render and causes extra SignalR re-subscriptions.
6. The nginx health endpoints and the `remoteEntry` caching rule need confirming.
The full register is in section 6 of `KT.md`."

## 0:44 to 0:45: Open questions, handover and Q&A

**Say:**
"Open items I need to hand to you or to other teams: confirm with the shell team how the hashed `remoteEntry` is found after a deploy, and whether our React Query and `@hcecbp/provider` versions match theirs; fill in the release, rollback, monitoring and contact details in `KT.md` 5.8; and check the backend Swagger for the endpoints marked inferred. I will share all of these documents in the `KT` folder. Now questions. If an answer is long, I will put it in the parking lot and follow up in writing."

**Do:** Open the parking lot table (below) and capture each question live.

---

# Reference for Q&A (do not read aloud)

## Likely questions with short answers

| Question | Answer |
|---|---|
| Why no router? | The shell owns routing. Each exposed component is one page. |
| Can I use more than one site? | No. Everything reads `sites[0]`. |
| Can users acknowledge alarms? | Not in this module. It lists and shows status. |
| Why not put live values in React Query? | Updates are pushed per point and merged into widget state. The Dashboard routes them through callbacks. It avoids a cache write for every frame. |
| Why is `@forge/common` bundled? | It is not declared as shared. A trade-off for version independence. **(verify with the shell team)** |
| Do I need the backend to run locally? | No for layout work: the stub environment points at QA URLs. For real data you need a valid token from the shell. **(verify the current stub values.)** |
| What coverage does a new file need? | Thresholds are global (branches 65, functions 60, lines 89, statements 89). Team convention is at least 80% for new code. |
| Do I rebuild for each environment? | No. Same image, different environment variables. |
| How do I add a runtime setting? | Add the placeholder and the stub value in `utils/environment.ts`, add it to the AKS and OpenShift `env` lists, and set it for each environment. |
| How do I roll back? | Redeploy the previous image tag. **(confirm the process)** |
| How do I turn off a risky feature? | Remove the site from the LaunchDarkly flag. No deploy needed. |
| What if SignalR is down? | Widgets show a skeleton until the first update or 5 seconds. Reconnect retries up to 10 times. |
| What happens if the shell runs a different React version? | The singleton rule picks one version. If they are incompatible, expect console warnings and possible runtime errors. |

## Gotcha cheat sheet

| Area | Gotcha |
|---|---|
| Foundation | Provider context value rebuilt each render. Chatty `console.log` in provider and subscription hook. About 17 hooks read `BASE_URL` at module load. Two `usePermissions` hooks exist. |
| Dashboard | `siteId` versus `id`. `useSessionBanner` computes error-banner visibility from the wrong state. `Thermostats.tsx` is about 2,700 lines. |
| Alarms | Severity sort key. Desktop and mobile duplication. Class names do not match severities. No REST fallback per asset when a SignalR list exists but lacks the asset. |
| Equipment | `getSliderConfigForMode` writes during render. `°C` hardcoded. Edit gate. `toISOString().slice(0,10)`. Widgets defined inside render functions remount each render. |
| Config and Points | `{...sites}` spread. `removeConfiguration` ignores `type`. Existing mappings matched by label. Status filter ORs across groups. `sites?.[0].customerId` throws if empty. |
| Deployment | No `/health/*` in nginx. Prometheus port 7000 declared but nginx listens on 3000. `remoteEntry` caching rule. Missing env variable becomes an empty string. |
| Tooling | `HUSKY_SETUP.md` is stale. Mixed `.test.tsx` and `.spec.tsx`. `@store/*` alias does not exist. `@tanstack/react-query-devtools` ^5.81 versus react-query ^4.29. |

## Parking lot

| Question | Owner | Follow-up |
|---|---|---|
| | | |
