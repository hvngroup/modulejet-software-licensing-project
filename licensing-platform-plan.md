# ModuleJET — Licensing Platform Master Plan

**Thương hiệu:** ModuleJET — modulejet.com  
**Email:** support@modulejet.com  
**Mục đích:** Xây dựng platform licensing + distribution cho tất cả sản phẩm WHMCS module của ModuleJET  
**Sản phẩm đầu tiên:** Pricing Calculator  
**Ngày lập:** 11/03/2026 — **v1.1**

---

## 1. Tầm Nhìn

ModuleJET là thương hiệu bán WHMCS addon modules thương mại. Platform licensing cần đáp ứng:

- Bán nhiều sản phẩm module khác nhau trên cùng một hệ thống
- Mỗi sản phẩm có 3 tiers (Standard / Professional / Open Source) × 3 cycles (Monthly / Annually / Lifetime) = tối đa 8 SKU
- Hệ thống billing, client portal, support, downloads dùng chung cho tất cả sản phẩm
- Quy trình thêm sản phẩm mới < 1 ngày (chỉ tạo products + upload files)
- License verification code tái sử dụng 100% giữa các module

---

## 2. Kiến Trúc Platform

```
┌─────────────────────────────────────────────────────────────────┐
│                     KHÁCH HÀNG (N instances)                     │
│                                                                  │
│  WHMCS #1: mj_pricing_calculator ──┐                            │
│  WHMCS #2: mj_invoice_manager ─────┤  license check (HTTPS)     │
│  WHMCS #3: mj_pricing_calculator ──┤                            │
│  WHMCS #4: mj_server_manager ──────┘                            │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                  MODULEJET.COM (1 WHMCS Instance)                │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Software Licensing Addon                       │ │
│  │  - License key generation (MJ-{PROD}-{TIER}-...)          │ │
│  │  - Remote verification API                                 │ │
│  │  - License Manager (admin)                                 │ │
│  │  - Public verification page                                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │ Product Group:   │  │ Product Group:   │  │ Product Group: │ │
│  │ Pricing Calc     │  │ Invoice Manager  │  │ Server Manager │ │
│  │ 8 SKUs           │  │ 8 SKUs           │  │ 8 SKUs         │ │
│  └─────────────────┘  └─────────────────┘  └────────────────┘ │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │ Payment Gateway  │  │ Client Portal   │  │ Download Mgr   │ │
│  │ PayPal + Stripe  │  │ (shared)        │  │ (per-product)  │ │
│  └─────────────────┘  └─────────────────┘  └────────────────┘ │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │ Support Tickets  │  │ Knowledgebase   │  │ Email Templ.   │ │
│  │ (per-product     │  │ (per-product    │  │ (shared,       │ │
│  │  departments)    │  │  categories)    │  │  personalized) │ │
│  └─────────────────┘  └─────────────────┘  └────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. License Key Convention (Global)

### 3.1 Format

```
MJ-{PRODUCT}-{TIER}-{32_char_random}

