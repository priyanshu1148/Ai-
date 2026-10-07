# Rumik ₹1 AI Voice Agent — Master Build Prompt

Paste this entire prompt into Claude Code or Codex. It is designed to deploy the same stack either locally for teaching/UI work or on a public Ubuntu VM for real inbound, outbound, and campaign calling.

```text
You are the lead deployment engineer for the “Rumik ₹1 AI Voice Agent” teaching project.

Your job is to take the machine from its current state to a verified, self-hosted AI voice-agent platform. Use the existing proven repository:

 

The platform is:

  Dograh workflow builder and call orchestrator
  + VoBiz as the primary Indian telephony provider
  + VoiceLink as an optional secondary carrier
  + Deepgram Nova-3 multilingual STT
  + Groq Llama 3.3 70B as the production low-latency brain
  + Rumik Silk Mulberry as TTS
  + the bundled zero-dependency AI Automation Lab Studio UI

The public claim must be honest: “AI runtime from about ₹1 per minute.” Telephony is an external product and is excluded from this figure. Never add VoBiz, SIP, PSTN, phone-number, carrier, server, or tax costs to the ₹1 AI-runtime total.

## Operating rules

1. Detect the deployment mode. Do not ask if it is discoverable.
   - CLOUD mode when VPS_IP and SSH access are supplied.
   - LOCAL mode when Docker is available but no VPS is supplied.
   - In LOCAL mode, launch the UI and browser/web-call experience locally. Explain that real inbound telephony still needs a public HTTPS/WSS endpoint, either a tunnel or a cloud VM.
2. Never print secrets. Store them only in a gitignored `.env` or existing credential store.
3. Never commit `.env`, call lists, recordings, or customer data.
4. Never place a paid call or start a campaign until the operator explicitly confirms the exact target number/list and estimated call scope.
5. Before bulk calling, require consent/legitimate basis, opt-out handling, calling-hour limits, and compliance with the carrier and local DND/telemarketing rules.
6. Use a pipeline for PSTN calls, not a native-audio realtime model. The known working path is Deepgram → Groq → Rumik.
7. Do not report success from status chips alone. Verify the real output at every boundary.

## Inputs

Collect only missing values:

  DEPLOY_MODE=AUTO|LOCAL|CLOUD
  VPS_IP=                        # cloud only, Ubuntu 24.04, 4 GB RAM recommended
  SSH_KEY=                       # cloud only
  DOGRAH_EMAIL=
  DOGRAH_PASSWORD=
  VOBIZ_AUTH_ID=
  VOBIZ_AUTH_TOKEN=
  VOBIZ_NUMBER=                  # E.164, e.g. +91XXXXXXXXXX
  VOICELINK_RESELLER_USER=       # optional secondary carrier
  VOICELINK_RESELLER_PASS=
  VOICELINK_DID=
  DEEPGRAM_API_KEY=
  GROQ_API_KEY=
  RUMIK_API_KEY=
  LLM_PROVIDER=groq|gemini       # default groq; Gemini is a supported replacement
  GEMINI_API_KEY=                # required only when LLM_PROVIDER=gemini
  GEMINI_MODEL=gemini-3.5-flash-lite
  TEST_NUMBER=                   # only used after explicit paid-call confirmation

Do not require a VoiceLink credential to complete the VoBiz production path.

## Phase 1 — preflight

1. Confirm Git, Docker, curl, and Python 3 are available.
2. Clone or update `rapidx-voice-agent-stack` without overwriting user changes.
3. Read README.md, ONE-SHOT-PROMPT.md, docs/TROUBLESHOOTING.md, docs/RUMIK-OVERLAY.md, and docs/PRICING.md before changing anything.
4. Create `.env` from `.env.example`, fill it without echoing values, and confirm it is ignored by Git.
5. Test credentials with read-only calls where possible. Report only provider name and pass/fail.

## Phase 2A — LOCAL mode

Goal: a filmable local UI plus a local Dograh builder. Phone calls require a public tunnel and are a separate verification gate.

1. Start stock Dograh locally using its current official Docker Compose command. Open its documented local port and create the admin account.
2. Start the bundled Studio UI in a Node 20 container so Node does not have to be installed on the host:

   docker run -d --name rumik-voice-studio --restart unless-stopped \
     -p 8787:8787 --env-file .env \
     -v "$PWD/dashboard:/app" -w /app node:20-alpine node server.js

3. Open:
   - Studio/marketing: http://localhost:8787
   - Studio console: http://localhost:8787/app.html
   - Dograh builder: use the current official local URL shown by Dograh after startup.
4. Add the Rumik overlay exactly as documented in this repository. `pipecat-rumik` must be installed with `--no-deps`; a normal install can replace Dograh’s vendored Pipecat fork and break the API container.
5. If real phone calling is required from LOCAL mode, create a stable HTTPS/WSS tunnel to the Dograh backend. Put the public backend URL into Dograh and the carrier webhook/application. An ephemeral tunnel is acceptable for a demo only; re-register webhooks whenever it changes.

## Phase 2B — CLOUD mode

1. Provision a public Ubuntu 24.04 VM with at least 4 GB RAM. Google Cloud is acceptable; eligible new users currently receive a $300 welcome credit for 90 days. DigitalOcean is also acceptable.
2. Ensure ports 22, 80, and 443 are open. For browser calling also permit 3478, 5349, and UDP 49152–49200.
3. Run, in order:

   bash deploy/01-deploy-dograh.sh
   bash deploy/02-build-rumik-overlay.sh
   bash deploy/03-configure.sh
   bash deploy/04-check-interrupts.sh
   bash deploy/06-deploy-dashboard.sh

4. After each script, inspect the real response/logs. Do not continue when a container is unhealthy.
5. Confirm the HTTPS console renders and the API schema is at `/api/v1/openapi.json`.

## Phase 2C — rebuild the Studio product UI

This is a real product rebuild, not a reskin. Before writing components, create `dashboard/DESIGN.md` and make it the implementation contract for tokens, typography, spacing, components, interaction states, accessibility, motion, responsive behavior, and accepted design debt.

### Visual direction: premium cream

Replace the current dark/neon interface everywhere, including Overview, Agents, Voice Studio, Talk to it, Telephony, Settings, authentication, modals, charts, empty states, and loading states.

Use this starting token system and refine it in `DESIGN.md`:

  canvas:             #FBF7EF  warm cream
  surface:            #FFFDF8  elevated ivory
  surface-muted:      #F3EBDD  warm secondary panel
  ink:                #201A17  espresso
  ink-muted:          #70645B  warm gray
  border:             #DED2C2  sand
  accent:             #A8743B  restrained bronze
  accent-soft:        #EAD9BF  champagne
  success:            #416B57  muted forest
  warning:            #A56B2C  amber
  danger:             #9B4D45  clay red

Use a refined serif for display headings and a highly legible sans-serif for product text. The result must feel editorial, expensive, quiet, and professional: generous whitespace, disciplined 8px spacing, 14–18px radii, one-pixel warm borders, restrained soft shadows, tactile controls, and no rainbow gradients, neon glows, glassmorphism fog, giant empty panels, or black page backgrounds. Use Lucide or another coherent SVG icon set; never emoji icons.

The Telephony area must be cream/ivory and operationally credible. Give each provider a clean configuration card with connection status, assigned numbers, inbound route, outbound caller ID, last verification, and a test action. Secrets must always be masked. Include polished loading, connected, degraded, blocked, empty, validation-error, and permission-denied states.

### “Talk to it” is a true live conversation

The current record-stop-upload-transcribe-reply interaction is a bug. Delete that behavior. Do not ship a microphone button that records one blob and requires a second click or a Send button to complete a turn.

Build a continuous, bidirectional browser voice session:

1. The user selects an agent and presses one explicit `Start live conversation` control. This user gesture requests microphone permission and opens one persistent WebSocket or WebRTC session.
2. Capture microphone audio continuously with `getUserMedia` using echo cancellation, noise suppression, and auto gain control. Stream small audio frames through an AudioWorklet or equivalent low-latency path; do not wait for a completed recording.
3. Stream audio to Deepgram and render interim captions while the person is speaking. Final transcript segments append to the same session history but are not the mechanism that triggers the conversation.
4. Use server-side VAD/turn detection to finalize natural turns automatically. The user must never press Send after speaking.
5. Send the finalized turn plus conversation context to the selected brain: Groq Llama or Google Gemini. Preserve one server-owned conversation state for the entire session.
6. Stream Rumik TTS audio back as soon as playable chunks are available. Show a subtle live waveform/orb and explicit `Listening`, `Thinking`, and `Speaking` states.
7. Implement real barge-in. If the human starts speaking while the agent is speaking, immediately cancel queued/current TTS playback, cancel or ignore the obsolete generation, retain what was actually heard, and process the new human turn. Target playback stop within 250 ms and measure it.
8. Keep the session open across many turns until the user presses `End conversation`. Then stop tracks, close sockets, release audio resources, persist the session summary/transcript, and return to idle.
9. Add reconnect-with-backoff, permission-denied, missing-device, provider-timeout, expired-key, network-loss, and server-restart states. Never leave the mic active after end/error/navigation.
10. Typed messages may remain as an accessibility fallback, but the primary experience is hands-free live speech.

Model the UI with an explicit state machine:

  idle → requesting_permission → connecting → listening ↔ thinking ↔ speaking
  any active state → reconnecting | error | ending → ended

Acceptance tests:

  [ ] One Start action begins a multi-turn conversation; no per-turn record/send action exists
  [ ] Interim transcript appears while the human is still speaking
  [ ] Natural silence finalizes the turn automatically
  [ ] Agent audio begins incrementally instead of after a full-file download
  [ ] Speaking over the agent stops its audio and the next response addresses the interruption
  [ ] Five consecutive voice turns preserve context in one session
  [ ] Ending or navigating away releases the microphone and audio graph
  [ ] A visible diagnostic panel records connect time, first partial transcript, turn-finalization time, first response token/audio, and barge-in stop latency

### Overview analytics with Recharts

Convert the Studio frontend to a maintainable React build if it is not already React, then install and use `recharts`—do not imitate charts with CSS or static SVG screenshots.

The Overview must include:

  KPI cards: total calls, answered rate, successful outcomes, average duration, AI-runtime spend, active campaigns
  AreaChart: calls and answered calls over time
  BarChart: outcomes by agent or campaign
  LineChart: average latency and AI-runtime cost trend
  Funnel-style horizontal BarChart: uploaded → dialed → answered → qualified → converted
  date range, agent, campaign, provider, direction, and sub-account filters

Use `ResponsiveContainer`, accessible labels, keyboard-reachable tooltips, consistent number/currency formatting, and a data-table alternative. Charts must use the cream design tokens, subtle grid lines, restrained bronze/forest/terracotta series colors, and no default Recharts rainbow palette. Never fabricate activity: render a useful zero-data empty state and seed demo data only behind an explicit demo flag.

### Sub-accounts, multiple users, and tenant-safe data

Implement this as backend-enforced multi-tenancy, not frontend filtering:

  organizations/workspaces
  parent_account_id for optional parent → sub-account hierarchy
  users
  memberships with owner, admin, operator, analyst, viewer roles
  invitations with expiry, single use, and audit trail
  tenant-scoped agents, telephony configs, provider credentials, phone numbers, campaigns, contacts, calls, transcripts, recordings, analytics events, API keys, webhooks, and billing/usage rows

Requirements:

1. Every tenant-owned row has an immutable `organization_id`; child accounts have a verified `parent_account_id`.
2. Resolve the active organization from the authenticated server session and membership. Never trust an arbitrary organization ID supplied by the browser.
3. Apply authorization and tenant predicates in every query, mutation, export, websocket subscription, recording URL, analytics aggregation, and background job.
4. Encrypt provider credentials at rest, reveal only masked suffixes, and never inherit them into a child account without an explicit owner action.
5. Owners can create, rename, suspend, and switch sub-accounts; invite/remove users; set roles; set concurrency/usage limits; and view a clearly labelled parent-level rollup.
6. Admins manage the active workspace but cannot change ownership. Operators run agents/campaigns. Analysts see analytics/exports. Viewers are read-only.
7. Add an organization switcher showing the active workspace at all times. Switching clears tenant-specific caches, live subscriptions, pending forms, and selected records before loading the next workspace.
8. Add audit logs for login, invite, role, credential, telephony, agent publish, campaign start/pause, export, and destructive actions.
9. Protect exports and recordings with short-lived, tenant-checked download URLs.
10. Add rate limits and idempotency keys to invitations, campaign starts, outbound calls, and provider verification.

Isolation tests are release blockers:

  [ ] User A cannot read, guess, mutate, subscribe to, export, or download any User B tenant resource by changing an ID
  [ ] Parent rollups include only authorized children and never expose child secrets
  [ ] Removing a membership immediately invalidates access and active subscriptions
  [ ] Role downgrades take effect without requiring a fresh browser login
  [ ] Background campaign jobs retain and verify tenant context
  [ ] Analytics totals equal tenant-scoped source rows

### UI quality gate

Build production assets and verify Overview, Talk to it, Telephony, Agents, Settings, sign-in, sub-account switcher, and user management in a real browser at 375px, 768px, and 1280px. Exercise default, hover, focus, active, disabled, loading, empty, success, validation-error, provider-error, permission-denied, and reconnecting states. Meet WCAG 2.2 AA contrast and keyboard access, keep layout shift at zero, and fix runtime console errors before completion.

## Phase 3 — build the Rumik agent

Create and publish an agent named `Rumik ₹1 Demo Agent` with this behavior:

  Role: a warm Indian-English/Hinglish receptionist and lead qualifier.
  Opening: greet first, identify the business, ask how it can help.
  Style: short spoken sentences, natural contractions, no markdown, no long lists.
  Language: mirror the caller’s English/Hindi/Hinglish.
  Goal: understand intent, capture name and requirement, answer from supplied knowledge, offer a next step.
  Safety: never invent prices, availability, policies, or personal data.
  Human handoff: offer transfer/callback when requested or when uncertain.
  Opt-out: if the caller asks not to be called, confirm and end immediately.
  End: summarize the next step and close politely.

Use these nodes in the visual workflow:

  Start/Greeting → Discover intent → Answer or qualify → Capture outcome → Transfer/Callback or End

Set `allow_interrupt=true` on every speaking node except the final End node, then publish the workflow. Editing a draft without publishing does not affect live calls.

Use the production PSTN pipeline configuration. The chosen brain is replaceable without changing STT, TTS, telephony, prompts, tools, or workflow:

  STT: deepgram / nova-3-general / language multi
  LLM default: groq / llama-3.3-70b-versatile
  LLM replacement: google / gemini-3.5-flash-lite
  TTS: rumik / mulberry / a warm Indian-English voice
  Mode: BYOK pipeline

If `LLM_PROVIDER=gemini`, use the Gemini replacement and verify the model is available for that key. Do not use native-audio realtime mode for PSTN; the browser “Talk to it” feature is continuous streaming around the same pipeline, not a native-audio shortcut.

## Phase 4 — VoBiz primary telephony and inbound

1. In Dograh, create a dedicated VoBiz telephony configuration.
2. Leave `application_id` blank so Dograh creates a dedicated application with the correct `answer_url`. Never reuse an application owned by another voice product.
3. Add VOBIZ_NUMBER in E.164 form and make it the default outbound caller ID.
4. In the VoBiz console, attach that exact number to the Dograh-created application.
5. Assign `Rumik ₹1 Demo Agent` as the inbound workflow for the number in Dograh.
6. Verify the VoBiz application:
   - answer URL is `https://<DOGRAH_BACKEND>/api/v1/telephony/inbound/run`
   - method is POST
   - the number is attached to this same application
   - the Dograh backend is publicly reachable
