# SITE MANAGER — 45-MINUTE KT SPEAKER REFERENCE

Keywords and flow:

**WHAT → ARCHITECTURE → FEATURES → DATA → APIs → DEV → DEPLOYMENT → HANDOVER**

Main sentence:

> **Site Manager is a building-management UI delivered as a Module Federation remote, where REST gives us the state and SignalR gives us live updates.**

---

# TIMELINE AT A GLANCE

| Time | Block | Memory phrase |
|---|---|---|
| 0:00–0:02 | Opening | What → How → Runtime → Production |
| 0:02–0:09 | Foundation + Module Federation | Remote → Shell, REST = initial, SignalR = live |
| 0:09–0:14 | Dashboard | One subscription → many widgets |
| 0:14–0:18 | Alarms | Page + Summary |
| 0:18–0:24 | Equipment (Thermostats, Lights, TC500) | List → Click → Optimistic → API → SignalR |
| 0:24–0:28 | Configuration, Points, Reports | POST = show, PUT = hide |
| 0:28–0:32 | State management | Four homes |
| 0:32–0:35 | APIs | Data / Platform / Live |
| 0:35–0:39 | Dev workflow | Run → Test → Lint → Build → CI |
| 0:39–0:42 | Build and deploy | Build once, configure at runtime |
| 0:42–0:45 | Known issues, handover, conclusion | Run → Read → Trace → Verify |

---

# PRE-FLIGHT CHECKLIST (5 MINUTES BEFORE)

* [ ] Unified shell running and logged in with a test user.
* [ ] A **test site** selected (never demo writes on a real site).
* [ ] Site has: at least one thermostat, one light, one alarm, one report.
* [ ] VS Code open with these tabs pinned: `webpack.common.js`, `SiteManagerProvider.tsx`, `DashboardApp.tsx`, `Thermostats.tsx`, `useAlarms.ts`, `useSetPointValues.ts`, `reportDownload.ts`, `Dockerfile`.
* [ ] Browser DevTools open on Network, with WS and Fetch/XHR filters ready.
* [ ] `KT.md` and this file open on a second screen.
* [ ] Terminal ready in `src/app` with `yarn test:fast` typed but not run.
* [ ] Notifications and chat apps muted.

---

# 0. OPENING — 0:00–0:02

## SAY

"Hello everyone. For the next 45 minutes I'll be handing over the Site Manager application.

We will cover what it is, how the code is organised, how it runs at runtime, and how it gets to production.

The agenda is: architecture, the main features, state and data flow, APIs, the developer workflow, deployment, known issues, and then questions.

Please hold questions for the end unless something blocks you from following. I have a parking lot for anything we run out of time on."

## DO

1. Open VS Code.
2. Show code and github repo.
3. Open unified shell app.
4. Open the Site Manager application.
5. Show the Dashboard.

## EXTRA POINTS

* Repo: `HCECBP-SiteManagerApp`. The app code is under `src/app`; deployment manifests are under `deploy/`.
* Stack: React 18, TypeScript 5, Webpack 5, Tailwind, `@forge/common`, React Query v4, SignalR, LaunchDarkly, Jest, Husky, Yarn.
* Always use `yarn`, never `npm`.

## MEMORY TRIGGER

**What → How → Runtime → Production**

---

# 1. FOUNDATION & MODULE FEDERATION — 0:02–0:09

## 1.1 WHAT IS SITE MANAGER?

### SAY

"Site Manager is a building-management UI. A facility manager can open a site and see alarms, thermostats, lights and energy usage, and can control equipment.

The basic data hierarchy is Organization, then Site, then Asset, then Point.

An asset is a piece of equipment, such as a thermostat or light. A point is an individual value on that equipment, such as temperature, setpoint, mode or on/off status."

### DO

Open the Dashboard.

Point at a thermostat.

Say:

"Here the thermostat is the asset, and this temperature value is one of its points."

### EXTRA POINTS