MJ          Cố định — brand prefix
{PRODUCT}   3 uppercase letters — mã sản phẩm (exactly 3 chars)
{TIER}      STD / PRO / OSS
{random}    32 ký tự do WHMCS generate
```

### 3.2 Product Registry

| Code | Sản phẩm | Trạng thái |
|---|---|---|
| `PRC` | Pricing Calculator | Sản phẩm đầu tiên |
| `INV` | Invoice Manager | Planned |
| `SVM` | Server Manager | Planned |
| `EML` | Email Manager | Idea |
| `DMN` | Domain Manager | Idea |
| `BKP` | Backup Manager | Idea |

Khi tạo sản phẩm mới, đăng ký mã **đúng 3 ký tự** vào registry này để tránh trùng.

### 3.3 Tier Definitions (áp dụng cho MỌI sản phẩm)

| Tier | Key | Encoding | Domain | Sửa code | Redistribute | White-label | Support |
|---|---|---|---|---|---|---|---|
| Standard | `STD` | ionCube | 1 domain | ❌ | ❌ | ❌ | Email 48h |
| Professional | `PRO` | ionCube | Unlimited | ❌ | ❌ | ❌ | Priority 24h |
| Open Source | `OSS` | Plain PHP | Unlimited | ✅ | ❌ | ✅ | Priority 24h |

> **Open Source = proprietary license.** Code không mã hóa nhưng cấm phân phối lại.

### 3.4 Billing Cycles (áp dụng cho MỌI sản phẩm)

| Cycle | Thanh toán | Support/Updates |
|---|---|---|
| Monthly | Hàng tháng | Khi còn active |
| Annually | Hàng năm | Khi còn active |
| Lifetime | Một lần | **Trọn đời** |

**Lưu ý:** Open Source không có Monthly (giá trị quá cao cho monthly billing).

### 3.5 SKU Matrix chuẩn per product (8 SKUs)

| # | SKU Pattern | Billing | Tier |
|---|---|---|---|
| 1 | `{Product} Monthly Standard` | Monthly | STD |
| 2 | `{Product} Annually Standard` | Annually | STD |
| 3 | `{Product} Lifetime Standard` | One Time | STD |
| 4 | `{Product} Monthly Professional` | Monthly | PRO |
| 5 | `{Product} Annually Professional` | Annually | PRO |
| 6 | `{Product} Lifetime Professional` | One Time | PRO |
| 7 | `{Product} Annually Open Source` | Annually | OSS |
| 8 | `{Product} Lifetime Open Source` | One Time | OSS |

---

## 4. Pricing Framework

### 4.1 Pricing Template (base rate × product multiplier)

Mỗi sản phẩm có thể có giá khác nhau, nhưng tuân theo tỷ lệ chuẩn:

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | `1× base` | `8× base` (~33% off yearly) | `Annually × 2.5 = 20× base` |
| Professional | `3× base` | `24× base` | `Annually × 2.5 = 60× base` |
| Open Source | — | `36× base` | `Annually × 2.5 = 90× base` |

**Ví dụ base = $4.99:**

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | $4.99 | $39.99 | $99.99 |
| Professional | $14.99 | $119.99 | $299.99 |
| Open Source | — | $179.99 | $449.99 |

Sản phẩm phức tạp hơn (Server Manager) có thể dùng base cao hơn ($7.99, $9.99...).

### 4.2 Support/Updates Policy (global)

| Billing | Policy |
|---|---|
| Monthly / Annually | Bao gồm khi subscription active. Hết hạn = mất download + support |
| Lifetime | **Trọn đời.** Không cần addon riêng. Lifetime product = always Active |

Không dùng Support/Updates Addon riêng cho bất kỳ product nào — đơn giản hóa quản lý.

---

## 5. WHMCS Instance trên modulejet.com

### 5.1 One-time Setup (làm 1 lần, dùng cho tất cả products)

| Hạng mục | Chi tiết |
|---|---|
| WHMCS License | Self-hosted hoặc Cloud Growth/Expansion |
| Software Licensing Addon | $99.95 one-time, mua từ WHMCS Marketplace |
| Payment Gateways | PayPal + Stripe (bật sẵn cho tất cả products) |
| Support Departments | 1 department per product (tạo khi ra product mới) |
| Email Templates | Shared templates, dùng merge fields để personalize per product |
| Client Area | 1 portal cho tất cả products — khách quản lý mọi license tại 1 chỗ |
| Admin Theme | WHMCS default |
| Public License Verification | Enable — `modulejet.com/whmcs/index.php?m=licensing` |

### 5.2 Licensing Addon Config (global)

| Setting | Value |
|---|---|
| Maximum Allowed Reissues | `3` |
| Auto Logs Prune | `90 days` |
| Public Verification | ✅ Enable |
| Admin Roles | Full Access cho owner |

### 5.3 Per-Product Setup (lặp lại mỗi khi ra sản phẩm mới)

```
Checklist thêm sản phẩm mới:
□ Đăng ký product code vào registry (e.g. INV = Invoice Manager)
□ Tạo Product Group: "ModuleJET — {Product Name}"
□ Tạo 8 products theo SKU matrix (section 3.5)
□ Set key prefix cho mỗi product: MJ-{CODE}-STD- / MJ-{CODE}-PRO- / MJ-{CODE}-OSS-
□ Set domain rules: STD = no domain conflict, PRO/OSS = allow domain conflict
□ Set pricing theo framework (section 4.1)
□ Cấu hình upgrade paths (STD→PRO, Monthly→Annually→Lifetime, Encoded→Open Source)
□ Upload download files (encoded + Open Source ZIPs)
□ Tạo support department: "{Product Name} Support"
□ Tạo knowledgebase category: "{Product Name}"
□ Viết KB articles: Installation, Configuration, FAQ, Troubleshooting
□ Tạo product landing page trên website
□ Test end-to-end: order → pay → download → install → activate
```

**Estimated time cho mỗi product mới: < 1 ngày** (vì infrastructure đã sẵn).

### 5.4 Product Group Structure trong WHMCS

```
Product Groups
├── ModuleJET — Pricing Calculator
│   ├── PRC Monthly Standard ($4.99/mo)
│   ├── PRC Annually Standard ($39.99/yr)
│   ├── PRC Lifetime Standard ($99.99)
│   ├── PRC Monthly Professional ($14.99/mo)
│   ├── PRC Annually Professional ($119.99/yr)
│   ├── PRC Lifetime Professional ($299.99)
│   ├── PRC Annually Open Source ($179.99/yr)
│   └── PRC Lifetime Open Source ($449.99)
│
├── ModuleJET — Invoice Manager        ← tạo khi ra product mới
│   ├── INV Monthly Standard
│   ├── INV Annually Standard
│   └── ... (8 SKUs)
│
└── ModuleJET — Server Manager         ← tạo khi ra product mới
    └── ... (8 SKUs)
