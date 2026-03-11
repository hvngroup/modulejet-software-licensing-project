---
name: modulejet-whmcs-module-standard
description: >
  ModuleJET internal development standard for WHMCS Addon Modules.
  Derived from production experience on Pricing Calculator v1.1.0
  (originally hvn_pricing_calculator). Covers PHP backend, Alpine.js
  frontend, pure CSS design system, AJAX patterns, file structure,
  code style, and licensing integration. All prefixes use 'mj'.
author: ModuleJET
version: "2.1.0"
website: https://modulejet.com
---

# ModuleJET WHMCS Module Development Standard

---

## 1. Core Principles

1. **Non-invasive** — Never modify WHMCS core files. Inject via hooks only.
2. **No build step** — No Vue, React, Webpack, npm. Runs directly in browser.
3. **No CDN dependency risk** — All external libs (Alpine.js) must have a local fallback.
4. **No jQuery** — Unless the WHMCS theme forces it. Use native `fetch()` + Alpine.js.
5. **No Bootstrap** — Use the ModuleJET design system (pure CSS + CSS custom properties).
6. **Prefix everything** — All functions, classes, table names, CSS classes, and JS namespaces use `mj_` or `mj-`.
7. **Fail safe** — Every DB operation and every fetch call wrapped in try/catch.
8. **One source of truth** — Module settings read from `tbladdonmodules` via a static cache helper.
9. **License-aware** — Every commercial module integrates `LicenseChecker` for WHMCS Software Licensing verification.
10. **Multi-product ready** — Naming conventions, key prefixes, and code patterns designed for a growing product catalog.

---

## 2. Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Backend | PHP 8.1+, Laravel Capsule ORM | No raw PDO, no mysqli |
| Reactivity | Alpine.js 3.14.8 | CDN + local `/assets/js/vendor/alpine.min.js` fallback |
| CSS | Pure CSS + CSS custom properties | Ant Design–inspired token system |
| AJAX | Native `fetch()` + JSON | Header `X-Requested-With: XMLHttpRequest` |
| Templates | PHP `.php` include files | Variables passed via `include` scope |
| Licensing | WHMCS Software Licensing Addon | Remote verify via `LicenseChecker` class |
| Encoding | ionCube (encoded builds) | Target PHP 8.1+ |
| Prohibited | Vue, React, Bootstrap, jQuery | — |

---

## 3. Branding & Naming Conventions

### 3.1 Brand Identity

| Element | Value |
|---|---|
| Company name | ModuleJET |
| Website | modulejet.com |
| Support email | support@modulejet.com |
| Universal prefix (short) | `mj` |
| License key brand prefix | `MJ` |

### 3.2 Module Naming Pattern

```
Directory:    mj_{product_name}
Main file:    mj_{product_name}.php
Functions:    mj_{product_name}_{function}()
Helpers:      mj_{abbr}_{helper}()          (short functions in hooks.php)
DB table:     tbl_mj_{abbr}_{entity}
CSS prefix:   mj-
CSS file:     mj-{abbr}.css
JS file:      mj-{abbr}.js
JS config:    window.mj{Abbr}Config
JS namespace: window.mj{Abbr}
Alpine:       mj{Abbr}{Component}()
Constant:     MJ_{ABBR}_DIR
Dev constant: MJ_DEV
```

### 3.3 Concrete Examples

**Pricing Calculator** (`abbr` = `pricing` or `pricing` shortened contextually):

| Element | Value |
|---|---|
| Directory | `mj_pricing_calculator/` |
| Main file | `mj_pricing_calculator.php` |
| Functions | `mj_pricing_calculator_config()`, `_activate()`, `_output()` |
| Helpers | `mj_pricing_detectPage()`, `mj_pricing_getSettings()` |
| DB table | `tbl_mj_pricing_presets` |
| CSS prefix | `mj-` (e.g. `.mj-toolbar`, `.mj-btn--primary`) |
| CSS file | `mj-pricing.css` |
| JS file | `mj-pricing.js` |
| JS config | `window.mjPricingConfig` |
| JS namespace | `window.mjPricing` |
| Alpine | `mjPricingToolbar()`, `mjConfigManager()`, `mjConfigToolbar()` |
| Constant | `MJ_PRICING_DIR` |
| whmcs.json | `"name": "mj-pricing-calculator"` |

