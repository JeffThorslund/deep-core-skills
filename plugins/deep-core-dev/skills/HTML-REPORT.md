# HTML Report Format

Shared report-craft for the interactive `improve-codebase-*` skills. The review is rendered as a single self-contained HTML file in the OS temp directory so nothing lands in the repo. Tailwind and Mermaid both come from CDNs. Mermaid handles graph-shaped diagrams reliably; hand-built divs and inline SVG handle the more editorial visuals. Mix the two — don't lean on Mermaid for everything, it'll start to look generic.

This file is **domain-neutral**: it covers the scaffold, the card structure, the diagram patterns, and the style/tone discipline that every review report shares. Each skill supplies its **own** domain specifics — the title, the legend, the badge taxonomy, and the controlled vocabulary — from its `SKILL.md`. Where this file says *"(domain-specific — see the calling skill)"*, read those from the skill that linked you here.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>{{review type}} — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly.
         The calling skill defines the domain-specific classes it needs
         (e.g. dashed-line accents, danger edges, emphasis fills). */
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Review type, repo name, date, and a compact **legend** *(domain-specific — see the calling skill)* mapping each visual element to its meaning. No introduction paragraph — straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the calling skill's controlled vocabulary *(domain-specific)* without ceremony.

Each candidate is one `<article>`:

- **Title** — short, names the change in the domain's terms.
- **Badge row** — a **recommendation-strength** badge (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus any domain-specific tags/badges the calling skill defines *(domain-specific — e.g. a category tag, a severity badge)*.
- **Files** — monospaced list, `font-mono text-sm`.
- **Before / After diagram** — the centrepiece. Two columns, side by side. See patterns below.
- **Problem / weakness** — one sentence. What hurts.
- **Solution / change** — one sentence. What changes.
- **Wins** — bullets, ≤6 words each, named in the domain's vocabulary.
- **Callout** (if applicable) — one line in an amber-tinted box (e.g. an ADR conflict).

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point. Style edges/nodes with `classDef` to carry the domain's meaning (the calling skill says which colours mean what).

### Mermaid graph (the workhorse for dependencies / call flow / data flow)

Use a Mermaid `flowchart` or `graph` when the point is "X reaches Y reaches Z, and look at the problem." Wrap it in a Tailwind-styled card so it doesn't feel parachuted in. Sequence diagrams work well for "before: 6 round-trips; after: 1."

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[Handler] --> B[Validator]
      B --> C[Repo]
      C -.problem.-> D[Client]
      classDef problem stroke:#dc2626,stroke-width:2px;
      class C,D problem
  </pre>
</div>
```

### Hand-built boxes-and-arrows (when Mermaid's layout fights you)

Nodes as `<div>`s with borders and labels. Arrows as inline SVG `<line>`/`<path>` positioned absolutely over a relative container. Reach for this when you want the "after" diagram to feel a specific weight Mermaid won't render.

### Cross-section (good for layered problems)

Stack horizontal bands (`h-12 border-l-4`) to show layers a call passes through. Before: many thin layers; after: one thick band labelled with the consolidated responsibility.

### Before/after mass (good for "X is as big as Y")

Two rectangles per node — proportioned to show the problem before and the improvement after.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: one accent (emerald or indigo) plus red for the problem signal and amber for warnings.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for node labels inside diagrams — they should read as schematic, not as UI.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. The report is otherwise static — no app code, no interactivity beyond Mermaid's own rendering.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise — but the domain nouns and verbs come straight from the calling skill's controlled vocabulary. Concision is not an excuse to drift. Each skill states the terms to **use exactly** and the ones to **never substitute** — honour those.

**Wins bullets** name the gain in the domain's vocabulary, not generic praise. Don't write *"easier to maintain"* or *"cleaner code"* — those don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in the domain vocabulary, reach for one that is before inventing a new one.