* Organization and Site IDs come from the shell (`sites` prop) and are used in almost every URL: `/api/v1/orgs/{orgId}/sites/{siteId}/...`.
* A gateway is the device that connects the assets to the cloud. Writes go to a gateway (`/gateways/{id}/SetPointValues`).
* Points have roles (setpoint, temperature, mode, on/off). The Configure widget maps controls to these roles.

---

# 1.2 MODULE FEDERATION

### SAY

"The most important architectural fact about this repository is that it is not a standalone application. It is a Module Federation remote called siteManagerApp.

The unified shell loads this application at runtime.

A simple way to think about it is a shopping mall. The unified shell is the mall, and Site Manager is one shop. We can update the shop without rebuilding the entire mall."

### DO

Open:

`webpack.common.js`

Find:

`ModuleFederationPlugin`

Show:

* name
* filename
* exposes
* shared

### SAY

"The remote name is siteManagerApp.

The remote entry is the front door that the shell loads.

The application exposes the different Site Manager pages, including Dashboard, Alarms, Thermostats, Lights, Reports, Configuration, Points and Others."

### EXTRA POINTS

* **Host = unified shell. Remote = Site Manager.** The host decides when to load us.
* The plugin is `@module-federation/enhanced` (not the old built-in webpack plugin).
* There are 8 exposes. Adding a page means adding an entry in `exposes`.
* The entry file is named `remoteEntry.[contenthash].js`. The hash changes when the build changes, so the shell resolves the entry by its URL rather than a fixed name.
* `publicPath: 'auto'` means chunks are loaded relative to where `remoteEntry` was loaded from, so the same build works on any host URL.
* Think of `remoteEntry` as a **menu**: it lists what we expose and what we share. The shell reads the menu, then loads only what it needs.
* To see it locally: build and look in `dist/` for the `remoteEntry` file.

---

# 1.3 SHELL CONTRACT

### SAY

"The shell essentially needs three things from us: the remote name, the exposed modules and the props passed to those modules.

The important props are sites, userDetails and onTriggerActions."

### DO

Show the exposes section.

Then show:

`src/index.ts`

and

`src/App.tsx`

### SAY

"There is no normal router in this repository because the shell owns the routing."

### EXTRA POINTS

* `src/index.ts` is intentionally almost empty. It is a stub entry; the real entry points are the exposes.
* `onTriggerActions` is how we ask the shell to do something (for example navigate to Alarms). We never navigate ourselves.
* `userDetails` carries the user context; `sites` carries the site list. The code uses `sites[0]`.
* If the shell changes these props, our pages break. Treat them as a **public contract**.

---

# 1.4 SHARED DEPENDENCIES

### SAY

"React, React DOM, React Query and @hcecbp/provider are shared as singletons.

This is important because we don't want multiple copies of React or provider libraries running in the same application."

### DO

Show the `shared` section in webpack.

### EXTRA POINTS

* Singletons: `react`, `react-dom`, `@tanstack/react-query`, `@hcecbp/provider`. All are marked `eager`.
* Why `@hcecbp/provider` matters: it gives us `useClient` (auth token) from the shell. A second copy would not see the shell's context.
* Version mismatch with the shell shows up as console warnings or a blank screen. Check shared versions first.
* Known oddity: one shared entry in `environment.ts` has `requiredVersion: 'sitemanagerapp'`, which looks wrong. Confirm before changing.

---

# 1.5 PROVIDER

### SAY

"Every page uses SiteManagerProvider.

The provider gives us four important things: the toast provider, React Query, SignalR and the permissions gate."

### DO

Open:

`SiteManagerProvider.tsx`

Then:

`PermissionsWrapper.tsx`

### SAY

"Read permission is required to see the application. Write permission is enforced at the widget level."

### EXTRA POINTS

* There is **one** `QueryClient`, created in the provider. Every page shares it when mounted through the provider.
* Permissions come from the Buildings Manager API.
* No read permission means the user cannot use the page (verify the exact message shown).
* Write permission hides or disables controls; the backend still enforces it too.

---

# 1.6 REST + SIGNALR

## MOST IMPORTANT MEMORY

**REST = INITIAL**

**SIGNALR = LIVE**

### SAY