**Future product — Invoice Manager:**

| Element | Value |
|---|---|
| Directory | `mj_invoice_manager/` |
| DB table | `tbl_mj_invoice_{entity}` |
| CSS file | `mj-invoice.css` |
| JS config | `window.mjInvoiceConfig` |

### 3.4 License Key Prefix Convention

```
MJ-{PRODUCT}-{TIER}-{random_key}

PRODUCT:  2–4 uppercase letters (PC, IM, SM, ...)
TIER:     STD / PRO / UEN (Standard / Professional / Unencoded)
```

| Product | Code | Example Key |
|---|---|---|
| Pricing Calculator | `PC` | `MJ-PC-STD-A1B2C3D4...` |
| Invoice Manager | `IM` | `MJ-IM-PRO-X9Y8Z7...` |
| Server Manager | `SM` | `MJ-SM-UEN-K5L6M7...` |

---

## 4. File Structure

```
modules/addons/mj_{product_name}/
│
├── mj_{product_name}.php      # Main file: config, activate, deactivate, upgrade, output
│                               # + all AJAX handlers + render functions
│
├── hooks.php                   # Hook registrations + page detection + license gate
│
├── lib/                        # Required for LicenseChecker; add classes when logic > ~300 lines
│   └── LicenseChecker.php      # License verification (required for commercial modules)
│
├── assets/
│   ├── js/
│   │   ├── mj-{abbr}.js       # Single JS file (IIFE) — all Alpine components + utils
│   │   └── vendor/
│   │       └── alpine.min.js   # Alpine.js 3.14.8 local fallback
│   └── css/
│       └── mj-{abbr}.css      # Single CSS file — full design system
│
├── templates/
│   ├── admin-dashboard.php     # Main admin page (includes license status card)
│   └── *.php                   # Other admin views (include-based, not Smarty)
│
├── lang/
│   ├── english.php
│   └── vietnamese.php
│
├── whmcs.json                  # WHMCS Marketplace metadata
├── logo.png                    # 128×128
└── LICENSE                     # Proprietary / commercial
```

**Rules:**
- One JS file + one CSS file per module. No splitting unless justified.
- Templates are PHP includes — no Smarty for admin UI.
- `lib/` required for `LicenseChecker.php`. Additional classes only when logic exceeds ~300 lines.
- `assets/js/vendor/` for third-party libs with local fallback.

---

## 5. PHP Backend Rules

### 5.1 File Header

```php
<?php
/**
 * MJ {Product Name} — WHMCS Addon Module
 *
 * {Short description}
 *
 * @package    MJ\{ProductName}
 * @author     ModuleJET
 * @copyright  2026 ModuleJET
 * @license    Proprietary
 * @version    1.0.0
 * @link       https://modulejet.com
 */

defined("WHMCS") or die("Access Denied");

use Illuminate\Database\Capsule\Manager as Capsule;

if (!defined('MJ_{ABBR}_DIR')) {
    define('MJ_{ABBR}_DIR', __DIR__);
}
```

### 5.2 Module Functions

```php
function mj_{product_name}_config(): array { ... }
function mj_{product_name}_activate(): array { ... }
function mj_{product_name}_deactivate(): array { ... }
function mj_{product_name}_upgrade(array $vars): void { ... }
function mj_{product_name}_output(array $vars): void { ... }
```

### 5.3 Config — License Key Field (required)