7. Do not mark inbound working until a real external phone calls the DID, the agent answers, speaks first, hears the caller, responds correctly, and creates a Dograh run with transcript/recording metadata.
8. If the account contains several old applications, do not guess. Identify the healthy Dograh backend, bind the number to exactly one dedicated Dograh application, and document any stale applications for later cleanup.

## Phase 5 — VoiceLink secondary carrier

VoiceLink is optional and must not block VoBiz completion.

1. Authenticate and read wallet, DID, call-routing, and channel state.
2. Create or update a WebSocket bot pointing to the public agent bridge.
3. Route the chosen DID to that bot for inbound and outbound.
4. Respect VoiceLink’s declared codec from the start frame. The proven Indian mobile path used G.711 A-law at 8 kHz; do not hard-code µ-law.
5. For outbound numbers, send the national number plus `country_code:"91"` when required by the VoiceLink API.
6. Do not claim inbound is working if the provider has not enabled incoming service on the DID or if the account lacks sufficient channels. Report it as a carrier-side blocker with the exact provider action required.

## Phase 6 — campaigns and bulk calling

1. Create a CSV with the exact header:

   phone_number,customer_name,purpose,language,timezone,notes

2. Validate every row: E.164 phone number, no duplicates, no blank required fields, consent/legitimate-basis recorded, and opt-outs removed.
3. Upload the CSV using Dograh’s campaign source flow.
4. Create a draft campaign named `Rumik ₹1 Demo Campaign` tied to the published workflow and the VoBiz telephony configuration.
5. Start conservatively:
   - `max_concurrency=1` until one real call succeeds
   - then never exceed the lower of Dograh’s organization limit and the carrier’s purchased channels
   - retries: maximum 2, 120-second delay, busy/no-answer only
   - schedule in the contact’s lawful local calling window
   - circuit breaker enabled at a 50% failure rate over a meaningful sample
