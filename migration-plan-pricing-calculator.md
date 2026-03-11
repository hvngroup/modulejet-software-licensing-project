# ModuleJET — Kế Hoạch Triển Khai Software Licensing

**Module:** Pricing Calculator (sản phẩm đầu tiên của ModuleJET)  
**Thương hiệu:** ModuleJET — modulejet.com  
**Email hỗ trợ:** support@modulejet.com  
**Licensing Platform:** WHMCS Software Licensing Addon  
**Ngày lập:** 11/03/2026  
**Trạng thái:** Draft v3

---

## 1. Tổng Quan

### 1.1 Mục tiêu

Triển khai hệ thống licensing cho module Pricing Calculator sử dụng WHMCS Software Licensing Addon, cho phép ModuleJET bán license qua website modulejet.com với WHMCS làm nền tảng billing + licensing.

### 1.2 Mô hình sản phẩm hiện tại

Module hiện tại: `hvn_pricing_calculator` v1.1.0 (production, đã hoàn thành 3 phases). Cần rebrand sang **MJ Pricing Calculator** (thương hiệu ModuleJET) và tích hợp license verification.

### 1.3 Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────┐
│                    KHÁCH HÀNG                            │
│  (WHMCS Admin cài module Pricing Calculator)            │
│                                                          │
│  Module gọi license check → modulejet.com/whmcs         │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS (license verify)
                       ▼
┌─────────────────────────────────────────────────────────┐
│              MODULEJET.COM (WHMCS Instance)              │
│                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │ Software         │  │ Product Catalog               │  │
│  │ Licensing Addon  │  │ 8 license products            │  │
│  │ (license keys,   │  │ (không cần Support/Updates    │  │
│  │  verify API)     │  │  addon riêng cho Lifetime)    │  │
│  └─────────────────┘  └──────────────────────────────┘  │
│                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │ Client Portal    │  │ Download Manager              │  │
│  │ (order, pay,     │  │ (encoded + unencoded          │  │
│  │  manage license) │  │  packages)                    │  │
│  └─────────────────┘  └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 2. License Tiers & Pricing Structure

### 2.1 Key Prefix Convention (chuẩn hóa multi-product)

```
MJ-{PRODUCT}-{TIER}-{random_key}
```

| Segment | Giá trị | Mô tả |
|---|---|---|
| `MJ` | Cố định | Thương hiệu ModuleJET |
| `{PRODUCT}` | `PC` = Pricing Calculator | Mã sản phẩm (2–4 ký tự) |
| `{TIER}` | `STD` / `PRO` / `UEN` | Standard / Professional / Unencoded |

Ví dụ key: `MJ-PC-STD-A1B2C3D4E5F6G7H8...`

Sản phẩm tương lai: `MJ-IM-STD-` (Invoice Manager), `MJ-SM-PRO-` (Server Manager), v.v.

### 2.2 Bảng sản phẩm

| # | Tên sản phẩm | Billing | Encoding | Domain | Key Prefix | Support/Updates |
|---|---|---|---|---|---|---|
| 1 | Monthly Standard License | Monthly | ionCube | 1 domain | `MJ-PC-STD-` | Khi còn active |
| 2 | Annual Standard License | Annually | ionCube | 1 domain | `MJ-PC-STD-` | Khi còn active |
| 3 | Lifetime Standard License | One Time | ionCube | 1 domain | `MJ-PC-STD-` | **Lifetime** |
| 4 | Monthly Professional License | Monthly | ionCube | Unlimited | `MJ-PC-PRO-` | Khi còn active |
| 5 | Annual Professional License | Annually | ionCube | Unlimited | `MJ-PC-PRO-` | Khi còn active |
| 6 | Lifetime Professional License | One Time | ionCube | Unlimited | `MJ-PC-PRO-` | **Lifetime** |
| 7 | Annual Unencoded License | Annually | Không mã hóa | Unlimited | `MJ-PC-UEN-` | Khi còn active |
| 8 | Lifetime Unencoded License | One Time | Không mã hóa | Unlimited | `MJ-PC-UEN-` | **Lifetime** |

> **"Unencoded" ≠ Open Source.** License vẫn thương mại (proprietary). Khách mua quyền sử dụng source code không mã hóa, có thể đọc/tùy chỉnh nhưng **không được phân phối lại**.

