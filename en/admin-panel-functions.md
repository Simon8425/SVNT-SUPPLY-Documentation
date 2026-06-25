# Admin Panel — Function Registry

## Architecture

The admin panel is protected behind Supabase Auth + an ACL whitelist. Two layers power it:

- **Data stores** — pure TypeScript modules that call Supabase or edge functions. No React, testable in isolation.
- **UI components** — React components that consume the data stores. Desktop-only viewport enforcement.

Every mutation goes through Row-Level Security. Storage operations use signed URLs. No admin operation touches the database without authentication and ACL membership.

---

## Auth & Session Layer

| Function | Purpose |
|---|---|
| `signInWithManagement` | Supabase email/password sign-in + ACL check against the admin whitelist. Calls login-throttle edge function. Supports mock auth for local development. |
| `createManagementSession` | Creates session with crypto-random token, hostname fingerprint, absolute TTL, and inactivity timeout. |
| `getActiveManagementSession` | Validates and touches session. Constant-time fingerprint comparison. Returns null if expired/invalid. |
| `clearManagementSession` | Wipes session from localStorage and signs out of Supabase. |
| `getAttemptSnapshot` | Returns login attempt state: failures in window, remaining attempts, lockout status. |
| `registerFailedAttempt` | Records failed attempt and applies exponential cooldown. |
| `RequireManagementSession` | Route guard. Validates auth + ACL on mount and route changes. Redirects to login if invalid. |

---

## Product CRUD

| Operation | Function(s) | What it does |
|---|---|---|
| **List** | `listProducts` | Paginated product table with search by name/brand/category. |
| **Load bundle** | `loadProductBundle` | Full product: row, translations, colors, media assets, presentations, buy links, related products. |
| **Create** | `createEmptyBundle` | Draft product with English translation skeleton, card/PDP presentations, one buy link. |
| **Update** | `saveProductBundle` | Upserts product row + translations + colors + buy links + related products. Manages media lifecycle. |
| **Publish** | `publishProduct` | Validates required fields, sets status to published, invalidates catalog cache. |
| **Archive** | `archiveProduct` | Sets status to archived, invalidates catalog cache. |
| **Delete** | `deleteProduct` | Permanently deletes product + enqueues media assets for delayed cleanup. |

**Media operations:**

| Operation | Function(s) | What it does |
|---|---|---|
| **Request upload URL** | `requestMediaUploadUrl` | Gets signed upload URL from the product-media edge function. |
| **Upload file** | `uploadMediaFile` | PUTs file to signed URL. |
| **Finalize** | `finalizeUploadedProductBundle` | Edge function processes the image (dimensions, alpha detection, bounding box). Retries once on transient errors. |
| **Delete storage** | `deleteMediaStorageObjects` | Removes files from storage via edge function. |