```php
function mj_{product_name}_config(): array
{
    return [
        'name'        => 'ModuleJET — {Product Name}',
        'description' => '{description}',
        'version'     => '1.0.0',
        'author'      => '<a href="https://modulejet.com" target="_blank">ModuleJET</a>',
        'language'    => 'english',
        'fields'      => [
            'licenseKey' => [
                'FriendlyName' => 'License Key',
                'Type'         => 'text',
                'Size'         => '50',
                'Default'      => '',
                'Description'  => 'Enter your ModuleJET license key. '
                                . '<a href="https://modulejet.com" target="_blank">Purchase</a>',
            ],
            // ... product-specific fields ...
        ],
    ];
}
```

### 5.4 AJAX Handler Pattern

```php
function mj_{product_name}_output(array $vars): void
{
    $action = $_REQUEST['action'] ?? 'index';
    $isXhr  = !empty($_SERVER['HTTP_X_REQUESTED_WITH'])
              && strtolower($_SERVER['HTTP_X_REQUESTED_WITH']) === 'xmlhttprequest';

    if ($isXhr || in_array($action, AJAX_ACTIONS, true)) {
        while (ob_get_level() > 0) ob_end_clean();
        header('Content-Type: application/json; charset=utf-8');
        mj_{product_name}_handleAjax($action);
        exit;
    }

    match ($action) {
        'presets' => mj_{product_name}_renderPresets($vars),
        default   => mj_{product_name}_renderDashboard($vars),
    };
}
```

### 5.5 AJAX Response Format

```php
echo json_encode(['success' => true, 'data' => $result]);
echo json_encode(['success' => false, 'error' => 'Message']);
echo json_encode(['success' => true, 'count' => $n]);
echo json_encode(['success' => true, 'id' => $newId]);
```

### 5.6 Database Rules

```php
// Table naming: tbl_mj_{abbr}_{entity}
Capsule::table('tbl_mj_pricing_presets')->...

// Schema changes always in try/catch
try {
    Capsule::schema()->create('tbl_mj_...', function ($table) { ... });
} catch (\Exception $e) {
    logActivity('ModuleJET {Product}: Activation failed — ' . $e->getMessage());
    return ['status' => 'error', 'description' => $e->getMessage()];
}

// Check column existence before adding in upgrade
if (!Capsule::schema()->hasColumn('tbl_mj_...', 'new_col')) {
    Capsule::schema()->table('tbl_mj_...', function ($table) {
        $table->string('new_col')->default('value');
    });
}
```

### 5.7 Settings Cache Helper

```php
function mj_{abbr}_getSettings(): array
{
    static $cache = null;
    if ($cache !== null) return $cache;
    $cache = [];
    try {
        $rows = Capsule::table('tbladdonmodules')
            ->where('module', 'mj_{product_name}')
            ->get(['setting', 'value']);
        foreach ($rows as $row) $cache[$row->setting] = $row->value;
    } catch (\Exception $e) {}
    return $cache;
}
```

### 5.8 Input Validation

```php
$id     = (int) ($_POST['id'] ?? 0);
$name   = trim($_POST['name'] ?? '');
$pct    = max(0, min(100, (float) ($_POST['discount'] ?? 0)));
$method = in_array($_POST['method'] ?? '', ['none','ceil','floor','round'])
          ? $_POST['method'] : 'round';

if ($id <= 0) { echo json_encode(['success' => false, 'error' => 'Invalid ID']); exit; }
```

### 5.9 Seeding Default Data

```php
foreach ($presets as $preset) {
    $exists = Capsule::table('tbl_mj_...')->where('name', $preset['name'])->exists();
    if (!$exists) Capsule::table('tbl_mj_...')->insert($preset);
}
```

### 5.10 Legacy Migration (HVN → MJ)

```php
// In _activate(): detect old table, copy data, keep old for rollback
if (Capsule::schema()->hasTable('tbl_hvn_pricing_presets')
    && !Capsule::schema()->hasTable('tbl_mj_pricing_presets')) {
    
    Capsule::schema()->create('tbl_mj_pricing_presets', function ($table) { ... });
    $legacy = Capsule::table('tbl_hvn_pricing_presets')->get();
    foreach ($legacy as $row) {
        Capsule::table('tbl_mj_pricing_presets')->insert((array) $row);
    }
    logActivity('MJ Pricing Calculator: Migrated from HVN legacy table.');
}
```

