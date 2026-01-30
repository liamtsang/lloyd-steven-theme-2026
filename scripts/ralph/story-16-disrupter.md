# Story 16: Collection Grid Disrupter Block

## Overview

Add a "disrupter" editorial image block to the collection product grid. The disrupter spans multiple grid cells and contains a sticky image that pins in the viewport as the user scrolls past adjacent products.

**Reference:** [Sarah & Sebastian — Hoop Earrings](https://www.sarahandsebastian.com/en-us/collections/hoop-earrings)

### Visual Layout (3-column grid, desktop)

```
P  P  P        <- row 1: 3 normal products
D  D  P        <- row 2: disrupter starts (cols 1-2), product (col 3)
D  D  P        <- row 3: disrupter continues, product
D  D  P        <- row 4: disrupter continues, product
P  P  P        <- row 5: products resume normally
```

The disrupter occupies **2 columns × 3 rows**. The image inside it uses `position: sticky` so it stays visible while the user scrolls through the 3-row tall area.

---

## Critical Technical Context

### The Flexbox Problem

The current product grid uses **CSS Flexbox** (`display: flex; flex-wrap: wrap` in `assets/base.css` line 925). Flexbox is one-dimensional — items flow left-to-right and wrap, but **cannot span multiple rows and columns simultaneously**. There is no flex equivalent of `grid-column: span 2; grid-row: span 3`.

### The Solution

Convert the product grid to **CSS Grid** — but ONLY when a disrupter block is present. This is done by:

1. Adding a `.product-grid--has-disrupter` class to the `<ul>` when a disrupter exists
2. That class applies `display: grid !important` which overrides the base `.grid` flex declaration
3. When no disrupter exists, the class is absent, and the grid stays as flexbox

**DO NOT modify `assets/base.css`.** The base `.grid` class is used across the entire theme. All changes are additive via a new CSS file.

### How Sticky Works Here

`position: sticky` requires the parent element to be **taller than the sticky child**. Here's the chain:

1. `<li class="grid__item--disrupter">` — CSS Grid makes this span 3 rows, so its height equals ~3 product cards
2. `.collection-disrupter` — `height: 100%` fills the full `<li>`
3. `.collection-disrupter__media` — `position: sticky; top: 0; height: 100vh` pins the image

As the user scrolls, the image sticks to the viewport top. Once the bottom of the `<li>` parent scrolls past, the image scrolls away with it.

If the site has a sticky header, change `top: 0` to `top: var(--header-height, 0px)` or a fixed value like `top: 80px` to prevent the image from going under the header.

---

## Step 1: Create `assets/component-collection-disrupter.css`

This is a **new file**. It contains two sections: the CSS Grid override and the disrupter component styles.

```css
/* ============================================
   PRODUCT GRID: CSS Grid override
   Only active when .product-grid--has-disrupter is present.
   The base .grid flexbox in base.css is NOT touched.
   ============================================ */

.product-grid--has-disrupter {
  display: grid !important;
  grid-template-columns: repeat(2, 1fr);
  column-gap: var(--grid-mobile-horizontal-spacing);
  row-gap: var(--grid-mobile-vertical-spacing);
}

@media screen and (min-width: 750px) {
  .product-grid--has-disrupter {
    column-gap: var(--grid-desktop-horizontal-spacing);
    row-gap: var(--grid-desktop-vertical-spacing);
  }
}

@media screen and (min-width: 990px) {
  .product-grid--has-disrupter.grid--3-col-desktop {
    grid-template-columns: repeat(3, 1fr);
  }

  .product-grid--has-disrupter.grid--4-col-desktop {
    grid-template-columns: repeat(4, 1fr);
  }

  .product-grid--has-disrupter.grid--5-col-desktop {
    grid-template-columns: repeat(5, 1fr);
  }

  .product-grid--has-disrupter.grid--6-col-desktop {
    grid-template-columns: repeat(6, 1fr);
  }
}

/*
 * Reset flex-based width calculations from base.css.
 * CSS Grid's 1fr handles sizing; the old calc() widths would conflict.
 */
.product-grid--has-disrupter .grid__item {
  width: auto;
  max-width: none;
}

/* ============================================
   DISRUPTER: Grid spanning
   ============================================ */

/* Desktop (990px+): 2 columns, 3 rows */
@media screen and (min-width: 990px) {
  .grid__item--disrupter {
    grid-column: span 2;
    grid-row: span 3;
  }
}

/* Tablet (750-989px): full width, 2 rows */
@media screen and (min-width: 750px) and (max-width: 989px) {
  .grid__item--disrupter {
    grid-column: span 2;
    grid-row: span 2;
  }
}

/* Mobile (<750px): full width, single row, no spanning */
@media screen and (max-width: 749px) {
  .grid__item--disrupter {
    grid-column: 1 / -1;
    grid-row: span 1;
  }
}

/* ============================================
   DISRUPTER: Component styles
   ============================================ */

.collection-disrupter {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  border-radius: var(--card-corner-radius, 0);
}

/*
 * Sticky image wrapper.
 * Pins the image to the viewport as the user scrolls through the
 * tall disrupter area. Only works because the parent <li> is taller
 * (grid-row: span 3) than this element.
 *
 * If there is a sticky header, change top: 0 to
 * top: var(--header-height, 0px) to avoid overlap.
 */
.collection-disrupter__media {
  position: sticky;
  top: 0;
  width: 100%;
  height: 100vh;
  max-height: 100%;
  overflow: hidden;
}

.collection-disrupter__image {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
}

/* Mobile: no sticky, static image with fixed aspect ratio */
@media screen and (max-width: 749px) {
  .collection-disrupter__media {
    position: relative;
    height: auto;
    aspect-ratio: 3 / 4;
  }
}

/* ---- Optional text overlay ---- */

.collection-disrupter__content {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 2rem;
  z-index: 2;
  color: #fff;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.5) 0%,
    rgba(0, 0, 0, 0) 100%
  );
}

.collection-disrupter__heading {
  font-size: 1.5rem;
  margin: 0 0 0.5rem;
  font-weight: 400;
}

.collection-disrupter__link {
  display: inline-block;
  color: #fff;
  text-decoration: underline;
  text-underline-offset: 0.3em;
}
```

### Important Notes

- The `!important` on `display: grid` is intentional — it must override `display: flex` from the base `.grid` class without editing `base.css`.
- The `width: auto; max-width: none` reset on `.grid__item` is critical. Without it, the old `calc(33.33% - ...)` widths from `base.css` will fight with CSS Grid's `1fr` columns.
- The sticky `top` value may need adjustment if the theme has a sticky/fixed header. Check `layout/theme.liquid` for `--header-height` or similar CSS variables.

---

## Step 2: Create `snippets/collection-disrupter.liquid`

This is a **new file**. It renders the disrupter's inner content.

```liquid
{%- comment -%}
  Renders a disrupter block within the collection product grid.

  Accepts:
  - image:       {Image}  The disrupter image (required)
  - heading:     {String} Optional heading text
  - link_url:    {String} Optional URL to link to
  - link_text:   {String} Optional link label
  - lazy_load:   {Boolean} Whether to lazy-load the image
{%- endcomment -%}

{%- if image != blank -%}
  <div class="collection-disrupter">
    <div class="collection-disrupter__media">
      {%- if link_url != blank -%}
        <a href="{{ link_url }}" class="collection-disrupter__link-wrapper full-unstyled-link">
      {%- endif -%}

      {%- assign image_widths = '450, 660, 900, 1100, 1500, 1780, 2000' -%}
      {{
        image
        | image_url: width: 1500
        | image_tag:
          loading: lazy_load | default: true | ternary: 'lazy', 'eager',
          sizes: '(min-width: 990px) 66vw, 100vw',
          widths: image_widths,
          class: 'collection-disrupter__image'
      }}

      {%- if link_url != blank -%}
        </a>
      {%- endif -%}
    </div>

    {%- if heading != blank or link_text != blank -%}
      <div class="collection-disrupter__content">
        {%- if heading != blank -%}
          <h3 class="collection-disrupter__heading">{{ heading }}</h3>
        {%- endif -%}
        {%- if link_url != blank and link_text != blank -%}
          <a href="{{ link_url }}" class="collection-disrupter__link">
            {{ link_text }}
          </a>
        {%- endif -%}
      </div>
    {%- endif -%}
  </div>
{%- endif -%}
```

### Notes

- The `image_tag` filter with `widths` and `sizes` generates a responsive `<img>` with `srcset`. The `sizes` attribute tells the browser the disrupter is ~66vw on desktop (it spans 2 of 3 columns) and 100vw on mobile.
- The `ternary` filter handles lazy vs eager loading. The first 2 products in the grid use eager loading; the disrupter should use lazy since it's typically inserted after position 3+.
- The `full-unstyled-link` class is a standard Dawn/Shopify theme utility class that removes default link styling.

---

## Step 3: Modify `sections/main-collection-product-grid.liquid`

There are 4 distinct changes to this file. Make them in order.

### 3a. Add conditional CSS loading (top of file, after line 3)

After the existing `component-price.css` stylesheet tag, add the disrupter detection and CSS load:

```liquid
{{ 'component-price.css' | asset_url | stylesheet_tag }}

{%- assign has_disrupter = false -%}
{%- for block in section.blocks -%}
  {%- if block.type == 'disrupter' and block.settings.image != blank -%}
    {%- assign has_disrupter = true -%}
    {%- break -%}
  {%- endif -%}
{%- endfor -%}
{%- if has_disrupter -%}
  {{ 'component-collection-disrupter.css' | asset_url | stylesheet_tag }}
{%- endif -%}
```

This ensures the CSS is only loaded on collection pages that actually use a disrupter.

### 3b. Add CSS Grid class to the `<ul>` container

Find the `<ul id="product-grid">` element (around line 146-153) and add the conditional class:

**Before:**
```liquid
<ul
  id="product-grid"
  data-id="{{ section.id }}"
  class="
    grid product-grid grid--{{ section.settings.columns_mobile }}-col-tablet-down
    grid--{{ section.settings.columns_desktop }}-col-desktop
    {% if section.settings.quick_add == 'bulk' %} collection-quick-add-bulk{% endif %}
  "
>
```

**After:**
```liquid
<ul
  id="product-grid"
  data-id="{{ section.id }}"
  class="
    grid product-grid grid--{{ section.settings.columns_mobile }}-col-tablet-down
    grid--{{ section.settings.columns_desktop }}-col-desktop
    {% if has_disrupter %} product-grid--has-disrupter{% endif %}
    {% if section.settings.quick_add == 'bulk' %} collection-quick-add-bulk{% endif %}
  "
>
```

### 3c. Replace the product loop to insert the disrupter

Find the product loop (lines 155-182) and replace it entirely:

**Before:**
```liquid
{% assign skip_card_product_styles = false %}
{%- for product in collection.products -%}
  {% assign lazy_load = false %}
  {%- if forloop.index > 2 -%}
    {%- assign lazy_load = true -%}
  {%- endif -%}
  <li
    class="grid__item{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--slide-in{% endif %}"
    {% if settings.animations_reveal_on_scroll %}
      data-cascade
      style="--animation-order: {{ forloop.index }};"
    {% endif %}
  >
    {% render 'card-product',
      card_product: product,
      media_aspect_ratio: section.settings.image_ratio,
      image_shape: section.settings.image_shape,
      show_secondary_image: section.settings.show_secondary_image,
      show_vendor: section.settings.show_vendor,
      show_rating: section.settings.show_rating,
      lazy_load: lazy_load,
      skip_styles: skip_card_product_styles,
      quick_add: section.settings.quick_add,
      section_id: section.id
    %}
  </li>
  {%- assign skip_card_product_styles = true -%}
{%- endfor -%}
```

**After:**
```liquid
{% assign skip_card_product_styles = false %}

{%- comment -%} Find the disrupter block if one exists {%- endcomment -%}
{%- assign disrupter_block = nil -%}
{%- assign disrupter_position = 3 -%}
{%- for block in section.blocks -%}
  {%- if block.type == 'disrupter' and block.settings.image != blank -%}
    {%- assign disrupter_block = block -%}
    {%- assign disrupter_position = block.settings.position -%}
    {%- break -%}
  {%- endif -%}
{%- endfor -%}

{%- for product in collection.products -%}
  {% assign lazy_load = false %}
  {%- if forloop.index > 2 -%}
    {%- assign lazy_load = true -%}
  {%- endif -%}

  {%- comment -%} Insert disrupter at the configured position {%- endcomment -%}
  {%- if disrupter_block and forloop.index == disrupter_position -%}
    <li
      class="grid__item grid__item--disrupter{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--slide-in{% endif %}"
      {{ disrupter_block.shopify_attributes }}
      {% if settings.animations_reveal_on_scroll %}
        data-cascade
        style="--animation-order: {{ forloop.index }};"
      {% endif %}
    >
      {% render 'collection-disrupter',
        image: disrupter_block.settings.image,
        heading: disrupter_block.settings.heading,
        link_url: disrupter_block.settings.link_url,
        link_text: disrupter_block.settings.link_text,
        lazy_load: lazy_load
      %}
    </li>
  {%- endif -%}

  <li
    class="grid__item{% if settings.animations_reveal_on_scroll %} scroll-trigger animate--slide-in{% endif %}"
    {% if settings.animations_reveal_on_scroll %}
      data-cascade
      style="--animation-order: {{ forloop.index }};"
    {% endif %}
  >
    {% render 'card-product',
      card_product: product,
      media_aspect_ratio: section.settings.image_ratio,
      image_shape: section.settings.image_shape,
      show_secondary_image: section.settings.show_secondary_image,
      show_vendor: section.settings.show_vendor,
      show_rating: section.settings.show_rating,
      lazy_load: lazy_load,
      skip_styles: skip_card_product_styles,
      quick_add: section.settings.quick_add,
      section_id: section.id
    %}
  </li>
  {%- assign skip_card_product_styles = true -%}
{%- endfor -%}
```

Key points:
- `{{ disrupter_block.shopify_attributes }}` enables the theme editor to highlight/select the disrupter block
- The disrupter is inserted **before** the product at the configured position, so it appears after `position - 1` products
- If `disrupter_position` exceeds the product count, the disrupter simply won't render (safe fallback)

### 3d. Add the blocks array to the section schema

The section schema currently has only `"settings": [...]`. Add a `"blocks"` array as a sibling, right before the closing `}` of the schema object.

Find:
```json
    }
  ]
}
```

(The last setting `padding_bottom` followed by the closing of `settings` array and schema object)

Add `"blocks"` between `]` and `}`:

```json
    }
  ],
  "blocks": [
    {
      "type": "disrupter",
      "name": "Disrupter image",
      "limit": 1,
      "settings": [
        {
          "type": "image_picker",
          "id": "image",
          "label": "Image"
        },
        {
          "type": "range",
          "id": "position",
          "min": 1,
          "max": 24,
          "step": 1,
          "default": 4,
          "label": "Insert after product number",
          "info": "The disrupter appears after this many products. For a 3-column grid, use multiples of 3 (e.g. 3, 6, 9) to align with row boundaries."
        },
        {
          "type": "header",
          "content": "Optional content"
        },
        {
          "type": "text",
          "id": "heading",
          "label": "Heading"
        },
        {
          "type": "url",
          "id": "link_url",
          "label": "Link URL"
        },
        {
          "type": "text",
          "id": "link_text",
          "label": "Link text",
          "default": "Shop now"
        }
      ]
    }
  ]
}
```

---

## Edge Cases

| Scenario | Expected Behavior |
|----------|-------------------|
| No disrupter block added | Grid stays as flexbox, CSS not loaded, zero impact |
| Position exceeds product count | Disrupter does not render (safe no-op) |
| 4-column grid | Disrupter spans 2 of 4 columns — works correctly, just narrower |
| Page 2+ of pagination | Position is relative to current page's products |
| Disrupter image not set | Snippet guards with `if image != blank`, nothing renders |
| Mobile viewport | Full-width image, no sticky, `aspect-ratio: 3/4` |

---

## Testing Checklist

- [ ] Add disrupter block in theme editor, upload an image, verify it renders at the configured position
- [ ] Scroll on desktop — image should stick while 3 product cards scroll alongside
- [ ] Test with 3-column desktop grid
- [ ] Test with 4-column desktop grid
- [ ] Test tablet (750-989px) — 2-col span, 2-row height
- [ ] Test mobile (<750px) — full width, no sticky, static image
- [ ] Remove disrupter block — verify grid returns to normal with no visual changes
- [ ] Test various position values (1, 3, 6, last product)
- [ ] Test with position > product count — disrupter should not appear
- [ ] Verify heading and link text overlay render correctly
- [ ] Test without heading/link — clean image only, no overlay
- [ ] Check lazy loading with DevTools Network tab
- [ ] Verify no CLS (Cumulative Layout Shift)
- [ ] Confirm scroll animations still cascade correctly
- [ ] Verify the theme editor can select/highlight the disrupter block
