---
name: tools-and-apis
description: Complete inventory of all backend APIs, external services, MCP tools, and data sources. Use when the user needs to get data from the internet, research something, use external tools/APIs, scrape websites, search the web, find company info, analyze keywords, or needs to know what services are available.
---

# Tools & APIs — Full Inventory

This is the complete reference for ALL data sources, research tools, APIs, and MCPs available in the HowTheF* monorepo.

**Priority order: Built backend APIs first → MCP tools second → Raw external APIs last.**

---

## 1. Built Backend APIs (howthef-backend/)

These are custom-built services with error handling, caching, fallbacks, and business logic. **Always use these first.**

### 1.1 Web Search & General Research

#### Anthropic Web Search Tool
- **File:** `features/llm/services/web_search_service.py`
- **What:** General web search using Claude's built-in web search capability
- **How:** Uses Anthropic Messages API with `web_search` tool type
- **When to use:** General fact-finding, verifying information, current events
- **Env:** `ANTHROPIC_API_KEY`

```python
from features.llm.services.web_search_service import web_search_tool
result = await web_search_tool.ainvoke({"query": "latest AI funding rounds 2025"})
```

#### LLM Service (Multi-Provider)
- **File:** `features/llm/services/llm_service.py`
- **What:** Unified interface to 6 LLM providers
- **Providers:** `openai`, `anthropic`, `grok` (xAI), `google` (Gemini), `groq`, `perplexity`
- **Key methods:**
  - `generate_structured_output()` — Get Pydantic-validated responses
  - `invoke_structured_output()` — Enhanced with Live Search for Grok
  - `invoke_with_messages()` — Raw message-based calls
- **Env:** `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GROK_API_KEY`, `GOOGLE_GEMINI_API_KEY`, `GROQ_API_KEY`, `PERPLEXITY_API_KEY`

```python
from features.llm.services.llm_service import LLMService
llm = LLMService()
result = await llm.generate_structured_output(
    prompt="Analyze this company", response_model=MyModel, provider="anthropic"
)
```

### 1.2 Twitter/X Data (xAI/Grok Live Search)

#### Grok Live Search — Twitter/X Content
- **File:** `features/scraping/services/news_scraping_service.py`
- **What:** Scrape Twitter/X posts and threads using Grok's native X data access
- **How:** Uses `xai_sdk` with Live Search tools: `x_keyword_search`, `x_semantic_search`
- **When to use:** Any Twitter/X content — tweets, threads, user posts, trending topics
- **Model:** `grok-4` (primary), `gpt-4o-mini` (fallback)
- **Env:** `GROK_API_KEY`

```python
from features.scraping.services.news_scraping_service import NewsContentScraperService
scraper = NewsContentScraperService()
# Automatically uses Grok Live Search for twitter.com/x.com URLs
result = await scraper.scrape_content(url="https://x.com/user/status/123")
```

### 1.3 Website Scraping & Company Data

#### Website Scraping (Firecrawl-based)
- **File:** `features/scraping/services/website_scraping_service.py`
- **What:** Scrape websites and extract structured company profiles
- **Features:** Redis caching (24h TTL), RIA page pattern detection, multi-page scraping
- **When to use:** Scraping a known URL for structured data extraction
- **Env:** `FIRECRAWL_API_KEY`

```python
from features.scraping.services.website_scraping_service import WebsiteScrapingService
scraper = WebsiteScrapingService()
profile = await scraper.scrape_and_extract_profile(url="https://example.com")
```

#### News Content Scraping (Multi-Provider Fallback)
- **File:** `features/scraping/services/news_scraping_service.py`
- **What:** Scrape news articles with automatic fallback chain
- **Fallback order:** Firecrawl → Tavily → Apify
- **When to use:** Scraping news articles, blog posts, general web content
- **Env:** `FIRECRAWL_API_KEY`, `TAVILY_API_KEY`, `APIFY_API_TOKEN`

