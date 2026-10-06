# Global Control Frontend: Product Requirements Document (reverse-engineered)

This spec was reconstructed from the shipped frontend code, not written before it. It describes what the app does today. Where behavior is a guess from naming or structure, that is stated. The backend was not reviewed.

## 1. Problem statement

Small businesses and agencies use several separate tools for contact lists, email and SMS follow-up, sales pipelines, booking pages and chat. Data is split across them, and automation across tools is fragile. Agencies also need to run these tools for many clients under their own brand.

Global Control puts these in one app. Contacts are the center. Tags and workflows turn contact events into automatic actions. Agency features let a reseller manage client accounts with white-label branding.

## 2. Target users

| User | What they need |
|---|---|
| Business owner or marketer | Import contacts, send broadcasts, build follow-up sequences, see what worked |
| Sales user | Track leads on a pipeline, log activity, book appointments |
| Agency | Create and manage client accounts, brand the login and menu, connect custom domains |
| Enterprise team member (sub-user) | Work inside a parent account |
| Contact or lead (no login) | Book, cancel or reschedule an appointment, chat with an embedded bot, view a shared pipeline |

## 3. Scope

### 3.1 Implemented (working UI wired to backend calls)

- Login, registration, password reset, enterprise login, white-label login, auto-login from a platform link
- Contacts: list, filter, bulk tag and delete, import, detail tabs
- Tag manager, tag groups, tag activity, failed tag re-fire
- Workflows: step editor, canvas editor, groups, clone, share and import by code, goal tracking, per-workflow domain and SMS number, test send
- Broadcast email (current and legacy versions), templates, AI email helper, engagement and re-engagement campaigns
- Broadcast SMS and Twilio number purchase and management
- Email domain setup (Mailgun and SMTP)
- Reporting: email overview, broadcast, newsletter, workflow reports, re-engagement dashboard, uptime and broadcast monitoring, live queue log
- Pipelines: board, list and activity views, deals, lead import, read-only public pipeline view
- Appointments: configuration, groups, availability, public booking, cancel and reschedule, Google, Outlook and Zoom connections
- AI chat bots: builder, embed (page and widget), public view
- Integrations catalog (API key and OAuth), inbound webhooks, Facebook lead ads
- Smart lists, custom lists, custom fields and groups
- Agency client management, white-label branding, CNAME domains, enterprise sub-users
- Image library, QR codes, rewards, onboarding video screens

### 3.2 Scaffolded or partial

- **Billing.** Route and menu entry exist, the page is empty.
- **Agency purchase.** A single button that calls the purchase endpoint and redirects. No plan selection or form.
- **General settings.** Domains only. The search box does nothing.
- **Main chat screen.** A static copy of a template chat UI with placeholder data. The chat bot builder is the real chat feature.
- **Appointment payments.** Configuration defaults to a PayPal gateway, but no payment flow was found in the frontend.
- **Several legacy duplicates** (old dashboards, old workflow editors, old integration page) remain routable or in the tree and may not match current behavior.
- **Map report.** The Google Maps key is an empty string in the component, so the map likely does not load.

### 3.3 Out of scope for this frontend

- Server logic: sending, scheduling, queueing, classification of contact status, authorization
- Billing and subscription handling (no payment provider code found in the frontend)
- Mobile apps
- Translation and localization
- Automated tests

## 4. Key user flows

### 4.1 Log in

1. User opens the app. If a token is stored, the app loads the current user and starts the socket connection. Otherwise it redirects to `/login` and keeps the target path as a callback.
2. User submits email and password. The token is stored locally and the app opens the dashboard or the callback path.
3. Agency users can enter a client account. The app then sends a second client token with each request.
4. White-label users sign in on `/application/<permalink>/login` and see the agency's branding.

### 4.2 Import contacts and tag them

1. User uploads a CSV or spreadsheet on the import page.
2. The user maps columns, picks tags and starts the import.
3. The contact table shows the new people. Tag activity shows tag actions running, then completed or failed. Failed tags can be re-fired.

### 4.3 Build and run a workflow

1. User creates a workflow in a group and adds steps: email, SMS, timer, split, tag changes, move to another workflow, integration call, custom field.
2. User sets a sending domain and phone number, runs a test send, and sets an optional goal.
3. A tag action starts the workflow for a contact. The workflow pages show queues as active, frozen, completed or goal met.

### 4.4 Send a broadcast

1. User chooses recipients by status segment (new, active, inactive, dead and so on) and sees the count.
2. User picks a sending domain or SMTP config, writes the message (optionally with the AI helper), and sends.
3. Reports show delivery and engagement. Re-engagement campaigns target inactive contacts, and the dashboard shows who came back.

### 4.5 Work a pipeline

1. User opens a pipeline and sees stages as columns.
2. User adds or imports leads, drags them between stages, logs activities and attaches a deal.
3. Dropping a lead on won or lost closes it.

### 4.6 Book an appointment (public)

1. A visitor opens a booking link and sees available slots in their time zone.
2. The visitor registers and gets a confirmation page.
3. The visitor can later cancel or reschedule from a link.

### 4.7 Agency onboards a client

1. Agency creates a client and sets white-label branding.
2. Agency optionally connects a custom domain.
3. Client signs in on the branded login page and uses the white-label shell.

## 5. Non-functional characteristics

- **Platform.** Browser SPA, desktop-first admin layout. Light mode is forced.
- **Realtime.** Queue logs, tag task states and notifications update over Socket.io.
- **Configuration.** API and socket URLs come from `VITE_*` environment variables, with a few hardcoded exceptions.
- **Deployment.** Manual build and file copy to a web server. No CI.
- **Security model.** Authorization is the backend's job. The frontend hides menu items by plan but has no route-level role checks.

## 6. Risks

| Risk | Detail | Effect |
|---|---|---|
| Code duplication | Multiple versions of the same feature, backup files tracked in git | Hard to know which version is live, bugs fixed in one copy only |
| No tests or linting | Nothing catches regressions before a manual deploy | Releases depend on manual checking |
| Old dependencies | Vite 2, Axios 0.26, an old SheetJS, mixed CKEditor versions | Security and compatibility debt, slow upgrades |
| Hardcoded hosts | Some views ignore the environment variable | Features fail on dev or local builds |
| Tracked env files | Currently URLs only | A future key added there would be committed |
| Heavy bundle | Four rich text editors, three date pickers, large inline images in one component | Slower loads |
| Frontend authorization | Menu hiding only | Backend must enforce every permission |
| Inconsistent state code | Mixed store styles, a few components call the HTTP client directly | Harder maintenance, breaks the stated service-layer rule |
| Manual deploys | Copy to server, no rollback automation beyond a backup copy | Release errors are slow to undo |

## 7. Open questions

- Which of the duplicate views (dashboards, workflow editors, integrations) are still meant to be reachable?
- Is billing handled outside this app or still planned?
- Is appointment payment meant to be live?
