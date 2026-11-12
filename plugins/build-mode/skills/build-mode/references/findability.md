# Findability - SEO, GEO and AIO

For any page a stranger should be able to find: marketing sites, landing pages, service and area pages, blogs, client websites. Screens behind a sign-in get `noindex` and nothing else from this file.

Decide it at kickoff without asking, from what the build is, and write one line in `docs/ARCHITECTURE.md`: `Findability: public`, `private`, or `mixed (<public routes>)`. Every later prompt reads it.

**It is one job, not three.** Google's guide (May 2026) says AI Overviews and AI Mode come from the normal Search index, with no extra requirements and no special files. So good SEO is most of AIO. GEO, being cited by ChatGPT, Claude and Perplexity, adds one hard rule on top.

## The hard rule: the words are in the first HTML

OpenAI's, Anthropic's and Perplexity's crawlers do not run JavaScript. Text that only appears after a script runs does not exist for them, so it can never be quoted or cited. On public routes:

- Server components by default. `"use client"` goes on the interactive leaf, never on a `page.tsx` or layout.
- Copy, prices, reviews and FAQs render on the server. Never fetched in `useEffect`.
- Showcase effects decorate words already in the HTML. SplitText splits server-rendered text; a 3D or video hero sits beside a real h1, never instead of one.

## Roadmap tasks on a public build

- **Phase 0 gets crawl basics**, in the deploy task or one of its own: `app/robots.ts` (allow everything by default, including OAI-SearchBot, Claude-SearchBot and PerplexityBot, because blocking a search crawler removes the site from that AI's answers), `app/sitemap.ts` built from the routes, `metadataBase` in the root layout, and a canonical on every page. Only the real domain is indexable: the host's default address (`*.up.railway.app`, `*.vercel.app`) and preview URLs send an `X-Robots-Tag: noindex` header. Whether to block AI training crawlers (GPTBot, ClaudeBot, Google-Extended) is the owner's call; write it in ARCHITECTURE.md.
- **An `[@]` task for the user once the domain is live:** verify it in Google Search Console and Bing Webmaster Tools and submit the sitemap. Bing feeds Copilot.
- **Replacing an existing site?** Phase 0 also crawls the old site's URLs and writes a 301 map into `next.config` redirects before launch. A relaunch without redirects throws away the old site's rankings. Ask for the old site's Search Console access first, so the pages that bring traffic are known.
- **Page plan and copy research, before any public page is designed.** List the services and the areas served, group them so one group becomes one page, and collect the questions each page must answer: the old site's Search Console queries, Google's "People also ask" box, and the real questions in the client's enquiry emails. If the SearchFit SEO plugin is installed in Cowork, its `keyword-clustering` and `content-brief` skills do this well. Write each page's target phrase and its questions into its entry in `docs/DESIGN.md` §Screens, so the answer-first copy has something real to answer.
- **Every milestone that adds public pages ends with a findability task** (`Mx-Tn Findability: <pages>`), next to its polish task.

## What every public page carries

- A unique title (about 60 characters) and meta description (about 155) from `generateMetadata`, plus an Open Graph image.
- One h1 that says what the page is in the words a customer would search. Headings in order. `header`, `nav`, `main` and `footer` landmarks.
- JSON-LD in the server HTML, marking up only what is visible on the page: `Organization` or `LocalBusiness` site-wide, with name, address and phone matching the Google Business Profile exactly; `Service` on service pages; `BreadcrumbList`; `Article` with author and date on posts.
- **Answer-first copy.** Each h2 is a question a customer asks ("How much does a boiler service cost in Leeds?") and its first sentence answers it, with the real number. Areas covered, accreditations, prices and named people are what both Google and AI answers lift. One page per service and per main area served, not one long page.
- Internal links: every service page links to related services and to contact. No orphan pages.
- `next/image` with width and height, a descriptive filename and alt text.

The word budgets in `references/design-brief.md` govern app screens and the hero. Below the hero, a public service page needs enough real copy to answer what a customer asks before they get in touch. A thin page is a findability fail even if it scores 5 on design.

## Sites that receive posts from another tool

If something other than Claude Code will publish posts into the site (a writing pipeline, an agent that opens pull requests, a headless CMS), the blog is part of the build, not an add-on.

**Agree the contract once, in one place.** File layout, frontmatter fields, reserved slugs and the validation conditions live in whichever repo owns the writer, because one writer may serve several sites. The site keeps a copy in `docs/BLOG-CONTRACT.md` with a line at the top saying it is a copy and linking back. Never reshape it for one site.

The roadmap then gets these tasks:

- **Blog route.** `/blog` and `/blog/[slug]`, built at build time from the post files and server-rendered. The page supplies the h1 from the post's title; the body starts at h2. Show the hero with its alt text, the author, an "Updated <date>", and the post's sources as a visible list. Article JSON-LD (headline, description, author, `datePublished`, `dateModified`, image) built from the frontmatter, plus `BreadcrumbList`.
- **Build-time validator.** Every condition in the contract fails the build, and no others: a check the writer does not know about turns an approved post into a red pull request with no preview.
- **Feeds.** Every post goes in `app/sitemap.ts` with `lastModified` from its updated date. Add `/rss.xml`. No route of the site may use a reserved slug.
- **A preview deploy on every pull request**, so a post can be looked at before it is merged. Vercel does this by default; on Railway, switching on PR environments is an `[@]` task.
- **`[@]` Connect it.** The user gives the writer the narrowest access that works (for a GitHub App, Contents read and write on that one repo) and runs the writer's connection test until it passes.
- **Done when:** a sample post that passes the validator renders at `/blog/<slug>` with its h1, sources and Article JSON-LD in the raw HTML and appears in the sitemap, and a malformed sample fails the build.

## Do not bother

`llms.txt` (Google says Search ignores it and AI search crawlers barely request it; only worth it for developer docs read by coding agents), separate Markdown or "chunked" pages for bots, keyword stuffing, bought mentions. If the user or a client asks for these, say why in one line and move on.

## Checking it

**Name SearchFit in findability prompts** only once `docs/TOOLING.md` says the plugin is installed on the Claude Code side too (`/plugin marketplace add searchfit/searchfit-seo`, then `/plugin install searchfit-seo`): `searchfit-seo:schema-markup` for the JSON-LD, `searchfit-seo:on-page-seo` and `searchfit-seo:internal-linking` for the page itself. Point at this file first. Where SearchFit and this file disagree, this file wins.

**Claude Code, as Done-when ticks on a findability task:** `curl -sL <live-url>` contains the h1 text, the title and an `application/ld+json` block, and the Lighthouse command from the polish task scores SEO 90 or more.

**Cowork, in Sync step 3, on any task that touched a public page:** with the browser's JavaScript tool, run `fetch(location.href).then(r => r.text())` and confirm the page's main copy is in that raw HTML, because that is all the AI crawlers see. Read `document.title`, the meta description and the canonical. Count h1s: exactly one. `JSON.parse` every JSON-LD block. Then open Google's Rich Results Test on the URL. Where SearchFit is installed in Cowork, run its `seo-auditor` agent on the live site on a milestone's findability task and check what it finds against this file. Any miss keeps the task `[?]` and the next prompt is a findability fix, the same way a design fail works.

Log anything this file gets wrong as a `[skill]` note, as usual.