#### Apify Actor Runner
- **File:** `features/scraping/services/apify_service.py`
- **What:** Run any Apify actor and get results
- **How:** POST to start actor → poll for completion → GET dataset items
- **When to use:** LinkedIn scraping, custom web scraping actors, data extraction
- **Env:** `APIFY_API_KEY` or `APIFY_API_TOKEN`

```python
from features.scraping.services.apify_service import ApifyService
apify = ApifyService()
results = await apify.run_actor(
    actor_id="apify/web-scraper",
    input_data={"startUrls": [{"url": "https://example.com"}]},
    timeout_seconds=300
)
```

### 1.4 Company & Website Discovery

#### Serper — Google SERP Search
- **File:** `features/scraping/services/serper_website_finder_service.py`
- **What:** Google search results via Serper.dev API
- **Cost:** ~$0.001 per search (very cheap)
- **When to use:** Finding company websites, general Google searches, SERP data
- **Env:** `SERPER_DEV_API_KEY`

```python
from features.scraping.services.serper_website_finder_service import SerperWebsiteFinderService
finder = SerperWebsiteFinderService()
result = await finder.find_company_website(company_name="Acme Wealth", state="CA")
```

#### Perplexity — Company Website Finder
- **File:** `features/scraping/services/company_website_finder_service.py`
- **What:** Multi-strategy company website discovery (direct, brand name, DBA names)
- **Cost:** ~$0.05 per search (more expensive but smarter)
- **When to use:** When Serper can't find it — complex company names, SEC legal names vs brand names
- **Env:** `PERPLEXITY_API_KEY`

```python
from features.scraping.services.company_website_finder_service import CompanyWebsiteFinderService
finder = CompanyWebsiteFinderService()
result = await finder.find_company_website(
    company_name="ROUTE ONE INVESTMENT COMPANY, L.P.", city="San Francisco", state="CA"
)
```

### 1.5 SEO & Keyword Research

#### Keywords Everywhere
- **File:** `internal_agents/blog_agent/seo_research/services/blog_seo_keyword_service.py`
- **What:** Keyword volume, CPC, competition data
- **API:** POST `https://api.keywordseverywhere.com/v1/get_keyword_data`
- **When to use:** Blog topic validation, SEO research, content planning
- **Env:** `KEYWORDS_EVERYWHERE_API_KEY`

```python
from internal_agents.blog_agent.seo_research.services.blog_seo_keyword_service import BlogSeoKeywordService
seo = BlogSeoKeywordService()
data = await seo.get_keyword_data(keywords=["ai automation", "llm agents"], country="us")
# Returns: [{keyword, vol, cpc, competition, trend}, ...]
```

#### Serper SERP Analysis
- **File:** `internal_agents/blog_agent/seo_research/services/blog_seo_serp_service.py`
- **What:** SERP analysis for blog topics — what's ranking, content gaps
- **When to use:** Competitive content analysis, finding content opportunities
- **Env:** `SERPER_DEV_API_KEY`

### 1.6 Social Media & Content

#### LinkedIn Research (via Apify)
- **File:** `internal_agents/content_agent/linkedin/services/content_linkedin_research_service.py`
- **What:** LinkedIn post scraping and research
- **How:** Uses Apify's LinkedIn Post Search Scraper actor
- **When to use:** Competitor analysis, content research, influencer tracking
- **Env:** `APIFY_API_KEY`

#### Reddit Research
- **File:** `internal_agents/reddit_research/services/reddit_client.py`
- **What:** Reddit API access via asyncpraw
- **When to use:** Community research, sentiment analysis, finding pain points
- **Env:** `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`, `REDDIT_REFRESH_TOKEN`

#### Late Social Media
- **File:** `internal_agents/social_media_agent/services/late_social_service.py`
- **What:** Social media posting/management
- **Env:** `LATE_SOCIALMEDIA_API_KEY`

