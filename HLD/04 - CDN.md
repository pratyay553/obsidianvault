---
tags: [hld]
---

# Content Delivery Network (CDN)

## 🧠 Problem
SAloon's servers are all in one data center in Mumbai. A user in Toronto loading salon photos waits 400ms just for the network round trip, before the server has even started working.

## 🤔 Why does the problem exist
Physical distance costs time — light (and data) can only travel so fast, and every network hop adds latency. A single data center, no matter how fast its servers are, can't shrink the distance to a user on the other side of the planet.

## 💡 Idea
A **CDN** is a network of geographically distributed servers ("edge servers" or "points of presence") that cache and serve static (and increasingly dynamic) content from the location physically closest to the requesting user.

## 🏗️ Structure
```
User in Toronto ──▶ Nearest Edge Server (Toronto) ──▶ HIT: serve cached asset instantly
                                    │
                                    └──▶ MISS: fetch from Origin (Mumbai), cache it at edge, serve it
User in Delhi   ──▶ Nearest Edge Server (Mumbai)  ──▶ likely already cached, fast either way
```

## ⚙️ How it works
1. Static assets (images, CSS, JS, videos) get a CDN URL instead of pointing directly at the origin server.
2. The first request for an asset from a given region is a cache miss — the edge server fetches it from the origin, serves it, and caches it.
3. Every subsequent request from that region is served straight from the edge — no trip back to the origin at all.
4. This is [[03 - Caching]] applied geographically instead of just in-memory.

## 🏢 SAloon Example
Salon profile photos and the marketing landing page are served through a CDN. A user in Toronto gets photos from an edge node in North America; SAloon's Mumbai servers never see that request at all after the first cache warm-up. Meanwhile the *booking* API (dynamic, per-user, must-be-fresh) still goes straight to the origin — CDNs are for content that's the same for everyone, not for personalized or highly volatile data.

## 🎯 When should I use it?
Static or semi-static assets served to a geographically spread-out audience: images, videos, JS/CSS bundles, downloadable files, and increasingly whole static site builds.

## ⚠️ When NOT to use it?
Don't route highly personalized or rapidly changing data (a user's live booking status, an account balance) through a CDN — you'll either serve stale/wrong data or defeat the cache entirely with a 0% hit rate, while adding an extra hop for no benefit.

## 🔥 Interview Question
*"How is a CDN different from a regular cache?"*
→ A regular cache (e.g. Redis) sits close to your application servers and speeds up backend computation. A CDN sits close to the *end user*, geographically distributed, and primarily reduces network latency for static content — it's solving a distance problem, not just a computation-repetition problem.

## 🔗 Related
[[03 - Caching]]
[[02 - Load Balancing]]
