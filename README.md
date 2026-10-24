# dynamic web scraping: how to handle JavaScript-rendered pages, what each method really costs, and how to pick a plan without burning credits

Most pages that look empty when you fetch them with a plain HTTP request aren't broken. The data loads after JavaScript runs — React components mount, infinite scroll fires, an AJAX call fills the table. That's the core problem of dynamic web scraping: the content you see in the browser simply isn't in the raw HTML your script receives.

There are three ways to deal with it, and the right one depends on how often you scrape and how hostile the target site is. Call the site's underlying API directly if you can. Run your own headless browser if you need full control. Or use a scraping API with built-in JavaScript rendering — which is where ScraperAPI comes in — when you want rendering, proxies, and CAPTCHA handling handled for you. This guide walks through all three, then digs into something almost nobody explains properly: what each request actually costs once you account for the credit system.

> **Quick picks.** If you just want a working answer: the **Hobby plan** at $49/month (100,000 credits, JS rendering included) covers most people getting started with dynamic web scraping. Before paying anything, grab the **7-day trial with 5,000 credits — no credit card required** — and there's also a permanent free plan with 1,000 credits at 5 concurrent requests. 👉 [See all ScraperAPI plans and current prices](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How to tell whether a page is actually dynamic

Open the page's raw source (Ctrl+U in the browser, or fetch the URL with curl). Compare what you see with the rendered page. If the data shows up in the browser but is missing from the raw HTML, JavaScript injects it after load — the page is dynamic, and a plain HTTP request will never see the content.

The usual suspects: single-page apps built on React, Vue, or Angular; product listings loaded via AJAX; infinite-scroll feeds; dashboards. Some sites also move data over WebSockets, which rules out even simple AJAX interception. Once you've confirmed the content is dynamic, you have a decision to make about method.

## Three ways to scrape dynamic content, ranked by effort

**1. Reverse-engineer the site's own API.** Open the browser's DevTools network tab, find the JSON endpoint that actually delivers the data, and call it directly. Tools like curl_cffi exist specifically for mimicking a browser's TLS fingerprint so raw HTTP clients don't get blocked. This is the cheapest and fastest route, and in most cases it should be your first attempt, not browser automation. The downside: endpoints change without notice, and sites with serious anti-bot protection may reject raw clients regardless of fingerprinting.

**2. Run your own headless browser.** Selenium, Playwright, and Puppeteer can execute JavaScript, wait for elements, and screenshot pages. You get full control — but you also inherit everything that comes with it: browser maintenance, proxy rotation, CAPTCHA solving, retry logic, and scaling across machines. For a one-off script this is fine. For anything running daily at volume, it turns into an infrastructure project.

**3. Use a scraping API with built-in rendering.** You send a URL plus a render flag; the service runs the headless browser, routes the request through its proxy pool, and handles CAPTCHAs and bot detection. With ScraperAPI, `render=true` triggers full JavaScript rendering, and `wait_for_selector` tells it to wait until a specific element appears before returning — which is exactly what you need when content mounts asynchronously. A rendering request costs 10 extra credits (see the cost section below), and the docs list the same multipliers for combinations like rendering plus premium proxies (25 credits) or ultra premium with rendering (75 credits).

The practical rule most experienced scrapers converge on: try the site's own API first. If it's locked down or the site rotates endpoints constantly, hand the browser work to a service instead of maintaining it yourself.

## ScraperAPI plans: the full lineup

Here's every plan currently shown on the pricing page, with monthly and annual pricing. Annual billing works out 10% cheaper across the board.