### 1.7 CRM & Contact Enrichment

#### Apollo — Contact Enrichment
- **File:** `features/crm_enrichment/services/apollo_client.py`
- **What:** Find and enrich B2B contact data (emails, titles, company info)
- **Env:** `APOLLO_API_KEY`

#### KITT — Email Enrichment
- **File:** `features/crm_enrichment/services/kitt_client.py`
- **What:** Email discovery and enrichment
- **Env:** `KITT_API_KEY`

#### BounceBan — Email Verification
- **File:** `features/crm_enrichment/services/email_verification_service.py`
- **What:** Verify email deliverability before sending
- **Env:** `BOUNCEBAN_API_KEY`

### 1.8 Communication Channels

#### Slack
- **File:** `shared/services/slack_service.py`
- **What:** Send messages, handle webhooks, agent routing
- **Env:** `SLACK_USER_OAUTH_TOKEN`, `HOWTHEF_SLACK_USER_OAUTH_TOKEN`, `HOWTHEF_SLACK_CHANNEL_ID`

#### Reply.io — SDR Campaigns
- **File:** `internal_agents/sdr_agents/services/replyio_service.py`
- **What:** Email campaign management, contact sequences
- **Env:** `REPLY_IO_API_KEY`

#### Resend — Transactional Email
- **File:** `features/channel_providers/email/services/email_service.py`
- **What:** Email delivery (news editions, transactional)
- **Env:** `RESEND_API_KEY`

#### WhatsApp
- **File:** `features/channel_providers/whatsapp/`
- **What:** WhatsApp message templates and delivery
- **Env:** `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`

### 1.9 Analytics & Tracking

#### PostHog
- **File:** `internal_agents/company_ops/services/gtm_data_service.py`
- **What:** Product analytics data
- **Env:** `POSTHOG_PERSONAL_API_KEY`, `POSTHOG_PROJECT_ID`

#### GA4
- **File:** `internal_agents/company_ops/services/gtm_data_service.py`
- **What:** Google Analytics 4 data
- **Env:** `GA4_PROPERTY_ID`, `GA4_CREDENTIALS_PATH`

#### LangSmith
- **File:** `shared/services/langsmith_query_service.py`
- **What:** LLM run tracing, cost tracking, debugging
- **Env:** `LANGSMITH_API_KEY`

### 1.10 Infrastructure

#### Cloudflare
- **File:** `features/email_and_domain_tools/cloudflare/services/cloudflare_service.py`
- **What:** Domain management, DNS
- **Env:** `CLOUDFLARE_API_TOKEN`

#### Google Workspace
- **File:** `features/email_and_domain_tools/google_workspace/services/google_workspace_service.py`
- **What:** User management, workspace admin
- **Env:** `GOOGLE_SERVICE_ACCOUNT_JSON`, `GOOGLE_WORKSPACE_ADMIN_EMAIL`

---

## 2. MCP Tools (Cursor-Level)

These are available directly in Cursor via MCP servers. Use when there's no built backend API for what you need.

### 2.1 Research & Search MCPs

#### Perplexity MCP
- **Server:** `user-perplexity`
- **Tools:** `perplexity_ask`, `perplexity_reason`, `perplexity_research`, `perplexity_search`
- **When to use:** Quick web research questions directly in Cursor, when you don't need to call the backend

#### Firecrawl MCP
- **Server:** `user-firecrawl-mcp`
- **Tools:** `firecrawl_scrape`, `firecrawl_crawl`, `firecrawl_extract`, `firecrawl_map`, `firecrawl_search`, `firecrawl_deep_research`, `firecrawl_batch_scrape`
- **When to use:** Quick scraping directly in Cursor, deep research on a topic, mapping site structure

### 2.2 Database & Infrastructure MCPs

