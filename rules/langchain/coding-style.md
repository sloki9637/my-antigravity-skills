# NestJS + Prisma + LangChain + Redis + Puppeteer Coding Rules

> These rules are **MANDATORY**.
> Do not infer intent. Follow exactly.
> Any violation MUST be rejected.

---

# 1. Responsibility Separation

## Controller MUST handle:

- HTTP request/response mapping
- DTO validation
- Calling Service methods
- Returning Response DTO only

## Controller MUST NOT:

- Contain business logic
- Access Prisma directly
- Access Redis directly
- Access Puppeteer directly
- Call LangChain directly
- Handle transactions

---

## Service MUST handle:

- Business logic
- Prisma transaction boundaries
- LangChain orchestration
- Redis caching orchestration
- Puppeteer orchestration
- DTO ↔ Entity conversion

## Service MUST NOT:

- Access Express `req` / `res`
- Return Prisma model directly
- Contain raw SQL
- Perform browser-level low logic (must delegate to PuppeteerService)

---

## Repository MUST handle:

- Prisma data access only

## CacheRepository MUST handle:

- Redis operations only

## PuppeteerService MUST handle:

- Browser lifecycle
- Page navigation
- DOM extraction
- Screenshot generation

---

# 2. Directory Structure

```
src/
  global/
    config/
    security/
    exception/
    cache/
    browser/
      puppeteer.service.ts
  domain/
    [feature]/
      controller/
      service/
      repository/
      entity/
      dto/
      enums/
  prisma/
```

---

### Rules

- Puppeteer MUST be inside `global/browser/`
- Feature modules MUST NOT instantiate browser directly
- Browser instance MUST NOT be created per request unless required
- Reusable browser instance SHOULD be managed via singleton service

---

# 3. Puppeteer Architecture Rules

## 3.1 Browser Lifecycle

- Browser MUST be launched once on application bootstrap
- Browser MUST be closed gracefully on shutdown
- Headless mode MUST be configurable via environment variable
- Sandbox flags MUST be configurable for Docker

Example required flags:

```
--no-sandbox
--disable-setuid-sandbox
```

---

## 3.2 PuppeteerService Responsibilities

PuppeteerService MUST:

- Provide high-level methods:
  - `renderPage(url)`
  - `extractText(url)`
  - `screenshot(url)`

- Implement timeout control
- Implement navigation error handling
- Implement retry logic (max 3)

PuppeteerService MUST NOT:

- Contain business logic
- Access Prisma
- Access Redis
- Call LangChain

---

# 4. Security Rules (Critical)

Puppeteer introduces SSRF risk.

## MUST:

- Validate URLs before navigation
- Block internal IP ranges:
  - 127.0.0.1
  - 169.254.x.x
  - 10.x.x.x
  - 192.168.x.x
  - 172.16.x.x

- Restrict protocol to:
  - http
  - https

- Implement navigation timeout
- Disable file:// access
- Disable request interception unless necessary

## MUST NOT:

- Allow arbitrary local network crawling
- Allow execution of downloaded scripts
- Allow user to inject browser launch args

---

# 5. Service Orchestration Rule (LLM + Puppeteer)

When using Puppeteer with LangChain:

Service MUST:

1. Fetch web content using PuppeteerService
2. Sanitize extracted content
3. Optionally cache in Redis
4. Pass sanitized content to LangChain
5. Persist result via Repository

Order MUST be:

```
Puppeteer → (optional Cache) → LangChain → Prisma → Cache invalidation
```

Never:

```
LangChain → Puppeteer
```

(LLM must not control browser actions)

---

# 6. Redis Caching for Scraped Data

## MUST:

- Cache rendered HTML separately from parsed result
- Use key namespace:

```
crawl:html:{hash}
crawl:text:{hash}
```

- TTL MUST be configurable
- Large HTML (>1MB) MUST NOT be cached

---

# 7. Transaction Rules with Puppeteer

Browser operations MUST occur:

- OUTSIDE Prisma transaction

Database writes MUST occur:

- INSIDE transaction

Correct pattern:

```
1. Crawl
2. Process
3. Begin transaction
4. Save
5. Commit
6. Invalidate cache
```

Forbidden:

```
Begin transaction
Crawl external site
Wait 5 seconds
Commit
```

(No long I/O inside transaction)

---

# 8. Performance Rules

- Max concurrent pages MUST be limited
- Page instance MUST be closed after use
- Network idle wait MUST use timeout limit
- Heavy crawling MUST use queue system (BullMQ recommended)

---

# 9. Logging Rules

MUST log:

- Crawled URL (sanitized)
- Execution time
- Retry count

MUST NOT log:

- Full HTML content
- Sensitive tokens
- Internal IP errors

---

# 10. Rate Limiting Rules

When exposing crawl endpoint:

- MUST apply rate limiter
- MUST limit per IP
- MUST implement concurrency guard

---

# 11. Testing Rules

## Unit Test

- PuppeteerService MUST be mocked
- No real browser launch in unit tests

## Integration Test

- Use headless mode
- Use controlled test URLs
- Must verify timeout handling

---

# 12. AI + Crawl Safety Rules

When combining Puppeteer + LangChain:

- Extracted content MUST be sanitized
- Scripts MUST be removed
- HTML tags SHOULD be stripped before LLM input
- LLM prompt MUST be fixed template
- User input MUST NOT be appended to system prompt directly

---

# 13. Environment Variables

Required:

```
PUPPETEER_HEADLESS=true
PUPPETEER_TIMEOUT=10000
PUPPETEER_MAX_CONCURRENCY=5
```

---

# 14. Forbidden Patterns

- new Browser() inside Controller
- Using Puppeteer inside Repository
- Long crawling inside transaction
- Caching raw HTML without size limit
- LLM deciding which URL to crawl

All MUST be rejected.

---

# 15. Rule Priority

1. Security (SSRF prevention)
2. Transaction safety
3. Architecture separation
4. Performance
5. Convenience

---

# 16. Enforcement

If any code:

- Instantiates puppeteer in Controller → REJECT
- Uses browser inside Repository → REJECT
- Crawls without URL validation → REJECT
- Performs crawling inside DB transaction → REJECT

No exceptions.

---

# 🔥 Final Architecture Summary

Layer Flow:

```
Controller
   ↓
Service
   ↓
PuppeteerService
   ↓
LangChain
   ↓
Prisma Repository
   ↓
Redis Cache
```

Strict orchestration boundary 유지.

---

원하면 다음 단계로:

- 🔥 대규모 크롤링 아키텍처 (Queue + Worker 분리)
- 🔥 AI Agent + Puppeteer Tool 구조
- 🔥 Distributed crawler 설계
- 🔥 Production-grade AI scraping 서버 설계