| Plan | Monthly price | Annual (per month) | Credits/month | Concurrency | Get started |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Start with Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Start with Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Start with Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Start with Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Start with Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Start with Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Get a custom quote](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Every paid plan includes JS rendering, premium residential and mobile IPs, JSON auto-parsing, rotating proxy pools, custom headers, CAPTCHA and anti-bot detection, session control, desktop and mobile user agents, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. A permanent free plan (1,000 credits, 5 concurrent requests) exists separately for testing.

A few plan-level differences don't fit in the table. Geotargeting on Hobby and Startup is limited to US and EU locations; Business and above add country-level targeting. Usage analytics only go back 30 days on the lower tiers; Business and up get unlimited history.

## The credit math that pricing pages gloss over

This is the part most reviews skip, and it's the difference between a plan that lasts a month and one that dies in three days.

A standard page costs 1 credit. That's the baseline. But certain domains have automatic multipliers that apply before you've enabled a single feature: Amazon, eBay, and Walmart cost **5 credits per request**; Google and Bing (including all subdomains) cost **25**; LinkedIn costs **30**. So that Hobby plan's 100,000 credits translates to 20,000 Amazon product pages, 4,000 Google searches, or roughly 3,333 LinkedIn pages. The marketing number and the real number are not the same thing.

On top of domain costs, parameters stack:

- `render=true` adds 10 credits — so a rendered page on a normal site costs 11, meaning Hobby covers about 9,000 rendered pages per month
- `premium=true` adds 10; `premium` + `render` together cost 25
- `ultra_premium=true` costs 30; with rendering, 75
- Bypassing protected sites — Cloudflare, Datadome, PerimeterX/Human, Akamai, Cloudflare Turnstile — adds 10 credits per request once the bypass activates

Now the genuinely good news, which almost no comparison article mentions: several parameters you'll absolutely need for dynamic scraping cost nothing extra. `wait_for_selector` (waiting for your element to appear), `country_code`, `session_number`, `device_type`, `output_format`, `keep_headers`, and `autoparse` are all free. The waiting logic that makes dynamic scraping work doesn't inflate your bill — rendering does.

Billing is also narrower than you might expect. ScraperAPI only charges for successful requests — status codes 200 and 404 — and for requests you cancel yourself before giving the service its 70-second window to finish. Failed renders and blocked attempts inside that window don't get billed.

Two tools keep this under control. Every response includes an `sa-credit-cost` header telling you exactly what that request consumed. And before scraping anything new, check the cost first: the dashboard has a Domain Multiplier lookup, the API Playground lets you test a URL and see actual usage, and there's a direct endpoint at `https://api.scraperapi.com/account/urlcost` if you'd rather script it. You can also set a `max_cost` per request as a hard ceiling. Checking one URL costs nothing and prevents the classic mistake of burning a month's credits on a domain you assumed was 1 credit. 👉 [Check your target URLs' credit costs before committing to a plan](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Which plan fits which situation

If you're scraping normal websites with rendering enabled and stay under roughly 9,000 pages a month, Hobby is the honest fit — and $49 is less than the cost of maintaining your own proxy pool. If most of your volume is Amazon or other e-commerce domains, recalculate at 5 credits per page before choosing; Hobby's 100,000 credits is really 20,000 pages there.

Concurrency matters more than people expect. Exceed your plan's concurrent limit and the API returns a 429 — the request is turned away rather than queued, and there's no charge for it. You just retry at a lower concurrency. Hobby allows 20 simultaneous requests, Startup 50, Scaling 200.

What happens when credits run out mid-month? By default, scraping pauses until the next cycle. But there are two knobs: plans from Hobby through Professional can enable auto-upgrade, which bumps you to the next tier automatically when credits exhaust; and from Scaling upward you can use pay-as-you-go, continuing at a fixed per-credit rate with a monthly cap you set yourself. Annual subscribers can also renew early to get a fresh batch of credits.

On billing period: if you're confident you'll use the service for more than a couple of months, annual billing saves 10% on every plan — Hobby drops from $49 to $44.10 per month. 👉 [Compare monthly and annual pricing side by side](https://www.scraperapi.com/pricing?fp_ref=coupons)

## When ScraperAPI isn't the right tool

Being straight about this: a scraping API isn't the answer to every dynamic scraping problem, and the alternatives are worth knowing even though we won't send you anywhere but the product above.

- If your output feeds an LLM or RAG pipeline and you want clean Markdown and structured JSON without preprocessing, tools built around that — Firecrawl is the most prominent example — will fit the job more naturally than a raw HTML API.
- If what you actually want is scheduled, repeatable runs with pre-built scrapers you configure once, Apify's actor ecosystem is designed around exactly that workflow.
- At enterprise volumes with strict reliability requirements, Bright Data's proxy network is the heavyweight option most comparisons recommend for that tier.
- And if you're scraping one mostly-static site occasionally, a plain HTTP request plus the site's own JSON endpoints may be all you need. Paying for rendering you never trigger is wasted money.

## Questions people ask before signing up

**Does JS rendering actually work on React and Vue sites?** Yes — `render=true` runs a real browser before returning HTML, and `wait_for_selector` holds the request until your target element exists. Both are documented, current API parameters, not marketing claims.

**What happens if I go over my concurrency limit?** You get a 429 response and the request is turned away, at no charge. Reduce concurrency and retry.

**Can I test it before paying?** There's a 7-day trial with 5,000 API credits and no credit card required, plus the permanent free plan with 1,000 credits at 5 concurrent requests. That's enough to render a few hundred dynamic pages and see whether the output quality works for your use case. 👉 [Start the free 5,000-credit trial](https://www.scraperapi.com/?fp_ref=coupons)

**How do I avoid surprise credit burn?** Check the `sa-credit-cost` header on your first requests, use `max_cost` as a ceiling, and look up unfamiliar domains in the Domain Multiplier before pointing a script at them.

## Before you subscribe

A few concrete rules to close on:

- Occasional dynamic pages from normal sites → Hobby monthly, paid for a month or two while you validate your scripts.
- E-commerce-heavy workloads → do the 5-credits-per-page math first; Hobby may be tighter than it looks.
- Heavy SERP or LinkedIn scraping → Hobby covers only about 4,000 Google queries or 3,300 LinkedIn pages; count your real volume before committing.
- More than 20 concurrent requests or non-US/EU geotargeting → step up to Startup or Business.

One last check before checkout: confirm whether you want monthly or annual billing, since the 10% discount only applies to annual plans — and if you're still unsure whether rendering handles your specific target site, the trial exists precisely for that question. 👉 [Start with the 7-day trial and test your target sites first](https://www.scraperapi.com/?fp_ref=coupons)
