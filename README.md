# Yelp API alternative: how do you get full review text, what do 1,000 pages cost, and which ScraperAPI plan fits? (All 7 plans compared)

The official Yelp API returns three short review excerpts per business, which is enough to display a star rating and not much else. Anyone building review analysis, reputation monitoring, or a local lead list runs into that within the first hour of work.

Two kinds of products sit on the other side of it. Prebuilt Yelp scrapers give you finished JSON fields, chosen by whoever built the scraper. General-purpose scraping APIs give you the fully rendered page and let you decide what counts as a field. ScraperAPI is the second kind: a general scraping API that works on any URL, Yelp included, with no Yelp developer account anywhere in the flow. New accounts get 5,000 credits over 7 days without a credit card, which is enough to test its output against your own parser before paying.

This page covers the plans first, then the field-level comparison against Yelp's official API, then the part most comparison pages skip: what a Yelp page actually costs in credits once JavaScript rendering enters the picture, and why the answer can move by a factor of ten.

> **Quick pick:** ScraperAPI makes sense if you want Yelp and other sources on one bill, or you need fields no off-the-shelf actor exposes. One-off research fits inside the free trial. Continuous pipelines should start at Scaling, the first plan with pay-as-you-go overage, at **$475/month**, or **$427.50/month** billed annually. 👉 [Compare ScraperAPI plans and start the 5,000-credit trial](https://www.scraperapi.com/?fp_ref=coupons)

## Every ScraperAPI plan, and which prices are actually published

There are seven paid tiers plus a free plan. Credit allowances and thread counts are published for all of them. Monthly prices were confirmed only for Scaling and Professional at the time of writing, so the other rows say so instead of carrying a number I could not verify.

| Plan | Credits / month | Concurrent threads | Price (monthly / annual, per month) | Start |
| --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | $0 | [Sign up free](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | 100,000 | 20 | Not confirmed at time of writing | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | Not confirmed at time of writing | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Not confirmed at time of writing | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | $475 / $427.50 | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | $975 / $877.50 | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Not confirmed at time of writing | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom quote | [Check price](https://www.scraperapi.com/?fp_ref=coupons) |

Details that apply across the table:

- **Geotargeting:** Hobby and Startup cover US and EU only. Business and above go country-level.
- **Annual billing:** 10% off on every tier except Enterprise, which is quoted individually. The annual figures above already include it.
- **Overage:** pay-as-you-go past your monthly allowance starts at Scaling and is available on Advanced and Enterprise too. On Hobby, Startup and Business, running out of credits pauses scraping until the cycle renews, and auto-upgrade to the next tier is something you switch on rather than a default.
- **Shared features:** every paid tier includes JavaScript rendering, premium proxies, CAPTCHA and anti-bot handling, unlimited bandwidth, and a 99.9% uptime commitment. Rendering and premium proxies are inputs that cost extra credits per request, not free switches, which matters for the math further down.
- **Concurrency:** exceeding your thread limit returns a 429 rather than an extra charge.

The free plan's 1,000 credits with 5 concurrent connections is separate from the 7-day trial, so you can burn through the trial and still keep a small monthly allowance for testing.

## Why people leave the official API in the first place

Yelp's official offering, now called the **Places API (formerly Fusion)**, still exists and still requires a developer app plus a private API key. Commercial use is sold through Yelp Data Licensing. So the official path hasn't disappeared; it has a gate on it.

The review limit is the part that pushes people elsewhere. Three short excerpts per business will not support sentiment analysis across a market, a competitor review timeline, or an agency deliverable. An excerpt is the shape of the official review object rather than a full review with the rest hidden behind a paywall, which is why workarounds that claim to "unlock" Fusion reviews are describing something else entirely. If you need what only a partner gets, stay on the official API.

What a prebuilt scraper returns instead looks like this. One Apify actor built specifically as a review alternative returns full review text, star rating, date, language, useful/funny/cool votes, photos, owner replies and Elite reviewer stats, accepts a business URL, alias or name as input, and needs no Yelp key or approval. A second actor focuses on business records: more than 25 structured fields including business ID, alias, categories, coordinates, price range, both rounded and unrounded ratings, review count, machine-readable hours, attributes such as "Wi-Fi: Free" or "Outdoor Seating: Yes", health inspection score, and a menu link. Its sample record for The Old Fashioned in Madison, WI shows a 4.0 average (4.1 unrounded) across 2,529 reviews at `$$`. It also supports query × location combinations in a single run, direct lookup by URL, alias or ID, automatic deduplication, and export to JSON, CSV, Excel, XML or HTML.

Note that the same page advertises "no rate limits". That's vendor copy, and unverified. Even with a scraping API absorbing the proxy work, request pacing still affects your CAPTCHA exposure.

## What ScraperAPI does with a Yelp URL, and what you build yourself

ScraperAPI routes each request through a rotating pool of residential and datacenter IPs, renders JavaScript when you ask it to, retries automatically when a request is blocked or errors out, and can return the page as clean Markdown instead of raw HTML. That last option is useful if you're feeding Yelp pages into an LLM for summarization, since Markdown is easier to parse than a wall of divs.

The company has also published its own walkthrough for scraping Yelp search results, business details and reviews, including pagination and dynamically loaded content with CSS selectors. It's a signal the approach holds up beyond theory.

The extraction work stays with you, and it is real work:

1. Send the Yelp URL through the API with your key, with rendering enabled for fields that load after the initial HTML.
2. Parse the returned HTML with BeautifulSoup in Python or Cheerio in Node for names, ratings, addresses and links to individual business pages.
3. Follow those links into business pages for phone numbers, service options and full review threads.
4. Handle pagination twice over, on search results and on each business page's review list.
5. Throttle and randomize timing between calls.

Fields you can pull this way include business name, category, Yelp business ID, star rating, total review count, address, phone, website, price range, service options like delivery or outdoor seating, review text with reviewer name, rating and date, photos, and menu items on restaurant pages.

That list is roughly what a prebuilt actor gives you. The difference is who maintains the parser when Yelp changes its markup, and whether you can add a field nobody thought to expose. Those are the two questions that should decide this for you.

## What 1,000 Yelp pages actually cost in credits

ScraperAPI bills in credits, and the per-request cost depends on the target domain and the parameters you send.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (including subdomains) | 25 |
| LinkedIn | 30 |
| Bot-protection bypass (Cloudflare, Turnstile, DataDome, PerimeterX/Human, Akamai) | +10 |
| `render=true` (JavaScript rendering) | +10 |
| `premium=true` (premium proxies) | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` | 30 (paid plans) |
| `ultra_premium=true` + `render=true` | 75 (paid plans) |

**Yelp is not named on the published per-domain list.** That list calls out Amazon/Walmart/eBay, Google/Bing and LinkedIn by name and leaves everything else in the standard category. So a plain Yelp fetch should bill as 1 credit, with the caveat that any protection bypass or rendering flag stacks on top. Before you commit to a large job, run one Yelp URL through the Domain Cost Estimator in the dashboard, and watch the `sa-credit-cost` response header afterwards. You can also set `max_cost` per request so a surprise on one URL can't blow up a batch.

With that assumption stated, the arithmetic at published prices:

- **Scaling:** $475 for 5,000,000 credits works out to $0.000095 per credit, so **1,000 one-credit requests cost about $0.095**. Billed annually at $427.50/month, the same 1,000 requests cost roughly **$0.086**.
- **Professional:** $975 for 10,500,000 credits is about **$0.093 per 1,000** one-credit requests, or roughly **$0.084** on annual billing.
- **With `render=true`:** a page becomes 11 credits, so 1,000 pages run about **$1.05** at Scaling and about **$1.02** at Professional. This is the realistic scenario for review counts and photos that load after the initial HTML.
- **With rendering plus a bot-protection bypass:** 21 credits per page puts 1,000 pages near **$2.00** at Scaling.

That 1-credit figure is the one to be careful with. The difference between $0.095 and $1.05 per thousand is the difference between a rounding error and a line item, and it hinges on whether your target needs rendering. Test one URL before sizing a plan.

Two billing mechanics also break budgets that look fine on paper. **Unused credits do not roll over** between cycles; your balance resets at renewal, so buying a tier above your actual monthly usage is money donated. And **pay-as-you-go overage only exists from Scaling upward**, which is why the cheap-looking entry tiers can stall a pipeline mid-run if your volume is variable. If you expect uneven monthly volume rather than steady volume, that alone argues for Scaling over a bigger allowance on a lower tier.

## When buying parsed Yelp fields is the better deal

Purpose-built Yelp data APIs sell finished JSON, and their pricing is set per call rather than per credit.

**API.SH** exposes four Yelp endpoints (search, business, reviews, menu), all at 1 credit per call. Credits run $1 per 1,000 on a monthly plan, or $2 per 1,000 pay-as-you-go; the Starter plan is $50/month for 50,000 credits, top-ups start at $15 and don't expire, failed calls aren't billed, and every account gets 1,000 free credits a month. No Yelp app or key required. Reviews come back with rating, text, date, author details, photos, reactions, the owner's reply and a rating breakdown, filterable by rating or language.

**Apify** runs the two actors described earlier as jobs rather than single URL calls, which suits scheduled scrapes. Neither no-key requirement nor per-run pricing could be verified to a number for this article, so treat cost as something to check on the actor page itself.

**fetcher.sh** offers a Yelp endpoint that returns public review text as a plain GET without Fusion partner status. Its published per-call price could not be verified either.

**Outscraper** fits when the deliverable is a spreadsheet of an entire city; **SerpAPI's Yelp endpoint** fits if SerpAPI is already on your bill; **Bright Data** fits enterprise bulk under contract.

Here's the honest comparison. At roughly $1 per 1,000 rendered Yelp pages against $1 per 1,000 parsed Yelp calls, ScraperAPI is not cheaper for a Yelp-only job at small volume. The economics flip when a Yelp page bills as a plain 1-credit request (about $0.095 per 1,000), when your volume is large enough for the higher tiers to matter, or when the same subscription also covers Google Maps, directories and company sites you'd otherwise buy separately. What you're really choosing between is a parser someone else maintains and a parser you own.

## FAQ

**Is the Yelp API still available?**
Yes. It's now the Places API (formerly Fusion), documented on Yelp's developer site. You need to create an app and use a private API key, and commercial access is sold through Yelp Data Licensing.

**Is the three-review limit real?**
Yes, and it's a property of the official review object, not a setting you can raise. Plan around it or use a different data source.

**Do I need a Yelp account or API key to scrape Yelp through ScraperAPI?**
No. ScraperAPI fetches the public page at whatever URL you give it; there's no Yelp developer app, key or OAuth step in that path. Login-gated content is a different matter, and you shouldn't be pulling it.

**Is the free allowance enough to test properly?**
The trial gives 5,000 credits over 7 days with no card, and the free plan adds 1,000 credits a month at 5 concurrent connections. On plain 1-credit pages that's thousands of fetches; with rendering on, divide by eleven. Enough for a few hundred business pages plus their review threads in most cases.

**Will I get blocked?**
No one can promise that. ScraperAPI handles proxy rotation, CAPTCHA and anti-bot bypass, and automatic retries when a request fails. You still control request rate, and firing thousands of requests in a short window raises CAPTCHA risk regardless of whose proxies you're behind.

## Which option fits your situation

- **A few dozen to a few hundred reviews for qualitative analysis:** a prebuilt actor is less work. You get finished fields without writing a parser, and the volume is too small for per-credit savings to matter.
- **Yelp plus Google Maps plus company sites, custom fields, or steady volume:** ScraperAPI's credit model is more controllable, and the 1-credit case is dramatically cheaper than any per-call Yelp API.
- **Already on Fusion and only needs basic business data:** don't switch. Three excerpts cover address, rating and hours fine.
- **Enterprise bulk with a contract and a procurement process:** Bright Data's model fits that shape better than a self-serve plan.
- **Variable monthly volume:** weight the overage rule heavily. Scaling is the first tier where a traffic spike doesn't stall the pipeline.

Two things to do before you pay for anything. Run a real Yelp business URL through the Domain Cost Estimator and check whether it bills as 1 credit or 11, because that single number decides which tier you need. Then size the plan to your actual monthly volume rather than your peak, since credits reset at renewal and the leftover doesn't follow you into next month.

If the estimator comes back at 1 credit per page and you're pulling more than a few thousand pages a month, the entry tiers won't hold you. 👉 [Start the 7-day trial with 5,000 credits, no card required](https://www.scraperapi.com/?fp_ref=coupons)