"Data arrives in two main ways.

REST through React Query gives us lists and initial state.

SignalR gives us live updates."

### DO

Open:

`useSignalRSubscription.ts`

Then:

Browser → DevTools → Network → WS

Show a SignalR frame.

### SAY

"When something changes in the field, SignalR sends an update. The page receives it, merges it into the local state and the widget updates without needing to refetch everything."

### MEMORY

**REST starts the screen.**

**SignalR keeps the screen alive.**

### EXTRA POINTS

* Hub library: `@microsoft/signalr` 8.
* Hook: `useSignalRSubscription`. You pass a group and a handler, it manages connect, subscribe and cleanup.
* Reconnect uses increasing delays, up to 10 attempts.
* Reports and the Alarms list do not use SignalR. Only the Dashboard, equipment and the alarm summary do.
* To debug live data: DevTools, Network, WS, then read the frames.

---

# 1.7 FEATURE FLAGS

### SAY

"Feature flags are managed through LaunchDarkly.

Some important flags are configureWidget, enableSchedules, enableTc500, tc500ModeCustomizations and others."

### DO

Open:

`LDFlagUtils.ts`

Optional:

Run:

`yarn test:fast LDFlagUtils`

### EXTRA POINTS

* Flags let us turn features on or off **without a deployment**. This is the first response to a production issue.
* Key flag names: `configureWidget`, `enableSchedules`, `enableTc500`, `tc500ModeCustomizations`.
* Flags are read through a utility, not directly from the LaunchDarkly client, so new flags should be added there.
* When adding a flag: add it to LaunchDarkly, add it to `LDFlagUtils.ts`, write a test for both states.

---

# 2. DASHBOARD — 0:09–0:14

### DO

Open Dashboard.

Point left and right.

### SAY

"The Dashboard has the Summary column on the left and HVAC and Lights grids on the right.

On mobile, it switches between sections using a segmented control."

### EXTRA POINTS

* Summary column includes alarm severity counts and energy (verify the full widget list on screen).
* Desktop and mobile are separate render paths (for example `ReportsPageContent` vs `ReportsPageMobile`). A fix on one often needs the same fix on the other.
* Resize the browser window during the demo to show the mobile layout.
* Known bug: the Dashboard uses `sites[0].siteId` in some places and `.id` in others. Confirm which the shell provides.

---

# 2.1 ONE SIGNALR SUBSCRIPTION

### DO

Open:

`DashboardApp.tsx`

Find:

`handleSignalRUpdates`

### SAY

"The Dashboard makes one SignalR subscription using the dashboard group.

The message handler decides where each update goes.

A point update goes to the appropriate thermostat or light.

Schedule updates go to schedule handling.

Alarm summaries go into AlarmContext.

Manual override updates go into their state."

### DO

Show Network → WS.

---

# 2.2 ALARM DEEP LINK

### SAY

"When a user clicks an alarm severity badge, Dashboard does not navigate itself.

It calls onTriggerActions.

The shell is responsible for navigation."

### DO

Click an alarm badge if the demo environment supports it.

---

# 2.3 DRAG AND DROP

### SAY

"The HVAC and Lights grids use drag and drop.

The order is updated optimistically. The API is called, and if the API fails, the previous order is restored."

### DO

Drag a card.

### EXTRA POINTS

* This is the same pattern as every write in the app: **update UI, call API, revert on failure, toast the error**.
* Order is saved through the API (verify whether it is per user or per site).

---

# 2.4 ENERGY

### SAY

"The Energy widget only renders when the site has a CEM bundle.

It polls periodically, with 10 minutes as the default interval, and pauses when the browser tab is hidden."

### EXTRA POINTS

* Energy is polled over REST, not pushed over SignalR.
* CEM = the energy management bundle. No bundle means no widget and no error.
* If energy looks empty, check the site's bundle first, then the API response.

---

# 3. ALARMS — 0:14–0:18

There are two jobs.

### JOB 1

Alarm page.

### JOB 2

AlarmContext / site-wide summary.

---

# 3.1 ALARM PAGE

### SAY

