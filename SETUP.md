# RAISE HELL console: setup

The deployed console is a **single self-contained `index.html`** served by GitHub Pages. Styles, JavaScript, the MP4 muxer, and fonts are bundled into that file to keep the repository and deployment simple. The original modular files remain in the source ZIP for future editing. It connects three things you already have:

- **n8n**: the existing `Raise Hell - Generation` workflow writes with Claude. The console only calls its webhook (`/webhook/raise-hell/generate`).
- **Zernio**: the Instagram account, the music catalog, uploads, scheduling, publishing and analytics.
- **GitHub**: the doctrine, and `standards/gold-standards.md`, which the console adds published examples to when you ask it to.

There is no server, database or build step. Drafts and settings stay in your browser. Scheduled and published posts live in Zernio.

## 1. Publish the console

In the repository, open Settings, then Pages. Deploy from the `main` branch, root folder. The console is then at `https://jolthevc.github.io/raisehell/`.

Use current Chrome or Edge on a computer, or Safari 16.4 or later on a Mac or iPhone. The Reel video is made in the browser with its built-in H.264 encoder (WebCodecs). Firefox works only where it offers an H.264 encoder; if it does not, the console says so and nothing is uploaded.

Every Reel file is checked before upload: the console reads the MP4 back and compares it with Instagram's Reels specification (H.264, 4:2:0, moov atom at the front, no edit list, 23 to 60 fps, 3 seconds to 15 minutes, AAC if there is audio), then plays it back in the page. A file that fails the check is never uploaded. Preview shows the result on its "Checked:" line.

## 2. Connect Instagram to Zernio with Facebook Login

Catalog music only works when Instagram is connected **with Facebook Login**.

1. Make the Instagram account a Business or Creator account.
2. Link it to a Facebook Page you manage. In Instagram, open Accounts Center and connect the Page.
3. In Zernio, connect Instagram and choose the **Facebook** option. Approve the permissions.
4. In Zernio, open API keys and create a key (it starts with `sk_`).

## 3. Settings, Connections

### n8n (the existing Generation workflow)

Paste the production webhook URL of `Raise Hell - Generation`: `https://<your n8n host>/webhook/raise-hell/generate`. Not the `/webhook-test/` address. If the webhook uses Header Auth, add its key. In the workflow's Webhook node, set **Allowed Origins (CORS)** to `https://jolthevc.github.io` so the browser may call it.

Press **Test with one post**. This asks the workflow for one real post (one Claude call) and confirms that the request and response shapes match. The post is not added to the desk.

The console sends and expects exactly this. No new workflow is needed; if yours differs, the Test says what came back.

| Request (JSON POST) | Response the console reads |
|---|---|
| `{"action":"generate","count":10,"theme":"","direction":"","avoid_topics":[]}` | `{"posts":[{"text":"...", "territory":"", "emotion":"", "register":"", "archetype":"", "editorial_note":""}], "batch_note":""}` |
| `{"action":"revise","text":"...","direction":"..."}` | `{"revisions":[{"text":"...","note":"..."}], "diagnosis":"..."}` |

Only `posts[].text` is required; the labels are optional. Also accepted: `success` omitted (`"success": false` is treated as an error), n8n's one-item array wrapper, `candidates` instead of `posts`, and plain strings as posts. **Revise with Claude** works only if the workflow handles `action: "revise"`. If it does not, the console says so and hand editing still works.

The console does not send Gold Standards to the workflow, and does not assume the workflow reads `standards/gold-standards.md`.

### Zernio

Choose **Direct** and paste the API key, then press **Find the Instagram account**. If that fails with "Could not reach Zernio from this browser", Zernio is refusing calls from web pages (CORS). Use the relay in section 4. Zernio's documentation does not say either way, so this is the first live check.

### GitHub

The repository and branch are filled in. To let the console write the Gold Standards file itself, add a fine-grained token with **Contents: read and write** on this repository only, with an expiry date. Without a token, the console downloads the updated file for you to commit.

### What browser storage does and does not protect

