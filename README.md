# Batch Web Scraping API: What It Actually Is, How ScraperAPI Makes It Work, and Which Plan Fits Your Use Case — Plus a Brutally Honest Look at Credits, Concurrency, and Real-World Success Rates

If you've ever tried to scrape a list of 10,000 product pages one request at a time, you already know what "agonizing" feels like. You watch a progress bar creep across your terminal. You check the clock. You go make coffee. You come back. Still 7,843 pages to go.

That's the problem a **batch web scraping API** is designed to solve — and it's the reason so many developers eventually land on a tool like ScraperAPI. Instead of firing off requests sequentially and waiting for each one to come back before sending the next, a batch API lets you dump an entire list of URLs into a single call and let the service handle the parallel processing, proxy rotation, retry logic, and CAPTCHA solving in the background.

Sounds great. And honestly, when it works well, it is great. But there's a lot of nuance that most articles gloss over — the credit math, the concurrency limits per plan, the sites where success rates drop to zero, the features that cost way more than you expect when you combine them. This article covers all of it.

---

## **What "Batch Web Scraping API" Actually Means (And Why It Matters)**

Let's start simple, because this gets confused a lot.

A standard scraping API call is synchronous: you send one URL, the service scrapes it, and you get HTML back in the same response. Simple, predictable, works fine at low volume. The moment your list grows — product catalogs, SERP result pages, real estate listings, competitor pricing tables — the sequential approach hits a wall.

A **batch web scraping API** does something different. It accepts a collection of URLs in a single request, runs them in parallel across distributed infrastructure, and returns the results either via a status-check endpoint or delivered directly to a webhook your system is listening on. You submit the work, walk away, and get called back when the data is ready.

The benefits are real:

- **No more waiting on each request** before the next one can start
- **Massively reduced wall-clock time** for large-scale jobs
- **Webhook delivery** means your pipeline gets the data pushed to it automatically — no polling loop needed
- **Retry logic handled server-side** — if a page fails, the service keeps retrying so you don't have to babysit it
- **Cleaner code** — your application logic separates cleanly from the scraping infrastructure

The practical result: what used to take hours of sequential scraping can be compressed into minutes of parallel processing.

---

## **ScraperAPI's Async Batch Endpoint: How It Actually Works**

[ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons) built its batch processing on top of its Async API, and the design is worth understanding before you assume it works like every other service.

The endpoint is `https://async.scraperapi.com/batchjobs`. You POST to it with your API key and an array of URLs. ScraperAPI spins up a background job for each URL, retries until each one returns a successful response (or until 24 hours has elapsed), and stores the results for up to 72 hours. You retrieve them either by polling the individual status URLs returned in the initial response, or by having the service call your webhook when each job completes.

Here's the core of what you get back after submitting a batch:

json
[
  {
    "id": "04888c53-e322-4976-969d-8f8b39f016da",
    "attempts": 0,
    "status": "running",
    "statusUrl": "https://async.scraperapi.com/jobs/04888c53-e322-4976-969d-8f8b39f016da",
    "url": "https://wikipedia.org/wiki/Cowboy_boot"
  },
  {
    "id": "946ada9c-2f57-490b-900a-fa14193ae029",
    "attempts": 0,
    "status": "running",
    "statusUrl": "https://async.scraperapi.com/jobs/946ada9c-2f57-490b-900a-fa14193ae029",
    "url": "https://wikipedia.org/wiki/Web_scraping"
  }
]


A few hard limits worth knowing upfront:

- **50,000 URLs per batch job** — that's the maximum in a single submission
- **24-hour job window** — jobs run until they succeed or time out at 24 hours
- **72-hour data retention** — results are available for retrieval up to 72 hours; after that, they're deleted
- **Cost control** — you can set a `max_cost` parameter to cap how many credits a batch job can consume; if the job would exceed your cap, it returns a 403 error instead of running up your bill

The webhook option deserves more attention than it usually gets. Instead of writing a polling loop that hammers the status endpoint every few seconds, you give ScraperAPI a webhook URL at job submission time. When each page finishes scraping, ScraperAPI POSTs the result directly to your endpoint. For large batches running asynchronously, this is dramatically cleaner — your server wakes up when the data arrives, rather than constantly asking "are we there yet?"

