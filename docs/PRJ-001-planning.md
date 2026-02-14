# PRJ-001: Job Search Assistant

**Status:** Planning
**Created:** 2026-02-13
**Owner:** Jon (for Ken)

---

## Goal

Build an AI-enabled, unconventional job search system to help Ken find relevant opportunities.

## Ken's Background

- **Role:** Sr. Director of Customer Support Operations
- **Company:** Arctera (laid off December 2025 due to acquisition)
- **LinkedIn:** https://www.linkedin.com/in/ken-schonberg-5270053
- **Location:** Minnesota (US Central)

---

## Phase 1: Job Discovery

### Requirements
- Search job boards for relevant roles
- Search company career sites directly
- Match against Ken's profile/skills
- Aggregate and prioritize results

### Technical Approaches to Consider

#### Tier 1: Free/Low-Cost APIs (Recommended Start)

1. **Indeed Publisher API** — Free with publisher ID
   - Sign up at indeed.com/publisher
   - JSON API, supports keyword + location search
   - 25 results per call, pagination available
   - Includes salary data, company info
   - **Verdict:** Good starting point, free, legitimate

2. **JSearch (RapidAPI)** — Free tier available
   - Aggregates from Google for Jobs + open web
   - Live listings, salary data
   - Free tier: limited requests/month
   - **Verdict:** Excellent for broad search, worth trying

3. **Jobs Search API (RapidAPI)** — Free tier
   - Aggregates LinkedIn, Indeed, ZipRecruiter
   - Real-time updates
   - **Verdict:** Good coverage, check rate limits

4. **Adzuna API** — Publisher program
   - UK/EU focused but has US listings
   - Requires partnership approval
   - **Verdict:** Secondary option

5. **Jooble API** — Free for publishers
   - Job aggregator across multiple boards
   - Simple REST API
   - **Verdict:** Worth investigating

#### Tier 2: RSS Feeds (Zero API Key Needed)

Many job boards have RSS feeds:
- **Indeed RSS** — `https://rss.indeed.com/rss?q=TITLE&l=LOCATION`
- **RemoteOK** — `https://remoteok.com/remote-jobs.rss`
- **WeWorkRemotely** — RSS available
- **GitHub Jobs mirror sites** — Some still exist
- **StackOverflow Jobs RSS** — If still available

**Verdict:** Excellent passive monitoring, no rate limits, free

#### Tier 3: Scraping (Use Cautiously)

**Low Risk:**
- Company career pages (Workday, Greenhouse, Lever, Taleo)
- Smaller job boards without aggressive protection

**High Risk (Avoid or proxy):**
- LinkedIn — Aggressive anti-bot, legal risk
- Indeed/Glassdoor — CAPTCHAs, IP blocks

**Best Practice:** Use scraping only for:
- Specific company pages Ken is interested in
- Weekly/monthly checks, not constant polling
- Respect robots.txt and rate limits

#### Tier 4: Unconventional Sources

1. **Google Programmable Search Engine**
   - Create a CSE that searches career pages
   - Query: `site:company.com/careers "customer support" "director"`
   - Free tier: 100 queries/day

2. **Twitter/X Advanced Search**
   - `#hiring "customer support" "director" min_faves:10`
   - Monitor company accounts for job tweets
   - No API needed for occasional manual searches

3. **Hacker News "Who is Hiring"**
   - Monthly thread, first of each month
   - Many tech companies post leadership roles
   - Can parse via web_fetch

4. **Reddit Job Subreddits**
   - r/jobs, r/remotework, r/customer service
   - RSS feeds available

5. **Newsletter Digests**
   - The Hustle, Morning Brew have job sections
   - Sign up, parse incoming emails
   - Could use IMAP skill to monitor

6. **Funding News → Job Posts**
   - Monitor TechCrunch, Crunchbase for funding rounds
   - Companies that just raised = likely hiring
   - Lead time: 1-3 months post-announcement

7. **Company Signal Monitoring**
   - Watch for "We're hiring" LinkedIn posts from target companies
   - Monitor company blogs for growth announcements
   - Track leadership changes (new VP = often building teams)

### Profile Extraction Needed
- Key skills to search for
- Title variations (Customer Support, Customer Success, CX Operations, etc.)
- Industry experience
- Leadership level targeting

---

## Phase 2+: To Discuss

- Automated application submission?
- Resume tailoring per job?
- Cover letter generation?
- Interview prep assistance?
- Network mapping (LinkedIn connections)?
- Company research automation?

---

## Open Questions for Tomorrow

1. What types of companies/industries interest Ken?
2. Open to relocation? Remote preference?
3. Compensation expectations?
4. What makes a role "unconventional" in his view?
5. Does he want proactive application or just discovery?
6. Any companies he's particularly interested in or wants to avoid?

---

## Next Steps

- [ ] Fetch and analyze Ken's LinkedIn profile
- [ ] Research available job APIs
- [ ] Prototype a job search query system
- [ ] Evaluate scraping vs API approaches
- [ ] Discuss Phase 2+ ideas

---

## Proposed Architecture

### Phase 1 MVP: Job Discovery Engine