"The Alarms page is searchable, filterable, sortable and paginated.

The main endpoint is:

POST /metrics/alarmdetails

Paging, search and sorting are handled through the query string, while the filters are sent in the request body."

### DO

Open:

`AlarmsPageContent.tsx`

Then:

`useAlarms.ts`

Then browser Network.

Filter:

`alarmdetails`

Change a filter and show the request.

### EXTRA POINTS

* It is a **POST for reads**, because filters go in the body. Do not be surprised by this.
* The query key includes org, site, page, search, sort and filters, so any change refetches.
* The Alarms module is **read-only**. There is no acknowledge or clear action here.
* Severity colours on the Dashboard cards come from the alarm summary, not from this list.

---

# 3.2 ALARM SUMMARY

### SAY

"The alarm summary initially comes from REST.

SignalR provides live updates.

The alarm list itself does not use SignalR. The live SignalR behavior is for the summary."

### DO

Open:

`AlarmContext.tsx`

Then show a Dashboard equipment card with a severity border.

### EXTRA POINTS

* `AlarmContext` holds the summary for the whole site, so any widget can read it.
* Known gap: there is no per-asset REST fallback. If SignalR misses an update, the card can be stale until reload.
* Clicking a badge calls `onTriggerActions`; the shell opens the Alarms page.

---

# 4. EQUIPMENT CONTROL — 0:18–0:24

## MAIN MEMORY

**LIST → CLICK → OPTIMISTIC → API → SIGNALR**

### SAY

"Thermostats and Lights follow a similar pattern.

We load the assets, display a widget for each asset, allow the user to make a change, update the UI optimistically, send the command and then receive confirmation through SignalR."

### DO

Open Thermostats.

---

# 4.1 SETPOINT API

### DO

Open:

`useSetPointValues.ts`

### SAY

"Every equipment write goes through the SetPointValues endpoint.

The request contains the gateway information, system GUID, a command ID and the point/value pairs."

Endpoint:

`POST /gateways/{gatewayId}/SetPointValues`

### EXTRA POINTS

* The command ID lets us match the later SignalR confirmation to the request we sent.
* Lights and Thermostats share this one write path.
* The write is accepted by the API first; the actual device change arrives later via SignalR. Success on the API is not the same as the device changing.

---

# 4.2 POWER TOGGLE

### SAY

"For a power toggle, the UI changes first.

We send the command.

If the command fails, we revert the UI and show a toast."

### DO

Open:

`Thermostats.tsx`

Find:

`toggleWidget`

---

# 4.3 SETPOINT BUTTONS

### SAY

"The plus and minus buttons change the setpoint by 0.5.

The write is debounced by 800 milliseconds per thermostat."

### DO

On a test site:

1. Click + several times quickly.
2. Open Network.
3. Show that only one request is sent after the debounce.

### IMPORTANT

"Because this changes real equipment, only demonstrate this on a test site."

---

# 4.4 UNITS

### SAY

"One important detail: there is no unit conversion in this logic. Units are display-only."

### EXTRA POINTS

* `useUpdateSchedule` hardcodes `°C`. For a site that uses °F this would be wrong.
* Schedule editing has a permission gate bug. Confirm the behaviour before changing it.

---

# 4.5 TC500 (PART OF EQUIPMENT BLOCK, ~2 MINUTES)

### SAY

"TC500 is a thermostat model detected from the model name.

It gets a slider widget on the Dashboard and is controlled through the enableTc500 feature flag.

The slider configuration depends on the current schedule and configuration points."

### DO

Open:

`Thermostats.tsx`

Find:

`getSliderConfigForMode`

### IMPORTANT WARNING

"One fragile area to be aware of is that getSliderConfigForMode can trigger writes during render."

### EXTRA POINTS

* TC500 detection is by model name, so a new model naming scheme would need a code change.
* Both `enableTc500` and `tc500ModeCustomizations` must be considered when debugging.
* Writes during render are a React anti-pattern. It is on the known-issues list to fix first.

---

# 6. CONFIGURATION, POINTS & REPORTS — 0:24–0:28

