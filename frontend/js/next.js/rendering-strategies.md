# Rendering Strategies in Next.js

## CSR — Client-Side Rendering

The page arrives as empty HTML; all rendering happens in the browser after JS loads.

**Pros:**
- Full interactivity immediately after initial render
- Cheap to host — can be served from a static CDN
- Good for authenticated apps (dashboards, personal accounts)

**Cons:**
- Poor SEO — crawlers see an empty page
- Slow FCP — user sees a white screen until JS loads
- Waterfall: load JS → execute → fetch data → render

**Use when:**
- Entire app is behind a login (SEO doesn't matter)
- Components with frequently changing user data (cart, settings)
- Interactive widgets with no data on first render

**Avoid when:**
- Public pages (landing, blog, catalog)
- SEO or Time to First Byte matters

---

## SSR — Server-Side Rendering

HTML is generated on the server on every request and sent ready.

**Pros:**
- Good SEO — crawlers receive ready HTML
- Data is always fresh at request time
- Can read cookies, headers, and session directly on the server

**Cons:**
- Higher TTFB than SSG — server must render before responding
- Server load scales linearly with traffic
- Cannot be CDN-cached without extra effort

**Use when:**
- Pages with personalized data (`/dashboard`, `/profile`)
- Data that must be shown fresh
- Access to `cookies()` / `headers()` in a Server Component is needed

**Avoid when:**
- Pages with identical content for all users
- High-traffic public pages

---

## SSG — Static Site Generation

HTML is generated once at build time and served as a static file.

**Pros:**
- Fastest TTFB — files served from CDN
- Excellent SEO
- No server load at runtime
- Cheap and reliable

**Cons:**
- Data goes stale — requires a new build to update
- Long build times with thousands of pages
- Not suitable for personalized content

**Use when:**
- Marketing pages, landing pages
- Documentation, blog (if real-time isn't needed)
- Catalog pages with rarely changing data

**Avoid when:**
- Data changes frequently (prices, stock, news)
- Personalized content
- Hundreds of thousands of pages with unique content

---

## ISR — Incremental Static Regeneration

SSG + background revalidation by time or by event. The page rebuilds in the background while users receive the cached version.

```ts
// time-based revalidation
export const revalidate = 60; // rebuild every 60 seconds

// event-based revalidation (on-demand)
revalidatePath('/blog/[slug]');
```

**Pros:**
- SSG speed + relatively fresh data
- No full rebuild needed when content updates
- On-demand revalidation gives near real-time updates

**Cons:**
- First user after TTL expires gets stale data (stale-while-revalidate)
- On-demand revalidation requires a webhook from CMS/API
- Harder to debug than SSG

**Use when:**
- Blog, news site — content changes but not every second
- Product catalog with prices
- Pages with data from a CMS

**Avoid when:**
- Data changes faster than once per minute
- Absolute freshness is critical (financial data, chats)

---

## PPR — Partial Prerendering

Experimental feature introduced in Next.js 14. The page component itself is static (pre-rendered at build time); `<Suspense>` boundaries mark dynamic islands that are streamed on request.

```tsx
// next.config.ts
experimental: {
  ppr: true,
}

// app/page.tsx
export default function Page() {
  return (
    <>
      <StaticContent /> {/* pre-rendered at build time */}
      <Suspense fallback={<Skeleton />}>
        <DynamicContent /> {/* streamed on request */}
      </Suspense>
    </>
  );
}
```

**Pros:**
- Instant TTFB for the static part (from CDN)
- Dynamic data is still fresh
- No architecture rewrite needed — just `<Suspense>` boundaries

**Cons:**
- Experimental status — API may change
- Requires correctly placed `<Suspense>` boundaries
- Dynamic parts still require server rendering

**Use when:**
- Page is mostly static + a few personalized blocks
- Landing with a button that depends on auth state
- E-commerce: static product card + dynamic price/availability

**Avoid when:**
- The entire page is dynamic — PPR gives no benefit
- Simple cases where plain SSR is sufficient

---

## Comparison

| | CSR | SSR | SSG | ISR | PPR |
|---|---|---|---|---|---|
| TTFB | Fast | Medium | Fast | Fast | Fast |
| SEO | Poor | Good | Excellent | Excellent | Excellent |
| Data freshness | Real-time | Real-time | At build | By TTL | Mixed |
| Server load | None | High | None | Low | Low |
| Personalization | Yes | Yes | No | No | Partial |
| Complexity | Low | Medium | Low | Medium | High |

---

## How to Choose

```
Data changes in real-time and is personalized?
└── Yes → SSR or CSR (behind auth)

Data is public and rarely changes?
├── Never changes → SSG
└── Changes sometimes → ISR

Page is mixed (static shell + dynamic blocks)?
└── PPR (if comfortable with experimental)
```

---

## Glossary

**TTFB (Time To First Byte)** — the time between the browser sending a request and receiving the first byte of the server's response.