Keys typed into Settings are saved in this browser's `localStorage`. That keeps them out of the repository and off the public page, but it is **not server-side security**. They can be read by anyone using this browser profile, by browser extensions allowed on the page, and by any script running on `jolthevc.github.io` (every GitHub Pages site under your account shares that storage). Treat this device as holding the keys.

- **Direct mode** stores the Zernio API key itself. Anyone who reads it has full API access to your Zernio account until you revoke the key in Zernio.
- **Relay mode** stores only the relay passphrase. The Zernio key lives in the Worker's encrypted secrets and never reaches a browser. The passphrase still lets its holder use your Zernio account through the relay, so guard it the same way. The gain is that the Zernio key is never exposed, and you can cut off access by changing one Worker secret.
- The n8n key and the GitHub token are stored the same way. Keep the token narrow and short-lived.
- **Export** in Settings leaves keys out.

## 4. The relay (only if Direct fails, or to keep the Zernio key out of the browser)

A Cloudflare Worker of about 30 lines. It adds the Zernio key on the server side and answers the browser's CORS checks. It is the only backend piece and it is optional. The free plan is enough.

1. Sign in at dash.cloudflare.com. Open **Workers & Pages**, then **Create**, then **Create Worker** (the "Hello World" starter). Name it `raisehell-relay` and press **Deploy**.
2. Press **Edit code**, replace everything with the code below, and press **Deploy**.
3. Open the Worker's **Settings**, then **Variables and Secrets**. Add, each with type **Secret**:
   - `ZERNIO_API_KEY`: your `sk_` key.
   - `PASSPHRASE`: a long random phrase you invent (for example 6 random words).
   - `ALLOWED_ORIGIN` (type Text is fine): only if the console is not at `https://jolthevc.github.io`. Use the scheme and host only, no path.
   Press **Deploy** after adding them.
4. Copy the Worker address shown on its overview, like `https://raisehell-relay.<your-subdomain>.workers.dev`.
5. Check it from a terminal. Replace the address and passphrase:
   ```
   curl -s https://raisehell-relay.<your-subdomain>.workers.dev/v1/accounts?platform=instagram -H "Authorization: Bearer <PASSPHRASE>"
   ```
   It should print your Instagram account as JSON. The same command without the header must answer `Wrong relay passphrase.`
6. In the console, Settings, Zernio: choose **Relay**, enter the Worker address and the passphrase, delete any API key you typed earlier, and press **Find the Instagram account**.

To revoke access later, change `PASSPHRASE` (every browser must then enter the new one), or revoke the key in Zernio.

```js
// RAISE HELL relay for Zernio (Cloudflare Worker). Keeps the Zernio API key
// out of the browser and answers the browser's CORS checks.
// Secrets: ZERNIO_API_KEY, PASSPHRASE. Optional: ALLOWED_ORIGIN.
export default {
  async fetch(request, env) {
    const cors = {
      'Access-Control-Allow-Origin': env.ALLOWED_ORIGIN || 'https://jolthevc.github.io',
      'Access-Control-Allow-Headers': 'Authorization, Content-Type, Idempotency-Key',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Max-Age': '86400'
    };
    if (request.method === 'OPTIONS') return new Response(null, { status: 204, headers: cors });
    if (request.headers.get('Authorization') !== 'Bearer ' + env.PASSPHRASE) {
      return new Response('{"error":"Wrong relay passphrase."}', { status: 401, headers: { ...cors, 'Content-Type': 'application/json' } });
    }
    const url = new URL(request.url);
    const headers = new Headers();
    let target;
    if (url.pathname === '/__upload') {
      // Carries a file to the presigned storage URL that Zernio issued.
      target = url.searchParams.get('url') || '';
      const t = new URL(target);
      if (t.protocol !== 'https:' && t.hostname !== '127.0.0.1') return new Response('Bad upload URL', { status: 400, headers: cors });
      headers.set('Content-Type', request.headers.get('Content-Type') || 'application/octet-stream');
    } else {
      target = (env.ZERNIO_BASE || 'https://zernio.com/api') + url.pathname + url.search;
      headers.set('Authorization', 'Bearer ' + env.ZERNIO_API_KEY);
      for (const h of ['Content-Type', 'Idempotency-Key']) if (request.headers.get(h)) headers.set(h, request.headers.get(h));
    }
    const body = ['GET', 'HEAD'].includes(request.method) ? undefined : await request.arrayBuffer();
    const res = await fetch(target, { method: request.method, headers, body });
    const out = new Response(res.body, { status: res.status, headers: res.headers });
    for (const [k, v] of Object.entries(cors)) out.headers.set(k, v);
    return out;
  }
};
```