# 6.1 CONFIGURATION

### SAY

"Configuration is the admin screen.

It controls which HVAC and Light assets appear on the Dashboard."

### DO

Open Configuration.

Show:

`POST /configurations`

and

`PUT /configurations/remove`

### SAY

"Showing an asset uses POST.

Hiding an asset uses PUT.

The UI is optimistic and reverts if the request fails."

### EXTRA POINTS

* Known bug: `Configuration.tsx` spreads `{...sites}` (an array) where a single site is expected. Do not copy this pattern.
* `removeConfiguration` ignores the `type` argument.
* Configuration visibility depends on permissions (verify the exact permission name).

---

# 6.2 CONFIGURE WIDGET

### SAY

"The Configure modal maps widget controls to point roles.

It requires the ConfigureWidget permission and the configureWidget feature flag.

After applying the configuration, we re-subscribe to SignalR so the new points start receiving live updates."

### DO

Open the Configure modal if available.

Otherwise show:

`ConfigurationWidget.tsx`

`handleApply`

### EXTRA POINTS

* Needs **both** the `ConfigureWidget` permission and the `configureWidget` flag.
* Re-subscribing after apply is what makes new points start updating without a page reload.
* There is also a Points page (verify what it shows before demoing).

---

# 6.3 REPORTS

### SAY

"Reports is relatively simple.

It supports list, search, sort and PDF download."

### DO

Open:

`reportDownload.ts`

### SAY

"The download fetches a blob, creates an object URL and triggers a temporary download link."

### EXTRA POINTS

* The app has **no fixed list of reports**. It shows whatever the backend returns for the site. Each item has only `id`, `name` and `createdDate`.
* List: `GET /api/v1/orgs/{o}/sites/{s}/reports` with `Page`, `PerPage`, `Search`, `SortBy`, `Sort`.
* Download: `GET .../reports/{id}/download`. The file name is `<name>.pdf`; the `.pdf` is hardcoded.
* Sort is by `ReportName` or `CreatedDate`. Default is `CreatedDate` descending.
* There is no permission check and no SignalR on this page.
* What the reports contain and how they are generated: ask the backend team (verify in Swagger).

---

# 7. STATE MANAGEMENT — 0:28–0:32

# FOUR HOMES

## 1. React Query

**SERVER STATE**

## 2. SignalR

**LIVE STATE**

## 3. React Context

**SHARED UI STATE**

## 4. Component State

**LOCAL STATE**

### SAY

"There is no Redux, Zustand, Recoil or Jotai.

When we add a new feature, we should first decide which of these four places the state belongs to."

### DECISION RULE

* Comes from an API and can be refetched? **React Query.**
* Pushed live from the hub? **SignalR into component state.**
* Needed by several distant components? **Context.**
* Only one component cares? **`useState`.**

Note: an `@store/*` alias exists in the docs but not in the config. Do not rely on it.

### DO

Show examples:

`useAlarms.ts`

`DashboardApp.tsx`

`ScheduleProvider.tsx`

`Thermostats.tsx`

---

# 7.1 REACT QUERY

### SAY

"The query key convention is feature first, then organisation, then site, then the specific parameters.

This means switching sites produces a different cache entry."

### IMPORTANT

"Hooks own the API calls. Components do not call fetch directly."

### EXTRA POINTS

* Example key: `['reports', orgId, siteId, page, perPage, search, sort, order]`.
* Anything that changes the response must be in the key, or the cache will show wrong data.
* Version note: the app uses React Query v4; the devtools package is ^5.81. Verify compatibility before upgrading either.

---

# 7.2 SIGNALR LIFECYCLE

### SAY

"SignalR subscriptions follow what is visible.

When the visible IDs change, the subscription changes.

The hook uses sorted IDs, a serialized subscription key and a ref for the latest callback to avoid unnecessary subscriptions."

### WARNING

"One known weakness is that the provider context value is rebuilt on every render, which can cause additional subscriptions."

### EXTRA POINTS