> 👉 [Start scraping with ScraperAPI — free trial included, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

## **The Credit System: What "100,000 Credits" Actually Gets You**

Here's the part that trips up almost everyone who's new to ScraperAPI — or to credit-based scraping APIs in general. When you look at the Hobby plan and see "100,000 API credits," you might picture scraping 100,000 pages. That number is almost certainly wrong for most real-world use cases.

Credits are not a flat 1:1 ratio with requests. The domain you're scraping and the features you enable both determine how many credits each request consumes. Here's the base cost by domain type:

| Domain Type | Credits per Request | Examples |
|---|---|---|
| Standard websites | 1 | Blogs, news, documentation |
| E-commerce | 5 | Amazon, eBay, Walmart |
| Search engines (SERP) | 25 | Google, Bing |
| Social media | 30 | LinkedIn |

And then feature flags stack on top of those base costs:

| Parameter | Extra Credits | Note |
|---|---|---|
| `render=true` (JavaScript) | +10 | All plans |
| `premium=true` (premium proxies) | +10 | All plans |
| `screenshot=true` | +10 | All plans |
| `ultra_premium=true` | +30 | Paid plans only |
| `premium=true` + `render=true` combined | **+25** (not +20) | Non-linear pricing |
| `ultra_premium=true` + `render=true` | **+75** (not +40) | Non-linear pricing |
| Anti-bot bypass (Cloudflare, DataDome, etc.) | +10 per bypass | Applied automatically |

That combined ultra-premium + JavaScript rendering at 75 credits per request means your "100,000 credit" Hobby plan effectively gives you **1,333 actual page fetches** in the worst case — not 100,000.

Some parameters don't add any cost: `country_code`, `session_number`, `device_type`, `wait_for_selector`, `output_format`, `keep_headers`, `autoparse`. These are essentially free.

One more thing worth knowing: **ScraperAPI only charges for successful requests** (HTTP 200 and 404 responses). If your request fails and returns a 500 error, you're not charged. That's a meaningful consumer protection that not all scraping APIs offer.

Credits do not roll over. Whatever you don't use by the end of your billing cycle is gone.

---

## **All ScraperAPI Plans: Full Breakdown**

Below is every current plan, with pricing for both monthly and annual billing (annual saves roughly 10%).

| Plan | Monthly Price | Annual (per mo) | API Credits | Concurrent Threads | Geotargeting | PAYG When Credits Exhausted |
|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000 | 5 | None | No |
| **Hobby** | $49 | $44.10 | 100,000 | 20 | US & EU only | No |
| **Startup** | $149 | $134.10 | 1,000,000 | 50 | US & EU only | No |
| **Business** | $299 | $269.10 | 3,000,000 | 100 | 50+ countries | No |
| **Scaling** | $475 | $427.50 | 5,000,000 | 200 | 50+ countries | ✅ Yes |
| **Professional** | $975 | ~$877 | Custom high volume | 200+ | 50+ countries | ✅ Yes |
| **Advanced** | $1,975 | Custom | Custom high volume | Custom | 50+ countries | ✅ Yes |
| **Enterprise** | Custom | Custom | 5,000,000+ | Custom (200+) | 50+ countries | ✅ Yes |

A few observations on that table:

**Geotargeting** beyond US and EU requires the Business plan at minimum. If your batch scraping jobs need to pull data that varies by country — localized pricing, regional search results, country-specific product availability — you need to budget for Business or above.

**Pay-As-You-Go** (PAYG) is only available on Scaling and higher tiers. On Hobby, Startup, and Business, if you exhaust your credits mid-cycle, you're cut off until renewal. No PAYG top-up. Your only option is to upgrade to the next tier.

**Concurrent threads** directly determine your batch processing speed. The Business plan's 100 threads and the Scaling plan's 200 threads are significant differentiators. In ScraperAPI's own internal testing, bumping concurrent threads from 100 to 500 (Enterprise-level) cut processing time on a 1,000-URL batch from ~100 seconds to ~23 seconds — nearly 4x faster.

**The 7-day free trial** gives you 5,000 credits to test against your actual target sites before committing to a paid plan. No credit card required. This is legitimately useful — you can run a representative sample of your real URLs, see the actual credit consumption with your specific parameters enabled, and extrapolate what a monthly budget actually looks like for your use case.

> 👉 [Claim your free ScraperAPI trial — 5,000 credits, no credit card](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Real-World Performance: Where ScraperAPI's Batch API Shines (and Where It Doesn't)**

No scraping infrastructure performs equally well across every target. Independent benchmarks from early 2026 tell a pretty sharp story about where ScraperAPI's strengths and weaknesses lie.

**Where it performs well:**

| Target Site | Success Rate | Avg Speed |
|---|---|---|
| Zillow | ~100% | ~10.5s |
| Amazon | ~98% | ~6.5s |
| Etsy | ~99% | ~4.8s |
| Walmart | ~93% | ~11.4s |
| LinkedIn | ~95% | ~17.8s |

For batch scraping of e-commerce product pages and real estate listings — arguably the most common production batch use case — ScraperAPI is genuinely reliable. The structured data endpoints for Amazon in particular are a strong point: submit a batch of ASINs, get back parsed JSON with 18+ fields (price, reviews, BSR, variants, images, seller info), no HTML parsing needed.

**Where it falls short:**

| Target Site | Success Rate |
|---|---|
| Instagram | 0% |
| Twitter / X | 0% |
| Booking.com | 0% |
| Realtor.com | ~12% |

Social platforms are largely a dead zone. If your batch scraping project involves Instagram, Twitter, or most travel booking sites, ScraperAPI isn't going to help you. You'll know this from your free trial — test your actual targets before paying.

There's also a **10-minute forced cache** on some harder targets. If you're batch scraping time-sensitive data (pricing that changes by the hour, live inventory), be aware that results from difficult sites may be up to 10 minutes stale. For most catalog-style scraping this is fine; for real-time competitive pricing it's a genuine constraint.

---

## **Structured Data Endpoints: The Batch API's Best Friend for E-Commerce**

For batch web scraping on supported domains, ScraperAPI's structured data endpoints are worth serious attention. Instead of getting raw HTML that your code then needs to parse, you get back clean JSON objects already organized into fields.

Current structured data coverage includes:

- **Amazon** — Product details (by ASIN), search results, competitor offers. Returns price, ratings, BSR, reviews, images, variant info, seller details. Supports 21 regional marketplaces.
- **Google** — SERP results (organic, knowledge graph, videos, related questions, pagination), Shopping, Maps, News, Jobs
- **Walmart** — Product, Search, Category, Reviews
- **eBay** — Product, Search
- **Redfin** — Search, Agent Details, Rental Properties, For Sale listings

These endpoints are available on all plans, including the free tier. For batch jobs, this means you can submit 500 ASINs, get 500 structured JSON responses, and pipe them directly into your database — without writing a single HTML parser or maintaining it when Amazon updates its page layout.

The credit cost is domain-rate (5 credits per Amazon request), but the time saved on parser development and maintenance is real. A team spending two days building and debugging an Amazon HTML parser is spending developer hours that could be deployed elsewhere.

---

## **Concurrent Threads vs. Async Batch: Choosing Your Architecture**

There are two primary ways to run large-scale batch scraping with ScraperAPI, and they're designed for different situations.

**Concurrent threads via the Sync API**: You send multiple requests simultaneously using a thread pool in your own code. This is fast and gives you tight control — you can watch responses come in and react to failures immediately. Best when you need results quickly and can handle the connection management on your end.

python
from concurrent.futures import ThreadPoolExecutor
import requests

API_KEY = 'your_key_here'

def scrape_url(url):
    params = {'api_key': API_KEY, 'url': url}
    response = requests.get('http://api.scraperapi.com/', params=params)
    return response.text

urls = ['https://example.com/page-1', 'https://example.com/page-2', ...]

with ThreadPoolExecutor(max_workers=100) as executor:
    results = list(executor.map(scrape_url, urls))


**Async batch via `/batchjobs`**: You submit all URLs in a single POST and let ScraperAPI handle the parallel processing server-side. This is better when success rate matters more than raw speed — the Async API keeps retrying each job for up to 24 hours, making it ideal for heavily protected pages or scrapers that run overnight. Webhook delivery removes the need to poll.

python
import requests

r = requests.post(
    url='https://async.scraperapi.com/batchjobs',
    json={
        'apiKey': 'your_key_here',
        'urls': [
            'https://example.com/product/1',
            'https://example.com/product/2',
        ]
    }
)
# r.json() returns a list of job objects with statusUrls


**Decision rule of thumb**: If you need results in seconds and your targets are fairly cooperative, use concurrent threads against the Sync API. If you're hammering heavily protected sites overnight and care more about eventually getting every page than getting them all fast, use the Async batch endpoint.

> 👉 [Start your free ScraperAPI trial and test both approaches](https://www.scraperapi.com/?fp_ref=coupons)

---

## **What Real Users Actually Say**

Across Trustpilot (4.5/5 from 42 reviews), G2 (4.4/5), and Capterra (4.6/5, Ease of Use 4.9/5), the consistent pattern in user feedback follows a fairly predictable split.

**Positive signals cluster around:**
- Initial setup speed — "You can start scraping in minutes" appears repeatedly
- Documentation quality — the ScraperAPI docs are genuinely thorough compared to many competitors
- Reliable performance on the supported mainstream targets (Amazon, Google, Zillow)
- Responsive customer support team

**Negative signals cluster around:**
- Credit math surprises — particularly the non-linear cost of combining premium proxy + JS rendering
- No usage alerts — there's no email notification when you're burning through credits fast; you have to check the dashboard manually
- Performance drops on more obscure or aggressively protected targets
- The DataPipeline feature costs up to 6 credits per request versus 1 credit via the standard API — a difference that hits hard if you set up no-code pipelines without reading the fine print

One real-world complaint that comes up on Reddit involves Amazon pricing: a user was quoted a price for 60 million credits at 1 credit per request, only to discover post-payment that Amazon applies a 5× multiplier — turning their 60M request expectation into an effective 12M. The credit cost documentation exists in the docs, but it's not prominently surfaced during the purchase flow.

The fix: use ScraperAPI's Domain Multiplier tool in your dashboard before making any purchasing decisions. Plug in your actual target URLs and see the real credit cost per request before you commit.

---

## **Tips for Getting the Most from ScraperAPI Batch Jobs**

A few practices that make a meaningful difference once you're running production batch workloads:

**Test with the free tier first.** The 7-day trial gives you 5,000 credits and no credit card requirement. Use all of them on your actual target URLs with your actual parameters enabled. Document exactly how many credits each request type consumes. Then extrapolate your real monthly credit needs before picking a paid plan.

**Know which parameters add cost (and which don't).** You don't need `render=true` for pages that don't actually use JavaScript to render their content. Disabling it on pages that serve static HTML cuts your effective per-request cost by 10 credits. Same with `premium=true` — only enable premium proxies on pages that actually require residential IPs to avoid blocks.

**Use `max_cost` on batch jobs.** The async batch endpoint supports a `max_cost` parameter that caps total credits consumed by a job. Set this to avoid surprise bill spikes when a batch job hits unexpectedly expensive targets.

**Check the Domain Multiplier before scraping at scale.** Available in your dashboard, this tool tells you the credit cost for any URL before you run it. Use it before queuing a batch of 50,000 URLs to make sure your credit math is right.

**Set a manual dashboard check cadence.** ScraperAPI does not send proactive usage alerts. Until you've built strong intuition for your credit burn rate, check the dashboard daily. Particularly in the first few weeks on a new plan.

**Use structured data endpoints for supported domains.** If you're batch scraping Amazon or Google at any meaningful scale, the structured data endpoints save real engineering time versus writing and maintaining HTML parsers. Factor in your team's developer hours when comparing the credit cost premium.

**Split very large batches into chunks of 50,000 or fewer.** The documented limit per batch job is 50,000 URLs. For larger lists, split them and submit sequentially. ScraperAPI recommends this approach explicitly to maintain stability and processing efficiency.

---

## **Batch Web Scraping API Use Cases That Fit ScraperAPI Well**

To put this all in concrete terms, here are the workloads where batch scraping through ScraperAPI tends to produce solid results:

**E-commerce price monitoring**: Submit a batch of competitor product ASINs nightly, retrieve structured JSON with current prices, inventory status, and seller offers. ScraperAPI's Amazon structured data endpoint handles this well at 98% success rates.

**SERP rank tracking**: Submit a batch of keyword queries to Google, get back organic result positions, featured snippets, and People Also Ask data. Costs 25 credits per query, so plan your keyword list accordingly.

**Real estate data collection**: Batch scrape Zillow or Redfin listing pages. ScraperAPI's Zillow success rate sits near 100% in independent benchmarks, and the Redfin structured data endpoints return parsed listing data without HTML parsing.

**Content aggregation and monitoring**: Large-scale scraping of news sites, blogs, or documentation pages for content analysis, NLP training data, or brand monitoring. Standard pages at 1 credit each, this is where your headline credit count most closely matches your actual request count.

**Lead generation at scale**: Product-level detail pages from B2B directories, professional listings, or industry databases. Combine with concurrent thread pooling for maximum throughput.

> 👉 [Try ScraperAPI's batch API free — no credit card, 5,000 credits to start](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Which Plan Should You Actually Choose?**

Here's a quick framework based on what you've just read:

**Free plan** — Genuinely useful only for testing whether ScraperAPI works for your specific targets. 1,000 credits and 5 concurrent threads isn't enough for any real production workload. But combined with the 7-day trial's 5,000 credits, it's enough to validate your use case.

**Hobby ($49/mo)** — Works for solo developers running small batch jobs against simple targets. 100,000 credits at 1 credit per request = 100,000 pages. But if you're scraping Amazon (5 credits each) or using JavaScript rendering (10 more credits), your effective capacity shrinks fast. 20 concurrent threads is the ceiling here.

**Startup ($149/mo)** — The first tier that makes practical sense for regular data collection workflows. 1 million credits, 50 threads, and US/EU proxy coverage. Still no full geotargeting beyond US and EU.

**Business ($299/mo)** — The sweet spot for most production teams. 3 million credits, 100 concurrent threads, and access to 50+ country geotargeting. PAYG is still not available here, so you can get cut off if you blow through your credits.

**Scaling ($475/mo)** — Where serious batch processing starts to make financial sense. 5 million credits, 200 concurrent threads, and crucially: Pay-As-You-Go kicks in when you exceed your credit limit instead of cutting you off. Annual billing drops this to about $427.50/month.

**Professional / Advanced / Enterprise** — Custom and high-volume tiers with PAYG included, custom concurrency limits, and negotiated pricing for enterprise-scale batch workloads.

| Plan | Best For | Monthly |  Link |
|---|---|---|---|
| Free | Testing only | $0 |  [Start Free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | Side projects, small batches | $49/mo |  [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | Regular data workflows | $149/mo |  [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | Production teams, geotargeting | $299/mo |  [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | High-volume batch + PAYG | $475/mo |  [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | Very high volume + PAYG | $975/mo |  [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | Large-scale enterprise + PAYG | $1,975/mo |  [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | Custom, unlimited scale | Custom |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

---

## **The Bottom Line**

A **batch web scraping API** is genuinely useful infrastructure — and ScraperAPI's Async batch endpoint is one of the more capable implementations you'll find. The 50,000 URL per batch limit, webhook delivery, 24-hour retry window, and `max_cost` control cover the core requirements of a serious batch scraping pipeline.

The things worth remembering before you commit:

The credit math is the most important thing to understand before choosing a plan. Run your actual target URLs through the Domain Multiplier, apply the relevant feature flag multipliers, and calculate your real monthly credit consumption before picking a tier. The numbers on the pricing page are technically accurate — they're just the most optimistic version of reality.

ScraperAPI performs well on e-commerce and real estate. It doesn't work at all on social platforms. Know your target sites before you pay.

The free trial exists for a reason. Use all 5,000 trial credits deliberately, on your actual production URLs, with your actual parameters. That data is worth more than any pricing guide you'll read.

> 👉 [Start your ScraperAPI free trial — 5,000 credits, no credit card needed](https://www.scraperapi.com/?fp_ref=coupons)

If your batch job list is real, your target sites are supported, and you run the credit math correctly, ScraperAPI will probably do exactly what you need it to do. That's honestly all any scraping infrastructure needs to deliver.