**Product editing form fields:**
- Slug, Brand, Category (multi-select with primary), Prices (EUR/USD), Drop assignment
- Image: drag-and-drop PNG upload, dual preview panes, reposition/zoom/auto-fit
- Colors: toggle grid of classic swatches
- Buy links: URL, Label (multiple links per product)
- Editorial: Name, Verdict, What it is, Why it made the cut, Pros/Cons, Durability score, Who it's for/not for
- Toggles: Staff Pick (editor's choice), Status badges

---

## Category CRUD

| Operation | Function(s) | What it does |
|---|---|---|
| **List** | `listCategories` | All categories ordered by desktop sort order. |
| **Batch save** | `saveCategories` | Upserts labels, sort orders, and image URLs. Invalidates landing page cache on write. |
| **Upload image** | `uploadCategoryImage` | Optimizes + uploads tile image via signed URL. Returns public URL. |

**Category editor UI features:**
- Separate desktop/mobile sort orders with drag-and-drop reordering
- Inline label editing per tile
- Image upload per tile with preview
- Desktop/Mobile preview toggle (mobile inside an iPhone mockup)
- Saves as batch — no individual create/delete (categories are a fixed curated set)

---

## Drop (Product Release) CRUD

| Operation | Function(s) | What it does |
|---|---|---|
| **List** | `listDrops` | All drops with computed state and product count. |
| **Load** | `loadDrop` | Single drop by ID. |
| **Create** | `createDrop` | New drop with Sunday release date. Invalidates catalog cache. |
| **Update** | `updateDrop` | Change name or release date of a scheduled drop. Invalidates catalog cache. |
| **Soft delete** | `softDeleteDrop` | Sets a deleted-at timestamp. Drop becomes hidden. Invalidates catalog cache. |
| **Restore** | `restoreDrop` | Clears deleted-at. Drop becomes visible again. Invalidates catalog cache. |
| **List products** | `getDropProducts` | Products assigned to a drop with brand + name. |
| **Remove product** | `removeProductFromDrop` | Unassigns a product from its drop. |

**Business rules:**
- Release dates must be Sundays at a fixed afternoon time.
- Drop names auto-normalize to `DROP 001`, `DROP 002`, etc.
- Products can only belong to one drop at a time.
- Only scheduled drops can be edited; released drops are immutable.

---

## Brand CRUD

| Operation | Function(s) | What it does |
|---|---|---|
| **List** | `listStorefrontBrands` | All brands sorted alphabetically. |
| **Batch save** | `saveStorefrontBrands` | Validates (no duplicate slugs/names), upserts brands array. |
| **Delete** | `deleteStorefrontBrands` | Deletes brands by ID. Also cleans up logo files from storage. |
| **Upload logo** | `uploadBrandLogo` | Optimizes + uploads via signed URL. Returns logo URL + storage metadata. |
| **Delete logo** | `deleteBrandLogoFiles` | Removes logo files from storage. |

**UI features:**
- Responsive grid of brand cards
- Inline name editing with auto-save
- Logo upload via click or drag-and-drop per card
- Replace overlay for existing logos
- Product brand suggestion chips (auto-detected from catalog)

---

## Newsletter Campaign CRUD

| Operation | Function(s) | What it does |
|---|---|---|
| **List** | `listCampaigns` | Paginated campaigns with status filter + text search. |
| **Load** | `loadCampaign` | Single campaign with full HTML content. |
| **Save** | `saveCampaign` | Creates or updates campaign in the backend (with localStorage fallback). |
| **Delete** | `deleteCampaign` | Deletes campaign + its storage assets + any local cached assets. |
| **Mark sent** | `markCampaignSent` | Sets status to sent with recipient count. |
| **Last sent** | `getLatestSentCampaignAt` | Timestamp of the most recently sent campaign. |

**Canva import pipeline:**

| Step | Function(s) | What it does |
|---|---|---|
| **Import folder** | `importCanvaFolder` | Selects HTML entry point, resolves image assets, optimizes to WebP, returns analysis. |
| **Analyze** | `analyzeCanvaHtmlDocument` | Sanitizes HTML and extracts image references with warnings. |
| **Sanitize** | `sanitizeCanvaHtmlDocument` | Removes dangerous blocks/scripts and hardens email-client compatibility. |
| **Rewrite URLs** | `rewriteCanvaHtmlImageUrls` | Replaces asset URLs in HTML with public storage URLs. |
| **Preview** | `buildCanvaPreviewHtml` | Builds preview HTML with resolved image URLs. |
| **Persist** | `persistCanvaImportedAssets` | Stores optimized assets locally for later save. |

**Campaign builder UI features:**
- Drag-and-drop folder import or directory picker
- Desktop/Mobile email preview
- Asset statistics sidebar
- Save modal: campaign name, subject, sender name

---

## Newsletter Analytics (Read-only)

| Operation | Function(s) | What it does |
|---|---|---|
| **Snapshot** | `getSnapshot` | Aggregated metrics for a time range: total subscribers, active, unsubscribed, bounced. |
| **Overview** | `getOverview` | Quick stats: total subscribers, campaign count, open rates, bounce rates. |
| **Subscribers page** | `getSubscribersPage` | Paginated subscriber table with status, dates, search. |
| **Campaigns page** | `getCampaignsPage` | Paginated campaign performance with delivery stats. |

**Time-series bucketing:** Groups metrics into standard windows. Rate series are computed as both bucketed and cumulative.

---

## Edge Functions

| Function | Purpose |
|---|---|
| `management-login-throttle` | IP-hashed rate limiting for login attempts. |
| `product-media-upload` | Signed upload URLs + delete for product images. |
| `product-media-cleanup-worker` | Background deletion of orphaned product assets. |
| `product-media-public` | Caching proxy for product images. |
| `category-media-upload` | Optimized category tile image upload. |
| `brand-logo-upload` | Brand logo upload + deletion. |
| `newsletter-campaign-save` | Save draft campaign with HTML content. |
| `newsletter-campaign-finalize` | Lock campaign for sending. |
| `newsletter-campaign-send` | Dispatch campaign via email provider. |
| `newsletter-analytics-snapshot` | Campaign analytics data. |
| `newsletter-bounce-retry-worker` | Retry soft bounces. |
| `wishlist-share-create` / `wishlist-share-read` | Create and read anonymous shared wishlist links. |

---

## Database Entities

| Entity | Purpose |
|---|---|
| `management_admins` | ACL whitelist of admin emails with active flag. RLS-protected. |
| `management_login_attempts` | Audit log of login attempts with hashed IPs. |
| `products` | Product catalog: slug, brand, price, status, staff pick, popularity, drop assignment. |
| `product_translations` | English editorial content per product. |
| `product_colors` | Color swatches per product. |
| `product_media_assets` | Image metadata and storage references. |
| `product_media_presentations` | Per-surface crop/scale/trim for card and hero views. |
| `product_buy_links` | Affiliate purchase links. |
| `product_related_products` | Curated "you might also like" relationships. |
| `product_categories` | Landing page category tiles with sort orders and images. |
| `product_drops` | Sunday Drop releases with soft-delete support. |
| `storefront_brands` | Admin-managed brand metadata with logos. |
| `newsletter_subscribers` | Subscribers with status and unsubscribe tokens. |
| `newsletter_campaigns` | Draft/sent campaigns with HTML content and delivery stats. |
| `newsletter_campaign_recipients` | Per-recipient delivery tracking. |

---

## Security Model

1. **Auth**: Supabase email/password. No magic links.
2. **ACL**: Admin whitelist checked on every session validation. Only active members pass.
3. **Session**: Absolute TTL, inactivity timeout, hostname fingerprint, crypto-random token.
4. **Rate limiting**: Client-side exponential cooldown + server-side IP-hashed throttle.
5. **Storage**: All uploads use signed URLs. Public bucket access is proxied through edge functions with strong cache headers.
6. **RLS**: Every table has Row-Level Security. Admin tables are locked to authenticated users in the ACL.