```

### 5.5 Download Manager

Mỗi product có 2 download files:

| File | Gắn vào | Tên file convention |
|---|---|---|
| Encoded ZIP | STD + PRO products | `mj-{product}-encoded-v{version}.zip` |
| Open Source ZIP | OSS products | `mj-{product}-opensource-v{version}.zip` |

WHMCS enforce: chỉ Active product mới download được. Upload file mới khi release version mới.

### 5.6 Upgrade Paths (chuẩn cho mọi product)

```
Within same product:
  STD Monthly  →  STD Annually  →  STD Lifetime
  PRO Monthly  →  PRO Annually  →  PRO Lifetime
  OSS Annually →  OSS Lifetime
  
Cross-tier:
  STD (any cycle)  →  PRO (any cycle)
  STD (any cycle)  →  OSS (any cycle)
  PRO (any cycle)  →  OSS (any cycle)

WHMCS auto-prorates khi upgrade.
```

### 5.7 Email Templates

Tạo 1 bộ shared templates, dùng WHMCS merge fields:

| Template | Trigger | Key merge fields |
|---|---|---|
| License Welcome | Product Create | `{$service_product_name}`, `{$service_dedicatedip}` (license key), download link |
| License Renewal Reminder | Before expiry | Product name, expiry date, renew link |
| License Suspended | Overdue | Product name, invoice link |
| License Terminated | Termination | Product name, re-order link |

---

## 6. Shared LicenseChecker (Code Library)

### 6.1 Thiết kế

Một class PHP duy nhất tái sử dụng cho **mọi** module ModuleJET. Mỗi module copy file này vào `lib/LicenseChecker.php`, chỉ thay namespace.

```php
<?php
namespace MJ\{ProductName};

class LicenseChecker
{
    private string $licenseKey;
    private string $localKey;
    
    // ── Config ──────────────────────────────────────
    private string $whmcsUrl          = 'https://modulejet.com/whmcs/';
    private string $licensingSecretKey = '{per-product-secret}';
    private int    $localKeyDays       = 15;    // remote check mỗi 15 ngày
    private int    $allowCheckFailDays = 5;     // grace 5 ngày nếu server down
    
    // ── Constructor ─────────────────────────────────
    public function __construct(string $licenseKey, string $localKey = '')
    {
        $this->licenseKey = trim($licenseKey);
        $this->localKey   = trim($localKey);
    }
    
    // ── Primary Methods ─────────────────────────────
    
    /**
     * Full license check (remote + local key fallback).
     * Based on WHMCS check_sample_code.php.
     * @return array{status:string, localkey:string, message:string}
     */
    public function check(): array { /* ... */ }
    
    public function isValid(): bool
    {
        return $this->check()['status'] === 'Active';
    }
    
    // ── Key Parsing ─────────────────────────────────
    
    /**
     * Parse tier from key prefix: MJ-{PROD}-{TIER}-...
     */
    public function getTier(): string
    {
        if (preg_match('/^MJ-[A-Z]{3}-(STD|PRO|OSS)-/', $this->licenseKey, $m)) {
            return match($m[1]) {
                'STD' => 'standard',
                'PRO' => 'professional',
                'OSS' => 'opensource',
                default => 'unknown',
            };
        }
        return 'unknown';
    }
    