---

## 6. License Integration

### 6.1 LicenseChecker Class

Every commercial module ships `lib/LicenseChecker.php`:

```php
<?php
namespace MJ\{ProductName};

class LicenseChecker
{
    private string $licenseKey;
    private string $localKey;
    private string $whmcsUrl = 'https://modulejet.com/whmcs/';
    private string $licensingSecretKey = '';
    private int $localKeyDays = 15;
    private int $allowCheckFailDays = 5;

    public function __construct(string $licenseKey, string $localKey = '') { ... }
    public function check(): array { /* Based on WHMCS check_sample_code.php */ }
    public function isValid(): bool { return $this->check()['status'] === 'Active'; }

    public function getTier(): string
    {
        if (preg_match('/^MJ-[A-Z]{2,4}-(STD|PRO|UEN)-/', $this->licenseKey, $m)) {
            return match($m[1]) { 'STD'=>'standard', 'PRO'=>'professional', 'UEN'=>'unencoded', default=>'unknown' };
        }
        return 'unknown';
    }

    public function getProduct(): string
    {
        preg_match('/^MJ-([A-Z]{2,4})-/', $this->licenseKey, $m);
        return $m[1] ?? 'UNKNOWN';
    }
}
```

### 6.2 License Gate in Hooks

```php
add_hook('AdminAreaFooterOutput', 1, function($vars) {
    $page = mj_{abbr}_detectPage();
    if (!$page) return '';

    $settings = mj_{abbr}_getSettings();
    $key = $settings['licenseKey'] ?? '';
    if (empty($key)) return mj_{abbr}_renderLicenseNotice('no_key');
    
    $checker = new \MJ\{ProductName}\LicenseChecker($key, $settings['localKey'] ?? '');
    $result = $checker->check();
    
    if (!empty($result['localkey'])) mj_{abbr}_saveLocalKey($result['localkey']);
    if ($result['status'] !== 'Active') return mj_{abbr}_renderLicenseNotice($result['status']);
    
    // License valid → inject toolbar/features ...
});
```

### 6.3 License Notice UI

```php
function mj_{abbr}_renderLicenseNotice(string $status): string
{
    $messages = [
        'no_key'    => 'No license key configured. <a href="https://modulejet.com">Purchase</a>.',
        'Invalid'   => 'Invalid license key. Check module settings.',
        'Expired'   => 'License expired. <a href="https://modulejet.com/clientarea">Renew</a>.',
        'Suspended' => 'License suspended. <a href="https://modulejet.com/clientarea">Contact support</a>.',
    ];
    $msg = $messages[$status] ?? 'License verification failed.';
    return '<div class="mj-license-notice mj-license-notice--' . strtolower($status) . '">'
         . '<strong>ModuleJET</strong>: ' . $msg . '</div>';
}
```

### 6.4 Caching Rules

- Local key: `tbladdonmodules` setting `localKey`
- Remote check: every 15 days
- Grace period: 5 days if unreachable
- After grace: warning banner, **never crash WHMCS**
- Dashboard shows license status card (key masked, status, tier)

---

## 7. Hooks Rules

### 7.1 Hook File Structure

```php
// hooks.php — five sections:
// 1. Helpers (detect page, build config, license helpers)
// 2. AdminAreaHeaderOutput  → CSS for normal pages
// 3. AdminAreaHeadOutput    → CSS + JS for popup pages
// 4. AdminAreaFooterOutput  → license gate + HTML + JS config
// 5. AdminProductConfigFieldsSave → save form data
```

### 7.2 CSS Injection

```php
$ver = defined('MJ_DEV') ? time() : '1.2.0';
return '<link rel="stylesheet" href="' . $base . '/css/mj-{abbr}.css?v=' . $ver . '">';
```