### 2.3 So sánh tính năng theo tier

| Tính năng | Standard | Professional | Unencoded |
|---|---|---|---|
| Pricing Calculator Toolbar | ✅ | ✅ | ✅ |
| Config Options Manager | ✅ | ✅ | ✅ |
| Presets & Settings | ✅ | ✅ | ✅ |
| Domain giới hạn | 1 domain | Unlimited | Unlimited |
| Source code | ionCube encoded | ionCube encoded | Plain PHP |
| Quyền sửa code | ❌ | ❌ | ✅ (chỉ cho domain mình) |
| Quyền phân phối lại | ❌ | ❌ | ❌ |
| Hỗ trợ kỹ thuật | Email (48h) | Priority (24h) | Priority (24h) |
| Branding removal | ❌ | ✅ | ✅ |
| White-label | ❌ | ❌ | ✅ |

### 2.4 Pricing

| Tier | Monthly | Annually | Lifetime |
|---|---|---|---|
| Standard | $4.95/mo | $39.95/yr (~33% off) | $99.95 |
| Professional | $9.95/mo | $79.95/yr (~33% off) | $199.95 |
| Unencoded | — | $119.95/yr | $299.95 |

### 2.5 Support/Updates Policy

| Billing | Support/Updates |
|---|---|
| Monthly | Bao gồm khi subscription active |
| Annually | Bao gồm khi subscription active |
| Lifetime | **Bao gồm trọn đời** — không cần addon riêng |

Triển khai: WHMCS tự quản lý qua product status. Lifetime = always Active = luôn download được.

---

## 3. Cấu Hình WHMCS Software Licensing Addon

### 3.1 Cài đặt trên modulejet.com

- Mua WHMCS Software Licensing Addon ($99.95 one-time)
- Upload + activate tại Configuration → Addon Modules
- Maximum Allowed Reissues: `3`
- Auto Logs Prune: `90 days`
- Public License Verification Tool: ✅ Enable

### 3.2 Product Group

```
Product Group: "ModuleJET — Pricing Calculator"
├── Standard: Monthly ($4.95) / Annual ($39.95) / Lifetime ($99.95)
├── Professional: Monthly ($9.95) / Annual ($79.95) / Lifetime ($199.95)
└── Unencoded: Annual ($119.95) / Lifetime ($299.95)
```

### 3.3 Module Settings per product

**Chung:**
- Module Name: `License Software`
- Key Length: `32`
- Allow Reissue: ✅
- Support/Updates Addon: **Để trống**

