# Building and Deploying the App

## TL;DR

Deploy on **Lovable**. It's the right tool for this — you get frontend, backend function, and a public URL in one place, and you already know it. Railway, Vercel, and friends are overkill for v1.

This guide walks you through the Lovable build step by step, then explains the alternatives if you outgrow it.

---

## Part 1 — Lovable build (the paste-and-go path)

### Step 1. New Lovable project

Create a new Lovable project. When it asks what to build, paste this prompt verbatim:

```
Build a single-page web app called "Investor Voice."

PURPOSE: It rewrites founder investor letters in the voice of one of three famous business writers: Mark Leonard (Constellation Software), Marc Andreessen, or Jeff Bezos.

LAYOUT (single page, no routing, max width ~720px, centered):

1. Header
   - Title: "Investor Voice"
   - Subtitle: "Rewrite your investor letter in the voice of the people who wrote the best ones."
   - Small link in the corner labeled "GitHub" (placeholder URL "#" for now)

2. Step 1 — Input
   - Toggle (two buttons): "I have a draft" / "I have bullet points"
   - Below it: a large textarea, ~12 rows
   - Placeholder text changes based on the toggle:
     - Draft mode: "Paste your draft investor letter here..."
     - Bullets mode: "List the facts and updates you want in the letter, one per line..."
   - Below the textarea: a smaller textarea (3 rows) labeled "Context (optional)" with placeholder "Audience, stage, occasion — e.g. 'Series A investors, annual letter, just closed a tough quarter'"

3. Step 2 — Pick a voice
   - Three cards side by side on desktop, stacked on mobile.
   - Each card has: author name (large), one-line description, and an invisible click target so the whole card is clickable.
   - Selected card has a clearly highlighted border.
   - Cards:
     - "Mark Leonard" — "Dry, metric-driven, no hype. The serious capital allocator."
     - "Marc Andreessen" — "Punchy, contrarian, structurally aggressive. The operator-philosopher."
     - "Jeff Bezos" — "Plain language, long-term, principle-anchored. The patient builder."

4. Step 3 — Transform
   - Large primary button: "Rewrite"
   - Disabled until both an input AND an author are present.
   - When clicked, calls a backend endpoint /api/transform that calls the Anthropic Claude API.
   - While waiting, show a loading state: "Rewriting in [Author Name]'s voice..."

5. Step 4 — Output (conditional, appears below after first rewrite)
   - The rewritten letter in a styled card, serif font, generous line height.
   - "Copy" button copies the text to clipboard with a brief "Copied" confirmation.
   - "Start over" button resets all state.

STYLING:
- Cream / off-white background (#FAF8F3 or similar).
- Dark gray text (#2A2A2A or similar).
- Serif font for output (Lora, Source Serif Pro, or system serif).
- Sans-serif for UI (Inter or system sans).
- No emojis, no gradients, no glassmorphism. Editorial restraint.
- Single column, max-width ~720px, generous padding.

BACKEND:
- Create a backend function at /api/transform.
- It receives JSON: { inputType: "draft"|"bullets", userInput: string, context: string|null, author: "leonard"|"andreessen"|"bezos" }.
- It calls the Anthropic Messages API (model "claude-opus-4-7", max_tokens 2000) with:
  - system: the SYSTEM_PROMPT constant
  - messages: [{ role: "user", content: <templated string with author pack + user input> }]
- API key comes from process.env.ANTHROPIC_API_KEY — do NOT hardcode it.
- The Anthropic SDK is "@anthropic-ai/sdk".
- Returns { output: string }.

STATE:
- All state is local React state. No database. No accounts. No auth.

I will paste in the SYSTEM_PROMPT and the three author packs (LEONARD_PACK, ANDREESSEN_PACK, BEZOS_PACK) as string constants — please scaffold the backend so I can drop them in.
```

### Step 2. After Lovable scaffolds

Lovable will build the UI and the backend stub. You'll then need to:

1. **Paste the four prompt strings.** Open the file Lovable created with the backend function. You'll see placeholder constants for SYSTEM_PROMPT, LEONARD_PACK, ANDREESSEN_PACK, and BEZOS_PACK. Replace each with the corresponding file contents:

   - SYSTEM_PROMPT ← contents of `/prompts/system.md` (whole file as a string)
   - LEONARD_PACK ← contents of `/authors/leonard.md`
   - ANDREESSEN_PACK ← contents of `/authors/andreessen.md`
   - BEZOS_PACK ← contents of `/authors/bezos.md`

   Easiest way: in Lovable, ask "Paste this into the file as a string constant called SYSTEM_PROMPT" and drop in the contents. Repeat for each.

2. **Build the user message template.** In the backend function, after the author pack is selected, the user message sent to Claude should be:

```js
const authorPack = { leonard: LEONARD_PACK, andreessen: ANDREESSEN_PACK, bezos: BEZOS_PACK }[author];

const userMessage = `
<author_pack>
${authorPack}
</author_pack>

<user_input_type>
${inputType === "draft" ? "draft_to_rewrite" : "bullet_points_to_compose"}
</user_input_type>

