# WP Self Ad · Self-Service Advertising System for WordPress

> A WordPress plugin that lets visitors **buy ad slots by themselves**. You set up the positions and prices; users pick a slot, pay, and fill in their content — and it goes offline automatically when it expires.
> Built for the Zibll theme, but works on any WordPress site.

English | [简体中文](README.md)

![Version](https://img.shields.io/badge/version-1.3.7-blue)
![WordPress](https://img.shields.io/badge/WordPress-5.5%2B-21759b)
![PHP](https://img.shields.io/badge/PHP-7.4%2B-777bb4)
![License](https://img.shields.io/badge/license-GPL--2.0%2B-green)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

---

## Table of Contents

- [What It Does](#what-it-does)
- [Previews](#previews)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Slot Types](#slot-types)
- [Shortcode Usage](#shortcode-usage)
- [Payment Configuration](#payment-configuration)
- [Admin Features](#admin-features)
- [Database Tables](#database-tables)
- [Directory Structure](#directory-structure)
- [FAQ](#faq)
- [Changelog](#changelog)
- [Development Notes](#development-notes)
- [Author & License](#author--license)
---

## What It Does

Traditional WordPress ad slots rely on pasting code, so selling ads means manual work for the site owner. This plugin turns it into a **self-service transaction flow**:

1. **Site owner** configures ad slots in the admin panel (position, slots per row, monthly price, purchasable durations)
2. **Visitors** see empty slots in the frontend (showing "For rent" and the price) and click one to open a purchase dialog
3. They fill in their ad content → choose a duration → choose a payment method → pay
4. **The ad goes live automatically once payment succeeds**, the slot is occupied and no longer for sale
5. A scheduled task **takes it offline automatically** on expiry, freeing the slot for the next buyer

### Key Features

| Feature | Description |
|---|---|
| 🎯 **Self-service purchase** | Visitors click an empty slot to order — no need to contact the site owner |
| 🎨 **5 built-in slot types** | Large / small / mini / block banners, and text ads — or define your own |
| 💰 **Day-based pricing** | Monthly price × duration ÷ 30, calculated **server-side only** so the frontend can't tamper with it |
| 💳 **Multiple payment channels** | Alipay Official, Epay; payment methods include Alipay / WeChat / QQ Wallet |
| 🔀 **Automatic channel routing** | With multiple channels configured, the plugin picks by priority (Alipay Official first, then Epay) |
| 🔒 **Concurrency-safe** | Order creation runs inside a transaction with row locks — two people can't buy the same slot |
| ⏰ **Auto-expiry** | Checked hourly; expired ads go offline and release their slot |
| 🛠️ **Manual admin control** | Add/edit ads manually, publish or unpublish, adjust durations |
| 📱 **Responsive** | Wraps and rescales on narrow screens without distortion |
| 🌐 **Zero dependencies** | Plain hand-written PHP — no Composer, no Node, no third-party libraries |

---

## Previews

The `docs/` folder contains static preview pages you can open directly in a browser
(they inline the plugin's real CSS, so what you see is what you get):

| Preview file | Contents |
|---|---|
| `preview-文字广告色块风.html` | Color-block style for text ads |
| `preview-支付方式点选式.html` | Payment method / duration picker cards in the purchase dialog |
| `preview-方块横幅排版.html` | Block banner layout — 6 per row |
| `preview-版权信息显示位置.html` | Where the copyright block appears in the admin panel |

> Note: preview page filenames are in Chinese.

---

## Requirements

| Item | Requirement |
|---|---|
| WordPress | 5.5 or higher |
| PHP | **7.4 or higher** |
| MySQL | 5.6 or higher (InnoDB required) |
| Dependencies | **None** (no Composer, no Node) |

> ⚠️ PHP 7.4 is a hard requirement — the plugin uses typed properties and arrow functions.
>
> The WordPress version is a **conservative estimate** (the APIs used, such as `get_sites()`
> and `wp_json_encode()`, have existed since 4.6). The plugin header does not declare
> `Requires at least`, so it will also activate on older sites.

---

## Installation

### Option 1: Upload via admin (recommended)

1. Download the latest `wp-self-ad-v1.3.7.zip`
2. WordPress admin → **Plugins → Add New → Upload Plugin**
3. Choose the zip file → click "Install Now" → click "Activate"

### Option 2: Manual upload

1. Unzip and upload the entire `wp-self-ad` folder to `/wp-content/plugins/`
2. Admin → **Plugins** → find "WP Self Ad - 自助广告系统" → click "Activate"

### What happens on activation

- Creates 3 database tables (`wsa_ad_slots` / `wsa_orders` / `wsa_user_ads`)
- Inserts 5 default ad slots
- Registers the payment callback rewrite rules and flushes them once

> If you see 404 errors or "order creation failed" after installing, go to
> **Settings → Permalinks** and click "Save Changes" to force a rewrite-rule refresh.

---

## Quick Start

Four steps to get running:

### Step 1: Configure ad slots

Admin → **自助广告 → 广告位设置** (Ad Slots)

Each slot accepts:

| Field | Description |
|---|---|
| Slot name | e.g. "Homepage top banner" |
| Slots per row | How many cells per row — determines the frontend column count |
| Monthly price | Pricing basis; actual amount = monthly price × days ÷ 30 |
| Purchasable durations | Comma-separated days offered to users, e.g. `7,30,90` |
| Enabled | When off, the slot is not rendered in the frontend |

### Step 2: Configure payment

Admin → **自助广告 → 支付设置** (Payment), configure at least one channel:

- **Alipay Official**: AppID, app private key, Alipay public key
- **Epay**: API endpoint, merchant PID, merchant key, plus the payment methods you've enabled

> After configuring, click **"易支付接口自检"** (Epay connection self-test). The plugin sends a
> real 0.01 probe order, captures the provider's exact response, and gives you a plain-language
> diagnosis. This is by far the most effective way to debug payment issues.

### Step 3: Place the shortcode

Paste into any page, post, or widget:

```
[wsa_ad slot="banner_large"]
```

### Step 4: Test a purchase in the frontend

Log in with any account and click an empty slot (the one showing "For rent"), then complete the flow.

---

## Slot Types

Five are built in and created automatically on activation:

| Shortcode | Name | Type | Per Row | Desktop Height | Mobile Height |
|---|---|---|---|---|---|
| `[wsa_ad slot="banner_large"]` | Large banner | Image | 1 | 70px | 70px |
| `[wsa_ad slot="banner_small"]` | Small banner | Image | 2 | 70px | 60px |
| `[wsa_ad slot="banner_mini"]` | Mini banner | Image | 3 | 50px | 44px |
| `[wsa_ad slot="banner_block"]` | Block banner | Image | 6 | 80px | 60px |
| `[wsa_ad slot="text_ad"]` | Text ad | Text | 4 | min 38px | min 36px |

**Layout rule**: the plugin always keeps **at least one empty slot** available, and appends a
whole new row once a row is sold out. A 4-per-row text ad slot with 4 sold becomes 8 cells
(the last 4 remain for rent).

### Creating custom slots

You can create any additional slot in the admin panel — just use your own `slot_key`:

```
[wsa_ad slot="your_slot_key"]
```

---

## Shortcode Usage

### Basic

```
[wsa_ad slot="banner_large"]
```

### Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `slot` | No | `banner_large` | The slot's key (`slot_key`) |

> If the key doesn't exist, the shortcode renders **nothing** for visitors. When an administrator
> is logged in, it emits an HTML comment explaining the problem.

---

## Payment Configuration

The plugin uses a **two-layer decoupled** payment model. Once you understand both layers, the whole configuration makes sense.

### Layer 1: Payment method (chosen by the user at checkout)

| Value | Label |
|---|---|
| `alipay` | Alipay |
| `wechat` | WeChat Pay |
| `qq` | QQ Wallet |

### Layer 2: Payment channel (the actual API integrated in the backend)

| Channel | Priority | Notes |
|---|---|---|
| Alipay Official | 10 (higher) | Alipay Open Platform — requires a business account |
| Epay | 20 (lower) | Third-party aggregator — individuals can integrate |

**Routing rule**: after the user picks a payment method, the plugin selects the
lowest-priority channel among those that are **available and support that method**.
The same resolution is shared across all three touchpoints (checkout display, server-side
validation, and the actual API call).

### Two Epay interface styles

Both are supported, with **auto** detection by default:

1. **Cashier API**: `POST {base}/mapi.php` → returns `{code, msg, payurl}` → redirect to `payurl` (modern platforms, recommended)
2. **Page redirect**: `GET {base}/submit.php?...` — sends the browser directly (older Epay)

In auto mode, if `mapi.php` doesn't return JSON, the plugin **falls back** to option 2 automatically.

> Before redirecting, the plugin performs a **cashier pre-check**: on some platforms a
> misconfigured `payurl` returns a JSON error instead of a web page. The pre-check only
> intervenes when the response body is JSON — it surfaces the provider's own error message
> to the user, and lets genuine web pages through.

### How to enter the API endpoint

Just enter the **domain** — the plugin derives `mapi.php` / `submit.php` automatically:

```
https://your-epay-domain.com
```

A full URL (e.g. `https://xxx.com/mapi.php`) also works.

### Extending with new channels

Two filters let you add channels without modifying the plugin source:

```php
// Register an additional payment method
add_filter( 'wsa_payment_types', function ( $types ) {
    $types['my_pay'] = 'My Payment';
    return $types;
} );

// Register an additional payment channel
add_filter( 'wsa_payment_channels', function ( $channels ) {
    $channels['my_channel'] = [
        'label'    => 'My Channel',
        'priority' => 30,
        'handler'  => 'my_channel',
        'check'    => 'my_channel_is_configured',
        'types'    => [ 'my_pay' => 'my_type_code' ],
    ];
    return $channels;
} );
```

---

## Admin Features

A top-level **「自助广告」** menu appears with 6 pages:

| Page | Purpose |
|---|---|
| **Dashboard** | Revenue / order / slot overview, plus the last database error (if any) |
| **Ad Slots** | Configure name, capacity, price, durations, and enabled state per slot |
| **Ads** | View all ads; **manually add**, edit content, publish / unpublish, delete |
| **Orders** | Order list and status (pending / paid / expired / closed / review) |
| **Payment** | Alipay Official and Epay config, plus **connection self-test** and **database repair** tools |
| **About** | Plugin information and author contact details |

### Notes on ad management

- **Admin-created and user-purchased ads share one table**, distinguished by the `source` field (`admin` / `user`)
- Ads have exactly three statuses: `active` / `paused` (taken down by admin) / `expired`
- **Unpublishing immediately frees the slot** and never affects an already-paid order
- Admins **can** change the duration of admin-created ads; user-purchased ads allow **content edits only** — position and duration are locked so they stay consistent with the order
- Publishing an already-expired ad **auto-renews** it based on its duration, so it never "publishes then instantly disappears"

---

## Database Tables

Three tables are created (`{prefix}` is your WordPress table prefix):

### `{prefix}wsa_ad_slots` — Slot definitions

| Column | Description |
|---|---|
| `slot_key` | Slot identifier — this is what the shortcode uses |
| `slot_name` | Display name |
| `slot_type` | `image` or `text` |
| `max_items` | Slots per row |
| `price` | Monthly price |
| `duration_options` | Purchasable durations (stored as a JSON array) |
| `is_enabled` | Whether the slot is active |

### `{prefix}wsa_orders` — Orders

| Column | Description |
|---|---|
| `order_no` | Order number |
| `user_id` / `slot_id` / `slot_position` | Buyer, slot, and position index |
| `duration` | Duration in days |
| `amount` | Actual amount (calculated server-side) |
| `payment_method` | Payment method chosen by the user |
| `payment_channel` | Payment channel actually used |
| `status` | `pending` → `paid` → `expired`; plus `closed` / `review` |
| `image_url` / `text_content` / `link_url` | Ad content |

### `{prefix}wsa_user_ads` — Ads (user-purchased + admin-created)

| Column | Description |
|---|---|
| `source` | `user` or `admin` |
| `slot_position` | Occupied position index |
| `status` | `active` / `paused` / `expired` |
| `duration` | Duration in days (used for renewal) |
| `expires_at` | Expiry time (`2099-12-31 23:59:59` means permanent) |
| `admin_note` | Admin-only note |

> ⚠️ **Uninstalling the plugin permanently deletes all data!**
> When you click "Uninstall" in the WordPress admin, `uninstall.php` runs
> `DROP TABLE` on the three tables above and deletes the `wsa_alipay_config` /
> `wsa_epay_config` options (in a multisite setup, this runs for every subsite).
>
> **Deactivating is safe** — it does not touch your data, and you can re-activate at any time.
> If you only want to stop using it temporarily, choose **Deactivate**, not **Uninstall**.
> Back up your database before uninstalling.

---

## Directory Structure

```
wp-self-ad/
├── wp-self-ad.php              # Entry point: constants, autoloader, activation hooks, assets
├── uninstall.php               # Cleanup on uninstall
├── admin/
│   └── class-wsa-admin.php     # All admin pages and form handling
├── includes/
│   ├── class-wsa-db.php        # Data layer: schema, table creation, column sync, migrations
│   ├── class-wsa-ad-slots.php  # Slots: capacity calculation, shortcode rendering
│   ├── class-wsa-ad-manager.php# Single read/write entry point for ads
│   ├── class-wsa-orders.php    # Orders: creation, payment callbacks, transactions, idempotency
│   ├── class-wsa-payment.php   # Payment: method/channel registry, routing, signing, callbacks
│   ├── class-wsa-ajax.php      # Frontend endpoints (admin-ajax)
│   ├── class-wsa-cron.php      # Scheduled task: hourly expiry check
│   └── class-wsa-about.php     # Admin-side copyright block
└── assets/
    ├── css/wsa-frontend.css    # Frontend styles (responsive)
    ├── js/wsa-frontend.js      # Frontend interactions (purchase dialog)
    └── js/wsa-admin.js         # Admin interactions
```

### Code conventions

- **Plain hand-written PHP** — no Composer, no namespaces
- Class names use a `WSA_` prefix with Camel_Snake_Case, e.g. `WSA_Ad_Manager`
- **Custom autoloader**: `WSA_Xxx_Yyy` → `includes/class-wsa-xxx-yyy.php`
- Data access is centralized in `WSA_DB`; **`WSA_DB::schema()` is the single source of truth for table structure**
- Time always goes through `WSA_DB::now()` (site timezone) — **never** MySQL's `NOW()` in SQL

---

## FAQ

### The slot doesn't appear in the frontend?

1. Check that the `slot` attribute matches the `slot_key` in the admin panel
2. Check that the slot is **enabled**
3. Log in as an administrator and refresh — a wrong key emits an HTML comment explaining the problem

### Payment succeeded but the ad didn't go live?

1. Confirm **Settings → Permalinks** has been saved (payment callbacks rely on rewrite rules)
2. Go to the **Payment** page and run the Epay self-test to confirm the API itself works
3. Check **Orders** to see whether the order status changed to `paid`

### "Order creation failed"?

This is a database-layer issue. Go to **自助广告 → Dashboard** — the page shows the
**raw text of the last database error** (timestamp, message, SQL). You can also go to the
**Payment** page and click "Database Repair" to automatically add any missing tables or columns.

> Regular users only see "Order creation failed" — no database details are leaked.
> Administrators see the full reason.

### An ad disappeared after being sold?

If the site owner manually placed an ad in a later cell (e.g. the 7th cell in a 4-per-row slot),
the page keeps rendering up to that cell. This is **intentional** — otherwise that ad would be
invisible to visitors.

### I changed the slot capacity but the frontend didn't change?

`max_items` is the **slots-per-row** setting. Column rendering and order validation share the
same rules. If it still doesn't take effect, clear your browser cache (`Ctrl + F5`).

### Where are the payment keys stored? Are they safe?

They're stored in the `wp_options` table (`wsa_alipay_config` / `wsa_epay_config`).
Pair this with database access controls and regular backups. **Never** commit keys to a Git repository.

---

## Changelog

| Version | Highlights |
|---|---|
| v1.0.0 | Initial release |
| v1.1.0 | Fixed rewrite flush, concurrent ordering, and payment callback validation |
| v1.2.0 | **Payment refactor**: two-layer decoupling of methods and channels, priority-based routing |
| v1.2.1 | Alipay private key — supports both PKCS#1 and PKCS#8 |
| v1.2.2 | Epay dual-path (mapi.php / submit.php) + cashier pre-check + connection self-test |
| v1.2.3 | **Root-caused table creation** (removed the dbDelta trap) + database errors made visible |
| v1.3.0 | Admin-defined ads with manual publish / unpublish; new Ads admin page |
| v1.3.1 | Fixed block banner layout regression (6 per row — CSS width now subtracts gap) |
| v1.3.2 | Added author copyright block (admin-side only) |
| v1.3.3 | Payment method switched from a dropdown to selectable cards |
| v1.3.4 | Fixed "all cards appear selected" (a DOM-timing pitfall) |
| v1.3.5 | Text ads restyled as color blocks |
| v1.3.6 | Vertical spacing between slots: 10px → 5px |
| **v1.3.7** | Duration switched from a dropdown to selectable cards |

Detailed write-ups explaining **why** each change was made (written for non-technical readers)
live in the numbered documents under `docs/` — in Chinese.

---

## Development Notes

### Current status

The plugin is production-usable. Planned improvements:

- [ ] **Ad review workflow** — ads currently go live immediately after payment; a manual review step is planned
- [ ] **User center** — a "My Ads" page for users to view and renew their ads
- [ ] **Frontend image upload** — currently only a URL field is available
- [ ] **Order rate limiting** — to prevent abuse
- [ ] **Layered refactor** — Repository / Service / Controller
- [ ] **Gateway interface abstraction** — standardize payment channels
- [ ] Template separation, i18n, unit tests and CI

### Contributing

Issues and pull requests are welcome. Before submitting:

1. Make sure the code passes a PHP syntax check (`php -l`)
2. Never commit real payment keys, AppIDs, or other secrets
3. If you change the database schema, bump `WSA_DB_VERSION` and add the corresponding
   migration logic in `WSA_DB::migrate()`

### Security design notes

- Amounts are **calculated server-side only** (`WSA_Ad_Slots::calc_amount()`) — the frontend can't tamper with them
- Order creation uses a **database transaction with row locks**; the lock order is fixed as
  `wsa_ad_slots` first, then `wsa_orders`
- Payment callbacks **verify app_id and amount**; `mark_paid()` is fully idempotent
- Frontend output is escaped; user input goes through `esc_html` / `esc_url` / `esc_attr`
- All admin actions are protected by nonce verification

---

## Author & License

| | |
|---|---|
| **Author** | 墨桅 (JetMast) |
| **Blog** | https://jetmast.com |
| **Email** | Email@5so.cc |
| **QQ** | 910049360 |
| **WeChat** | JetMast |

**License**: [GPL-2.0+](https://www.gnu.org/licenses/gpl-2.0.html)

```
Copyright (C) 2026 墨桅 (JetMast)

This program is free software; you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation; either version 2 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
```

> If this plugin helps you, feel free to drop a note on the [blog](https://jetmast.com).