* Fix idea: memoise the provider context value.
* Symptom to watch for: repeated subscribe frames in Network, WS.
* The latest-callback ref pattern lets the handler change without re-subscribing.

---

# 8. APIs — 0:32–0:35

# THREE PLACES

## 1. SITE MANAGER API

**DATA**

* Assets
* Points
* Alarms
* Overrides
* Configuration
* Reports
* Setpoints

## 2. BUILDINGS MANAGER API

**PLATFORM**

* Permissions
* Subscriptions
* Site logo
* Calendar

## 3. SIGNALR

**LIVE DATA**

### MEMORY SENTENCE

> "Site Manager owns the data, Buildings Manager owns platform information, and SignalR owns live updates."

---

# 8.1 HOW AN API CALL WORKS

### SAY

"Most hooks follow the same pattern.

We get the token provider from the shell, build the query key, get a fresh token, construct the URL with organisation and site IDs, call fetch with authorization, check the response and return the JSON."

### EXTRA POINTS

* Base URLs come from `environment.ts`, which reads the runtime-injected values.
* Token comes from `useClient('findApi')`. We never store tokens ourselves.
* A good file to copy as a template for a new read hook: `useReports.ts`.
* A good template for a write hook: `useSetPointValues.ts`.

---

# 8.2 IMPORTANT ENDPOINTS

The four endpoints to know:

### ALARMS

`POST /metrics/alarmdetails`

### EQUIPMENT WRITE

`POST /gateways/{id}/SetPointValues`

### SHOW

`POST /configurations`

### HIDE

`PUT /configurations/remove`

### ALSO USEFUL

### REPORTS

`GET /api/v1/orgs/{o}/sites/{s}/reports`

`GET .../reports/{id}/download`

Full list is in `KT/09-APIs-and-Endpoints.md`.

---

# 9. DEV WORKFLOW — 0:35–0:39

# MEMORY

**RUN → TEST → LINT → BUILD → CI**

### COMMANDS

`yarn host`

`yarn start`

`yarn test:fast`

`yarn test`

`yarn lint:all`

`yarn format:all`

`yarn build`

### SAY

"These are the main commands we use during development."

### EXTRA POINTS

* `yarn host` runs the app inside a local host for layout work; `yarn start` runs the standalone dev server (verify against `package.json` before the session).
* `yarn test:fast` is for quick feedback; run full `yarn test` with coverage before a PR.
* Target coverage is at least 80%.
* New component = folder with `.tsx`, `.spec.tsx`, `.stories.tsx`, and the copyright header.
* Tailwind only. No custom CSS and no inline styles.
* Imports use aliases like `@components/*`.

---

# 9.1 LINT

### SAY

"Linting enforces several rules.

There should be a copyright header, no console calls, React component conventions, handler naming conventions and other code-quality rules."

---

# 9.2 TESTING

### SAY

"We use Jest and React Testing Library.

SignalR is mocked globally, so tests don't establish real SignalR connections."

### HOOK TEST RECIPE

**mock useClient**

↓

**mock fetch**

↓

**QueryClient**

↓

**renderHook**

↓

**waitFor**

---

# 9.3 QUALITY GATES

### SAY

"These checks protect the repository before changes reach the main branches."

### IMPORTANT

"Never bypass the checks using --no-verify."

### EXTRA POINTS

* Husky runs the hooks. See `HUSKY_SETUP.md` and `HUSKY_QUICK_REFERENCE.md` in the repo root.
* If a hook fails, fix the cause; do not skip it.
* Use conventional commit messages.

---

# 10. BUILD & DEPLOYMENT — 0:39–0:42

# MOST IMPORTANT MEMORY

## BUILD ONCE → CONFIGURE AT RUNTIME

### DO

Open:

`setup.cake`

Then:

`Dockerfile`

Then:

`env-variables.sh`

### SAY

"The React application is built into dist.

The Docker image contains the static application and nginx."

Then the key point:

"The same image can be used across environments."

---

# 10.1 ENVIRONMENT INJECTION

### SAY

"The application contains placeholders such as $SiteManagerApiBaseUrl.

When the container starts, the startup script uses environment variables to replace those placeholders."