### 7.3 JS Config Object

```php
$jsConfig = [
    'page'          => $page,
    'ajaxUrl'       => 'addonmodules.php?module=mj_{product_name}',
    'presets'       => $presets,
    'currencies'    => $currencies,
    'defaultPreset' => $settings['defaultPreset'] ?? '',
    'token'         => $_SESSION['token'] ?? '',
    'licenseTier'   => $licenseTier,
];
$html .= '<script>window.mj{Abbr}Config=' . json_encode($jsConfig, JSON_HEX_TAG | JSON_HEX_APOS | JSON_HEX_QUOT) . ';</script>';
$html .= '<script src="' . $base . '/js/mj-{abbr}.js?v=' . $ver . '"></script>';
$html .= '<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.14.8/dist/cdn.min.js"></script>';
```

### 7.4 Page Detection

```php
function mj_{abbr}_detectPage(): ?array
{
    $script = basename($_SERVER['SCRIPT_NAME'] ?? '');
    $action = $_REQUEST['action'] ?? '';

    return match(true) {
        $script === 'configproducts.php' && $action === 'edit' => [
            'type' => 'product_edit',
            'product_id' => (int) ($_REQUEST['id'] ?? 0),
            'inject' => ['pricing_calculator', 'config_options_manager'],
        ],
        $script === 'configproductoptions.php'
            && !empty($_REQUEST['manageoptions']) && !empty($_REQUEST['cid']) => [
            'type' => 'config_options',
            'option_id' => (int) $_REQUEST['cid'],
            'inject' => ['pricing_calculator'],
        ],
        $script === 'configaddons.php' && $action === 'manage' => [
            'type' => 'addon_edit',
            'addon_id' => (int) ($_REQUEST['id'] ?? 0),
            'inject' => ['pricing_calculator'],
        ],
        default => null,
    };
}
```

---

## 8. JavaScript Rules

### 8.1 File Structure (IIFE)

```javascript
(function () {
    'use strict';

    var CFG = window.mj{Abbr}Config || {};

    /* ================================================================
       UTILITIES
       ================================================================ */
    var Utils = { ... };

    /* ================================================================
       BUSINESS LOGIC
       ================================================================ */
    var Calc = { ... };

    /* ================================================================
       UNDO MANAGER
       ================================================================ */
    var Undo = { ... };

    /* ================================================================
       ALPINE COMPONENTS
       ================================================================ */
    function mjPricingToolbarData() { return { ... }; }
    function mjConfigManagerData() { return { ... }; }

    /* ================================================================
       REGISTER ALPINE
       ================================================================ */
    document.addEventListener('alpine:init', function () {
        Alpine.data('mjPricingToolbar', mjPricingToolbarData);
        Alpine.data('mjConfigManager', mjConfigManagerData);
    });

    /* ================================================================
       BOOT
       ================================================================ */
    function boot() { ... }
    if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', boot);
    else boot();

    window.mj{Abbr} = { Utils, Calc, Undo };
})();
```

### 8.2 Utils — Required Methods

```javascript
var Utils = {
    round: function (val, method, precision) { ... },
    fmt: function (v) {
        if (v === -1) return '-1.00';
        if (v === 0)  return '0.00';
        return parseFloat(v).toFixed(2);
    },
    parse: function (s) {
        if (!s && s !== 0) return null;
        var n = parseFloat(String(s).replace(/,/g, ''));
        return isNaN(n) ? null : n;
    },
    toast: function (msg, type) { ... },
    highlight: function (el) {
        el.classList.add('mj-changed');
        setTimeout(function () { el.classList.remove('mj-changed'); }, 3000);
    },
    fetchJson: async function (url, opts) {
        opts = opts || {};
        opts.headers = Object.assign({ 'X-Requested-With': 'XMLHttpRequest' }, opts.headers || {});
        try { return await (await fetch(url, opts)).json(); }
        catch (e) { Utils.toast('Network error: ' + e.message, 'error'); return { success: false, error: e.message }; }
    }
};
```