<user_input>
${userInput}
</user_input>
${context ? `\n<optional_context>\n${context}\n</optional_context>\n` : ""}

Rewrite the user_input in the voice described in the author_pack, following all rules in your system instructions.
`.trim();
```

3. **The API call:**

```js
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

const response = await client.messages.create({
  model: "claude-opus-4-7",
  max_tokens: 2000,
  system: SYSTEM_PROMPT,
  messages: [{ role: "user", content: userMessage }]
});

const output = response.content
  .filter(block => block.type === "text")
  .map(block => block.text)
  .join("\n");

return Response.json({ output });
```

4. **Add the API key as a Lovable environment variable.**
   - Go to your Lovable project Settings → Environment Variables (or Secrets).
   - Add `ANTHROPIC_API_KEY` with your key from https://console.anthropic.com/.
   - Lovable will redeploy automatically.

5. **Test locally in the Lovable preview.** Use the testing checklist below.

### Step 3. Publish

In Lovable, click Publish (or Deploy). You'll get a public URL like `your-project.lovable.app`. That's the link you'll use in your LinkedIn post and in the GitHub README.

You can also connect a custom domain if you want (`investorvoice.com` etc.) — Lovable supports this on paid plans.

### Step 4. Update the GitHub README

Once you have the Lovable URL, edit `/README.md` and replace `[LINK_TO_LOVABLE_APP]` with the real URL. Commit and push.

---

## Testing checklist

Before you share the link, run all of these:

- [ ] Paste a real draft of one of your past investor updates, pick **Leonard**. Output should be dry, metric-led, no hype words. Sentences should vary — some short and blunt, some long with parentheticals.
- [ ] Paste bullets, pick **Andreessen**. Output should have em-dashes, numbered structure, and at least one slightly cocky aside. No corporate-speak.
- [ ] Paste a long draft, pick **Bezos**. Output should anchor on ONE named principle, not stack five. Plain language, no jargon.
- [ ] **Critical: fact preservation.** Include a specific number in your input (e.g. "$4.2M ARR"). Check the number appears in the output exactly. Check no other numbers were invented.
- [ ] Try empty input — Rewrite button should be disabled.
- [ ] Try with very long input (~2000 words) — should still return within ~30 seconds.
- [ ] Open browser dev tools, network tab. Check the request to /api/transform — the API key should NOT appear anywhere in the client-side request or response.
- [ ] Resize to mobile width. Cards should stack, layout should hold.
- [ ] Copy button works.
- [ ] Start over button resets all state.

If any of these fail, the most common fix is rewording the system prompt or the relevant author pack. Iterate.

---

## Cost

Each rewrite is one Claude Opus 4.7 API call.

- System prompt + author pack ≈ 2,500 tokens input
- User input ≈ 100–1,500 tokens
- Output ≈ 300–800 tokens

So a typical call is ~3,500 input + 500 output. At current Opus 4.7 pricing (check https://www.anthropic.com/pricing for live rates), figure on the order of single-digit cents per rewrite.

For a free open-source side project, you might serve 100–1,000 rewrites in the first weeks. Budget $20–$100. Monitor your usage in the Anthropic console.

**If you want to keep costs lower:** swap `claude-opus-4-7` for `claude-sonnet-4-6` or `claude-haiku-4-5-20251001`. Sonnet is much cheaper and still strong on style transfer. For v1 I recommend Opus because voice fidelity matters most — but if usage grows, drop to Sonnet.

---

## Part 2 — Alternative deploy targets (if you outgrow Lovable)

### Vercel

When to switch: you want more control, you're comfortable with Next.js, or Lovable's pricing stops making sense.

How: port the UI to a Next.js app (`pages/index.tsx` or `app/page.tsx`), put the API call in `pages/api/transform.ts` or `app/api/transform/route.ts`. Push to GitHub. Connect to Vercel. Add `ANTHROPIC_API_KEY` to Vercel env vars. Deploy.

Free tier is generous (100GB bandwidth/month). Should easily handle the early traffic from a LinkedIn post.

### Railway

When to switch: you grow this into a real product with a database, user accounts, scheduled jobs, queue workers, etc.

How: not yet. Railway is great but it's optimized for stateful apps with backends that need to run continuously. The current app is a thin frontend plus one serverless function — that's a Lovable / Vercel shape, not a Railway shape.

If you ever do go this route: build a Node/Express or FastAPI backend, deploy it on Railway, and host the frontend separately on Vercel or Cloudflare Pages.

### Cloudflare Pages + Workers

When to switch: cost is critical and traffic is high.

How: frontend on Cloudflare Pages, API call in a Cloudflare Worker. Free tier covers a huge amount of traffic. More setup work than Lovable.

---

## Recommendation

**Ship on Lovable. Don't optimize prematurely.** You can move later if you need to. The author packs and the system prompt are completely portable — they're just strings. The only thing tied to Lovable is the UI and the HTTP wiring, both of which take an hour to redo elsewhere if it ever matters.