### MEMORY

**Same image. Different environment variables.**

### EXTRA POINTS

* Build output includes an `env.[contenthash].js` chunk holding the placeholders. `envsubst` rewrites it at container start.
* New variable checklist: placeholder in code, entry in the startup script, value in each environment's deployment file.
* Deploy folders: `deploy/application`, `deploy/acceptance`, `deploy/performance`, `deploy/featureToggle`.

---

# 10.2 NGINX / KUBERNETES

### SAY

"nginx serves the application.

The container runs on port 3000.

The Kubernetes deployment has startup, liveness and readiness probes."

### DO

Show:

`nginx/nginx.conf`

Then:

`aks-deployment.yml`

### EXTRA POINTS

* There are both AKS and OpenShift deployment files. Check which one your environment uses.
* nginx has no `/health/*` route. Confirm what the probes actually hit.
* The no-cache rule targets `manager-bundle.js`, not `remoteEntry`. Since `remoteEntry` has a content hash this is probably fine, but confirm how the shell references it.

---

# 11. KNOWN ISSUES — 0:42–0:43

Do NOT try to memorize the entire tech-debt list.

The six to know:

## 1. Configuration

`{...sites}` spread.

## 2. Dashboard

`siteId` vs `id`.

## 3. Schedule

Hardcoded °C / edit permission behavior.

## 4. SignalR

Provider context can cause extra subscriptions.

## 5. TC500

Potential write during render.

## 6. Deployment

Health endpoints and remoteEntry caching need confirmation.

### SAY

"These are the areas I would look at first when taking ownership."

### PRIORITY

| Risk | Impact | Effort |
|---|---|---|
| TC500 write during render | Unexpected device writes | Small |
| Provider context rebuilt | Extra subscriptions | Small |
| Config `{...sites}` spread | Wrong data sent | Small |
| Dashboard `siteId` vs `id` | Wrong site in some calls | Small |
| Schedule hardcoded °C | Wrong units | Medium |
| Deployment health and caching | Rollout surprises | Needs confirmation |

---

# 12. NEW OWNER HANDOVER — 0:43–0:44

### SAY

"If I were taking ownership of this application, my first steps would be:

First, run yarn host and yarn test:fast.

Second, read KT.md.

Third, trace one complete feature end to end.

The thermostat power toggle is a good example because it takes us from the UI, through the API, and back through SignalR.

Fourth, pick one small known issue and fix it with a test. That is the fastest way to learn the codebase."

### FIRST-WEEK CHECKLIST

* Get repo access, run `yarn install`, then `yarn host`.
* Get LaunchDarkly access and a test site in the shell.
* Read `KT.md`, then `KT/09` (APIs) and `KT/10` (Module Federation).
* Confirm backend contacts for Site Manager API, Buildings Manager API and the SignalR hub.
* Confirm the deployment runbook and rollback process.

---

# 13. CONCLUSION — 0:44–0:45

# FIVE THINGS

## 1. ARCHITECTURE

**Federated React remote**

## 2. STATE

**REST + React Query**

## 3. LIVE DATA

**SignalR**

## 4. WRITES

**Optimistic + revert/confirmation**

## 5. DEPLOYMENT

**Same image + runtime configuration**

---

# FINAL SCRIPT

Say:

"To summarise in five points.

First, Site Manager is a federated React remote loaded by the unified shell.

Second, REST and React Query provide the initial server state, while SignalR provides live updates.

Third, permissions and feature flags control what users can access and which features are enabled.

Fourth, equipment writes are optimistic and are confirmed or reverted based on the result.

And finally, we build one image and inject the environment configuration when the container starts.

For a new owner, I would start by running the application and tests, reading the KT documentation and tracing one complete flow end to end.

The thermostat power toggle is a good example because it takes us from the UI, through the API and back through SignalR.

That's the overview. Happy to take questions."

---

# QUICK Q&A CHEAT SHEET

## Why is there no router?

The unified shell owns routing. Each exposed component represents a page.

## Can we use multiple sites?

The current implementation uses `sites[0]`.