`ALLOWED_ORIGIN` only stops other websites from using the relay inside a visitor's browser. It does not stop someone who has the passphrase from calling the relay with a script, which is why the passphrase matters.

## 5. Music, posting times, then one test Reel

1. **Settings, Instagram music**: search, listen, and choose a default track plus up to two backups.
2. **Settings, Posting**: enter your times (for example `09:00, 18:00`) and the time zone, then press **Save to Zernio**. This becomes the account's posting queue in Zernio.
3. **Settings, Stories**: pick one of the two modes (section 7 explains the difference).
4. **Studio**: generate, select one piece, and approve it.
5. **Queue**: open Preview, then **Make the Reel video**. Watch it play and read the "Checked:" line: it should start with `H.264`. Tick the piece, then press **At a time** and pick ten minutes from now.
6. When it publishes, open it on Instagram. Confirm that it plays as a Reel, the music plays, the text is right, and the Story follows.

## 6. Scheduled generation (optional, a change inside the existing workflow)

Not needed for the console. If you want batches to arrive without pressing Generate, add to `Raise Hell - Generation` (do not create another workflow):

1. A **Schedule Trigger** whose output carries the same request body the webhook receives, `{"action":"generate","count":10,"theme":"","direction":"","avoid_topics":[]}`, connected to the node that currently reads the webhook body.
2. After the node that produces the posts, a **GitHub** node (Create file) that runs only for scheduled executions and writes `content/inbox/<date and time>.json` containing `{"posts":[{"text":"..."}],"batch_note":""}`.

Node names depend on how your workflow is built, so adapt the wiring. Then **Check for scheduled batches** in the Studio imports new files (it needs the GitHub token). The repository is public, so `content/inbox/` is public; point the GitHub node at a private repository if that matters. This was not tested against your workflow.

## 7. What runs without the console, and what does not

The console is a web page. Nothing in it runs while it is closed. Zernio is a service, and it does run.

