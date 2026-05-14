# Investor Voice

**Rewrite your investor letter in the voice of Mark Leonard, Marc Andreessen, or Jeff Bezos.**

→ Live app: LINK
---

## What it does

You paste a draft letter (or just bullet points). You pick one of three authors. You get back a rewritten version that sounds like that person wrote it.

That's the whole tool. One screen, one button, free, no signup.

## Why I built it

I started a company at 17. I never properly learned how to communicate with investors — there's no class for it, and the people who do it well rarely explain how. Saagar introduced me to *one letter a day* and I got obsessed with reading the people who got it right.

So I pulled apart how three of the best founder-investor communicators actually write — Mark Leonard at Constellation Software, Marc Andreessen in his pmarca essays, and Jeff Bezos in his Amazon shareholder letters — and turned each one into something you can write through.

If you're a founder who never learned this either, this is for you.

## The three voices

- **Mark Leonard** — Dry, metric-driven, no hype. The serious capital allocator. Reach for this when you want investors to take you seriously over thinking you're exciting.
- **Marc Andreessen** — Punchy, contrarian, structurally aggressive. The operator-philosopher. Reach for this when you're making a thesis-driven argument that pushes against consensus.
- **Jeff Bezos** — Plain language, long-term, principle-anchored. The patient builder. Reach for this when you need to communicate strategic patience to potentially impatient investors.

## How it works (technical)

There's no model training. No fine-tuning. No vector database. No RAG.

Each author has a hand-written **style pack** in `/authors/` — a compressed fingerprint of how they actually write. Sentence rhythm. Vocabulary. Signature rhetorical moves. Anti-patterns. A synthetic input→output calibration example so the model knows what a good rewrite looks like for that voice.

The system prompt in `/prompts/system.md` tells Claude how to use the pack: preserve every fact, adopt the voice, never invent numbers, never blend authors, never stack every cliché into one paragraph.

That's it. One Claude API call per rewrite. The whole repo is small enough to read in 15 minutes and fork in 30.

The reason it works better than asking ChatGPT to "write like Bezos" is that the packs encode the *specific* things each writer does and doesn't do — Leonard's parenthetical qualifiers, Andreessen's em-dash chains and enumerated arguments, Bezos's principle-naming. Generic models don't know these by default. The pack tells them.

## Repo structure

```
investor-voice/
├── README.md               this file
├── LICENSE                 MIT
├── authors/
│   ├── leonard.md          Mark Leonard style pack
│   ├── andreessen.md       Marc Andreessen style pack
│   └── bezos.md            Jeff Bezos style pack
├── prompts/
│   ├── system.md           the system prompt for the API call
│   └── transform.md        the user-message template
├── corpus/
│   └── sources.md          links to the public source material
└── app/
    └── README.md           Lovable build + deploy instructions
```

## Forking — add your own author

About a day's work end-to-end.

1. Pick a writer with a real corpus of investor-facing prose actually written by them (not interviews, not transcripts of talks).
2. Read 8–10 of their letters carefully.
3. Copy `authors/leonard.md` as a template and fill in:
   - Voice fingerprint (tone, sentence structure, vocabulary register)
   - Sentence-length statistics
   - Structural fingerprint (opening shape, transition shape, closing shape)
   - Signature rhetorical moves
   - Anti-patterns (things they would never write)
   - One synthetic input→output calibration example
4. Add the author to the picker in the Lovable app.

The hard part isn't the writing — it's being honest about what the writer actually does, versus what you wish they did. Read closely.

## Credits

Inspired by Saagar's *one letter a day* practice — the reason I started reading these in the first place.

The pack-based style-transfer technique is the same approach the Strawberry founder used to scrape and analyze the Lovable founder's posts.

## License

MIT for the code and the author packs.

Source letters belong to their respective authors and publishers. This repo doesn't redistribute them — see `/corpus/sources.md` for links to the public originals.

## Built with

- [Claude API](https://www.anthropic.com/api) for the rewrite
- [Lovable](https://lovable.dev/) for the frontend and deploy
- A weekend
