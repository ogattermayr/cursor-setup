# Research Tools

Find the right tool to get data from the internet. Prioritizes our built backend APIs first, then MCP tools.

## Usage

```
/research <what you need>
```

## Step 1: Read the Tools Inventory

Read the full tools skill at `.cursor/skills/tools-and-apis/SKILL.md` for the complete inventory.

## Step 2: Identify the Right Tool

Based on what the user needs, select the best tool using this priority order:

### Priority 1 — Built Backend APIs (howthef-backend/)

These have custom logic, caching, error handling, and cost optimization. **Always try these first.**

| Need | Service File | Provider |
|------|-------------|----------|
| **Web search** | `features/llm/services/web_search_service.py` | Anthropic web search |
| **Twitter/X content** | `features/scraping/services/news_scraping_service.py` | xAI Grok Live Search |
| **Scrape a website** | `features/scraping/services/website_scraping_service.py` | Firecrawl (Redis cached) |
| **Scrape news/articles** | `features/scraping/services/news_scraping_service.py` | Firecrawl → Tavily → Apify |
| **Run Apify actors** | `features/scraping/services/apify_service.py` | Apify |
| **Google search / SERP** | `features/scraping/services/serper_website_finder_service.py` | Serper ($0.001/search) |
| **Find company website** | `features/scraping/services/company_website_finder_service.py` | Perplexity (multi-strategy) |
| **Keyword volume / SEO** | `internal_agents/blog_agent/seo_research/services/blog_seo_keyword_service.py` | Keywords Everywhere |
| **SERP analysis** | `internal_agents/blog_agent/seo_research/services/blog_seo_serp_service.py` | Serper |
| **LinkedIn posts** | `internal_agents/content_agent/linkedin/services/content_linkedin_research_service.py` | Apify |
| **Reddit data** | `internal_agents/reddit_research/services/reddit_client.py` | asyncpraw |
| **Contact enrichment** | `features/crm_enrichment/services/apollo_client.py` | Apollo |
| **Email discovery** | `features/crm_enrichment/services/kitt_client.py` | KITT |
| **Email verification** | `features/crm_enrichment/services/email_verification_service.py` | BounceBan |
| **LLM calls (any provider)** | `features/llm/services/llm_service.py` | OpenAI, Anthropic, xAI, Gemini, Groq, Perplexity |

### Priority 2 — MCP Tools (when no built API exists)

Use MCPs for things we haven't built custom services for, or for quick one-off tasks directly in Cursor.

| Need | MCP Server | Key Tools |
|------|-----------|-----------|
| **Quick web research** | `user-perplexity` | `perplexity_ask`, `perplexity_research`, `perplexity_reason` |
| **Deep topic research** | `user-firecrawl-mcp` | `firecrawl_deep_research` |
| **Quick scrape (no backend)** | `user-firecrawl-mcp` | `firecrawl_scrape`, `firecrawl_extract` |
| **Site mapping** | `user-firecrawl-mcp` | `firecrawl_map`, `firecrawl_crawl` |
| **Database queries** | `user-supabase` | `execute_sql`, `list_tables` |
| **GitHub operations** | `user-MCP_DOCKER` | `create_pull_request`, `search_code`, `list_issues` |
| **Stripe billing** | `user-MCP_DOCKER` | `create_customer`, `create_invoice`, `list_subscriptions` |
| **LLM debugging** | `user-langsmith-howthef` | `fetch_runs`, `list_experiments` |
| **Product analytics** | `user-posthog` | PostHog API queries |
| **Issue tracking** | `user-Linear` | `create_issue`, `list_issues`, `update_issue` |
| **Task management** | `user-clickup` | `create_task`, `get_tasks` |
| **Browser testing** | `cursor-ide-browser` | `browser_navigate`, `browser_snapshot`, `browser_take_screenshot` |
| **Performance profiling** | `user-chrome-devtools` | `performance_start_trace`, `evaluate_script` |

## Step 3: Recommend and Execute

1. **Tell the user** which tool you recommend and why (cost, speed, data quality)
2. **If it's a built API** — write the code or integrate it into the relevant agent/service
3. **If it's an MCP** — call it directly using `CallMcpTool`
4. **If neither works** — suggest what new service to build and where it should go in the backend

## Decision Shortcuts

- **Cheapest web search:** Serper ($0.001) → Perplexity MCP (free in Cursor) → Perplexity backend ($0.05)
- **Best for Twitter/X:** Grok Live Search (only real option — direct X data access)
- **Best for scraping:** Built `website_scraping_service.py` (cached) → Firecrawl MCP (uncached)
- **Best for company research:** Serper (cheap, fast) → Perplexity (smart, expensive) → Firecrawl deep research (thorough)
- **Best for SEO:** Keywords Everywhere (volume/CPC) + Serper SERP analysis (competitive)
- **Best for LinkedIn:** Apify via `content_linkedin_research_service.py`

## Important

- **Cost awareness:** Serper is $0.001/search, Perplexity is $0.05/search, Apify varies by actor. Prefer cheaper options unless the user needs higher quality.
- **Caching:** Built APIs like `website_scraping_service.py` cache results in Redis. MCPs don't cache.
- **Rate limits:** Be mindful of API rate limits, especially with Apify and Apollo.
- **Backend vs MCP:** If you're building a feature that will be reused, build a backend service. If it's a one-off research task, use MCPs directly.
