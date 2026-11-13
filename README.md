# SaaSassin Performance

**Free, open-source Lattice alternative.** Run fair, reversible performance reviews with evidence you can keep.

Performance management for organizations with 25–250 employees.

- **Try it:** [performance.saasassin.net](https://performance.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/lattice/research](https://saasassin.net/targets/lattice/research)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-performance&project-name=saasassin-performance&repository-name=saasassin-performance&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-performance%231-create-a-firebase-project)

Lattice is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Performance is an independent, clean-room product and is not affiliated
with or endorsed by Lattice.

## What you can do today

| Stage | Capability | Who |
| --- | --- | --- |
| 1 | Launching a cycle generates **self, manager, upward, and peer** reviews. Peers come from approved nominations, or from teammates (max 3) when nomination is off | Review admins |
| 1 | **Peer nomination**: employees nominate up to 5 peers (pending). Managers and admins nominate with auto-approval and approve or decline. Writing cannot open while any nomination is pending | Employees, managers |
| 1 | **Deadlines**: edit the cycle deadline. Per-reviewer status shows on track, due soon, or overdue | Review admins |
| 1 | **Reminders + escalation**: one reminder goes to each outstanding reviewer, and overdue reviewers are escalated to their manager. These land in a new in-app **Inbox** on Overview | Review admins → everyone |
| 2 | **Calibration**: each employee gets a review signal (averages by direction from rating questions), a calibrated 1–5 rating, and a private note | `reviews.calibrate` |
| 2 | **Review packets** are assembled on release. Self and manager reviews are attributed. Peer and upward reviews are anonymized ("Peer 1", "Direct report 1"). Calibration notes are never included | Automatic |
| 2 | **Acknowledgement** with an optional comment. **Safe reopen** of one packet requires a reason, and the prior acknowledgement is kept in packet history. Reopening the whole phase withdraws packets without losing reviews | Employee, calibrators |
| 3 | **Goals**: you or your manager create a goal (visible to everyone, or to owner and manager only). Owner or manager can check in with progress, status, and a note | Everyone |
| 3 | **Feedback**: praise or constructive, visible to the recipient only, recipient and manager, or everyone. The text never enters the audit log | Everyone |
| 3 | **One-to-ones** between an employee and their manager: shared agenda (add or tick), shared notes, complete | Participants only |
| 3 | **Employee timeline**: newest first, covering goals, check-ins, feedback, submitted reviews, packet release, acknowledgement, and 1:1s. It only shows what the viewer may see | Everyone |
| 4 | **Reports** filtered by cycle and team: completion by direction and team, packet acknowledgement %, goal health, feedback counts, calibrated-rating distribution | `reports.read` |
| 4 | **Complete workspace export**: one JSON download using the `net.saasassin.workspace-export` 1.0.0 manifest. It has 14 product files, each with its own schema id, version, record count, and sha256, plus migration notes. The export is audited | `workspace.export` |
| R2 | **Imported employees can sign in and act as themselves.** On a verified-email sign-in (or the next page load), each pending invite for that email is claimed. The directory record gets `authUid`, and `memberLinks/{uid}` maps the uid to the imported member id. Reviews, goals and packets keep their existing ids | Everyone |
| R2 | **Anonymity thresholds**: each cycle sets a minimum number of peer and upward responses (default 3). Below the minimum, those responses are withheld entirely, and the packet explains how many came in and how many are needed | Review admins |
| R2 | **Scheduled cycles**: set a launch date and end dates for nomination, writing (the deadline), calibration and acknowledgement. Overdue phases are detected each time the page loads ("Phase overdue Nd"). **Advance due phases** moves every due cycle one phase through the normal previewed, audited transition. Blocked cycles are reported, never forced | Review admins |
| R2 | **Saved report views**: save the current cycle and team filters, either private or shared with report viewers. Owners and admins can delete a view | `reports.read` |
| R2 | **API keys**: scoped (`directory:read`, `cycles:read`, `goals:read`), stored only as SHA-256 hashes, shown once, revocable, and audited, with a last-used time | `organization.manage` |
| R2 | **Read-only REST API**: `GET /api/lattice/v1/organizations/{org}/members\|cycles\|goals` with `Authorization: Bearer sk_perf.…`. Cycles return counts only, goals return public goals only, and review content is never included | API key |
| R2 | **Signed webhooks** for `review_cycle.transitioned`, `review_packet.acknowledged`, and a test event. Each request carries HMAC-SHA256 in `X-SaaSassin-Signature: t=…,v1=…`, computed over `t.body`. Only public HTTPS endpoints are allowed (SSRF guard), with a 5s timeout. Every attempt is logged with HTTP status, error and duration, and failures are shown as failures | `organization.manage` |

New UI tabs: **Growth**, **Timeline**, **Reports** (plus the **API and webhooks** panel for workspace managers). Reviews tab gains the
nominations, deadlines, calibration and packets panel; Overview gains Inbox and
My review packets. All controls are shadcn/ui (Select, Dialog, Checkbox, Table).
There is no native select, `confirm()`, or `alert()`.

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Performance is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Performance signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` limits browser reads to organization members with the right role and permission, and server-owned records (nominations, calibrations, packets, goals, feedback, one-to-ones) can only be written by the server. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Performance at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=performance`,
and `src/proxy.ts` maps `/` to the product (`/lattice`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-performance&project-name=saasassin-performance&repository-name=saasassin-performance&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-performance%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Lattice, running Performance for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/lattice#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
