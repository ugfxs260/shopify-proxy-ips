# Shopify proxies: How to Pick IPs That Survive Store Rate Limits, Multi-Store Logins and Geo Tests

"Shopify proxies" is one of those search terms that means four different things depending on who types it. A reseller wants checkout throughput on a Queue-it drop. Someone running nine dropshipping stores wants each store's admin login to look like a different person in a different city. A pricing analyst wants product and price data out of a few thousand storefronts without babysitting retries. A merchant in Berlin wants to see what their shop actually looks like to a shopper in Ohio.

Same keyword, four different jobs, four different failure modes. The proxy that fixes one will waste money on another. So before getting to plans and prices, it's worth being precise about what Shopify does to your traffic and which model matches which workload.

## What Shopify actually does to your IP

Shopify isn't hostile to requests. It's metered, and the meter is attached to your IP address, not to you as a person.

**Rate limiting is per IP, per store.** Public storefront pages (collections, product pages) run on bucket-style limiting. Community write-ups of the behaviour put the refill rate at roughly 2–4 requests per second, and the critical detail is that the counter is scoped to one IP and one store. Using one IP to hammer Store A and Store B at the same time burns two separate buckets, so a pool of 100 IPs sending two requests per second each gives you something in the neighbourhood of a couple hundred requests per second of headroom — far better than ten IPs each sprinting at 20 requests per second until they hit `429` and start the retry spiral.

**You get told when you're over.** Shopify returns `429 Too Many Requests` with a `Retry-After` header. That is not a ban, it's a queue ticket. Handling it correctly is cheaper than buying more IPs.

**Merchants bury the front end in Cloudflare.** A lot of stores run Cloudflare or a bot-protection app in front of Shopify, and Shopify's own CDN is Cloudflare-backed, so a `cf-ray` header proves nothing on its own. What it does mean practically is that datacenter ranges get filtered at the edge long before application-level logic ever sees you.

**Drops run Queue-it.** On limited releases, the queue system weighs IP reputation and network type alongside browser fingerprint and click behaviour. Datacenter IPs are scored low by default; residential and mobile IPs sit at the top of that hierarchy. The proxy tables in sniping communities are extremely blunt about this — the difference between datacenter and residential success rates on those drops isn't a few percentage points.

**Storefronts are light.** A Shopify page is a few hundred KB of HTML, and many stores expose product and collection data as JSON by appending `.json` to the URL — an officially supported path that returns a fraction of the bytes. Bandwidth is almost never your bottleneck with Shopify. IP reputation and request pacing are.

> Scraping publicly visible catalogue and pricing data is one thing. Pushing traffic through proxies to get past login walls, payment checks, or purchase limits on a store you don't own is a different conversation, and it's governed by the store's terms and by local law. Shopify's own API documentation and the merchants' terms are the reference points there, not a proxy provider's marketing page.

## Match the job to the proxy model

| What you're doing | What usually breaks | Model that fits |
| --- | --- | --- |
| Scraping catalogues, prices, stock from many stores | Per-store rate limits and Cloudflare | Rotating residential, ideally IP-based with a big pool |
| Managing several shops or staff accounts | Session resets triggering security locks | Sticky residential sessions, one IP per store |
| Checking currency, shipping and geoblocks | Localisation that only appears from the right country | Residential with country or city targeting |
| Limited drops and Queue-it releases | Queue position assigned by IP reputation | Residential or mobile, sticky sessions, low concurrency per IP |
| Bulk API work on your own store | REST 2 req/s, GraphQL 50 points/s | A handful of sticky IPs, plus delays and 429 handling |

The pattern: anything that depends on staying logged in or holding a session wants stickiness. Anything that depends on volume across many targets wants rotation. Proxies priced per IP with unmetered bandwidth (9Proxy is one example) fit the first pattern; per-GB plans fit the second.

## Where 9Proxy fits into a Shopify workflow

9Proxy is a residential proxy network — 20M+ IPs across 90+ countries, HTTP/HTTPS and SOCKS5, pay-as-you-go with no subscription requirement. Three things about it line up reasonably well with the Shopify use cases above.