### 8.3 Alpine Component Pattern

```javascript
function mjExampleData() {
    return {
        loading: false, saving: false, items: [],
        init: async function () { await this.load(); },
        load: async function () {
            this.loading = true;
            var r = await Utils.fetchJson(CFG.ajaxUrl + '&action=get_items');
            if (r.success) this.items = r.data || [];
            else Utils.toast('Load failed: ' + (r.error || ''), 'error');
            this.loading = false;
        },
        save: async function () {
            if (!this.validate()) return;
            this.saving = true;
            var r = await Utils.fetchJson(CFG.ajaxUrl + '&action=save_item', {
                method: 'POST',
                body: new URLSearchParams(this.formData()),
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
            });
            if (r.success) { Utils.toast('Saved!', 'success'); await this.load(); }
            else Utils.toast('Error: ' + (r.error || ''), 'error');
            this.saving = false;
        }
    };
}
```

### 8.4 License Tier in JS

```javascript
var tier = CFG.licenseTier || 'standard';
// Show "Powered by ModuleJET" for Standard tier, hide for Pro/Unencoded
```

### 8.5 AJAX Rules

```javascript
// Always fetch() + URLSearchParams for POST
// Always check r.success
// Always set loading/saving state around async
// Disable buttons: <button :disabled="saving">
```

### 8.6 Code Style

```javascript
// Single-line for short methods
cancel: function () { this.editing = null; },

// Multi-line for complex methods
calcCycles: function () {
    var dc = Utils.defaultCurrency();
    if (!dc) { Utils.toast('No default currency.', 'error'); return; }
    // ...
},
```

---

## 9. CSS Rules

### 9.1 Design Tokens

```css
:root {
    --mj-blue: #1677ff;
    --mj-blue-hover: #4096ff;
    --mj-blue-bg: #e6f4ff;
    --mj-blue-border: #91caff;
    --mj-green: #52c41a;
    --mj-green-active: #389e0d;
    --mj-green-bg: #f6ffed;
    --mj-red: #ff4d4f;
    --mj-red-bg: #fff2f0;
    --mj-gold: #faad14;
    --mj-gold-bg: #fffbe6;

    --mj-text: rgba(0,0,0,0.88);
    --mj-text-secondary: rgba(0,0,0,0.65);
    --mj-text-tertiary: rgba(0,0,0,0.45);
    --mj-text-quaternary: rgba(0,0,0,0.25);

    --mj-bg: #ffffff;
    --mj-bg-layout: #f5f5f5;
    --mj-border: #d9d9d9;
    --mj-border-light: #f0f0f0;

    --mj-radius: 8px;
    --mj-radius-sm: 6px;
    --mj-radius-lg: 12px;

    --mj-font: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Arial, sans-serif;
    --mj-font-xs: 11px;
    --mj-font-sm: 12px;
    --mj-font-base: 14px;

    --mj-transition: all 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
}
```

### 9.2 Class Naming

```
mj-{block}               → .mj-toolbar
mj-{block}-{element}     → .mj-toolbar-title
mj-{block}--{modifier}   → .mj-btn--primary
mj-{utility}             → .mj-mt-16, .mj-text-muted, .mj-hidden
```

### 9.3 License Notice Styles (required)

```css
.mj-license-notice {
    padding: 12px 16px;
    border-radius: var(--mj-radius);
    font-size: var(--mj-font-base);
    margin-bottom: 16px;
}
.mj-license-notice--no_key   { background: var(--mj-gold-bg); border: 1px solid var(--mj-gold); }
.mj-license-notice--invalid  { background: var(--mj-red-bg); border: 1px solid var(--mj-red); }
.mj-license-notice--expired  { background: var(--mj-red-bg); border: 1px solid var(--mj-red); }
.mj-license-notice--suspended { background: var(--mj-gold-bg); border: 1px solid var(--mj-gold); }
```

