# Impactful Challenges

---

## Case 1 — Lokalise ($profit)

One of the more impactful challenges I dealt with recently was related to **localization in a multilingual product**.

We support around **10 languages**, including RTL languages like Hebrew and more complex ones like Chinese. Over time, we started seeing a pattern — every sprint, a noticeable number of small translation-related tasks were coming in: text fixes, copy updates, urgent wording changes.

Individually, these tasks were minor, but collectively they were causing **a lot of context switching** for developers. Especially when working on complex features, engineers had to constantly interrupt their flow to handle these quick requests. It started affecting focus, delivery predictability, and overall team efficiency.

I realized this wasn't really a technical problem, but more of a **process and ownership issue**. So I took ownership of investigating it.

I did some research on localization management tools and identified a few options. Then I created a small proof of concept and proposed integrating **Lokalise** — mainly because it provides a good UI for non-developers and has seamless GitHub integration.

Initially, there wasn't much enthusiasm from the product side, mainly because this solution introduced additional costs. However, I was able to build a case around efficiency — showing that the time developers were spending on translation tasks was actually **more expensive than the tool itself**.

### Key idea

Decouple translation work from the development workflow:

- Move translation management to product/translation stakeholders
- Sync translation files automatically with the repository
- Let the system generate pull requests with updates instead of developers doing manual changes

I aligned this approach with our CTO and product stakeholders, and then drove the implementation.

### Results

- Developers were no longer interrupted by translation tasks
- Non-engineering teams could manage translations independently
- Significantly reduced context switching
- Sprint planning became more stable and predictable
- Reduced costs by freeing up developer time for more valuable work

> Additionally, the solution proved to be valuable beyond our team — **two other teams** working on related products adopted the same approach after seeing the results.

---

## Case 2 — Image Upload Performance

One of the more technical challenges I faced recently was related to a **production incident** after a change in our image upload flow.

Our application is used by inspectors in the field — they create issues and attach photos of defects. Previously, images were uploaded through our backend, but we migrated to a more scalable approach where the frontend uploads images **directly to S3** using pre-signed URLs.

The feature was implemented, tested, and released to production. However, shortly after release, we started receiving complaints — especially from mobile users — that image uploads became **significantly slower**. In some cases, uploading multiple images took up to **30–40 seconds**.

To make things more challenging, the developer who implemented the feature was on vacation, so I had to quickly step in, investigate, and deliver a hotfix.

Since the issue was not reproducible for all users, I relied heavily on **logs and monitoring** — we use Coralogix — to analyze real production behavior.

### Root causes

After digging into it, I identified **two main problems**:

**1. Excessive network requests**

Previously, we had a single request per upload batch via the backend. With the new approach, each image required multiple requests:

- Pre-signed URL requests (for both original and thumbnail)
- Actual upload requests

Since a single issue could include up to **20 images**, this resulted in a large number of **sequential requests** — a major bottleneck, especially on slower mobile networks.

**2. Broken compression logic**

Image compression only worked for files larger than 4MB, meaning most images were **not being optimized** at all.

### Fix

- **Parallelized** both the pre-signed URL generation and the upload process, significantly reducing total upload time
- **Removed the compression threshold** so that all images were properly optimized before upload

### Results

- Upload time dropped from **~30–40s** to **~2–3s** for multiple images
- Issue resolved for mobile users
- Reduced network load overall

> A good example of handling a production issue under time pressure — combining debugging, performance optimization, and delivering a fast, high-impact fix.
