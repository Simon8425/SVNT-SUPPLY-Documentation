# Sunday Drop System

## Philosophy

SVNT SUPPLY is not a marketplace with endless inventory. It is a **curated editorial publication** that releases a small, deliberate selection of products each week. The Sunday Drop is the atomic unit of this rhythm — every Sunday, a new batch of hand-picked products goes live.

The name is intentional: **drops**, not collections or releases. It evokes the limited-edition culture of streetwear and design objects, where anticipation and timing matter. You don't browse SVNT SUPPLY casually — you check in on Sunday for what's new.

---

## How it works

### Release cadence

- Drops release **every Sunday at a fixed afternoon time**.
- Each drop has a name (auto-normalized to `DROP 001`, `DROP 002`, etc.) and a release date.
- A product belongs to exactly one drop (or none, if it predates the drop system).

### Drop states

| State | Meaning | Visible to public? |
|---|---|---|
| **Scheduled** | Future release date. Products are hidden until the release time. | No |
| **Released** | Release time has passed. Products are publicly visible. | Yes |
| **Hidden** | Soft-deleted by admin. Removed from public view but preserved. | No |

State is computed on read, not stored. The drop state is derived from release time and soft-delete flag.

### Product visibility gating

The public catalog query filters products by release time. Scheduled products are invisible to storefront visitors until their drop releases. This means an admin can prepare a drop weeks in advance — adding products, writing editorial, uploading images — and it all goes live automatically on Sunday without a manual publish step.

---

## Admin workflow

1. **Create a drop**: Set a name (auto-suggested `DROP 00X`) and a Sunday date.
2. **Assign products**: In the product editor, pick a drop from the dropdown. The product's release time is set to the drop's release time.
3. **Preview**: The drop is "scheduled." Products are hidden publicly but visible in the admin.
4. **Release**: On Sunday, the time gate lifts. Products appear in the catalog automatically.
5. **Archive**: Old drops remain in the archive. The landing page boosts the latest released drop in its popularity score, so the newest drop dominates the grid for its release week.

### Drop management UI

The drop manager provides:
- List view with search, state badges (Scheduled/Released/Hidden), product counts
- Create/Edit view with name and date picker (validates Sunday)
- Soft delete (Hide) and Restore for any drop
- Product assignment view: see all products in a drop, remove products from a drop
- Drops sort chronologically, newest first

---

## Landing page integration

The landing page recommendation algorithm gives a meaningful boost to products in the latest released drop. Combined with the newness signal, this means new-drop products dominate the top of the grid for their release week. As newer drops release, older products naturally decay down the popularity ranking.

### Soft delete design

Drops are never hard-deleted. Soft-delete sets a timestamp rather than removing the row. This preserves referential integrity and allows restoration. Products in hidden drops remain in the database but are excluded from public queries.

---

## Future considerations

- **Multiple drops per week**: The system supports it already. The UI could be extended to show a calendar view.
- **Drop themes**: Drops could carry a thematic label ("Summer Essentials", "Gift Guide") beyond the numbered naming convention.
- **Notifications**: Since drops release at a known time, a notification system could alert subscribers when a new drop goes live.