**Standard (#1–3):**
- Key Prefix: `MJ-PC-STD-`
- Allow Domain Conflict: ❌ No (enforce 1 domain)
- Allow IP Conflict: ✅ / Allow Directory Conflict: ✅

**Professional (#4–6):**
- Key Prefix: `MJ-PC-PRO-`
- Allow Domain Conflict: ✅ Yes (unlimited)

**Unencoded (#7–8):**
- Key Prefix: `MJ-PC-UEN-`
- Allow Domain Conflict: ✅ Yes (unlimited)

### 3.4 Downloads

| Download | Products | Require Active |
|---|---|---|
| `mj-pricing-calculator-encoded-v1.2.0.zip` | Standard #1–3, Professional #4–6 | ✅ |
| `mj-pricing-calculator-unencoded-v1.2.0.zip` | Unencoded #7–8 | ✅ |

---

## 4. Thay Đổi Codebase

### 4.1 Rebrand: hvn_pricing_calculator → mj_pricing_calculator

| Hạng mục | Cũ | Mới |
|---|---|---|
| Directory | `hvn_pricing_calculator/` | `mj_pricing_calculator/` |
| Main file | `hvn_pricing_calculator.php` | `mj_pricing_calculator.php` |
| Function prefix | `hvn_pricing_calculator_` | `mj_pricing_calculator_` |
| Helper prefix | `hvn_pricing_` | `mj_pricing_` |
| DB table | `tbl_hvn_pricing_presets` | `tbl_mj_pricing_presets` |
| CSS class prefix | `hvn-` | `mj-` |
| CSS filename | `hvn-pricing.css` | `mj-pricing.css` |
| JS filename | `hvn-pricing.js` | `mj-pricing.js` |
| JS config | `window.hvnPricingConfig` | `window.mjPricingConfig` |
| JS namespace | `window.hvnPricing` | `window.mjPricing` |
| Alpine components | `hvnPricingToolbar()` | `mjPricingToolbar()` |
| | `hvnConfigManager()` | `mjConfigManager()` |
| | `hvnConfigToolbar()` | `mjConfigToolbar()` |
| Constants | `HVN_PRICING_DIR` | `MJ_PRICING_DIR` |
| Dev constant | `HVN_DEV` | `MJ_DEV` |
| Author | HVN GROUP | ModuleJET |
| Author URL | hvn.vn | modulejet.com |
| whmcs.json name | `hvn-pricing-calculator` | `mj-pricing-calculator` |

**Migration:** `_activate()` detect `tbl_hvn_pricing_presets` → copy to `tbl_mj_pricing_presets` → giữ bảng cũ (rollback safety).

### 4.2 License Verification

#### 4.2.1 Flow

```
Module load
  → Đọc license key từ settings (admin nhập khi config)
  → Đọc local key từ DB (tbladdonmodules, setting 'localKey')
  → check_license()
      ├─ Active     → Module hoạt động bình thường
      ├─ Invalid    → Warning, disable toolbar
      ├─ Suspended  → Thông báo, link client area
      └─ Expired    → Thông báo, link renew
```

#### 4.2.2 Config field

```php
'licenseKey' => [
    'FriendlyName' => 'License Key',
    'Type'         => 'text',
    'Size'         => '50',
    'Default'      => '',
    'Description'  => 'Enter your ModuleJET license key. '
                    . '<a href="https://modulejet.com" target="_blank">Purchase</a>',
],
```

#### 4.2.3 LicenseChecker class

File: `lib/LicenseChecker.php`

```php
<?php
namespace MJ\PricingCalculator;

class LicenseChecker
{
    private string $licenseKey;
    private string $localKey;
    private string $whmcsUrl = 'https://modulejet.com/whmcs/';
    private string $licensingSecretKey = '';
    private int $localKeyDays = 15;
    private int $allowCheckFailDays = 5;

    public function check(): array { /* WHMCS check_sample_code.php based */ }
    
    public function isValid(): bool
    {
        return $this->check()['status'] === 'Active';
    }
    
    public function getTier(): string
    {
        if (preg_match('/^MJ-[A-Z]{2,4}-(STD|PRO|UEN)-/', $this->licenseKey, $m)) {
            return match($m[1]) {
                'STD' => 'standard',
                'PRO' => 'professional',
                'UEN' => 'unencoded',
                default => 'unknown',
            };
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

#### 4.2.4 License gate (hooks.php)

```php
add_hook('AdminAreaFooterOutput', 1, function($vars) {
    $page = mj_pricing_detectPage();
    if (!$page) return '';

    $settings = mj_pricing_getSettings();
    $licenseKey = $settings['licenseKey'] ?? '';
    
    if (empty($licenseKey)) {
        return mj_pricing_renderLicenseNotice('no_key');
    }
    
    $checker = new \MJ\PricingCalculator\LicenseChecker($licenseKey, $settings['localKey'] ?? '');
    $result = $checker->check();
    
    if (!empty($result['localkey'])) {
        mj_pricing_saveLocalKey($result['localkey']);
    }
    
    if ($result['status'] !== 'Active') {
        return mj_pricing_renderLicenseNotice($result['status'], $result);
    }
    
    // License valid → proceed with toolbar injection ...
});
```

#### 4.2.5 Caching

- Local key: `tbladdonmodules` setting `localKey`
- Remote check: mỗi 15 ngày
- Grace period: 5 ngày nếu server unreachable
- Sau grace: warning banner, **không crash WHMCS**

### 4.3 Hai bản phân phối

#### Encoded (Standard + Professional)

```
mj_pricing_calculator/
├── mj_pricing_calculator.php        ← ionCube encoded
├── hooks.php                        ← ionCube encoded
├── lib/
│   └── LicenseChecker.php          ← ionCube encoded
├── assets/
│   ├── js/
│   │   ├── mj-pricing.js           ← Plain
│   │   └── vendor/
│   │       └── alpine.min.js       ← Plain (3rd party)
│   └── css/
│       └── mj-pricing.css          ← Plain
├── templates/
│   ├── admin-dashboard.php         ← Plain
│   ├── admin-presets.php           ← Plain
│   └── config-options-manager.php  ← Plain
├── lang/
│   ├── english.php                 ← Plain
│   └── vietnamese.php              ← Plain
├── whmcs.json
├── logo.png
├── LICENSE                          ← Proprietary
└── README.md
```

#### Unencoded

Cùng cấu trúc, tất cả `.php` plain text. License vẫn proprietary.

---

## 5. Quy Trình Mua & Kích Hoạt

### 5.1 Customer Journey

```
1. modulejet.com → xem Pricing Calculator → so sánh plans
2. Chọn plan + cycle → Cart → Checkout → Thanh toán
3. WHMCS generate key (MJ-PC-XXX-...)
4. Email: license key + download link + hướng dẫn
5. Upload module → Activate → nhập key → Save
6. Module verify → hoạt động
```

### 5.2 Client Area

- View license + status
- Reissue (max 3 lần)
- Download (require Active)
- Upgrade tier/cycle
- Manage billing

### 5.3 Upgrade Paths

| Từ → Đến | Cách |
|---|---|
| Standard → Professional | WHMCS Product Upgrade (prorate) |
| Monthly → Annual/Lifetime | WHMCS Product Upgrade |
| Any Encoded → Unencoded | WHMCS Product Upgrade (prorate) |
| Unencoded Annual → Lifetime | WHMCS Product Upgrade |

---

## 6. Website modulejet.com

```
modulejet.com/
├── Home (landing — Pricing Calculator spotlight)
├── Products/ → Pricing Calculator (features, pricing, docs, changelog)
├── Client Area (WHMCS: licenses, downloads, tickets, invoices)
├── Knowledge Base (install, FAQ, troubleshoot)
├── Support (WHMCS tickets)
├── Terms of Service / Privacy Policy / Refund Policy (30 ngày)
```

Payment: PayPal + Stripe. Email templates customized. Support department: "Pricing Calculator Support".

---

## 7. ionCube Encoding

### 7.1 Build encoded

```bash
#!/bin/bash
SRC="./src/mj_pricing_calculator"
OUT="./build/encoded/mj_pricing_calculator"
VER="1.2.0"

rm -rf "./build/encoded" && mkdir -p "$OUT"
cp -r "$SRC/"* "$OUT/"

ioncube_encoder --php81 --optimize max --obfuscate all \
  --into "$OUT" \
  "$SRC/mj_pricing_calculator.php" \
  "$SRC/hooks.php" \
  "$SRC/lib/LicenseChecker.php"

cd "./build/encoded"
zip -r "../mj-pricing-calculator-encoded-v${VER}.zip" mj_pricing_calculator/
```

### 7.2 Build unencoded

```bash
#!/bin/bash
SRC="./src/mj_pricing_calculator"
OUT="./build/unencoded/mj_pricing_calculator"
VER="1.2.0"

rm -rf "./build/unencoded" && mkdir -p "$OUT"
cp -r "$SRC/"* "$OUT/"
cp ./licenses/COMMERCIAL.txt "$OUT/LICENSE"

cd "./build/unencoded"
zip -r "../mj-pricing-calculator-unencoded-v${VER}.zip" mj_pricing_calculator/
```

### 7.3 Test matrix

| PHP | WHMCS | ionCube Loader |
|---|---|---|
| 8.1 | 8.x | 13.0.2+ |
| 8.2 | 8.x / 9.x | 13.0.2+ |
| 8.3 | 9.x | 14.4.0+ |

---

## 8. Timeline

### Phase 1 — Rebrand & License (Tuần 1–2)

| # | Task | Est. |
|---|---|---|
| 1.1 | Rebrand toàn bộ: hvn_ → mj_ (files, functions, CSS, JS, DB) | 1.5 ngày |
| 1.2 | Migration logic: tbl_hvn_ → tbl_mj_ | 0.5 ngày |
| 1.3 | Tạo `lib/LicenseChecker.php` | 1 ngày |
| 1.4 | `licenseKey` field + license gate trong hooks | 1 ngày |
| 1.5 | License UI: dashboard card, warning notices | 1 ngày |
| 1.6 | Local key caching + graceful degradation | 0.5 ngày |
| 1.7 | Update whmcs.json, lang files, README | 0.5 ngày |
| 1.8 | Test tất cả license status | 1 ngày |

### Phase 2 — WHMCS trên modulejet.com (Tuần 2–3)

| # | Task | Est. |
|---|---|---|
| 2.1 | Cài WHMCS + Software Licensing Addon | 1 ngày |
| 2.2 | Tạo 8 products + upgrade paths | 1.5 ngày |
| 2.3 | Payment gateways + email templates | 1 ngày |
| 2.4 | Download manager + knowledgebase + support | 1.5 ngày |

### Phase 3 — Build & Test (Tuần 3)

| # | Task | Est. |
|---|---|---|
| 3.1 | ionCube setup + build scripts | 1 ngày |
| 3.2 | Test encoded on PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x | 2 ngày |

### Phase 4 — Website & Launch (Tuần 4)

| # | Task | Est. |
|---|---|---|
| 4.1 | Landing page + pricing table + docs | 3 ngày |
| 4.2 | TOS + Privacy + Refund Policy | 1 ngày |
| 4.3 | End-to-end test: order → activate | 1 ngày |

**Tổng: ~4 tuần**

---

## 9. Cấu Trúc Thư Mục Dự Án

```
mj-pricing-calculator/                 ← Repo root
│
├── src/
│   └── mj_pricing_calculator/         ← Module source
│       ├── mj_pricing_calculator.php  # Main: config, activate, output, AJAX
│       ├── hooks.php                  # Hooks + license gate
│       ├── lib/
│       │   └── LicenseChecker.php     # License verification
│       ├── assets/
│       │   ├── js/
│       │   │   ├── mj-pricing.js      # Single IIFE: Utils, Calc, Alpine
│       │   │   └── vendor/
│       │   │       └── alpine.min.js  # Alpine.js 3.14.8 local fallback
│       │   └── css/
│       │       └── mj-pricing.css     # Ant Design–inspired design system
│       ├── templates/
│       │   ├── admin-dashboard.php    # Dashboard + license status
│       │   ├── admin-presets.php      # Preset CRUD
│       │   └── config-options-manager.php
│       ├── lang/
│       │   ├── english.php
│       │   └── vietnamese.php
│       ├── whmcs.json
│       ├── logo.png
│       └── LICENSE
│
├── build/                             ← gitignored
│   ├── mj-pricing-calculator-encoded-v1.2.0.zip
│   └── mj-pricing-calculator-unencoded-v1.2.0.zip
│
├── scripts/
│   ├── build-encoded.sh
│   └── build-unencoded.sh
│
├── licenses/
│   └── COMMERCIAL.txt
│
├── docs/
│   ├── installation.md
│   ├── configuration.md
│   ├── licensing-faq.md
│   └── CHANGELOG.md
│
└── README.md
```

---

## 10. Checklist Trước Khi Launch

### Technical
- [ ] Không còn reference nào đến `hvn` trong code
- [ ] License check hoạt động với 3 prefixes (STD, PRO, UEN)
- [ ] Graceful degrade — không crash WHMCS
- [ ] ionCube OK trên PHP 8.1/8.2/8.3 + WHMCS 8.x/9.x
- [ ] Local key caching hoạt động
- [ ] Migration từ bản HVN hoạt động
- [ ] Build scripts tạo đúng 2 bản

### Business
- [ ] Payment gateways test OK
- [ ] Welcome email có key + download link
- [ ] TOS (bao gồm: Unencoded ≠ open source, cấm redistribute)
- [ ] Refund Policy (30 ngày)
- [ ] Support ticket system OK
- [ ] Public license verification OK

---

## 11. Mở Rộng Sản Phẩm Tương Lai

Khi ra sản phẩm mới (ví dụ Invoice Manager):

1. Key prefix: `MJ-IM-STD-`, `MJ-IM-PRO-`, `MJ-IM-UEN-`
2. Directory: `mj_invoice_manager/`
3. DB: `tbl_mj_invoice_{entity}`
4. LicenseChecker: tái sử dụng, chỉ đổi namespace
5. Infrastructure: tận dụng toàn bộ WHMCS + payment + portal đã có

---

*Tài liệu v3 — prefix `mj_` / `mj-` / `MJ` toàn bộ.*  
*Infrastructure xây ở đây tái sử dụng cho mọi sản phẩm ModuleJET sau.*
