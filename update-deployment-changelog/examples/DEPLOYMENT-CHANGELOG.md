# Deployment Changelog

## 2026-06-12: Sibling discount at checkout

### Description
Applies a 10% discount automatically when a cart contains enrolments for two or more children on the same account, using a WooCommerce coupon applied in code.

### Modified Files
- `wp-content/themes/site-child/inc/sibling-discount.php`: new file with the cart rule that applies the coupon
- `wp-content/themes/site-child/functions.php`: loads the new file

### Database Changes
- None

### Manual Actions
- [ ] **Create the `SIBLING10` coupon** (10% off, percentage discount) in WooCommerce on staging, then production: the code applies it by code name and does nothing if it's missing.
- [ ] **Clear the page cache** after deploying so the cart shows the discount line.

## 2026-06-10: Order sync to Monday.com

### Description
New orders are pushed to a Monday.com board through its API, with retries when the API rate-limits.

### Modified Files
- `wp-content/plugins/order-sync/order-sync.php`: new plugin
- `wp-content/plugins/order-sync/includes/class-monday-client.php`: API client with retry logic

### Database Changes
- Adds the `order_sync_last_run` option on activation.

### Manual Actions
- [ ] **Set the `MONDAY_API_TOKEN` environment variable** on staging and production: the plugin can't authenticate without it.
- [ ] **Activate the Order Sync plugin** under Plugins after deploying.
- [ ] **Set the target board ID** under Settings → Order Sync.

## 2026-06-08: Fix tax rounding on checkout

### Description
Tax was rounded per line instead of per order, causing one-cent mismatches on some orders.

### Modified Files
- `wp-content/themes/site-child/inc/tax.php`: round at order level

### Database Changes
- None

### Manual Actions
- None
