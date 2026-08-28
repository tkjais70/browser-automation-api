# Browser Automation API Complete Guide: What Is It, How to Choose the Best One, How Does ScraperAPI Handle JavaScript Rendering, Clicks, and Scroll — Plus Full Plan Breakdown and Real Cost Analysis (With Honest Competitor Comparison)

---

There's a specific moment most developers know well. You've built a scraper, it works perfectly on localhost, you push it to production — and within 48 hours it's returning empty pages. The site now renders everything in JavaScript. Your clean HTML scraper is pulling back a skeleton with zero data.

That's the exact moment when a **browser automation API** starts making a lot of sense.

This guide is for people at that crossroads. We'll walk through what browser automation APIs actually are, how they work, what to look for when choosing one, and how ScraperAPI's implementation stacks up in practice — including its JavaScript rendering engine, Render Instruction Set for browser control, plan tiers, and the credit math you need to understand before committing.

---

**What Is a Browser Automation API, and Why Do You Actually Need One?**

The short version: a browser automation API is a hosted service that runs a real browser (usually headless Chromium) in the cloud and returns the fully rendered page to you via a simple HTTP call.

The longer version is more interesting.

When you send a plain HTTP GET request to a modern website — anything built with React, Vue, Angular, or any other SPA framework — you get back an HTML shell. The actual content you want (prices, product listings, search results) gets injected by JavaScript after the page loads. A raw request never sees any of it.

Your three options at that point:

1. **Run your own headless browser** — Playwright, Puppeteer, Selenium. You manage the instances, memory, driver versions, and retry logic yourself. This works fine for a handful of requests. At scale, it becomes a full-time infrastructure job.

2. **Use a browser automation API** — you send a URL, the provider runs the browser in their cloud, and you get back the rendered HTML (or structured JSON). No infrastructure to manage.

3. **Ignore the problem** — only works if the data you need happens to be in static HTML, which gets rarer every year.

The browser automation API model wins for most production scraping workloads because the hard parts — proxy rotation, browser fingerprinting evasion, CAPTCHA handling, auto-retries — are someone else's problem. You pay per successful request and focus on parsing.

---

**How Browser Automation APIs Handle JavaScript — Under the Hood**

The mechanics are worth understanding because they explain why certain features cost more credits.

When you add `render=true` to a ScraperAPI request (or enable JS rendering with any competing service), here's roughly what happens on their end:

1. A headless Chromium instance spins up (or is pulled from a warm pool)
2. Your target URL is loaded through a rotating residential proxy
3. The browser executes all JavaScript on the page — same as a real user's browser would
4. The final DOM state is captured after load events fire (or after a specified wait condition)
5. The rendered HTML is returned to you