## Can users acknowledge alarms?

Not in this module. The Alarms module is read-only.

## Why not keep live values in React Query?

SignalR updates are pushed directly into widget state rather than constantly updating the React Query cache.

## Do we need the backend to run locally?

For layout work, the stub environment points toward QA URLs. Real data requires a valid token from the shell.

## Do we build separately for every environment?

No.

**Same Docker image + different environment variables.**

## How do we disable a risky feature?

Use the LaunchDarkly feature flag. No deployment is required.

## What happens if SignalR goes down?

The application retries the connection with increasing delays, up to 10 attempts.

## How do I add a runtime environment variable?

Add the placeholder/stub, add the deployment environment variable, and configure it for each environment.

## How do I roll back?

Redeploy the previous image tag. Confirm the exact operational process from the deployment runbook.

## Which reports exist?

The app has no fixed list. It shows what the backend returns for the site (name and created date) and downloads each as PDF. Ask the backend team for the report catalogue.

## Why is Alarms a POST?

Filters are sent in the request body. It is still a read.

## Where do I add a new page?

Create the component, wrap it in `SiteManagerProvider`, add it to `exposes` in `webpack.common.js`, and tell the shell team to route to it.

## Where do I add a new feature flag?

Create it in LaunchDarkly, add it to `LDFlagUtils.ts`, and test both states.

## How do I debug a blank screen in the shell?

Check the console for shared-version warnings, check that `remoteEntry` loaded in Network, and check the env chunk values.

## Why does the page not update live?

Check Network, WS for the subscription and incoming frames. Then check the subscribed IDs and group.

## Is mobile a separate app?

No. Same app, separate render paths for some pages.

## Who owns the unified shell?

The unified shell team. Confirm contacts before the KT ends.

---

# IF YOU ARE RUNNING OUT OF TIME

| Cut | Saves |
|---|---|
| Skip the drag-and-drop demo | 1 min |
| Skip the 800 ms debounce demo | 1 min |
| Show TC500 as a slide-only mention | 1 min |
| Skip the lint rules detail | 1 min |
| Compress the state section to the four-homes list | 2 min |

**Never cut:** Module Federation, REST vs SignalR, the optimistic-write pattern, build once and configure at runtime, and the handover steps.

# IF YOU ARE RUNNING AHEAD

* Trace the thermostat power toggle end to end live.
* Show a hook test and run it.
* Show the `dist/` folder and find `remoteEntry`.
* Walk through the known-issues table in more detail.

# PARKING LOT

Write unanswered questions here during the session and send answers afterwards:

*
*
*

---

# LAST-MINUTE 60-SECOND REVISION

Before the KT starts, look at only this:

```text
SITE MANAGER
Building management UI

ARCHITECTURE
Org → Site → Asset → Point
Remote → Unified Shell
Name → Exposes → Props

PROVIDER
Toast + Query + SignalR + Permissions

DATA
REST = Initial
SignalR = Live

DASHBOARD
Summary + HVAC + Lights
One subscription → many widgets
Drag → Optimistic → API → Revert

ALARMS
Page + Summary
alarmdetails
REST → SignalR

EQUIPMENT
List → Click → Optimistic → API → SignalR
SetPointValues
0.5 step
800ms debounce

TC500
Slider + Schedule + Limits
WARNING: write during render

CONFIG
POST = Show
PUT = Hide
Control → Point Role → Apply → Resubscribe

REPORTS
List + Search + Sort + PDF
No fixed list; backend decides

STATE
React Query = Server
SignalR = Live
Context = Shared
Component = Local

APIs
Site Manager = Data
Buildings Manager = Platform
SignalR = Live

DEV
Run → Test → Lint → Build → CI

DEPLOY
Build → Docker → nginx
Same image
Runtime env injection
Port 3000

ENDING
Federated React
REST
SignalR
Optimistic writes
Same image / runtime config
```

**If you get stuck while presenting:** don't try to recall the exact sentence. Look at the current section heading, explain it in your own words, and then do the corresponding screen action. That will sound much more natural than memorizing a 45-minute script.
