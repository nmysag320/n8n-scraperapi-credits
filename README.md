# n8n scrape javascript website: Render JS-Heavy Pages Without a Headless Browser, and Know What ScraperAPI Credits Actually Cost

If you're searching for ways to scrape JavaScript websites with n8n, the problem you've almost certainly hit is this: n8n's built-in HTTP Request node returns the initial HTML of a page, and for modern sites built with React, Vue, or similar frameworks, that HTML is mostly an empty shell. The content you want gets assembled by JavaScript in the browser — which n8n doesn't run. The fix is to hand the request to a service that renders the page in a real browser before returning it, and the cleanest option inside n8n is the official ScraperAPI community node, which handles JavaScript rendering, proxy rotation, and CAPTCHA bypass in a single node.

This article covers why the built-in nodes fall short, the three realistic ways to scrape dynamic sites from n8n, what ScraperAPI's plans and credits actually cost, and the billing details that most tutorials skip — like which plans stop working when credits run out.

> **Quick take:** For most people scraping JS-rendered pages in n8n, the **Startup plan at $149/month (1,000,000 credits)** is the sensible entry point — that works out to roughly $1.49 per 1,000 JavaScript-rendered pages. Registration comes with 5,000 free credits, no credit card required, so you can test your actual target sites before paying anything. 👉 [Check ScraperAPI's current plans and free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Why n8n's built-in nodes struggle with JavaScript sites

n8n is a workflow automation tool, and its HTTP Request node does exactly one thing: it sends an HTTP request and returns whatever the server sends back. There's no browser involved. When the target site is a single-page application or relies on client-side rendering, lazy loading, or infinite scroll, the "content" in that first response is a JavaScript bundle plus an empty DOM. Your next node gets raw HTML with no data in it.

Two more problems stack on top. First, protected sites detect the request coming from a datacenter IP with no browser fingerprint and return a 403, a CAPTCHA page, or empty data — and the failure is silent, so downstream nodes keep processing nothing. Second, even if you get HTML back, it's raw markup that your next node has to parse manually.

A scraping API flips this around. Your n8n workflow makes one normal HTTP call to the API, the API renders the page in a real browser on its own infrastructure, rotates through residential proxies, solves CAPTCHAs, and returns the final rendered content — as HTML, markdown, or pre-parsed JSON. Nothing to install on your n8n server, no browser to maintain, no proxy pool to manage.

## Three ways to scrape dynamic sites from n8n

There are three approaches that actually work, and they differ a lot in effort and reliability.

**1. Run a headless browser in a Code node.** Technically possible on a self-hosted n8n instance if you install Puppeteer or Playwright alongside it. You get full control, but you're now maintaining a browser, managing proxies and retries yourself, and debugging why a site changed its markup. Most teams that go this route end up rebuilding the same anti-bot plumbing every scraping API already sells.

**2. Call a scraping API from an HTTP Request node.** This is the pattern most guides recommend: one HTTP Request node pointed at a scraping API endpoint, with the target URL and a rendering flag as query parameters. It works fine, and several vendors support it — Firecrawl has a community node (n8n-nodes-firecrawl-v2) with JavaScript rendering and AI-powered extraction that works with both its cloud service and self-hosted instances; other scraping APIs can be called through plain HTTP Request nodes the same way. The downside of the raw-HTTP approach is that failures come back as API responses you have to interpret yourself, and structured output depends on what each vendor returns.

**3. Use the official ScraperAPI node.** ScraperAPI maintains an official n8n community node (n8n-nodes-scraperapi-official) that wraps its whole API in native n8n operations. Beyond single-URL scraping, it includes structured data endpoints that return pre-parsed JSON for Amazon, Google, eBay, Walmart, and Redfin. Concretely: an Amazon product request comes back with fields like title, price, stock status, rating, and review count already mapped — the kind of output you could send straight to Google Sheets or Slack without writing a parser. Failed requests return a clear error instead of silent empty data, which matters a lot when a scheduled workflow runs unattended at 6 a.m.

If your automation already lives in n8n, the native node keeps triggers, logic, and data handling in one place. Point-and-click tools like Browse AI run robots on their own platform and schedule instead, which is a different workflow shape entirely.

## ScraperAPI plans and pricing

ScraperAPI sells credits on six self-serve tiers plus a custom Enterprise plan. Annual billing knocks 10% off every tier. Here's the full current lineup:

| Plan | Monthly | Annual (per month) | Credits/month | Concurrent threads | Subscribe |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [ Sign up for free credits](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ Subscribe to Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ Subscribe to Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ Subscribe to Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ Subscribe to Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ Subscribe to Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ Subscribe to Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22M+ | 500+ | [ Request a quote](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Every paid tier includes the same feature set: JavaScript rendering, premium proxy tiers, CAPTCHA handling, JSON autoparsing, DataPipeline, and API access. What changes is volume, concurrency, and geotargeting — Hobby and Startup are limited to US and EU geotargeting, while Business and above unlock country-level targeting worldwide.

There's also a permanent free plan with 1,000 credits and 5 concurrent connections, separate from the 5,000-credit trial you get at signup.

### What a request actually costs in credits

This is where most pricing comparisons stop too early. A standard page costs 1 credit, but JavaScript rendering changes the math immediately:

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| With `render=true` (JS rendering) | 10 |
| Amazon / Walmart / eBay pages | 5 |
| With `render=true` + premium proxies | 25 |
| Google / Bing pages (incl. subdomains) | 25 |
| LinkedIn pages | 30 |
| With `render=true` + ultra-premium proxies | 75 |
| Cloudflare / DataDome / PerimeterX bypass | +10 per request |

A few parameters are free and don't add credits: `country_code`, `session_number`, `device_type`, `output_format`, `wait_for_selector`, and `autoparse`. Geotargeting and sticky sessions being free is uncommon — some competitors charge extra for both.

You can check the exact cost of any URL with the Domain Cost Estimator in the dashboard, and set a `max_cost` per scrape so a single request can't blow through your allowance.

## Setting up the ScraperAPI node in n8n

Setup takes about five minutes, and most of it is one-time configuration:

1. In your n8n workflow, click **+** to add a node and search for **ScraperAPI** in the Nodes panel, then click Install. It works on both n8n Cloud and self-hosted instances.
2. Grab your API key from the ScraperAPI dashboard and paste it in as a credential.
3. Select the **API** resource, enter the target URL, and configure optional parameters.
4. For JavaScript-rendered pages, toggle **render** on. It increases processing time, so the official docs recommend enabling it only when the page actually needs it.
5. Execute the workflow. The node returns rendered content, or a clear error if the request failed.

The parameter list is worth a skim before you build, because several options are mutually exclusive — a detail that trips people up when a workflow suddenly stops working:

- `premium` (residential/mobile proxies) and `ultraPremium` (advanced bypass for the hardest sites) can't be combined with each other.
- `sessionNumber` (reusing the same proxy session) is incompatible with both `premium` and `ultraPremium`.
- `retry404` is unavailable when `premium` or `ultraPremium` is on.
- `customHeaders` are discarded under `ultraPremium`, and the docs warn that overriding ScraperAPI's default header management often lowers success rates — only use headers when you genuinely need them.

For AI-driven workflows, the node also plugs into an n8n AI Agent as a tool: add a Chat Message trigger, an AI Agent node, connect a chat model and memory, then wire the ScraperAPI node into the Agent's Tool input. ScraperAPI additionally runs a hosted MCP server that works with n8n's MCP Client Tool node, if you'd rather expose scraping to agents that way. If you'd rather not configure anything at all, official workflow templates — like daily Amazon and Walmart price-drop digests — can be imported and adapted.

You can test all of this on the free signup credits before committing: 👉 [Sign up and get 5,000 free API credits — no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

## The credit math most tutorials skip

Here's the part that determines whether a plan is actually cheap for you: credits don't roll over. When your subscription renews, unused credits reset to zero. That means you should size the plan against your peak month, not your average one — a team that picks a plan based on average usage finds their crawler halted on the 22nd.

The overage rules also differ by tier, and this is easy to miss:

- **Hobby, Startup, and Business:** when credits run out, scraping pauses until renewal. You can opt into auto-upgrade, which bumps you to the next plan automatically, but there's no pay-as-you-go option.
- **Scaling, Professional, Advanced, and Enterprise:** you can enable pay-as-you-go to keep scraping past your limit at a fixed per-credit rate, capped at a monthly spending limit you set.

On cost per page, the curve flattens quickly. Hobby works out to about $0.49 per 1,000 standard pages; Startup drops that to $0.149; Business and above hover around $0.09–$0.10. For rendered pages specifically, Startup runs roughly $1.49 per 1,000 — that's the number most teams building JS-scraping workflows should budget against. Moving from Startup to Business cuts the unit price by about a third; beyond Business, the extra $1,000+ per tier mostly buys concurrency and overage flexibility rather than cheaper pages.

A practical way to pick a tier, using the free credits: take 200 URLs that represent your real crawl (including the awkward ones, not just homepages), run them with no flags, and count how many return usable content in the raw HTML — that's your 1-credit share. Re-run the failures with `render=true`; anything that now works is a 10-credit page. Whatever's still empty needs premium or ultra-premium at 25 or 75 credits. Multiply your weighted average by monthly volume, then pick the tier covering your peak month.

## When ScraperAPI isn't the right pick

To be fair about the alternatives: if your target sites don't block bots and you're comfortable maintaining infrastructure, calling a rendering service through a plain HTTP Request node can be cheaper at small scale. Firecrawl's community node is a reasonable option if you want markdown output with AI-powered extraction and don't need proxy tiers or geotargeting granularity.

ScraperAPI also has real constraints worth knowing. Hobby and Startup only support US and EU geotargeting, so if you need localized results from other countries, Business is the practical floor. LinkedIn pages at 30 credits each and ultra-premium rendering at 75 add up fast — scraping LinkedIn at any volume pushes you toward higher tiers quickly.

## FAQ

**Does n8n need a proxy or browser to scrape JavaScript sites with ScraperAPI?**
No. Proxy rotation across 40M+ IPs, CAPTCHA handling, and JavaScript rendering all run automatically on ScraperAPI's infrastructure. Even self-hosted n8n instances don't need proxies or a browser installed.

**Do failed requests cost credits?**
No — you're billed only for successful requests, defined as 200 and 404 responses, plus requests you cancel before the 70-second timeout. Requests that return 403 or time out don't consume credits.

**What success rate does ScraperAPI claim?**
The official n8n page cites up to a 99% success rate. That's a vendor claim, not an independent measurement, which is exactly why testing your specific target sites on free credits first is worth the hour it takes.

**Do unused credits roll over?**
No. Credits reset when your subscription renews, though annual plans can renew early for a fresh batch.

## Which plan should you pick

A few concrete rules based on the verified numbers:

- **Just testing, or a few hundred pages a month:** stay on the free plan (1,000 credits, 5 concurrent connections). It's permanent, not a trial.
- **Scraping JS-rendered pages regularly, up to a few hundred thousand per month:** Startup at $149/month is the workhorse tier, with pay-as-you-go headroom if you upgrade the moment you outgrow it.
- **Need non-US/EU geotargeting:** Business at $299/month, since country-level targeting starts there.
- **Scraping Amazon, Google, or LinkedIn heavily:** budget with those domain costs (5/25/30 credits) rather than the 1-credit standard rate — a Google-heavy workflow burns credits 25× faster than the sticker price suggests.

One last check before you subscribe: run your real target URLs through the free credits first, confirm the content you need comes back, and only then pick the tier sized to your peak month. 👉 [Start with ScraperAPI's free credits and test your target sites](https://www.scraperapi.com/?fp_ref=coupons)
