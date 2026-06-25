# Product Surface & Refresh System

## Landing page grid

The storefront landing page shows published products in a single grid. The default sort is **Featured** (labelled *Editor's choice*).

Featured order is driven by a server-side `popularity` score recomputed on a regular cadence for every published product. The score blends five signals:

| Signal | Weight | Source |
|---|---|---|
| **Newness** | ~35% | Sequential counter assigned on first publish. Higher = newer. |
| **Staff pick** | ~20% | Manual flag set in the admin product editor. |
| **Freshness** | ~10% | Time decay since release or publish date. Newer items score higher. |
| **Latest drop** | ~15% | Product belongs to the most recently released Sunday Drop. |
| **Period jitter** | ~20% | Deterministic hash of product ID + current time bucket. Ensures order changes every refresh without being random. |

Newness + latest-drop = about half the score, so the newest drop dominates the top of the grid for its release week. Staff picks provide editorial override. Jitter keeps the surface feeling alive.

## Periodic refresh job

A scheduled database job runs several times per day. It checks a single-row guard table and skips if the last refresh was too recent. Each run:
1. Recalculates popularity for every published product
2. Identifies the latest released drop and applies the boost
3. Rebuilds related-product suggestions per product
4. Updates the guard timestamp

## Client-side freshness

The storefront polls the backend on a short interval and subscribes to real-time catalog changes. In-memory catalog state expires quickly. If a real-time event is missed, the next poll or window-focus fetches fresh data. Typical latency is sub-second; worst-case fallback is under a minute.

## Sequential ordering

A database trigger fires on product insert and on first publish. It assigns the next sequential order value, which persists permanently. This guarantees the newness signal is never lost — even if a product is archived and re-published, its original order stays.

## Product images

### Background removal

All product cutout images start as raw photos. Backgrounds are removed with **PhotoRoom**, which is currently free for this workflow. The cutout is uploaded to the admin as a PNG with transparency. A future API connection to PhotoRoom (or a similar service) is planned to automate this step.

### Upload and optimization

After background removal, images go through a three-step admin workflow:
1. **Request upload URL** — signed URL from a media edge function
2. **Upload** — direct PUT to object storage
3. **Finalize** — edge function processes the image: detects dimensions, alpha channel, bounding box, and optimizes the file for delivery

Public delivery goes through a caching proxy that adds long-lived immutable cache headers per storage object. Images are never served directly from the raw storage endpoint.

### Storage

During beta, images are stored in a **Supabase S3-compatible storage bucket**. For production, the plan is to migrate to a dedicated object storage provider such as **Google Cloud Storage**, **Amazon S3**, or **Yandex S3**, depending on cost and performance.

## Newsletter signup

Public signup form posts to a newsletter edge function. The function validates email format, checks a honeypot field, enforces rate limits by IP and email, hashes the IP for an audit log, and inserts the subscriber into a locked-down table. The subscriber table has no public write access — the edge function is the only entry point.