That's a lot more compute than a simple HTTP fetch, which is why JavaScript rendering typically costs 5–10x more credits than a plain request. On ScraperAPI, `render=true` adds **+10 credits** per request. On ScrapingBee, it's **5x** the base cost (and it's on by default). On Firecrawl, rendering is included in the base request.

The key trade-off is always cost vs. necessity. If a page is static HTML, turning on rendering just burns credits for no benefit. If it's a React app, rendering is mandatory.

---

**Beyond Rendering: The Render Instruction Set and True Browser Automation**

Pure JavaScript rendering — load the page, wait for DOM, return HTML — handles most cases. But some workflows require actual browser interactions:

- Clicking a "Load More" button to paginate
- Scrolling to trigger lazy-loaded content
- Filling in a search field before the results appear
- Waiting for a specific element to appear before capturing the DOM

ScraperAPI addresses this with its **Render Instruction Set** — a set of declarative commands you include in your request to instruct the browser to execute specific actions during page rendering. You can define sequences like: wait for selector, click element, scroll to position, input text, then capture the HTML.

This is where the line between "scraping API" and "browser automation API" gets blurry in a useful way. For many workflows, the Render Instruction Set eliminates the need to run your own Playwright or Puppeteer instance — the automation happens inside ScraperAPI's cloud infrastructure, routing through their proxy pool, with all the anti-bot handling already in place.

A simplified example of what that looks like in practice:

python
import requests

payload = {
    "api_key": "YOUR_API_KEY",
    "url": "https://example.com/products",
    "render": "true",
    "instructions": [
        {"wait_for_selector": ".product-grid"},
        {"click": ".load-more-button"},
        {"wait": 2000},
        {"scroll_to": "bottom"}
    ]
}

response = requests.get("https://api.scraperapi.com/", params=payload)
print(response.text)


The credit cost for instruction-heavy requests scales with what you're doing — each browser action adds to request latency, which in turn affects how long the session runs on their infrastructure.

---

**ScraperAPI's Core Feature Set: What the API Actually Does**

Before diving into pricing math, here's a clear-eyed summary of what ScraperAPI actually provides:

**Proxy rotation** — 40 million+ IPs across 50+ countries, automatically rotated per request. You don't choose IPs; the system selects based on the target site's requirements.

**JavaScript rendering** — Headless Chromium, triggered via `render=true`. Handles SPAs, lazy-loaded content, and dynamic pages.

**Render Instruction Set** — Browser control for click, scroll, input, wait operations during rendering. Enables true browser automation without running your own browser infrastructure.

**CAPTCHA solving** — Basic CAPTCHA handling is built in. For aggressive protections (Cloudflare Turnstile, DataDome, PerimeterX), dedicated bypass credits apply.

**Geotargeting** — Country-level targeting via `country_code` parameter. Free on all plans for US and EU; global targeting (50+ countries) requires the Business plan.

**Structured Data Endpoints (SDEs)** — 18 pre-built endpoints across Amazon, Google, Walmart, eBay, and Redfin that return parsed JSON instead of raw HTML. Available on all plans.

**Async Scraper Service** — Submit large batches of URLs asynchronously and poll for results. Designed for millions of requests without blocking your application.

**DataPipeline** — A no-code solution for scheduling recurring scrapes with webhook delivery. Useful for teams that need regular data refreshes without writing cron jobs — though note the credit cost is higher (up to 6 credits per basic request versus 1 via the standard API).

**Session management** — The `session_number` parameter maintains the same IP across multiple requests to the same site, useful for sites with session-aware anti-bot systems.

---

**The Credit System: What Every User Needs to Know Before Spending a Dollar**

ScraperAPI charges on a credit model. The core principle: 1 request = 1 credit. In reality, that's almost never what you pay because domain type and feature flags multiply the base cost.

**Domain-based multipliers (applied automatically — you don't opt in):**

| Domain Category | Credits per Request |
| --- | --- |
| Normal websites (blogs, news, static HTML) | 1 |
| E-commerce (Amazon, eBay, Walmart) | 5 |
| Search engines (Google, Bing, all subdomains) | 25 |
| Social media (LinkedIn) | 30 |

**Parameter-based add-ons:**

| Parameter | Extra Credits |
| --- | --- |
| `render=true` (JavaScript rendering) | +10 |
| `screenshot=true` | +10 |
| `premium=true` (premium proxy pool) | +10 |
| `ultra_premium=true` | +30 |
| Anti-bot bypass (Cloudflare, DataDome, PerimeterX) | +10 each (auto-detected) |
| `premium=true` + `render=true` combined | +25 (not +20) |
| `ultra_premium=true` + `render=true` combined | +75 (not +40) |

That last point is the one that surprises people. Combining features costs more than adding them individually — it's non-linear stacking, not additive. A page behind Cloudflare with ultra-premium proxy and JavaScript rendering costs **75 credits per request**, not the 40 you'd expect from adding 30 + 10.

**Parameters that cost zero extra:** `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format`, `keep_headers`, `autoparse`.

The practical implication: always use the [Domain Multiplier tool](https://www.scraperapi.com/?fp_ref=coupons) in the dashboard or the API endpoint `https://api.scraperapi.com/account/urlcost` to check the actual credit cost for your target URL before running a batch.

---

**Full Plan Comparison: Every ScraperAPI Tier**

Here's the complete picture of every current plan, including annual pricing and what each tier actually unlocks:

| Plan | Monthly Price | Annual (per mo) | Credits/mo | Concurrent Threads | Geotargeting | Pay-As-You-Go | Get Started |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Free** | $0 | — | 1,000 | 5 | No | No | [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| **Hobby** | $49 | $44/mo | 100,000 | 20 | US & EU only | No | [Get Hobby Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Startup** | $149 | $134/mo | 1,000,000 | 50 | US & EU only | No | [Get Startup Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Business** | $299 | $269/mo | 3,000,000 | 100 | 50+ countries | No | [Get Business Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Scaling** | $475 | $427/mo | 5,000,000 | 200 | 50+ countries | Yes | [Get Scaling Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 5M+ | 200+ | 50+ countries | Yes | [Contact Sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

**Notes on the table:**
- Annual billing saves roughly 10% across all paid plans
- Ultra-premium proxies (`ultra_premium=true`) are only available on paid plans
- Pay-As-You-Go (PAYG) — where you continue using credits at a set rate after hitting your monthly limit — is only available on Scaling ($475/mo) and above. Hobby, Startup, and Business users who exhaust credits mid-cycle are cut off until the next billing period.
- Global geotargeting beyond US & EU requires the Business plan at minimum
- The 7-day trial (5,000 credits) is available when you first sign up, in addition to the 1,000/mo free tier

**What each plan actually gets you in real requests (with multipliers applied):**

| Plan | Standard (1 credit) | With JS Rendering (10 credits) | Amazon (5 credits) | Google SERP (25 credits) | Ultra-Premium + JS (75 credits) |
| --- | --- | --- | --- | --- | --- |
| Hobby ($49) | 100,000 pages | 10,000 pages | 20,000 pages | 4,000 pages | ~1,333 pages |
| Startup ($149) | 1,000,000 pages | 100,000 pages | 200,000 pages | 40,000 pages | ~13,333 pages |
| Business ($299) | 3,000,000 pages | 300,000 pages | 600,000 pages | 120,000 pages | ~40,000 pages |
| Scaling ($475) | 5,000,000 pages | 500,000 pages | 1,000,000 pages | 200,000 pages | ~66,666 pages |

The gap between "100,000 credits" and "1,333 usable pages" on the Hobby plan is real and it's the single most important number to internalize before choosing a plan.

---

**Where ScraperAPI Performs Well (and Where It Doesn't)**

Independent benchmark data from Scrapeway (April 2026) tells a story that's sharply split by site category:

**Strong performers:**
- Zillow: **100% success rate**, 10.5s avg
- Etsy: **99%**
- Amazon: **98%**, 6.5s avg
- LinkedIn: **95%** (but 30 credits/request)
- Walmart: **93%**

**Weak performers:**
- Instagram: **0%** — complete failure
- Twitter/X: **0%**
- Booking.com: **0%**
- Realtor.com: **12%**

The takeaway is practical: ScraperAPI is genuinely excellent for e-commerce data (Amazon, Walmart, eBay), SERP data (Google), and real estate (Zillow). If your workflow lives in those categories, the structured data endpoints alone are worth evaluating. If you need social media data or travel sites, something else is needed.

Overall average success rate across all tested sites sits at around **63%**, which is slightly above the industry average of ~58%. Response times average 5–7 seconds, which is competitive.

---

**ScraperAPI's Structured Data Endpoints: When Raw HTML Isn't Enough**

For specific high-value targets, ScraperAPI offers pre-built endpoints that skip HTML entirely and return structured JSON. The current set covers:

- **Amazon** (3 endpoints): Product details by ASIN, search results, competitor offers — 18+ fields including price, ratings, BSR, variants, images, seller info, and reviews. Supports 21 regional marketplaces.
- **Google** (5 endpoints): SERP (organic + ads + knowledge graph + PAA), Shopping, Maps, News, Jobs
- **Walmart** (4 endpoints): Product, Search, Category, Reviews
- **eBay** (2 endpoints): Product, Search
- **Redfin** (4 endpoints): Search, Agent Details, Rental Properties, For Sale

These endpoints are available on all plans, including Free. For teams scraping Amazon or Google at scale, they save significant development time — no custom parser to build or maintain. The trade-off is that Amazon still costs 5 credits per request (versus 1 for a plain request), so for very high volumes, teams with engineering bandwidth sometimes build their own parsers against the raw HTML to optimize cost.

👉 [Explore ScraperAPI's Structured Data Endpoints](https://www.scraperapi.com/?fp_ref=coupons)

---

**ScraperAPI vs. Key Browser Automation API Alternatives**

The browser automation API space is crowded. Here's an honest side-by-side of the options most developers end up evaluating:

| Feature | ScraperAPI | ScrapingBee | Browserless | Bright Data | Firecrawl |
| --- | --- | --- | --- | --- | --- |
| Core model | Proxy + rendering API | Proxy + rendering API | Direct browser control (Playwright/Puppeteer) | Enterprise proxy + scrapers | AI-native, LLM-ready output |
| JS rendering | Yes (`render=true`, +10 credits) | Yes (5x cost, default on) | Yes (native) | Yes (flat rate) | Yes (default, included) |
| Browser instruction control | Yes (Render Instruction Set) | Limited | Full Playwright/Puppeteer | Yes (Scraping Browser) | Yes (/interact endpoint) |
| Output format | Raw HTML (+ JSON via SDEs) | Raw HTML | Raw HTML | HTML / JSON | Markdown / JSON |
| Proxy network | 40M+ IPs, 50+ countries | Datacenter + premium | Managed | 150M+ residential | Managed |
| Structured endpoints | 18 endpoints (Amazon, Google, Walmart, eBay, Redfin) | None | None | 120+ pre-built scrapers | Schema-based any site |
| Entry price | $49/mo | $49/mo | $89/mo | Usage-based | $16/mo |
| Free tier | 1,000 credits/mo + 7-day trial | 1,000 credits | Limited | Trial only | 1,000 credits/mo |
| LLM/AI workflow support | No | No | No | Partial | Native |
| Official SDK | No (raw requests) | Yes (Python) | Yes | Yes | Yes (Python, Node) |

The choice between them comes down to what you're building:

**Choose ScraperAPI if:** You have existing scraper code that needs a proxy + rendering layer. You're primarily scraping Amazon, Google, Walmart, or Zillow. You want structured JSON without building a parser. You need the Render Instruction Set for browser automation without running your own browser infrastructure.

**Choose ScrapingBee if:** You're newer to scraping APIs and want an official Python SDK and cleaner docs. Note that JS rendering is on by default (5x credit cost) — disable it explicitly if you're scraping static pages.

**Choose Browserless if:** You need full Playwright or Puppeteer control and want to run your existing automation scripts in the cloud with managed proxies.

**Choose Bright Data if:** You're at enterprise scale, need a 150M+ IP residential network, need access to 120+ pre-built scrapers, or are operating in a compliance-heavy environment.

**Choose Firecrawl if:** Your pipeline feeds into an LLM or RAG system and you need Markdown output instead of raw HTML. The /interact endpoint also provides browser automation capabilities in natural language.

---

**Getting Started with ScraperAPI: From Zero to First Request**

The setup is genuinely fast. Here's the path:

**Step 1** — Sign up. The free plan gives you 1,000 credits per month, and the first 7 days include 5,000 extra trial credits. No credit card required for the free tier.

👉 [Start Your Free ScraperAPI Trial](https://www.scraperapi.com/?fp_ref=coupons)

**Step 2** — Grab your API key from the dashboard.

**Step 3** — Make your first request. The simplest version in Python:

python
import requests

response = requests.get(
    "https://api.scraperapi.com/",
    params={
        "api_key": "YOUR_API_KEY",
        "url": "https://example.com"
    }
)

print(response.text)


To enable JavaScript rendering, add `"render": "true"` to the params dict.

**Step 4** — Before running anything at scale, check the actual credit cost for your target URLs using the Domain Multiplier in the dashboard or the `urlcost` API endpoint. Don't skip this step — it's how you avoid credit surprises.

**Step 5** — Monitor your credit consumption daily during the first billing cycle. There are no proactive usage alerts in the dashboard; you have to check manually. Build that habit early.

---

**Honest Assessment: Who ScraperAPI Is and Isn't Built For**

ScraperAPI is a focused tool. It does one thing extremely well: it gives your existing scraper code a reliable, high-volume proxy and browser rendering layer. The 40M+ IP pool, the Render Instruction Set for browser automation, and the structured data endpoints for Amazon and Google are genuinely strong.

The credit multiplier system is the main complexity to navigate. The gap between "3,000,000 credits" (Business plan headline) and your actual request volume depends entirely on what you're scraping and which features you enable. For ultra-premium + JavaScript rendering on a Cloudflare-protected site, that's 75 credits per request — meaning the Business plan's $299/mo gets you about 40,000 pages, not 3 million. Run your own numbers before committing.

For teams with developer resources scraping well-supported targets at volume, ScraperAPI is a reasonable and competitively priced choice. For teams without engineering bandwidth who need data in a spreadsheet, a no-code scraping tool is probably a better fit.

The 7-day refund policy and no-questions-asked cancellation make it low-risk to test. Start with the free tier, validate your target sites in the API Playground, understand your effective per-request cost with multipliers, then decide.

👉 [Get Started with ScraperAPI for Free](https://www.scraperapi.com/?fp_ref=coupons)

---

**Frequently Asked Questions**

**What is a browser automation API?**
A browser automation API is a cloud-hosted service that runs a headless browser (typically Chromium) on your behalf, executes JavaScript on target pages, and returns the fully rendered HTML or structured data via a simple HTTP call. It replaces the need to run your own Playwright/Puppeteer infrastructure while providing proxy rotation and anti-bot handling.

**How does ScraperAPI's JavaScript rendering work?**
You add `render=true` to your request. ScraperAPI routes the URL through a headless Chromium instance behind their proxy pool, executes the page's JavaScript, and returns the final DOM state. This adds +10 credits per request. The Render Instruction Set lets you additionally control browser actions like click, scroll, and input during rendering.

**Does ScraperAPI support clicking and scrolling (not just rendering)?**
Yes — through the Render Instruction Set, you can define sequences of browser actions (click, scroll, wait for selector, input text) that run during page rendering. This makes it a functional browser automation API, not just a rendering proxy.

**Which ScraperAPI plan should I start with?**
Use the free tier first. Test your actual target URLs in the API Playground and check the credit cost per request with the Domain Multiplier. If your targets are normal websites with JavaScript rendering, 10 credits/request × your monthly volume will tell you which paid plan makes sense. Most small teams start with Hobby ($49/mo); teams doing meaningful e-commerce or SERP scraping typically land on Startup ($149/mo) or Business ($299/mo).

**Can ScraperAPI scrape any website?**
No. ScraperAPI forbids scraping data behind login walls. It also shows 0% success rates on Instagram, Twitter/X, and Booking.com in independent benchmarks. It performs best on e-commerce sites, Google SERPs, and real estate platforms. Always test your specific targets before committing.

**Is the free plan actually useful?**
For evaluation: yes. 1,000 credits/month is enough to test your target URLs and understand credit consumption. Combined with the 5,000-credit 7-day trial, you have a realistic window to validate whether ScraperAPI works for your use case before paying anything.