**IP-based plans come with unmetered bandwidth.** You buy a number of residential IPs and pay nothing for the data that moves through them. For catalogue scraping, where each page is small but you might run ten thousand pages a day, that removes the guessing game of how many gigabytes a month's work will consume. The tradeoff is that IP-based plans run through the 9Proxy desktop app, which handles local port forwarding and optional proxy authentication. That's a real constraint: it means Windows or Mac in the loop, and it makes multi-device or headless-server setups more work than a plain `user:pass` endpoint.

**GB-based plans give you a dashboard endpoint.** With these you authenticate with username and password or an IP whitelist, no app required, and you get unlimited endpoints — the IP rotates per request in rotating mode, or holds for a configured session in sticky mode. Validity runs 180 days, and the enterprise tiers don't expire at all. If your Shopify work is bursty, or you're driving automation from a server, this is the cleaner path.

**Sessions are holdable, which matters more on Shopify than on most targets.** Sticky sessions let one IP hold across multiple requests for a configurable window. That's what you need for logging into a store admin, stepping through cart and checkout during a geo test, or keeping a multi-step form submission from looking like four different people.

Targeting goes down to country, state, city and ISP level, though third-party comparisons describe the city-level selection as narrower than what the largest premium networks offer. If you need very specific metro coverage — say, a particular US ZIP for a regional shipping test — test that before you commit volume.

Two smaller mechanical details are worth knowing because they're unusual. 9Proxy advertises a 60-second replacement window: if an IP fails within the first minute of activation, it's credited back rather than counted as consumed. And the "Today List" lets you reuse any IP you've touched in the last 24 hours at no extra charge, which cuts waste on testing runs and abandoned sessions. One reviewer's teardown estimated that reuse behaviour trims IP consumption noticeably for teams that rotate frequently.

## Full plan and price list

9Proxy bills in USD with no recurring commitment. IP-based and bundle pricing changed in 2026 — the company announced an adjustment dated for early June, absorbed by higher connection stability and throughput claims, while GB-based pricing stayed where it was. Rates below are the published figures after that change.

