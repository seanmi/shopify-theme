# Free Shipping Progress Bar

A simple, animated progress bar for Shopify Dawn theme that shows customers how close they are to free shipping.

## Configuration

Added shop metafield `custom.free_shipping_threshold` to allow user to configure the free shipping threshold.

User also needs to configure the Shipping and Delivery or Create A discount that would match the threshold

The current threshold is set to 5000

## Files

- `snippets/free-shipping-progress-bar.liquid` - Main component template
- `assets/free-shipping-progress-bar.js` - JS Component
- `assets/free-shipping-progress-bar.css` - Styling and animation
- `locales/en.default.json` - Translations

## Web Component

The progress bar is implemented as a custom web component `<free-shipping-progress-bar>`.

## Liquid Snippet

The snippet calculates progress (excluding gift cards) and renders the component:

````liquid
{% liquid
  assign threshold_metafield = shop.metafields.custom.free_shipping_threshold
  assign free_shipping_threshold = threshold_metafield

  # Calculate cart total excluding gift cards
  assign cart_total_cents = 0
  for item in cart.items
    assign product_type_lower = item.product.type | downcase
    unless product_type_lower == 'giftcard'
      assign cart_total_cents = cart_total_cents | plus: item.line_price
    endunless
  endfor

  assign cart_total = cart_total_cents | divided_by: 100.0
  assign remaining = free_shipping_threshold | times: 100 | minus: cart_total_cents
  assign progress_percentage = cart_total | times: 100 | divided_by: free_shipping_threshold | floor
%}


## Pure CSS Animation

The progress bar animates using only CSS:

**Liquid sets the CSS variable**:
```liquid
<div
  class="free-shipping-progress-bar__fill"
  style="--progress-width: {{ progress_percentage }}%;"
>
````

**CSS handles the animation**:

```css
.free-shipping-progress-bar__fill {
  width: 0%;
  animation: progressFill 0.8s ease-out forwards;
}

@keyframes progressFill {
  to {
    width: var(--progress-width, 0%);
  }
}
```

## Snippet Rendering

Added to cart drawer (`snippets/cart-drawer.liquid`):

```liquid
<div class="drawer__cart-items-wrapper">
  {% render 'free-shipping-progress-bar' %}
  <table class="cart-items">

  </table>
</div>
```

## How It Works

1. **Check threshold**: Reads `custom.free_shipping_threshold` metafield (hides progress bar if not set)
2. **Calculate remaining**: `threshold - cart total`
3. **Calculate progress**: `(cart total / threshold) * 100`
4. **Set CSS variable**: Liquid sets `--progress-width` to the calculated percentage
5. **CSS animates**: Pure CSS animation from 0% to target percentage
6. **Update on cart change**: Dawn re-renders cart HTML, animation plays again

## Features

- Pure CSS animation
- Smooth animation from 0% to current progress
- Automatically updates when cart changes which leverages dawn theme architecture on cart
- Configurable threshold via metafield
- Accessible (ARIA labels)
- Lightweight and performant