### 9.4 Button Variants (required)

```css
.mj-btn  .mj-btn--primary  .mj-btn--success  .mj-btn--default
.mj-btn--danger  .mj-btn--warning  .mj-btn--text
.mj-btn--xs (24px)  .mj-btn--sm (28px)  .mj-btn--lg (40px)  .mj-btn--block
```

### 9.5 Utilities (required)

```css
.mj-mt-8 / .mj-mt-12 / .mj-mt-16   .mj-mb-8 / .mj-mb-16
.mj-text-muted  .mj-text-danger  .mj-text-sm  .mj-text-xs
.mj-hidden
```

### 9.6 Toast

```css
.mj-toast-container  .mj-toast  .mj-toast--visible
.mj-toast--success / --error / --info / --warning
/* Fixed top-center, auto-dismiss 3.5s */
```

### 9.7 File Organization

```css
/* 1. :root tokens        */
/* 2. License notice       */
/* 3. Primary components   */
/* 4. Inputs               */
/* 5. Buttons              */
/* 6. Feedback (toast)     */
/* 7. Layout (card, tabs)  */
/* 8. Page-specific        */
/* 9. Utilities            */
/* 10. Responsive          */
```

### 9.8 No Inline Styles

```javascript
h += '<span style="font-weight:600">';   // ❌
h += '<span class="mj-label--bold">';    // ✅
```

---

## 10. Template Rules

### 10.1 Header

```php
<?php
/**
 * ModuleJET {Product} — {Template Name}
 * Variables: $var1, $var2, ...
 */
defined("WHMCS") or die("Access Denied");
?>
```

### 10.2 HTML + Alpine

```php
<?php echo htmlspecialchars($value); ?>

<!-- ✅ Component reads from window config -->
<div x-data="mjPricingToolbar()">

<!-- ❌ Inline JSON -->
<div x-data="{ items: <?php echo json_encode($items); ?> }">
```

---

## 11. AJAX Conventions

### 11.1 URL

```
addonmodules.php?module=mj_{product_name}&action={action}
```

### 11.2 Actions

```
get_{entity}  save_{entity}  delete_{entity}  quick_create_{entity}  assign_{entity}
```

### 11.3 Responses

```json
{ "success": true, "data": [...] }
{ "success": true, "id": 123 }
{ "success": true, "count": 42 }
{ "success": false, "error": "Message" }
```

---

## 12. Database Conventions

### 12.1 Table Naming

```sql
tbl_mj_{abbr}_{entity}
-- Examples: tbl_mj_pricing_presets, tbl_mj_invoice_jobs
```

### 12.2 Required Columns