```
┌─────────────────────────────────────────────────────────┐
│                    Job Search System                     │
├─────────────────────────────────────────────────────────┤
│  Config Layer                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Keywords    │  │ Companies   │  │ Locations   │     │
│  │ Titles      │  │ Industries  │  │ Remote Y/N  │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
├─────────────────────────────────────────────────────────┤
│  Data Sources                                            │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌─────────┐ │
│  │ Indeed    │ │ JSearch   │ │ RSS Feeds │ │ Company │ │
│  │ API       │ │ (RapidAPI)│ │           │ │ Sites   │ │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └────┬────┘ │
│        └──────────────┼────────────┼────────────┘      │
│                       ▼            ▼                    │
│              ┌────────────────────────┐                │
│              │   Job Aggregator       │                │
│              │   - Dedupe             │                │
│              │   - Normalize          │                │
│              │   - Score relevance    │                │
│              └───────────┬────────────┘                │
│                          ▼                             │
│              ┌────────────────────────┐                │
│              │   Job Database         │                │
│              │   SQLite/JSON files    │                │
│              └───────────┬────────────┘                │
│                          ▼                             │
│              ┌────────────────────────┐                │
│              │   Output Layer         │                │
│              │   - Daily digest       │                │
│              │   - Telegram alerts    │                │
│              │   - CSV export         │                │
│              └────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

### Key Components

1. **Search Config** — JSON file with:
   - Job titles to search (variations)
   - Keywords (skills, tech stack)
   - Companies to watch (whitelist/blacklist)
   - Location preferences
   - Remote requirements

2. **Fetchers** — One module per source:
   - `fetchers/indeed.js` — Publisher API
   - `fetchers/jsearch.js` — RapidAPI
   - `fetchers/rss.js` — Generic RSS parser
   - `fetchers/company.js` — Direct site scraping

3. **Aggregator** — Combines, dedupes, scores:
   - Hash jobs by (title + company + location)
   - Score relevance against Ken's profile
   - Flag "high match" opportunities

4. **Storage** — SQLite or JSON:
   - Track seen jobs (avoid duplicates)
   - Store full job details
   - Log search history

5. **Output** — Multiple formats:
   - Daily digest via Telegram
   - Weekly CSV export
   - Real-time alerts for high matches

### Cron Schedule (Heartbeat Alternative)

- **Every 6 hours:** Check RSS feeds
- **Daily 8am:** Full API search
- **Weekly:** Deep company site scan
- **Monthly:** HN "Who is Hiring" parse

---

## Title Variations to Search

For Customer Support Operations leadership:

- "Director of Customer Support"
- "Sr. Director, Customer Support"
- "VP of Customer Support"
- "Head of Customer Support"
- "Customer Support Operations Director"
- "Customer Experience Operations Director"
- "Director of CX Operations"
- "Head of Support Operations"
- "Senior Manager, Support Operations"
- "Director, Customer Success Operations"
- "VP of Customer Experience"
- "Chief Customer Officer"
- "Customer Operations Director"

---

## Companies to Consider (Initial List)

**Tech Companies with Large Support Orgs:**
- SaaS companies (Zendesk, Freshworks, Salesforce)
- Cloud infrastructure (AWS, GCP, Azure)
- Consumer tech (Apple, Microsoft, Google)
- Fintech (Stripe, Square, Plaid)
- E-commerce (Shopify, Amazon)

**Minnesota-Based:**
- Target
- Best Buy
- UnitedHealth Group
- 3M
- General Mills
- Medtronic
- Cargill

**Remote-First Companies:**
- GitLab
- Automattic
- Zapier
- Stripe
- Okta

---

## Unconventional Job Search Strategies

### 1. The "Reverse Funnel" Approach
Instead of searching for jobs, identify companies first:
- Find companies with **growing support teams** (check LinkedIn headcount trends)
- Find companies with **recent funding** (Crunchbase API)
- Find companies with **bad support reputation** (ripe for transformation)
- Then watch only those specific companies

### 2. Signal-Based Alerts
Monitor signals that indicate hiring intent:
- Company posts about "scaling support"
- New VP of Customer Success hired (they'll need a team)
- Company launches enterprise product (needs support ops)
- Company has high churn (might be support-related)

### 3. Network Graph Mining
- Find Ken's 2nd-degree connections at target companies
- Identify mutual connections for warm intros
- Map reporting structures at target companies

### 4. Content Marketing Approach
- Have Ken publish articles on LinkedIn about support ops
- Monitor who engages → potential leads
- Positions him as thought leader

### 5. Competitor Intelligence
- Watch where Arctera competitors' support leaders go
- They often need similar talent
- Industry knowledge is valuable

### 6. Conference/Event Mining
- Parse attendee lists from support conferences (SupportDriven, etc.)
- Find companies sending multiple people = growing
- Find speakers → company thought leadership

### 7. Job Posting Age Tracking
- Track how long roles stay open
- Roles open 30+ days = hiring manager may be frustrated
- Reach out with "saw this has been open a while, here's why I'm the solution"

### 8. Internal Referral Optimization
- Research companies with strong referral bonuses
- Find employees most likely to refer (new hires, active on LinkedIn)
- Engage with their content, build relationship

---

## Implementation Priority

**Week 1:**
1. Set up Indeed Publisher account
2. Get JSearch API key (RapidAPI free tier)
3. Build basic job fetcher script
4. Define search parameters with Ken

**Week 2:**
1. Add RSS feed monitoring
2. Build deduplication/scoring logic
3. Set up Telegram alerts
4. Create first daily digest

**Week 3+:**
1. Add company-specific tracking
2. Implement unconventional strategies
3. Build Phase 2 features (applications, resume tailoring)

---

_Last updated: 2026-02-14 (overnight research)