| Zernio does this with the page closed | Only happens while the page is open |
|---|---|
| Publishes every scheduled Reel at its time, with its catalog track | Generating, editing, approving |
| Fails a Reel whose track has disappeared instead of publishing it silent | Making the videos and scheduling them |
| Publishes Stories that were scheduled together with their Reel, **even if that Reel failed** | Removing the Story of a Reel that failed (only if the page is opened before the Story's time) |
| Reposts winners already handed to its recycling, with their music | Picking winners and handing them to Zernio's recycling |
| Syncs metrics | Stories in "only after the Reel has published" mode, and Stories after reposts |
| | Gold Standards promotion |

The settings labelled "when this page is open" act each time the page loads Zernio's data, on whatever happened since it was last opened. Nothing is picked, promoted or created in between.

### The two Story modes

- **Schedule it together with the Reel.** When you schedule a Reel, the console also uploads the Story image and schedules it in Zernio for the chosen number of hours after the Reel. Both then publish with the page closed. Zernio does not link the two posts: if the Reel fails, the Story still publishes. The console removes such a Story only if someone opens the page after the Reel failed and before the Story's time.
- **Only after the Reel has published.** No Story exists until the Reel is live. The console creates it the next time the page is opened, within three days of the Reel, at least the chosen number of hours after it. A failed Reel never gets a Story. If nobody opens the page, no Story is posted.

Failure-aware Stories with the page closed would need a background job. It is not built.

### The seven-day media limit

Zernio keeps an uploaded file in temporary storage for 7 days, and the copy to permanent storage happens when the post publishes. A post scheduled to publish after its upload expired would fail. So the console never schedules a Reel, or its Story, more than **6 days** after uploading its media (one day of margin, set by `mediaScheduleDays` in the inline `window.RAISEHELL_CONFIG` block within `index.html`):

- **At a time** refuses later times and says the latest allowed one.
- **Schedule** (queue mode) checks Zernio's next free posting time first and stops when it is beyond the limit. If Zernio still places a post later, the console deletes it again and says so.
- **Reschedule** of an existing post refuses times after its media deadline, which is stored on the post.
- With Stories scheduled together with the Reel, the Reel must publish early enough for its Story to fit too (6 days minus the Story delay).

In practice: schedule up to six days of posts in one sitting, and come back each week for the next days. Reposts through Zernio's recycling are not affected: by then the original has published and its media is permanent.

Note: on 12 October 2026 Zernio's media guide said the copy happens when a post is created or updated, which would make far-ahead scheduling safe. Earlier wording, quoted in late September 2026, said it happens at publication. The console follows the stricter reading. Raise `mediaScheduleDays` only if Zernio confirms in writing that scheduled media is kept until it publishes.

### Full autonomy

New writing generated, chosen and published with nobody opening the page would need one scheduled job that generates, picks, makes the video and calls Zernio. It is not built: it adds a background service and an automatic approval policy that you should decide on first.

## 8. What was tested, and what still needs a live check

Tested here:

- Every console section on desktop and phone width, in demo mode with simulated Zernio and GitHub: 65 browser checks, including the seven-day limit for At a time, the queue, Zernio placing a post too late, and reschedules.
- The network path: n8n and Zernio stand-ins over HTTP, through the relay, with CORS enforced by the browser: 20 checks. Covers idempotent post creation, uploads to the presigned URL, the Zernio key replaced by the relay, and the n8n response variations listed in section 3.
- **The real H.264 path.** The console's own `video.js` ran in Chromium 140 (inside Electron) with OpenH264, the software H.264 encoder Chrome uses. Two Reels (15 s and 20 s) came out as H.264 High 4.0, yuv420p, progressive, 1080 × 1920, 30 fps, moov first, no edit list, no B-frames, every keyframe an IDR frame (closed GOP). ffprobe agreed with the console's own check, ffmpeg decoded every frame without errors, and the page played the files back.

Not tested, because no such browser was available here: Safari's and Windows or macOS Chrome's hardware encoders, and the silent AAC track, which is added only where the browser has an AAC encoder (Chrome on macOS and Windows, recent Safari; not Linux). The check before upload covers these at run time: a file that does not meet the specification is not uploaded.

Needs a live check with your accounts, in this order:

1. **Test with one post** against `/webhook/raise-hell/generate`, and **Revise with Claude** once.
2. Whether Zernio accepts direct browser calls (CORS). If not, the relay.
3. Facebook Login: the music search returns the catalog for your account.
4. The test Reel in section 5: Instagram accepts the H.264 file as a Reel and the catalog track plays.
5. Story publishing, and Zernio's posting queue placing posts at your times.
6. Analytics fields as Zernio returns them for your account.
7. Zernio recycling keeps the catalog track on the reposted copy.
8. The Gold Standards file write with your GitHub token.

Nothing here has run against your live accounts yet, so none of it should be treated as working until these checks pass.

## Files

The following `console/` paths describe the original modular source package, **not additional files required on GitHub Pages**. The deployed `index.html` already contains their contents.


| File | What it does |
|---|---|
| `index.html` | The complete, deployed console, including inlined CSS, scripts, fonts and rendering code. |
| `console/app.js` | The five sections. |
| `console/services.js` | Calls to n8n, Zernio and GitHub. |
| `console/video.js` | Makes the Reel MP4 and the Story JPG in the browser, and checks each MP4 against Instagram's specification. |
| `console/render.js` | Typesetting, shared by previews and the published media. |
| `console/demo.js` | Demo mode: sample writing and a simulated Zernio and GitHub. |
| `console/config.js` | Public defaults, including the 6-day media limit. Never put a key here. |
| `console/styles.css`, `console/fonts/` | Look and type. |
| `console/vendor/mp4-muxer.js` | MIT library that writes the MP4 file (license alongside). |