| Package | Model | What you get | Price (USD) | Validity | Buy |
| --- | --- | --- | --- | --- | --- |
| 100 IPs | IP-based | 100 residential IPs, unmetered bandwidth | $24 | Unused IPs don't expire | [Start with the 100-IP package](https://bit.ly/9-Proxy) |
| 500 IPs | IP-based | 500 residential IPs, unmetered bandwidth | $72 | Unused IPs don't expire | [Get 500 IPs](https://bit.ly/9-Proxy) |
| 1,000 IPs + 500 bonus | IP-based | 1,500 IPs total, unmetered bandwidth | $126 | Unused IPs don't expire | [Take the 1,500-IP deal](https://bit.ly/9-Proxy) |
| 2,500 IPs | IP-based | Unmetered bandwidth | $210 | Unused IPs don't expire | [Buy 2,500 IPs](https://bit.ly/9-Proxy) |
| 5,000 IPs | IP-based | Unmetered bandwidth | $360 | Unused IPs don't expire | [Buy 5,000 IPs](https://bit.ly/9-Proxy) |
| 15,000 IPs | IP-based | Unmetered bandwidth | $720 | Unused IPs don't expire | [Buy 15,000 IPs](https://bit.ly/9-Proxy) |
| 25,000 IPs | IP-based | Unmetered bandwidth | $863 | Unused IPs don't expire | [Buy 25,000 IPs](https://bit.ly/9-Proxy) |
| 50,000 IPs | IP-based | Unmetered bandwidth | $1,438 | Unused IPs don't expire | [Buy 50,000 IPs](https://bit.ly/9-Proxy) |
| 100,000 IPs | IP-based (business) | Unmetered bandwidth | $2,300 | Unused IPs don't expire | [Request the 100,000-IP tier](https://bit.ly/9-Proxy) |
| 200,000 IPs | IP-based (business) | Unmetered bandwidth | $4,140 | Unused IPs don't expire | [Request the 200,000-IP tier](https://bit.ly/9-Proxy) |
| 500,000 IPs | IP-based (business) | Unmetered bandwidth | $8,625 | Unused IPs don't expire | [Request the 500,000-IP tier](https://bit.ly/9-Proxy) |
| 5 GB | GB-based | Rotating or sticky residential, unlimited endpoints | $15 | 180 days | [Buy a 5 GB test pack](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | GB-based | Rotating or sticky residential | $105 | 180 days | [Buy the 55 GB pack](https://bit.ly/9-Proxy) |
| 100 GB | GB-based | Rotating or sticky residential | $150 | 180 days | [Buy 100 GB](https://bit.ly/9-Proxy) |
| 200 GB | GB-based | Rotating or sticky residential | $200 | 180 days | [Buy 200 GB](https://bit.ly/9-Proxy) |
| 1,000 GB | GB-based | Rotating or sticky residential | $800 | 180 days | [Buy 1,000 GB](https://bit.ly/9-Proxy) |
| 2,000 GB | GB-based | Rotating or sticky residential | $1,500 | 180 days | [Buy 2,000 GB](https://bit.ly/9-Proxy) |
| 3,000 GB | GB-based (enterprise) | Rotating or sticky residential | $2,160 | No expiry | [Buy the 3,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| 6,000 GB | GB-based (enterprise) | Rotating or sticky residential | $4,200 | No expiry | [Buy the 6,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| 10,000 GB | GB-based (enterprise) | Rotating or sticky residential | $6,800 | No expiry | [Buy the 10,000 GB enterprise pack](https://bit.ly/9-Proxy) |
| 100 IPs + 5 GB | Bundle | IPs plus bandwidth in one pack | $30 | Traffic valid 180 days | [Buy the starter bundle](https://bit.ly/9-Proxy) |
| 1,500 IPs + 50 GB | Bundle | IPs plus bandwidth in one pack | $180 | Traffic valid 180 days | [Buy the mid-size bundle](https://bit.ly/9-Proxy) |
| 5,000 IPs + 500 GB | Bundle | IPs plus bandwidth in one pack | $720 | Traffic valid 180 days | [Buy the pro bundle](https://bit.ly/9-Proxy) |

The top two IP tiers are quoted rather than shelf-priced at most providers, including this one, so treat the six-figure IP rows as indicative and confirm the figure at signup. Everything from 100 to 50,000 IPs is standard published pricing.

## The cost math for three real Shopify workloads

**Nine stores, one admin login each.** You need nine IPs that stay put and don't get shared with another store's session. That's the 100-IP package at $24 — you use nine and keep the rest parked for rotation on the scraping side. Unused IPs don't expire, so the leftovers aren't wasted money.

**Five thousand storefronts scraped weekly.** At roughly 40–60 KB per request through `.json` endpoints, 5,000 stores × 20 products works out to a few gigabytes a week, not a few hundred. The 200 GB pack at $200 over 180 days is more bandwidth than this workload will touch. The real constraint is concurrency, not traffic: with 2 requests per second per IP per store, 100 IPs handle this comfortably in an afternoon. More IPs buy you speed here, not capacity.

**Ongoing monitoring with a light touch.** Price checks on a few hundred competitor stores, a couple of times a day, plus a bit of ad verification. This is where the 50 GB + 5 GB pack at $105 lands — the effective rate drops to $2.10/GB, and 180 days of validity means a quiet month doesn't burn the balance.

## Setup details that decide whether it works

**Rotate per store, not per page.** Rotating your IP on every request through a single store looks less like a careful crawler and more like a distributed attack. Assign one IP per store, run it at two requests per second, and move on. The throughput comes from breadth of IPs, not from pressure on each one.

**Use sticky sessions for anything with a login.** Residential IPs on the IP-based model naturally live anywhere from a few hours to about 24 hours, depending on whether the underlying home connection stays up. Every IP change during an admin session is a fresh signal to whatever security layer the store runs. If long, unbroken sessions are how your workflow operates, that's the honest reason to look at static ISP proxies rather than residential ones — a point 9Proxy's own support team has made when answering complaints about IPs dropping after an hour.

**Handle 429 properly before scaling.** Read `Retry-After`, sleep that long, and back off. If you solve rate limiting by buying more IPs instead of honouring the header, you'll pay for the same data twice.

**Split private apps per task.** Shopify tracks the access token alongside the IP. Five thousand price updates pushed through one token will hit limits no matter how many IPs are behind it. Separate tokens per job is cheaper than a bigger proxy package.

**Check the client before you commit.** If your stack runs headless on a Linux VPS, the GB-based plans are the ones that work with plain credentials. The IP-based plans want the desktop app in the loop, and that's a genuine architectural decision, not a footnote.

You can 👉 [create a 9Proxy account and check current package availability](https://bit.ly/9-Proxy) before buying volume — a small pack plus a week of testing on your own targets tells you more than any comparison table, including this one.

## Limits worth knowing before you pay

Third-party reviews of 9Proxy are mixed, and the negative ones cluster around the same complaints: IPs expiring faster than expected, refund friction, and support response times. Trustpilot's page for the provider carries a low aggregate score with the company replying to essentially every negative review — including detailed explanations that residential IPs can't be guaranteed a fixed lifespan because they belong to real households. That's an accurate description of how residential networks work, and it's also a warning that if your Shopify workflow assumes a stable IP for 30 days, residential proxies are the wrong product category.

On the feature side, the pool is 20M+ IPs — large enough that exhaustion isn't a practical concern for storefront scraping, but a tier below the networks advertising 70–100M+. Datacenter proxies are listed as coming soon rather than available, so there's no cheap tier for the restock-monitoring jobs where datacenter IPs would be fine. City-level targeting is supported but narrower than premium alternatives. And the mandatory app for IP-based plans is a real constraint if you're allergic to desktop software.

Set against that: the pricing is among the lowest in the residential category. Independent cost breakdowns place 9Proxy at roughly $1.30–$2 per GB at typical volumes, versus $8–11 per GB at the enterprise-tier providers. For Shopify scraping, where your targets are readable storefronts rather than Amazon's or Google's detection stack, that gap is the whole argument.

## FAQ

**Do I need residential proxies to scrape Shopify stores?**
For a handful of stores with light Cloudflare protection, datacenter IPs can work. Once you're sweeping hundreds of stores or running more than a couple of requests per second per store, datacenter ranges get filtered at the edge and residential is the practical option.

**How many proxies do I need for a Shopify scraping job?**
Divide your target request rate by two. If you want 100 requests per second across a store list, 50 IPs running at 2 requests per second each will do it. More IPs raise throughput, but the pacing per IP is what keeps you out of the retry loop.

**Can I use one proxy for multiple Shopify stores?**
You can, but you shouldn't for admin sessions. Store A and Store B hitting the same exit IP links the two accounts in any log that cares to look. One IP per store is the cheap insurance.

**Is it cheaper to buy IPs or gigabytes here?**
IP-based, if you know you'll move a lot of data through few targets — bandwidth is unmetered. GB-based, if your work is bursty, spread across thousands of rotating targets, or driven from a server without a desktop app.

**Can I test before buying?**
Trials have been offered on a limited basis depending on availability, and a 5 GB pack at $15 is the low-risk alternative. Either way, run it against your actual store list before scaling — success rate on your targets is the number that matters.

## The short version

Shopify's rate limits are per IP and per store, which means the fix for catalogue scraping is breadth of clean residential IPs, and the fix for multi-store management is one sticky IP per store. Those are different products, and 9Proxy sells both under one account — unmetered IP packages for the first, rotating or sticky GB packages with a dashboard endpoint and 180-day validity for the second. The 100-IP tier at $24 and the 55 GB tier at $105 are the two entry points that cover most Shopify work, and neither locks you into a subscription.

Pick based on the job, not the price per unit. The cheapest line on a proxy pricing page is usually priced for a workload you don't have.
