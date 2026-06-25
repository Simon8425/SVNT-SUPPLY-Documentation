# Business Model

## How SVNT SUPPLY makes money

### 1. Affiliate links

Every product has one or more **buy links** that point to external retailers (Amazon, brand direct stores, specialty shops). These links carry affiliate tags. When a visitor clicks through and purchases, SVNT SUPPLY earns a commission.

Key details:
- **Multiple links per product**: A product can list several purchase options (e.g. Amazon US, Amazon DE, brand store). The admin UI supports multiple buy links with custom labels and availability notes.
- **No inventory, no checkout**: SVNT SUPPLY never holds stock, processes payments, or handles shipping. It's purely a discovery and referral layer.
- **Price display**: Products show prices in EUR and USD. The currency switcher in the storefront lets visitors toggle.

### 2. Newsletter monetization

The newsletter is both a retention tool and a revenue channel:

- **Current**: The newsletter drives repeat traffic. Subscribers get notified of new drops, editorial highlights, and product picks. Higher traffic = more affiliate clicks.
- **Future**: Sponsored placements within newsletter campaigns. Brands pay to have their product featured in a dedicated "Partner Pick" section or as the lead story. The newsletter infrastructure (campaign builder, Canva import, analytics dashboard, email provider integration) already supports this — the commercial layer is the missing piece.

### 3. Multi-site potential

The entire platform is built as a reusable system:

- The backend (products, categories, drops, brands, newsletter, wishlists) is multi-tenant capable
- The admin panel manages one catalog, but the architecture supports running separate instances with different domains, different product sets, different branding
- Potential verticals: tech-only, home/lifestyle, fashion, gifting — each with its own domain, catalog, and newsletter
- Shared infrastructure: one backend project, one hosting account, one codebase with environment-based config

### What we don't do

- **No paid placements in the catalog**: Products in the main grid are editorially selected, not paid. The trust depends on this.
- **No user data selling**: Subscriber emails, wishlist data, browsing behavior are not sold or shared.
- **No ad networks**: No display ads, no tracking pixels beyond the newsletter open tracker (which is first-party and privacy-compliant).

---

## Revenue streams summary

| Stream | Status | Notes |
|---|---|---|
| Affiliate commissions | **Live** | Multiple links per product. Primary revenue driver. |
| Newsletter sponsorships | **Planned** | Infrastructure ready. Needs commercial partnerships. |
| Multi-site licensing | **Future** | White-label the platform for other curators/verticals. |

---

## The curation moat

The business value is not the code — it's the **editorial filter**. Anyone can build an affiliate link page. The hard part is:
1. Consistently picking good products
2. Writing honest, useful reviews
3. Building an audience that trusts the picks
4. Maintaining quality as the catalog grows

SVNT SUPPLY competes on taste, not technology. The tech serves the editorial voice — fast page loads, clean design, no clutter, no dark patterns. That's the product.
