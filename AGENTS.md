# Limrun docs — agent handoff

A working document for any AI coding agent picking up this docs project. Catches you up to the state at the end of 2026-05-17.

## TL;DR

You are continuing a docs rewrite for **Limrun**, cloud infrastructure for mobile development. The client is **Muvaf** (founder). The repo is `maniculehq/lim-docs`. Current branch is `docs/16-may-rewrite`, PR #3. The first three pages in nav order (`introduction.mdx`, `quickstart.mdx`, `agents/cli.mdx`) have been rewritten and verified. Several pages remain.

Before you touch a single page, read **Standing rules** below. They are non-negotiable.

## Standing rules

These apply to every page audit and every edit. They were established after Muvaf rejected an earlier PR for being "heavily LLM-generated."

### Rule 1 — Every claim must be verifiable

Any factual assertion about how the system works, what a command does, what a flag defaults to, what's GA vs alpha, what a status field is named, what the server does internally, or what an audience can or can't do **must be traceable to an authoritative source** (see [Sources of truth](#sources-of-truth)).

If a sentence makes such an assertion and you can't point to the source that proves it, the sentence does not ship. Flag it, ask the client.

The canonical failure mode the previous draft hit: an invented sentence about *"the server decodes them into a temporary keychain"* that was simply not true. Confident, specific, plausible-but-unverified claims are the LLM tell that broke the client's trust. Don't repeat it.

### Rule 2 — Voice

- **No em dashes anywhere.** Not in docs, not in chat with the user. Use periods, commas, colons, or parentheses. Em dashes are an LLM-writing tell.
- **No fluff phrases.** `powerful`, `comprehensive`, `leverage`, `enables you to`, `By the end of this guide...`. If a sentence reads better without the word, cut the word.
- **Conversational but polished.** Like explaining to a smart colleague who doesn't have your context yet. Connect ideas; show why things work the way they do.
- **Outcomes first.** Structure each page around what the reader needs to achieve. Cut any section that doesn't serve that outcome.
- **No sections for the sake of sections.** Every heading must earn its place. Cut trivial "Advanced Usage" or "Best Practices" filler.
- **Concise.** Developers want concise, to the point. No long preambles, no narrative buildup. State what the thing is and move on.
- **No forward references.** Don't mention a concept before explaining it.
- **Code snippets must be introduced and explained.** Never dump a block and move on.

### Rule 3 — No customer / product name-drops

Don't anchor docs to specific company names (Replit, Conductor, Block) or coding-agent products (Claude Code, Cursor, Codex, Devin) where they're not load-bearing. Use category names: "coding agents," "platform integrators," "developer tools." Exception: when the name is genuinely the reference (a `--agents` CLI flag has enum values `claude|cursor|codex` — those stay; UI screenshots that show real button labels stay).

### Rule 4 — CLI and SDK are different personas, never mix

The `lim` CLI is for cloud coding agents and CI. The SDK (`@limrun/api`, plus Python/Go/Java) is for platform integrators embedding live simulators. The previous PR drew Muvaf's "it suddenly jumps from CLI to SDK code??" feedback for putting both on the same page. Keep them separated by page; cross-link between them; never use a CodeGroup that mixes them.

### Rule 5 — Cloud agents are headless

A cloud agent on a Linux VM cannot open a browser. Anywhere the audience is "an agent," **lead with `LIM_API_KEY` env var** and do not mention `lim login`. `lim login` is fine on the human quickstart page.

### Rule 6 — Don't pitch defaults as features

Things like "build logs stream to stdout" are the default behavior of any CLI build command, not a notable feature. Don't write sentences that pitch them. Describe them only when describing what a command does.

### Rule 7 — Don't surface `lim xcode sync` as a standalone step

`lim xcode build` already syncs before it builds. Don't tell readers to run `lim xcode sync` as a separate step. (The `lim xcode sync` command exists for watch-mode use cases; if you reference it, scope it to that.)

### Rule 8 — Unverified claims protocol

Inline flag in the page **and** a one-liner in `QUESTIONS-FOR-CLIENT.md` at this repo root (if it doesn't exist, create it). Nothing ships unflagged. The plan file at `~/.claude/plans/well-the-whole-point-recursive-aho.md` has the running list of open questions.

## Review process

Reviewing a docs page is two passes: **technical verification first, writing review second**. The order matters. Editing prose on unverified facts polishes wrong claims into more confident wrong claims, which is the failure mode that broke Muvaf's trust the first time. Verify before you stylize.

### Pass 1 — Technical verification

Goal: every factual claim on the page traces to source, transcript, or hands-on observation.

1. **Pull the source files you'll need.** For most pages, the relevant ones live in `/tmp/lim-typescript-sdk`:
   - `src/resources/{ios,android,xcode}-instances.ts` — spec / status shape, model and region enums
   - `packages/cli/src/commands/**/*.ts` — every CLI command and flag
   - `src/ios-client.ts`, `src/resources/xcode-instances-helpers.ts` — SDK method shapes
   - `src/folder-sync.ts` + `src/folder-sync-ignore.ts` — sync rules
   - `src/exec-client.ts` — build streaming over SSE
   - `packages/cli/src/lib/{auth,config}.ts` — login callback, `.env` auto-load

2. **Walk the page paragraph by paragraph.** For every sentence that asserts something the system does (a default, a flag name, a sync rule, a status field, a region, an error format, an auto-detected behavior), do one of:
   - **Cite** the source file + line that proves it.
   - **Verify** hands-on per [How to verify a CLI claim hands-on](#how-to-verify-a-cli-claim-hands-on).
   - **Flag** it as unverified and either remove the sentence or surface the question under [Open questions](#open-questions-for-the-client).

3. **Specific things to double-check:**
   - **Defaults.** Does the page state a default value? Find where it's set in source. SDK type docstrings sometimes contradict the CLI's `--help` output; trust empirical observation over both. (See product bug #3: the inactivity-timeout default was wrong precisely because of this.)
   - **Flag enums.** `--region us-west`, `--model iphone`, `--sdk iphoneos`. Cross-check the `options:` array in the CLI's flag definition.
   - **Status field names.** `apiUrl`, `endpointWebSocketUrl`, `mcpUrl`, `signedStreamUrl`, `targetHttpPortUrlPrefix`. Check the resource's `Status` interface.
   - **SDK method names.** `client.elementTree`, `client.performActions`, `xcode.xcodebuild`, etc. Check the `InstanceClient` and `XcodeClient` type declarations.
   - **Internal-behavior claims.** Anything starting with "the server …" or "Limrun does X internally" must be traceable to Muvaf's transcript, a Slack answer, or a verifiable source file. Otherwise cut or flag.
   - **Performance / latency claims.** "Fast", "instant", "under a second" all need a real measurement or they don't ship.

4. **Cross-page links.** Run a grep for internal links and confirm targets exist:
   ```bash
   grep -oE '\(/[^)]+\)' page.mdx | sort -u
   ```

5. **Output of Pass 1:** a citation per claim, a flag list for anything unverified, and a short list of questions for Muvaf if any.

### Pass 2 — Writing review

Run after verification is clean. Now you're polishing prose on facts you trust.

**Structure**
- Page opens with what the reader is here to achieve. Frontmatter description handles the meta pitch; the body jumps into action. Cut any sentence that's about the page itself ("this page covers …") rather than about the topic.
- Each H2 / H3 earns its place. If a section is three lines of code with no narrative, fold it into a neighbor or drop the heading entirely.
- Concepts are introduced before they're referenced. No forward references unless they're explicit links to a later section.
- The canonical agent loop (provision → build → drive → ship → cleanup) is surfaced early on agent-facing pages.
- Reader's understanding builds naturally as they read. Avoid "basic → intermediate → advanced" sectioning that breaks the flow; let later sections add depth to earlier concepts.

**Voice and tone**
- **No em dashes anywhere.** Periods, commas, colons, or parentheses. Search: `grep '—\|–' page.mdx`.
- **No fluff phrases.** Search: `grep -niE 'powerful|comprehensive|leverage|enables you to|allows you to|seamless|robust|effortless|simply|easily|by the end of this guide' page.mdx`.
- **No vague performance words** without a number: `fast`, `instant`, `quick`, `slow`, `optimized`, `lightweight`. Replace with the measurement or cut the sentence.
- **Active voice, specific subjects.** "The CLI syncs the source" beats "The source is synced."
- **Smart-colleague tone.** Conversational but polished. Connect ideas; show *why* the system works the way it does. Avoid both textbook-stiff and Tweet-thread-choppy.
- **No customer / product name-drops** where they aren't load-bearing. See [Rule 3](#rule-3--no-customer--product-name-drops).

**Code snippets — never dump, always frame**
- Every block has a one-sentence introduction explaining the problem it solves.
- After non-trivial snippets, a short follow-up explains the key parts (which params matter, what the return value carries, what comes next).
- Large examples are broken into smaller chunks, each chunk explained.
- CLI and SDK are **never mixed inside one `<CodeGroup>`**. The pattern is: SDK languages in one `<CodeGroup>`, then "Or from the CLI:", then the CLI block. See [Rule 4](#rule-4--cli-and-sdk-are-different-personas-never-mix).
- Examples are runnable as-is modulo `<placeholder>` substitution. No truncated `// …` blocks that hide the meaningful part.

**Tables earn their place**
- A table encodes a real shape — field × meaning, language × support, status field × use. If it's a bulleted list in disguise, use a bulleted list.
- Headers are short and concrete. Cells are one short phrase, not multi-sentence prose that breaks the visual rhythm.
- SDK-coverage tables list only the SDKs we currently ship (TypeScript / Python / Go). Java is out of scope per Muvaf 2026-05-17.

**Internal-detail discipline**
- Don't explain the server's internals unless Muvaf has described them. The canonical failure mode was an invented *"the server decodes them into a temporary keychain"* claim that was simply false.
- Observable behavior is fair game (a flag default, a sync rule, a status field). Unobservable internals (server-side caching, compression algorithms, internal data structures) need a source or they don't ship.

**Anti-patterns to specifically grep for**
- Shallow "Advanced Usage" / "Best Practices" sections with trivial content. Cut or merge into the main flow.
- Sections that exist for symmetry rather than for an outcome.
- "This page covers" / "In this guide" / "By the end of this…". Meta-narration belongs in frontmatter, not in the body lede.
- Tables longer than 10 rows that try to be reference docs inside a tutorial.
- `<CodeGroup>` blocks with only one language inside (just use the bare code fence).
- A `description` in the frontmatter that duplicates the body's first paragraph. On Mintlify it's hidden; on the lim-run-docs framework it renders as a subtitle, so duplicates show up twice.

### Final checklist

Before committing:

- [ ] Every factual claim traces to source, transcript, or hands-on observation.
- [ ] Every unverified claim is removed or flagged in [Open questions](#open-questions-for-the-client).
- [ ] No em dashes (`grep '—\|–' page.mdx` returns nothing).
- [ ] No fluff phrases (`grep -niE 'powerful|comprehensive|leverage|enables you to|allows you to|seamless|robust|easily' page.mdx` returns nothing).
- [ ] No vague performance words without measurements.
- [ ] Every code block has an introduction and a follow-up where it matters.
- [ ] CLI and SDK live in separate blocks. No mixed CodeGroups.
- [ ] Page lede states the reader's outcome, not the page's scope.
- [ ] Every H2 / H3 earns its place.
- [ ] No forward references unless they're links.
- [ ] Tables encode real shape, not filler.
- [ ] Cross-page links resolve. Verify with `grep -oE '\(/[^)]+\)' page.mdx`.
- [ ] Frontmatter title + description match the page's actual content.
- [ ] `mint dev` hot-reloads without errors.

## Product context (from May 8 call with Muvaf)

### What Limrun is

Cloud infrastructure for mobile development. Runs the parts of mobile development that need a Mac (or specialized hardware) as cloud services so any process — especially cloud coding agents on Linux VMs — can do iOS and Android work without local hardware.

Muvaf's definition verbatim: *"What we do is we enable mobile developers to use cloud agents."*

### Three products + Asset Storage

| Product | What it is |
|---|---|
| **iOS simulator** | Real Mac running the iOS Simulator. Streamable in browser via signed URL (token embedded). Custom device-control API surface (taps, screenshots, video, app lifecycle, element-tree queries). |
| **Xcode build sandbox** | Real Mac running `xcodebuild`. Delta-syncs local source, builds remotely, streams logs back over SSE. |
| **Android emulator** | Booted emulator with an ADB tunnel. Local Android Studio / Appium / scrcpy attach as if it were a USB device. |

**Asset Storage** is the supporting binary store underneath (`.ipa`, `.apk`, `.app.tar.gz`). Upload once, pre-install on new instances, or reference by name in shareable preview URLs.

The iOS simulator and Xcode sandbox compose into a paired instance (`spec.sandbox.xcode.enabled` on iOS create, or `--xcode` on the CLI). A successful build auto-installs and auto-launches on the paired sim. That composition is the canonical agent loop.

### Audience priority

1. **Cloud coding agents and the developers using them.** Headless. Primary launch target. Use the CLI with `LIM_API_KEY`.
2. **Platform integrators.** Companies embedding live simulators in their own product UI. Use the SDK from a backend + `<RemoteControl />` from `@limrun/ui` in their frontend. Slower deals, hands-on integration support, less docs-driven.
3. **Humans evaluating the product.** Small share of readers. Need the intro and architecture.

**Explicitly not for:** a human at a Mac writing code by hand. Muvaf's exact quote: *"doesn't make sense to use this if you have your Mac locally and yourself writing the code."*

### Muvaf's three goals for these docs

1. **Discovery by cloud agents.** When a coding agent on a Linux VM hits "I can't do iOS from here," it finds Limrun's docs and surfaces the option to the user → new customer.
2. **A skill-formatted page** an agent can read once and operate Limrun without mistakes.
3. **Humans understanding value + architecture.** Tertiary.

### Voice direction Muvaf gave

Cookbook style; he cited Vercel docs as the reference. *"Only really two end-to-end flows"* (individual dev with cloud agent, platform integrator embedding). Everything else is capability breakdown. **Agent-readable beats elegant.**

### Other voice references the user shared

- `https://supermemory.ai/docs/intro` — lead with `[Product] is [category]`, capability-focused mechanics, no customer names.
- `https://www.greptile.com/docs/introduction` — same pattern; "Why teams choose" + "How it works" + "Technical details."

## Sources of truth

1. **Code** at `github.com/limrun-inc`. 21 public, non-archived repos. The TypeScript SDK and the CLI both live in `typescript-sdk`. A local clone is at `/tmp/lim-typescript-sdk` from prior verification work. The most relevant files:
   - `typescript-sdk/src/resources/ios-instances.ts` — instance status fields, model enum
   - `typescript-sdk/src/resources/android-instances.ts` — Android variants including `playwrightAndroid`
   - `typescript-sdk/src/resources/xcode-instances.ts` — Xcode sandbox resource
   - `typescript-sdk/src/folder-sync.ts` + `folder-sync-ignore.ts` — sync logic
   - `typescript-sdk/src/exec-client.ts` — confirms `xcodebuild` streaming is over SSE (`eventsource-client`)
   - `typescript-sdk/src/client.ts` — default `baseURL`, env-var resolution
   - `typescript-sdk/packages/cli/src/lib/auth.ts` — confirms callback port 32412 and `/authn/callback` path
   - `typescript-sdk/packages/cli/src/lib/config.ts` — confirms `.env` auto-load via `dotenv`
   - `typescript-sdk/packages/cli/src/commands/**/*.ts` — every CLI command, every flag
   - `typescript-sdk/packages/cli/skills/limrun-ios/SKILL.md` — the canonical skill (not currently audited)

   Other repos worth knowing:
   - `lim` — older standalone CLI repo, last updated 2026-04-24. Probably legacy; current source is in `typescript-sdk/packages/cli/`. Flag for client if you find docs linking to the wrong one.
   - `homebrew-tap` — stale (2026-04-09). Per Muvaf's feedback, do not mention Homebrew anywhere in the docs.
   - `skills` — separate repo, no description, last updated 2026-04-13. Probably the staging ground for `npx skills` distribution that Muvaf said is coming. Don't reference it until he confirms it's live.
   - `ios-preview-action` — packaged GitHub Action for PR previews. Relevant for `ios/pr-previews.mdx`.

2. **Hands-on.** Ask the user for a test `LIM_API_KEY`. Use it via env var, not via `lim login` (the browser flow currently doesn't redirect back; see [Real product bugs](#real-product-bugs-discovered-not-doc-issues)). When you create a verification instance, always pass `--reuse-if-exists` and a stable label set so reruns don't multiply instances, and always clean up with `lim ios delete`.

3. **The May 8 transcript with Muvaf.** The user has the full transcript and pasted relevant chunks during this session. Key quotes are embedded throughout this doc.

**Source order on conflicts:** running CLI/SDK behavior > source code > transcript > README > prior drafts. Drafts are not a source.

## Pages: status

### Done (committed at HEAD of `docs/16-may-rewrite`)

| Page | Status | Notes |
|---|---|---|
| `introduction.mdx` | ✅ Rewritten | Lead is "Limrun is cloud infrastructure for mobile development." Three products + Asset Storage framing. Two main flows ("How it works"). Two access shapes ("Common use cases"). Architecture section with the instance definition + signed-stream URL. No customer names. The previous draft is preserved at `introduction-old.mdx` for side-by-side comparison; it's also wired into `docs.json` under Getting Started. |
| `quickstart.mdx` | ✅ Rewritten + hands-on verified | Lede one sentence. Top callout offers the browser Playground as an alternative path. Prerequisites + 3-step Steps block. Drive-the-app section with verified commands. Tear-down section. One CardGroup. Real timing claims (no "under a second" — verified install is ~5s). |
| `agents/cli.mdx` | ✅ Rewritten + hands-on verified | Authenticate is `LIM_API_KEY` only (no `lim login`). Verified every flag in the create / build / perform / record / launch-app / app-log / list / session / skills-install tables against `--help` output. Fixed three wrong claims that were in the previous version: latency numbers, sync default exclusions, inactivity-timeout default. Added the success-state screenshot. The Gotchas section is incomplete; see [Expert-level gotchas not yet covered](#expert-level-gotchas-not-yet-covered-on-agentscli). |

### Not yet audited

- `agents/mcp.mdx`
- `ios/build-with-xcode.mdx`
- `ios/run-simulator.mdx`
- `ios/pr-previews.mdx`
- `ios/in-app-purchases.mdx`
- `android/run-emulator.mdx`
- `testing/appium.mdx`
- `testing/playwright.mdx`
- `platform/embed-simulator.mdx`
- `platform/asset-storage.mdx`
- `reference.mdx`

Don't start one until you've internalized the standing rules.

### About Ritik's "readability changes"

A teammate (Ritik Sahni) pushed a `readability changes` commit (`7531ee3`) while the introduction rewrite was in flight. It touched many files: `agents/cli.mdx`, `agents/mcp.mdx`, `android/run-emulator.mdx`, `introduction.mdx`, `ios/build-with-xcode.mdx`, `platform/embed-simulator.mdx`, `quickstart.mdx`, `reference.mdx`. During the introduction rebase, the user dropped Ritik's "How it fits together" section addition to introduction.mdx because the new Architecture section already covered the material. Ritik's edits to the other pages are still in HEAD. When you audit a not-yet-audited page, expect to find Ritik's changes folded in; treat them as part of the starting state, not gospel.

## Open questions for the client

Pending answers from Muvaf. The user is tracking these and asking him separately.

### Resolved by Muvaf on 2026-05-17

- **(1) Streaming protocol.** WebRTC with a hardware encoder. Page text doesn't currently need to surface this; the status field names (`endpointWebSocketUrl` etc.) are about the control plane, which is genuinely WebSocket.
- **(2) iPad / Watch status.** "We can show iPad and Watch." Dropped the alpha caveat and the "use iphone for production" warning on ios/run-simulator.mdx.
- **(3) Asset Storage formats.** APK for Android; iOS apps must be simulator builds (a `.app` folder) zipped or tarballed. Signed device IPAs still upload but are downloadable, not installable on the cloud simulator. Updated introduction.mdx, platform/asset-storage.mdx, and the `lim ios install-app` examples on agents/cli.mdx + ios/run-simulator.mdx (was `./MyApp.ipa`, now `./MyApp.app.zip`).
- **(5) Detox.** "We have a new detox example, you can leave the skill for now." Removed Detox name-drops from android/run-emulator.mdx and reference.mdx; no docs page references the unreleased `limrun-detox` skill.
- **(7) Java SDK.** "Java is not needed." Dropped Java code blocks and Java mentions across reference.mdx, platform/asset-storage.mdx, platform/embed-simulator.mdx, agents/mcp.mdx, ios/build-with-xcode.mdx, ios/run-simulator.mdx, ios/in-app-purchases.mdx, android/run-emulator.mdx, testing/playwright.mdx.
- **(9) CocoaPods/SPM.** "We automatically detect and resolve it." Added a one-sentence note to ios/build-with-xcode.mdx's "What gets synced" section: the remote sandbox detects `Podfile` / `Package.swift` / `Cartfile` and resolves deps remotely, which is why `Pods/` and `.swiftpm/` don't ship.

### Still open

- **(4) `lim_` API-key prefix:** contractual or descriptive? Low-priority.
- **(6) `lim` repo vs `typescript-sdk`:** is the older `lim` repo legacy?
- **(8) API-keys screenshot** sanitization for `images/console/13-api-keys.png`.
- **(10) iOS system permission dialogs:** does the sim image pre-accept location / notifications / photos / camera, or does the agent always need to dismiss them?
- **(11) `--region eu-north1`:** is it a real region? Page now uses `us-west` everywhere.
- **(12) Example asset name** like `appstore/Expo-Go-54.0.6.tar.gz` — is there a real public asset name worth surfacing? Pages now use generic placeholders.

## Real product bugs discovered (not doc issues)

These are surfaced during verification; flag them with the team separately. Don't paper over them with doc workarounds.

1. **`lim login` redirect bug.** The CLI implements its half correctly (listens on `localhost:32412/authn/callback`, opens `console.limrun.com/authn/login?user-agent=lim/<version>`, writes config on callback). The console side does not redirect back after a successful login. User logs in, lands on dashboard, callback never fires, CLI hangs. Worked once during this session, fails reliably otherwise. The fix needs to happen on `console.limrun.com/authn/login` — when the request has the `user-agent=lim/<version>` query param, the response must redirect to `http://localhost:32412/authn/callback?api-key=<key>` regardless of whether the user already had a session.

2. **Console Connect modal shows a fake CLI command.** The modal at the cable icon shows `lim connect ios <id>` as the suggested CLI command. That command does not exist (`lim connect`, `lim ios connect` both return "Command not found"). The actual equivalent is `lim session start --id <id>`. The modal needs to either change the displayed command or the team needs to add `lim connect` as an alias.

3. **Inactivity-timeout doc was wrong.** Previous docs claimed default 3m; CLI help says "default comes from your org settings." Fixed in agents/cli.mdx.

4. **`lim ios syslog` / `client.streamSyslog()` returns "Cannot run while sandboxed".** Known issue per Muvaf (2026-05-17), team is working on a fix. Per his guidance, dropped both the CLI command and the SDK method from the docs for now (ios/run-simulator.mdx section retitled "Read app logs", reference.mdx capability matrix row, agents/cli.mdx CLI snippet). Per-app logs via `appLogTail` / `streamAppLog` / `lim ios app-log <bundleId>` are the documented path. Reintroduce once Muvaf confirms it's fixed image-side.

## Expert-level gotchas not yet covered on `agents/cli`

These are gaps the current docs don't address. They're real, agent-hitting failures, but most need the client's input before documenting. The user is going to ask Muvaf the consolidated question list before we write workarounds.

- **CocoaPods/SPM** (covered above as open question #9).
- **App crashed on launch.** Build succeeds, install succeeds, auto-launch happens, but app crashes in didFinishLaunchingWithOptions. Sim ends up on Springboard. Agent thinks all is well. The right pattern: after `lim xcode build`, always run `lim ios element-tree` to confirm the app UI is present, not Springboard.
- **iOS system permission dialogs** (covered above as open question #10).
- **`npx @limrun/cli` works without global install.** The CLI's own README documents this. Useful for agents without sudo. Worth adding to Install section.
- **Stable labels for `--reuse-if-exists`.** If an agent uses ephemeral identifiers (timestamps, UUIDs) as labels, reuse silently fails and instances orphan every run. Worth explicit warning.
- **Scheme/workspace discovery.** When the agent first hits an iOS project it hasn't seen before, it doesn't know the scheme name. Discovery: `xcodebuild -list -workspace MyApp.xcworkspace` or grep the project for `.xcworkspace` / `.xcodeproj`.
- **Stale `~/.lim/last-instances.json`.** If a previous CLI run crashed, the cached last-instance ID may point to a terminated instance. Cleaner pattern: `lim ios list` first.
- **Project root vs cwd.** `lim xcode build .` syncs the current working directory. If the agent is in a repo root with a nested iOS project (`apps/ios/`), the sync misses or includes the wrong thing.

The current `agents/cli.mdx` Gotchas section has only 4 bullets. Once Muvaf answers the question list, the gotchas section should be expanded with the items above.

## Useful artifacts on disk

- **Plan file:** `~/.claude/plans/well-the-whole-point-recursive-aho.md` — the running plan, including the page 1 verification report and the protocol decisions.
- **Memory:** `~/.claude/projects/-Users-namanbansal-lim-docs/memory/` — durable feedback memories (the verification rule, the voice/no-em-dash rule). Loaded into context by the Claude Code memory system at the start of each conversation.
- **TypeScript SDK clone:** `/tmp/lim-typescript-sdk/` — for source-level verification.
- **Old intro draft:** `introduction-old.mdx` at the repo root, with a sidebar entry "Introduction (old draft)" for side-by-side comparison. Don't delete until the user explicitly says we're done with the comparison.
- **Verified post-build screenshot:** `images/console/06-ios-app-launched.png` — captured during a real verification build of `sample-native-app`. Reuse it where you need a success-state image.

## How to verify a CLI claim hands-on

1. Ask the user for a test `LIM_API_KEY`.
2. `export LIM_API_KEY=lim_...` (don't pass it as `--api-key` if you can avoid it; env var is less likely to show up in process listings).
3. `export PATH="/opt/homebrew/opt/node@20/bin:$PATH"` — the user's system Node is 25, which the CLI does not support. Node 20 LTS is at the homebrew path.
4. Provision a verification instance with a stable label: `lim ios create --xcode --reuse-if-exists --label test=verify-<page-name> --inactivity-timeout 5m`. Short inactivity timeout means the instance auto-terminates if you forget cleanup.
5. Run the commands you're verifying. Most help output is in `lim <command> --help`.
6. Clean up: `lim ios delete` (no args; deletes the last created).
7. For batched help inspection, combine multiple `--help` calls in one bash command and pipe through `head` to keep output bounded.

## Local rendering

`mint dev` runs in the docs repo. The user's system Node is too new; the CLI runs on the homebrew `node@20`. The command we've been using:

```bash
export PATH="/opt/homebrew/opt/node@20/bin:$PATH" && mint dev
```

Default port is 3000; if taken it auto-bumps to 3001, 3002, etc. The user's last running instance was on port **3002**.

After file edits, `mint dev` hot-reloads. If something seems stuck, kill the existing process (`TaskStop` if you started it via a Bash background task, or `lsof -ti :3002 | xargs kill -9` otherwise) and re-launch.

## Common pitfalls to avoid

- **Don't run `lim login` from a Bash subshell without backgrounding it.** It blocks waiting for the callback, which won't come (see bug #1 above). If you must test login, use `--api-key <value>` to bypass the browser flow entirely: `lim login --api-key lim_...` writes the key to `~/.lim/config.yaml` without opening a browser.
- **Don't commit any `lim_...` value to git.** API keys appear in chat during verification work but must stay out of the repo.
- **Don't add `QUESTIONS-FOR-CLIENT.md` unsolicited.** The user has been handling client questions out-of-band (Slack with Muvaf). If you accumulate flags, just add them to the open-questions section in the plan file or this file.
- **Don't rewrite a page from scratch.** Apply targeted edits using `Edit`. Save `Write` (full file replacement) for cases where the structural change is so large that line-by-line edits would be harder to review.
- **Don't trust the previous draft as a source.** The whole reason we're rewriting is the previous draft was LLM-generated and unverified. When a fact in the existing page is convenient, that's the moment to suspect it.

## How the user prefers to work

- **Push back when you think the user is wrong.** They've explicitly invited this. They prefer evidence-based disagreement over compliance.
- **Be specific about what you're changing and why.** Reference line numbers and source files.
- **Don't ask for plan approval mid-flow.** They prefer "here's the change, push back if wrong" over "should I do X."
- **When verifying something, do it.** Don't speculate when source can answer.
- **Keep chat messages tight.** Long preambles in chat are as bad as long preambles in docs.
- **Don't use em dashes anywhere, including in chat.**

---

If you got this far, you're ready. Read the standing rules once more, then pick a not-yet-audited page from the list and apply the same shape: verification → mechanical fixes → voice pass → ask the user about anything ambiguous → commit + push.

Good luck.