#### Supabase MCP (HowTheF*)
- **Server:** `user-supabase` — Project: `enjpbshridmqjvlyelrx`
- **Tools:** `execute_sql`, `apply_migration`, `list_tables`, `list_migrations`, `get_logs`, `deploy_edge_function`
- **When to use:** DB queries, schema changes, migrations

#### Supabase MCP (Bright Brains)
- **Server:** `user-supabase-bright-brains`

#### Supabase MCP (Mindless Academy)
- **Server:** `user-supabase-mindless-academy`

### 2.3 Project Management MCPs

#### Linear MCP
- **Server:** `user-Linear`
- **Tools:** `create_issue`, `list_issues`, `update_issue`, `list_projects`, `create_document`, `search_documentation` + 30 more
- **When to use:** Issue tracking, project management, documentation

#### ClickUp MCP
- **Server:** `user-clickup`
- **Tools:** `create_task`, `get_tasks`, `update_task`, `search_docs`, `create_checklist` + 40 more
- **When to use:** Task management, checklists, workspace management

### 2.4 DevOps & Monitoring MCPs

#### LangSmith MCP
- **Server:** `user-langsmith-howthef`
- **Tools:** `fetch_runs`, `list_experiments`, `list_prompts`, `push_prompt`, `run_experiment`, `create_dataset`
- **When to use:** Debugging LLM runs, managing prompts, running experiments

#### PostHog MCP
- **Server:** `user-posthog`
- **When to use:** Querying product analytics data directly

#### Docker Multi-Tool MCP
- **Server:** `user-MCP_DOCKER`
- **Bundles:** GitHub, Stripe, Firecrawl, Perplexity, Docker, file ops
- **Key tools:** `create_pull_request`, `merge_pull_request`, `create_customer` (Stripe), `perplexity_ask`, `docker`, `execute_command`
- **When to use:** GitHub PR operations, Stripe billing, Docker management

### 2.5 Browser & Testing MCPs

#### Cursor IDE Browser
- **Server:** `cursor-ide-browser`
- **Tools:** `browser_navigate`, `browser_click`, `browser_fill`, `browser_snapshot`, `browser_take_screenshot`, `browser_profile_start/stop` + 25 more
- **When to use:** Frontend testing, visual verification, performance profiling

#### Chrome DevTools MCP
- **Server:** `user-chrome-devtools`
- **Tools:** `navigate_page`, `evaluate_script`, `take_screenshot`, `performance_start_trace`, `fill_form` + 20 more
- **When to use:** Advanced browser debugging, performance tracing, script evaluation

---

## 3. Quick Decision Guide

**"I need to search the web"**
→ Built: `web_search_service.py` (Anthropic) | MCP: `perplexity_ask`

**"I need Twitter/X data"**
→ Built: Grok Live Search in `news_scraping_service.py` (ONLY option for X data)

**"I need to scrape a website"**
→ Built: `website_scraping_service.py` (Firecrawl, cached) | MCP: `firecrawl_scrape`

**"I need to find a company's website"**
→ Built: `serper_website_finder_service.py` ($0.001) → `company_website_finder_service.py` ($0.05 fallback)

**"I need keyword/SEO data"**
→ Built: `blog_seo_keyword_service.py` (Keywords Everywhere)

**"I need LinkedIn data"**
→ Built: `content_linkedin_research_service.py` (Apify LinkedIn scraper)

**"I need Reddit data"**
→ Built: `reddit_client.py` (asyncpraw)

**"I need to enrich contacts/companies"**
→ Built: Apollo (`apollo_client.py`), KITT (`kitt_client.py`), BounceBan (`email_verification_service.py`)

**"I need to query the database"**
→ Built: `crud_service.py` | MCP: Supabase `execute_sql`

**"I need to manage issues/tasks"**
→ MCP: Linear or ClickUp

**"I need to debug LLM runs"**
→ MCP: LangSmith `fetch_runs`

**"I need to do deep research on a topic"**
→ MCP: `firecrawl_deep_research` or `perplexity_research`