    /**
     * Parse product code from key: MJ-{PROD}-...
     */
    public function getProduct(): string
    {
        preg_match('/^MJ-([A-Z]{3})-/', $this->licenseKey, $m);
        return $m[1] ?? 'UNKNOWN';
    }
    
    /**
     * Validate key belongs to the expected product.
     */
    public function isCorrectProduct(string $expectedCode): bool
    {
        return $this->getProduct() === strtoupper($expectedCode);
    }
}
```

### 6.2 Tích hợp vào module

```php
// hooks.php — pattern chung cho MỌI module
$settings = mj_{abbr}_getSettings();
$key      = $settings['licenseKey'] ?? '';

if (empty($key)) {
    return mj_{abbr}_renderLicenseNotice('no_key');
}

$checker = new \MJ\{ProductName}\LicenseChecker($key, $settings['localKey'] ?? '');

// Validate correct product (tránh dùng key module khác)
if (!$checker->isCorrectProduct('{PRODUCT_CODE}')) {
    return mj_{abbr}_renderLicenseNotice('wrong_product');
}

$result = $checker->check();
if (!empty($result['localkey'])) {
    mj_{abbr}_saveLocalKey($result['localkey']);
}
if ($result['status'] !== 'Active') {
    return mj_{abbr}_renderLicenseNotice($result['status']);
}