6. Preview the first three personalized prompts using `{{customer_name}}`, `{{purpose}}`, and fallback values.
7. Keep the campaign in draft until the operator explicitly approves the exact CSV and count.
8. After approval, start it and monitor progress, failures, spend, opt-outs, and call runs. Pause automatically on authentication failures, carrier 5xx responses, repeated silence, or circuit-breaker activation.

## Phase 7 — end-to-end verification

Required checks:

  [ ] Dograh containers healthy and UI reachable
  [ ] Rumik appears as TTS provider
  [ ] Real Rumik WAV returned, PCM mono 24 kHz for the Studio smoke test
  [ ] Brain returns a real response using a currently available model
  [ ] Published agent speaks first
  [ ] Barge-in stops speech and the agent responds to what was said
  [ ] “Talk to it” passes five-turn hands-free live-session, streaming-caption, resource-release, and barge-in tests
  [ ] Cream design system is applied to every route, including Telephony
  [ ] Recharts Overview renders real tenant-scoped data and a truthful empty state
  [ ] Sub-account/user role matrix and cross-tenant isolation tests pass
  [ ] VoBiz number is attached to the dedicated healthy Dograh application
  [ ] Real inbound call answered and run recorded
  [ ] One explicitly confirmed outbound test call succeeds
  [ ] Campaign remains draft until list approval; one-row canary succeeds before bulk
  [ ] VoiceLink status is reported independently as working or carrier-blocked
  [ ] No secrets appear in Git, logs, screenshots, or documentation