```sql
id         INT UNSIGNED AUTO_INCREMENT PRIMARY KEY
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

### 12.3 Upgrade Pattern

```php
function mj_{product_name}_upgrade(array $vars): void
{
    $v = $vars['version'];
    if (version_compare($v, '1.1.0', '<')) {
        try {
            if (!Capsule::schema()->hasColumn('tbl_mj_...', 'new_col')) {
                Capsule::schema()->table('tbl_mj_...', function ($table) {
                    $table->string('new_col')->default('value');
                });
            }
            logActivity('ModuleJET {Product}: Upgraded to v1.1.0.');
        } catch (\Exception $e) {
            logActivity('ModuleJET {Product}: Upgrade failed — ' . $e->getMessage());
        }
    }
}
```

---

## 13. Build & Distribution

### 13.1 Encoded Build

ionCube encode: `mj_{product_name}.php`, `hooks.php`, `lib/LicenseChecker.php`

Keep plain: `assets/`, `templates/`, `lang/`, `whmcs.json`, `logo.png`, `LICENSE`

### 13.2 Unencoded Build

All files plain PHP. Same codebase, proprietary license.

### 13.3 ionCube Settings

```
Target: PHP 8.1+    Obfuscation: Level 4    Expiry: None    Callback: None
```

### 13.4 whmcs.json Template

```json
{
    "schema": "1.0",
    "type": "whmcs-addons",
    "name": "mj-{product-name}",
    "license": "proprietary",
    "category": "utilities",
    "description": {
        "name": "ModuleJET — {Product Name}",
        "tagline": "{Short description}.",
        "long": "{Detailed description}."
    },
    "logo": { "filename": "logo.png" },
    "support": {
        "homepage": "https://modulejet.com",
        "docs_url": "https://modulejet.com/docs/{product}",
        "email": "support@modulejet.com"
    },
    "authors": [{ "name": "ModuleJET", "homepage": "https://modulejet.com" }],
    "min_whmcs_version": "8.0.0",
    "max_whmcs_version": "9.99.99",
    "min_php_version": "8.1.0"
}
```

---

## 14. Security Checklist

- [ ] `defined("WHMCS") or die("Access Denied")` first line every PHP file
- [ ] All `$_POST` / `$_GET` cast and validated
- [ ] No SQL concatenation — Capsule binding only
- [ ] All HTML through `htmlspecialchars()`
- [ ] AJAX returns JSON only
- [ ] Sensitive data via `encrypt()` / `decrypt()`
- [ ] No stack traces in production
- [ ] WHMCS token in form submissions
- [ ] License key in settings, not hardcoded
- [ ] Local key not exposed to clients
- [ ] ionCube encoded files don't leak source

---

## 15. Anti-Patterns

| Never | Use instead |
|---|---|
| `style="..."` in JS | CSS class |
| jQuery `$.ajax()` | `fetch()` + `URLSearchParams` |
| Bootstrap classes | MJ CSS design system |
| Smarty for admin UI | PHP include templates |
| Multiple JS files | Single IIFE |
| `$_REQUEST` | `$_POST` or `$_GET` |
| Modify WHMCS core | Hooks + Modules |
| Raw SQL | Capsule ORM |
| `var_dump()` | `logActivity()` |
| `hvn_` / `hvn-` prefix | `mj_` / `mj-` prefix |
| Skip license check | License gate required |
| Hardcoded license key | `licenseKey` config field |

---

## 16. Documentation

### README Structure

```markdown
# ModuleJET — {Product Name}
## 1. Executive Summary
## 2. Features
## 3. Compatibility
## 4. Installation
## 5–N. Feature descriptions
## N+1. Admin Settings / Database Schema
## N+2. Technical Architecture (files, hooks, AJAX)
## N+3. Development Status
## N+4. License
## N+5. Author
```

### Code Comments

```javascript
// Explain WHY, not WHAT
/* ================================================================
   SECTION NAME
   ================================================================ */
```

---

## 17. Quick Start — New Module

```
□ Create modules/addons/mj_{product_name}/
□ Create mj_{product_name}.php (config with licenseKey, activate, deactivate, upgrade, output)
□ Create hooks.php (5 sections, license gate in FooterOutput)
□ Create lib/LicenseChecker.php (copy template, update namespace + product code)
□ Create assets/css/mj-{abbr}.css (token system, mj- prefix)
□ Create assets/js/mj-{abbr}.js (IIFE, Utils, Alpine components)
□ Copy assets/js/vendor/alpine.min.js
□ Create templates/admin-dashboard.php (license status card)
□ Create lang/english.php
□ Create whmcs.json (ModuleJET branding)
□ Create logo.png (128×128)
□ Verify: zero hvn_ or hvn- references
□ Test: activate → deactivate → re-activate
□ Test: upgrade path
□ Test: license check (Active, Invalid, Expired, Suspended, no key)
□ Test: encoded build PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x
□ Write README.md
□ Prepare build scripts (encoded + unencoded)
```

---

*ModuleJET — https://modulejet.com*  
*Standard v2.1.0 — March 2026*  
*Prefix: `mj_` / `mj-` / `MJ` toàn bộ*  
*Supersedes: HVN WHMCS Module Development Standard v1.0.0*
