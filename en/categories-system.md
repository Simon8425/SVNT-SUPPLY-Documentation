# Categories System

## Role of categories

SVNT SUPPLY uses categories as the primary browse structure. The landing page has a curated set of category tiles (tech, home, books, lifestyle, jewelry, clothing, cosmetics, gifts, pets, travel, beauty, skincare, fragrance, fashion, accessories, shoes, bags, and an "all" catch-all). These are not auto-generated from product data — they are a deliberate, hand-maintained taxonomy.

## Data model

Each category has:

| Field | Purpose |
|---|---|
| `id` | Short string key |
| `slug` | URL-safe unique identifier |
| `label` | Display name in the UI |
| `image_url` | Tile background image (optional, falls back to a product image) |
| `desktop_sort_order` | Position in desktop grid |
| `mobile_sort_order` | Position in mobile list |

Categories exist independently of products. A product references a primary category, and a junction table supports **multi-category** assignment (one product can appear in multiple categories with a sort order per assignment).

## Sorting

Desktop and mobile use **separate sort orders**. This is intentional — the desktop grid layout benefits from a different arrangement than the mobile vertical list. The admin UI provides independent drag-and-drop reordering for each viewport, with live previews.

## Admin UI

The category editor provides:

- **Desktop preview**: A responsive grid of category tiles matching the public landing page
- **Mobile preview**: The same tiles rendered inside an iPhone mockup with fake storefront chrome
- **Drag-and-drop reorder**: Pointer events on desktop, native HTML5 drag on mobile
- **Inline label editing**: Click the pencil icon on any tile, type, blur to save
- **Image upload per tile**: Click a tile to upload/replace its background image. Images are optimized server-side via a signed edge function
- **Batch save**: All changes (labels, orders, images) are saved together. A "Reset" button reverts to the last saved state

## Public display

On the storefront landing page, categories render as a tiled grid (desktop) or scrollable list (mobile). Each tile shows the category label and an image. Tapping a category navigates to a collection page, which shows all products in that category in a filtered product grid.

The "All" category exists permanently — it can be reordered but not deleted. It shows every published product regardless of category.

## Multi-category products

A product can belong to multiple categories through a junction table. One category is designated as "primary," which determines the product's breadcrumb context. Additional categories are secondary — the product appears in those browse pages but its primary category is used for labeling.

In the admin product editor, this is a multi-select dropdown with a "Set primary" button. Categories are searchable and selected by toggle.

## Edge function

A category-media edge function handles tile image uploads. The function accepts image data via signed URL, optimizes it, and returns the public URL.

## Future directions

- Category descriptions or taglines shown on collection pages
- Category-specific editorial intros (the "Why we love tech" paragraph per category)
- Dynamic category creation from the admin (currently categories are a fixed curated set)
