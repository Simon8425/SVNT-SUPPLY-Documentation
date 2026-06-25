# Wishlist & Sharing System

## Current state

SVNT SUPPLY has a **Saved Items** drawer (the "wishlist") that persists product IDs in `localStorage`. Users can:

- Save products from the product card or detail page
- View saved items in a slide-out drawer from the right edge
- Remove items individually
- Share their saved list as a URL

### Sharing architecture

Saved lists are shared through two Supabase edge functions backed by a `wishlist_shares` table:

**Create flow:**
1. User clicks "Share saved list" in the drawer
2. Client validates items (products must exist in the live catalog)
3. Edge function receives product IDs, hashes a random share token, stores the row
4. Returns share token + URL to the client
5. Client copies the URL to clipboard (or falls back to a visible text input + manual copy button)

**Read flow:**
1. Someone opens a shared link
2. Edge function looks up the token hash, returns product IDs
3. Client fetches product data from the catalog and renders a read-only grid

### Security

- Share tokens are random strings, hashed before storage
- The token hash is unique — brute-force scanning is impractical
- Rate limiting: IP-hashed creation log, enforced per-IP
- Shares are immutable — once created, product lists cannot be edited
- No authentication required to create or read (anonymous sharing)
- No personally identifiable information stored (only product IDs and hashed tokens)

### Limits

- Minimum and maximum products per share (enforced client and server)
- Rate limit per IP (configurable in the edge function)
- Shares reference product IDs by string, not foreign keys (tolerant of deleted products)

---

## Future: Public profiles

The next evolution of the wishlist system is **public user profiles**. Instead of one-off share links, users will have persistent profile pages showing their curated picks.

### Planned features

- **Profile creation**: Users sign up and get a public profile at `/@username`
- **Persistent saved lists**: Items sync to the backend instead of localStorage. Survives browser changes.
- **Multiple lists**: "Summer wishlist", "Gift ideas", "Daily carry" — users can create named collections
- **Profile page**: Shows the user's lists, their taste profile (favorite categories, brands), and a short bio
- **Follow system**: Users can follow other curators' profiles. The landing page could show "Trending among people you follow."
- **Editorial picks**: The admin team's profile is a first-class citizen — "The SVNT Edit" as a curated list that updates with each drop

### What stays the same

- Sharing a single list via link continues to work (the current share system)
- Profiles are optional — the saved drawer still works anonymously via localStorage
- The per-share item limit remains

### Technical considerations for profiles

- New tables: user profiles, saved lists, list items, follows
- Auth integration: Supabase Auth with email/password or OAuth
- RLS policies: Users can only read/write their own lists; public profiles are readable by anyone
- The existing share table becomes a secondary feature — profiles are the primary sharing mechanism

---

## Why this matters for the business

Shared wishlists are viral discovery. When someone shares "10 things I'm saving from SVNT SUPPLY," every recipient sees the brand, the curation, and the affiliate links. Public profiles make this permanent — influencers, editors, and power users get a home on the platform. Their followers become SVNT SUPPLY visitors. Their affiliate links generate revenue.
