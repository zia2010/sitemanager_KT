# SITE MANAGER — 45-MINUTE KT SPEAKER REFERENCE

Remember the keywords and flow:

**WHAT → ARCHITECTURE → FEATURES → DATA → APIs → DEV → DEPLOYMENT → HANDOVER**

Main sentence:

> **Site Manager is a building-management UI delivered as a Module Federation remote, where REST gives us the state and SignalR gives us live updates.**

---

# 0. OPENING — 0:00–0:02

## REMEMBER

**WHAT → HOW → RUNTIME → PRODUCTION**

## SAY

"Hello everyone. For the next 30 minutes I'll be handing over the Site Manager application — what it is, explain the code, how to deploy.

## DO

1. Open VS Code.
2. Show code and github repo.
3. Open unified app.
4. Open the Site Manager application.
5. Show the Dashboard.

## MEMORY TRIGGER

**What → How → Runtime → Production**

---

# 1. FOUNDATION & MODULE FEDERATION — 0:02–0:09

## 1.1 WHAT IS SITE MANAGER?

### REMEMBER

**Organization → Site → Asset → Point**

### SAY

"Site Manager is a building-management UI. A facility manager can open a site and see alarms, thermostats, lights and energy usage, and can control equipment.

The basic data hierarchy is Organization, then Site, then Asset, then Point.

An asset is a piece of equipment, such as a thermostat or light. A point is an individual value on that equipment, such as temperature, setpoint, mode or on/off status."

### DO

Open the Dashboard.

Point at a thermostat.

Say:

"Here the thermostat is the asset, and this temperature value is one of its points."

---

# 1.2 MODULE FEDERATION

## REMEMBER

**NOT STANDALONE → REMOTE → SHELL**

### SAY

"The most important architectural fact about this repository is that it is not a standalone application. It is a Module Federation remote called HCEBP-siteManagerApp.

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

---

# 1.3 SHELL CONTRACT

## REMEMBER

**NAME → EXPOSES → PROPS**

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

---

# 1.4 SHARED DEPENDENCIES

## REMEMBER

**REACT = SINGLETON**

### SAY

"React, React DOM, React Query and @hcecbp/provider are shared as singletons.

This is important because we don't want multiple copies of React or provider libraries running in the same application."

### DO

Show the `shared` section in webpack.

---

# 1.5 PROVIDER

## REMEMBER

**TOAST + QUERY + SIGNALR + PERMISSIONS**

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

---

# 1.7 FEATURE FLAGS

## REMEMBER

**LAUNCHDARKLY**

### SAY

"Feature flags are managed through LaunchDarkly.

Some important flags are configureWidget, enableSchedules, enableTc500, tc500ModeCustomizations and others."

### DO

Open:

`LDFlagUtils.ts`

Optional:

Run:

`yarn test:fast LDFlagUtils`

---

# 2. DASHBOARD — 0:09–0:14

## REMEMBER

**SUMMARY + HVAC + LIGHTS**

### DO

Open Dashboard.

Point left and right.

### SAY

"The Dashboard has the Summary column on the left and HVAC and Lights grids on the right.

On mobile, it switches between sections using a segmented control."

---

# 2.1 ONE SIGNALR SUBSCRIPTION

## REMEMBER

**ONE SUBSCRIPTION → MANY WIDGETS**

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

### REMEMBER

**DASHBOARD → SHELL → ALARMS**

### SAY

"When a user clicks an alarm severity badge, Dashboard does not navigate itself.

It calls onTriggerActions.

The shell is responsible for navigation."

### DO

Click an alarm badge if the demo environment supports it.

---

# 2.3 DRAG AND DROP

## REMEMBER

**DRAG → OPTIMISTIC → API → REVERT**

### SAY

"The HVAC and Lights grids use drag and drop.

The order is updated optimistically. The API is called, and if the API fails, the previous order is restored."

### DO

Drag a card.

---

# 2.4 ENERGY

## REMEMBER

**CEM → ENERGY → 10 MINUTES**

### SAY

"The Energy widget only renders when the site has a CEM bundle.

It polls periodically, with 10 minutes as the default interval, and pauses when the browser tab is hidden."

---

# 3. ALARMS — 0:14–0:18

## REMEMBER

**PAGE + SUMMARY**

There are two jobs.

### JOB 1

Alarm page.

### JOB 2

AlarmContext / site-wide summary.

---

# 3.1 ALARM PAGE

### REMEMBER

**SEARCH → FILTER → SORT → PAGE**

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

---

# 3.2 ALARM SUMMARY

## REMEMBER

**REST FIRST → SIGNALR LIVE**

### SAY

"The alarm summary initially comes from REST.

SignalR provides live updates.

The alarm list itself does not use SignalR. The live SignalR behavior is for the summary."

### DO

Open:

`AlarmContext.tsx`

Then show a Dashboard equipment card with a severity border.

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

### REMEMBER

**SetPointValues**

### DO

Open:

`useSetPointValues.ts`

### SAY

"Every equipment write goes through the SetPointValues endpoint.

The request contains the gateway information, system GUID, a command ID and the point/value pairs."

Endpoint:

`POST /gateways/{gatewayId}/SetPointValues`

---

# 4.2 POWER TOGGLE

## REMEMBER

**FLIP → SEND → REVERT**

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

## REMEMBER

**0.5 STEP → 800ms**

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

---

# 5. TC500 — 0:21–0:22

## REMEMBER

**TC500 = SLIDER + SCHEDULE + LIMITS**

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

---

# 6. CONFIGURATION, POINTS & REPORTS — 0:24–0:28

# 6.1 CONFIGURATION

## REMEMBER

**POST = SHOW**

**PUT = HIDE**

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

---

# 6.2 CONFIGURE WIDGET

## REMEMBER

**CONTROL → POINT ROLE → APPLY → RESUBSCRIBE**

### SAY

"The Configure modal maps widget controls to point roles.

It requires the ConfigureWidget permission and the configureWidget feature flag.

After applying the configuration, we re-subscribe to SignalR so the new points start receiving live updates."

### DO

Open the Configure modal if available.

Otherwise show:

`ConfigurationWidget.tsx`

`handleApply`

---

# 6.3 REPORTS

## REMEMBER

**LIST → SEARCH → SORT → PDF**

### SAY

"Reports is relatively simple.

It supports list, search, sort and PDF download."

### DO

Open:

`reportDownload.ts`

### SAY

"The download fetches a blob, creates an object URL and triggers a temporary download link."

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

### DO

Show examples:

`useAlarms.ts`

`DashboardApp.tsx`

`ScheduleProvider.tsx`

`Thermostats.tsx`

---

# 7.1 REACT QUERY

## REMEMBER

**FEATURE → ORG → SITE → DETAILS**

### SAY

"The query key convention is feature first, then organisation, then site, then the specific parameters.

This means switching sites produces a different cache entry."

### IMPORTANT

"Hooks own the API calls. Components do not call fetch directly."

---

# 7.2 SIGNALR LIFECYCLE

## REMEMBER

**VISIBLE → SUBSCRIBE → UPDATE → MERGE**

### SAY

"SignalR subscriptions follow what is visible.

When the visible IDs change, the subscription changes.

The hook uses sorted IDs, a serialized subscription key and a ref for the latest callback to avoid unnecessary subscriptions."

### WARNING

"One known weakness is that the provider context value is rebuilt on every render, which can cause additional subscriptions."

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

## REMEMBER

**TOKEN → URL → FETCH → JSON**

### SAY

"Most hooks follow the same pattern.

We get the token provider from the shell, build the query key, get a fresh token, construct the URL with organisation and site IDs, call fetch with authorization, check the response and return the JSON."

---

# 8.2 IMPORTANT ENDPOINTS

Remember these four:

### ALARMS

`POST /metrics/alarmdetails`

### EQUIPMENT WRITE

`POST /gateways/{id}/SetPointValues`

### SHOW

`POST /configurations`

### HIDE

`PUT /configurations/remove`

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

---

# 9.1 LINT

### REMEMBER

**NO CONSOLE + HEADER + REACT RULES**

### SAY

"Linting enforces several rules.

There should be a copyright header, no console calls, React component conventions, handler naming conventions and other code-quality rules."

---

# 9.2 TESTING

## REMEMBER

**JEST + RTL + MOCKS**

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

## REMEMBER

### PRE-COMMIT

**LINT → BUILD → FAST TESTS**

### CI

**LINT → COVERAGE → SONAR → SECURITY**

### SAY

"These checks protect the repository before changes reach the main branches."

### IMPORTANT

"Never bypass the checks using --no-verify."

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

## REMEMBER

**PLACEHOLDER → envsubst → REAL VALUE**

### SAY

"The application contains placeholders such as $SiteManagerApiBaseUrl.

When the container starts, the startup script uses environment variables to replace those placeholders."

### MEMORY

**Same image. Different environment variables.**

---

# 10.2 NGINX / KUBERNETES

## REMEMBER

**NGINX → PORT 3000 → PROBES**

### SAY

"nginx serves the application.

The container runs on port 3000.

The Kubernetes deployment has startup, liveness and readiness probes."

### DO

Show:

`nginx/nginx.conf`

Then:

`aks-deployment.yml`

---

# 11. KNOWN ISSUES — 0:42–0:43

Do NOT try to memorize the entire tech-debt list.

Remember these six:

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

---

# 12. NEW OWNER HANDOVER — 0:43–0:44

## REMEMBER

**RUN → READ → TRACE → VERIFY**

### SAY

"If I were taking ownership of this application, my first steps would be:

First, run yarn host and yarn test:fast.

Second, read KT.md.

Third, trace one complete feature end to end.

The thermostat power toggle is a good example because it takes us from the UI, through the API, and back through SignalR."

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

---

# 🚨 LAST-MINUTE 60-SECOND REVISION

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
⚠️ write during render

CONFIG
POST = Show
PUT = Hide
Control → Point Role → Apply → Resubscribe

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

**If you get stuck while presenting:** don't try to remember the exact sentence. Look at the current **bold memory phrase**, explain it in your own words, and then do the corresponding screen action. That will sound much more natural than memorizing a 45-minute script.