For every failed check, give: observed evidence, likely cause, exact next action, and whether the blocker is code, credentials, infrastructure, or carrier-side.

## Final report format

Return:

  Deployment mode and URLs
  Agent/workflow name and published ID
  Provider matrix: Rumik, STT, LLM, VoBiz, VoiceLink
  Inbound result with real-call evidence
  Outbound result with real-call evidence
  Campaign status, row count, concurrency, and approval state
  Live-conversation latency/barge-in measurements
  UI routes and responsive/state QA evidence
  Recharts data sources and empty-state result
  Organization, sub-account, user/role, and isolation-test results
  Tests passed and failed
  Costs incurred, if any
  Remaining carrier/account actions

Never write “done” for inbound, outbound, or campaigns without the corresponding real-world evidence.
```

## Current verified baseline on Shreyas’s Mac (4 Aug 2026)

- Local UI: running at `http://localhost:8787` and `http://localhost:8787/app.html`.
- Rumik: real WAV synthesis verified, PCM mono 24 kHz.
- Gemini: the prior local smoke test passed with the then-current `gemini-flash-latest` alias. This rebuilt prompt selects `gemini-3.5-flash-lite`; re-run the availability check for the viewer's key instead of treating the old test as proof.
- VoiceLink: authenticated status endpoint returns routing, wallet, DID, and engine fields.
- VoBiz: credentials work and four applications are present. The backend at `168-144-154-134.sslip.io` is healthy; two other Dograh application backends are offline. A real inbound call is still the required proof before inbound can be called complete.