// License valid → inject features
```

### 6.3 License Notice (shared CSS pattern)

```css
/* Mọi module dùng chung design pattern */
.mj-license-notice { padding:12px 16px; border-radius:var(--mj-radius); margin-bottom:16px; }
.mj-license-notice--no_key       { background:var(--mj-gold-bg); border:1px solid var(--mj-gold); }
.mj-license-notice--invalid      { background:var(--mj-red-bg);  border:1px solid var(--mj-red); }
.mj-license-notice--expired      { background:var(--mj-red-bg);  border:1px solid var(--mj-red); }
.mj-license-notice--suspended    { background:var(--mj-gold-bg); border:1px solid var(--mj-gold); }
.mj-license-notice--wrong_product { background:var(--mj-red-bg); border:1px solid var(--mj-red); }
```

### 6.4 Caching Rules (global)

| Rule | Value | Lý do |
|---|---|---|
| Local key storage | `tbladdonmodules` setting `localKey` | Đã có sẵn, không cần custom table |
| Remote check interval | 15 ngày | Giảm load cho server modulejet.com |
| Grace period | 5 ngày | Tránh disable module khi server tạm down |
| Failure behavior | Warning banner, features vẫn chạy trong grace | Không bao giờ crash WHMCS |
| After grace period | Warning banner, disable features mới | Vẫn không crash |

---

## 7. ionCube Encoding (Global Build System)

### 7.1 Encoding Strategy

| File type | Encode? | Lý do |
|---|---|---|
| Main `.php` (module file) | ✅ | Core logic |
| `hooks.php` | ✅ | License gate + injection logic |
| `lib/LicenseChecker.php` | ✅ | License verification |
| `lib/*.php` (other classes) | ✅ | Business logic |
| `assets/js/*.js` | ❌ | Browser runtime, obfuscate nếu muốn |
| `assets/css/*.css` | ❌ | Styling |
| `assets/js/vendor/*` | ❌ | Third-party |
| `templates/*.php` | ❌ | HTML only, no sensitive logic |
| `lang/*.php` | ❌ | Translation strings |
| `whmcs.json` | ❌ | Metadata |

### 7.2 Build Script Template (reusable)

```bash
#!/bin/bash
# build.sh — Universal build script for any MJ module
# Usage: ./build.sh <module_dir_name> <version>
# Example: ./build.sh mj_pricing_calculator 1.2.0

MODULE="$1"
VERSION="$2"
SRC="./src/${MODULE}"
IONCUBE="/path/to/ioncube_encoder"

if [ -z "$MODULE" ] || [ -z "$VERSION" ]; then
    echo "Usage: ./build.sh <module_name> <version>"
    exit 1
fi

ABBR=$(echo "$MODULE" | sed 's/^mj_//' | cut -d_ -f1)  # e.g. "pricing"

# ── Encoded build ───────────────────────────────
OUT_ENC="./build/encoded/${MODULE}"
rm -rf "./build/encoded" && mkdir -p "$OUT_ENC"
cp -r "$SRC/"* "$OUT_ENC/"

# Find all PHP files to encode (skip templates, lang, vendor)
find "$SRC" -name "*.php" \
    ! -path "*/templates/*" \
    ! -path "*/lang/*" \
    ! -path "*/vendor/*" \
    -print0 | xargs -0 $IONCUBE \
        --php81 --optimize max --obfuscate all \
        --into "$OUT_ENC"

cd "./build/encoded"
zip -r "../mj-${ABBR}-encoded-v${VERSION}.zip" "${MODULE}/"
cd ../..

# ── Open Source build ─────────────────────────────
OUT_OSS="./build/opensource/${MODULE}"
rm -rf "./build/opensource" && mkdir -p "$OUT_OSS"
cp -r "$SRC/"* "$OUT_OSS/"
cp ./licenses/COMMERCIAL.txt "$OUT_OSS/LICENSE"

cd "./build/opensource"
zip -r "../mj-${ABBR}-opensource-v${VERSION}.zip" "${MODULE}/"
cd ../..

echo "✅ Encoded:     build/mj-${ABBR}-encoded-v${VERSION}.zip"
echo "✅ Open Source:  build/mj-${ABBR}-opensource-v${VERSION}.zip"
```

### 7.3 Test Matrix (áp dụng cho mọi product)

| PHP | WHMCS | ionCube Loader | Must pass |
|---|---|---|---|
| 8.1 | 8.x | 13.0.2+ | ✅ |
| 8.2 | 8.x / 9.x | 13.0.2+ | ✅ |
| 8.3 | 9.x | 14.4.0+ | ✅ |

---

## 8. Website modulejet.com

### 8.1 Site Structure

```
modulejet.com/
│
├── / (Home)
│   ├── Hero: "WHMCS Modules That Just Work"
│   ├── Product cards (grid — grows as catalog grows)
│   └── Trust signals, testimonials
│
├── /products/
│   ├── /pricing-calculator/          ← per-product landing page
│   │   ├── Features & screenshots
│   │   ├── Pricing table (3 tiers)
│   │   ├── Documentation link
│   │   ├── Changelog
│   │   └── "Buy Now" → WHMCS order form
│   │
│   ├── /invoice-manager/             ← thêm khi ra sản phẩm mới
│   └── /server-manager/
│
├── /docs/
│   ├── /pricing-calculator/          ← per-product docs
│   │   ├── installation
│   │   ├── configuration
│   │   ├── faq
│   │   └── troubleshooting
│   └── /invoice-manager/
│
├── /clientarea/                      ← WHMCS Client Area
│   ├── My Licenses (all products, 1 dashboard)
│   ├── Downloads
│   ├── Support Tickets
│   └── Invoices
│
├── /support/                         ← WHMCS ticket submission
├── /verify/                          ← Public license verification
│
├── /legal/
│   ├── terms-of-service
│   ├── privacy-policy
│   ├── refund-policy
│   └── acceptable-use
│
└── /about/
```

### 8.2 Per-Product Page Template

Mỗi product page tuân theo layout chuẩn:

```
1. Hero section (product name + tagline + screenshot)
2. Pain point → Solution (before/after)
3. Feature list với icons
4. Pricing table (Standard / Professional / Open Source)
5. FAQ accordion
6. CTA "Buy Now" → WHMCS order form
7. Compatibility table (WHMCS versions, PHP versions)
```

### 8.3 Legal Pages

| Page | Nội dung chính |
|---|---|
| Terms of Service | Licensing terms, Open Source ≠ open source, no redistribution, termination rights |
| Privacy Policy | Data collected (billing info, license checks), GDPR compliance |
| Refund Policy | 30 ngày money-back, điều kiện (không áp dụng nếu đã reissue >1 lần) |
| Acceptable Use | Cấm crack/redistribute, cấm sử dụng cho mục đích bất hợp pháp |

---

## 9. Support System

### 9.1 Department Structure

```
Support Departments (tạo trong WHMCS):
├── Pricing Calculator Support      ← tạo khi launch PRC
├── Invoice Manager Support         ← tạo khi launch INV
├── Server Manager Support          ← tạo khi launch SVM
├── Billing & Licensing             ← chung, tạo ngay từ đầu
└── Pre-Sales Questions             ← chung, tạo ngay từ đầu
```

### 9.2 SLA per Tier

| Tier | Response Time | Channels |
|---|---|---|
| Standard | 48h business hours | Email/ticket |
| Professional | 24h business hours | Email/ticket |
| Open Source | 24h business hours | Email/ticket |
| No active license | Best effort | Email/ticket (limited) |

### 9.3 Knowledgebase

```
KB Categories:
├── Pricing Calculator
│   ├── Installation Guide
│   ├── Configuration Guide
│   ├── How to Use the Toolbar
│   ├── FAQ
│   └── Troubleshooting
├── Invoice Manager
│   └── ... (same pattern)
├── General
│   ├── How to Install ionCube Loader
│   ├── License Activation Guide
│   ├── How to Reissue a License
│   ├── How to Upgrade Your License
│   └── Billing FAQ
```

---

## 10. Triển Khai Theo Phases

### Phase 0 — Platform Foundation (Tuần 1–2) ← LÀM MỘT LẦN

Xây dựng toàn bộ infrastructure, không gắn với product cụ thể nào.

| # | Task | Ưu tiên | Est. |
|---|---|---|---|
| 0.1 | Cài WHMCS trên modulejet.com (hosting, SSL, domain) | Critical | 1 ngày |
| 0.2 | Mua + activate Software Licensing Addon | Critical | 0.5 ngày |
| 0.3 | Cấu hình Licensing Addon (reissues, prune, verification) | Critical | 0.5 ngày |
| 0.4 | Setup PayPal gateway | Critical | 0.5 ngày |
| 0.5 | Setup Stripe gateway | Critical | 0.5 ngày |
| 0.6 | Customize email templates (welcome, suspended, terminated, renewal) | High | 1 ngày |
| 0.7 | Tạo support departments: "Billing & Licensing", "Pre-Sales" | High | 0.5 ngày |
| 0.8 | Tạo KB category "General" + viết 5 general articles | High | 1 ngày |
| 0.9 | Viết Terms of Service | Critical | 0.5 ngày |
| 0.10 | Viết Privacy Policy | Critical | 0.5 ngày |
| 0.11 | Viết Refund Policy | Critical | 0.5 ngày |
| 0.12 | Mua ionCube Encoder license (hoặc setup online encoding) | Critical | 0.5 ngày |
| 0.13 | Tạo universal build script (`build.sh`) | High | 0.5 ngày |
| 0.14 | Tạo shared `LicenseChecker.php` template | Critical | 1 ngày |
| 0.15 | Tạo shared license notice CSS snippet | Medium | 0.5 ngày |
| 0.16 | Website: home page, /about, /legal, /support | High | 2 ngày |
| 0.17 | Website: product page template (reusable layout) | High | 1 ngày |
| 0.18 | Configure product upgrade path template trong WHMCS | Medium | 0.5 ngày |

**Subtotal Phase 0: ~12 ngày**

### Phase 1 — First Product: Pricing Calculator (Tuần 3–4)

| # | Task | Est. |
|---|---|---|
| 1.1 | Rebrand codebase: hvn_ → mj_ (all files, functions, CSS, JS, DB) | 1.5 ngày |
| 1.2 | Migration logic: tbl_hvn_ → tbl_mj_ | 0.5 ngày |
| 1.3 | Integrate LicenseChecker vào module (hooks + dashboard) | 1.5 ngày |
| 1.4 | License UI: notice banners, dashboard card, config field | 1 ngày |
| 1.5 | Tạo 8 products trong WHMCS (MJ-PRC-STD/PRO/OSS) | 0.5 ngày |
| 1.6 | Set upgrade paths cho 8 products | 0.5 ngày |
| 1.7 | Upload download files (encoded + Open Source ZIPs) | 0.5 ngày |
| 1.8 | Tạo support department "Pricing Calculator Support" | 0.25 ngày |
| 1.9 | Tạo KB: Installation, Configuration, FAQ, Troubleshooting | 1 ngày |
| 1.10 | Website: /products/pricing-calculator/ page | 1 ngày |
| 1.11 | Build encoded + Open Source packages | 0.5 ngày |
| 1.12 | Test encoded on PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x | 1.5 ngày |
| 1.13 | End-to-end test: order → pay → download → install → activate | 1 ngày |
| 1.14 | Soft launch → Public launch | 0.5 ngày |

**Subtotal Phase 1: ~11 ngày**

### Phase N — Mỗi Product Tiếp Theo (ước tính)

Vì infrastructure đã sẵn, mỗi product mới chỉ cần:

| Task | Est. |
|---|---|
| Develop module (scope tùy complexity) | Tùy module |
| Integrate LicenseChecker (copy + namespace) | 0.5 ngày |
| Tạo 8 WHMCS products + upgrade paths + downloads | 0.5 ngày |
| Support department + KB articles | 0.5 ngày |
| Product landing page | 0.5 ngày |
| Build encoded/Open Source + test | 1 ngày |
| End-to-end test | 0.5 ngày |

**Overhead per product: ~3.5 ngày** (ngoài development time của module).

---

## 11. Module Source Code Organization

### 11.1 Monorepo Structure

```
modulejet/                              ← Single repo for all products
│
├── products/
│   ├── mj_pricing_calculator/          ← Product #1
│   │   └── src/
│   │       └── mj_pricing_calculator/  ← Module source
│   │           ├── mj_pricing_calculator.php
│   │           ├── hooks.php
│   │           ├── lib/
│   │           │   └── LicenseChecker.php
│   │           ├── assets/
│   │           ├── templates/
│   │           ├── lang/
│   │           ├── whmcs.json
│   │           └── logo.png
│   │
│   ├── mj_invoice_manager/            ← Product #2 (future)
│   │   └── src/
│   │       └── mj_invoice_manager/
│   │           └── ... (same structure)
│   │
│   └── mj_server_manager/             ← Product #3 (future)
│
├── shared/
│   ├── LicenseChecker.template.php     ← Master template, copy vào mỗi module
│   ├── license-notice.css              ← CSS snippet dùng chung
│   └── COMMERCIAL.txt                  ← Proprietary license text
│
├── scripts/
│   ├── build.sh                        ← Universal build (section 7.2)
│   ├── build-all.sh                    ← Build all products
│   └── test-matrix.sh                  ← Test PHP/WHMCS compatibility
│
├── build/                              ← Output (gitignored)
│   ├── mj-pricing-encoded-v1.2.0.zip
│   ├── mj-pricing-opensource-v1.2.0.zip
│   ├── mj-invoice-encoded-v1.0.0.zip
│   └── ...
│
├── docs/
│   ├── platform/
│   │   ├── licensing-platform-plan.md  ← This document
│   │   ├── dev-standard.md
│   │   └── product-registry.md
│   └── products/
│       ├── pricing-calculator/
│       │   ├── README.md
│       │   ├── CHANGELOG.md
│       │   └── licensing-plan.md
│       └── invoice-manager/
│
└── README.md                           ← Repo overview
```

### 11.2 Shared Code Workflow

Khi tạo module mới:

```bash
# 1. Scaffold
mkdir -p products/mj_{name}/src/mj_{name}/{lib,assets/{js,css,js/vendor},templates,lang}

# 2. Copy LicenseChecker template
cp shared/LicenseChecker.template.php \
   products/mj_{name}/src/mj_{name}/lib/LicenseChecker.php

# 3. Update namespace in LicenseChecker.php
sed -i 's/MJ\\Template/MJ\\{ProductName}/' \
   products/mj_{name}/src/mj_{name}/lib/LicenseChecker.php

# 4. Copy Alpine.js fallback
cp products/mj_pricing_calculator/src/mj_pricing_calculator/assets/js/vendor/alpine.min.js \
   products/mj_{name}/src/mj_{name}/assets/js/vendor/

# 5. Copy license notice CSS snippet into new CSS file
cat shared/license-notice.css >> products/mj_{name}/src/mj_{name}/assets/css/mj-{abbr}.css
```

---

## 12. Monitoring & Operations

### 12.1 KPIs (across all products)

| Metric | Target |
|---|---|
| Total licenses sold (all products) | Growing month-over-month |
| Monthly recurring revenue (MRR) | Track per product |
| License activation success rate | > 95% |
| Support response time (STD) | < 48h |
| Support response time (PRO/OSS) | < 24h |
| Refund rate | < 10% |
| WHMCS licensing server uptime | > 99.5% |

### 12.2 Operational Tasks (recurring)

| Task | Frequency |
|---|---|
| Monitor WHMCS licensing logs | Weekly |
| Prune old license access logs | Auto (90 days) |
| Update ionCube encoding for new PHP versions | Per PHP release |
| Test module compatibility with new WHMCS versions | Per WHMCS release |
| Update download files after new module release | Per release |
| Review support tickets for common issues → update KB | Monthly |
| Review refund requests | As needed |
| Backup WHMCS database on modulejet.com | Daily (automated) |

### 12.3 Alerts

| Event | Action |
|---|---|
| Payment gateway error rate > 5% | Investigate immediately |
| License check failures spike | Check modulejet.com uptime + SSL |
| Support ticket queue > 20 unresolved | Prioritize + hire help |
| Refund rate > 15% for any product | Review product quality + pricing |

---

## 13. Risk Management

| Rủi ro | Impact | Xác suất | Giải pháp |
|---|---|---|---|
| modulejet.com downtime | License checks fail | Medium | Local key 15 ngày + grace 5 ngày, server monitoring |
| ionCube Encoder giá cao | Build cost | Low | Online encoding service thay thế, amortize across products |
| ionCube Loader không có trên customer server | Cannot run encoded module | Medium | KB article "How to Install ionCube", check in module installer |
| Encoded module bị crack | Revenue loss | Low | Focus value on support + updates, accept small % loss |
| Open Source module bị redistribute | Revenue loss | Medium | Proprietary license + DMCA, license check vẫn hoạt động |
| WHMCS deprecate Software Licensing | Must migrate | Very Low | WHMCS dùng chính hệ thống này, unlikely to remove |
| New PHP version breaks ionCube | Encoded modules fail | Medium | Test early, re-encode promptly, communicate timeline |
| Customer uses wrong product key | Module doesn't work | Low | `isCorrectProduct()` check + clear error message |
| Too many products, hard to manage | Operational overhead | Medium | Standardized process, templates, build scripts |

---

## 14. Checklist Tổng Thể

### One-Time Platform Setup
- [ ] WHMCS installed + SSL on modulejet.com
- [ ] Software Licensing Addon purchased + activated
- [ ] PayPal gateway configured + tested
- [ ] Stripe gateway configured + tested
- [ ] Email templates customized
- [ ] Support departments created (Billing, Pre-Sales)
- [ ] General KB articles written
- [ ] Legal pages published (TOS, Privacy, Refund)
- [ ] ionCube Encoder available
- [ ] Universal build.sh script working
- [ ] Shared LicenseChecker.template.php finalized
- [ ] Website home page + template live
- [ ] Product registry document created

### Per-Product Launch (repeat for each new product)
- [ ] Product code registered in registry (exactly 3 chars)
- [ ] Module code complete + LicenseChecker integrated
- [ ] `isCorrectProduct()` validates against expected code
- [ ] 8 WHMCS products created with correct prefixes
- [ ] Domain rules set (STD=restricted, PRO/OSS=unlimited)
- [ ] Upgrade paths configured
- [ ] Encoded + Open Source ZIPs built
- [ ] Encoded tested on PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x
- [ ] Download files uploaded + linked to products
- [ ] Support department created
- [ ] KB articles written (Install, Config, FAQ, Troubleshoot)
- [ ] Product landing page live
- [ ] End-to-end test passed (order → activate)
- [ ] Pricing verified in all currencies
- [ ] Soft launch → public launch

---

## 15. Tóm Tắt

| Yếu tố | Giải pháp |
|---|---|
| Platform | 1 WHMCS instance trên modulejet.com |
| Licensing | WHMCS Software Licensing Addon ($99.95 one-time) |
| Key format | `MJ-{PRODUCT}-{TIER}-{random32}` (PRODUCT = 3 chars) |
| Tiers | Standard (1 domain, encoded) / Professional (unlimited, encoded) / Open Source (unlimited, plain PHP, proprietary license) |
| Cycles | Monthly / Annually / Lifetime (Lifetime = support trọn đời) |
| SKUs per product | 8 (3 tiers × 3 cycles, trừ Open Source Monthly) |
| Encoding | ionCube PHP 8.1+ |
| License verification | Shared `LicenseChecker.php`, 15-day local key cache |
| Build system | Universal `build.sh` cho mọi product |
| Time to add new product | ~3.5 ngày overhead (ngoài dev time) |
| Code organization | Monorepo, shared templates, per-product directories |

---

*ModuleJET Platform Licensing Plan v1.1 — March 2026*  
*Infrastructure xây một lần, tái sử dụng cho mọi sản phẩm.*
